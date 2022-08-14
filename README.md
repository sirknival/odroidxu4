# odroidxu4
 Everything about my NAS 

## Setup
Following Hardware is used:  
+ Odroid HC 1
+ Power Supply 5v/4A
+ Ethernet Cable 
+ SD-Card 16GB
+ SSD 500GB

## Resources
This guide is based on the following instuctions:  
[Installing OS and OMV](https://forum.odroid.com/viewtopic.php?t=28974#p206351 "odroid.org")  
[Move OS to SSD](https://forum.openmediavault.org/index.php?thread/28789-installing-omv5-on-raspberry-pi-s-armbian-sbc-s-i386-32-bit-platforms/ "forum.openmediavault.org")

## Write Image to SD-Card 
Download Armbian-Image and write it to the SD-Card eg. with Etcher    
[Armbian Images](https://www.armbian.com/odroid-hc1/#kernels-archive-all "armbian.org")
[Armbian HC1 Download](https://www.armbian.com/odroid-xu4/)

## First login
Open a CMD and type  
```ssh root@192.168.0.???```  
and enter the password ***1234***. Now you're forced to change the password. Next, it is required to create a new user.   

## Establish automatic ssh login
Create a file using ```nano .ssh/authorized_keys``` and paste the public key from your host computer into this file. You will find your rsa-public keys in the folder ```.ssh\id_rsa.pub``` 

## Update Armbian Config
For HC1 Armbian provides specially optimized config (for kernel 4.14.y or higher) which has to be applied manually. This results in shorter boot time and lower consumption. Run armbian-config utility and go to section system -> DTB and select optimized board configuration for Odroid HC1.

## Setting the SSD up
To create a partion for the os and a partition for all the data enter the command  
```fdisk /dev/sda``` 
To print the current partition table: ```p```    
To delte the current partition table: ```d```    
To create a new partition table: ```n```
To set the size of the new partition enter +[SIZE][UNIT] eg +20G    
To write the new partition table to the drive: ```w```  

Next, format the new created partitions using  
```mkfs.ext4 /dev/sda1``` and ```mkfs.ext4 /dev/sda2```  

Mount the partiton of the future system partiton   
```mount /dev/sda1 /mnt```  

Finally, copy all the existing files to the new partition  
```time tar --one-file-system -cpf - / | (cd /mnt && tar -xpf -)```  

## Defining the root filesystem
To use the SSD as our root file system, you need to tell the kernel the UUID of the SSD. Therefore, to get the UUID execute
```lsblk -f```  
Copy the UUID from the SSD (/dev/sda1).  
To update the boot.ini type
```nano /boot/armbianEnv.txt``` 
There enter the following line 
```rootdev=UUID=[NEUE UUID]``` where [NEUE UUID] is the one of the SSD.
Save using CTRL+X. Next, update the UUID in the fstab file using
 ```nano /etc/fstab```  
 In this file, simply replace the one and only UUID with the new one. Save and close. To test, where the changes where successful, just reboot if the system. If everything works, you're fine. To proof it, you can use the ```df``` command

## Updating the system
First update and upgrade the system using  
```sudo apt-get update```  
```sudo apt-get upgrade```  
`  

## Installing OMV 
### Current (OMV 6)
In Order to install OMV, simple execute the commands stated on this page:
[Instructions](https://forum.openmediavault.org/index.php?thread/39490-install-omv6-on-debian-11-bullseye/)

```cat <<EOF >> /etc/apt/sources.list.d/openmediavault.list
deb http://packages.openmediavault.org/public shaitan main
# deb http://downloads.sourceforge.net/project/openmediavault/packages shaitan main
## Uncomment the following line to add software from the proposed repository.
# deb http://packages.openmediavault.org/public shaitan-proposed main
# deb http://downloads.sourceforge.net/project/openmediavault/packages shaitan-proposed main
## This software is not part of OpenMediaVault, but is offered by third-party
## developers as a service to OpenMediaVault users.
# deb http://packages.openmediavault.org/public shaitan partner
# deb http://downloads.sourceforge.net/project/openmediavault/packages shaitan partner
EOF
```

```export LANG=C.UTF-8
export DEBIAN_FRONTEND=noninteractive
export APT_LISTCHANGES_FRONTEND=none
apt-get install --yes gnupg
wget -O "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc" https://packages.openmediavault.org/public/archive.key
apt-key add "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc"
apt-get update
apt-get --yes --auto-remove --show-upgraded \
    --allow-downgrades --allow-change-held-packages \
    --no-install-recommends \
    --option DPkg::Options::="--force-confdef" \
    --option DPkg::Options::="--force-confold" \
    install openmediavault-keyring openmediavault
omv-confdbadm populate
cat /etc/issue
```

### Legacy (OMV 5)
OMV can be installed using a preconfigured script. Excecute the command 
```wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash```  
This will download and install openmediavault 4.  
Note: During the last install, an error occured: Package ntp not installed. To fix this error simply type  
```sudo apt-get install ntp```

## Disable blue heartbeat LED
To disable the blue heartbeat led of the odroid HC1 board create an entry in ***rc.local***
```sudo nano /etc/rc.local```  
Paste the following line at the end of the file  
```echo none > /sys/class/leds/blue\:heartbeat/trigger```  


FINISHED!



