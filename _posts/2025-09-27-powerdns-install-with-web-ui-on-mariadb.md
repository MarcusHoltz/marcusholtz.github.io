---
layout: post
title: PowerDNS Install with Web GUI and DB
date: 2025-09-27 11:33:00 -0700
categories: [Proxmox, ProxmoxNetworking]
tags: [virtualization, proxmox, networking, security, server, firewall, DNS, PowerDNS, SDN]
pin: false
image:
  path: /assets/img/header/header--proxmox--powerdns-and-admin-using-mariadb.jpg
  alt: Proxmox SDN uses PowerDNS to manage DNS
---

# Installing PowerDNS with PowerDNS-Admin on Debian 12


* * *

## What is PowerDNS and How Can I Harness this Power

PowerDNS is a powerful, flexible authoritative DNS server that can be used to manage your domain's DNS records programmatically. But sometimes you dont want to look at text and use a GUI instead. PowerDNS can be paired paired with PowerDNS-Admin, so you get a modern web interface for your DNS management. 

In this write-up, we'll walk through installing both PowerDNS and PowerDNS-Admin on Debian 12 (Bookworm). We'll be using MariaDB to provide a database for both these services.

Proxmox's new SDN uses PowerDNS for it's DNS features, and so a PowerDNS LXC should be standard on most Proxmox multi-client/doamin setups.

And that's what we're doing today! By the end of this write-up, you'll have a fully functional DNS server with a web-based management interface, complete with API access for Proxmox SDN's DNS automation.


* * *

## What Does This Write Up Talk About

- **PowerDNS Authoritative Server**: A local DNS server, NOT one that does global lookups on the internet!
- **MariaDB Database**: Backend storage for DNS records and application data
- **PowerDNS-Admin**: A modern web interface for managing your DNS zones
- **REST API**: Programmatic access to your DNS infrastructure, essential for Proxmox SDN's DNS automation with PowerDNS
- **Multi-tenant Support**: Ready for Proxmox multi-client and Proxmox multi-domain deployments


* * *

## 🏁 Speedrun This Process with a Script

If like to download scripts from the internet - you're in luck!

I have, at the bottom of this post, a - [🎚️ PowerDNS EZ-MODE script](#️-powerdns-ez-mode-script). 

> This script will complete the entire write-up for you! Just be sure to change the config variables first.

Please continue on to go through the steps of setting up PowerDNS to use MariaDB, then PowerDNS-Admin.


* * *

## Part 1: System Preparation and MariaDB Installation

### Step 1: Update Your System

Let's start by ensuring your system is up to date:

```bash
apt update && apt upgrade -y
```

### Step 2: Install Essential Dependencies

Install the core packages we'll need throughout this process:

```bash
apt install -y sudo curl git libpq-dev software-properties-common gnupg2
```

### Step 3: Install MariaDB

MariaDB will serve as the database backend for both PowerDNS (storing DNS records for your Proxmox multi-domain environments) and PowerDNS-Admin (storing users and settings).

First, download and install the official MariaDB repository setup script:

```bash
curl -LsS -o mariadb_repo_setup https://downloads.mariadb.com/MariaDB/mariadb_repo_setup && echo "$(curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup.sha256)" > mariadb_repo_setup.sha256 && sha256sum -c mariadb_repo_setup.sha256 && sudo bash mariadb_repo_setup && rm mariadb_repo_setup mariadb_repo_setup.sha256
```

Now install MariaDB server and client:

```bash
sudo apt update && sudo apt install -y mariadb-server mariadb-client
```

Start and enable the MariaDB service to run on boot:

```bash
sudo systemctl enable --now mariadb
```

### Step 4: Secure Your MariaDB Installation

For new MariaDB installations, the next step is to run a security script. This script changes some of the less secure default options for things like remote root logins and sample users.

Optionally, run the security script to improve your database security posture:

```bash
sudo mysql_secure_installation
```

I answered, `no` to the first two, defaults for everything else.

I used the following answers:

```
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

### Step 5: Create Database and User

Now we'll create the databases and user account that PowerDNS and PowerDNS-Admin will use. 

**Important Note**: Avoid special characters in the password, as PowerDNS has been known to have issues with certain special characters, which can result in authentication errors: “Access denied for user ‘powerdns_user’@’localhost’ (using password: YES)“

For this guide, we'll use:
- **Username**: `powerdnsmariaccessuser`
- **Password**: `MikeJohnsonLikes9091Hamburgers`
- **PowerDNS Database**: `powerdns`
- **PowerDNS-Admin Database**: `powerdnsadmin`

Log into MariaDB as root:

```bash
sudo mysql -u root
```

Execute the following SQL commands:

```sql
CREATE DATABASE powerdns;
CREATE DATABASE powerdnsadmin;

CREATE USER 'powerdnsmariaccessuser'@'localhost' IDENTIFIED BY 'MikeJohnsonLikes9091Hamburgers';

GRANT ALL PRIVILEGES ON *.* TO 'powerdnsmariaccessuser'@'localhost';

FLUSH PRIVILEGES;
EXIT
```

* * *

## Part 2: Installing PowerDNS

### Step 6: Add the PowerDNS Repository

To get the latest version of PowerDNS (version 5.0 as of writing), we'll add the official PowerDNS repository. This ensures we're running the most current and secure version: https://doc.powerdns.com/authoritative/installation.html

You have to install their repository:

https://repo.powerdns.com/

Create the repository configuration:

```bash
printf "deb [signed-by=/etc/apt/keyrings/auth-50-pub.asc] http://repo.powerdns.com/debian bookworm-auth-50 main" > /etc/apt/sources.list.d/pdns.list

printf "Package: pdns-*\nPin: origin repo.powerdns.com\nPin-Priority: 600" > /etc/apt/preferences.d/auth-50
```

Add the GPG key and install PowerDNS:

```bash
sudo install -d /etc/apt/keyrings; curl https://repo.powerdns.com/FD380FBB-pub.asc | sudo tee /etc/apt/keyrings/auth-50-pub.asc &&
sudo apt-get update &&
sudo apt-get install pdns-server

```

### Step 7: Verify PowerDNS is Running

Verify the port 53 is open for PowerDNS to provide DNS:

```bash
sudo ss -alnp4 | grep pdns
```

You should see output showing PowerDNS listening on port 53.

* * *

## Part 3: Configuring PowerDNS with MariaDB

### Step 8: Install the MariaDB Backend

PowerDNS supports multiple database backends. Install the MariaDB/MySQL backend to enable an easy-to-backup database for your Proxmox multi-domain DNS zones:

```bash
sudo apt install pdns-backend-mysql
```

### Step 9: Initialize the PowerDNS Database Schema

Import the database schema that PowerDNS needs to store DNS records:

```bash
mariadb --user=powerdnsmariaccessuser \
        --password=MikeJohnsonLikes9091Hamburgers \
        --database=powerdns < /usr/share/pdns-backend-mysql/schema/schema.mysql.sql
```

Verify the schema was imported successfully:

```bash
sudo mysql -u root
use powerdns;
show tables;
exit
```

You should see several tables including `domains`, `records`, and `cryptokeys`.

### Step 10: Configure PowerDNS Database Connection

Create a configuration file that tells PowerDNS how to connect to MariaDB:

```bash
sudo bash -c 'cat <<EOF > /etc/powerdns/pdns.d/pdns.local.gmysql.conf
# MySQL Configuration
# Launch gmysql backend
launch+=gmysql

# gmysql parameters
gmysql-host=127.0.0.1
gmysql-port=3306
gmysql-dbname=powerdns
gmysql-user=powerdnsmariaccessuser
gmysql-password=MikeJohnsonLikes9091Hamburgers
#gmysql-dnssec=yes
# gmysql-socket=
EOF'
```

Set the appropriate permissions on this configuration file:

```bash
sudo chown pdns: /etc/powerdns/pdns.d/pdns.local.gmysql.conf
sudo chmod 640 /etc/powerdns/pdns.d/pdns.local.gmysql.conf
```

### Step 11: Test the Database Connection

Stop the PowerDNS service:

```bash
sudo systemctl stop pdns.service
```

Run PowerDNS in the foreground with verbose logging to verify the database connection:

```bash
sudo pdns_server --daemon=no --guardian=no --loglevel=9
```

You should see log output indicating a successful database connection. Press `Ctrl+C` to stop the server.

If successful, restart PowerDNS as a service and enable it to start on boot:

```bash
sudo systemctl restart pdns && sudo systemctl enable --now pdns
```

### Step 12: Verify PowerDNS is Working

Check that PowerDNS is responding to DNS queries:

```bash
dig @127.0.0.1
```

* * *

## Part 4: Installing PowerDNS-Admin

PowerDNS-Admin is a web-based interface that makes managing your DNS server much easier, especially in Proxmox multi-client environments where multiple administrators need access. It's built with Python Flask and requires Node.js for asset compilation.


### Step 13: Install Python and Build Dependencies

Install the necessary packages for building alllllllll the Python libraries:

```bash
sudo apt install -y apt-transport-https build-essential curl git libffi-dev libldap2-dev libmariadb-dev libpq-dev libsasl2-dev libssl-dev libxml2-dev libxmlsec1-dev libxslt1-dev pkg-config python3-dev python3-flask python3-venv virtualenv
```

### Step 14: Install Node.js

Add the NodeSource repository for Node.js 20.x to get npm:

```bash
curl -sL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

Install Node.js (npm is included automatically):

```bash
sudo apt install -y nodejs
```

**Important**: Do not install npm separately from Debian's repositories—it's outdated and broken. Always use NodeSource or nvm.

Verify the installation:

```bash
npm -v
```

### Step 15: Install Yarn

Yarn is a package manager we'll use to build the frontend assets:

```bash
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | \
    sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | \
    sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt update && sudo apt install -y yarn
```

### Step 16: Clone and Set Up PowerDNS-Admin

Clone the PowerDNS-Admin repository:

```bash
git clone https://github.com/PowerDNS-Admin/PowerDNS-Admin.git /opt/web/powerdns-admin
cd /opt/web/powerdns-admin
```

Create a Python virtual environment:

```bash
python3 -m venv ./venv
```

Activate the virtual environment and install Python dependencies:

```bash
source ./venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

### Step 17: Configure PowerDNS-Admin

Deactivate the virtual environment temporarily:

```bash
deactivate
```

Copy the development configuration to create a production config:

```bash
cp /opt/web/powerdns-admin/configs/development.py /opt/web/powerdns-admin/configs/production.py
```

Generate a secret key for Flask sessions:

```bash
python3 -c 'import secrets; print(secrets.token_hex())' 'ToddHowardLikes4126Turkeysammies'
```

Copy the output—you'll need it in the next step.

Edit the production configuration:

```bash
nano /opt/web/powerdns-admin/configs/production.py
```

Find and update the database configuration section:

```python
# Change from:
SQLA_DB_USER = 'pda'
SQLA_DB_PASSWORD = 'changeme'
SQLA_DB_NAME = 'pda'

# To:
SQLA_DB_USER = 'powerdnsmariaccessuser'
SQLA_DB_PASSWORD = 'MikeJohnsonLikes9091Hamburgers'
SQLA_DB_NAME = 'powerdnsadmin'
```

Scroll down to find the SQLite database URI section and update it:

```python
# Comment out the SQLite line:
# SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'pdns.db')

# Add the MySQL connection string:
SQLALCHEMY_DATABASE_URI = 'mysql://powerdnsmariaccessuser:MikeJohnsonLikes9091Hamburgers@127.0.0.1/powerdnsadmin'
```

Also, add your generated secret key to the `SECRET_KEY` variable in this file.

Save and close the file.

### Step 18: Initialize the Application

Set up the Flask environment:

```bash
cd /opt/web/powerdns-admin
source ./venv/bin/activate
export FLASK_CONF=../configs/production.py
```

Run database migrations:

```bash
export FLASK_APP=powerdnsadmin/__init__.py
flask db upgrade
```

Generate frontend assets:

```bash
yarn install --pure-lockfile
flask assets build
```

### Step 19: Test PowerDNS-Admin

You can now test the application:

```bash
./run.py
```

Visit `http://your-server-ip:9191` in your web browser. You should see a registration page.

**Important Notes**:
- Disable any ad blockers or script blockers while accessing the interface
- On that page - you need to **register** your first user, that first user will automatically be granted Administrator rights. 
- After registration, you'll be redirected to the login page—this is normal. Yes, it worked. You got no afirmation of user creation, you just got re-directed back to the login page. Use your newly created credentials.

When testing is complete, press `Ctrl+C` and deactivate the virtual environment:

```bash
deactivate
```

* * *

## Part 5: Enabling the PowerDNS API

The PowerDNS API is crucial for API access for Proxmox SDN's DNS automation with PowerDNS. This allows PowerDNS-Admin and Proxmox SDN to manage your DNS records programmatically, enabling seamless integration with your virtualization infrastructure.


### Step 20: Generate an API Key

You can generate a secure API key at [https://codepen.io/corenominal/pen/rxOmMJ](https://codepen.io/corenominal/pen/rxOmMJ) or use any UUID generator.

For this guide, we'll use: `135af146-c29c-4dd6-8608-14b3c0386ae9`

### Step 21: Configure the API

Edit the main PowerDNS configuration file:

```bash
sudo nano /etc/powerdns/pdns.conf
```

Add or uncomment these lines to enable API access for Proxmox SDN's DNS automation with PowerDNS and PowerDNS-Admin:


```ini
#################################
# api   Enable/disable the REST API (including HTTP listener)
#
api=yes

#################################
# api-key       Static pre-shared authentication key for access to the REST API
#
api-key=135af146-c29c-4dd6-8608-14b3c0386ae9
```

Save the file and restart PowerDNS:

```bash
sudo systemctl restart pdns
```

### Step 22: Clean Up Configuration Issues

Some PowerDNS configurations may contain invalid directives. Check your configuration directory:

```bash
cd /etc/powerdns/pdns.d/
```

If you have a `bind.conf` file, edit it:

```bash
sudo nano bind.conf
```

Remove or comment out any lines except the first one (typically `launch+=bind`). The `bind-config` directive may not be valid in your version and should be removed.

* * *

## Part 6: Creating a System Service

To be able to manage PowerDNS Admin just like other system services, we need to create a service file for it.

### Step 23: Create the Service File

```bash
sudo nano /etc/systemd/system/pdnsadmin.service
```

Add the following content:

```ini
[Unit]
Description=PowerDNS-Admin
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/web/powerdns-admin
User=root
Group=root

# Environment variable for Flask config
Environment=FLASK_CONF=../configs/production.py

# Use the python interpreter inside your virtual environment
ExecStart=/opt/web/powerdns-admin/venv/bin/python /opt/web/powerdns-admin/run.py

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Step 24: Create the Socket File

Also, create a socket file:

```bash
sudo nano /etc/systemd/system/pdnsadmin.socket
```

Add these lines:

```ini
[Unit]
Description=PowerDNS-Admin socket

[Socket]
ListenStream=/run/pdnsadmin/socket

[Install]
WantedBy=sockets.target
```

### Step 25: Set Up Runtime Directory

Create the runtime directory and configure it to persist across reboots:

```bash
mkdir -p /run/pdnsadmin/
echo "d /run/pdnsadmin 0755 pdns pdns -" >> /etc/tmpfiles.d/pdnsadmin.conf
```

### Step 26: Set Permissions

```bash
chown -R pdns: /run/pdnsadmin/
chown -R pdns: /opt/web/powerdns-admin
```

### Step 27: Enable and Start the Service

Reload systemd to recognize the new service:

```bash
systemctl daemon-reload
```

Enable and start the PowerDNS-Admin service:

```bash
systemctl enable --now pdnsadmin.service pdnsadmin.socket
```

Check the service status:

```bash
systemctl status pdnsadmin.service pdnsadmin.socket
```

* * *

## Part 7: Connecting PowerDNS-Admin to PowerDNS

### Step 28: Configure the API Connection

1. Log into the PowerDNS-Admin web interface at `http://your-server-ip:9191`
2. Navigate to the settings or configuration section
3. Add your PowerDNS server with:
   - **API URL**: `http://your-server-ip:8081`
   - **API Key**: `135af146-c29c-4dd6-8608-14b3c0386ae9` (the key from your `pdns.conf`)

Once connected, you'll be able to create and manage DNS zones through the web interface!


* * *


## Part 8: Integrating with Proxmox SDN

Since Proxmox SDN uses PowerDNS for its DNS features, your installation is now ready for Proxmox integration. Here's how to leverage your PowerDNS setup with Proxmox VE:

### Configuring Proxmox SDN DNS Integration

1. **In Proxmox VE**, navigate to Datacenter → SDN → DNS
2. **Add DNS Server** with these settings:
   - **Type**: PowerDNS
   - **Server**: Your PowerDNS server IP
   - **API URL**: `http://your-powerdns-ip:8081`
   - **API Key**: `135af146-c29c-4dd6-8608-14b3c0386ae9`

### Benefits for Proxmox Multi-Client Environments

Your PowerDNS setup now enables:
- **Automatic DNS record creation** when VMs are deployed in Proxmox SDN zones
- **Proxmox multi-client support** through PowerDNS-Admin's user management and role-based access control
- **Centralized DNS management** across multiple Proxmox clusters
- **API-driven automation** for infrastructure-as-code deployments

### Benefits for Proxmox Multi-Domain Configurations

The database-backed PowerDNS installation provides:
- **Multiple DNS zones** for different departments or customers
- **Domain delegation** capabilities for Proxmox multi-domain setups
- **Consistent DNS management** across all your Proxmox environments
- **Scalable architecture** that grows with your infrastructure


> And remember... if there's any problems, it's never the DNS.



* * *

## 🎚️ PowerDNS EZ-MODE script

But if you are having problems, just use the script below.

You need to make a few changes before running it:

### PowerDNS and PowerDNS Admin with MariaDB Script Configuration Overview

Before running the script, update the variables below to match your environment:

| Variable             | Purpose                                           | Example Value                          |
|----------------------|---------------------------------------------------|----------------------------------------|
| `DB_USER`            | Database username for PowerDNS & PowerDNS-Admin   | `powerdnsmariaccessuser`               |
| `DB_PASSWORD`        | Database password for the above user              | `TableSauceisnotRefrigerated1987`      |
| `PDNS_DB_NAME`       | Database name used by PowerDNS                    | `powerdns`                             |
| `PDNSADMIN_DB_NAME`  | Database name used by PowerDNS-Admin              | `powerdnsadmin`                        |
| `PDNS_API_KEY`       | Secret key for PowerDNS API                       | `135af146-c29c-4dd6-8608-14b3c0386ae9` |
| `PDNSADMIN_PATH`     | Installation path (no need to change)             | `/opt/web/powerdns-admin`              |
| `FLASK_SECRET_KEY`   | Secret for securing web sessions (ok if empty)    | `leave empty to auto-generate`         |


* * *

### PowerDNS and PowerDNS Admin with MariaDB Script

Please be sure to make changes to the configuration variables below. Enjoy PowerDNS using MariaDB!

```bash

#!/bin/bash

#############################################
# PowerDNS & PowerDNS-Admin Installation Script
# For Debian 12 (Bookworm)
#############################################

#############################################
# CONFIGURATION VARIABLES - EDIT THESE
#############################################

# Database Configuration
# Please change the DB_USER and DB_PASSWORD to something unique you can remember
DB_USER="powerdnsmariaccessuser"
DB_PASSWORD="MikeJohnsonLikes9091Hamburgers"
PDNS_DB_NAME="powerdns"
PDNSADMIN_DB_NAME="powerdnsadmin"

# PowerDNS API Configuration
# You can generate an API key at https://codepen.io/corenominal/pen/rxOmMJ
PDNS_API_KEY="135af146-c29c-4dd6-8608-14b3c0386ae9"

# PowerDNS-Admin Installation Path
PDNSADMIN_PATH="/opt/web/powerdns-admin"

# Flask Secret Key (auto-generated if left empty)
FLASK_SECRET_KEY=""

#############################################
# PRE-FLIGHT CHECKS
#############################################

# Ensure the script is run as root (via sudo or directly)
if [[ $EUID -ne 0 ]]; then
  echo "This script must be run as root. Re-running with sudo..."
  exec sudo "$0" "$@"
fi

# Exit on any error
set -e

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Function to print colored output
print_status() {
    echo -e "${GREEN}[✓]${NC} $1"
}

print_error() {
    echo -e "${RED}[✗]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[!]${NC} $1"
}

# Tray tables in the upright position
print_status "Starting PowerDNS installation script..."

# Check if Debian 12
if ! grep -q "bookworm" /etc/os-release; then
    print_warning "This script is designed for Debian 12 (Bookworm). Proceeding anyway..."
fi

#############################################
# SYSTEM PREPARATION
#############################################

print_status "Updating system packages..."
apt update && apt upgrade -y

print_status "Installing essential dependencies..."
apt install -y sudo curl git libpq-dev software-properties-common gnupg2

#############################################
# MARIADB INSTALLATION
#############################################

print_status "Installing MariaDB repository..."
curl -LsS -o mariadb_repo_setup https://downloads.mariadb.com/MariaDB/mariadb_repo_setup && echo "$(curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup.sha256)" > mariadb_repo_setup.sha256 && sha256sum -c mariadb_repo_setup.sha256 && sudo bash mariadb_repo_setup && rm mariadb_repo_setup mariadb_repo_setup.sha256

print_status "Installing MariaDB server and client..."

apt update && DEBIAN_FRONTEND=noninteractive apt-get install -y mariadb-server

print_status "Starting and enabling MariaDB..."
systemctl enable --now mariadb

print_status "Securing MariaDB installation..."
# Automate mysql_secure_installation
mysql -u root <<EOF
DELETE FROM mysql.user WHERE User='';
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
FLUSH PRIVILEGES;
EOF

print_status "Creating PowerDNS databases and user..."
mysql -u root <<EOF
CREATE DATABASE IF NOT EXISTS ${PDNS_DB_NAME};
CREATE DATABASE IF NOT EXISTS ${PDNSADMIN_DB_NAME};
CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASSWORD}';
GRANT ALL PRIVILEGES ON *.* TO '${DB_USER}'@'localhost';
FLUSH PRIVILEGES;
EOF

#############################################
# POWERDNS INSTALLATION
#############################################

print_status "Adding PowerDNS repository..."
printf "deb [signed-by=/etc/apt/keyrings/auth-50-pub.asc] http://repo.powerdns.com/debian bookworm-auth-50 main" > /etc/apt/sources.list.d/pdns.list
printf "Package: pdns-*\nPin: origin repo.powerdns.com\nPin-Priority: 600" > /etc/apt/preferences.d/auth-50

install -d /etc/apt/keyrings
curl https://repo.powerdns.com/FD380FBB-pub.asc | tee /etc/apt/keyrings/auth-50-pub.asc > /dev/null

print_status "Installing PowerDNS server..."
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y pdns-server

print_status "Installing PowerDNS MySQL backend..."
DEBIAN_FRONTEND=noninteractive apt-get install -y pdns-backend-mysql

#############################################
# POWERDNS CONFIGURATION
#############################################

print_status "Importing PowerDNS database schema..."
mariadb --user=${DB_USER} --password=${DB_PASSWORD} --database=${PDNS_DB_NAME} < /usr/share/pdns-backend-mysql/schema/schema.mysql.sql

print_status "Configuring PowerDNS database connection..."
cat > /etc/powerdns/pdns.d/pdns.local.gmysql.conf <<EOF
# MySQL Configuration
# Launch gmysql backend
launch+=gmysql

# gmysql parameters
gmysql-host=127.0.0.1
gmysql-port=3306
gmysql-dbname=${PDNS_DB_NAME}
gmysql-user=${DB_USER}
gmysql-password=${DB_PASSWORD}
#gmysql-dnssec=yes
EOF

chown pdns: /etc/powerdns/pdns.d/pdns.local.gmysql.conf
chmod 640 /etc/powerdns/pdns.d/pdns.local.gmysql.conf

print_status "Enabling PowerDNS API..."
# Backup original config
cp /etc/powerdns/pdns.conf /etc/powerdns/pdns.conf.backup

# Add API configuration
cat >> /etc/powerdns/pdns.conf <<EOF

#################################
# API Configuration
#################################
api=yes
api-key=${PDNS_API_KEY}
webserver=yes
webserver-address=127.0.0.1
webserver-port=8081
webserver-allow-from=127.0.0.1,::1
EOF

print_status "Cleaning up bind.conf if it exists..."
if [ -f /etc/powerdns/pdns.d/bind.conf ]; then
    # Keep only the launch line
    sed -i '/^launch/!d' /etc/powerdns/pdns.d/bind.conf
fi

print_status "Starting PowerDNS..."
systemctl restart pdns
systemctl enable pdns

# Verify PowerDNS is running
if systemctl is-active --quiet pdns; then
    print_status "PowerDNS is running successfully"
else
    print_error "PowerDNS failed to start"
    systemctl status pdns
    exit 1
fi

## Assigning Version Number
PDNS_VERSION=$(pdns_server --version | grep "PowerDNS Authoritative Server" | awk '{print $4}')

#############################################
# POWERDNS-ADMIN DEPENDENCIES
#############################################

print_status "Installing PowerDNS-Admin build dependencies..."
apt install -y apt-transport-https build-essential curl git \
    libffi-dev libldap2-dev libmariadb-dev libpq-dev libsasl2-dev \
    libssl-dev libxml2-dev libxmlsec1-dev libxslt1-dev pkg-config \
    python3-dev python3-flask python3-venv virtualenv

print_status "Installing Node.js 20.x..."
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

print_status "Installing Yarn..."
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | tee /etc/apt/sources.list.d/yarn.list
apt update && apt install -y yarn

#############################################
# POWERDNS-ADMIN INSTALLATION
#############################################

print_status "Cloning PowerDNS-Admin repository..."
if [ -d "$PDNSADMIN_PATH" ]; then
    print_warning "PowerDNS-Admin directory already exists. Backing up..."
    mv "$PDNSADMIN_PATH" "${PDNSADMIN_PATH}.backup.$(date +%Y%m%d_%H%M%S)"
fi

git clone https://github.com/PowerDNS-Admin/PowerDNS-Admin.git "$PDNSADMIN_PATH"
cd "$PDNSADMIN_PATH"

print_status "Creating Python virtual environment..."
python3 -m venv ./venv

print_status "Installing Python dependencies..."
source ./venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
deactivate

#############################################
# POWERDNS-ADMIN CONFIGURATION
#############################################

print_status "Creating PowerDNS-Admin production configuration..."
cp configs/development.py configs/production.py

# Generate Flask secret key if not provided
if [ -z "$FLASK_SECRET_KEY" ]; then
    FLASK_SECRET_KEY=$(python3 -c 'import secrets; print(secrets.token_hex())')
    print_status "Generated Flask secret key: $FLASK_SECRET_KEY"
fi

# Update production config
sed -i "s/SQLA_DB_USER = .*/SQLA_DB_USER = '${DB_USER}'/" configs/production.py
sed -i "s/SQLA_DB_PASSWORD = .*/SQLA_DB_PASSWORD = '${DB_PASSWORD}'/" configs/production.py
sed -i "s/SQLA_DB_NAME = .*/SQLA_DB_NAME = '${PDNSADMIN_DB_NAME}'/" configs/production.py

# Update SQLAlchemy URI
sed -i "s|^SQLALCHEMY_DATABASE_URI = 'sqlite.*|# SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'pdns.db')\nSQLALCHEMY_DATABASE_URI = 'mysql://${DB_USER}:${DB_PASSWORD}@127.0.0.1/${PDNSADMIN_DB_NAME}'|" configs/production.py

# Add secret key if not already present
if ! grep -q "^SECRET_KEY = " configs/production.py; then
    echo "SECRET_KEY = '${FLASK_SECRET_KEY}'" >> configs/production.py
else
    sed -i "s/SECRET_KEY = .*/SECRET_KEY = '${FLASK_SECRET_KEY}'/" configs/production.py
fi

print_status "Initializing PowerDNS-Admin database..."
source ./venv/bin/activate
export FLASK_CONF=../configs/production.py
export FLASK_APP=powerdnsadmin/__init__.py
flask db upgrade

print_status "Building frontend assets..."
yarn install --pure-lockfile
flask assets build
deactivate

#############################################
# SYSTEMD SERVICE CONFIGURATION
#############################################

print_status "Creating systemd service files..."

cat > /etc/systemd/system/pdnsadmin.service <<EOF
[Unit]
Description=PowerDNS-Admin
After=network.target

[Service]
Type=simple
WorkingDirectory=${PDNSADMIN_PATH}
User=pdns
Group=pdns

Environment=FLASK_CONF=../configs/production.py

ExecStart=${PDNSADMIN_PATH}/venv/bin/python ${PDNSADMIN_PATH}/run.py

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

cat > /etc/systemd/system/pdnsadmin.socket <<EOF
[Unit]
Description=PowerDNS-Admin socket

[Socket]
ListenStream=/run/pdnsadmin/socket

[Install]
WantedBy=sockets.target
EOF

print_status "Setting up runtime directory..."
mkdir -p /run/pdnsadmin/
echo "d /run/pdnsadmin 0755 pdns pdns -" > /etc/tmpfiles.d/pdnsadmin.conf

print_status "Setting permissions..."
chown -R pdns: /run/pdnsadmin/
chown -R pdns: "$PDNSADMIN_PATH"

print_status "Enabling and starting PowerDNS-Admin service..."
systemctl daemon-reload
systemctl enable --now pdnsadmin.service pdnsadmin.socket

# Wait a few moments for the service to start
sleep 6

# Check if service is running
if systemctl is-active --quiet pdnsadmin.service; then
    print_status "PowerDNS-Admin is running successfully"
else
    print_error "PowerDNS-Admin failed to start"
    systemctl status pdnsadmin.service
fi

# Wait a few more moments to be sure the service to started and doesnt spew text to the terminal
sleep 6

#############################################
# INSTALLATION COMPLETE
#############################################

echo ""
echo "=========================================="
print_status "Installation Complete!"
echo "=========================================="
echo ""
echo "Next Steps:"
echo "  1. Visit PowerDNS-Admin web interface"
echo "     - http://$(hostname -I | awk '{print $1}'):9191"
echo "  2. Register your first user"
echo "  3. In PowerDNS-Admin setup, add your PowerDNS server:"
echo "     - API URL: http://127.0.0.1:8081"
echo "     - API Key: ${PDNS_API_KEY}"
echo "     - Version: ${PDNS_VERSION}"
echo ""
echo "----------------------------------------------"
echo "PowerDNS Configuration:"
echo "  - Database: ${PDNS_DB_NAME}"
echo "  - API URL: http://$(hostname -I | awk '{print $1}'):8081"
echo "  - API Key: ${PDNS_API_KEY}"
echo "  - Version: ${PDNS_VERSION}"
echo ""
echo "PowerDNS-Admin Configuration:"
echo "  - Web Interface: http://$(hostname -I | awk '{print $1}'):9191"
echo "  - Database: ${PDNSADMIN_DB_NAME}"
echo "  - Installation Path: ${PDNSADMIN_PATH}"
echo ""
echo "Database User:"
echo "  - Username: ${DB_USER}"
echo "  - Password: ${DB_PASSWORD}"
echo ""
echo "Service Status:"
echo "  - PowerDNS: $(systemctl is-active pdns)"
echo "  - PowerDNS-Admin: $(systemctl is-active pdnsadmin)"
echo ""
echo "=========================================="

```
