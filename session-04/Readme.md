# Pi-hole Deployment with Docker on Ubuntu

## Project Overview

For this lab, I deployed **Pi-hole inside a Docker container on an Ubuntu virtual machine**. The purpose of this project was to get more hands-on experience with Ubuntu, Docker, networking, DNS, SSH, and troubleshooting.

This installation did not work perfectly on the first try. I ran into several issues involving Docker permissions, container networking, and port 53. Instead of only documenting the successful installation, I wanted to include the errors I encountered and how I resolved them because troubleshooting was a major part of this lab.

## Lab Environment

* Ubuntu Linux VM
* Docker
* Docker Compose
* Pi-hole
* SSH
* DNS / Port 53
* Web browser for Pi-hole administration

---

## 1. Preparing the Ubuntu VM

I first checked my Ubuntu VM resources to make sure I had enough space to run Pi-hole.

My system showed approximately:

```text
System load:       0.0
Disk usage:        30.6% of 24.44 GB
Memory usage:      8%
Swap usage:        0%
Processes:         156
```

The VM had more than enough resources for this project.

I also verified that Ubuntu had network connectivity:

```bash
ping -c 3 8.8.8.8
```

I received three successful replies with 0% packet loss.

---

## 2. Connecting to the Ubuntu VM

After rebuilding my VM, I received a new IP address and connected to the Ubuntu machine through SSH.

```bash
ssh <username>@<ubuntu-ip-address>
```

This allowed me to manage the Ubuntu server remotely instead of working directly inside the VM console.

---

## 3. Creating the Pi-hole Docker Configuration

I created a directory for the Pi-hole project and added my Docker Compose YAML file.

Example structure:

```text
pihole/
├── docker-compose.yml
└── etc-pihole/
```

My Docker Compose configuration included the Pi-hole image, DNS ports, web interface port, persistent storage, and upstream DNS servers.

Example:

```yaml
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole

    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:80/tcp"

    environment:
      TZ: "America/Los_Angeles"
      FTLCONF_webserver_api_password: "<YOUR_PASSWORD>"
      FTLCONF_dns_listeningMode: "ALL"

    volumes:
      - ./etc-pihole:/etc/pihole

    dns:
      - 1.1.1.1
      - 8.8.8.8

    restart: unless-stopped
```

**Security note:** I do not store my real Pi-hole administrative password in my public GitHub repository.

---

# Troubleshooting

This was the most important part of the project.

## Issue 1: Docker Permission Denied

My first error was:

```text
unable to get image 'pihole/pihole:latest':
permission denied while trying to connect to the docker API
at unix:///var/run/docker.sock
```

This happened because my Ubuntu user did not have permission to communicate with the Docker daemon through `/var/run/docker.sock`.

For this lab, I ran the Docker Compose command with elevated privileges:

```bash
sudo docker compose up -d
```

Docker was then able to pull the Pi-hole image.

### What I Learned

Docker runs as a system-level service. A regular Linux user may not automatically have permission to communicate with the Docker daemon.

---

## Issue 2: Port 53 Was Already in Use

The next error was:

```text
failed to bind host port 0.0.0.0:53/tcp:
address already in use
```

This was important because **port 53 is the standard port used for DNS**, and Pi-hole needs to listen on this port.

I investigated what was using port 53:

```bash
sudo ss -ltnup | grep ':53'
```

The results showed:

```text
systemd-resolved
```

Ubuntu's `systemd-resolved` service was already using port 53 for its local DNS stub resolver.

Because Pi-hole was also attempting to use port 53, Docker could not publish the required DNS port.

---

## 4. Disabling the systemd-resolved DNS Stub Listener

I used the Pi-hole documentation to resolve the Ubuntu port 53 conflict.

I created a configuration file that disabled the DNS stub listener:

```bash
sudo sh -c 'mkdir -p /etc/systemd/resolved.conf.d && printf "[Resolve]\nDNSStubListener=no\n" | tee /etc/systemd/resolved.conf.d/no-stub.conf'
```

This prevents `systemd-resolved` from occupying port 53.

---

## 5. Updating `/etc/resolv.conf`

Disabling the stub listener alone could interfere with Ubuntu's DNS resolution because `/etc/resolv.conf` may still point to the local stub resolver.

I updated the symbolic link:

```bash
sudo sh -c 'rm -f /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'
```

I then restarted `systemd-resolved`:

```bash
sudo systemctl restart systemd-resolved
```

I verified port 53 again:

```bash
sudo ss -ltnup | grep ':53'
```

This allowed Docker/Pi-hole to use port 53.

---

## 6. Starting Pi-hole

I started the container using:

```bash
sudo docker compose up -d
```

I checked the container:

```bash
sudo docker ps
```

I also reviewed the Pi-hole logs:

```bash
sudo docker logs pihole
```

During troubleshooting, I learned that a container showing as `Started` does not necessarily mean that every service inside the container is fully operational. Checking the logs and testing connectivity helped me understand what was actually happening.

---

## 7. Accessing the Pi-hole Web Interface

After resolving the Docker networking and port 53 conflict, I entered the Ubuntu VM's IP address and configured web port into my browser.

For my configuration, the web interface was mapped to port `8081`.

Example:

```text
http://<ubuntu-ip-address>:8081/admin/
```

The Pi-hole administration interface successfully loaded.

**Pi-hole was officially up and running.**

---

# Commands That Helped Me Troubleshoot

Check Ubuntu internet connectivity:

```bash
ping -c 3 8.8.8.8
```

Check DNS resolution:

```bash
ping -c 3 google.com
```

Check Docker containers:

```bash
sudo docker ps
```

Check Pi-hole logs:

```bash
sudo docker logs pihole
```

Check what is using DNS port 53:

```bash
sudo ss -ltnup | grep ':53'
```

Inspect Docker networks:

```bash
sudo docker network ls
```

Inspect the Pi-hole Docker network:

```bash
sudo docker network inspect pihole_default
```

Inspect Pi-hole's network attachment:

```bash
sudo docker inspect pihole --format '{{json .NetworkSettings.Networks}}'
```

---

# What I Learned

This project taught me much more than simply installing Pi-hole.

I gained hands-on experience with:

* Linux command-line administration
* SSH remote administration
* Docker containers and images
* Docker Compose
* Docker bridge networking
* TCP and UDP ports
* DNS and port 53
* `systemd-resolved`
* Linux configuration files
* `/etc/resolv.conf`
* Reading Docker logs
* Testing network connectivity
* Troubleshooting services instead of immediately reinstalling software

One of my biggest takeaways was learning how different layers of a system can affect each other.

At first, Pi-hole appeared to have a Docker networking problem because the container could not reach the network. After troubleshooting further, I discovered that the underlying problem was **Ubuntu's `systemd-resolved` service already using port 53**. Docker could not successfully publish Pi-hole's DNS port, which prevented the container from being configured correctly.

I also learned the importance of testing one layer at a time:

```text
Ubuntu Host
    ↓
Network Connectivity
    ↓
Docker
    ↓
Docker Network
    ↓
Pi-hole Container
    ↓
DNS / Port 53
    ↓
Pi-hole Web Interface
```

Instead of assuming Pi-hole itself was broken, I tested each part of the environment until I found the actual conflict.

---

## Final Result

✅ Ubuntu VM running
✅ SSH connectivity established
✅ Docker installed and operational
✅ Pi-hole Docker image deployed
✅ Docker networking configured
✅ Port 53 conflict identified
✅ `systemd-resolved` stub listener disabled
✅ Ubuntu DNS resolution maintained
✅ Pi-hole container running
✅ Pi-hole web administration page accessible

## References

I used the official Pi-hole Docker documentation while troubleshooting the Ubuntu port 53 conflict:

**Pi-hole Documentation — Docker Tips and Tricks**
`https://docs.pi-hole.net/docker/tips-and-tricks/`

---

*This project is part of my continued hands-on learning with Linux, networking, cybersecurity, and system administration.*

