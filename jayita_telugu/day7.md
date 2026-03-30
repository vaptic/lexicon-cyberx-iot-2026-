# Day 7 - Firmware Emulation with QEMU/Chroot

## Tools Used
- Kali Linux VM
- QEMU (qemu-user, qemu-user-binfmt)
- binwalk (used on Day 5)
- chroot

## Method Used
Used qemu-mipsel + chroot emulation because DVRF v0.3 only 
includes a single .bin file with no pre-split kernel/rootfs files.
The lab originally expected DVRF_v03_kernel and DVRF_v03_rootfs.ext2
which are not present in this version.

## Steps Completed

### 1. Install QEMU
```bash
apt install -y qemu-user qemu-user-static binfmt-support
```

### 2. Copy QEMU binary into firmware filesystem
```bash
cp /usr/bin/qemu-mipsel \
_DVRF_v03.bin.extracted/squashfs-root/usr/bin/
```

### 3. Mount proc, sys, dev
```bash
mount -t proc /proc squashfs-root/proc
mount -t sysfs /sys squashfs-root/sys
mount -o bind /dev squashfs-root/dev
```

### 4. Chroot into firmware
```bash
chroot squashfs-root /bin/sh
```

## Findings Inside Emulated Router
- Successfully entered as root: `uid=0 gid=0`
- Architecture confirmed: MIPS (Linux kali 6.16.8 mips unknown)
- Shell: BusyBox v1.7.2

## Pwnable Challenges Found
- `/pwnable/Intro/` → heap_overflow_01, stack_bof_01, uaf_01
- `/pwnable/ShellCode_Required/` → socket_bof, socket_cmd, stack_bof_02

## Running Services
- ps aux not fully supported in BusyBox msh shell
- Used `ps w` instead for wide process output

## Key Learnings
- chroot emulation is an alternative to full QEMU system emulation
- QEMU translates MIPS instructions to run on x86 host
- Firmware filesystem can be explored and executed without real hardware
- Dynamic analysis lets us interact with running firmware services

## Automation Script
Created: `~/scripts/start-dvrf-qemu.sh`
- Automates mounting and chrooting into DVRF firmware
- Made executable with chmod +x

   <img width="807" height="315" alt="Screenshot 2026-03-30 133125" src="https://github.com/user-attachments/assets/7e9aeb86-a445-4c52-9331-f4eed2cb9275" />
<img width="582" height="440" alt="Screenshot 2026-03-30 140413" src="https://github.com/user-attachments/assets/9bcf952a-abf4-4f1b-90f3-4b52d0074a3e" />
