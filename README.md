# roger_skyline_1

## 1. VM installation in VirtualBox

Check in Preferences that the Default Machine Folder is */sgoinfre/goinfre/Perso/epham/VM*

**VM settings :**
- in Storage : make sure Controller: IDE has debian-9.9.0 disk
- in Network : Attached to *Bridged Adapter*, Name : *en0: ethernet*

**VM installation**  
  
Hostname :
```
roger
```
Leave Domain name empty  
Follow the steps to create new user

**Partitioning requirements :**  
- Disk size of 8 Go.
- At least one partition of 4.2 Go.
- The VM should also be up to date with all the packages necessary for the project installed.

Partitioning method :  
```
Manual
```

Select a partition to modify its settings :
```
SCSI1 (0, 0, 0) (sda) 8.6 GB ATA VBOX HARDDISK
```

Create new empty partition table on this device : **<*Yes*>**
Select a partition to modify its settings :
```
pri/log 8.6 GB FREE SPACE
```
Select *Create a new partition*  
  
New partition size :  
```
4.5 GB
```
  
Type for the new partition :
```
Primary
```

Select *Done setting up the partition*

Select a partition to modify its settings :
```
pri/log 4.1 GB FREE SPACE
```
Select *Create a new partition*  
  
New partition size :  
```
4.1 GB
```
  
Type for the new partition :
```
Logical
```

Select *Finish partitioning and write changes to disk*

Do you want to return to the partitioning menu ? **<*No*>**  
Write the changes to disks ? **<*Yes*>**
Scan another CD or DVD ? **<*No*>**
  
Debian archive mirror country
```
France
ftp.fr.debian.org
```

Leave *HTTP proxy information* blank

Participate in the package usage survey ? **<*No*>**  

Software to install :
- [ ] Debian desktop environment
- [ ] ... GNOME
- [ ] ... Xfce
- [ ] ... KDE
- [ ] ... Cinnamon
- [ ] ... MATE
- [ ] ... LXDE
- [ ] web server
- [ ] print server
- [x] SSH server
- [x] Standard system utilities

Install GRUB boot loader to the master boot record ? **<*Yes*>**  
Device for boot loader installation
```
/dev/sda (ata-VBOX_HARDDISK)
```
