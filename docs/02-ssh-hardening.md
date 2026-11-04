# 02 - SSH Hardening and Key Management

## Goal
Make SSH on the server (`servlab`) safe by using key-based login only, and manage keys so that no single compromised machine exposes everything.

## Design
- Each machine that needs access gets its **own key pair**.
- The server only stores **public** keys. Private keys stay on the machine that made them.
- A VM used for attack tools (Kali) does not get access by default.

| Key | Made on | Used for |
|---|---|---|
| `id_servlab` | Windows host | Logging in to the server |
| `id_github` | Windows host | Pushing to GitHub |
| `id_servlab_kali` | Kali VM | Optional, revocable access to the server |

Each key has its own passphrase.

## Steps

### 1. Create a key on the host (Git Bash)
    ssh-keygen -t ed25519 -C "windows-to-servlab" -f ~/.ssh/id_servlab

### 2. Add the public key to the server
Appended the contents of `id_servlab.pub` as a new line in `~/.ssh/authorized_keys` on the server, then set permissions:

    chmod 600 ~/.ssh/authorized_keys

### 3. Add a shortcut on the host
In `~/.ssh/config`:

    Host servlab
        HostName <server-ip>
        User <username>
        IdentityFile ~/.ssh/id_servlab

Now `ssh servlab` is enough to connect.

### 4. Disable password login
Created `/etc/ssh/sshd_config.d/01-hardening.conf`:

    PasswordAuthentication no
    PermitRootLogin no

Validated the config before restarting:

    sudo sshd -t
    sudo systemctl restart ssh

### 5. Verify the effective settings
    sudo sshd -T | grep -iE 'passwordauthentication|permitrootlogin'

Then tested from a second terminal:
- `ssh servlab` logs in with the key.
- `ssh -o PubkeyAuthentication=no <user>@<server-ip>` is refused with `Permission denied (publickey)`.

I kept my original session open until both tests passed, so a mistake could not lock me out.

### 6. Retire the old key
The first key had been generated inside the Kali VM. Since that VM runs attack tools and gets snapshotted and rolled back, I:
1. Generated a new key on the Windows host.
2. Confirmed the new key worked.
3. Removed the old public key line from `authorized_keys` on the server.
4. Deleted the old private key in the VM with `shred -u`.

### 7. Optional access from Kali
Created a separate key in the VM (`id_servlab_kali`) with its own passphrase and added its public key to the server. If the VM is ever compromised, deleting that one line from `authorized_keys` removes its access without affecting the Windows key.

## Problems and fixes

### Password login stayed on after "hardening"
- **Symptom:** `ssh <user>@<server-ip>` still asked for a password.
- **Cause:** `sshd -t` showed `Bad configuration option: PasswordAuthentification`. A typo on line 1 made sshd reject the whole file, so none of the settings applied.
- **Fix:** corrected the spelling, renamed the file to `01-hardening.conf` so it is read before `50-cloud-init.conf` (sshd uses the first value it finds), and confirmed with `sshd -T`.
- **Lesson:** check the *effective* config with `sshd -T`, and always run `sshd -t` before restarting the service.

## Result
Only key-based login works, root login is disabled, and each machine has its own revocable key.

## What I learned
- Private keys should stay on the machine that created them.
- Separate keys per machine mean I can revoke one without affecting others.
- Files in `sshd_config.d/` are read in alphabetical order and the first value wins.
- Never close the working session until the new settings are tested from a second one.