# 04 - Pi-hole

## Goal
Run Pi-hole in Docker on the home server as a network-wide DNS filter, to learn how DNS works and to cover devices that cannot run a browser extension (phones, TVs, consoles).

## How it works
Pi-hole acts as the network's DNS server. It refuses to resolve domains on a blocklist, so those ads and trackers never load, and forwards everything else to an upstream resolver.

    Device -> Pi-hole (blocks listed domains) -> upstream DNS (Quad9) -> answer

## Setup
- **Secrets:** kept in a `.env` file with restricted permissions. The repo contains only `.env.example`.
- **Compose file** (`configs/pihole/docker-compose.yml`): ports bound to the server's LAN address only, a volume for persistent data, and `restart: unless-stopped`.
- **Firewall:** DNS and the admin page restricted to the home network with `ufw`. Docker can bypass `ufw` for published ports, so the real protection is that the router forwards no ports.
- **Testing:** set DNS on one laptop first and checked with `nslookup` that normal domains resolved and ad domains were blocked.
- **Blocklist and upstream:** added the HaGeZi Pro list, and chose Quad9 (filtered) because it is run by a non-profit, does not log client IPs, and blocks known malicious domains.
- **Router:** DHCP DNS Server 1 is Pi-hole and DNS Server 2 is a backup resolver, so the house keeps internet if the server is off. IPv6 was left off so devices do not bypass Pi-hole.

## Failover test
Stopped the container, flushed the DNS cache with `ipconfig /flushdns`, and tested with `ping` and the browser. Windows fell back to the backup DNS and everything worked.

`nslookup` with no server specified failed during the test, because it only asks the first DNS server and does not fall back like the rest of Windows. It did not mean the backup was broken.

## Problems and fixes
- **Ads still appeared:** DNS blocking cannot stop same-domain ads, popups, or redirects, and browsers with their own encrypted DNS can bypass Pi-hole. I added uBlock Origin and turned off the browser's "secure DNS" setting.
- **Failover looked broken:** `nslookup` and cached answers gave a misleading result. Testing with `ping` and the browser after flushing the cache showed the real behavior.

## Limits and trade-offs
- It does not stop YouTube ads or popups, and it does not hide browsing from the upstream resolver.
- It logs every device's queries on my server, so that data is sensitive.
- It has no effect on game ping or download speed, because DNS only happens at the start of a connection.
- Backup DNS means some lookups skip Pi-hole and a few ads get through, but internet keeps working if the server is down.

## What I learned
- How DNS resolution works and where filtering fits.
- Why binding ports and not forwarding router ports matters with Docker.
- How to keep secrets out of Git with `.env` and `.env.example`.
- That test tools can behave differently from the real system, and cached answers can hide the real result.