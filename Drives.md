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

**Advanced ZFS Commands**  
* **Snapshots**: Snapshots are a saved state of the zfs pool. If any data is list of corrupt a snapshot can be used to restore
the pool to a working state. It is good to run snapshots as frequent as backups but be prepared for them to take up space. Snapshots
are stored within the file system of the pool under the directory `poolname/.zfs/poolname@snapname`. Initially snapshots dont take up
any space until the filesystem is modified. When this occurs the snapshot size grows with the size of the changed made to the file system.
To create a snapshot run the command `sudo zfs snapshot poolname/dataset_name@snapname`. Snapshots are commonly used with datasets but can
also be used to snapshot the entire pool as well. To do so use the command `sudo zfs snapshot -r poolname@sanpname`. To list all snapshots
as well as the space they are taking up use the command `sudo zfs list -t snapshot`. If you are having trouble finding the snapshot dir, it
may not be set to visible. To set the sanpshot to visible use the command `sudo zfs set snapdir=visible zroot/usr`.  

* **Managing ZFS Properties**: Each zpool has properties that can be changed via the command `sudo zfs set key=value poolname/dataset`. You
can view the value of a key with the command `sudo zfs get key poolname/dataset`. To get a list of all the properties the pool has use the
command `sudo get all poolname/dataset`. To see documentation on ZFS properties visit this page [here](http://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html)   

### My Configuration
**Goal**: In my server have 1 230GB 1 300GB and 1 4TB drives. I want to combine the storage capacity of the 230 and 300 GB
drives and have a hot spare drive in case a drive happens to fail. This should give the functionality of that of a RAID 5
but with the performance of a RAID 0. (RAID 0 With a hot spare)

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

###Potential Issues When Using ZFS:
**Hot Spare**:    
The hot spare configuration is currently not supported on ubuntu 16.04, see ongoing issue
[here](https://github.com/zfsonlinux/zfs/issues/4358#issuecomment-233812603) as well as my superuser.com post [here](http://superuser.com/questions/1102939/zfs-hot-spare-not-working)  

**Replacing Drives / Expanding Storage**:  
The best way to add capacity to the pool is to add more `vdev's`. If this is not an option drives within `mirror` and
`raidz` vdev's can be replaced or upgraded via a `resilver -> replace -> resilver` process where one drive is upgraded
and rebuilt by the raid at a time.  

**Drive changed sda name**:  
If a drive's sda name is changed due to an change in partitions or drives the pool will no longer be detected by the OS.
In order to fix this you need to import the pool. Start by runing `zpool import` it should be able to find the previously
built pool on what was `/dev/sdaX` and is now `/dev/sdaY`. You will more then likely get an output like the fallowing:  
```
# zpool import

  pool: dozer
  id: 2704475622193776801
  state: ONLINE
  action: The pool can be imported using its name or numeric identifier.
  config:
    dozer       ONLINE
      c1t9d0    ONLINE
```  
I you get this first attempt importing by pool ID with `zpool import 2704475622193776801`.
If this is unsuccessful you may have to rename the pool by importing it as a different name with the command `zpool import old_name new_name`. See [this post](http://superuser.com/questions/1106503/expanding-size-of-virtual-hard-drive-zfs) and [Importing ZFS Storage Pools](http://docs.oracle.com/cd/E19253-01/819-5461/gazuf/index.html) for more details.

### Sources
* [Becoming a ZFS Ninja](https://www.youtube.com/watch?v=tPsV_8k-aVU)  
* [ZFS Cheatsheet](http://www.datadisk.co.uk/html_docs/sun/sun_zfs_cs.htm)
* [Managing ZFS Storage Pool Properties](http://docs.oracle.com/cd/E19253-01/819-5461/gfifk/index.html)
* [ZFS RAID levels](http://www.zfsbuild.com/2010/05/26/zfs-raid-levels/)

## Mount .img as a Loop device
*Source*: [How can I mount a disk image?](http://superuser.com/questions/344899/how-can-i-mount-a-disk-image)  

In the event that you would like to grab some files off of a .img disk image the best and easiest way to do so is by using `kpartx`.  
When you run `kpartx -a -v myimage.disk`, `kpartx` automatically scans the disk for the offset sectors and adds the drive as a loop
device located in `/dev/mapper/loop0px`. After this is done it can be easily mounted via `mount /dev/mapper/loop0p1 /mnt/myimage`.
If `kpartx` does not get the job done look into `losetup`, although i did not have much luck with it.
