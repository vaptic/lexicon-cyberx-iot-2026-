#!/bin/bash
# DVRF QEMU Emulation Setup Script
# Generated with AI assistance, verified manually

SQUASHFS="$HOME/Downloads/DVRF/_DVRF_v03.bin.extracted/squashfs-root"

echo "[*] Copying qemu-mipsel into firmware..."
cp /usr/bin/qemu-mipsel $SQUASHFS/usr/bin/

echo "[*] Mounting proc, sys, dev..."
mount -t proc /proc $SQUASHFS/proc
mount -t sysfs /sys $SQUASHFS/sys
mount -o bind /dev $SQUASHFS/dev

echo "[*] Entering chroot..."
chroot $SQUASHFS /bin/sh
