# Day 6 - Static Binary Analysis with Ghidra

## Tools Used
- Kali Linux VM
- Ghidra 12.0.4
- Java OpenJDK 21
- DVRF firmware binary

## Steps Completed

### 1. Installation
- Installed Java 21: `apt install -y openjdk-21-jdk`
- Downloaded Ghidra 12.0.4 from ghidra-sre.org
- Extracted to `/home/jayita59/ghidra/ghidra_12.0.4_PUBLIC`
- Launched with `./ghidraRun`

### 2. Project Setup
- Created new project: DVRF-Analysis (Non-Shared)
- Imported binary: `stack_bof_01`
- Path: `_DVRF_v03.bin.extracted/squashfs-root/pwnable/Intro/stack_bof_01`
- Ran Auto Analysis with default options

### 3. Vulnerability Found
- Function: `main`
- Vulnerable call: `strcpy((char *)&local_d0, ...)`
- Buffer size: `auStack_ce [198]` — only 198 bytes allocated
- Vulnerability type: Stack-based Buffer Overflow
- CVE category: CWE-121
- Attacker controls: input copied directly into fixed buffer with no bounds check
- Bookmarked as: `VULNERABLE`

## AI Tool Task
## Vulnerability Analysis

The code is vulnerable because it uses `strcpy()` to copy attacker-controlled input into a very small local stack variable:

```c
strcpy((char *)&local_d0, *(char **)(param_2 + 4));

local_d0 is only 2 bytes long, but strcpy() copies the entire input string without checking its size.

Vulnerability Type
Stack-based Buffer Overflow
CWE Category
CWE-121: Stack-based Buffer Overflow
CWE-120: Buffer Copy without Checking Size of Input
Possible related CWE: CWE-787 (Out-of-bounds Write)
Exploitation

An attacker can provide a long input string to overflow local_d0 and overwrite nearby stack memory, including:

Other local variables
Saved frame pointer
Return address

This can lead to:

Program crash (Denial of Service)
Arbitrary code execution
Return-Oriented Programming (ROP)
Full compromise of the IoT device

Example payload:

./program AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Since many IoT systems lack protections such as ASLR, NX, stack canaries, and PIE, exploitation may be easier.

## Key Learnings
- Ghidra decompiles MIPS binaries to readable C pseudocode
- strcpy() is dangerous because it has no length checking
- Buffer overflows overwrite stack memory beyond allocated buffer
- Static analysis finds vulnerabilities without running the binary
