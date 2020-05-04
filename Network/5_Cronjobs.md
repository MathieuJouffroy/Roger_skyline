## Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled task for this script once a week at 4AM and every time the machine reboots.
https://www.cyberciti.biz/faq/linux-show-what-cron-jobs-are-setup/ <br>
https://phoenixnap.com/kb/how-to-list-display-view-all-cron-jobs-linux
http://crontab.org/crontab.1.html

```
$ sudo vim auto_update.sh
```
in the file add:<br>
```
#!/bin/bash

date >> /var/log/update_script.log
sudo apt-get -y -q update >> /var/log/update_script.log
sudo apt-get -y -q upgrade >> /var/log/update_script.log
echo "" >> /var/log/update_script.log
```
edit current crontab:
```
$ sudo crontab -e
```
add:<br>
```
# Update every week at 4 am
0 4 * * 1 /home/mat/autoupdate.sh

# Update at every reboot
@reboot /home/mat/auto_update.sh

```
