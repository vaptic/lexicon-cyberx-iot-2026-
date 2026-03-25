
# Day 4 — Firmware Extraction with Binwalk 🔍
**Phase 2: Firmware Analysis 

##  Theory — DVRF Structure & Binwalk Flags

### What is DVRF?
**Damn Vulnerable Router Firmware** — built by Praetorian Inc. as a learning tool. It mimics a real Linksys router but has intentional flaws baked in:
- **Stack buffer overflows** — program writes more data than a buffer can hold, corrupts memory
- **Format string bugs** — attacker controls format specifiers like `%x %s` to read/write memory
- **Hardcoded credentials** — passwords literally written into the firmware code

### Binwalk Flags

| Flag | What it does | Why it matters |
|------|-------------|----------------|
| `-A` | Scans for CPU architecture | Tells you MIPS/ARM/x86 — needed for emulation later |
| `-e` | Extracts files | Unpacks firmware contents |
| `-M` | Recursive/matryoshka extraction | Firmware has archives inside archives — this digs through all of them |
| `-W` | Hex compare multiple files | Spot differences between two firmware versions |
| `-h` | Help/usage | Lists all available flags |

>  Always run `man binwalk` to read the full manual. Navigate with `Space` (down), `b` (up), `q` (quit), `/keyword` (search).
---

##  Overview

Today we cracked open a real (intentionally vulnerable) router firmware — **DVRF v03** (Damn Vulnerable Router Firmware) by Praetorian Inc. The goal was to extract it, map its filesystem, and start identifying where the vulnerabilities live.

Spoiler: it wasn't as straightforward as the lab expected. And that's kind of the point.

---

##  Environment Setup

- **Tool:** Binwalk on Kali Linux (VirtualBox VM)
- **Target:** `DVRF_v03.bin`
- **Source:** [https://github.com/praetorian-inc/DVRF](https://github.com/praetorian-inc/DVRF)

>  **Note:** DVRF is now archived on GitHub — no longer actively maintained. This explains some of the missing components we'll talk about later.

---

##  Step-by-Step Findings

### Step 1 — Navigate to DVRF Directory
```bash
cd ~/Downloads/DVRF/Firmware
```
Simple enough — but we had to clone the repo first since the folder didn't exist:
```bash
git clone https://github.com/praetorian-inc/DVRF.git
```
The clone threw some errors about spaces in filenames (`Source Code/`) but the important stuff — the `.bin` file — came through fine.

---

### Step 2 — Architecture Scan
```bash
binwalk -A DVRF_v03.bin
```

**Expected:** MIPS  
**Actual finding:** ARM 👀

| Decimal | Hexadecimal | Description |
|---------|-------------|-------------|
| 2089227 | 0x1FE00B | ARMEB instructions, function prologue |
| 3948573 | 0x3C401D | ARM instructions, function prologue |

> This was the first surprise of the day. The lab said MIPS but the firmware is actually ARM (both big and little endian). This matters for later when emulating or running the binaries — you'd need the right QEMU target.

---

### Step 3 — Full Recursive Extraction
```bash
binwalk -eM DVRF_v03.bin
```

The `-M` flag is crucial here — firmware often has compressed archives **inside** archives (like Russian dolls). Binwalk recurses into each one.

>  **Real talk:** This step failed twice because the VM ran out of disk space (only had 19GB allocated). Had to resize the VM disk from 20GB → 50GB using VBoxManage + GParted Live ISO before this worked. Always allocate enough disk space before starting firmware analysis!

The extraction produced a lot of symlink warnings — these are normal. Binwalk redirects dangerous symlinks (ones pointing outside the extraction directory) to `/dev/null` for security.

---

### Step 4 — List Extracted Contents
```bash
ls -R ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/ | head -50
```

Confirmed extraction worked. Found the main extracted file and the `squashfs-root` directory.

---

### Step 5 — Found squashfs-root 🎯
```bash
ls ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/
```

**Output:**
```
bin  dev  etc  lib  mnt  proc  pwnable  sbin  tmp  usr  var  www
```

This is the **actual router filesystem** — everything the router runs is right here.

---

### Step 6 — Map the Entire Filesystem
```bash
find ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/ -type f > filesystem-tree.txt
```

Also ran a visual tree:
```bash
tree ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/
```

**Result: 56 directories, 391 files**

That's the entire brain of this router, sitting right on disk.

---

### Step 7 — Count Files by Type
```bash
grep -E '\.sh$' filesystem-tree.txt | wc -l
grep -E '\.conf$' filesystem-tree.txt | wc -l
```

| File Type | Count |
|-----------|-------|
| Shell scripts (`.sh`) | 2 |
| Config files (`.conf`) | 3 |

> Minimal numbers — expected for an intentionally stripped-down firmware.

---

### Step 8 — Web Interface Analysis
```bash
ls ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/www/
```

Only one file: `index.asp` — no `.cgi` files visible at first glance.

But looking **inside** `index.asp`:
```bash
cat ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/www/index.asp
```

Found a hidden gem:
```html
action="upgrade.cgi"
```

>  **Finding:** `upgrade.cgi` is a firmware upload script referenced inside `index.asp`. This is a classic high-value attack target — firmware upload endpoints are notorious for command injection and buffer overflows. Real-world router exploits often start exactly here.

---

### Step 9 — Startup Scripts (init.d)
```bash
ls ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/etc/init.d/
```

**Result:** No `init.d` directory found in the firmware.

> **Observation:** DVRF is a minimal archived firmware — it was never meant to be a full router OS. It doesn't have traditional init scripts because it's stripped down to just demonstrate vulnerabilities. Real router firmware would have many init.d scripts launching services like telnet daemons, web servers, etc.

---

### Step 10 — The Pwnable Directory 💀
Since there's no init.d, we explored what makes DVRF unique — the `/pwnable` directory:
```bash
ls ~/Downloads/DVRF/Firmware/_DVRF_v03.bin.extracted/squashfs-root/pwnable/
```

This is where the intentionally vulnerable binaries live — stack buffer overflows, format string bugs, heap overflows. This is what Days 5-7 will focus on.

---

[filesystem-tree.txt](https://github.com/user-attachments/files/26238800/filesystem-tree.txt)

## AI Task — Filesystem Vulnerability Analysis

**Prompt used:**
> "Based on this IoT router filesystem, which directories and file types are most likely to contain security vulnerabilities? Explain each."

### Key Findings from Analysis:

| Attack Surface | Location | Risk |
|---------------|----------|------|
| Web Interface | `/www` | Command injection, auth bypass |
| Network Services | `/usr/sbin` | Buffer overflows, RCE |
| Shared Libraries | `/lib`, `/usr/lib` | Memory corruption |
| Auth Utilities | `smbd`, `smbpasswd` | Hardcoded credentials |
| Kernel Modules | `/lib/modules` | Full system compromise |
| Shell Scripts | `*.sh` files | Command injection |
| Debug Binaries | `/pwnable` | Intentional overflows |
| Config Files | `/etc` | Info disclosure |

### Likely Attack Path:
```
Web interface (HTTP)
    → Command injection / input validation flaw
        → Shell on device
            → NVRAM / service persistence
                → Kernel privilege escalation
```

### Conclusion from AI:
The most critical entry point is the **web interface**, followed by **network services** and **shared libraries**. The firmware runs an outdated Linux 2.6.x kernel which increases exposure to known CVEs.

### Future Work Identified:
- Static analysis on `httpd` and `libnvram.so`
- Emulate firmware using QEMU for dynamic testing
- Fuzz `dnsmasq` and `pppd`
- Map known CVEs to included components

---

##  Summary of Findings

| # | Finding | Significance |
|---|---------|-------------|
| 1 | Architecture is ARM, not MIPS | Changes emulation approach |
| 2 | 56 dirs, 391 files extracted | Full filesystem accessible |
| 3 | `upgrade.cgi` referenced in `index.asp` | High-value attack target |
| 4 | No `init.d` — minimal firmware | DVRF is stripped by design |
| 5 | `/pwnable` directory present | Intentional vuln playground |
| 6 | Only 2 shell scripts, 3 config files | Minimal attack surface by design |

---

##  Tools Used
- Kali Linux VM (VirtualBox)
- Binwalk
- GParted Live (for VM disk resize)
- tree, find, grep
- ChatGPT (AI filesystem analysis)

---

## 📁 Files in this Commit
- `day4.md` — this file
- `filesystem-tree.txt` — complete file listing of squashfs-root
- Screenshots of key steps

---

*Day 4 complete. Tomorrow we start actually exploiting these binaries. 🔓*
