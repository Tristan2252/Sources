  
### Network
______________________________________________________________________________________________________  
##### Mount Samba Share  
  *Source*: [samba.org](https://wiki.samba.org/index.php/Mounting_samba_shares_from_a_unix_client)  
  *Source*: [manpages.ubuntu.com](http://manpages.ubuntu.com/manpages/precise/man8/mount.cifs.8.html)  
  *Source*: [wiki.ubuntu.com](https://wiki.ubuntu.com/MountWindowsSharesPermanently)  
  From the terminal: `mount -t cifs -o user=luke //192.168.1.104/share /mnt/linky_share`  
  To Mount at boot add `//192.168.2.100/share   /mnt/folder cifs  user=user,password=password 0 0` to fstab  
  * To make and use a password file for cifs add `username=user password=pass` to file and then use file with `credentials=filename` in fstab. `chmod 600 file` to prevent unwanted access.  
  * set `user` flag in fstab to allow non-root users to mount  
  
##### Display Adapter Speed
`ethtool eth0`

##### Change MTU  
*Source*:[cyberciti.biz](http://www.cyberciti.biz/faq/centos-rhel-redhat-fedora-debian-linux-mtu-size/)  
Syntax:`ifconfig ${Interface} mtu ${SIZE} up`  
Example: `ifconfig eth1 mtu 9000 up`  

##### PXE Setup
[maketecheasier](https://www.maketecheasier.com/configure-pxe-server-ubuntu/)  
Use [ubuntuforums](https://help.ubuntu.com/community/UbuntuLTSP/ProxyDHCP) for dnsmasq proxy config  
* Add Image to PXE Server
  - Add Ubuntu 14.04 Desktop Boot Images to PXE Server
  ```
  sudo mount -o loop /mnt/ubuntu-14.04.3-desktop-amd64.iso /media/
  sudo cp -r /media/* /var/lib/tftpboot/Ubuntu/14.04/amd64/
  sudo cp -r /media/.disk /var/lib/tftpboot/Ubuntu/14.04/amd64/
  sudo cp /media/casper/initrd.lz /media/casper/vmlinuz /var/lib/tftpboot/Ubuntu/
  ```
  - Configure NFS Server to Export ISO Contents
  ```
  sudo nano /etc/exports
    /var/lib/tftpboot/Ubuntu/14.04/amd64 *(ro,async,no_root_squash,no_subtree_check) # add line to exports
  sudo exportfs -a
  sudo /etc/init.d/nfs-kernel-server start
  ```  
  
##### Diskless Ubuntu Over PXE
[serenux](http://www.serenux.com/2011/04/howto-create-a-diskless-workstation-that-boots-from-pxe-using-ubuntu/)
```
sudo vim /etc/initramfs-tools/initramfs.conf
            MODULES=netboot
            BOOT=nfs

sudo mkinitramfs -o ./initrd.img
cp /boot/vmlinuz-`uname -r` ./vmlinuz

###### Be sure to add nfs mount folder to /etc/exports before mounting ######
sudo apt-get install nfs-common
mkdir /dev/shm/nfs
sudo mount -t nfs -nolock 192.168.0.10:/srv/nfs/disklessboot /dev/shm/nfs

sudo cp -avx /. /dev/shm/nfs/.
sudo cp -avx /dev/. /dev/shm/nfs/dev/.

sudo mkdir -p /srv/tftp/disklessboot
sudo cp /srv/nfs/disklessboot/home/USERNAME/vmlinuz /srv/tftp/disklessboot/
sudo cp /srv/nfs/disklessboot/home/USERNAME/initrd.img /srv/tftp/disklessboot/

sudo vim /etc/fstab
           proc            /proc           proc    defaults        0       0                                             
           /dev/nfs        /               nfs     defaults        1       1                                            
           none            /tmp            tmpfs   defaults        0       0                                             
           none            /var/run        tmpfs   defaults        0       0                                             
           none            /var/lock       tmpfs   defaults        0       0                                             
           none            /var/tmp        tmpfs   defaults        0       0                                             
           /dev/hdc        /media/cdrom0   udf,iso9660 user,noauto 0       0    
```  

##### List DHCP Leases
[thekelleys.org](http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2010q3/004384.html)  
`less /var/lib/misc/dnsmasq.leases`  

##### Disable Firmware Inherited Nic Names, 16.04  
*Source*: [askubuntu.com](http://askubuntu.com/questions/767786/changing-network-interfaces-name-ubuntu-16-04)  
Edit `/etc/default/grub` and change `GRUB_CMDLINE_LINUX=""` to `GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"`  
Save and run `sudo update-grub` and reboot.

