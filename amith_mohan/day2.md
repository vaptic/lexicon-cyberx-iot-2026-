# Day 2 —
Firmware Analysis: IoTGoat  
**Date:** 2026-03-23  

---

## Checklist
- [x] Run file command on firmware  
- [x] Inspect magic bytes with hexdump  
- [x] Scan firmware with binwalk  
- [x] Extract firmware with binwalk -eM  
- [x] Navigate extracted filesystem  
- [x] Explore /etc directory  
- [x] Document findings  

---------------------------------------------------

## Commands Used

```bash
file IoTGoat-raspberry-pi2.img
hexdump -C IoTGoat-raspberry-pi2.img | head -20
binwalk IoTGoat-raspberry-pi2.img
binwalk -eM IoTGoat-raspberry-pi2.img
cd _IoTGoat-raspberry-pi2.img.extracted/squashfs-root
ls -la
cat etc/passwd
cat etc/shadow
cat etc/banner
cat etc/device_info

---------------------------------------------------

Here’s your updated **Markdown (.md) file** with added sections for **hexdump, binwalk, and magic bytes**:

````md id="fw8k2p"
# Day 2 — Firmware Analysis: IoTGoat  
**Date:** 2026-03-23  

---

## Checklist
- [x] Run file command on firmware  
- [x] Inspect magic bytes with hexdump  
- [x] Scan firmware with binwalk  
- [x] Extract firmware with binwalk -eM  
- [x] Navigate extracted filesystem  
- [x] Explore /etc directory  
- [x] Document findings  

---

## Commands Used

```bash
file IoTGoat-raspberry-pi2.img
hexdump -C IoTGoat-raspberry-pi2.img | head -20
binwalk IoTGoat-raspberry-pi2.img
binwalk -eM IoTGoat-raspberry-pi2.img
cd _IoTGoat-raspberry-pi2.img.extracted/squashfs-root
ls -la
cat etc/passwd
cat etc/shadow
cat etc/banner
cat etc/device_info
````

---

## Key Findings

1. Firmware is **OpenWRT-based** — OWRT signature found at offset `0x1b0`
2. **SquashFS filesystem** extracted successfully at offset `0x1C00000`
3. **4 hardcoded X.509 certificates** found embedded in firmware
4. Two user accounts found — `root` and `iotgoatuser`
5. Both passwords hashed with weak **MD5crypt ($1$)** — crackable
6. No firewall rules configured — running on defaults
7. Banner reveals device identity on SSH login
8. `DEVICE_REVISION='v0'` — never updated

---

## OWASP IoT Top 10 Violations Found

* **I1 — Weak Credentials**
  MD5crypt hashes in `/etc/shadow`

* **I5 — Outdated Components**
  `DEVICE_REVISION='v0'`

* **I7 — Insecure Storage**
  Hardcoded certificates in firmware

* **I9 — Insecure Defaults**
  No firewall rules, default configuration

---

## Tools Used

* `file`
* `hexdump`
* `binwalk`
* `john`
* `cat`

---

## Firmware Analysis Concepts

### 1. Magic Bytes

Magic bytes are **specific byte sequences at the beginning of a file** used to identify its type.
They act like a signature that helps tools determine the format of a file.

**Example:**

* `0x7F 45 4C 46` → ELF executable
* `hsqs` → SquashFS filesystem

**Why Important:**

* Helps identify unknown firmware formats
* Detects embedded files within firmware
* Used by tools like `file` and `binwalk`

---

### 2. Hexdump

`hexdump` is a tool used to display binary data in **hexadecimal and ASCII format**.

**Command Used:**

```bash
hexdump -C IoTGoat-raspberry-pi2.img | head -20
```

**What it Shows:**

* Raw bytes of the firmware
* ASCII representation alongside hex
* Helps locate magic bytes and headers

**Why Important:**

* Manual inspection of firmware structure
* Identify hidden data or anomalies
* Useful for reverse engineering

---

### 3. Binwalk

`binwalk` is a firmware analysis tool used to **scan and extract embedded files** from binary images.

**Commands Used:**

```bash
binwalk IoTGoat-raspberry-pi2.img
binwalk -eM IoTGoat-raspberry-pi2.img
```

**Functions:**

* Detects file signatures using magic bytes
* Identifies compressed or embedded filesystems
* Extracts contents automatically

---

**Why Important:**

* Quickly reveals firmware structure
* Automates extraction of filesystems like SquashFS
* Essential for IoT security analysis

---

## AI Tool Task

### What is SquashFS?

SquashFS is a **compressed, read-only filesystem** for Linux. It uses compression algorithms like **gzip, LZMA, or XZ** to reduce file size, making it ideal for embedded systems.

---

### Why it’s Used in IoT / Embedded Firmware

* Minimizes firmware size
* Faster boot times
* Easy distribution as a single compressed image
* Read-only nature prevents accidental corruption

---

### Security Implications

* Read-only filesystem requires **full firmware updates** for patching
* Tools like `unsquashfs` may have vulnerabilities (e.g., buffer overflows)
* Compression can hide malicious payloads

---

### Mitigations

* Keep SquashFS tools updated
* Validate firmware sources before installation
* Use sandboxing when analyzing unknown firmware
* Vendors should implement **cryptographic signing** for firmware integrity

---
