# Lessons Learned

A running list of mistakes, fixes, and things I would do differently.

## Ubuntu install: "boot device not found"
- **What happened:** after a successful install, the HP EliteBook 2560p (legacy BIOS) would not boot.
- **Cause:** the installer created a GPT disk, and the old BIOS did not see the boot flag in the protective MBR.
- **Fix:** booted the installer, opened a shell, and ran `sudo parted /dev/sda disk_set pmbr_boot on`.
- **Takeaway:** on old hardware, check boot mode (UEFI vs legacy) and the disk layout before reinstalling.

## Network: "autoconfiguration failed" in the installer
- **What happened:** the installer could not get an IP over Ethernet.
- **Fix:** (write what actually solved it, for example a different cable or switch port, or the router's DHCP.)
- **Takeaway:** check cables, switch uplink, and router DHCP before suspecting the laptop.

## SSH hardening: typo kept password login enabled
- **What happened:** I thought password login was off, but it was still accepted.
- **Cause:** a misspelled option made sshd ignore my whole config file.
- **Takeaway:** verify with `sshd -t` and `sshd -T` instead of assuming.

## Generating keys on the wrong machine
- **What happened:** I first created the server key inside a Kali VM.
- **Fix:** made a new key on the host and removed the old one.
- **Takeaway:** private keys belong on the host machine, not on disposable VMs.

## Things I would do differently
- Take notes while working instead of afterward.
- Test every security change from a second session before closing the first.