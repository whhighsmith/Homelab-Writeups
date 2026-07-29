# Home Lab Security Write-ups

A self-hosted penetration testing lab built from scratch on a Proxmox VE hypervisor, used to develop hands-on offensive security and detection-engineering skills. This repo documents the environment build and each exploitation exercise performed against it.

Built and maintained by Will — IT Coordinator (healthcare/HIPAA environment) working toward a SOC/Security Analyst role. CompTIA Security+ (SY0-701), Google Cybersecurity Certificate.

## Why this repo exists

Most of my day-to-day security experience comes from a live production environment (endpoint monitoring, Active Directory, phishing simulations, vulnerability scanning). This lab is where I build and demonstrate the offensive side — understanding how attacks actually work, end to end, so I can reason better about detection and response. Each write-up follows a consistent format: recon, vulnerability, exploitation, and — most importantly — **detection opportunities**, connecting the offensive finding back to what a SOC analyst would actually look for.

## Environment

* **Hypervisor:** Proxmox VE 9, bare-metal on a small form factor PC
* **Network design:** isolated internal bridge (no physical NIC, no internet/LAN access) hosting all vulnerable targets — full details in [`homelab-build.md`](./homelab-build.md)
* **Attack platform:** Kali Linux
* **Tools:** nmap, Metasploit Framework, (SIEM planned — see Roadmap)

Full build process, network isolation design, and troubleshooting notes: [`homelab-build.md`](./homelab-build.md)

## Write-ups

|#|Target / Vulnerability|Severity|Summary|
|-|-|-|-|
|01|[vsftpd 2.3.4 Backdoor](./writeups/01-vsftpd-234-backdoor.md)|Critical|Unauthenticated remote root via a known backdoor in vsftpd 2.3.4|

*(Updated as new exercises are completed.)*

## Repo structure

```
├── README.md                      # this file
├── homelab-build.md               # environment build log \& network design
├── writeups/
│   └── 01-vsftpd-234-backdoor.md  # individual finding write-ups
└── images/
    └── ...                        # screenshots referenced in write-ups
```

## Roadmap

* \[ ] Deploy a SIEM (Wazuh or Security Onion) on the isolated network and re-test whether documented detection signatures are caught by default rule sets
* \[ ] Add additional target VMs: SMB/Samba-focused target, a web application target (DVWA/Mutillidae), a Windows/Active Directory target
* \[ ] Layer in host- and network-level firewall rules as a defense-in-depth exercise
* \[ ] Continue the write-up series as new exercises are completed

## About me

IT Coordinator with hands-on experience in endpoint security monitoring, Active Directory administration, and compliance in a regulated (HIPAA) environment, actively transitioning into a SOC/Security Analyst role. Connect with me on [LinkedIn](https://www.linkedin.com/in/will-highsmith/) or reach out at [email](whhighsmith@gmail.com).

