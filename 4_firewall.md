## You have to set the rules of your firewall on your server only with the services used outside the VM.
  
To do so we will use iptables.

*Installing iptables-persistent to make the rule change permanent*
```
sudo apt-get install iptables-persistent
```
  
*Start the service, it should create the folder* **/etc/iptables/** *containing the rules files (ipv4 and ipv6)*
```
sudo service netfilter-persistent start
```

*Adding rules to the firewall*  
  
First, get the IP address of the machine (```ifconfig```, starting with *en*, or ```ip a```)  
  
1. allow all traffic for ssh port (because we want to be able to connect through ssh)
```
sudo iptables -A INPUT -p tcp -i enp0s3 --dport 24 -j ACCEPT
```
(to allow any port 24 connection)  
  
Then save it as a permanent change
```
sudo service netfilter-persistent save
```
  
We can also add rules directly by editing to the files ```/etc/iptables/rules.v4``` or ```/etc/iptables/rules.v6```. I added the following rules
```
# SSH CONNECTION
-A INPUT -i enp0s3 -p tcp -m tcp --dport 24 -j ACCEPT

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
