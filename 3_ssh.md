username : emilie

# 2. Network and security part

## We don’t want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.
  
Get IP address of VM
```
sudo ifconfig
```
*My ip address : 10.13.0.175*  
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
  
  
```
sudo service sshd restart
```
  
### Connect on your machine via SSH to the VM
```
ssh emilie@10.13.0.175 -p 24
```
  
*Generate a publickey to access VM via SSH*  
  
On your machine (not the VM)
```
ssh-keygen
```
file in which to save key ```/home/username/.ssh/id_rsa```  
  
Copy the publickey into the VM publickeys file, then **from the machine (not VM)**
```
cd .ssh/
ssh-copy-id -i id_rsa.pub emilie@10.13.0.175 -p 24
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
