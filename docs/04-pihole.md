# 04 - Pi-hole

## Goal
Run Pi-hole in Docker on the home server as a network-wide DNS filter, to learn how DNS works and to cover devices that cannot run a browser extension (phones, TVs, consoles).

## How it works
DNS turns domain names into IP addresses. Pi-hole acts as the DNS server for the network. When a device asks for a domain on a blocklist, Pi-hole refuses to answer, so the ad or tracker never loads. Everything else is forwarded to an upstream resolver.

    Device -> Pi-hole (blocks listed domains) -> upstream DNS (Quad9) -> answer

## Setup

### 1. Private values
Created a project folder on the server with a `.env` file (restricted permissions) for the values that must not be public. The repo contains only `.env.example`.

### 2. Compose file
`configs/pihole/docker-compose.yml`. Key decisions:
- Ports are bound to the server's LAN address, not every interface.
- The admin page uses a non-default port, so the standard web port stays free for a reverse proxy later.
- Data is stored in a volume so settings survive container recreation.
- The password comes from `.env`, so no secret is in the compose file.
- `restart: unless-stopped` brings it back after a reboot.

### 3. Start it
```
sudo docker compose up -d
sudo docker ps
```

### 4. Firewall
Restricted DNS and the admin page to the home network only, using `ufw` rules. Docker can bypass `ufw` for published ports, so the real protection is that the router forwards no ports to the server.

### 5. Test on one device first
Set DNS manually on one laptop only, then checked:
```
nslookup google.com <server-ip>
nslookup doubleclick.net <server-ip>
```
The first resolved normally and the second came back blocked. The Query Log showed the laptop's requests.

### 6. Blocklist and upstream
- Added the HaGeZi Pro list under Lists, then ran Update Gravity.
- Set the upstream to Quad9 (filtered, DNSSEC). I chose it because it is run by a non-profit, does not log client IPs, and also blocks known malicious domains, which suits a security lab.

### 7. Router
In the router's DHCP settings, under the DNS section:
- DNS Server 1: the Pi-hole server
- DNS Server 2: a backup resolver, so the house keeps internet if the server is off

I left the router's own WAN DNS setting alone, because that only controls what the router itself uses, not what it hands out to devices.

Reconnected devices and confirmed in the Query Log that phones and other devices appeared. IPv6 was left off so devices do not bypass Pi-hole. I set the laptop's DNS back to automatic so it follows the router.

## What the DNS list looks like on a client
`ipconfig /all` on Windows shows three DNS servers:
1. The Pi-hole server
2. The backup resolver
3. The router itself, which adds its own address and forwards to the ISP's DNS

Windows tries them in order and moves to the next one if the first does not answer.

**Trade-off:** because the backups are in the list, devices occasionally use them instead of Pi-hole, so some ads and trackers get through. Removing the backup would give consistent blocking, but the whole house would lose internet whenever the server is off.

## Testing failover
1. Stopped the container: `sudo docker compose stop`
2. Flushed the cache: `ipconfig /flushdns`
3. Tested with `ping google.com` and by opening a site I had not visited recently.
4. Started it again: `sudo docker compose start`

**Result:** the first attempt did not resolve, because Windows was still holding cached DNS answers from Pi-hole. After flushing the cache, `ping google.com` and the browser worked, so Windows fell back to the backup DNS and the failover works.

**Note:** `nslookup google.com` with no server specified failed during the test. That is because `nslookup` only asks the first DNS server in the list and does not fall back, unlike the rest of Windows. It did not mean the backup was broken.

## Problems and fixes

### Ads still appeared on some sites
- **Cause:** DNS blocking cannot stop ads served from the same domain as the content, popups, or redirect ads. Browsers with their own encrypted DNS can also bypass Pi-hole.
- **Fix:** installed uBlock Origin in the browser, turned off "secure DNS" in the browser settings, and flushed the DNS cache with `ipconfig /flushdns`.

### Failover test looked like it failed
- **What happened:** with the container, `nslookup google.com` timed out, and a test site still showed no ads.
- **Cause:** `nslookup` only queries the first server, and Windows was holding cached answers. The missing ads came from the browser ad blocker, not from Pi-hole.
- **Fix:** flushed the DNS cache and tested with `ping` and the browser instead, which use Windows' normal fallback.

## Result
Pi-hole filters DNS for the whole network, with Quad9 as the upstream and a backup DNS on the router.

## Limits and trade-offs
- DNS blocking does not stop YouTube ads, same-domain ads, or popups.
- It does not hide browsing. The upstream resolver sees every domain lookup, which is why I picked a privacy-focused one.
- Pi-hole logs every device's queries on my server, so it holds sensitive data. Privacy settings can reduce it.
- It does not affect game ping or download speed. DNS only happens at the start of a connection, and cached answers are often faster than my ISP's.
- If the server is down, devices rely on the backup DNS, so some ads get through but internet keeps working.

## What I learned
- How DNS resolution works and where filtering fits.
- Why binding ports and not forwarding router ports matters with Docker.
- Keeping secrets out of Git with a `.env` file and an `.env.example`.
- How to test a change on one device before rolling it out to the network.
- That a test tool can behave differently from the real system. `nslookup` and `ping` do not handle DNS fallback the same way, and cached answers can hide the real result.