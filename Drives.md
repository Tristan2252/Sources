# Drives
___________________________________________________________________________________________________________
## Find UUID of Drive  
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

## Hide Partition
*Source*: [askubuntu.com](http://askubuntu.com/questions/124094/how-to-hide-an-ntfs-partition-from-ubuntu)  
Create `hide-drives.rules` in `/etc/udev/rules.d/` and add `KERNEL=="sda1", ENV{UDISKS_IGNORE}="1"` for each dirve  
Load changes with `sudo udevadm control --reload-rules` and then `sudo udevadm trigger`  

## ZFS  
ZFS is very easy to configure and consists of to basic commands `zfs` and `zpool` to use.  
To create a basic zfs pool you can run the command `sudo zpool create [pool name] [device0] [device1]...`  
The default pool type of zfs is a dynamic stripe where the capacity of the devices are added to the pool and the
data written to them is striped across them. The drives are combined together within the pool as what is known as a vdev.
Vdev's can have the fallowing types: `disk` `file` `mirror` `raidz` `spare` `log` and `cache`. Pools created can be listed
using the command `sudo zpool list` as well as `zfs list`. The configuration and status of each drive within the pool can
be seen with `sudo zpool status`.

### My Configuration
**Goal**: In my server have 1 230GB 1 300GB and 1 4TB drives. I want to combine the storage capacity of the 230 and 300 GB
drives and have a spare drive in case a drive happens to fail.  

Firs I started out by creating a 540GB partition on the 4TB drive to be used as the spare drive. I figured 540GB because it is
roughly the size of the pool capacity. Then I created the pool using the command `zpool create file-pool /dev/sdb1 /dev/sdc1`,
`sdb1` and `sdc1` being the partitions on the 230 and 300GB drives. After the pool was created I ran `zpool add file-pool spare /dev/sda2`
witch added the 540GB partition of the 4TB drive to the pool as a spare. Running `sudo zpool status` shows the the configuration of the
pool:  
```
pool: file-pool
state: ONLINE
scan: none requested
config:

      NAME        STATE     READ WRITE CKSUM
      file-pool   ONLINE       0     0     0
        sdb1      ONLINE       0     0     0
        sdc1      ONLINE       0     0     0
      spares
        sda2      AVAIL

errors: No known data errors
```
After this the pool was ready to be used.
