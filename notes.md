username : emilie
mdp : epham

3. Partie Réseau et Sécurité

Packages installed

apt install sudo
apt install net-tools (for ifconfig)
apt install vim

Vous devez créer un utilisateur non root pour vous connecter et travailler.
usermod -aG sudo username

Utilisez sudo pour pouvoir, depuis cet utilisateur, effectuer les operations demandant des droits speciaux.
sudo fdisk -l

Nous ne voulons pas que vous utilisiez le service DHCP de votre machine. A vous
donc de la configurer afin qu’elle ait une IP fixe et un Netmask en /30.
vim /etc/network/interfaces
replace dhcp by static
add these lines :
address 10.12.1.117/30 # IP address of VM
gateway 10.12.254.254 # Gateway address of VM
broadcast 10.12.255.255 # Broadcast address of VM

and sudo reboot

Vous devez changer le port par defaut du service SSH par celui de votre choix.
L’accès SSH DOIT se faire avec des publickeys. L’utilisateur root ne doit pas
pouvoir se connecter en SSH.

POUR POUVOIR SE CONNECTER EN SSH

sur le mac:
ssh-keygen
file in which to save key /home/emilie/.ssh/id_rsa

connect via SSH to the machine

On the mac :
ssh-copy-id -i id_rsa.pub emilie@10.12.1.117 -p 42

Check on machine via ssh that a new file authorized_keys has been created in .ssh/


sudo vim /etc/ssh/sshd_config

uncomment line # Port 22 and change the 22 as 42 (or any other number)
uncomment line # PermitRootLogin and replace restrict-password by "no" (pour que le root user ne puisse pas se connecter en SSH)

CES DEUX LIGNES DOIVENT ETRE CHANGEES EN MEME TEMPS
uncomment line # PubkeyAuthentication yes
uncomment line # PasswordAuthentication yes and replace yes by no

uncomment line # AuthorizedKeysFile and delete 



Vous devez mettre en place des règles de pare-feu (firewall) sur le serveur avec
uniquement les services utilisés accessible en dehors de la VM.

First, check with netstat -tpln to see which ports are open (should only be one port for ssh with 2 different protocols tcp and tcp6) will change later

NEXT
installing firewall
sudo apt-get install ufw

activate it with command
sudo ufw enable

adding rules
1. allow all traffic for ssh (because we want to be able to connect through ssh)
-> sudo ufw allow 42 (to allow any port 42 connection)




Vous devez mettre en place une protection contre les DOS (Denial Of Service
Attack) sur les ports ouverts de votre VM.

sudo apt install fail2ban

cd /etc/fail2ban
sudo vim jail.conf

in SSH SERVERS SECTION
replace all port = ssh by port = 42 (or any other port number depending on the one chosen)

to check if firewall rule applied

try to connect via ssh to the machine with wrong login or wrong password until blocked

to unblock user, go back to the machine
sudo fail2ban-client status sshd         to check that your ipaddress is in the banned section

sudo fail2ban-client set sshd unbanip your_ip_address




Vous devez mettre en place une protection contre les scans sur les ports ouverts
de votre VM.

sudo apt install portsentry

Configuration de portsentry
https://www.noobunbox.net/serveur/securite/installer-et-configurer-portsentry-debian-ubuntu

cd /etc/default
sudo vim portsentry

replace
TCP_MODE="tcp" by TCP_MODE="atcp"
UDP_MODE="udp" by UDP_MODE="audp"

cd /etc/portsentry/portsentry.conf
change
BLOCK_UDP="0"
BLOCK_TCP="0"

to 
BLOCK_UDP="1"
BLOCK_TCP="1"




Arretez les services dont vous n’avez pas besoin pour ce projet.

To check running services
sudo systemctl -l | grep running

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



