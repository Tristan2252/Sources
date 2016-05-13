### Latex
__________________________________________________________________________________________
##### System of Equations
[wikibooks](https://en.wikibooks.org/wiki/LaTeX/Advanced_Mathematics)
```
f(x) = \begin{dcases*}
          a           & if n = 1\\
          f(x/2) + b  & if n > 2
       \end(dcases*}
```
#####  Mathematical Symbols
[List of LaTeX Methematical Symbols](https://oeis.org/wiki/List_of_LaTeX_mathematical_symbols)  
Percent `\%`  

##### Commands with parameters  
*Source*:[sharelatex.com](https://www.sharelatex.com/learn/Commands)  
{\Command Name}[parameter number]{\Command{#parameter number}  
`\newcommand{\bb}[1]{\mathbb{#1}}`  

##### Import PDF into Latex
*Source*:[stackexchange.com](http://tex.stackexchange.com/questions/105589/insert-pdf-file-in-latex-document)  
NOTE: There are options to only add slected pages  
```
\usepackage[final]{pdfpages}
\begin{document}
          \includepdf[pages=-]{file.pdf}
\end{document}
```  

### System
__________________________________________________________________________________________
##### Shell Scripts and Cron Jobs
*Source*:[help.ubuntu.com](https://help.ubuntu.com/lts/serverguide/backup-shellscripts.html)  
Edit Cron config file `sudo crontab -e`  
Config file syntax: `m h dom mon dow   command`  
- *m*: minute the command executes on, between 0 and 59.
- *h*: hour the command executes on, between 0 and 23.
- *dom*: day of month the command executes on.
- *mon*: the month the command executes on, between 1 and 12.
- *dow*: the day of the week the command executes on, between 0 and 7. Sunday may be specified by using 0 or 7, both values are valid.
- *command*: the command to execute.  
Example:
```
# m h dom mon dow   command
0 0 * * * bash /usr/local/bin/backup.sh
```  
##### Grub Password Reset  
*Source*[inuxconfig.org](https://linuxconfig.org/recover-reset-forgotten-linux-root-password)  
Open GRUB and select kernel.  
Then press `e` to edit. Remove `ro quiet splash` and replace with `rw init=/bin/bash`. Then boot kernel  
Use `passwd [USER]` to change password  

##### XFCE Window Compositing  
*Source* [manpages.ubuntu.com](http://manpages.ubuntu.com/manpages/trusty/man1/compton.1.html)  
Config file located in `~/.config/compton.conf`, see link for options.  
  
### KVM  
_________________________________________________________________________________________
##### List VM's
[virt-tools.org](http://virt-tools.org/learning/start-list-with-command-line/)  
`list --all`  

##### Create disk img  
[raymii.org](https://raymii.org/s/tutorials/KVM_add_disk_image_or_swap_image_to_virtual_machine_with_virsh.html#Create_and_attach_the_disk_image)  
`qemu-img create -f raw example-vm-swap.img 1G`  

##### Disk Resize  
[serverfault](http://serverfault.com/questions/324281/how-do-you-increase-a-kvm-guests-disk-space)  
`qemu-img resize vmdisk.img +10G`

##### Mount CD media
[ndchost.com](https://www.ndchost.com/wiki/libvirt/change-media)  
`virsh # change-media vm1 hdc /var/lib/libvirt/images/cd2.iso --insert`

##### Ejecting the existing media
[ndchost.com](https://www.ndchost.com/wiki/libvirt/change-media)  
```
virsh # domblklist vm1
Target     Source
------------------------------------------------
hda        /dev/storage1/vm1_disk1
hdc        /var/lib/libvirt/images/cd1.iso
```
`virsh # change-media vm1 hdc --eject`

### Drives
___________________________________________________________________________________________________________
##### Find UUID of Drive  
[stackexchange](http://unix.stackexchange.com/questions/658/linux-how-can-i-view-all-uuids-for-all-available-disks-on-my-system)  
`ls -al /dev/disk/by-uuid`  
Output:
```
lrwxrwxrwx 1 root root  10 Mar 14 11:40 05b30c9f-eefb-4f5f-9536-6957563ca7c7 -> ../../sda1
lrwxrwxrwx 1 root root  10 Mar 14 11:40 0B2952FC6BC88D0E -> ../../sdd1
lrwxrwxrwx 1 root root  10 Mar 14 11:40 1160a5cc-442c-4019-9c61-907481d9afbd -> ../../sde1
lrwxrwxrwx 1 root root  10 Mar 14 11:40 520889705D4C8F8A -> ../../sdc1
lrwxrwxrwx 1 root root  10 Mar 14 11:40 6C90399C90396DA8 -> ../../sdb1
lrwxrwxrwx 1 root root  10 Mar 14 11:40 f26e1bbc-1612-49a6-aea6-6445ee87344c -> ../../sda5
```
Then use `lsblk` to determine the disk you are looking for: 
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 232.9G  0 disk
├─sda1   8:1    0 212.9G  0 part /
├─sda2   8:2    0     1K  0 part
└─sda5   8:5    0    20G  0 part [SWAP]
sdb      8:16   0 232.9G  0 disk
└─sdb1   8:17   0 232.9G  0 part /mnt/FileDrive
sdc      8:32   0 298.1G  0 disk
└─sdc1   8:33   0 298.1G  0 part /mnt/MediaDrive
sdd      8:48   0 298.1G  0 disk
└─sdd1   8:49   0 298.1G  0 part /mnt/VmDrive
sde      8:64   0 465.8G  0 disk
└─sde1   8:65   0 465.8G  0 part /mnt/BackupDrive
```  

##### Hide Partition
*Source*: [askubuntu.com](http://askubuntu.com/questions/124094/how-to-hide-an-ntfs-partition-from-ubuntu)  
Create `hide-drives.rules` in `/etc/udev/rules.d/` and add `KERNEL=="sda1", ENV{UDISKS_IGNORE}="1"` for each dirve  
Load changes with `sudo udevadm control --reload-rules` and then `sudo udevadm trigger`  
  
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

### Media
_________________________________________________________________________________________________________
##### Read Bluray Linux  
*Source*:[http://askubuntu.com](http://askubuntu.com/questions/565516/can-linux-play-blu-rays)  
```
sudo add-apt-repository ppa:heyarje/makemkv-beta
sudo apt-get update
sudo apt-get install makemkv-bin makemkv-oss

sudo apt-get remove libaacs0

cd /usr/lib
sudo ln -s libmmbd.so.0 libaacs.so.0
sudo ln -s libmmbd.so.0 libbdplus.so.0
```  

