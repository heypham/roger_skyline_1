username : emilie

# 3. Partie Réseau et Sécurité

**Packages installed**
```
apt install sudo
apt install net-tools (for ifconfig)
apt install vim
```
  
   
## Vous devez créer un utilisateur non root pour vous connecter et travailler.
```
usermod -aG sudo username
```

## Utilisez sudo pour pouvoir, depuis cet utilisateur, effectuer les operations demandant des droits speciaux.
```
sudo fdisk -l
```

## Nous ne voulons pas que vous utilisiez le service DHCP de votre machine. A vous donc de la configurer afin qu’elle ait une IP fixe et un Netmask en /30.
  
Get IP address of VM
```
sudo ifconfig
```

*Pour utiliser le service static au lieu du service DHCP de la machine*
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

## Vous devez changer le port par defaut du service SSH par celui de votre choix. L’accès SSH DOIT se faire avec des publickeys. L’utilisateur root ne doit pas pouvoir se connecter en SSH.

  
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
  
Connect via SSH to the VM to copy the publickey into the VM publickeys file, then **from the machine (not VM)**
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

then restart the ssh service
```
sudo service sshd restart
```
  
## Vous devez mettre en place des règles de pare-feu (firewall) sur le serveur avec uniquement les services utilisés accessible en dehors de la VM.  
  
To do so we will use iptables.

*Installing iptables-persistent to make the rule change permanent*
```
sudo apt-get install iptables-persistent
```
  
*Create the folder containing the rules (ipv4 and ipv6)*
```
sudo service netfilter-persistent start
```

*Adding rules to the firewall*  
  
First, get the IP address of the machine (```ifconfig```, starting with *en*, or ```ip a```)  
  
1. allow all traffic for ssh port (because we want to be able to connect through ssh)
```
sudo iptables -A INPUT -p tcp -i enp0s3 --dport 42 -j ACCEPT
```
(to allow any port 42 connection)  
  
Then save it as a permanent change
```
sudo service netfilter-persistent save
```
  
We can also add rules directly to the files ```/etc/iptables/rules.v4``` or ```/etc/iptables/rules.v6```. I added the following rules
```
# SSH CONNECTION
-A INPUT -i enp0s3 -p tcp -m tcp --dport 42 -j ACCEPT

# WEB PORTS
-A INPUT -i enp0s3 -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -i enp0s3 -p tcp -m tcp --dport 443 -j ACCEPT

# DNS PORTS (tcp and udp)
-A INPUT -i enp0s3 -p tcp -m tcp --dport 53 -j ACCEPT
-A INPUT -i enp0s3 -p udp -m udp --dport 53 -j ACCEPT

# TIME PORT
-A INPUT -i enp0s3 -p udp -m udp --dport 123 -j ACCEPT
```
To save them directly from these files, we can use the command
```
sudo service netfilter-persistent reload
```
  
## Vous devez mettre en place une protection contre les DOS (Denial Of Service Attack) sur les ports ouverts de votre VM.
  
*Installing protection against DOS attacks on open ports*
```
sudo apt install fail2ban
```
  
*Copy the configuration file to a local one and configurate the protection*  
```
cd /etc/fail2ban
sudo cp jail.conf jail.local
sudo vim jail.local
```

in ```SSH SERVERS SECTION```  
replace all ```port = ssh``` by ```port = PORTNUMBER```

in ``` JAILS ``` section add the following  
```
# Block login attmepts
[apache]

enabled = true
port = http,https
filter = apache-auth
logpath = /var/log/apache2/*error.log
          /var/log/apache2/*errors.log
maxretry = 3
bantime = 600

# DOS protection
[apache-dos]

enabled = true
port = http,https
filter = apache-dos
logpath = /var/log/apache2/access.log
bantime = 600
maxretry = 300
findtime = 300
action = iptables[name=HTTP, port=http, protocol=tcp]
```

Create *apache-dos.conf* file in **filters.d** folder:
```
cd /etc/fail2ban/filters.d/
sudo touch apache-dos.conf
sudo vim apache-dos.conf
```
Add the following :
```
[Definition] 
failregex = ^<HOST> -.*"(GET|POST).*
ignoreregex =
```
Then enable **[apache dos]** in the file *defaults-debian.conf*
```
cd /etc/fail2ban/jail.d
sudo vim defaults-debian.conf

# ADD THESE LINES
[apache-dos]
enabled = true
```

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

then restart fail2ban service
```
sudo service fail2ban restart
```
  
## Protection contre les DDOS sur les ports ouverts de votre VM. (aka multi attaques)
  
**Using IPTABLES**  
https://javapipe.com/blog/iptables-ddos-protection/

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
#NOT THIS ONE SINCE OUR IP ADDRESS IS 10.12.1.112
# sudo iptables -t mangle -A PREROUTING -s 10.0.0.0/8 -j DROP 
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

*To test if working, use slowloris*
  
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
