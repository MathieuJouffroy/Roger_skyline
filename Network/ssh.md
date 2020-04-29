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
switch to new sudo user and verify sudo access
```
$ su - username
$ sudo whoami
```

## We don’t want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.

VirtualBox settings -> Network -> change ***NAT*** on ***Bridged Adapter***.
I will use ifconfig for the configuration (install net-tools and vim):
```
$ apt install net-tools
$ apt install vim
```

Get IP address and network information of VM:
```
$ sudo ifconfig
```
*My ip address: 10.0.2.15*

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
address     10.0.2.15       # IP address of VM
netmask     255.255.255.252 # Netmask /30
gateway     10.0.2.2        # Gateway address of VM
broadcast   10.0.2.255      # Broadcast address of VM
```
then restart the network service and verify:
```
$ sudo service networking restart
$ sudo ifconfig
```

## Change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.

Check status of ssh server:
```
$ sudo systemctl status ssh
```
*Change default port of SSH service to one of your choice* (try not using port numbers 0-1023 as they are reserved for various system services).

```
$ sudo vim /etc/ssh/sshd_config
uncomment line ```# Port 22``` and change the ```22``` as ```22222``` (or any other available port number)
uncomment line ```# PasswordAuthentication yes
$ sudo service sshd restart
```

### Connect on your machine via SSH to the VM

