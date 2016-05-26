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
 
##### DPKG Error with Corrupt or Missing package  
`sudo vim /var/lib/dpkg/status` remove entries of Corrupt package  
`sudo apt-get update` refresh dpkg  
`sudo apt-get install -f` run fix on dpkg  
  
##### System Integrity Protection in OS X El Capitan  
*Sources* 
[Create a OS X El Capitan Boot Installer](http://osxdaily.com/2015/09/30/create-os-x-el-capitan-boot-install-drive/), 
[Disable System Integrity Protection](http://osxdaily.com/2015/10/05/disable-rootless-system-integrity-protection-mac-os-x/)  
  
First create a OS X recovery usb using the fallowing command  
`sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/ElCapInstaller --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app`  
  
Boot into Recovery USB by holding `Option Key` on boot and selecting the usb.  
After boot go to menu and open terminal app.  
Then run `csrutil disable; reboot`  

After reboot into original OS use `csrutil status` to check status of System Integrity  


