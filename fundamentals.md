---
layout: default
permalink: /RE101/section1/
title: Fundamentals
---
[Go Back to Reverse Engineering Malware 101](https://securedorg.github.io/RE101/)

# Section 1: Fundamentals #

## Environment Setup ##

In this section you will be setting up a safe virtual malware analysis environment. The virtual machine (VM) that you will be running the malware on should not have internet access nor network share access to the host system. This VM will be designated as the **Victim VM**. On the other hand, the **Sniffer VM** will have a passive role in serving and monitoring the internet traffic of the Victim VM. This connection remains on a closed network within virtualbox.

### Installing VirtualBox ###

For windows and osx, follow the instructions in the install binary.

| Windows | Mac OSX | Linux |
| --- | --- | --- |
| [![alt text](https://securedorg.github.io/images/VBwin.png "Windows Virualbox")](http://download.virtualbox.org/virtualbox/5.1.14/VirtualBox-5.1.14-112924-Win.exe) | [![alt text](https://securedorg.github.io/images/VBmac.png "OSX Virtualbox")](http://download.virtualbox.org/virtualbox/5.1.14/VirtualBox-5.1.14-112924-OSX.dmg) | [![alt text](https://securedorg.github.io/images/Vblinux.png "Linux Virtualbox")](https://www.virtualbox.org/wiki/Linux_Downloads) |

### Download Victim and Sniffer VMs ###

Please use the utility [7zip](http://www.7-zip.org/download.html). Unzip the files with 7zip below and open the .ovf or .vbox file with VirtualBox. 

These VMs require an **Intel** Architecture. AMD will not work for these snapshots (I will update support for them later).

[Victim VM](https://drive.google.com/file/d/0B_0DJl2kuzoNZkpveEtiMWJKWDA/view?usp=sharing)

* MD5sum: 4ad7b30b341db57dffb97e44189aed38
* OS: Windows 7 Service Pack 1
* Architecture: Intel 32bit
* Username: victim
* Password: re1012017
* IP Address: 192.168.0.2
* Gateway: 192.168.0.1

[Sniffer VM](https://drive.google.com/file/d/0B_0DJl2kuzoNT3IwNElLV3VRdms/view?usp=sharing)

* MD5sum: be459de4cdee86f0c35582973356d506
* OS: Ubuntu 16.04.2 LTS Desktop
* Architecture: Intel 64bit
* Username: sniffer
* password re1012017
* IP Address: 192.168.0.1
* Gateway: 192.168.0.1

---

### Post Install Instructions ###

1. Install VirtualBox CD on both VMs: Devices->Insert Guest Additions CD Image
  * If it doesn't auto appear, navigate to the CD Drive to install
  * Follow install directions from the Guest Additions Dialog
  * Note: it will require install privileges so insert passwords for each VM
  * Shutdown Both VMs after you have installed the Guest Additions CD.
2. Victim VM: Devices->Drag and Drop->Bidrectional 
3. Victim VM: Devices->Shared Clipboard->Bidirectional
4. Both VMs: Devices->Network->Network Settings
  *  Select Attached to `Internal Network`
  *  Name should mirror both VMs. Default is `intnet`
5. Run/Play both VMs to verify network connectivity
6. Sniffer VM: Ensure `inetsim` is running
  * Open terminal and run: `ps -ef | grep inetsim`
  * If no output, run: `/etc/init.d/inetsim start`
  * Run the ps command again to confirm it's running.
  * Expected output: ![alt text](https://securedorg.github.io/images/VerifyInetsim.png "ps output")
7. Victim VM: test connection to Sniffer VM
  * In the search bar, type `cmd.exe` to open terminal
  * Run command: `ping 192.168.0.1`
  * Expected output: ![alt text](https://securedorg.github.io/images/PingGateway.png "Ping Output")
8. Sniffer VM: Devices->Shared Folders->Shared Folders Settings
  * On your Host, create a folder called `sniffershare`
  * In virtual box select Add New Shared Folder icon and navigate to the folder you just created (sniffershare)
  * In Sniffer VM, open the terminal and run command:`mkdir ~/host; sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) sniffershare ~/host`

[Intro <- Back](https://securedorg.github.io/RE101/intro) | [Next -> Anatomy of PE](https://securedorg.github.io/RE101/section1.2)
