## You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.
  
https://www.supinfo.com/articles/single/2660-proteger-votre-vps-apache-avec-fail2ban  
  
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
replace all ```port = ssh``` by ```port = 24```

in ``` JAILS ``` under ```HTTP servers``` section add the following  
```
# Block login attempts
[apache]

enabled  = true
port     = http,https
filter   = apache-auth
logpath  = /var/log/apache2/*error.log
maxretry = 3
bantime  = 600
ignoreip = 10.13.1.175

# DOS protection
[apache-dos]

enabled  = true
port     = http,https
filter   = apache-dos
logpath  = /var/log/apache2/access.log
bantime  = 600
maxretry = 300
findtime = 300
action   = iptables[name=HTTP, port=http, protocol=tcp]
ignoreip = 10.13.1.175

# ADD THE FOLLOWING LINES TO THE SECTIONS [apache-badbots] [apache-noscript] [apache-overflows]

enabled  = true
filter   = section-name
logpath  = /var/log/apache2/*error.log
ignoreip = 10.13.1.175
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
And restart the service
```
sudo service fail2ban restart
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
  
## DDOS protection on open ports of your VM. (aka multiple attacks)
  
**Using IPTABLES**  
https://javapipe.com/blog/iptables-ddos-protection/

Installing service that will make rule changes permanent
```
sudo apt-get install iptables-persistent
```

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

To give back access to the machine that has been blocked
```
sudo iptables -L --line-numbers

# LOOK FOR THE REJECT line (f2b-HTTP section)

sudo iptables -D f2b-HTTP 1
sudo reboot
```
