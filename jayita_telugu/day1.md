# Day 1 — IoT Security Lab Setup

**Date:** 2026-03-20

---

## Tasks

- [x] Download and install VirtualBox
- [x] Import Kali VM and boot it
- [x] Study OWASP IoT Top 10 (theory)
- [x] Verify all tools in Kali
- [x] Download IoTGoat + DVRF firmware
- [x] AI task — Gemini summary of OWASP Top 10
- [x] Write day1.md and push to GitHub
- [ ] Explore Kali Linux menus — identify tools you'll use

---

## Notes

- Kali VM imported and booted successfully via VirtualBox
- Studied OWASP IoT Top 10 — covered all 10 risks with examples
- Verified core tools: nmap, wireshark, binwalk
- Downloaded IoTGoat and DVRF firmware for upcoming labs
- AI summary of OWASP Top 10 completed using Gemini

  ---

  OWASP IoT Top 10
Specific to Internet of Things devices — routers, smart devices, industrial sensors, wearables, etc.
I1 — Weak, Guessable, or Hardcoded Passwords
Default or hardcoded credentials that users never change. Mirai botnet exploited exactly this. Devices should force password changes on setup.
I2 — Insecure Network Services
Unnecessary or vulnerable network services running on the device — open ports, unneeded protocols (Telnet, FTP), no authentication on services.
I3 — Insecure Ecosystem Interfaces
Vulnerabilities in web UI, backend APIs, mobile apps, or cloud interfaces that interact with the device. Weak API auth, no input validation.
I4 — Lack of Secure Update Mechanism
No firmware signing, updates over unencrypted channels, no rollback protection. Allows attackers to push malicious firmware.
I5 — Use of Insecure or Outdated Components
Deprecated OS/libraries, third-party software with known CVEs baked into firmware — hard to patch once shipped.
I6 — Insufficient Privacy Protection
Unnecessary collection of personal data, insecure storage of PII on device, sharing data without consent or transparency.
I7 — Insecure Data Transfer and Storage
Sensitive data transmitted without encryption (plain HTTP, unencrypted MQTT), or stored in plaintext on the device's flash memory.
I8 — Lack of Device Management
No asset inventory, no remote management capability, no way to decommission or wipe devices. Orphaned devices become attack vectors.
I9 — Insecure Default Settings
Devices shipped with insecure defaults — open ports, debug interfaces exposed, no authentication enabled by default.
I10 — Lack of Physical Hardening
No protection against physical attacks — exposed UART/JTAG debug ports, removable storage, easy firmware extraction. Physical access = full compromise.
