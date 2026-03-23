Day 2 — Firmware Analysis: IoTGoat
Date: 2026-03-23

# Checklist
1) Run file command on firmware
2) Inspect magic bytes with hexdump
3) Scan firmware with binwalk
4) Extract firmware with binwalk -eM
5) Navigate extracted filesystem
6) Explore /etc directory
7) Document findings

# Commands Used
1. bashfile IoTGoat-raspberry-pi2.img
2. hexdump -C IoTGoat-raspberry-pi2.img | head -20
3. binwalk IoTGoat-raspberry-pi2.img
4. binwalk -eM IoTGoat-raspberry-pi2.img
5. cd _IoTGoat-raspberry-pi2.img.extracted/squashfs-root
6. ls -la
7. cat etc/passwd
8. cat etc/shadow
9. cat etc/banner
10. cat etc/device_info

# Key Findings

1. Firmware is OpenWRT based — OWRT signature found at offset 0x1b0
2. Squashfs filesystem extracted successfully at offset 0x1C00000
3. 4 hardcoded X.509 certificates found embedded in firmware
4. Two user accounts found — root and iotgoatuser
5. Both passwords hashed with weak MD5crypt ($1$) — crackable
6. No firewall rules configured — running on defaults
7. Banner reveals device identity on SSH login
8. DEVICE_REVISION='v0' — never updated


# IoT Top 10 Violations Found
I1 — Weak credentialsMD5crypt hashes in /etc/shadow
I5 — Outdated componentsDEVICE_REVISION='v0'
I7 — Insecure storageHardcoded certificates in firmware
I9 — Insecure defaultsNo firewall rules, default config throughout

# Tools Used
file | hexdump | binwalk | john | cat


# AI Tool Task
# What is SquashFS?
SquashFS is a compressed, read-only filesystem for Linux. It uses compression algorithms like gzip, LZMA, or XZ to reduce file size — making it ideal for devices with limited storage and memory.

# Why it's used in IoT/embedded firmware:
Minimizes firmware size
Faster boot times
Easy to distribute as a single compressed block
Read-only nature prevents accidental file corruption

# Security implications:
Read-only means you can't patch individual files — a full firmware update is required for any fix, which delays patching
Tools like unsquashfs have had vulnerabilities (buffer overflows) that attackers can exploit with malicious images
Compression can hide payloads, making malware detection harder

# Mitigations:
Keep SquashFS tools updated
Validate firmware sources before installation
Use sandboxing when analyzing unknown images
Vendors should cryptographically sign firmware to prevent tampering

