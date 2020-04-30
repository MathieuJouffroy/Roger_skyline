## You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.
### You can also set DDOS(Distributed Denial of Service)
https://bobcares.com/blog/centos-ddos-protection/<br>
https://www.garron.me/en/go2linux/fail2ban-protect-web-server-http-dos-attack.html<br>
https://www.supinfo.com/articles/single/2660-proteger-votre-vps-apache-avec-fail2ban<br>

I will use Fail2Ban
```
$ sudo apt install fail2ban
```
Check status:<br>
```
$ sudo fail2ban-client status 
```
Fail2Ban keeps its configuration files in /etc/fail2ban folder. The configuration file is jail.conf(/etc/fail2ban/jail.conf).
This file can be modified by package upgrades so we will keep a copy of it as jail.local and edit it.
```
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$ sudo vim /etc/fail2ban/jail.local
```
in ```SSH SERVERS SECTION```
replace all ```port = ssh``` by ```port = 2222```
```
[sshd]

enabled		= true
port		= 2222
logpath		= %(sshd_log)s
backend		= %(sshd_backend)s
maxretry	= 3
bantime		= 10
```

in ``` JAILS ``` under ```HTTP servers``` add:
```
# Block login attempts
[apache]

enabled  = true
port     = http,https
filter   = apache-auth
logpath  = /var/log/apache2/*error.log
maxretry = 3
bantime  = 600
ignoreip = 10.0.2.15

# DOS protection
[apache-dos]

enabled  = true
port     = http,https
filter   = apache-dos
logpath  = /var/log/apache2/access.log
maxretry = 300
bantime  = 600
findtime = 300
action   = iptables[name=HTTP, port=http, protocol=tcp]
ignoreip = 10.0.2.15

# in sections [apache-badbots] [apache-noscript] [apache-overflows], add:

enabled  = true
filter   = section-name
logpath  = /var/log/apache2/*error.log
ignoreip = 10.0.2.15
```
Now we need to create the filter:
Create *apache-dos.conf* file in **/etc/fail2ban/filter.d** folder:
```
$ sudo vim  /etc/fail2ban/filters.d/apache-dos.conf
```
Add:
```
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*
ignoreregex =
```
Then enable **[apache dos]** in the file *defaults-debian.conf*
```
$ sudo vim /etc/fail2ban/jail.d/defaults-debian.conf

# ADD THESE LINES
[apache]
enabled = true

[apache-noscript]
enabled = true

[apache-overflows]
enabled = true

[apache-badbots]
enabled = true

[apache-dos]
enabled = true
```
And restart the service
```
sudo service fail2ban restart
```

### Verify firewall rules (list):
```
$ sudo fail2ban-client status 
```
edit /etc/ssh/sshd_config :
line ```# PubkeyAuthentication yes``` replace ```yess``` by ```no```
line ```# PasswordAuthentication no``` replace ```no``` by ```yess```
then restart the ssh service:
```
sudo service sshd restart
```
Try to connect via ssh to the machine with wrong login/password until blocked (IP from the machine that will be blocked).
To unblock IP address, in the VM verify if ipaddress is in banned section
```
$ sudo fail2ban-client status sshd
```
Unban ip address:
```
$ sudo fail2ban-client set sshd unbanip your_ip_address
```
Verify:
```
$ sudo fail2ban-client status sshd
```
Restart fail2ban:
```
$ sudo service fail2ban restart
```
re-edit /etc/ssh/sshd_config :
line ```# PubkeyAuthentication no``` replace ```no``` by ```yess```
line ```# PasswordAuthentication yess``` replace ```yess``` by ```no```
then restart the ssh service:
```
$ sudo service sshd restart
```


## DDOS protection on open ports of your VM. (aka multiple attacks)
I will use iptables
```
sudo apt-get install iptables-persistent
```

```
# BLOCKING INVALID PACKETS
sudo iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP

# BLOCKING NEW PACKETS THAT ARE NOT SYN
sudo iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP

# BLOCKING Packets With Bogus TCP Flags
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,RST FIN,RST -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,ACK FIN -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,URG URG -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,FIN FIN -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,PSH PSH -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL ALL -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL NONE -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL FIN,PSH,URG -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,FIN,PSH,URG -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP

# Block Packets From Private Subnets (Spoofing)
sudo iptables -t mangle -A PREROUTING -s 224.0.0.0/3 -j DROP
sudo iptables -t mangle -A PREROUTING -s 169.254.0.0/16 -j DROP
sudo iptables -t mangle -A PREROUTING -s 172.16.0.0/12 -j DROP
sudo iptables -t mangle -A PREROUTING -s 192.0.2.0/24 -j DROP
sudo iptables -t mangle -A PREROUTING -s 192.168.0.0/16 -j DROP
### sudo iptables -t mangle -A PREROUTING -s 10.0.0.0/8 -j DROP      # NOT this one as our IP is: 10.0.2.15
sudo iptables -t mangle -A PREROUTING -s 0.0.0.0/8 -j DROP
sudo iptables -t mangle -A PREROUTING -s 240.0.0.0/5 -j DROP
sudo iptables -t mangle -A PREROUTING -s 127.0.0.0/8 ! -i lo -j DROP
```

Save these rules as permanent
```
sudo service netfilter-persistent save
sudo service netfilter-persistent restart
```

then reboot
```
sudo reboot
```
You can use slowlowris to verify<br>
On your machine and when in network's available host:
```
host terminal
 
$ slowloris 10.0.2.15
```
To give back access to the machine that has been blocked :
```
$ sudo iptables -L # sudo iptables -S of see banned ip
$ sudo fail2ban-client set apache-dos unbanip your_ipaddress
$ sudo service fail2ban restart
```
