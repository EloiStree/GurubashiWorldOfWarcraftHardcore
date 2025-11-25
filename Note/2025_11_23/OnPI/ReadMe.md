# GPT

---

# Before we start — assumptions & tips

* You have a **Raspberry Pi 5** with **8 GB** RAM (strongly recommended).
* You’re running **Raspberry Pi OS 64-bit (Bookworm / aarch64)** — AzerothCore and its toolchain expect a 64-bit environment. If you’re on a 32-bit OS switch to 64-bit first. ([Cloudy Nights][2])
* Use an **external USB3 SSD** for the server files & database — SD cards are slow and wear out quickly.
* I’ll give both “native compile” steps and a Docker option (Docker is faster to get going). AzerothCore docs recommend source installation but also provide Docker compose files. ([AzerothCore][1])

---

# Quick plan (high level)

1. Prepare OS & update.
2. Install build tools & dependencies (CMake, compilers, libs).
3. Create a dedicated user, clone AzerothCore.
4. Build the code (cmake + make).
5. Install and secure MariaDB, import AzerothCore DB files.
6. Configure auth/world servers and run.
7. (Optional) Docker method to avoid compiling. ([AzerothCore][3])

---

# Full step-by-step (with comments) — do these in order

> I’ll prefix each command with a one-line comment explaining it, then give the actual command(s). Run as a normal user unless `sudo` is shown.

---

## 0) Plug things in & prepare

* Make sure the Pi is connected to power, network (ethernet preferred), and SSD.
* Boot into **64-bit Raspberry Pi OS**. If you’re not sure, run:

```bash
# Shows if kernel is 64-bit (aarch64) or 32-bit
uname -m
```

* If it prints `aarch64`, you’re good. If it prints `armv7l` or similar, switch to the 64-bit image first. ([Cloudy Nights][2])

---

## 1) System update — always first

```bash
# Update package lists and upgrade existing packages
sudo apt update && sudo apt upgrade -y
```

> Keeps the OS packages current so installs later won’t fail.

---

## 2) Install build dependencies (native build)

```bash
# Install the compilers, cmake, libs AzerothCore typically needs, plus mariadb server/client
sudo apt install -y build-essential gcc g++ clang cmake make \
libssl-dev libbz2-dev libreadline-dev libncurses-dev \
libboost-all-dev mariadb-server mariadb-client \
libmariadb-dev libzmq3-dev libzstd-dev git libtbb-dev
```

> This installs compilers, CMake, Boost, OpenSSL, MariaDB and other dev libs used by AzerothCore. If an individual package is missing, the AzerothCore docs list required packages per distro. ([AzerothCore][3])

---

## 3) Create an unprivileged user for AzerothCore (good practice)

```bash
# Create a dedicated user so server files aren't running as root
sudo adduser --gecos "" --disabled-password acore
# Add to sudo if you want convenience (optional)
sudo usermod -aG sudo acore
```

> Running servers as a separate user improves security and keeps files tidy.

---

## 4) Switch to that user and clone the AzerothCore repo

```bash
# Switch to the new user (or run these as that user)
sudo -i -u acore bash

# Make folder and clone source
cd ~
git clone https://github.com/azerothcore/azerothcore-wotlk.git
cd azerothcore-wotlk
```

> Clones the up-to-date source code from the official repo. You’ll build from source (recommended). ([GitHub][4])

---

## 5) Prepare to build (create build dir)

```bash
# Create separate build directory (standard CMake pattern)
mkdir build && cd build
```

> Keeps source tree clean and is the recommended CMake workflow.

---

## 6) Run CMake to configure the build

```bash
# Configure the project for Release builds (fastest/optimized)
cmake .. -DCMAKE_BUILD_TYPE=Release
```

> This tells CMake to generate build files for a Release (optimized) build. If missing deps are detected, CMake will print errors — install the listed dev packages and re-run.

---

## 7) Build the project (this is CPU & RAM heavy)

```bash
# Build using 4 cores. Adjust -j value if you have more/less cores.
make -j4
```

> `make` will compile the server. On a Pi 5 this will take time but is doable. If you run out of RAM during compile you might see errors — consider closing other apps or temporarily adding a small swapfile (see note below).

**Swap note (only if compile fails due to RAM):**

```bash
# Create a 4GB swapfile (temporary — useful for compile)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# After successful build you can turn off swap:
# sudo swapoff /swapfile && sudo rm /swapfile
```

> Swap on SSD helps compilation avoid OOM, but don’t run database or worldserver with heavy swap — it slows things down.

---

## 8) (Alternative) Use Docker instead of compiling — quicker to get running

If you prefer not to compile locally, the AzerothCore Docker compose repo can start everything (worldserver, auth, db) quickly. Clone their Docker helper and run composer. Note: some arm64 manifests may be absent for certain images — there are occasional issues on ARM64 platforms; keep that in mind. ([AzerothCore][5])

```bash
# Exit acore user if needed (do this as root or your normal user)
exit

# Clone the docker repo and run
git clone https://github.com/azerothcore/acore-docker.git
cd acore-docker
# Build and run (first run may take a while as client data & databases are prepared)
docker compose up --build
```

> Docker isolates all components and sets up DB, server, and client-data volumes for you. If an image lacks an arm64 manifest you may see an error — check the Docker logs or use a Pi-compatible tag. ([GitHub][6])

---

## 9) MariaDB: secure, create user & import databases

If you built natively, set up MariaDB locally and import the SQL files included in `azerothcore-wotlk`. Work as a privileged user to create DBs and import.

```bash
# Start MariaDB service (systemd will normally do this)
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Secure installation (set root password, remove test DB etc)
sudo mysql_secure_installation
```

> `mysql_secure_installation` walks you through setting a root password and removing insecure defaults.

Now create the databases and import the SQL files (run as root or using mysql client):

```bash
# Login to mysql as root
sudo mysql -u root -p

# In mysql prompt, run (replace <password> and adjust names if you want)
CREATE DATABASE auth;
CREATE DATABASE characters;
CREATE DATABASE world;

CREATE USER 'acore'@'localhost' IDENTIFIED BY 'YourStrongPassword';
GRANT ALL PRIVILEGES ON auth.* TO 'acore'@'localhost';
GRANT ALL PRIVILEGES ON characters.* TO 'acore'@'localhost';
GRANT ALL PRIVILEGES ON world.* TO 'acore'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

```bash
# Import SQL files (run from your azerothcore-wotlk/sql/ directory)
cd ~/azerothcore-wotlk/sql
# Example import commands — these filenames may vary slightly by repo version
mysql -u acore -p world < acore_world.sql
mysql -u acore -p characters < acore_characters.sql
mysql -u acore -p auth < acore_auth.sql
```

> The SQL files populate the game world data, character schema and auth data. If you used Docker, the docker compose will usually auto-import these on first run. ([AzerothCore][7])

---

## 10) Tunings for MariaDB on low-memory device (sample my.cnf excerpt)

Create `/etc/mysql/mariadb.conf.d/99-acore-small.cnf` with conservative settings so MariaDB doesn’t use too much RAM:

```ini
# /etc/mysql/mariadb.conf.d/99-acore-small.cnf
[mysqld]
innodb_buffer_pool_size = 512M
key_buffer_size = 64M
query_cache_size = 0
max_connections = 100
innodb_log_file_size = 64M
innodb_flush_method = O_DIRECT
```

```bash
# After saving the file, restart mariadb
sudo systemctl restart mariadb
```

> These lower memory settings keep MariaDB stable on 8GB Pi; tweak upward if you have more RAM and many players.

---

## 11) Configure AzerothCore server conf files

After building, copy config dist files and adapt:

```bash
# from build directory (or from source conf/ directory)
cd ~/azerothcore-wotlk
cp conf/dist/worldserver.conf.dist conf/worldserver.conf
cp conf/dist/authserver.conf.dist conf/authserver.conf

# edit the files to set DB credentials and IPs
nano conf/worldserver.conf
# change the login database / hostname / user / password under the [LoginDatabaseInfo] and [WorldDatabaseInfo] sections
```

> Set `LoginDatabaseInfo`, `WorldDatabaseInfo`, and `CharacterDatabaseInfo` to the DB user `acore` and password you created. Also set `BindIP` to your Pi's local IP if you want LAN access.

---

## 12) Run the servers (native build)

Assuming the build produced `bin/worldserver` and `bin/authserver`:

```bash
# Start auth server (auth manages logins)
cd ~/azerothcore-wotlk/bin
./authserver 2>&1 | tee ~/authserver.log &

# Start worldserver (game server)
./worldserver 2>&1 | tee ~/worldserver.log &
```

> `tee` writes the log to a file and stdout so you can watch. Use `ps`/`top` to check processes. If you used Docker, `docker compose up` already started these services inside containers.

---

## 13) Create an in-game account & test

On the worldserver console (or via remote console depending on your setup) create an account:

```bash
# Connect to worldserver console (if you're running it interactively you already have it)
# Example in worldserver console:
account create YourUser YourPass YourPass
account set gmlevel YourUser 3 -1
```

> The console accepts `account` commands to create and promote accounts for testing. Then use a WoW 3.3.5a client configured to connect to your Pi's IP to login.

---

## 14) Basic troubleshooting tips

* If CMake errors: missing dev packages — read the error, install listed libs, re-run cmake. ([AzerothCore][3])
* If MariaDB connection failed: check credentials in `worldserver.conf` and that MariaDB is running (`sudo systemctl status mariadb`).
* If Docker fails with `no matching manifest for linux/arm64` — that means a used image lacks an arm64 build; try alternate/arm64 tags or build locally. There have been arm64 issues reported for some images. ([GitHub][6])

---

## 15) Performance & final tips

* SSD for DB and logs — big performance win.
* Use Release build (done above) — significantly faster than Debug.
* Limit playerbots / AI modules on Pi — they use CPU.
* Use the Docker route if you want repeatable, containerized deployments — but check for arm64 compatibility on first run. ([AzerothCore][5])

---

# Quick recap with the most critical commands (copy-paste friendly)

```bash
# Update
sudo apt update && sudo apt upgrade -y

# Install deps (native build)
sudo apt install -y build-essential gcc g++ clang cmake make \
libssl-dev libbz2-dev libreadline-dev libncurses-dev \
libboost-all-dev mariadb-server mariadb-client \
libmariadb-dev libzmq3-dev libzstd-dev git libtbb-dev

# Create user & clone
sudo adduser --gecos "" --disabled-password acore
sudo -i -u acore bash
cd ~
git clone https://github.com/azerothcore/azerothcore-wotlk.git
cd azerothcore-wotlk
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j4

# MariaDB (as root)
sudo mysql_secure_installation
sudo mysql -u root -p
# then create DBs and user as shown in step 9

# Configure server confs
cp conf/dist/worldserver.conf.dist conf/worldserver.conf
cp conf/dist/authserver.conf.dist conf/authserver.conf
# edit them to set DB credentials

# Run servers
cd ~/azerothcore-wotlk/bin
./authserver 2>&1 | tee ~/authserver.log &
./worldserver 2>&1 | tee ~/worldserver.log &
```

---

# Sources & further reading

* Official AzerothCore installation guide & Linux requirements (source of recommended steps). ([AzerothCore][1])
* AzerothCore Docker (useful if you prefer docker compose). ([AzerothCore][5])
* Reported ARM64 manifest issues for some Docker images — you may hit this on Pi; if so see the issue notes and consider building images locally. ([GitHub][6])
* Raspberry Pi 5 / 64-bit OS recommendations and user notes. ([Cloudy Nights][2])

---

If you want **I will** (pick one, and I’ll give exact commands for that choice, no waiting):

1. Give a **ready-to-paste full install script** (one script that runs everything end-to-end, with safe pauses).
2. Give a **docker-compose.yml** tuned for Raspberry Pi 5 / arm64 that I craft for you.
3. Give a **MariaDB tuning file** and `systemd` service file for auto-start with safe defaults.

Tell me which of the three you want and I’ll paste it now — fully commented.

[1]: https://www.azerothcore.org/wiki/installation?utm_source=chatgpt.com "Installation Guide"
[2]: https://www.cloudynights.com/forums/topic/914113-raspberry-pi5-with-64-bit-os-setup/?utm_source=chatgpt.com "Raspberry Pi5 with 64-bit OS setup"
[3]: https://www.azerothcore.org/wiki/linux-requirements?utm_source=chatgpt.com "Linux Requirements"
[4]: https://github.com/azerothcore/azerothcore-wotlk?utm_source=chatgpt.com "azerothcore/azerothcore-wotlk: Complete Open Source ..."
[5]: https://www.azerothcore.org/wiki/install-with-docker?utm_source=chatgpt.com "Install with Docker"
[6]: https://github.com/azerothcore/azerothcore-wotlk/issues/10851?utm_source=chatgpt.com "./acore.sh docker client-data failed on macbook m1 · Issue ..."
[7]: https://www.azerothcore.org/wiki/database-installation?utm_source=chatgpt.com "Database Installation"
