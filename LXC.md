### LXC
_________________________________________________________________________________________________________  

##### Create Unprivileged Container
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


### LXD

##### Creating an LXD container:  

To create a container make sure LXD is installed with `sudo apt-get install lxd`  
Run command `lxc launch ubuntu:<release#> <name of container>`  
With the ubuntu release option you can use `ubuntu:` to automatically obtain the latest release of ubuntu  

##### LXD Usage commands:  

Use `lxc list` to list all the launched containers  
`lxc exec <container> -- command` will run the command as the root user of the container,  
This method can also be used to attach to the container by executing `lxc exec <container> -- /bin/bash`  
You can also pull and push files to and from the container with the commands `lxc <file> pull/push source/dest`  
Stoping containers is done by `lxc stop <container>` and deleting with `lxc delete <container>`  

**Some More Advanced Commands**  
To edit the config file of the container use `lxc config edit <container>`. This will launch the container config
in your default terminal text editor. To change the default terminal editor see [this page](https://github.com/Tristan2252/Sources/).
To show an expanded view of the container config use `lxc config show --expanded <container>` Some conf options found [here](https://github.com/Tristan2252/Sources/). To add a disk device to the container use the command  
`lxc config device add <container> <device name> <type> path=</path/to/device> source=</path/of/source>`.  
For example to add a shared folder a command such as  
`lxc config device add File-Server shared-folder disc path=/mnt/share source=/home/server/share`  
This command will add a device of type *disk* found on the host at */home/server/share* and attached to the guest at */mnt/share*.  
See [this page](https://github.com/Tristan2252/Sources) for more details on sharing folders.  

**My LXD configuration**  
**Goal**: To create an lxc container to host a samba server for file storage on the network. The shared files are stored on a ZFS pool
located on the host. The samba server would require access to the storage pool on the host and the `rw` ability to it. See more on ZFS
[here](https://github.com/Tristan2252/Sources)  
To Start, run `lxc launch ubuntu: File-Server` to create the container and then `lxc start File-Server` to start it. The container needs
access to the LAN in order to share files to clients on it, therefore we need to bridge the lxc network with the LAN. To do so create
a [bridge interface](https://github.com/Tristan2252/Sources) and edit `/etc/default/lxd-bridge` to let lxc know this is the interface
you want to use. In to config file update values for `LXD_BRIDGE=` and `LXD_IPV4_ADDR=`. I have experienced issues with the bride interface
if the `LXD_IPV4_ADDR` is not set, so be sure to set its value to the ip of the host bridge interface. If using DHCP it may work with no value.
Also set `LXD_IPV4_NAT` to false so that the server can be accessed directly from the LAN.
