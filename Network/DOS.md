## You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.
### You can also set DDOS(Distributed Denial of Service)
I will use Fail2Ban
```
$ sudo apt install fail2ban
```
Fail2Ban keeps its configuration files in /etc/fail2ban folder. The configuration file is jail.conf(/etc/fail2ban/jail.conf).
This file can be modified by package upgrades so we will keep a copy of it as jail.local and edit it.
```
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$ sudo vim /etc/fail2ban/fail2ban.local
```
