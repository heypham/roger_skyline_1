## Packages installed
```
apt install sudo
apt install net-tools (for ifconfig)
apt install vim
```
  
   
## Create a non-root user to connect to the machine and work (add user to sudoers)
```
usermod -aG sudo username
```

## Check if user can do sudo operations
```
sudo fdisk -l
```
