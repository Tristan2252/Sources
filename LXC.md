# LXC
_________________________________________________________________________________________________________  

## Create Unprivileged Container
*Source*: [gist.github.com/julianlam](https://gist.github.com/julianlam/4e2bd91d8dedee21ca6f)  
*Source*: [Linux Sysadmin Basics](https://www.youtube.com/watch?v=SPk7EL1jja4)

* Create lxcuser account:
```
$ sudo adduser lxcuser
$ usermod --add-subuids 100000-165536 lxcuser
$ usermod --add-subgids 100000-165536 lxcuser
$ chmod +x /home/lxcuser
```  
* Log into lxcuser and create config file:
```
> mkdir -p .config/lxc/
> vim .config/lxc/default.conf
    lxc.network.type = veth
    lxc.network.link = lxcbr0
    lxc.network.flags = up
    # lxc.network.hwaddr = 00:16:3e:xx:xx:xx
    lxc.id_map = u 0 100000 65536 # OR adjust for autocreated values in /etc/subuid
    lxc.id_map = g 0 100000 65536 # same as above for /etc/subgid
```  
* Logount of lxcuser and add `lxcuser veth lxcbr0 10` to `/etc/lxc/lxc-usernet` then restart  
* Loginto lxcuser (not over ssh) and run `lxc-create -t download -n [CONTAINER NAME] -- -d ubuntu -r trusty -a amd64`


## LXD

### Creating an LXD container:  

To create a container make sure LXD is installed with `sudo apt-get install lxd`  
Run command `lxc launch ubuntu:<release#> <name of container>`  
With the ubuntu release option you can use `ubuntu:` to automatically obtain the latest release of ubuntu  

### LXD Usage commands:  

Use `lxc list` to list all the launched containers  
`lxc exec <container> -- command` will run the command as the root user of the container,  
This method can also be used to attach to the container by executing `lxc exec <container> -- /bin/bash`  
You can also pull and push files to and from the container with the commands `lxc <file> pull/push source/dest`  
Stoping containers is done by `lxc stop <container>` and deleting with `lxc delete <container>`  

**Some More Advanced Commands**  
To edit the config file of the container use `lxc config edit <container>`. This will launch the container config
in your default terminal text editor. To change the default terminal editor see [this page](https://github.com/Tristan2252/Sources/blob/master/System.md#change-default-terminal-editor).
To show an expanded view of the container config use `lxc config show --expanded <container>` Some conf options found [here](https://github.com/Tristan2252/Sources/). To add a disk device to the container use the command  
`lxc config device add <container> <device name> <type> path=</path/to/device> source=</path/of/source>`.  
For example to add a shared folder a command such as  
`lxc config device add File-Server shared-folder disc path=/mnt/share source=/home/server/share`  
This command will add a device of type *disk* found on the host at */home/server/share* and attached to the guest at */mnt/share*.  
See "Adding ZFS Pool" for more details on sharing folders.  

### My LXD configuration  
**Goal**: To create an lxc container to host a samba server for file sharing on the network. The shared files will be stored on a ZFS pool
located on the host. The samba server will need access to the storage pool on the host and the `rw` ability to it. See more on ZFS
[here](https://github.com/Tristan2252/Sources/blob/master/Drives.md#zfs)  

**Container Setup**  
To Start, run `lxc launch ubuntu: File-Server` to create the container and then `lxc start File-Server` to start it. The container needs
access to the LAN in order to share files to the clients on it, therefore we need to create a bridge for LXC to connect with the LAN.
To do so, create a [bridge interface](https://github.com/Tristan2252/Sources/blob/master/Network.md#create-bridge-interface) in `/etc/network/interfaces` and run
`lxc config edit <container>` to add your newly created bridge interface to the container config file.
 Configuration for a network device looks like the fallowing:
```
eth0:
  name: eth0
  nictype: bridged
  parent: br0
  type: nic
```
Since i will no longer be using the default lxcbr0 that was created with the install of LXD, I updated the `/etc/default/lxd-bridge`
to let lxc know that i dont want to use lxcbr0 or to add it to newly launched containers. To do this open `/etc/default/lxd-bridge` and
change the value of `USE_LXD_BRIDGE=` to false. Lastly we need to setup the container's network interface. Attach to the container with
`lxc exec File-Server -- /bin/bash` and edit the network config file located in `/etc/network/interfaces.d/eth0.cfg` with a static ip
for the File-Server. Test for network connectivity and install samba with `sudo apt-get install samba`. Lastly set the container to auto
start at boot with `lxc config set container_name boot.autostart 1` or in the config as the fallowing:  
```
config:
  boot.autostart: "1"
```

**Adding ZFS Pool**  
Next we need to configure the ZFS storage pool, instructions on how to do so are found [here](https://github.com/Tristan2252/Sources/blob/master/Drives.md#zfs).
Once the storage pool is created we need to add it to the container config. This is done with the `device add` command like so:  
`lxc config device add File-Server shared-folder disc path=/mnt/share-pool source=/share-pool`  
The device can also be manually added to the config file by running `lxc config edit container` adding the fallowing lines:  
```
shared-folder:
  path: /mnt/share-pool/
  source: /share-pool/
  type: disk
```
Keep in mind this is essentially adding the folder where the device is mounted on the host, not the device itself. In the case of a zfs
pool this seems to be ok because the pool is represented by its folder on the host system.
Now that the device is added we need to get the permissions set correctly so the the clients as well as the server have `rw` access to
the shared storage pool. This requires us to change the owner of the pool to those of the user in the container. A simple `chmod 777`
could fix this issue but it would be insecure due to the `rwx` permissions of everyone. To change the owner of the pool to the
container user we first need to find the uid of the user. Running `ls -l` on the *rootfs* folder located at
`/var/lib/lxd/containers/<container>/rootfs/` will give us the uid number we need. This number can be found in the 3rd column of the output.
```
$ ls -l /var/lib/lxd/containers/File-Server/rootfs/

total 84
drwxr-xr-x  2 165536 165536 4096 Jul 14 14:46 bin
drwxr-xr-x  3 165536 165536 4096 Jul 14 14:48 boot
drwxr-xr-x  6 165536 165536 4096 Jul 14 14:48 dev
drwxr-xr-x 90 165536 165536 4096 Jul 17 19:28 etc
drwxr-xr-x  5 165536 165536 4096 Jul 16 13:15 home
                        .
                        .
                        .
```
NOTE: `ls` needs to be ran as root because no other users have viewing access to */var/lib/lxd/containers/*.  
With the uid of the container user we can now set the owner of the storage pool on the host as the uid we found. This is done with
`chown -R uid:gid /path/to/zfs-pool/` in my case `chown -R 165536:165536 /share-pool/`. The `-R` flag is used to recessively set the
permissions in case there are any perviously stored files within the pool. The container user now has `rw` permissions and can be tested
by attaching to the container and touching a file to the device we added earlier. NOTE: the host user will not be abel to write to the
storage pool now that the permissions are changed, but in my case there is no need to. If access is need by the host, it can be done by
mounting the samba share and writing that way. All that is left now is to setup the samba share and start storing files.

### Sources
* [LXD 2.0](http://insights.ubuntu.com/2016/03/14/the-lxd-2-0-story-prologue/)  
* [LXD 2.0: Your first LXD container](https://insights.ubuntu.com/2016/03/22/lxd-2-0-your-first-lxd-container/)  
* [LXD Networking](https://wiki.archlinux.org/index.php/LXD)  
* [Installing LXD and the command line too](https://linuxcontainers.org/lxd/getting-started-cli/)  
* [LXD 2.0: Blog post series](https://www.stgraber.org/2016/03/11/lxd-2-0-blog-post-series-012/)  
* [Adding a shared host directory to an LXC/LXD Container](http://askubuntu.com/questions/691039/adding-a-shared-host-directory-to-an-lxc-lxd-container)
8 [Autostarting LXD containers](https://bitsandslices.wordpress.com/2015/08/26/autostarting-lxd-containers/)
