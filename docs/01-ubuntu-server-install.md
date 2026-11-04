# 01 - Ubuntu Server Install

## Goal
Install Ubuntu Server LTS on an old laptop to use as an always-on home lab server.

## Hardware
- HP EliteBook 2560p
- 4 GB RAM
- Corsair Force LS 240 GB SSD
- Wired Ethernet through a switch

## Steps
1. Downloaded the Ubuntu Server LTS ISO and wrote it to a USB stick.
2. Booted the laptop from the USB (F9 boot menu).
3. Chose "Use an entire disk" on the internal SSD, with LVM.
4. Enabled OpenSSH server during install.
5. Rebooted and logged in.

## Problems and fixes

### Problem: "Boot device not found" after install
The laptop has a legacy BIOS and refused to boot the GPT disk the installer created.

**Fix:** booted the installer USB, opened a shell, and set the boot flag on the disk:

    sudo parted /dev/sda disk_set pmbr_boot on

After removing the USB and rebooting, the server booted normally.

### Problem: "Autoconfiguration failed" on the network screen
I connected the ethernet cable only after entering setup and the ipv4 was turned off for the ethernet connection. The fix was to set it on Automatic(DHCP).

## Result
Server boots and I can log in as `servlab`.

## What I learned
- Old laptops with legacy BIOS can need a protective MBR boot flag on GPT disks.
- Check the boot mode and the disk layout before reinstalling.