## Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled task for this script once a week at 4AM and every time the machine reboots.

Since we need the cron task to come from the root, we will connect as root user
```
su
touch autoupdate.sh
sudo vim autoupdate.sh
```
  
Then, we need to create the log file named */var/log/update_script.log *
```
touch /var/log/update_script.log
```

Now, write the updating script **AND** to log the whold updating process in the logfile
```
#!/bin/bash

date >> /var/log/update_script.log                    # date of update
apt-get -y -q update >> /var/log/update_script.log
apt-get -y -q upgrade >> /var/log/update_script.log
echo "\n" >> /var/log/update_script.log               # \n between each update
```

Then, as root
```
crontab -e
```
At the end of the file, to set the update at 4 am every week :
```
# minute hour dayofmonth month dayofweek command
0 4 * * 1 /bin/sh autoupdate.sh
```

## Make a script to monitor changes of the /etc/crontab file and sends an email to root if it has been modified. Create a scheduled script task every day at midnight.


```
apt-get install mailutils
```

# /etc/aliases
mailer-daemon: postmaster
postmaster: root
nobody: root
hostmaster: root
usenet: root
news: root
webmaster: root
www: root
ftp: root
abuse: root
noc: root
security: root
root: root

reboot
