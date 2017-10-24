# Window Manages

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


