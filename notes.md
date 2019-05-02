username : emilie

## 3. Partie Réseau et Sécurité

**Packages installed**
```
apt install sudo
apt install net-tools (for ifconfig)
apt install vim
```

**Vous devez créer un utilisateur non root pour vous connecter et travailler.**
```
usermod -aG sudo username
```

**Utilisez sudo pour pouvoir, depuis cet utilisateur, effectuer les operations demandant des droits speciaux.**
```
sudo fdisk -l
```

**Nous ne voulons pas que vous utilisiez le service DHCP de votre machine. A vous
donc de la configurer afin qu’elle ait une IP fixe et un Netmask en /30.**

*Pour utiliser le service static au lieu du service DHCP de la machine*
```
sudo vim /etc/network/interfaces
```
replace ```dhcp``` by ```static```  
add these lines :
```
address 		10.12.1.117/30 	# IP address of VM
gateway 		10.12.254.254 	# Gateway address of VM
broadcast 		10.12.255.255 	# Broadcast address of VM
```
then
```
sudo reboot
```

**Vous devez changer le port par defaut du service SSH par celui de votre choix.
L’accès SSH DOIT se faire avec des publickeys. L’utilisateur root ne doit pas
pouvoir se connecter en SSH.**  

  
*Pour changer le port par défault du service SSH*
```
sudo vim /etc/ssh/sshd_config
```
uncomment line ```# Port 22``` and change the ```22``` as ```42``` (or any other available port number)  
  
*Générer une publickey pour avoir accès en SSH à la VM une fois la configuration d'accès en SSH via publickeys terminée*  
  
Sur la machine depuis laquelle vous voulez vous connecter en SSH
```
ssh-keygen
```
file in which to save key ```/home/username/.ssh/id_rsa```  
  
Connect via SSH to the VM to copy the publickey into the VM publickeys file
```
ssh-copy-id -i id_rsa.pub username@ipaddress -p PORTNUMBER
```
  
Check on VM that a new file ```authorized_keys``` has been created in ```.ssh/```  
  
*Pour empêcher l'utilisateur root de se connecter en SSH*  
```
sudo vim /etc/ssh/sshd_config
```
  
uncomment line ```# PermitRootLogin restrict-password``` and replace ```restrict-password``` by ```no```  
  
*Pour permettre l'accès en SSH avec des publickeys uniquement*  

uncomment line ```# PubkeyAuthentication yes```  
uncomment line ```# PasswordAuthentication yes``` and replace ```yes``` by ```no```  
  
**Vous devez mettre en place des règles de pare-feu (firewall) sur le serveur avec
uniquement les services utilisés accessible en dehors de la VM.**  
  
First, check with ```netstat -tpln``` to see which ports are open (should only be one port for ssh with 2 different protocols *tcp* and *tcp6*, will be more later)  

*Installing the firewall*
```
sudo apt-get install ufw
```
  
*Activate the firewall*
```
sudo ufw enable
```

*Adding rules to the firewall*  
  
1. allow all traffic for ssh port (because we want to be able to connect through ssh)
```
sudo ufw allow PORTNUMBER
```
(to allow any port 42 connection)  
  
**Vous devez mettre en place une protection contre les DOS (Denial Of Service
Attack) sur les ports ouverts de votre VM.**
  
*Installing protection against DOS attacks on open ports*
```
sudo apt install fail2ban
```
  
*Configurate the protection*  
```
cd /etc/fail2ban
sudo vim jail.conf
```

in ```SSH SERVERS SECTION```  
replace all ```port = ssh``` by ```port = PORTNUMBER```


*To check if firewall rule applied*  
  
Try to connect via ssh to the machine with wrong login/password until blocked  
*Note* : it is not the user that is blocked but the IP address of the machine from which the wrong user tried to log. Therefore, even a valid user will not be able to log via ssh to the VM from this IP address.

To unblock IP address, go back to the VM  
```
sudo fail2ban-client status sshd
```
to check that your ipaddress is in the banned section  

```
sudo fail2ban-client set sshd unbanip your_ip_address
```  
  
**Vous devez mettre en place une protection contre les scans sur les ports ouverts
de votre VM.**
  
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
cd /etc/portsentry/portsentry.conf
```
replace  
```BLOCK_UDP="0"``` by ```BLOCK_UDP="1"```
```BLOCK_TCP="0"``` by ```BLOCK_TCP="1"```


**Arrêtez les services dont vous n’avez pas besoin pour ce projet.**

*First check running services*
```
sudo systemctl -l | grep running
```
  
Here are all the running services :
```
cron.service                 loaded active running   Regular background program processing daemon 		# scheduling scripts/tasks in future or regularly
dbus.service                 loaded active running   D-Bus System Message Bus 							# allow multiple users on same machine
fail2ban.service             loaded active running   Fail2Ban Service 									# protection against DOS attack
getty@tty1.service           loaded active running   Getty on tty1 										# terminal + commands services
portsentry.service           loaded active running   LSB: # start and stop portsentry 					# protection against portscans
rsyslog.service              loaded active running   System Logging Service 							# saving all log messages over an IP network
ssh.service                  loaded active running   OpenBSD Secure Shell server 						# ssh
systemd-journald.service     loaded active running   Journal Service 									# collects and stores logging data in indexed journals (kernel, stout, errors)
systemd-logind.service       loaded active running   Login Service 										# manages user login
systemd-timesyncd.service    loaded active running   Network Time Synchronization 						# Sync local system clock with remote network time protocol server
systemd-udevd.service        loaded active running   udev Kernel Device Manager 						# listens to kernel uevents and executes matching instructions specified in udev rules
user@1000.service            loaded active running   User Manager for UID 1000           				# me
```
