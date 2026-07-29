# Home Lab Build Log

**Proxmox Hypervisor & Isolated Network Environment**

| | | | |
|---|---|---|---|
| **Hypervisor** | Proxmox VE 9 (Debian 13 "Trixie") | **Date** | July 2026 |
| **Hardware** | Small form factor PC (wiped, bare-metal install) | **Builder** | Will |

---

## 1. Overview

This document describes the design and build of a self-hosted penetration testing lab, constructed from a wiped small form factor PC. The environment uses Proxmox VE as a bare-metal hypervisor with a network-isolated internal segment for hosting intentionally vulnerable target machines. This is a living reference document — it covers the environment as built, and will be updated as new VMs, tools, or network segments are added.

> *Referenced by: individual finding write-ups (e.g., `01-vsftpd-234-backdoor.md`) rely on the environment described here rather than repeating setup details.*

## 2. Architecture

- **Hypervisor:** Proxmox VE 9, installed bare-metal (no host OS layer)
- **Management network (vmbr0):** bridged to the physical NIC, connected to the home LAN — `192.168.68.0/24`
- **Isolated lab network (vmbr1):** Linux bridge with no physical NIC attached — fully internal, no internet or home LAN access — `10.10.10.0/24`
- **Attack VM:** Kali Linux — dual-homed, one NIC on each network
- **Target VM(s):** isolated on vmbr1 only, no exception

**Design rationale:** any VM placed exclusively on vmbr1 has no network path to the internet or home LAN, regardless of what happens inside that VM (compromise, malware execution, etc.). This makes it safe to intentionally run vulnerable or hostile software without risk to production systems.

## 3. Hypervisor Installation

Proxmox VE was installed directly to bare metal using the official ISO written to USB. Management networking was configured with a static IP during install to keep the web UI address consistent:

```
IP Address (CIDR): 192.168.68.50/24
Gateway:           192.168.68.1
DNS Server:        192.168.68.1
Hostname (FQDN):   pve.homelab.local
```

Post-install, the default enterprise APT repository (requires paid subscription) was disabled and replaced with the free no-subscription repository:

```
deb http://download.proxmox.com/debian/pve trixie pve-no-subscription
```

## 4. Network Isolation Setup

A second Linux bridge, `vmbr1`, was created with no bridge ports assigned — meaning no physical NIC is attached to it. This is what makes the network fully internal/host-only.

- **vmbr0** — physical NIC attached, home LAN (`192.168.68.x`)
- **vmbr1** — no physical NIC, isolated internal-only network (`10.10.10.x`)

## 5. Attack VM: Kali Linux

Provisioned with 2 vCPU / 4GB RAM and two virtual NICs — one on vmbr0 (internet access for updates/tools), one on vmbr1 (reaching isolated lab targets).

### Issue: neither interface obtained an IP address

**Root cause:** NetworkManager had no active connection profile for either device. Resolved with:

```bash
sudo nmcli device connect eth0
sudo nmcli connection add type ethernet ifname eth1 con-name eth1-lab ip4 10.10.10.10/24
sudo nmcli connection up eth1-lab
```

Both connections were set to auto-connect on boot, and the system was fully updated (`apt update && apt full-upgrade`).

## 6. Target VM Provisioning: Metasploitable2

Metasploitable2 is distributed as a pre-built `.vmdk` disk image rather than an installable ISO. The image was transferred to the Proxmox host via SFTP (port 22) and imported into an empty VM shell:

```bash
qm importdisk <VMID> /root/Metasploitable.vmdk local-lvm
```

The VM's only network interface is vmbr1 — no internet or home LAN exposure by design.

### Issue: boot failure, dropped to (initramfs) recovery shell

**Root cause:** the imported disk was attached as a SCSI device by default, but Metasploitable's legacy kernel expects an IDE bus and has no SCSI driver support. Resolved by detaching the disk and reattaching it as `IDE0`, then setting it as the boot device.

Once booted, a static IP was set to place the target on the isolated lab subnet:

```bash
sudo ifconfig eth0 10.10.10.20 netmask 255.255.255.0
```

Connectivity between Kali (`10.10.10.10`) and Metasploitable2 (`10.10.10.20`) was confirmed via ping.

## 7. Current Environment Summary

- **Proxmox VE 9** — `192.168.68.50` (management)
- **Kali Linux** — `192.168.68.x` (DHCP, vmbr0) / `10.10.10.10` (vmbr1)
- **Metasploitable2** — `10.10.10.20` (vmbr1 only)

## 8. Lessons Learned

- Old/legacy virtual appliance disk images (`.vmdk`) often need IDE, not SCSI or VirtIO — check this first on any boot failure after import
- Kali's installer does not always create active NetworkManager profiles for every NIC automatically — verify with `nmcli device status` after first boot
- A bridge with no assigned physical port is the simplest way to get a fully isolated internal network in Proxmox — no VLANs or firewall rules required for basic isolation

## 9. Planned Additions

- [ ] SIEM deployment (Wazuh or Security Onion) on the isolated network for detection testing
- [ ] Additional target VMs: Samba/SMB-focused target, a web application target, a Windows/AD target
- [ ] Proxmox firewall rules on vmbr1 as a defense-in-depth exercise
