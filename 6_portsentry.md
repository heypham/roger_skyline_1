## You have to set a protection against scans on your VM’s open ports.
  
*Installing protection against port scanning*
```
sudo apt install portsentry
```

*Configuration of portsentry*  
https://wiki.debian-fr.xyz/Portsentry
```
cd /etc/portsentry
sudo vim portsentry.conf
```
  
Uncomment the first TCP and UDP lines (the highest protection)
Comment the second TCP and UDP lines (average protection)

replace  
```TCP_MODE="tcp"``` by ```TCP_MODE="atcp"```
```UDP_MODE="udp"``` by ```UDP_MODE="audp"```
  
then
```
sudo vim /etc/portsentry/portsentry.conf
```
replace  
```BLOCK_UDP="0"``` by ```BLOCK_UDP="1"```
```BLOCK_TCP="0"``` by ```BLOCK_TCP="1"```
  
Under **EXTERNAL COMMANDS**
```
KILL_RUN_CMD="/sbin/iptables -I INPUT -s $TARGET$ -j DROP && /sbin/iptables -I INPUT -s $TARGET$ -m limit --limit 3/minute --limit-burst 5 -j LOG --log-level debug --log-prefix 'Portsentry: dropping: '"
```
  
To make sure our own IP address doesn't get banned :
```
sudo vim /etc/portsentry/portsentry.ignore.static
```
Add your inet and IP addresses

then restart the portsentry service :  
```
sudo service portsentry restart
```
  
*To check if portscan protection applied*  
  
From a machine, try
```
nmap -Pn 10.12.1.140
```
*You should get kicked out from the VM if you were connected via ssh*

To deleting your IP address from the denied hosts file
```
sudo vim /etc/hosts.deny
```
Delete IP address of the machine from which you did the *nmap*  

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

To do so, we are going to do a linked clone of our VM. And disable the services on the cloned VM to see which services are needed for the project.

1. Log out , close and turn off your VM
2. Right click in VirtualBox on *Clone...*
3. *Continue* and tick *linked clone*

**Then, open the cloned VM**

*Check running services*
```
sudo systemctl list-units -t service
```
or
```
sudo systemctl list-unit-files --state=enabled
```
or
```
sudo service --status-all
```
  
Here are all the running services :
```
UNIT FILE                    STATE
autovt@.service              enabled
console-setup.service        enabled
cron.service                 enabled
fail2ban.service             enabled
getty@.service               enabled
keyboard-setup.service       enabled
netfilter-persistent.service enabled
networking.service           enabled
rsyslog.service              enabled
ssh.service                  enabled
sshd.service                 enabled
syslog.service               enabled
systemd-timesyncd.service    enabled
remote-fs.target             enabled
apt-daily-upgrade.timer      enabled
apt-daily.timer              enabled
```
  
All services listed above are useful, since they are the default services available.
  
To disable a service :  
```
sudo systemctl disable service_name
```
