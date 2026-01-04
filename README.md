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

```bash
nmap -sV -p- --min-rate 5000 10.10.10.245
# Open Ports Identified:
# 22/tcp (SSH): OpenSSH 8.2p1
# 80/tcp (HTTP): Gunicorn (Python WSGI Server)
