# 03 - Docker Install

## Goal
Install Docker on the server (`servlab`) so I can run self-hosted services in containers, starting with Pi-hole.

## Why Docker and not VMs
The server has 4 GB of RAM. Each VM needs its own full operating system, which would use most of that memory. Containers share the host's kernel, so they start fast and use far less RAM and disk. They are also easy to rebuild from a config file, which fits how I want to document the lab.

## Steps

### 1. Install prerequisites and add Docker's signing key
```
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
I used Docker's official repository instead of Ubuntu's default package, because the default is often outdated.

### 2. Add the repository
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 3. Install Docker
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### 4. Test
```
sudo docker run hello-world
sudo docker compose version
```
The `hello-world` container printed a message confirming the installation works.

## Security decisions
- **I did not add my user to the `docker` group.** Anyone in that group can start a container that mounts the host's filesystem, which is effectively root access. I use `sudo docker` instead.
- **Docker can bypass the `ufw` firewall.** Published container ports are added to iptables rules that `ufw` does not control. There is no port forwarding on my router, so services are only reachable from my home network, and I will bind ports to specific addresses on purpose.

## Problems and fixes
None

## Result
Docker and Docker Compose are installed and working. `Docker Compose version v5.7.0`

## What I learned
- Containers share the host kernel, so they are much lighter than VMs.
- Group membership can quietly grant root-level power.
- A firewall rule set is only useful if I know which tools can change it.