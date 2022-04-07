---
title: "Setting up Vuln Lab in Proxmox"
date: 2022-03-28T22:11:55+08:00
draft: true
tags: ["Tutorial"]
---

The machine i downloaded and install are from NetSecFocus Trophy [Room](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#), under the vulnhub section. I will install the machine highligted in the red section into my proxmox.

I only manage to install 32 machines out of 38 into the proxmox. 

6 Machines that i didn't installed due to: 

- Healthcare - (Too old for it to work, maybe?)
- Photography - (File too big for unknown reason : 500GB)
- GlasgowSmile - (Cannot boot, lead to grub rescue)
- Breach 3 - (Got IP in Proxmox but appear offline)
- LordoftheRoot - (Got IP in Proxmox but appear offline)
- Alpha 1 - (Link Broken)

### Download Vulnerable Machine from Vulnhub  

First, lets download 1 machine from vulnhub. Take note that there's 2 ways to download the file (Torrent or Browser). I will be using torrent because not all file can be downloaded directly from the browser.

After we are done with downloading, inspect the file. Look for .vmdk file. We are only interested in that file. If we only see .ova file, then proceed with the next step. 

### .ova to .vmdk

We need to convert the .ova file to .vmdk file. To do this, we simply need to extract the file inside .ova. There's a few way to do this. This is also doable using `winrar` for windows, just extract the .ova file. For linux, simply use the `tar` command in linux to extract it. 

```
# Using oscp.ova downloaded from the vulnhub for this example
root@Karma:/tmp# tar -xvf oscp.ova
oscp.ovf
oscp.mf
oscp-file1.flp
oscp-disk1.vmdk
oscp-file2.iso
```

> We will get extra files when extracting the .ova file. Just delete the other file, we only need .vmdk.

Now that we have extracted the .vmdk file from .ova, we can now send this to the our proxmox. 

### .vmdk file to proxmox

Since im using windows, i will transfer the .vmdk file to proxmox using `WinSCP`.

Just connect to proxmox server by specifying the IP address and credentials in the `WinSCP`. Transfer the .vmdk file from wherever you store it, to the /tmp folder of the proxmox. 

I believe you can also transfer the file through the `scp` command in linux. More info can be found [here.](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/)

### Create a VM Template in Proxmox 

Inside the Proxmox GUI, right click on our node and create VM. The settings is as follows: 

- General > Name = Vulnhub Template
- OS = Do not use any media 
- System = Just click next
- Disks = No disks
- CPU = Just click next 
- Memory = 1024
- Network = Just click next
- Confirm = Click finish

Now, we click on the VM that we just created. Click on the "More" box which located on the right side of the GUI. Click on it, select "Convert to Template" and Click "Yes" to confirm.

We should now see this VM inside our nodes: 

{{<image src="/Vulnhub-Tutorial/Template.png" position="center" style="border-radius: 8px;">}}


### Creating a VM from the Template and Importing the .vmdk file

Now that we have a template ready, we can now create a VM and give that VM the .vmdk file for it to boot. 

Right click on the VM Template and click on "Clone". Change the Mode to "Full Clone" and give it a name for that VM. 

> Usually i will give the VM name according to the file i download from vulnhub.

Upon creating the VM, it will appear in the node. Please take note that the VM ID will be different. 

{{<image src="/Vulnhub-Tutorial/Template2.png" position="center" style="border-radius: 8px;">}}

For the next step, we will import the disk to the VM. Click on the node and navigate to the `Shell`. 

Head to the directory that you stored the .vmdk earlier and run this command: 

```
# qm importdisk <vmid> <source> <storage> <-- Example
qm importdisk 125 oscp-disk1.vmdk local-lvm
importing disk 'oscp-disk1.vmdk' to VM 125 ...
...
```

Once the importing is complete, we can click on the VM that we created, and head to `hardware`. We should be able to see an unused disk. 

Double click on the unused disk and set it as SCSI. Next is to change the boot under that can be found in the `Options` tab. Edit the boot order by only checking the scsi0 device

{{<image src="/Vulnhub-Tutorial/Template3.png" position="center" style="border-radius: 8px;">}}

Start the VM. The VM should now boot up and DHCP will give the VM an IP address. 

{{<image src="/Vulnhub-Tutorial/Template4.png" position="center" style="border-radius: 8px;">}}

**Some vuln machines ip address cannot be seen in the VM. So you either have to scan it manually or check it on your router for the DHCP leasing**

One last thing, once i got my VM machine to work, i will always snapshot it. This will make it easier to revert the machine back to "Working State" when i finish playing around with it.

### Special Case: Black Screen after booting

Black Screen upon installation: 

- TommyBoy
- Sar

The machine mentioned above will only show black screen upon booting up when i installed it in my proxmox. I initially thought that i did something wrong. **DO NOT WORRY. THE MACHINE IS WORKING. JUST VERIFY IT BY EITHER MANUALLY SCANNING THE NETWORK OR CHECK YOUR DHCP LEASE IN THE ROUTER**  

### Troubleshooting: Cannot Boot

All machine that i installed can boot with SCSI except: 

- Bravery (iDE)
- Fall (iDE)
- RickdiculouslyEasy (iDE)

Bravery with SCSI:

{{<image src="/Vulnhub-Tutorial/Template5.png" position="center" style="border-radius: 8px;">}}

Simply detach the drives and then set the drive to IDE. Re-booting the VM will now resolved the issue. This work for the other 2 machine mentioned above. 

Same machine with the drive change to IDE:

{{<image src="/Vulnhub-Tutorial/Template6.png" position="center" style="border-radius: 8px;">}}

### Troubleshooting: Machine not recieving DHCP IP

Another issue we will come across is when the machine is not recieving IP address. 

**0xBEN** here explain very well on how to work this around. You can read it more [here.](https://benheater.com/proxmox-vulnhub-vm-network-interface-issue/)

If the machine is using netplan configuration, then follow this examples to resolve the issue: https://netplan.io/examples/





