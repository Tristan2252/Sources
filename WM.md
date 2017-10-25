# Window Managesp

## Better Version of i3lock

The current version of `i3lock` is nice but primitive. It does not allow for customization.
`Lixxia` version of `i3lock` can be used to fix this issue. His git repo is located [here](https://github.com/Lixxia/i3lock)

* After making the file it can be installed with `sudo checkinstall make install`.
* Then a systemd service needs to be created for allow for launching `i3lock` upon suspend
    - to do this `cd` into `/etc/systemd/system/` and create a `lock.service` file
    - place in the file the following;
    ```                                                                                                          
    [Unit]                                                                                                           
    Description=Lock the screen automatically after a timeout                                                        
    After=suspend.target                                                                                             

    [Service]                                                                                                        
    Type=forking                                                                                                     
    User=tristan                                                                                                     
    Environment=DISPLAY=:0                                                                                           
    ExecStart=/usr/bin/i3lock -i /home/tristan/.lockpic.png -c '#000000' -o '#191d0f' -w '#572020' -l '#ffffff' -e  
    ExecStop=/home/tristan/.dotfiles/scripts/lockpic.sh                                                              

    [Install]
    WantedBy=suspend.target
    ```

    - then run `systemctl daemon-reload` and `systemctl enable lock.service` to init the new service

## i3bar Powerline Config
These are the steps I went through to configure a Powerline style i3bar with real time status and 
dynamic icons.

### Conky
Conky is a vary useful tool to help with getting system status such as CPU, RAM, DISK, and even battery 
usage. Conky is the tool that I used to obtain almost all system statistics.

* Conky has a single config file that exists at `$HOME/.conkyrc` or can be loaded using `-c`
* The config file has two types of syntax. It is good to use the new version for better readability as well 
as to prevent from conky writing to `stderr` about using the old config syntax
* One of the most important settings within to config file is the `out_to_console` and `update_interval`
    - `out_to_console` prints the conky stats to `stdout` rather than to a conky window. This is important 
       my config because later I read conkys out put to format for i3bar.
    - `update_interval` is important because this is the amount of time conky will wait before refreshing 
       its output. The Value of `update_interval` needs to be `1.0` in order to allow for close to real time 
       status updates
