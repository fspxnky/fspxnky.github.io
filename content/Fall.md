---
title: "Fall"
date: 2022-04-15T19:05:23+08:00
draft: true
tags: ["Vulnhub"]
---

To celebrate the fifth year that the author has survived his infosec career, a new box has been born! This machine resembles a few different machines in the PEN-200 environment (making it yet another OSCP-like box). More enumeration practice indeed!

Feel free to contact the author at https://donavan.sg/blog if you would like to drop a comment.

> The machine IP address is `192.168.50.113` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine.

## Scanning 

Let us start by scanning using Nmap to find any opened ports in the machine.

```
sudo nmap -sS -p- 192.168.50.113 -oN initial.scan
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-15 07:19 EDT
Nmap scan report for 192.168.50.113
Host is up (0.0026s latency).
Not shown: 65363 filtered tcp ports (no-response), 159 filtered tcp ports (host-prohibited)
PORT      STATE  SERVICE
22/tcp    open   ssh
80/tcp    open   http
111/tcp   closed rpcbind
139/tcp   open   netbios-ssn
443/tcp   open   https
445/tcp   open   microsoft-ds
3306/tcp  open   mysql
8000/tcp  closed http-alt
8080/tcp  closed http-proxy
8443/tcp  closed https-alt
9090/tcp  open   zeus-admin
10080/tcp closed amanda
10443/tcp closed cirrossp
```

## Enumeration 

#### Port 80 and 443 (HTTP CMS Made Simple) 

- searchsploit -x 49345
- Need log in credentials
- Has a login page 
- test.php -> Need use burp  

#### Port 139 and 445 (Samba) 

- Nothing

#### Port 3306 (MySQL)

-  

#### Port 9090 (HTTP Zeus-Admin)

- Nothing as of now 


patrick and qiu