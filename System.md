## System
__________________________________________________________________________________________
## Shell Scripts and Cron Jobs
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
## Grub Password Reset  
*Source*[inuxconfig.org](https://linuxconfig.org/recover-reset-forgotten-linux-root-password)  
Open GRUB and select kernel.  
Then press `e` to edit. Remove `ro quiet splash` and replace with `rw init=/bin/bash`. Then boot kernel  
Use `passwd [USER]` to change password  

## XFCE Window Compositing  
*Source* [manpages.ubuntu.com](http://manpages.ubuntu.com/manpages/trusty/man1/compton.1.html)  
Config file located in `~/.config/compton.conf`, see link for options.  

## DPKG Error with Corrupt or Missing package  
`sudo vim /var/lib/dpkg/status` remove entries of Corrupt package  
`sudo apt-get update` refresh dpkg  
`sudo apt-get install -f` run fix on dpkg  

## System Integrity Protection in OS X El Capitan  
*Sources*
[Create a OS X El Capitan Boot Installer](http://osxdaily.com/2015/09/30/create-os-x-el-capitan-boot-install-drive/),
[Disable System Integrity Protection](http://osxdaily.com/2015/10/05/disable-rootless-system-integrity-protection-mac-os-x/)  

First create a OS X recovery usb using the fallowing command  
`sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/ElCapInstaller --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app`  

Boot into Recovery USB by holding `Option Key` on boot and selecting the usb.  
After boot go to menu and open terminal app.  
Then run `csrutil disable; reboot`  

After reboot into original OS use `csrutil status` to check status of System Integrity  

## SSH Port Forwarding  

Command: `ssh -L [LOCAL PORT]:localhost:[REMOTE PORT] user@hostip`  

## Powerline Theming  
Copy files in `/usr/local/lib/python2.7/powerline/config_files/` to `~/.config/powerline`  
Any changed made to files in the directory will take affect to powerline  

## Change Default Terminal Editor  
*Source*: [howtogeek.com](http://www.howtogeek.com/howto/ubuntu/change-the-default-editor-from-nano-on-ubuntu-linux/)  

Ubuntu comes with nano as the default terminal editor, for those who use vim this may cause some issues or inconveniences when editing
config files. For example running the `lxc config` command opens nano to edit the container conf. More about LXC [here](https://github.com/Tristan2252/Sources/blob/master/LXC.md#lxd). In ubuntu based systems the default terminal editor can be changed
with the command `sudo update-alternatives --config editor`. You will be prompted with a menu with different editors numbered in a list.
```
There are 4 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /bin/nano            40        auto mode
  1            /bin/ed             -100       manual mode
  2            /bin/nano            40        manual mode
  3            /usr/bin/vim.basic   30        manual mode
  4            /usr/bin/vim.tiny    10        manual mode

Press <enter> to keep the current choice[*], or type selection number:
```
Simply type in the number of your editor and the default will be changed!  

## Using dd to create large file  
*Source*: [How do I create a 1GB random file in Linux?](http://superuser.com/questions/470949/how-do-i-create-a-1gb-random-file-in-linux)

Sometimes you may need to create a large file for test purposes, such as testing drive configurations in zfs. See more about ZFS [here](https://github.com/Tristan2252/Sources/blob/master/Drives.md#zfs). The `dd` command is very useful for this type of task. You
can run the fallowing command to create a file of random data to test with:
```
dd if=/dev/urandom of=sample.txt bs=64M count=16
```
You can also set the input file `if` to `/dev/zero` to fill a file with zeros. The `bs` flag is the amount of data you want to write at a time while the `count` flag is the amount of times you want to write the `bs` to the file. In the above example `64M` is being written `16` times resulting in a file size of `1024` or `1G`.

## Create SSH Key  
*Source*: [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)  

The most secure way to remote to a unix machine is through ssh with kay authentication. Not only does this provide security but a more
convenient way of logging in by not requiring a password upon authentication. To get started with creating and SSH key, log into the client
machine. To generate a kay for the client run `ssh-keygen -t rsa` as the user you plan to use ssh with. Next we need to tell the SSH server
(the computer we a connecting to) what our ssh key is. You can send the key over ssh with the command `ssh-copy-id user@server.ip`. Be sure
to use the same user you plan to connect to with ssh. Lastly test the connection with `ssh user@server.ip`.  

**More Info on SSH keys**  
The ssh keys are located in the home directory of the user they were generated on within the `.ssh/` subdir. You will find both `id_rsa` and
`id_rsa.pub` keys. The `id_rsa.pub` is the public key, the key to be shared with the servers you are connecting to. The `id_rsa` is the private
key witch is to be kept withing the `.ssh/` directory and not shared with anyone. During the ssh authentication process the server checks its
public key against that of the clients private key, if they are from the same client the authentication is successful.

## Run Custom Script at Startup
*Source*: [How to run a command when boots up](http://www.cyberciti.biz/tips/linux-how-to-run-a-command-when-boots-up.html)  

Some time it is convenient to write a startup script to mount a drive or accomplish some other task on boot. After writing the script move the
executable to `/etc/init.d/` where the init scripts are located. Then run `sudo update-rc.d mystartup.sh defaults 100` to tell linux when and
how to start the script. The argument `defaults` refers to the default runlevels, which are 2 through 5. Number 100 means script will get executed
before any script containing number 101. Just run the command `ls -al /etc/rc3.d/` and you will see all script soft linked to `/etc/init.d` with numbers.  
