---
title: "Setting up Vuln Lab in Proxmox"
date: 2022-03-28T22:11:55+08:00
draft: true
---

The machine i downloaded and install are from NetSecFocus Trophy Room, under the vulnhub section. I will install the machine highligted in the red section into my proxmox.

I only manage to install 32 machines out of 38 into the proxmox. 

6 Machines that i didn't installed due to: 

- Healthcare - (Too old for it to work, maybe?)
- Photography - (File too big for unknown reason : 500GB)
- GlasgowSmile - (Cannot boot, lead to grub rescue)
- Breach 3 - (Got IP in Proxmox but appear offline)
- LordoftheRoot - (Got IP in Proxmox but appear offline)
- Alpha 1 - (Link Broken)

## Installation  

First, we download the machine file from vulnhub. Take note that there's 2 ways to download the file (Torrent or Browser). I will be using torrent because not all file can be downloaded directly from the browser.

After we are done with downloading, inspect the file. Look for .vmdk file. We are only interested in that file. If we only see .ova file, then proceed with the next step. 

#### .ova to .vmdk

We need to convert the .ova file to .vmdk file. To do this, we simply need to extract the file inside .ova. There's a few way to do this. This is also doable using `winrar` for windows. For linux, simply use the `tar` command in linux to extract it. 

```
tar -xvf <filename.ova>
```

Now that we have extracted the .vmdk file from .ova, we can send this to the proxmox. 

#### .vmdk file to proxmox

Since im using windows, i will transfer the .vmdk file to proxmox using `WinSCP`.

Just connect to proxmox server by specifying the IP address and credentials in the `WinSCP`. Transfer the .vmdk file from wherever you store it, to the /tmp folder of the proxmox. 

#### Create a VM Template in Proxmox 

#### Creating a VM and Importing the .vmdk file

#### Special Case: Black Screen after booting

Black Screen upon installation: 

Check up on TommyBoy machine. (Working)
Check up on Sar machine. (Working) 

#### Troubleshooting: Cannot Boot

- Change boot type from SCSI to IDE

All SCSI except: 

- Bravery (iDE)
- Fall (iDE)
- RickdiculouslyEasy (iDE)

#### Troubleshooting: Vuln Machine not recieving DHCP IP 

- Enter grub
- Change the setting to get root shell 
- Edit the network config file 
- /etc/network/interfaces
- /etc/netplan
- Reboot / exec restart 

https://netplan.io/examples/
https://infra.engineer/linux/39-setting-a-static-ip-address-in-ubuntu-16-04-vs-18-04-lts

## Summary 

asdjklasjdas



