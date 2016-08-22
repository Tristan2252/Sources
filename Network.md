
# Network
______________________________________________________________________________________________________  
## Mount Samba Share  
  *Source*: [samba.org](https://wiki.samba.org/index.php/Mounting_samba_shares_from_a_unix_client)  
  *Source*: [manpages.ubuntu.com](http://manpages.ubuntu.com/manpages/precise/man8/mount.cifs.8.html)  
  *Source*: [wiki.ubuntu.com](https://wiki.ubuntu.com/MountWindowsSharesPermanently)  
  From the terminal: `mount -t cifs -o user=luke //192.168.1.104/share /mnt/linky_share`  
  To Mount at boot add `//192.168.2.100/share   /mnt/folder cifs  user=user,password=password 0 0` to fstab  
  * To make and use a password file for cifs add `username=user password=pass` to file and then use file with `credentials=filename` in fstab. `chmod 600 file` to prevent unwanted access.  
  * set `user` flag in fstab to allow non-root users to mount  

## Display Adapter Speed
`ethtool eth0`

## Change MTU  
*Source*:[cyberciti.biz](http://www.cyberciti.biz/faq/centos-rhel-redhat-fedora-debian-linux-mtu-size/)  
Syntax:`ifconfig ${Interface} mtu ${SIZE} up`  
Example: `ifconfig eth1 mtu 9000 up`  

## PXE Setup
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

## Diskless Ubuntu Over PXE
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

## List DHCP Leases
[thekelleys.org](http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2010q3/004384.html)  
`less /var/lib/misc/dnsmasq.leases`  

## Disable Firmware Inherited Nic Names, 16.04  
*Source*: [askubuntu.com](http://askubuntu.com/questions/767786/changing-network-interfaces-name-ubuntu-16-04)  
Edit `/etc/default/grub` and change `GRUB_CMDLINE_LINUX=""` to `GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"`  
Save and run `sudo update-grub` and reboot.

## Create Bridge Interface  
*Source*: [manpages.ubuntu.com](http://manpages.ubuntu.com/manpages/precise/man5/bridge-utils-interfaces.5.html)  

First step in creating a bridge interface is to install bridge-utils with `sudo apt-get install bridge-utils`.  
Next we need to set up the interface by opening the network config file located at `/etc/network/interfaces`.
Basic bridge configuration looks like the fallowing:
```
auto br0
iface br0 inet static
        address
        network
        gateway
        broadcast
        netmask
        dns-nameservers
        bridge_ports eth1 eth2 eth3...
        bridge_stp off
        bridge_fd 0
        bridge_maxwait 0
```
The firs bridge parameter in the configuration is `bridge_ports` witch allows us to set the ports we would like
to bridge, these can be any of the ports/interfaces within the network configuration. Next is `bridge_stp` known
as the Spanning Tree Protocol witch is used for configuring redundancy within a multi-switch network. If multiple
bridges are on the same network this option would be set to on to be able to configure STP. `bridge_fd` allows
configuration of the bridge forward delay in seconds. This will pause the forwarding packets for x seconds when
going through the bridge. This should be set to 0. Lastly in this config is `bridge_maxwait`. This value in seconds
defines the time the stat up script should wait for the included ports to receive an UP status.

## Iptables Redirect Port Forward  
*Source*: [IPTables - Port to another ip & port](http://unix.stackexchange.com/questions/76300/iptables-port-to-another-ip-port-from-the-inside)

If you have an internal server running on a custom port that you would like the accept traffic from the internet to on its
services default port, you can use iptalbes to do so. An example of this would be if you have an ssh server on your LAN running
on port 2252 and you wold like to access it from the WAN using a default ssh client config. In terms of ports, internet facing
port `22` needs to be redirected to port `2252` at the ssh server LAN ip. To achieve this the configuration would look like the fallowing
```
nat*
-A PREROUTING -i $WAN -p tcp --dport 22 -j DNAT --to $LAN_IP:2252
-A POSTROUTING -p tcp -d $LAN_IP --dport 2252 -j MASQUERADE

```

## Finding original MAC address  
*Source*: [Finding original MAC address from Hardware itself](http://stackoverflow.com/questions/14955504/finding-original-mac-address-from-hardware-itself)  

When a bridge is setup on linux the bridge mac is set as the mac of its interface ports. If you need to get the true mac address of the interface
for boot on lan or some other reason you can cat the interface info files using `cat /sys/class/net/eth0/address`. If you have `ethtool` installed
you can also use `ethtool -P eth0`.
