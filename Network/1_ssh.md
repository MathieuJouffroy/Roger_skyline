### Before creating your VM:<br>
VirtualBox settings -> Network -> change ***NAT*** on ***Bridged Adapter***. This will be useful for setting a static IP and Netmask \30 with the host machine.

## Create non-root user to connect to machine
install `sudo` (for root privileges):
```
$ apt install sudo
```
login as root and add user to sudo group:
```
$ ssh root@server_ip_address
$ usermod -aG sudo username
```
switch to new sudo user and verify sudo access:
```
$ su - username
$ sudo whoami
```

## We don’t want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.
http://droptips.com/cidr-subnet-masks-and-usable-ip-addresses-quick-reference-guide-cheat-sheet<br>
https://ressourcesinformatiques.com/article.php?article=6121<br>
The /30 signifies that the network part of the address is 30 bits long, leaving 2 bits for the host part which equals 4 available addresses.<br>
On a /30 network you have 4 IPs available. Minus off 2 IPs for network ID and broadcast and it leaves you with 2 usable IPs.

I will use ifconfig for the configuration (install net-tools and vim):
```
$ apt install net-tools
$ apt install vim
```

Get IP address and network information of VM:
```
$ sudo ifconfig
```
*ip address: 192.168.1.74*

We can see that our `bridged adapter` is ***enp0s3***. 
*Setup static service instead of DHCP*
```
$ sudo vim /etc/network/interfaces
```
replace ```dhcp``` by ```static```
add these lines :
```
# The primary network interface
auto enp0s3
allow-hotplug enp0s3
iface enp0s3 inet static
address     192.168.1.74    # VM IP address
netmask     255.255.255.252 # Netmask /30
gateway     192.168.1.254   # VM Gateway address
broadcast   192.168.1.255   # VM Broadcast address
```
Restart the network service and verify:
```
$ sudo service networking restart
$ sudo ifconfig
$ sudo ip a | grep -A2 enp0s3:
$ sudo ip r
$ ping 8.8.8.8
```

## Change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.

Check status of ssh server:
```
$ sudo systemctl status ssh
```
*Change default port of SSH service to one of your choice* (try not using port numbers 0-1023 as they are reserved for various system services).

```
$ sudo vim /etc/ssh/sshd_config
```
uncomment line ```# Port 22``` and change the ```22``` as ```22222``` (or any other available port number).<br>
uncomment line ```# PasswordAuthentication yes```.
```
$ sudo service sshd restart
```
login with ssh and check status of our connection:
```
$ sudo ssh mat@192.168.1.74 -p 2222
$ sudo systemctl status ssh
```
### Connect via SSH to the VM on your machine
To connect 2 interfaces they must be in one subnet<br>
On the VM 2 ip adresses are allowed (netmask /30): 192.168.1.74 (ip addr that we set) and 192.168.1.73 (for host).<br>
Set up the ip addr to the host:<br>
***System Preferences*** -> ***Network*** -> ***Advanced*** -> ***TCP/IP*** -> ***Configure IPv4*** -> ***Using DHCP with manual address*** -> ***Enter the new ip addr (192.168.1.73)*** -> ***Apply***.
```
# host terminal

$ ping 192.168.1.74
$ ssh mat@192.168.1.74 -p 2222
$ exit (logout from the ssh)
```
#### SSH access HAS TO be done with publickeys.
Test the ssh conection from host.<br>
We need to setup SSH public key authentication to access the VM via SSH from machine (host terminal)
```
# host terminal

$ ssh-keygen
```
Copy the publickey (host) into the VM publickeys file:
```
# host terminal

$ cd .ssh/ && ssh-copy-id -i id_rsa.pub mat@192.168.1.74 -p 2222
```
**Verify on VM that the new file** *authorized_keys* **has been created in folder** *.ssh/*

#### SSH root access SHOULD NOT be allowed directly, but with a user who can be root.
*To forbid root to connect via SSH*
```
$ sudo vim /etc/ssh/sshd_config
```
uncomment line ```# PermitRootLogin restrict-password``` and replace ```restrict-password``` by ```no```

*To allow SSH access via publickeys ONLY*

uncomment line ```# PubkeyAuthentication yes```<br>
line ```# PasswordAuthentication yes``` replace ```yes``` by ```no```
<br>
Restart the ssh service
```
$ sudo service sshd restart
```
