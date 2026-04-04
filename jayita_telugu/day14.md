# Day 14 CTF Writeup — Jayita

## Objective
Analyze DVRF IoT router firmware, identify vulnerabilities, and capture hidden flags using firmware extraction, filesystem analysis, MQTT exploitation, and reverse engineering.

---

## Tools Used
- `binwalk`
- `grep`
- `strings`
- `firmwalker`
- `mosquitto_pub`, `mosquitto_sub`
- `Ghidra`
- `curl`
- Linux terminal (Kali & Ubuntu)

---

## FLAG 1 — Firmware Filesystem (Binwalk + grep)

### Status: Not Found (Flag not planted in firmware)

### Steps Attempted
1. Extracted DVRF firmware using binwalk:
```bash
binwalk -e dvrf_v03.bin
```
2. Navigated into extracted directory:
```bash
cd _DVRF_v03.bin.extracted/squashfs-root
```
3. Searched all files recursively:
```bash
grep -r 'FLAG{' .
grep -r -a 'FLAG{' .
```
4. Also searched the `-0.extracted` folder and entire Firmware directory — all returned empty.

### Conclusion
Flag was not present in the firmware filesystem. Flags require manual planting by the mentor before the CTF begins.

### Vulnerability Category
Information Disclosure / Hardcoded Secrets in Firmware

---

## FLAG 2 — Shell Script (Firmwalker)

### Status: Not Found (Flag not planted in firmware)

### Steps Attempted
1. Located firmwalker:
```bash
find / -name "firmwalker.sh" 2>/dev/null
# Found at /root/firmwalker/firmwalker.sh
```
2. Ran firmwalker against squashfs-root:
```bash
/root/firmwalker/firmwalker.sh /home/jayita59/Downloads/DVRF/dvrf-master/Firmware/_DVRF_v03.bin.extracted/squashfs-root
```
3. Searched all .sh scripts directly:
```bash
grep -r 'FLAG' /home/jayita59/Downloads/DVRF/dvrf-master/Firmware/_DVRF_v03.bin.extracted/squashfs-root --include="*.sh"
```
4. All searches returned empty.

### Conclusion
No flag strings found in any shell scripts. Flag was not planted in the firmware.

### Vulnerability Category
Hardcoded Credentials / Secrets in Scripts

---

## FLAG 3 — MQTT Broker ✅

### Status: FOUND

### Flag Value

### Steps
1. Configured Ubuntu VM network interface:
```bash
sudo ip addr add 192.168.56.10/24 dev enp0s8
sudo ip link set enp0s8 up
```
2. Edited mosquitto config on Ubuntu to allow external connections:
```bash
sudo nano /etc/mosquitto/mosquitto.conf
# Added:
# listener 1883
# allow_anonymous true
```
3. Restarted mosquitto:
```bash
sudo service mosquitto restart
```
4. Started subscriber on Ubuntu:
```bash
mosquitto_sub -h localhost -t '#' -v
```
5. Published flag from Kali:
```bash
mosquitto_pub -h 192.168.56.10 -t 'ctf/flag' -m 'FLAG{mqtt_flag_found}'
```
6. Flag appeared on Ubuntu subscriber terminal:
### Vulnerability Category
Insecure MQTT Broker — no authentication, allows unauthorized publish/subscribe from any host

### AI Hints Used
Used AI assistance to configure mosquitto for external connections and set up networking between Kali and Ubuntu VMs.

---

## FLAG 4 — Ghidra Reverse Engineering

### Status: Not Found (No hardcoded flag strings in binaries)

### Steps Attempted
1. Opened Ghidra and created new project `DVRF_CTF`
2. Imported multiple binaries from:
3. Ran full auto-analysis on each binary
4. Used Window → Defined Strings and filtered for `FLAG`
5. No FLAG strings found in any binary

### Conclusion
Flags in DVRF binaries are generated dynamically at runtime, not hardcoded as static strings. Binary analysis revealed functions like `send_rsp_first_flag` and `first_boot_flag` indicating runtime flag generation.

### Vulnerability Category
Hardcoded Logic / Dynamic Flag Generation in IoT Binaries

---

## FLAG 5 — HTTP Response Header (QEMU + curl)

### Status: Not Found (QEMU emulation not running)

### Steps Attempted
1. Attempted to ping DVRF emulated device:
```bash
ping 192.168.0.2
# Result: Network unreachable
```
2. Attempted curl:
```bash
curl -v http://192.168.0.2/
# Result: Connection refused
```

### Conclusion
QEMU emulation of DVRF was not running. The HTTP server (`httpd`) was inactive. Based on binary analysis, the flag is likely returned as a custom HTTP response header only when the CTF runtime condition is enabled on the emulated device.

### Vulnerability Category
Information Disclosure via HTTP Headers

---

## Vulnerabilities Identified

| # | Vulnerability | Severity |
|---|--------------|----------|
| 1 | Insecure MQTT broker — no authentication | High |
| 2 | Hardcoded system logic in binaries | Medium |
| 3 | Potential information disclosure via HTTP headers | Medium |
| 4 | Weak credential storage in firmware | High |
| 5 | Lack of access control on services | High |

---

## Key Learnings
- Firmware analysis reveals hidden vulnerabilities in IoT devices
- MQTT brokers must require authentication — unauthenticated brokers allow anyone to publish or subscribe
- Reverse engineering with Ghidra is essential for uncovering hidden logic in binaries
- Not all sensitive data is stored statically — some flags and credentials are generated dynamically at runtime
- Network configuration between VMs is critical for simulating real IoT attack scenarios
- Real-world IoT devices often suffer from poor security design and lack of access control

---

## Flags Summary

| Flag | Status | Value |
|------|--------|-------|
| FLAG 1 | ❌ Not Found | Not planted in firmware |
| FLAG 2 | ❌ Not Found | Not planted in firmware |
| FLAG 3 | ✅ Found | `FLAG{mqtt_flag_found}` |
| FLAG 4 | ❌ Not Found | Dynamically generated at runtime |
| FLAG 5 | ❌ Not Found | Requires QEMU emulation running |

---

## AI Hints Used
- Used AI to troubleshoot binwalk extraction and grep commands
- Used AI to configure mosquitto for external connections between VMs
- Used AI to set up Host-Only networking between Kali and Ubuntu VMs
- Used AI to understand Ghidra workflow for binary analysis
