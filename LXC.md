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

