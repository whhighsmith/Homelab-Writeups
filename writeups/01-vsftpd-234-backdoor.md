# Finding 01: vsftpd 2.3.4 Backdoor (Unauthenticated Remote Root — Metasploitable2)
			
## Target:	Metasploitable2 (10.10.10.20)

## Date:	July 2026

## Environment:	[homelab-build.md](../homelab-build.md)	

## Tester:	Will

## Tools Used:	nmap, Metasploit Framework	Severity	Critical

---
# Executive Summary
A vulnerability scan and exploitation exercise was conducted against a lab target running vsftpd 2.3.4. An unauthenticated remote attacker could exploit a known backdoor in this software version to gain full root-level access to the system. This represents a critical severity finding.

---
# Scope & Environment
Target: Metasploitable2, `10.10.10.20`
Network: isolated lab segment (vmbr1) — no internet or home LAN exposure
Tools: nmap, Metasploit Framework
Environment reference: full lab build and network design documented separately in homelab-build.md
Authorization: self-owned lab environment, isolated from production/home network

---
# Methodology
Reconnaissance — nmap service/version scan against the target
Vulnerability identification — matched an identified service version to a known critical vulnerability
Exploitation — used the corresponding Metasploit module to trigger the vulnerability
Post-exploitation verification — confirmed access level obtained on the target
---
# Reconnaissance
An nmap version/script scan was run against the target from Kali:
```bash
nmap -sV -sC 10.10.10.20
```
The scan returned numerous open ports consistent with Metasploitable2's intentionally vulnerable service set. Port 21 was identified running vsftpd 2.3.4 — a version with a well-documented, publicly known backdoor.

---
# Findings — Detailed
Finding 1: Unauthenticated Remote Root via vsftpd 2.3.4 Backdoor
Field	Details
Title	Unauthenticated Remote Root via vsftpd 2.3.4 Backdoor
Severity / CVSS	Critical (CVSS v3 ~9.8 — comparable public scoring for this CVE)
Affected Service	vsftpd 2.3.4, FTP service, TCP port 21
Description	The installed version of vsftpd (2.3.4) was a compromised release containing an intentionally inserted backdoor. A specially crafted string sent through the FTP USER field causes the service to open a listening shell on TCP port 6200, which can be connected to directly for unauthenticated command execution.
Evidence	nmap identified vsftpd 2.3.4 on port 21 (see Appendix, Fig. 1). Metasploit's `vsftpd_234_backdoor` module was used to trigger the backdoor and establish a session (Fig. 2). Post-exploitation commands confirmed root-level access (Fig. 3).
Impact	Full unauthenticated remote code execution as root. An attacker on the network segment could fully compromise the host with no credentials, no user interaction, and no prior access required.
Remediation	Upgrade vsftpd to a current, non-backdoored release; verify package integrity/signatures on install; restrict FTP service exposure to trusted network segments only; monitor for anomalous outbound connections on non-standard ports (e.g., 6200) originating from FTP service processes.
Exploitation Steps
The Metasploit module corresponding to this vulnerability was used from `msfconsole`:
```
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 10.10.10.20
set LHOST 10.10.10.10
exploit
```
The exploit succeeded and returned a Meterpreter session. Post-exploitation commands confirmed full root access:
```
meterpreter > getuid
Server username: root

meterpreter > sysinfo
```
---
# Detection Opportunities
Connecting this offensive finding back to defensive visibility is the most valuable part of this exercise for SOC-relevant skill-building.
Log/network signatures: an anomalous, non-standard string in the FTP USER command during the authentication attempt; an outbound/local connection immediately opened on TCP port 6200 shortly after the FTP session — a port with no legitimate association with FTP service behavior.
Data sources needed to detect this: FTP application/service logs, network flow data (netflow/Zeek), and EDR process-creation telemetry showing a shell spawned by the FTP service process.
Proposed detection logic: alert on any process spawned as a child of vsftpd/ftpd; alert on outbound or listening connections on port 6200; alert on FTP authentication attempts containing non-standard characters or exceeding expected username length/format.
Current tooling status: no SIEM/EDR was deployed in the lab at the time of this exercise — flagged as a next step to validate whether these signatures are caught by default rule sets.
6. Conclusion
This exercise validated a complete recon-to-exploitation cycle against a known critical vulnerability, resulting in full root access. It also produced a concrete set of detection signatures to test once a SIEM is deployed in the lab — the natural next step for this finding.
