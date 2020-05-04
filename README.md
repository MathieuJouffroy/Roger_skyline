# Roger_skyline
Initiation to system and network administration : VM Creation

## Part 1 : VM setup with debian(64-bit) 10.3 on Virtualbox
VM Settings:
- Storage: Make sur Controller:IDE has debian 10.3 disk
- Network: Attached to: Bridged Adapter.

Partitioning (manual):
Create a virtual hard disk :
- VDI, Fixed allocated, 8.00 GB
- Primary partition : 4.5 GB
- Logical partition : 4.1 GB
- Packages : web server, ssh server, standard system utilities
- Install GRUB boot loader to the master boot record ? Yes

## Part 2 : Network
- Create non-root sudo user : (adduser username) 
- Configure DHCP service to have static address & netmask in /30 (etc/network/interfaces)
- Change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys.
SSH root access SHOULD NOT be allowed directly, but with a user who can be root. (etc/ssh/sshd_config)
- Firewall : on your server only with the services used outside the VM.
Installing iptables-persistent to make the rule change permanent.

## Part 3 : Security
Installing protection against DOS attacks on open ports (fail2ban)
- DOS : (Denial O Service Attack) protection on your open ports of your VM.
- Set a protection against scans on your VM’s open ports.
- Stop the services you don’t need for this project.

## Part 4 : Script
-  updates all the sources of the packages and create a scheduled task for this script once a week at 4AM and every time the machine reboots
-  monitor changes of the /etc/crontab file and send an email to root if it has been modified and create a scheduled script task every day at midnight.

## Part 5 : Web Server
Set a web server available on the VM’s IP or a host
