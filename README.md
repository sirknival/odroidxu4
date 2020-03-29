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

## First login
Open a CMD and type  
```ssh root@192.168.0.???```  
and enter the password ***1234***. Now you're forced to change the password. Next, it is required to create a new user.   

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



