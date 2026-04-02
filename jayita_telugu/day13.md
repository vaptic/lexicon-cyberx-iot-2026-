markdown# Day 13 — Attack Tree Modelling for a Full IoT Scenario

**Phase:** Phase 4 — Threat Modelling | **Ratio:** 30% Theory / 70% Practical

---

## Mitigation Table

| Attack Path | Likelihood | Impact | CVSS Score | Mitigation |
|---|---|---|---|---|
| Hardcoded credentials → SSH login | High | Critical | 9.1 | Remove all hardcoded credentials from firmware; use encrypted secrets vaults |
| Vulnerable binary → Buffer overflow | Medium | High | 8.8 | Compile with stack canaries, ASLR, and NX bit; audit binaries regularly |
| ARP Spoof → MQTT intercept → Malicious publish | High | High | 8.3 | Enforce TLS 1.2+ on MQTT; enable Dynamic ARP Inspection on the network |
| MQTT wildcard subscribe → False data injection | High | Medium | 7.5 | Implement per-client ACLs; require authentication before any subscription |
| Default credentials → Admin panel takeover | Critical | Critical | 9.8 | Force unique password setup on first boot; disable all default accounts |

---

## Risk Register — All Vulnerabilities Found During Internship

### Critical (CVSS 9–10)

| # | Vulnerability | Day Found | CVSS | Attack Vector | Mitigation |
|---|---|---|---|---|---|
| 1 | Default web interface credentials left unchanged | Day 13 | 9.8 | Network | Force unique password setup on first boot; disable all default accounts |
| 2 | Hardcoded credentials embedded in firmware | Day 5 | 9.1 | Network | Never hardcode secrets; use encrypted vaults and rotate credentials regularly |
| 3 | OTA update with no signature verification | Day 13 (AI) | 9.4 | Network | Cryptographically sign all firmware images; reject unsigned or downgrade packages |
| 4 | Cloud account compromise → all devices exposed | Day 13 (AI) | 9.0 | Network | Enforce MFA on all cloud accounts; implement login anomaly detection |
| 5 | Lateral movement from IoT device into home LAN | Day 13 (AI) | 9.0 | Network | Isolate IoT devices on a dedicated VLAN; enforce strict inter-VLAN firewall rules |
| 6 | Supply chain compromise via malicious dependency | Day 13 (AI) | 9.2 | Network | Maintain a firmware SBOM; verify all dependency checksums before inclusion |

### High (CVSS 7–9)

| # | Vulnerability | Day Found | CVSS | Attack Vector | Mitigation |
|---|---|---|---|---|---|
| 7 | Buffer overflow in vulnerable DVRF binary | Day 6 | 8.8 | Network | Compile with stack canaries, ASLR, and NX bit; perform regular static binary analysis |
| 8 | ARP spoofing enabling MQTT credential intercept | Day 11 | 8.3 | Network | Enforce TLS 1.2+ on all MQTT; enable Dynamic ARP Inspection on switches |
| 9 | Mobile app embeds API keys and MQTT credentials | Day 13 (AI) | 8.1 | Network | Enforce certificate pinning; never embed long-lived keys in app binaries |
| 10 | Web/API vulnerabilities — injection, IDOR, XSS | Day 13 (AI) | 8.6 | Network | Sanitise all inputs server-side; disable GraphQL introspection; enforce strict RBAC |
| 11 | Exposed Telnet, UPnP, and management ports | Day 13 (AI) | 8.0 | Network | Disable all unused services; firewall management interfaces from the internet |
| 12 | MQTT wildcard subscribe → false data injection | Day 10 | 7.5 | Network | Enforce per-client ACLs; require authentication before any subscription |
| 13 | MQTT broker flood → denial of service | Day 13 (AI) | 7.5 | Network | Implement rate limiting and connection throttling on the MQTT broker |

### Medium (CVSS 4–7)

| # | Vulnerability | Day Found | CVSS | Attack Vector | Mitigation |
|---|---|---|---|---|---|
| 14 | Exposed UART/JTAG debug interfaces on hardware | Day 13 (AI) | 6.8 | Physical | Disable debug interfaces in production builds; enable secure boot |
| 15 | Zigbee/BLE insecure pairing → rogue device joins | Day 13 (AI) | 6.5 | Adjacent Network | Enforce authenticated pairing; disable wireless interfaces when not in use |

### Summary

| Severity | Count |
|----------|-------|
| Critical (9–10) | 6 |
| High (7–9) | 7 |
| Medium (4–7) | 2 |
| **Total** | **15** |

---

## AI Tool Task — Missing Attack Paths

**Prompt used:** "Here is my attack tree for an IoT smart home system. Review it and suggest any missing attack paths I may have overlooked."

ChatGPT identified 10 missing branches. Each has been evaluated and added to the diagram where relevant.

| # | Missing Path | Why It Matters | CVSS | Added to Diagram |
|---|---|---|---|---|
| 1 | Cloud/vendor account compromise | Centralised access to all devices at once | 9.0 | Yes |
| 2 | Mobile app reverse engineering | App often embeds credentials and API keys | 8.1 | Yes |
| 3 | OTA firmware update abuse | Malicious firmware persists through reboots | 9.4 | Yes |
| 4 | Physical UART/JTAG debug access | Bypasses all software-level protections | 6.8 | Yes |
| 5 | Zigbee/BLE rogue device attacks | Entry point without any Wi-Fi compromise | 6.5 | Yes |
| 6 | Web/API vulnerabilities | Strong passwords don't stop injection attacks | 8.6 | Yes |
| 7 | Exposed unnecessary services | Telnet and UPnP are still common in IoT | 8.0 | Yes |
| 8 | Supply chain compromise | Attack surface begins before device ships | 9.2 | Yes |
| 9 | Denial of service | Disabling locks/cameras is a serious threat | 7.5 | Yes |
| 10 | Lateral movement after compromise | IoT device used as pivot into home network | 9.0 | Yes |

**Key structural suggestion from ChatGPT:** Add a separate top-level branch called "Cloud / Mobile Ecosystem Attack" since modern smart-home compromise is often easier through the app or cloud account than through direct firmware exploitation. This has been reflected in the updated diagram.
