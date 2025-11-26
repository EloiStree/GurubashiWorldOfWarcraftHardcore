Disable Log In on the PC
```
sudo nano /etc/gdm3/custom.conf
```
```
AutomaticLoginEnable = true
AutomaticLogin = gurubashi 
```



If you dont care of security and want to remove sudo password
```
sudo visudo
sudo sudo -l
```
```
gurubashi ALL=(ALL) NOPASSWD:ALL
```

Draft of install on Linux device:  
```


echo "Can be done without thinking"
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev nodejs npm tmux net-tools wine64 wine32 unzip screen nano

git config --global user.name "HelloWorld"
git config --global user.mail "HelloWorld@gmail.com"

sudo npm install pm2 -g
sudo dpkg --add-architecture i386
wget https://github.com/wowgaming/client-data/releases/download/v18.0/Data.zip
wget https://github.com/HeidiSQL/HeidiSQL/releases/download/v12.13.0.7147/HeidiSQL_12.13_64_Portable.zip

sudo mkdir "$HOME/server_files"
sudo mkdir -p "$HOME/server_files/zip/heidisql"
sudo mkdir -p "$HOME/server_files/zip/wow_data"

sudo unzip -o "HeidiSQL_12.13_64_Portable.zip" -d "$HOME/server_files/zip/heidisql"
sudo unzip -o "Data.zip" -d "$HOME/server_files/zip/wow_data"

sudo chmod -R 777 ~/server_files
sudo git clone https://github.com/azerothcore/azerothcore-wotlk.git --branch master --single-branch "$HOME/server_files/git/source_code"
sudo git clone https://github.com/azerothcore/mod-ale.git "$HOME/server_files/git/source_code_ale/"

sudo chmod -R 777 ~/server_files

export AC_CODE_DIR="$HOME/server_files"
sudo rm -rf "$AC_CODE_DIR/source_code"
sudo rm -rf "$AC_CODE_DIR/build"  # Clean build directory too
sudo mkdir -p "$AC_CODE_DIR/source_code/modules"
sudo mkdir -p "$AC_CODE_DIR/build"


# 1. Clean previous build artifacts
cd "$HOME/server_files/source_code/build"
sudo rm -rf *

# 2. Copy source files (if needed)
sudo cp -r "$HOME/server_files/git/source_code/." "$HOME/server_files/source_code/"
sudo cp -r "$HOME/server_files/git/source_code_ale/." "$HOME/server_files/source_code/modules/mod-ale"

# 3. Set permissions before building
sudo chmod -R 777 ~/server_files

# 4. Run CMake configuration (this generates the missing headers)
cd "$HOME/server_files/source_code/build"
cmake "$HOME/server_files/source_code" \
  -DCMAKE_INSTALL_PREFIX="$HOME/server_files/build/core_files" \
  -DCMAKE_C_COMPILER=/usr/bin/clang \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
  -DCMAKE_BUILD_TYPE=Release \
  -DWITH_WARNINGS=1 \
  -DTOOLS_BUILD=all \
  -DSCRIPTS=static \
  -DMODULES=static

# 5. Build
make -j$(nproc --all)

# 6. Install
make install


sudo cp "$AC_CODE_DIR/build/core_files/etc/authserver.conf.dist" "$AC_CODE_DIR/build/core_files/etc/authserver.conf"
sudo cp "$AC_CODE_DIR/build/core_files/etc/worldserver.conf.dist" "$AC_CODE_DIR/build/core_files/etc/worldserver.conf"

sudo mkdir "$HOME/server_files/build/core_files/logs"
sudo mkdir "$HOME/server_files/build/core_files/data"

sudo cp -r "$HOME/server_files/zip/wow_data/"* "$HOME/server_files/build/core_files/data/"



echo "Now we nee to take attention"
sudo mysql_secure_installation
echo "No Yes Yes Yes Yes "
sudo mysql -e "UNINSTALL COMPONENT 'file://component_validate_password';"
sudo mysql -e "CREATE USER 'acore'@'localhost' IDENTIFIED BY 'acore'; GRANT ALL PRIVILEGES ON *.* TO 'acore'@'localhost' WITH GRANT OPTION; FLUSH PRIVILEGES;"


sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
echo "Remove comment on port=3306 if you need remote acces"
echo "Open the bind-address to 0.0.0.0 for any access"
echo "ONLY IF YOU NEED OUT HOME ACCESS"
sudo systemctl restart mysql
sudo systemctl daemon-reload


sudo nano "$HOME/server_files/build/core_files/etc/worldserver.conf"
WorldServerPort = 3700
LogsDir = "/home/gurubashi/server_files/build/core_files/logs"
DataDir = "/home/gurubashi/server_files/build/core_files/data"
echo "~ may not work then full path, check later"
echo "check that your port is open on device and on modem 3306 3724 3700 8085*"
-- To make the game free for all
GameType = 16
StartPlayerMoney = 100000
StartPlayerLevel = 80

Copy this script in:   
sudo nano "$HOME/server_files/build/core_files/bin/lua_scripts/HardCore.lua"  
https://github.com/EloiStree/GurubashiWorldOfWarcraftHardcore/blob/main/StepByStep/HardCore.lua  


sudo ufw status

sudo chmod -R 777 ~/server_files
```

Tmux to control the server
```
cd ~/server_files/build/core_files/bin
tmux kill-session -t auth
tmux new -s auth
sudo ./authserver

cd ~/server_files/build/core_files/bin
tmux kill-session -t world
tmux new -s world
sudo ./worldserver

echo "Ctrl + B then d"


echo "tmux attach-session -t auth"
echo "tmux attach-session -t world"
```

Create some script that auto-launch the server at start:   
https://github.com/EloiStree/2025_11_25_azeroth_bin_script

sudo git clone https://github.com/EloiStree/2025_11_25_azeroth_bin_script.git ~/server_files/build/core_files/bin/2025_11_25_azeroth_bin_script



Create account
```
--  in ./worldserver
.account create admin admin
account set gmlevel admin 3 -1

.account create student001 12345678
.account create student002 12345678
.account create student003 12345678
.account create student004 12345678
.account create student005 12345678
.account create student006 12345678
.account create student008 12345678
.account create student009 12345678
.account create student010 12345678
.account create student011 12345678
.account create student012 12345678
.account create student013 12345678
.account create student014 12345678
.account create student015 12345678
.account create student016 12345678




gnome-terminal -- bash -c "wine ~/server_files/zip/heidisql/heidisql.exe; exec bash"
echo "acore acore as name and password"

-- change realms ip name and port
-- change level of admin to owner


-- Set static local ip
https://github.com/EloiStree/GurubashiWorldOfWarcraftHardcore/issues/11

ip a
sudo nano /etc/netplan/01-network-manager-all.yaml

---- TO COPY
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp1s0:
      dhcp4: false
      addresses:
        - 192.168.1.88/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1

sudo netplan apply
ip a

-- Go on a window pc
Download a client
https://www.warmane.com/download 
https://www.bittorrent.com/downloads/complete/classic
Change the Data/enUS/realmlist.wtf by
-- If remote server  
set realmlist YOUR_SERVER_IP
-- If local server   
set realmlist 192.168.1.88    

```
