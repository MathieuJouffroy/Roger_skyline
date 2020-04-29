## You have to set a protection against scans on your VM’s open ports.
https://wiki.debian-fr.xyz/Portsentry<br>
https://en-wiki.ikoula.com/en/To_protect_against_the_scan_of_ports_with_portsentry 
```
$ sudo apt install portsentry
$ sudo vim /etc/portsentry/portsentry.conf
```
Uncomment the first TCP and UDP lines (the highest protection)
Comment the second TCP and UDP lines (average protection)

replace
```TCP_MODE="tcp"``` by ```TCP_MODE="atcp"```
```UDP_MODE="udp"``` by ```UDP_MODE="audp"```

We also wish that portsentry is a blockage. We therefore need to activate it by passing BLOCK_UDP and BLOCK_TCP to 1.<br>
modify /etc/portsentry/portsentry.conf:
replace
```BLOCK_UDP="0"``` by ```BLOCK_UDP="1"```
```BLOCK_TCP="0"``` by ```BLOCK_TCP="1"```

We opt for a blocking of malicious persons through iptables. We will therefore comment on all lines of the configuration file that begin with KILL_ROUTE except this one:
```
KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
```
Verify:
```
$ cat portsentry.conf | grep KILL_ROUTE | grep -v "#"
```
Relaunch (it will now begin to block the port scans):
```
$ sudo /etc/init.d/portsentry start
```
portsentry logs are in the /var/log/syslog file.

<br>
To make sure our own IP address doesn't get banned add IP address in file below:
```
sudo vim /etc/portsentry/portsentry.ignore.static
```
then restart the portsentry service :
```
sudo service portsentry restart
```

*To check if portscan protection applied*

From a machine:
```
host terminal
nmap -Pn 10.0.2.15
```
*You should get kicked out from the VM if you were connected via ssh*

To deleting your IP address from the denied hosts file
```
sudo vim /etc/hosts.deny
```
Delete IP address of the machine from which you did the *nmap*<br>
you should have in the file : 
```
ALL: 10.0.2.14 : DENY
```
Then, we need to delete our ban from the iptables
```
sudo iptables -D INPUT 1
sudo iptables -D INPUT 1
```
then reboot
```
sudo reboot
```

## Stop the services you don’t need for this project.
```
$ sudo systemctl list-unit-files | grep enabled
```
or 
```
$ ls /etc/init.d
```
```
$ sudo systemctl disable bluetooth.service
$ sudo systemctl disable console-setup.service
$ sudo systemctl disable keyboard-setup.service
```

Mandatory :<br>
autovt.service #Necessary for using virtual terminals<br>
cron.service #Scheduled tasks<br>
fail2ban.service #Protection against DOS<br>
getty.service #Necessary for login<br>
networking.service #Network<br>
ssh.service #Needed for SSH connection<br>
sshd.service #Needed for SSH connection<br>
