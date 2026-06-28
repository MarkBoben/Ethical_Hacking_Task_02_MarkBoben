# Ethical Hacking Task 02 — Network Scanning & Service Enumeration

**Intern:** Mark Boben
**Organization:** White Band Associates
**Track:** Ethical Hacking
**Scan Date:** 2026-06-28
**Target System:** Kali Linux VM — localhost (127.0.0.1)

---

## Table of Contents

- [Part A — Install Nmap](#part-a--install-nmap)
- [Part B — Scan Your Local Machine](#part-b--scan-your-local-machine)
- [Part C — Service Version Detection](#part-c--service-version-detection)
- [Part D — Operating System Detection](#part-d--operating-system-detection)
- [Part E — Common Port Research](#part-e--common-port-research)
- [Part F — Scan Analysis](#part-f--scan-analysis)
- [Part G — Scan Report](#part-g--scan-report)

---

## Part A — Install Nmap

### Installation

Nmap was already present at its latest version on the Kali Linux VM. The following command confirmed it was up to date:

```bash
sudo apt update && sudo apt install nmap -y
```

Output confirmed:
> `nmap is already the newest version (7.99+dfsg-1kali1).`

### Version Verification

```bash
nmap --version
```

**Output:**
```
Nmap version 7.99 ( https://nmap.org )
Platform: x86_64-pc-linux-gnu
Compiled with: liblua-5.4.8 openssl-3.6.2 libssh2-1.11.1 libz-1.3.1
               libpcre2-10.46 libpcap-1.10.6 nmap-libdnet-1.18.0 ipv6
Available nsock engines: epoll poll select
```

![Nmap Version Check](Screenshots/Version%20check.png)

---

## Part B — Scan Your Local Machine

### Command Used

```bash
nmap localhost
```

Equivalent to:

```bash
nmap 127.0.0.1
```

### Output

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-28 10:04 +0530
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000060s latency).
Other addresses for localhost (not scanned): ::1
All 1000 scanned ports on localhost (127.0.0.1) are in ignored states.
Not shown: 1000 closed tcp ports (reset)

Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds
```

### Findings

| Field | Result |
|---|---|
| Host Status | Up |
| Total Ports Scanned | 1000 |
| Open Ports | **0** |
| Port Numbers | None |
| Services Running | None |

**Observation:** All 1000 TCP ports were in a closed/reset state. This is expected for a minimal, freshly configured Kali Linux VM where no servers or services (such as SSH, HTTP, or FTP) have been started.

![Basic Scan](Screenshots/Basic%20scan.png)

---

## Part C — Service Version Detection

### Command Used

```bash
nmap -sV localhost
```

The `-sV` flag enables service and version detection on discovered open ports.

### Output

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-28 10:05 +0530
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000040s latency).
Other addresses for localhost (not scanned): ::1
All 1000 scanned ports on localhost (127.0.0.1) are in ignored states.
Not shown: 1000 closed tcp ports (reset)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 0.69 seconds
```

### Service Table

| Port | Service | Version |
|---|---|---|
| — | — | No open ports detected |

**Observation:** Since no ports were open, service version detection returned no results. This confirms that the Kali VM is not running any listening network services at this time. In a real-world engagement, discovered services would be listed here with their exact versions — which are then cross-referenced against known CVEs for vulnerability assessment.

![Version Detection](Screenshots/Version%20detection.png)

---

## Part D — Operating System Detection

### Command Used

```bash
sudo nmap -O localhost
```

The `-O` flag enables OS fingerprinting, which works by analyzing the way a host responds to various crafted TCP/IP packets.

### Output

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-28 10:05 +0530
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000054s latency).
Other addresses for localhost (not scanned): ::1
All 1000 scanned ports on localhost (127.0.0.1) are in ignored states.
Not shown: 1000 closed tcp ports (reset)
Too many fingerprints match this host to give specific OS details
Network Distance: 0 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 1.69 seconds
```

### Findings

| Question | Answer |
|---|---|
| Was OS detected? | Partially — OS detection was performed but could not pinpoint a specific OS |
| OS Identified | Inconclusive — "Too many fingerprints match this host to give specific OS details" |
| Network Distance | 0 hops (loopback/localhost) |

### Why is OS Detection Useful During Penetration Testing?

OS detection is a critical step during the enumeration phase of a penetration test. Knowing the target's operating system allows a tester to:

- **Narrow down applicable exploits** — vulnerabilities are OS-specific; a Windows exploit won't work on Linux.
- **Identify expected services** — certain ports and services are default to specific OS versions.
- **Assess patch level** — older OS versions may have well-known unpatched vulnerabilities.
- **Plan privilege escalation paths** — kernel exploits and local privilege escalation techniques differ entirely between Linux and Windows.

In this case, the scan returned an inconclusive result because OS fingerprinting relies on analyzing responses from open or closed ports. With all 1000 ports closed (reset), Nmap did not have enough varied network responses to distinguish between Linux kernel fingerprints with confidence.

![OS Detection](Screenshots/OS%20detection.png)

---

## Part E — Common Port Research

| Port | Protocol | Purpose | Common Security Risks |
|---|---|---|---|
| **20/21** | FTP (TCP) | File Transfer Protocol — used to transfer files between a client and server. Port 21 is the control channel; port 20 is the data channel. | Transmits credentials and data in **plaintext**. Susceptible to brute-force, credential sniffing, and anonymous login misconfigurations. Largely replaced by SFTP/FTPS. |
| **22** | SSH (TCP) | Secure Shell — provides encrypted remote login and command execution on a server. | Brute-force attacks on weak passwords. Vulnerable if outdated versions (e.g., OpenSSH with known CVEs) are used. Poor key management can also be exploited. |
| **23** | Telnet (TCP) | Remote login protocol that allows command-line access to a remote system. | All data including usernames and passwords is sent in **plaintext**. Extremely insecure; considered obsolete. Should never be exposed on a network. |
| **25** | SMTP (TCP) | Simple Mail Transfer Protocol — used to send emails between mail servers. | Open relays can be abused to send spam or phishing emails. Susceptible to email spoofing and SMTP injection if not properly configured. |
| **53** | DNS (TCP/UDP) | Domain Name System — resolves human-readable domain names (e.g., google.com) to IP addresses. | DNS spoofing/cache poisoning, DNS amplification DDoS attacks, and zone transfer leaks (if AXFR is not restricted) can expose internal network structure. |
| **80** | HTTP (TCP) | HyperText Transfer Protocol — serves unencrypted web pages and content to browsers. | Traffic is in **plaintext**, vulnerable to man-in-the-middle (MITM) attacks. Web apps on port 80 are exposed to SQL injection, XSS, and CSRF attacks. |
| **110** | POP3 (TCP) | Post Office Protocol v3 — allows email clients to download emails from a mail server. | Credentials often sent in plaintext (without TLS). Vulnerable to credential sniffing and brute-force attacks. |
| **143** | IMAP (TCP) | Internet Message Access Protocol — allows email clients to access and manage emails directly on the server. | Similar to POP3 risks — plaintext transmission without TLS. Subject to brute-force and MITM attacks if unencrypted. |
| **443** | HTTPS (TCP) | HTTP Secure — encrypted version of HTTP using TLS/SSL for secure communication. | SSL/TLS misconfigurations (expired certs, weak cipher suites), Heartbleed-style vulnerabilities in older OpenSSL versions, and certificate spoofing attacks. |
| **445** | SMB (TCP) | Server Message Block — used for Windows file sharing, printer sharing, and inter-process communication. | Extremely high risk. Exploited by EternalBlue (WannaCry ransomware), SMB relay attacks, and credential harvesting. Should never be exposed to the internet. |
| **3389** | RDP (TCP) | Remote Desktop Protocol — provides graphical remote desktop access to Windows systems. | Brute-force attacks, BlueKeep vulnerability (CVE-2019-0708), credential stuffing, and man-in-the-middle attacks. High-value target for ransomware operators. |

---

## Part F — Scan Analysis

### 1. Which services are currently running?

Based on all three Nmap scans (basic, version detection, and OS detection), **no services are currently running** on this Kali Linux VM. All 1000 TCP ports scanned on localhost (127.0.0.1) returned as closed. The host is live on the network (latency confirmed), but no applications are actively listening on any port.

### 2. Are all open ports necessary?

Since there are no open ports on this system, there is no unnecessary exposure at this time. This represents a minimal attack surface — which is ideal from a security hardening perspective. In a typical system where services like SSH or HTTP are running, each open port should be evaluated to confirm it serves a legitimate, required purpose.

### 3. Which services could become security risks if misconfigured?

If services were to be installed and misconfigured on this system, the highest risks would come from:

- **SSH (port 22)** — If enabled with password authentication instead of key-based auth, it becomes a brute-force target.
- **HTTP/HTTPS (ports 80/443)** — Any web application deployed without proper input validation opens the door to injection and XSS attacks.
- **SMB (port 445)** — Enabling Windows-style file sharing on a Kali machine with weak permissions would be a critical risk.
- **Telnet (port 23)** — If ever enabled, it would expose all traffic in plaintext and should be disabled immediately.

### 4. Which port would you disable if it wasn't required?

If Telnet (port 23) were active, it would be the first port to disable. It transmits all data — including login credentials — in unencrypted plaintext, making it inherently insecure regardless of configuration. SSH (port 22) is a far superior and equally capable replacement for remote access and should always be used instead.

---

## Part G — Scan Report

### Professional Nmap Scan Report

| Field | Details |
|---|---|
| **Scan Date** | 2026-06-28 |
| **Scan Time** | 10:04 – 10:05 IST (+0530) |
| **Target System** | Kali Linux VM — localhost (127.0.0.1) |
| **Nmap Version** | 7.99 |
| **Scanner Platform** | x86_64-pc-linux-gnu |

---

### Commands Used

| Part | Command | Purpose |
|---|---|---|
| A | `nmap --version` | Verify Nmap installation |
| B | `nmap localhost` | Basic TCP port scan (top 1000 ports) |
| C | `nmap -sV localhost` | Service and version detection |
| D | `sudo nmap -O localhost` | Operating system fingerprinting |

---

### Open Ports

**None detected.** All 1000 TCP ports on localhost (127.0.0.1) returned as closed across all scan types.

---

### Running Services

**None detected.** No listening services were identified on the target system during the scanning window.

---

### Operating System

OS detection was performed (`sudo nmap -O localhost`) but returned an inconclusive result:

> `Too many fingerprints match this host to give specific OS details`

This is a known limitation of Nmap OS fingerprinting when scanning a loopback interface with no open ports — the engine cannot gather sufficient varied TCP/IP stack responses to confidently distinguish between OS fingerprints. The system is confirmed to be running **Kali Linux** (verified from the terminal prompt and apt package manager output during Nmap installation).

---

### Observations

- The Kali Linux VM is in a **minimal, hardened state** with no network-facing services running.
- Scanning `localhost` (127.0.0.1) is done over the loopback interface, which means results reflect services running on the machine itself, not those exposed on LAN-facing interfaces.
- Nmap's OS fingerprinting requires at least one open port to generate a meaningful OS guess; with all ports closed, this feature could not return a specific result.
- All scan durations were very short (0.12s to 1.69s), which is consistent with scanning a local loopback interface with no open ports.

---

### Recommendations

| Priority | Recommendation |
|---|---|
| 🔴 High | Never enable Telnet (port 23). Use SSH for all remote access. |
| 🔴 High | If SSH is enabled in future, configure key-based authentication and disable password login. |
| 🟡 Medium | If a web server (Apache/Nginx) is ever started, ensure it is stopped when not in use and never left running on a public interface. |
| 🟡 Medium | Periodically run `nmap localhost` to audit your own system for unexpected open ports. |
| 🟢 Low | Disable or remove services that are installed but not actively used to reduce the potential attack surface. |

---

### Conclusion

This task provided hands-on experience with Nmap — one of the most widely used tools in a penetration tester's toolkit. Through the scanning and enumeration phase, I learned how to identify active hosts, probe for open ports, detect running service versions, and attempt operating system fingerprinting using a structured, methodical approach.

In this particular exercise, the Kali Linux VM presented a clean result: no open ports and no running services. While this may initially seem like an uninteresting outcome, it actually demonstrates an important security principle — a minimal attack surface is a secure attack surface. Real-world targets are rarely this clean, and understanding what a "safe baseline" looks like helps in recognizing anomalies during actual engagements.

OS detection highlighted a key limitation of automated fingerprinting tools: they depend on observable network behavior, and when a host is quiet (no open ports), the tool cannot gather enough data to make a definitive identification. This reinforces the idea that no single tool gives a complete picture — effective reconnaissance always combines multiple techniques.

Overall, this task built a strong foundation in the scanning phase of ethical hacking and reinforced the importance of performing these activities only on authorized systems within a controlled environment.

---

## Screenshots

| File | Description |
|---|---|
| `Screenshots/Version check.png` | Nmap installation and version verification |
| `Screenshots/Basic scan.png` | Basic scan — `nmap localhost` |
| `Screenshots/Version detection.png` | Version detection — `nmap -sV localhost` |
| `Screenshots/OS detection.png` | OS detection — `sudo nmap -O localhost` |

---

*Submitted as part of the White Band Associates IT Internship Program — Ethical Hacking Track.*
