

# Preperation of the machine  

Download Ubuntu 24.04:    
https://ubuntu.com/download/desktop     

You need Rufus to install it:  
https://rufus.ie/en/   

With that you need a USB Key 16 Go: to install the Linux partition.   
https://www.amazon.com.be/s?k=16go+usb+key  

You need a 512GB 2.5" SATA SSD crucial:   
https://www.amazon.com.be/s?k=512GB+2.5%22+SATA+SSD+crucial  

NVMe disk if you have:   
https://www.amazon.com.be/s?k=NVMe+disk  

An old but decent PC 64 Bits Linux:  
My is AMD FX8320Ex8 , 32 Gb  
With NVIDIA GeForce GTX 1650  
Question: Does Graphic card is needed for server >  
   
To do for the guide:  
What is the worst PC you can use for a wow server ?  


------------

# Some app to install before starting  

- Visual Studio  : to edit text https://code.visualstudio.com/docs/setup/linux  
- TeamVeiwer : to seek help https://www.teamviewer.com/en/download/linux/    
- Git (already install by Linux)   

All is doable by Terminal but be able to see folder and edit by hand files for beginner is usefull.  


-----------------------

# The man, the guy ;)    

Amro Yasir — [https://www.fiverr.com/amroyasir](https://www.fiverr.com/amroyasir)   

I wanted to learn how to set up a server…  
But not my first one, and only if FFA Hardcore was possible.  
After confirming this, he agreed to install it on a local PC that I prepared for him.  
I took notes on what he did in order to build this tutorial.  
Thanks to him for pulling it off.  

Now it is installed, I need to learn to do it by myself on an other computer.    
Before I do that, I writing down the step and vocabulary to learn.  

----------------------------

# Rewatch of the replay  

Log as a user.  
```
sudo -i
```

```
sudo apt-get update && sudo apt-get install git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev
```


```
mysql_secure_installatiin
```


```
cmake ../ -DCMAKE_INSTALL_PREFIX=/home/gurubashiaccount/server_files/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=all -DSCRIPTS=static -DMODULES=static
```



Used during installing Wow server.
```
sudo apt-get update && sudo apt-get install git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev
sudo -i
ip addr show
sudo git clone https://github.com/azerothcore/azerothcore-wotlk.git --branch master --single-branch /home/server_files/
ls -l
cd server_files/
pwd
sudo mv /home/server_files/ /home/gurubashiaccount/server_files/
sudo rm -r server_files/
cd modules/
sudo git clone https://github.com/azerothcore/mod-ale.git
mkdir build && cd build
nproc --all
make -j 8
sudo make install
cp authserver.conf.dist authserver.conf
cp worldserver.conf.dist worldserver.conf
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine64 wine32
mysql
wget https://github.com/wowgaming/client-data/releases/download/v18.0/Data.zip
unzip Data.zip 
rm -r Data.zip 
sudo apt install nodejs
sudo apt install npm
sudo npm install pm2 -g
sudo apt install tmux
tmux new -s auth
tmux new -s world
tmux attach-session -t auth
tmux attach-session -t world
sudo ufw status
sudo apt install net-tools
ifconfig
unzip HeidiSQL_12.13_64_Portable.zip -d heidisql
wine heidisql.exe
reboot
hostname -I
./worldserver
```








