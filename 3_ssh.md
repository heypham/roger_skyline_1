username : emilie

# 2. Network and security part

## We don’t want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.
  
Get IP address of VM
```
sudo ifconfig
```

*Use static service instead of DHCP*
```
sudo vim /etc/network/interfaces
```
replace ```dhcp``` by ```static```  
add these lines :
```
address 		10.12.1.119   	# IP address of VM
gateway 		10.12.254.254 	# Gateway address of VM
broadcast 		10.12.255.255   # Broadcast address of VM
netmask                 255.255.255.252 # Netmask /30
```
then
```
sudo reboot
```

## You have to change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.

  
*Change default SSH port*
```
sudo vim /etc/ssh/sshd_config
```
uncomment line ```# Port 22``` and change the ```22``` as ```24``` (or any other available port number)  
uncomment line ```# PasswordAuthentication yes```

### Connect on your machine via SSH to the VM
```
ssh emilie@10.12.1.129 -p 24
```
  
*Generate a publickey to access VM via SSH*  
  
On your machine (not the VM)
```
ssh-keygen
```
file in which to save key ```/home/username/.ssh/id_rsa```  
  
Go to the VM (ssh connected) to copy the publickey into the VM publickeys file, then **from the machine (not VM)**
```
ssh-copy-id -i id_rsa.pub emilie@10.12.1.129 -p 24
```
  
**Check on VM that a new file** *authorized_keys* **has been created in folder** *.ssh/*  
  
*To forbid root to connect via SSH*  
```
sudo vim /etc/ssh/sshd_config
```
  
uncomment line ```# PermitRootLogin restrict-password``` and replace ```restrict-password``` by ```no```  
  
*To allow SSH access via publickeys ONLY*  

uncomment line ```# PubkeyAuthentication yes```  
line ```# PasswordAuthentication yes``` replace ```yes``` by ```no```  

then restart the ssh service
```
sudo service sshd restart
```
  

  

  


## Arrêtez les services dont vous n’avez pas besoin pour ce projet.

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
