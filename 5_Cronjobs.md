## Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled task for this script once a week at 4AM and every time the machine reboots.
https://www.cyberciti.biz/faq/linux-show-what-cron-jobs-are-setup/<br>
https://phoenixnap.com/kb/how-to-list-display-view-all-cron-jobs-linux<br>
http://crontab.org/crontab.1.html<br>

```
$ sudo vim auto_update.sh
$ sudo chmod a+x auto_update.sh
```
in the file add:<br>
```
#!/bin/bash

date >> /var/log/update_script.log
sudo apt-get -y -q update >> /var/log/update_script.log
sudo apt-get -y -q upgrade >> /var/log/update_script.log
echo "" >> /var/log/update_script.log
```
edit current crontab (root):
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

## Make a script to monitor changes of the /etc/crontab file and sends an email to root if it has been modified. Create a scheduled script task every day at midnight.

```
$ sudo vim cron_changes.sh
$ sudo chmod a+x cron_changes.sh
```

in the file add:<br>
```
#!/bin/bash

sudo crontab -l > /etc/cron_status
diff /etc/cron_status /etc/cron_backups > /dev/null 2>&1
#echo "this is a test" | mail -s "test" root@localhost
# if last command not equal to 0
if [ $? -ne 0  ]
then
	echo "The crontab file has been modified\n" | mail -s "Crontab" root@localhost
	cp /etc/cron_status /etc/cron_backups
fi
```
