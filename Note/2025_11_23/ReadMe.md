
My best local computer is this one in hope that suffice
<img width="690" height="477" alt="image" src="https://github.com/user-attachments/assets/10e349a7-dc47-4832-82d4-1c504c5222fb" />


Does the Steam Machine could handle a local server ?  
[<img width="1061" height="557" alt="image" src="https://github.com/user-attachments/assets/e7d9cab0-a40b-4aaa-9e26-31007c519a64" />](https://store.steampowered.com/sale/steammachine)  
https://store.steampowered.com/sale/steammachine  





```
http://192.168.1.1/
```
Go to admin panel and open port:
Port to open on network: 3306 8085 3724

<img width="1837" height="159" alt="image" src="https://github.com/user-attachments/assets/30292e6d-0af4-4c01-9744-731ccf7aee5f" />



Keyword:
`sudo -i`
`CMake`
`mysql.conf.d`
`mysql_secure_installation`
`ls -l` `pwd`
`mv /Server_files/* /home/gurubashiaccount/server_files/`
`sudo rm  -r server_files/`
`sudo mkdir server_files/`

https://github.com/azerothcore/azerothcore-wotlk  
```
sudo git clone https://github.com/azerothcore/azerothcore-wotlk.git --branch master --single-branch /home/server_files/
```

```
cd modules/
cd ..
mkdir build && cd build
```


https://github.com/azerothcore/mod-ale  
```
sudo git clone https://github.com/azerothcore/mod-ale.git
```
Need:
- `AzerothCore server installation`
- `Git version control system`
- `CMake build system`

https://cmake.org/ is a C++ tool on Linux   


`sudo make install`


`/server_files/server_source/build`


Client: https://www.warmane.com/download
Download Warth of Lich King:
https://www.bittorrent.com/downloads/complete/classic



cp authserver.config.dist
and others


[heidisql](https://www.heidisql.com/download.php)  
https://www.heidisql.com/download.php    
Git https://github.com/HeidiSQL/HeidiSQL  
https://github.com/HeidiSQL/HeidiSQL/releases/tag/v12.13.0.7147  

  
```sudo dpkg --add-architecture i386```
```sudo apt update```


sudo apt install unzip

unzip KeidiSal...
wine heidisql.exe


new session name 

sudo  -i 
mysql

crestion session name server_localhost in keidasal


FLUSH PRIVILEGES


create user_admin with password

systemctl status mysql


unwip data downloaded with wget.

worldserver/config   
DataDir = "/jome_server files core files data something"   
LogsDir = "path of logs file"  

lua_script/extensions
- Download HardCore.lua and drop it in lua_scripts of core_filesbin




worldserver.conf
GameType = 16

Rate.XP.kill =8
Rate.XP.Quest =8
Rate.XP.Explore =8
...


sudo sudo apt update

```sudo apt install nodejs```
```sudo apt install npm```

```sudo npm install pm -g```
```sudo apt install tmux```

./authserver
tmux new -s 
```sudo ./worldserver```


AC . account create admin 1134566
set admin to 3

go int data base and do something on it. account_access

ral


!!! change realmlist as name and public address in deb heidisql


use portchecker.com    
to check that port are open   

ufw status
sudo ufw to open port
sudo ufw  3724


Check that the port ar open

tmux attach-session
server re 1

```
account create amroyasir test@noame
account crated
```








if on proximus change the worldserver.config 
to sue your modem operator.

we used
3700 instead of 8085
3724





.reload ale

sudo pm2 start worldserver



```
echo "Can be done without thinking"
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev nodejs npm tmux net-tools wine64 wine32 unzip

sudo -i
sudo npm install pm2 -g
sudo dpkg --add-architecture i386
wget https://github.com/wowgaming/client-data/releases/download/v18.0/Data.zip
wget https://github.com/HeidiSQL/HeidiSQL/releases/download/v12.13.0.7147/HeidiSQL_12.13_64_Portable.zip

sudo mkdir "$HOME/server_files"
sudo mkdir -p "$HOME/server_files/zip/heidisql"
sudo mkdir -p "$HOME/server_files/zip/wow_data"

sudo unzip -o "HeidiSQL_12.13_64_Portable.zip" -d "$HOME/server_files/zip/heidisql"
sudo unzip -o "Data.zip" -d "$HOME/server_files/zip/wow_data"





# Clone AzerothCore source
sudo git clone https://github.com/azerothcore/azerothcore-wotlk.git --branch master --single-branch "$HOME/server_files/git/source_code"
sudo git clone https://github.com/azerothcore/mod-ale.git "$HOME/server_files/git/source_code_ale/"

#!/bin/bash
set -e  # Exit on any error
# Instead of downloading every time I do a mistake in the script
export AC_CODE_DIR="$HOME/server_files"
sudo rm -rf "$AC_CODE_DIR/source_code"
sudo rm -rf "$AC_CODE_DIR/build"  # Clean build directory too
sudo mkdir -p "$AC_CODE_DIR/source_code/modules"
sudo mkdir -p "$AC_CODE_DIR/build"

# Copy source files (use -r for recursive directory copy)
sudo cp -r "$HOME/server_files/git/source_code/." "$HOME/server_files/source_code/"
sudo cp -r "$HOME/server_files/git/source_code_ale/." "$HOME/server_files/source_code/modules/"

# Navigate to build directory
cd "$AC_CODE_DIR/build"

# Run CMake with source directory specified
sudo cmake "$AC_CODE_DIR/source_code" \
  -DCMAKE_INSTALL_PREFIX="$AC_CODE_DIR/build/core_files" \
  -DCMAKE_C_COMPILER=/usr/bin/clang \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
  -DCMAKE_BUILD_TYPE=Release \
  -DWITH_WARNINGS=1 \
  -DTOOLS_BUILD=all \
  -DSCRIPTS=static \
  -DMODULES=static

# Build the project
sudo make -j"$(nproc --all)"

# Install (optional but recommended)
sudo make install

echo "Build completed successfully!"

sudo cp "$AC_CODE_DIR/build/core_files/etc/authserver.conf.dist" "$AC_CODE_DIR/build/core_files/etc/authserver.conf"
sudo cp "$AC_CODE_DIR/build/core_files/etc/worldserver.conf.dist" "$AC_CODE_DIR/build/core_files/etc/worldserver.conf"

sudo mkdir "$HOME/server_files/build/core_files/logs"
sudo mkdir "$HOME/server_files/build/core_files/data"

sudo cp -r "$HOME/server_files/zip/wow_data"* "$HOME/server_files/build/core_files/data"



echo "Now we nee to take attention"
mysql_secure_installation
echo "No Yes Yes Yes"

echo "Remove comment on port=3306 if you need remote acces"
echo "Open the bind-address to 0.0.0.0 for any access"
nano /etc/mysql/mysql.conf.d/mysqld.cnf
systemctl restart mysql


gnome-terminal -- bash -c "wine ~/heidisql/heidisql.exe; exec bash"

sudo nano "$HOME/server_files/build/core_files/etc/worldserver.conf"
WorldServerPort = 3700
LogsDir = "~/server_files/build/core_files/logs"
DataDir = "~/server_files/build/core_files/data"
echo "~ may not work then full path, check later"
echo "check that your port is open on device and on modem 3306 3724 3700 8085*"




mysql
tmux new -s auth
tmux new -s world
tmux attach-session -t auth
tmux attach-session -t world

sudo ufw status

ifconfig

reboot
hostname -I
./worldserver

```







