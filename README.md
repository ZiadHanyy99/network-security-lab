# Network Security Lab

> Hands-on 10-week network security lab covering packet analysis, host enumeration, traffic capture, secure vs insecure protocols, and firewall configuration — all performed in an isolated VirtualBox environment using industry-standard tools.

**Author:** Ziad Hany Mohamed Salem  
**Department:** Cybersecurity  
**Environment:** Kali Linux + Ubuntu (VirtualBox)  
**Date:** May 2026

---

## Lab Overview

| Weeks | Topic |
|-------|-------|
| 1–2 | Lab setup, network configuration, and ICMP capture |
| 3–4 | Network scanning and enumeration with Nmap |
| 5–6 | Packet capture and HTTP vs HTTPS traffic analysis |
| 7–8 | Secure protocols — TLS inspection and FTP vs SFTP |
| 9–10 | Firewall configuration and rule enforcement with UFW |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Wireshark | Packet capture and protocol analysis |
| Nmap 7.99 | Host discovery, port scanning, OS fingerprinting |
| OpenSSL | TLS certificate and session inspection |
| UFW | Linux firewall rule configuration and testing |
| VirtualBox 7.0 | Isolated multi-VM lab environment |

---

## Week 1–2 — Lab Setup & Network Configuration

Built the full lab environment with two VMs — Kali Linux (attacker/analyst) and Ubuntu (target server) — connected over a Host-Only network adapter.

**VirtualBox lab environment — both VMs running:**

![VirtualBox VMs Running](screenshots/01-virtualbox-vms-running.png)

**Kali Linux — IP configuration and connectivity verification:**

Assigned IP addresses and verified VM-to-VM connectivity using `ping`. Kali received `192.168.56.108` on eth1.

![Kali IP Address and Ping](screenshots/02-kali-ip-addr-ping.png)

**Ubuntu — IP configuration and ping response:**

Ubuntu at `192.168.56.107` responding to ICMP — 0% packet loss confirmed.

![Ubuntu IP Address and Ping](screenshots/03-ubuntu-ip-addr-ping.png)

**Wireshark — ICMP packets captured:**

Captured ICMP echo request/reply packets between both machines in Wireshark, confirming network-layer connectivity.

![Wireshark ICMP Capture](screenshots/04-wireshark-icmp-capture.png)

---

## Week 3–4 — Network Scanning & Enumeration

Performed structured reconnaissance on the lab network using Nmap — the same workflow used in real penetration test engagements.

```bash
# Host discovery — identify all live machines on the subnet
nmap -sn 192.168.56.0/24

# TCP SYN scan — identify open ports (stealthy half-open scan)
sudo nmap -sS -p 1-1000 192.168.56.107

# Service version detection
sudo nmap -sV 192.168.56.107

# Full scan with OS fingerprinting — save to report file
sudo nmap -A 192.168.56.107 -oN nmap_week2_report.txt
```

**Host discovery + SYN scan + service version detection:**

4 live hosts discovered. Target at `192.168.56.107` has two open ports: `21/tcp (vsftpd 3.0.5)` and `22/tcp (OpenSSH 8.9p1)`.

![Nmap Host Discovery and SYN Scan](screenshots/05-nmap-host-discovery-syn-scan.png)

**OS fingerprinting and automated report generation:**

OS detected as Linux 4.15–5.19. Full results saved to `nmap_week2_report.txt`.

![Nmap OS Fingerprint and Report](screenshots/06-nmap-os-fingerprint-report.png)

**Nmap report file output:**

![Nmap Report File](screenshots/07-nmap-report-file.png)

---

## Week 5–6 — Packet Capture & Traffic Analysis

Captured and compared HTTP vs HTTPS (TLS) traffic to demonstrate the critical importance of encryption.

**HTTP traffic — plaintext visible in Wireshark:**

Using the `http` display filter, the full TCP handshake and HTTP GET request are visible in cleartext. Any attacker on the network can read the request, response headers, and body.

![Wireshark HTTP Plaintext Traffic](screenshots/08-wireshark-http-traffic.png)

**HTTPS/TLS traffic — fully encrypted:**

Using the `tls` display filter, only the TLS handshake metadata is visible (Client Hello, Server Hello). All application data is shown as `Application Data` — unreadable without the session keys.

![Wireshark TLS Encrypted Traffic](screenshots/09-wireshark-tls-traffic.png)

**Finding:** HTTP exposes all data in cleartext. HTTPS encrypts everything after the handshake. This is why HTTP must never be used for sensitive data.

---

## Week 7–8 — Secure Network Protocols

### TLS Certificate Inspection with OpenSSL

```bash
openssl s_client -connect example.com:443
```

**TLS certificate chain — verified against Cloudflare CA:**

Full certificate chain extracted: `example.com` → Cloudflare TLS Issuing ECC CA → SSL.com TLS Transit ECC CA → SSL.com TLS ECC Root CA 2022.

![OpenSSL TLS Certificate Chain](screenshots/10-openssl-tls-certificate.png)

**TLS session details — cipher suite and verification:**

Protocol: TLSv1.3 | Cipher: `TLS_AES_256_GCM_SHA384` | Verification: OK

![OpenSSL TLS Session Details](screenshots/11-openssl-tls-session.png)

![OpenSSL TLS Session Ticket](screenshots/12-openssl-tls-session2.png)

---

### FTP vs SFTP — Security Comparison

**FTP — credentials visible in Wireshark:**

Wireshark capture of an FTP session shows the username (`ziad`) and password (`ziad1234`) transmitted in complete plaintext. Anyone intercepting the traffic has the credentials instantly.

![Wireshark FTP Plaintext Credentials](screenshots/13-wireshark-ftp-plaintext-credentials.png)

**FTP vs SFTP — terminal comparison:**

FTP connects and transfers files but credentials travel unencrypted. SFTP connects over SSH, verifies the host key fingerprint, and transfers the same file — all encrypted.

![FTP vs SFTP Terminal](screenshots/14-ftp-vs-sftp-terminal.png)

**SFTP traffic in Wireshark — fully encrypted:**

SFTP runs over SSH. Wireshark shows only `SSHv2` encrypted packets — username, password, and file contents are completely hidden.

![Wireshark SFTP Encrypted Traffic](screenshots/15-wireshark-sftp-encrypted.png)

| Feature | FTP | SFTP |
|---------|-----|------|
| Encryption | None | Full (SSH) |
| Credentials in Wireshark | Visible in plaintext | Encrypted |
| File contents in Wireshark | Visible | Encrypted |
| Safe to use | Never | Always |

---

## Week 9–10 — Firewalls & Access Control

Configured UFW on the Ubuntu target and enforced rules blocking HTTP while allowing SSH.

```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw allow 22/tcp      # allow SSH
sudo ufw deny 80/tcp       # block HTTP
sudo ufw status verbose
```

**UFW rules active — SSH allowed, HTTP denied:**

![UFW Firewall Rules](screenshots/16-ufw-firewall-rules.png)

**Firewall enforcement test — HTTP blocked, SSH allowed:**

`curl` to port 80 returns `Failed to connect` — HTTP is blocked by the firewall. `ssh` to port 22 connects and authenticates successfully — SSH is permitted.

![Firewall Test HTTP Blocked SSH Allowed](screenshots/17-firewall-test-http-blocked-ssh-allowed.png)

---

## Key Takeaways

- Unencrypted protocols (HTTP, FTP) expose credentials and data to anyone capturing traffic on the network — demonstrated live with Wireshark
- Nmap enumeration reveals open ports, running services, and OS versions before any exploitation begins — the same information a real attacker would gather
- Firewall rules must be explicitly configured and tested — the default state of most systems allows far more traffic than necessary
- TLS 1.3 with strong cipher suites (AES-256-GCM) is the current standard minimum for any production service

---

## Lab Environment

```
VirtualBox Host (Windows)
├── Kali Linux VM   (192.168.56.108) — analyst machine
└── Ubuntu VM       (192.168.56.107) — target server

Network: Host-Only Adapter — fully isolated, no internet exposure
```

---

## Full Report

Complete lab documentation is available in [`report.pdf`](./report.pdf).

---

## Disclaimer

All activities were performed in a fully isolated VirtualBox lab environment. No real networks, systems, or third parties were targeted or affected.
