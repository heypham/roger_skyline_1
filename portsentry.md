## Vous devez mettre en place une protection contre les scans sur les ports ouverts de votre VM.
  
*Installing protection against port scanning*
```
sudo apt install portsentry
```

*Configuration of portsentry*  
https://www.noobunbox.net/serveur/securite/installer-et-configurer-portsentry-debian-ubuntu  
```
cd /etc/default
sudo vim portsentry
```

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
nmap 10.12.1.117
```
*You should get kicked out from the VM if you were connected via ssh*

To unblock IP address, go back to the VM  
```
sudo iptables -L --line-numbers
```
Check the line that DROP the source of your machine
```
sudo iptables -D f2b-HTTP corresponding_line
```
Then also deleting your IP address from the denied hosts file
```
sudo vim /etc/hosts.deny
```
Delete IP address of the machine from which you did the *nmap*  

then restart portsentry service
```
sudo service portsentry restart
```
