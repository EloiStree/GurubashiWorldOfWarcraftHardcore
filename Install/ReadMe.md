Draft of install on Linux device:  
```
echo "Can be done without thinking"
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y git cmake make gcc g++ clang libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev mysql-server libboost-all-dev nodejs npm tmux net-tools wine64 wine32 unzip screen nano

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
echo "No Yes Yes Yes Yes or yes yes yes yes "
sudo mysql -e "UNINSTALL COMPONENT 'file://component_validate_password';"
sudo mysql -e "CREATE USER 'acore'@'localhost' IDENTIFIED BY 'acore'; GRANT ALL PRIVILEGES ON *.* TO 'acore'@'localhost' WITH GRANT OPTION; FLUSH PRIVILEGES;"


sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
echo "Remove comment on port=3306 if you need remote acces"
echo "Open the bind-address to 0.0.0.0 for any access"
sudo systemctl restart mysql
sudo systemctl daemon-reload


gnome-terminal -- bash -c "wine ~/heidisql/heidisql.exe; exec bash"

sudo nano "$HOME/server_files/build/core_files/etc/worldserver.conf"
WorldServerPort = 3700
LogsDir = "~/server_files/build/core_files/logs"
DataDir = "~/server_files/build/core_files/data"
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

tmux kill-session -t world
tmux new -s world
sudo ./worldserver

echo "Ctrl + B then d"


echo "tmux attach-session -t auth"
echo "tmux attach-session -t world"
```

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


sudo ufw status

```
