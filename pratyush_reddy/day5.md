# Day 5 — Hunting Secrets: Credentials, Keys & Hardcoded Data
**Phase:** Phase 2 — Firmware Analysis  
**Date:** 2026-03-26  
**Target:** DVRF v03 (Damn Vulnerable Router Firmware) — Netgear E1550  
**Working Directory:** `~/Downloads/DVRF/_DVRF_v03.bin.extracted/squashfs-root`

---

## 📋 Summary

Today's session focused on credential hunting and sensitive data discovery inside the extracted DVRF firmware filesystem. Since `/etc/passwd` and `/etc/shadow` were empty (no plaintext credentials in the standard locations), the investigation pivoted to binary analysis using `strings` and `grep` across executables, shared libraries, and configuration files. Multiple hardcoded credential references, password-handling functions, and sensitive internal strings were discovered.

---

## 🔍 Findings

---

### Finding 1 — Samba Password Binary (`smbpasswd`) Contains Sensitive Strings

**File:** `usr/local/samba/bin/smbpasswd`  
**Command Used:** `cat usr/local/samba/bin/smbpasswd`  
**Sensitive Data Found:**

- The binary output (images 1–5) revealed readable embedded strings including:
  - Password management options: `-a` (add user), `-d` (disable user), `-e` (enable user), `-n` (set no password), `-w` (set stdin admin password), `-x` (delete user)
  - Hardcoded error strings: `NO PASSWORD`, `NO PASSWORDc:axdehminjr:sw:R:D:U:LW`
  - Internal credential paths: references to `secrets.tdb` (Samba's password database)
  - Error messages: `"Failed to store the ldap admin password!"`, `"Failed to open passdb!"`, `"You must specify a username"`
  - Build path leaked: `/root/workspace/gpl/E3200/E1550/build/cbtn/e1550-26/src/router/shared/storage/storage.c`

**Risk:** The embedded build path exposes the manufacturer's internal development directory structure (Netgear E1550 build environment). Error strings confirm the device uses Samba for SMB/CIFS file sharing, and the password handling routines are present in an unprotected binary readable by anyone with filesystem access.

---

### Finding 2 — `smbadduser` Script: Hardcoded Password Passing Mechanism

**File:** `usr/local/samba/bin/smbadduser`  
**Command Used:** `cat usr/local/samba/bin/smbadduser`  
**Sensitive Data Found:**

```
#!/bin/sh
# script file to add user for samba

if [ $# -lt 2 ]; then echo "Usage:$0 user passwd"; exit 1 ; fi
(sleep 1; echo "$2"; sleep 1; echo "$2") | /usr/local/samba/bin/smbpasswd -a $1 -s
exit 0
```

**Risk:** This shell script accepts a password as a **plaintext command-line argument** (`$2`). On a live system, command-line arguments are visible to all users via `ps aux`. This is a CWE-214 (Information Exposure Through Process Environment) vulnerability. Any user on the device could observe passwords being passed to this script.

---

### Finding 3 — `store_machine_password` Binary: NVRAM Credential Storage

**File:** `sbin/store_machine_password`  
**Command Used:** `strings sbin/store_machine_password | head -30`  
**Sensitive Data Found:**

- References to `nvram_set`, `nvram_commit` — confirms passwords are stored in NVRAM (non-volatile RAM, persists across reboots)
- `libbcmcrypto.so` — Broadcom crypto library linked, suggesting some form of encryption used
- Usage string: `'Usage: %s <domain> <password> <last_change_time> <sec_channel_type>'`

**Risk:** The device stores domain/machine passwords in NVRAM. NVRAM contents are often extractable via UART/JTAG hardware access or through firmware dumps. The usage string confirms this binary writes credentials directly to persistent storage.

---

### Finding 4 — `store_domain_sid` Binary: Domain Credential Storage

**File:** `sbin/store_domain_sid`  
**Command Used:** `strings sbin/store_domain_sid | head -30`  
**Sensitive Data Found:**

- Same NVRAM write functions: `nvram_set`, `nvram_commit`
- Usage: `'Usage: %s <domain name> <sid data>'`
- String: `domain_%s_sid`

**Risk:** Domain Security Identifiers (SIDs) stored in NVRAM. Combined with the machine password, an attacker could perform Pass-the-Hash or SMB relay attacks on the local network.

---

### Finding 5 — `tmpuseradd` Binary: User Account Creation with Hardcoded Temp Password Path

**File:** `sbin/tmpuseradd`  
**Command Used:** `strings sbin/tmpuseradd | grep -i "pass\|admin\|root\|user" | head -20`  
**Sensitive Data Found:**

- `Usage: %s <user name>`
- `/tmp/passwd` — temporary password file written to `/tmp` (world-readable directory)
- `ERROR: user %s already exists!`
- `users_gid`, `tmpuseraddgroup`, `users`

**Risk:** The binary writes to `/tmp/passwd` — a world-readable location. Any process running on the device could read credentials written here. This is a CWE-732 (Incorrect Permission Assignment for Critical Resource) vulnerability.

---

### Finding 6 — Web Server Binary (`httpd`): Extensive Authentication Function Library

**File:** `usr/sbin/httpd`  
**Command Used:** `strings usr/sbin/httpd | grep -i "admin\|password\|passwd\|root\|login\|auth" | head -30`  
**Sensitive Data Found (selection):**

- `set_passwd`, `pool_encryption_password_is_recoverable`, `deallocate_password_data`
- `set_cifs_password_protection_for_share`, `authinfo`, `ssl_set_authmode`
- `do_login_ej`, `validate_wl_auth_0`, `login_warning_checked`
- `do_set_reset_pc_passwd`, `do_auth_pc_passwd`, `do_auth_pc_answer`
- `ej_get_pc_passwd_status`, `show_auth_list`, `basic_auth_fail`
- `hnap_password`, `hnaphost_httpserver_auth_check`, `hnap_host_httpserver_auth_fail`
- `add_webui_authexemption`, `check_parental_controls_password`, `has_parental_controls_password`
- `reset_parental_controls_password`, `isValidPasswd`, `Add_Account_Password`
- `nas_login`, `set password failed!!`, `create_passwd1`

**Risk:** The web server binary contains dozens of authentication bypass-adjacent functions. `pool_encryption_password_is_recoverable` is particularly alarming — it implies that encryption passwords can be recovered (i.e., they are not one-way hashed). The presence of `basic_auth_fail` without corresponding rate-limiting strings suggests no brute-force protection.

---

### Finding 7 — `httpd` Binary: Hardcoded Credential Format Strings and Default Account Patterns

**File:** `usr/sbin/httpd`  
**Command Used:** Extended strings analysis (images 15–19)  
**Sensitive Data Found:**

- `root::` — empty root password format string
- `root:!:0:0:99999:7:::` — shadow file format with locked/empty root password
- `root:x:0:root` — passwd format string
- `%s:2:admin:rw:guest:r` — **hardcoded share permission string with `admin` and `guest` accounts**
- `%s|%s:1:admin` — another admin credential format
- `Router Password` — label string for a router password field
- `2.4G RADIUS Shared Secret` — Wi-Fi RADIUS shared secret label
- `adduser: name=%s, passwd=%s, group=%s` — user creation with password logged as format string
- `Authorization fails (username or passwords)` — auth error message
- `HNAP:ERROR:Remote management password is same with default` — **confirms default password detection logic exists**
- `HNAP:SetDefaultWireless: GuestNetwork Password is error!`
- `HNAP:SetDefaultWireless: Wireless Password is error!`
- `HNAP:wan username or password has error(%s/%s)` — WAN credentials format string
- `GuestPassword`, `AdminPassword`, `AdminUI_Show`
- `/root/workspace/gpl/E3200/E1550/build/cbtn/e1550-26/src/router/shared/storage/storage.c` — **internal build path leaked again**
- `http_passwd`, `http_username` — HTTP authentication variables stored in NVRAM

**Risk:** The string `HNAP:ERROR:Remote management password is same with default` confirms the device has logic to detect default passwords — but this check happens at runtime, not at factory reset. The hardcoded format `%s:2:admin:rw:guest:r` reveals the default share permission model: admin gets read-write, guest gets read. The leaked internal source path (`/root/workspace/gpl/E3200/E1550/`) confirms this is a Netgear E1550/E3200 firmware build, giving an attacker exact model context for targeted CVE research.

---

### Finding 8 — `libnvram.so`: HTTP Password and Username NVRAM Keys

**File:** `usr/lib/libnvram.so`  
**Command Used:** `strings usr/lib/libnvram.so | grep -i "pass\|admin\|http_password\|http_username" | head -20`  
**Sensitive Data Found:**

- `passphrase_24g`, `wl0_passphrase`, `passphrase_5g` — Wi-Fi passphrase NVRAM keys for 2.4GHz and 5GHz bands
- `wl1_passphrase` — second Wi-Fi band passphrase key
- `ppp_passwd`, `pppoe_passwd`, `ppp_passwd_1`, `pppoe_passwd_1` — PPP/PPPoE WAN password NVRAM keys

**Risk:** All Wi-Fi passphrases and WAN authentication credentials (PPPoE) are stored as plaintext NVRAM variables with well-known key names. Any process with `nvram_get` access can retrieve `ppp_passwd` or `wl0_passphrase` without any authentication. This directly maps to **OWASP IoT Top 10 #7 — Insecure Data Transfer and Storage**.

---

### Finding 9 — `etc/Services.txt`: Bonjour/mDNS Service Discovery Entries

**File:** `etc/Services.txt`  
**Command Used:** `cat etc/Services.txt`  
**Sensitive Data Found:**

- `Tweedlebug`, `Tweedlebug2`, `Tweedlebug3` — mDNS/Bonjour service names registered via `_afpovertcp._tcp` (Apple Filing Protocol over TCP)
- Port: `548`

**Risk:** The device advertises AFP (Apple Filing Protocol) services on the local network via mDNS. The service names `Tweedlebug`, `Tweedlebug2`, `Tweedlebug3` are internal identifiers visible to any device on the same network segment. This aids network reconnaissance.

---

### Finding 10 — `pwnable/` Directory: Intentional Exploit Challenges Confirming MIPS32 Architecture

**Files:** `pwnable/Intro/README`, `pwnable/ShellCode_Required/README`  
**Command Used:** `cat pwnable/Intro/README`, `cat pwnable/ShellCode_Required/README`  
**Content Found:**

From `Intro/README`:
> "Congrats! If you're reading this then you either connected to the E1550's UART or you extracted the binary with binwalk. These pwnables are for teaching you how to exploit other CPU architectures — MIPS32 (Little Endian)."
- `stack_bof_01` — Stack Buffer Overflow (no shellcode needed, reach hidden function → execute `/bin/sh`)
- `heap_overflow_01` — Heap overflow to reach hidden function → `/bin/sh`
- `uaf_01` — Use-After-Free vulnerability

From `ShellCode_Required/README`:
> "All Pwnables in this directory require shellcode."
- `socket_bof` — Stack BOF, binds to port **1337**
- `socket_cmd` — Passes user data to `System()` — command injection via network socket
- `stack_bof_02` — BOF without built-in `/bin/sh`; jumps to `sleep(1)` returning `0x41`

**Risk:** This confirms DVRF is intentionally vulnerable. The `socket_cmd` binary passing user input to `System()` is a **critical command injection vulnerability** (CWE-78). `socket_bof` binding to port 1337 is a network-accessible overflow — exploitable remotely without authentication.

---

## 📊 Findings Summary Table

| # | File | Finding | CVE Category | Severity |
|---|------|---------|-------------|----------|
| 1 | `usr/local/samba/bin/smbpasswd` | Internal build path + password DB paths leaked | CWE-200 Information Exposure | Medium |
| 2 | `usr/local/samba/bin/smbadduser` | Password passed as plaintext CLI argument | CWE-214 Process Info Exposure | High |
| 3 | `sbin/store_machine_password` | Domain passwords stored in NVRAM plaintext | CWE-312 Cleartext Storage of Sensitive Info | High |
| 4 | `sbin/store_domain_sid` | Domain SIDs stored in NVRAM | CWE-312 Cleartext Storage | Medium |
| 5 | `sbin/tmpuseradd` | Credentials written to world-readable `/tmp/passwd` | CWE-732 Incorrect Permission Assignment | High |
| 6 | `usr/sbin/httpd` | `pool_encryption_password_is_recoverable` — reversible passwords | CWE-257 Storing Passwords in Recoverable Format | Critical |
| 7 | `usr/sbin/httpd` | Hardcoded admin/guest share format, default password detection, leaked build path | CWE-798 Hardcoded Credentials | Critical |
| 8 | `usr/lib/libnvram.so` | Wi-Fi passphrases + PPPoE passwords stored as plain NVRAM keys | CWE-256 Plaintext Storage of Passwords | Critical |
| 9 | `etc/Services.txt` | AFP service names aid network reconnaissance | CWE-200 Information Exposure | Low |
| 10 | `pwnable/` directory | Command injection via `socket_cmd` → `System()`, network BOF on port 1337 | CWE-78 OS Command Injection, CWE-121 Stack BOF | Critical |

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| `strings` | Extract readable ASCII strings from binaries |
| `grep -i` | Case-insensitive pattern matching for credential keywords |
| `cat` | Read shell scripts and config files |
| `claude AI assistant` | Help with lab |
| Kali Linux VM | Analysis environment |

---

## 📝 Notes

- `/etc/passwd` and `/etc/shadow` were **empty** in this firmware — no traditional Unix password hashes present.
- The firmware was confirmed as **Netgear E1550** (MIPS32 Little Endian) via internal build path strings.
- All password storage appears to use **NVRAM variables** rather than filesystem files — this is common in embedded router firmware.
- The `pwnable/` directory is intentionally included in DVRF as a learning resource for MIPS exploitation.
- No private SSH/TLS keys were found in this session — private key search (`grep -rl 'BEGIN.*PRIVATE KEY'`) returned no results, confirming DVRF does not include real private keys.

---

