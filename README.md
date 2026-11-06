# Home Lab

A cybersecurity home lab built to learn Linux administration, networking, and security. Everything here is documented as I build it, including the mistakes.

## Hardware
- **Server:** HP EliteBook 2560p, 4 GB RAM, 240 GB SSD, Ubuntu Server LTS (always on, managed over SSH)
- **Workstation:** 16 GB laptop, used for VMs and security labs
- **Network:** both laptops wired through an unmanaged gigabit switch to the home router

## Goals
- Run self-hosted services on a hardened Ubuntu server
- Build an isolated security lab with intentionally vulnerable VMs
- Practice attack and detection, and write up the results

## Security rules I follow
- No port forwarding on the router. Remote access will go through a VPN (Tailscale).
- SSH keys only, no password login.
- Each machine has its own SSH key pair, and private keys never leave the machine that made them.
- Vulnerable VMs stay on an isolated virtual network.
- Only my own machines are ever scanned or attacked.
- No secrets are committed to this repo.

## Progress
- [x] Ubuntu Server installed
- [x] SSH hardening (keys only, root login off)
- [x] SSH key management (separate keys for Windows, Kali VM, and GitHub)
- [x] Firewall, fail2ban, automatic updates
- [x] Docker
- [x] Pi-hole
- [x] Tailscale
- [ ] Isolated security lab (Kali + vulnerable VMs)
- [ ] Monitoring and detection (Wazuh)

## Docs
- [01 - Ubuntu Server install](docs/01-ubuntu-server-install.md)
- [02 - SSH hardening and key management](docs/02-ssh-hardening.md)
- [03 - Docker install](docs/03-docker-install.md)
- [04 - Pi-hole](docs/04-pihole.md)
- [05 - Tailscale](docs/05-tailscale.md)
- [Lessons learned](lessons-learned.md)