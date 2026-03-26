
# Day 5 - Firmware Security Analysis with Firmwalker

## Objective
Use automated and manual techniques to find sensitive data hidden inside DVRF firmware.

## Tool Used
Firmwalker - automated firmware security analysis script

## Firmware Analyzed
DVRF v03 (Damn Vulnerable Router Firmware)
Architecture: MIPS32 Little Endian
Path: ~/Downloads/DVRF-master/Firmware/_DVRF_v03.bin.extracted/squashfs-root/

## Steps Completed

### Step 1-2: Firmwalker Setup
Downloaded and made Firmwalker executable.

### Step 3-4: Firmwalker Scan
Ran Firmwalker against DVRF squashfs-root.
Firmwalker automatically detected sensitive binaries and files.

### Step 5: /etc/passwd
Result: Empty
Reason: DVRF generates passwd dynamically into /tmp/var/ only at boot time.
This is expected behavior for this firmware.

### Step 6: /etc/shadow
Result: Empty
Reason: Same as passwd - runtime generated, not stored statically.

### Step 7: Password strings in .conf/.sh files
Result: None found
Reason: DVRF only contains 2 shell scripts and neither contains plaintext passwords.

### Step 8: Hardcoded IPs
Result: None found in /etc/
Reason: DVRF is minimal firmware, network config is handled at runtime.

### Step 9: Private Keys
Result: CRITICAL FINDING
File: usr/lib/libpolarssl.so
Risk: A private cryptographic key is embedded directly in this library.
An attacker who extracts this firmware can use the key to decrypt
all encrypted communications between the router and its users.

## Deep Dive Findings (Binary Analysis)

### Finding 1: Hardcoded Credentials in Web Server - CRITICAL
File: usr/sbin/httpd
Strings found: new_admin_password, root:x:0:root, set password failed!!
Risk: The web server binary has credentials hardcoded directly into it.
Any attacker with access to the firmware can extract these credentials
and use them to log into the router's web interface.

### Finding 2: WiFi Password Storage Locations - CRITICAL
File: usr/lib/libnvram.so
What was found:
- ssid_24g, ssid_5g → WiFi network names for both bands
- passphrase_24g, passphrase_5g → actual WiFi passwords for both bands
- wl0_wpa_psk, wl1_wpa_psk → WPA pre-shared keys
- wl0_wep, wl1_wep → WEP encryption keys
- wl0_radius_key, wl1_radius_key → enterprise WiFi authentication keys
- ppp_passwd, pppoe_passwd → ISP/broadband passwords
- ppp_username, pppoe_username → ISP usernames

Risk: This library exposes the exact variable names and memory locations
where the router stores every sensitive credential at runtime including
WiFi passwords, WPA keys, WEP keys and ISP passwords. An attacker with
any code execution on the router can read these NVRAM variables and
steal every single password in one shot - WiFi password, ISP password,
WPA keys, everything.

### Finding 3: Samba Password Management - HIGH
File: usr/local/samba/bin/smbpasswd
Risk: Samba (Windows file sharing) is present in the firmware.
Samba is a historically vulnerable service and its presence
increases the attack surface significantly.

### Finding 4: DVRF Intentional Vulnerabilities
File: squashfs-root/pwnable/Intro/README
Architecture: MIPS32 Little Endian
Built-in vulnerabilities:
- stack_bof_01: Stack buffer overflow
- heap_overflow_01: Heap overflow
- uaf_01: Use-after-free vulnerability
Note: These are intentionally planted for security research and learning.

## Summary
DVRF contains multiple critical security issues including an embedded
private key, hardcoded web server credentials, and a comprehensive
exposure of WiFi and ISP password storage locations through the NVRAM
library. An attacker with firmware access can extract the private key
to decrypt communications, read hardcoded web credentials to access
the admin panel, and identify exactly where every password is stored
in memory to steal them at runtime. While DVRF is intentionally
vulnerable for learning purposes, these same issues are commonly found
in real-world router firmware and represent serious risks in production
devices.# Day 5 - Firmware Security Analysis with Firmwalker

## Objective
Use automated and manual techniques to find sensitive data hidden inside DVRF firmware.

## Tool Used
Firmwalker - automated firmware security analysis script

## Firmware Analyzed
DVRF v03 (Damn Vulnerable Router Firmware)
Architecture: MIPS32 Little Endian
Path: ~/Downloads/DVRF-master/Firmware/_DVRF_v03.bin.extracted/squashfs-root/

## Steps Completed

### Step 1-2: Firmwalker Setup
Downloaded and made Firmwalker executable.

### Step 3-4: Firmwalker Scan
Ran Firmwalker against DVRF squashfs-root.
Firmwalker automatically detected sensitive binaries and files.

### Step 5: /etc/passwd
Result: Empty
Reason: DVRF generates passwd dynamically into /tmp/var/ only at boot time.
This is expected behavior for this firmware.

### Step 6: /etc/shadow
Result: Empty
Reason: Same as passwd - runtime generated, not stored statically.

### Step 7: Password strings in .conf/.sh files
Result: None found
Reason: DVRF only contains 2 shell scripts and neither contains plaintext passwords.

### Step 8: Hardcoded IPs
Result: None found in /etc/
Reason: DVRF is minimal firmware, network config is handled at runtime.

### Step 9: Private Keys
Result: CRITICAL FINDING
File: usr/lib/libpolarssl.so
Risk: A private cryptographic key is embedded directly in this library.
An attacker who extracts this firmware can use the key to decrypt
all encrypted communications between the router and its users.

## Deep Dive Findings (Binary Analysis)

### Finding 1: Hardcoded Credentials in Web Server - CRITICAL
File: usr/sbin/httpd
Strings found: new_admin_password, root:x:0:root, set password failed!!
Risk: The web server binary has credentials hardcoded directly into it.
Any attacker with access to the firmware can extract these credentials
and use them to log into the router's web interface.

### Finding 2: WiFi Password Storage Locations - HIGH
File: usr/lib/libnvram.so
Strings found: passphrase_24g, passphrase_5g, wl0_passphrase,
wl1_passphrase, ppp_passwd, pppoe_passwd
Risk: This library reveals the exact variable names and memory locations
where the router stores WiFi passwords. An attacker can target these
specific locations to extract credentials from a running device.

### Finding 3: Samba Password Management - HIGH
File: usr/local/samba/bin/smbpasswd
Risk: Samba (Windows file sharing) is present in the firmware.
Samba is a historically vulnerable service and its presence
increases the attack surface significantly.

### Finding 4: DVRF Intentional Vulnerabilities
File: squashfs-root/pwnable/Intro/README
Architecture: MIPS32 Little Endian
Built-in vulnerabilities:
- stack_bof_01: Stack buffer overflow
- heap_overflow_01: Heap overflow
- uaf_01: Use-after-free vulnerability
Note: These are intentionally planted for security research and learning.

## Summary
DVRF contains multiple critical security issues including an embedded
private key, hardcoded web server credentials, and exposed password
storage locations. While DVRF is intentionally vulnerable for learning
purposes, these same issues are commonly found in real-world router
firmware and represent serious security risks in production devices.

## AI Analysis Summary

Finding 1 - Embedded Private Key (CWE-321)
One extracted key compromises every router of this model worldwide
since the same firmware is shipped to all devices.

Finding 2 - Hardcoded Credentials in httpd (CWE-798)
Attacker can grep the binary in minutes, log into the admin panel
and use the router as a pivot point to attack the entire network.

Finding 3 - NVRAM WiFi Password Exposure (CWE-312)
A single exploit gives an attacker every network credential on the
device in one shot - WiFi, WPA, WEP and ISP passwords all in plaintext.
