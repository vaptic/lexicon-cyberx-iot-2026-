# Day 7 - Firmware Emulation with QEMU

## Method Used
Used qemu-mipsel + chroot emulation (DVRF v0.3 only includes .bin file,
no pre-split kernel/rootfs).

## Steps
1. Installed qemu-user and qemu-user-static
2. Extracted squashfs filesystem using binwalk (Day 5)
3. Copied qemu-mipsel into firmware filesystem
4. Mounted proc, sys, dev into squashfs-root
5. Chrooted into squashfs-root using /bin/sh

## Findings
- Successfully entered emulated DVRF router as root (uid=0 gid=0)
- Architecture confirmed: MIPS
- Found pwnable challenges: /pwnable/Intro, /pwnable/ShellCode_Required



## Screenshot
<img width="1366" height="720" alt="Screenshot 2026-03-30 121641" src="https://github.com/user-attachments/assets/f0d93b06-cd65-4dc5-8e1b-9c27e46b7b84" />
<img width="1366" height="720" alt="Screenshot 2026-03-30 123646" src="https://github.com/user-attachments/assets/06a1798a-4ff7-4036-9bd8-10cc9410b3b2" />
