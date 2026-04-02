# Day 13 — Attack Tree Modelling for a Full IoT Scenario

---

## Attack Tree Summary

The attack tree models all known paths an attacker could use to achieve **full compromise of a smart home IoT system**. Three main branches were identified:

1. **Firmware Exploitation** — extracting and reverse-engineering device firmware
2. **Network Attack** — intercepting and injecting traffic over the local network
3. **Physical / Default Credentials** — abusing factory defaults and the admin web interface

---

## Mitigation Table

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| Extract Firmware | Medium | High | 6.8 | Encrypt firmware storage; disable JTAG/UART debug interfaces on production devices |
| Find Hardcoded Credentials (Day 5) | High | Critical | 9.1 | Never hardcode credentials; use environment variables or a secrets vault (e.g. HashiCorp Vault) |
| Login via SSH (using hardcoded creds) | High | Critical | 9.3 | Disable SSH password auth; enforce key-based authentication only; rotate all default credentials |
| Find Vulnerable Binary in Ghidra (Day 6) | Medium | High | 7.2 | Perform regular firmware audits and static analysis in CI/CD pipeline before release |
| Trigger Buffer Overflow via Emulated Service | Medium | High | 8.1 | Apply input length validation; compile binaries with stack canaries, ASLR, and NX-bit enabled |
| ARP Spoof (Day 11) | High | High | 7.4 | Enable Dynamic ARP Inspection (DAI) on switches; use static ARP entries for critical devices |
| Intercept MQTT Credentials | High | High | 8.6 | Enable TLS/SSL on MQTT broker (port 8883); never transmit credentials in plaintext |
| Publish Malicious MQTT Commands | High | Critical | 9.0 | Implement MQTT username/password auth with ACLs; whitelist allowed publish topics per client |
| MQTT Wildcard Subscribe (Day 10) | Medium | Medium | 6.5 | Disable wildcard subscriptions for unauthorised clients; enforce per-topic ACL rules in Mosquitto |
| Map All Device Topics | Medium | Medium | 5.3 | Avoid predictable topic names; implement topic-level authentication and access logging |
| Inject False MQTT Data | Medium | High | 8.2 | Implement message signing and integrity checks; validate all incoming payloads before acting on them |
| Default Web Interface Login | High | High | 8.8 | Force credential change on first boot; block default credentials at firmware level |
| Access Admin Panel | High | High | 8.5 | Restrict admin panel access to local network only; implement multi-factor authentication |
| Change Device Settings | Medium | High | 7.7 | Log all admin actions with timestamps; alert device owner on any configuration change |

---

## AI Tool Task — ChatGPT Review

### Prompt Used

> "Here is my attack tree for an IoT smart home system: [node list pasted]. Review it and suggest any missing attack paths I may have overlooked."

### ChatGPT Feedback Summary

ChatGPT rated the original tree **7/10** and identified the following gaps:

---

### Missing Attack Paths Identified by ChatGPT

#### 1. Web Application Vulnerabilities (High Priority)

The original tree only modelled login → admin panel → settings. ChatGPT highlighted that the web UI (`index.asp` found in firmware) is often the easiest entry point and should include:

- SQL Injection
- Command Injection
- Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)

**Added under:** `Default Web Interface Login → Exploit Web Vulnerability`

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| SQL Injection via web UI | High | Critical | 9.8 | Use parameterised queries; never build queries from raw user input |
| Command Injection via web UI | High | Critical | 9.8 | Sanitise all user input; use allowlists for accepted characters |
| XSS via web UI | Medium | High | 7.5 | Implement Content Security Policy (CSP); escape all output |
| CSRF via web UI | Medium | High | 7.3 | Use CSRF tokens on all state-changing requests |

---

#### 2. Wi-Fi Attacks (High Priority)

Smart home devices are on the local Wi-Fi network, making wireless attacks a primary initial access vector not covered in the original tree.

- Weak Wi-Fi password / WPA2 brute force
- WPS brute force
- Evil Twin / rogue AP attack

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| WPA2 brute force | Medium | Critical | 8.8 | Use WPA3 or strong WPA2 passphrase (20+ chars); disable WPS |
| WPS brute force | High | Critical | 9.0 | Disable WPS entirely on the router |
| Evil Twin AP attack | Medium | High | 8.1 | Use 802.1X authentication; educate users on rogue network warnings |

---

#### 3. Open Ports and Service Exploitation (Medium Priority)

The original tree jumped directly to attacks without a recon phase. Real attackers begin with discovery.

- Port scanning (Nmap)
- Identifying open services (Telnet, HTTP, SSH)
- Exploiting an identified vulnerable service

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| Port scan reveals open Telnet | High | Critical | 9.8 | Disable Telnet; use SSH only with key auth |
| Exposed HTTP service exploited | High | High | 8.5 | Firewall all non-essential ports; apply web server patches |

---

#### 4. Credential Attacks (Medium Priority)

Expanding on the existing credentials branch:

- Brute force login on web or SSH interface
- Credential reuse from other breached services
- Password sniffing via captured MQTT/ARP traffic

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| Brute force SSH/web login | High | High | 8.1 | Implement account lockout and rate limiting; use fail2ban |
| Credential reuse attack | Medium | High | 7.5 | Enforce unique passwords per device; use a password manager |
| Password sniffing via traffic capture | High | High | 8.6 | Encrypt all communications (TLS); avoid plaintext protocols |

---

#### 5. Firmware Update Exploitation (High Priority)

The original tree covered firmware extraction but not abuse of the update mechanism itself.

- Upload of malicious firmware via upgrade.cgi
- No cryptographic signature verification on firmware
- Rollback attack to a known-vulnerable firmware version

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| Malicious firmware upload | Medium | Critical | 9.3 | Enforce firmware signature verification; reject unsigned images |
| No signature verification | High | Critical | 9.5 | Implement RSA/ECDSA signing for all firmware packages |
| Firmware rollback attack | Medium | High | 7.8 | Implement secure boot with rollback prevention counter |

---

#### 6. Physical Access Attacks (Medium Priority)

Expanding the physical branch to include hardware-level access:

- UART console access (serial debug port)
- JTAG debugging interface
- Direct NAND/flash chip dumping

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| UART console access | Medium | Critical | 9.0 | Disable UART in production firmware; require authentication on serial console |
| JTAG debugging access | Low | Critical | 8.5 | Blow JTAG fuses before shipping; use tamper-evident enclosures |
| Flash chip dumping | Low | Critical | 8.8 | Encrypt flash storage with device-unique keys |

---

#### 7. Cloud and Mobile App Attack Surface (Bonus — Advanced)

Smart home devices often connect to cloud backends and mobile apps, which were not in scope for the internship but are real attack surfaces.

- API key theft from mobile app (reverse engineering APK)
- Insecure cloud API endpoints
- OAuth token theft

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| API key extracted from APK | Medium | High | 8.3 | Store API keys server-side; use certificate pinning in the app |
| Insecure cloud API | Medium | Critical | 9.1 | Enforce authentication on all API endpoints; use rate limiting |
| OAuth token theft | Medium | High | 7.8 | Use short-lived tokens; implement token rotation |

---

#### 8. Lateral Movement (Advanced)

After initial compromise, an attacker may pivot to other devices on the same network.

- Scan internal network for other IoT devices
- Reuse credentials across devices
- Use compromised device as a proxy

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| Pivot to other IoT devices | Medium | Critical | 9.2 | Segment IoT devices onto a separate VLAN; apply strict firewall rules between segments |
| Credential reuse across devices | High | High | 8.5 | Enforce unique credentials per device at manufacturing |

---

## Updated Attack Tree Structure (Post-AI Review)

The following branches were added or expanded after ChatGPT review:

```
GOAL: Full Compromise of Smart Home IoT System
│
├── Firmware Exploitation
│   ├── Extract Firmware
│   │   ├── Find Hardcoded Credentials → Login via SSH
│   │   └── Find Vulnerable Binary (Ghidra) → Trigger Buffer Overflow
│   └── [NEW] Firmware Update Abuse
│       ├── Upload Malicious Firmware (upgrade.cgi)
│       ├── No Signature Verification
│       └── Firmware Rollback Attack
│
├── Network Attack
│   ├── ARP Spoof → Intercept MQTT Credentials → Publish Malicious Commands
│   ├── MQTT Wildcard Subscribe → Map Topics → Inject False Data
│   └── [NEW] Wi-Fi Attacks
│       ├── WPA2 Brute Force
│       ├── WPS Brute Force
│       └── Evil Twin AP
│
├── Physical / Default Credentials
│   ├── Default Web Interface Login
│   │   ├── Access Admin Panel → Change Settings
│   │   └── [NEW] Exploit Web Vulnerability
│   │       ├── SQL Injection
│   │       ├── Command Injection
│   │       ├── XSS
│   │       └── CSRF
│   └── [NEW] Hardware Debug Interfaces
│       ├── UART Console Access
│       ├── JTAG Debugging
│       └── Flash Chip Dumping
│
├── [NEW] Recon / Initial Access
│   ├── Port Scan (Nmap)
│   ├── Identify Open Services
│   └── Exploit Vulnerable Service (Telnet / HTTP)
│
└── [NEW] Post-Compromise
    ├── Lateral Movement to Other IoT Devices
    ├── Cloud / API Exploitation
    └── Mobile App Reverse Engineering
```

