---
title: "Bravery"
date: 2022-03-30T17:46:31+08:00
draft: true
tags: ["Vulnhub"]
---

This machine hopes to inspire BRAVERY in you; this machine may surprise you from the outside. This is designed for OSCP practise, and the original version of the machine was used for a CTF. It is now revived, and made more nefarious than the original. BRAVERY is a VirtualBox VM built, but I host it on Proxmox Server. To get the VM image, click [here](https://www.vulnhub.com/entry/digitalworldlocal-bravery,281/).

Feel free to contact the author at https://donavan.sg/blog if you would like to drop a comment.

> The machine IP address is `192.168.50.109` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine.

## Scanning 

Let us start by scanning using Nmap to find any opened ports in the machine.

```
sudo nmap -sS -p- 192.168.50.109 -oN initial.scan

```

I find that several ports is open: 

- SSH = Port 22
- DNS = Port 53 
- HTTP/HTTPS = Port 80 and 443, Port 8080
- MySQL = Port 3306 
- NFS = Port 111 and 2049
- Samba = Port 139 and 445

Doing another scan to know more about the expose ports: 

```
nmap -sC -sV -p22,53,80,111,139,443,445,2049,3306,8080 192.168.50.109 -oN script.scan

```

## Enumeration

Looking at Port 2049 using the tool `showmount`, i found that the shared folder is `/var/nfsshare`. 

```
showmount -e 192.168.50.109
```

To mount this, i simply create a folder for it to mount in my `/tmp` folder. 

```
mkdir /tmp/mnt
sudo mount -t nfs 192.168.50.109:/var/nfsshare /tmp/mnt
cd /tmp/mnt
```

Viewing the folder that i've mount, I found a few files inside the shared folder. Every file of this is a clue of where the password is. 

## Initial Foothold via LFI/RFI 

## Privileged Escalation
#### apache to root

## Summary