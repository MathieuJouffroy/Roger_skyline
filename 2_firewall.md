## You have to set the rules of your firewall on your server only with the services used outside the VM.
https://opensource.com/article/18/9/linux-iptables-firewalld <br>
https://www.hostinger.com/tutorials/iptables-tutorial

I will use iptables.<br>

*Installing iptables-persistent to make the rule change permanent*
```
$ sudo apt-get install iptables-persistent
```

*Start the service, it should create the folder* **/etc/iptables/** *containing the rules files (ipv4 and ipv6)*
```
$ sudo service netfilter-persistent start
```
Check status of current iptables configuration:
```
$ sudo iptables -L -v # -L(list) -v(detailed format)
```

*Add rules to the firewall*

Allow all traffic for ssh port (because we want to be able to connect through ssh), here port 2222<br>
and save it as permanent:
```
$ sudo iptables -A INPUT -p tcp -i enp0s3 --dport 2222 -j ACCEPT
$ sudo service netfilter-persistent save
```
1) append this rule to the input chain (-A INPUT) so we look at incoming traffic
2) check to see if it is TCP (-p tcp). (Transmission Control Protocol)
3) check to see if the packet is coming on enp0s3
4) if so, check to see if the input goes to the SSH port (--dport ssh).
5) if so, accept the input (-j ACCEPT).

Add rules directly by editing to the files ```/etc/iptables/rules.v4``` or ```/etc/iptables/rules.v6```. I added the following rules
```
# SSH CONNECTION
-A INPUT -i enp0s3 -p tcp -m tcp --dport 2222 -j ACCEPT

# WEB PORTS (hhtp, https)
-A INPUT -i enp0s3 -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -i enp0s3 -p tcp -m tcp --dport 443 -j ACCEPT

# DNS PORTS (tcp and udp : user datagram protocol, port 53 : Domain name service)
-A INPUT -i enp0s3 -p tcp -m tcp --dport 53 -j ACCEPT
-A INPUT -i enp0s3 -p udp -m udp --dport 53 -j ACCEPT

# TIME PORT
-A INPUT -i enp0s3 -p udp -m udp --dport 123 -j ACCEPT
```

Save:
```
$ sudo service netfilter-persistent reload # or $ sudo service netfilter-persistent save
```
Verify:
```
$ sudo iptables -L -v  # or sudo iptables -S
```
Verify tcp port:<br>
netstat [address_family_options] [--tcp|-t] [--listening|-l] [--program|-p] [--numeric|-n] 
```
$ sudo netstat -tlpn # sudo netstat -tlpn | grep 2222
```
