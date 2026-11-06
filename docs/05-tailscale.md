# 05 - Tailscale

## Goal
Reach the home server securely from anywhere, without opening any ports on the router.

## Why Tailscale and not port forwarding
- Port forwarding exposes a service to the whole internet, and bots scan for open ports constantly.
- Some ISPs use CGNAT, which makes port forwarding impossible anyway.
- Tailscale devices connect outward to a coordination service, so nothing on the router is opened.

## How it works
- Tailscale builds a private network (a "tailnet") from my own devices, using the WireGuard protocol.
- Traffic goes directly between devices, encrypted end to end, or through an encrypted relay if a direct path is not possible.
- Tailscale and SSH do different jobs. Tailscale is the route to the server, and the SSH key is the identity check when I arrive.

## Setup
1. Signed in with an identity account that has 2FA enabled, since it controls which devices can join.
2. Downloaded the install script on the server and read it before running it, then ran `sudo tailscale up` and approved the device.
3. Installed Tailscale on the laptop and phone, using the same account.
4. Added a separate SSH config entry on the laptop so the same key is used over the tailnet (`ssh servlab-ts`).
5. Reviewed the device list in the admin console and kept exit nodes and subnet routing off.

The server runs Tailscale as a system service that starts at boot. The router still forwards no ports.

## Phone access
- Generated a separate ED25519 key on the phone, protected with a passphrase. The private key never leaves the phone.
- Sent only the public key to the laptop and added it to the server's `authorized_keys` on its own line.
- If the phone is lost, I remove its device in the admin console and delete its one line from `authorized_keys`, without affecting any other machine.

## Problems and fixes

### "Permission denied (publickey)" with the raw address
- **Cause:** SSH config entries only match the exact name typed after `ssh`. Using the raw address skipped my config, so no key was offered, and password login is disabled.
- **Fix:** use the alias, pass the key with `-i`, or add the address to the `Host` line.
- **Lesson:** a timeout is a network problem, while "permission denied" means the network works and authentication does not.

### Phone login failed with only "Normal Shutdown [preauth]" in the log
- **Cause:** the default log level does not show which key was offered, so the failure gave no clue.
- **Fix:** temporarily raised `LogLevel` to see the key fingerprints, compared them with `ssh-keygen -lf ~/.ssh/authorized_keys`, corrected the key setup, then removed the debug setting.
- **Lesson:** get the server to say why before changing things, and do not leave verbose logging on.

### A test that proved the wrong thing
- **What happened:** the server's alias worked while the Tailscale address timed out.
- **Cause:** the alias pointed at the home-network address, so it worked because I was on the LAN and said nothing about Tailscale.
- **Lesson:** know what a test is actually testing.

## Security notes
- I trust Tailscale's coordination service for logins and key distribution. It cannot read my traffic, but it controls which devices join.
- 2FA on the identity account matters, since anyone who takes it over could add a device.
- I remove devices I no longer use and review the list regularly.

## What I learned
- The difference between the network path (Tailscale) and authentication (SSH keys).
- How SSH config matching works.
- How to read and raise server logs to diagnose a failed login.
- How to reach services privately instead of exposing them publicly.