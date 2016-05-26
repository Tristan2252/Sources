### Media
_________________________________________________________________________________________________________
##### Read Bluray Linux  
*Source*:[http://askubuntu.com](http://askubuntu.com/questions/565516/can-linux-play-blu-rays)  
```
sudo add-apt-repository ppa:heyarje/makemkv-beta
sudo apt-get update
sudo apt-get install makemkv-bin makemkv-oss

sudo apt-get remove libaacs0

cd /usr/lib
sudo ln -s libmmbd.so.0 libaacs.so.0
sudo ln -s libmmbd.so.0 libbdplus.so.0
```  


