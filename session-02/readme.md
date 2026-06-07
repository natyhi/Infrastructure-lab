# Week 02 - Linux Ubuntu and SSH

## What I Built

Installed Ubuntu Server 24.04 on a VirtualBox VM and connect to it via SSH from my host machine.

## Virtual Machine Server Specs
- Ram 4096 MB
- Disk 25 GB
- OS: Ubuntu Server 24.04
- Network Bridge Adapter
- Hostname 'ubuntu-lab'
- Username 'labuser'

 ## Server IP Address

- IP:192.168.0.58
 
 ## Commands I Ran

 Updating the Server
 
 ```
sudo apt update
sudo apt upgrade
check disk space
 ```

```
df -h
```

Check memory use
```
free -h
```
Connected via SSH
```
ssh labuser@192.168.0.58
```

## Observations
1. The Server has 15 GB free space after installation
2. RAM usage was at 500 MB at idle
3. The SSH connection was instant after first connection was established
4. SSH has remembered the VM as trusted connection after confirming first connect

## Session Checklist
- [x] Ubuntu Server Installed
- [x] SSH access confirmed

## Problems

See 'troubleshooting.md' for the issues and resloutions
