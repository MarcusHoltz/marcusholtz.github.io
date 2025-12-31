---
title: nzyme - wireless monitoring system
date: 2023-10-08 11:33:00 -0700
categories: [Networking, WiFi]
tags: [security, wireless, WPA2, telemetry, hack]
pin: false
image:
  path: /assets/img/header/header--wireless--nzyme-wifi-penetration-defense.jpg
  alt: wireless penetration defense nzyme
---

# Monitor your Wireless Networks with nzyme

Detect and physically locate threats using an easy to build and deploy sensor system.

> The goal of nzyme is to be accessible to as many people as possible by being easy to understand, no matter your level of experience.


## Install nzyme with the v1 documentation -- and loose

There are a few gotchas in the [documentation](https://v1.nzyme.org/docs/intro) to get nzyme up and running. 

I would like the install and configuration of nzyme be as easy as it's use. The original goal of nzyme is to be accessible to as many people as possible -- install on a SoC, usb WiFi, and go. The install script below intends to fix that.


### How did I get this working?

The saving grace seems to be the [german libpcap version](https://ftp.uni-siegen.de/debian/debian-security/pool/updates/main/libp/libpcap/).


There is also the option to build from scratch (I was unable to get this version to work):

- https://github.com/nzymedefense/nzyme/discussions/339#discussioncomment-521229


* * *

## Install Wizard for nzyme - script below

### Setting up your system to be able to use nzyme

First you have to make sure to have a working WiFi USB card (read the [requirements](https://v1.nzyme.org/docs/next/installation-configuration/requirements)), and then pick how you're deploying this:


#### Raspberry pi, AML-S905X-CC (Le Potato), Old Laptop

If you're installing this on a SoC, it takes about `7min` to load the card with a debian/raspbian distribution, uncompress the system, reboot, and load the new system.

After the script below runs for an additional `13min+`, it should be a total of about `20min+`


* * * 


# Nzyme Install Wizard

There are a few gotchas in the nzyme v1 [documentation](https://v1.nzyme.org/docs/intro) to get up and running. 

I would like the install and configuration of nzyme be as easy as it's use. **This install script intends to fix that.**


## Install nzyme with this wizard

Use [the install script](https://raw.githubusercontent.com/MarcusHoltz/nzyme-install-wizard/main/nzyme-install-wizard.sh) to help guide you through the nzyme install process on Debian systems.

> The goal of nzyme is to be accessible to as many people as possible by being easy to understand, no matter your level of experience.


### Install with this command:

* * *

```bash
wget -O nzyme-install-wizard.sh https://raw.githubusercontent.com/MarcusHoltz/nzyme-install-wizard/main/nzyme-install-wizard.sh; chmod +x nzyme-install-wizard.sh; sudo bash nzyme-install-wizard.sh
```

* * *

![nzymeinstallwizard](/assets/img/posts/nzyme-install-wizard-script.gif)

* * *

## nzyme v1 install script

```bash
#!/bin/bash
###
########################################################
#####          What does this script do?           #####
########################################################
## Uses: https://v1.nzyme.org/                       ##
## Installs a copy of nzyme on a Debian distro       ##
#######################################################
## This script requires sudo or root for use         ##
#######################################################
## Setup dependencies for nzyme:                     ##
##    postgresql   wireless-tools   python3          ##
## Install specific versions required for use:       ##
## openjdk-11-jre-headless libpcap0.8_1.8.1-3+deb9u1 ##
#######################################################
#######################################################
######          B E G I N    S C R I P T         ######
#######################################################
### REQUIREMENTS: Check distro, version, and if ran as root
#######################################################
read -d . VERSION < /etc/debian_version
if [[ "${VERSION[0]}" == "11" || "${VERSION[0]}" == "12" ]]; then
    if [ "$EUID" -ne 0 ]
         then echo "Please run with sudo permissions or as root"
         exit 1
    fi
else
  echo -e "Debian is not on one of the required versions: 11 (Bullseye) or 12 (Bookworm).\nExiting the script.\n\n"
  exit 1
fi
###
#######################################################
### SET SYSTEM VALUES: Architecture, IP, first WiFi device found, if none found assign foo
#######################################################
export MY_SYS_PROC_TYPE=$(dpkg --print-architecture)
export MY_IP=$(ip a | grep 'inet ' | awk '{print $2;}' | tail -n +2 | cut -d/ -f1)
export MY_WIFI=$(ip -br l | awk '$1 !~ "lo|vir|eth|ens|enp" { print $1}')
[ -z "$MY_WIFI" ] && export MY_WIFI=wlan-card-not-found
###
#######################################################
### HELP SECTION: Check if config file exists, possibly previously run -- ask if they need help
#######################################################
if [ -f "/etc/nzyme/nzyme.conf" ]; then
    echo -e "You've re-run this script.\nWhat do you want to do now that nzyme is installed?"
    PS3='Please type one of the numbers above and hit enter: '
    options=("Reinstall nzyme." "Uninstall nzyme." "I am having trouble. Please setup the nzyme-fix-it-on-reboot script." "Display what IP address nzyme is hosted at." "Read the nzyme logs." "Quit this laucher")
    select opt in "${options[@]}"
    do
        case $opt in
            "Reinstall nzyme.")
                echo -e -n "\nOk then,\nREINSTALLING NZYME....";
                    for ((i=1; i<=18; i++)); do
                        echo -n "."
                        sleep 0.35
                    done
                echo -e "\n\n"
            break
                ;;
            "Uninstall nzyme.")
                echo -e "#####################################\nUNINSTALLING  NZYME  AND  COMPONENTS\n#####################################"
                sudo apt remove -y nzyme postgresql libpcap0.8 openjdk-11-jre-headless > /dev/null 2>&1
                sudo rm -rf /etc/nzyme/* > /dev/null 2>&1
                sudo rm /etc/apt/preferences.d/openjdk-pin /etc/apt/preferences.d/libpcap > /dev/null 2>&1
                echo -e "\nUNINSTALL COMPLETE\nYou can re-run this script anytime to reinstall."
                exit 0
            break
                ;;
            "I am having trouble. Please setup the nzyme-fix-it-on-reboot script.")
                echo "Setting up the nzyme-fix-it-on-reboot script..."
                sudo printf "sleep 20; sudo systemctl stop nzyme; sudo systemctl status nzyme; sudo systemctl daemon-reload; sudo ifconfig $MY_WIFI down; sudo iwconfig $MY_WIFI mode monitor; sudo ifconfig $MY_WIFI up; sudo setcap cap_net_raw,cap_net_admin=eip /usr/lib/jvm/java-1.11.0-openjdk-$MY_SYS_PROC_TYPE/bin/java; sudo systemctl start nzyme; sudo systemctl status nzyme;\n" | sudo tee -a /etc/nzyme/nzyme-reboot.sh > /dev/null 2>&1
                sudo chmod 755 /etc/nzyme/nzyme-reboot.sh
                crontab -l | { cat; echo "@reboot /etc/nzyme/nzyme-reboot.sh"; } | crontab -
                echo -e "\nYou will need to add the following REBOOT crontab for the script to take effect on next reboot.\n"
                echo "Copy and Paste this line:"
                echo -e "crontab -l | { cat; echo "@reboot /etc/nzyme/nzyme-reboot.sh"; } | crontab -"
                exit 0
            break
                ;;
            "Display what IP address nzyme is hosted at.")
                echo -e "\nPlease remember, you can access the web interface at: http://$MY_IP"; sleep 2;
                exit 0
            break
                ;;
            "Read the nzyme logs.")
                tail -n 200 /var/log/nzyme/nzyme.log
                exit 0
            break
                ;;
            "Quit this laucher")
                exit 0
            break
                ;;
            *) echo "invalid option $REPLY";;
        esac
    done
fi
###
#######################################################
### USER INPUT: db password / web password
#######################################################
echo -e "**************************************************************************\n     Welcome to nzyme installer, please follow instructions below\n**************************************************************************\nAnswer each prompt to generate the config.\n----------------------------------------------------"
sleep 1;
echo -e "Enter password for 'admin' on http web interface ($MY_IP):"
read -s data_admin_password
export data_admin_password_hash=$(echo -n $data_admin_password | sha256sum | cut -d ' ' -f1)
echo -e "\n(this next password can be anything you like, you wont need to enter it again)\nEnter password for backend postgresql database:"
read -s data_postgres_password
sleep .5;
echo -e "***********************************************************************************\n     Begin Install      -=Nzyme config questions complete=-     Up to 15min wait    \n***********************************************************************************"
sleep 1;
echo -e "##################################################\n##  Once install is complete, reboot is needed  ##\n##################################################"
###
#######################################################
### MAIN FUNCTION: Wrap this as a function so we can control output later
#######################################################
main_function() {
### Update and install requirements
    sudo apt update && sudo apt upgrade -y && sudo apt install -y wireless-tools python3
    sudo ln -s /usr/bin/python3 /usr/bin/python
    sudo apt install -y postgresql libpcap0.8
### Install Java based on distro version
    read -d . VERSION < /etc/debian_version
    if [[ "${VERSION[0]}" == "12" ]]; then
        printf "deb http://deb.debian.org/debian oldstable main" | sudo tee -a /etc/apt/sources.list
        sudo apt update
        sudo touch /etc/apt/preferences.d/openjdk-pin
        printf "Package: openjdk-11-jre-headless\nPin: release n=oldstable\nPin-Priority: 1001" | sudo tee /etc/apt/preferences.d/openjdk-pin
        sudo apt-cache policy openjdk-11-jre-headless
    fi
    sudo apt install -y openjdk-11-jre-headless
### Re-install this libpcap version that allows monitor mode to work
    sudo apt purge -y libpcap0.8
    wget https://ftp.uni-siegen.de/debian/debian-security/pool/updates/main/libp/libpcap/libpcap0.8_1.8.1-3%2Bdeb9u1_$MY_SYS_PROC_TYPE.deb
    sudo dpkg -i libpcap0.8_*
    sudo rm libpcap0.8_*
    sudo touch /etc/apt/preferences.d/libpcap
    printf "Package: libpcap0.8\nPin: version 1.8.1-3*\nPin-Priority: 999" | sudo tee /etc/apt/preferences.d/libpcap
    sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
### Install and enable latest Version 1 release of nzyme
    wget https://assets.nzyme.org/releases/nzyme-1.2.2.deb
    sudo dpkg -i nzyme-1.2*.deb
    sudo rm nzyme-1*.deb
    sudo apt --fix-broken install
    sudo systemctl enable nzyme
### Set up the database with the default db/user with user's password
    sudo -u postgres psql -c "create database nzyme;"
    sudo -u postgres psql -c "create user nzyme with encrypted password '$data_postgres_password';"
    sudo -u postgres psql -c "grant all privileges on database nzyme to nzyme;"
    sudo -u postgres psql -d nzyme -c "GRANT ALL ON schema public TO nzyme"
    sudo -u postgres psql -d nzyme -c "GRANT ALL on all tables in schema public TO postgres;"
    sudo -u postgres psql -d nzyme -c "GRANT ALL on all tables in schema public TO nzyme;"
    sudo -u postgres psql -c "\c nzyme"
    sudo -u postgres psql -c "GRANT ALL ON SCHEMA public TO nzyme;"
### Configure the required vaules in the .conf to work
    sudo cp /etc/nzyme/nzyme.conf.example /etc/nzyme/nzyme.conf
    sudo sed -i "s/admin_password_hash:.*/admin_password_hash: $data_admin_password_hash/" /etc/nzyme/nzyme.conf
    sudo sed -i "s/YOUR_PASSWORD/$data_postgres_password/" /etc/nzyme/nzyme.conf
    sudo sed -i 's/python3\.8/python/' /etc/nzyme/nzyme.conf
    sudo sed -i "s/rest_listen_uri:.*/rest_listen_uri: \"http:\/\/$MY_IP:80\/\"/" /etc/nzyme/nzyme.conf
    sudo sed -i "s/http_external_uri:.*/http_external_uri: \"http:\/\/$MY_IP:80\/\"/" /etc/nzyme/nzyme.conf
    sudo sed -i "s/wlx00c0ca971201.*/$MY_WIFI/" /etc/nzyme/nzyme.conf
}
###
#######################################################
### MANAGE TERMINAL OUTPUT: Take the function's output and send it somewhere else
#######################################################
if [ -z $TERM ]; then
  # if not run via terminal, log everything into a log file
  main_function 2>&1 >> /var/log/nzyme/script_for_nzyme.log
else
  # if run via terminal, DONT output to screen
  main_function > /dev/null 2>&1
fi
###
#######################################################
### FIN: Congratulations, we're done!
#######################################################
echo -e "\nTest after reboot with: \ntail -n 200 /var/log/nzyme/nzyme.log"
echo -e "\n######################################\n## Install complete, reboot needed  ##\n######################################"
```

* * *

## Nzyme 2.0 alpha

I also have a quick makeshift script for the 2.0 alpha, it's not as complete as the script above, and the version is hardcoded.


```bash
#!/bin/bash
set -e

# Configuration file paths
NZYME_CONF="/etc/nzyme/nzyme.conf"
TAP_CONF="/etc/nzyme/nzyme-tap.conf"

# Get current IP address (excluding localhost)
CURRENT_IP=$(hostname -I | awk '{print $1}')
DEFAULT_REST_URI="https://${CURRENT_IP}:22900/"
DEFAULT_HTTP_URI="https://${CURRENT_IP}:22900/"

clear
echo "========================================="
echo "nzyme Installation Script for Ubuntu 24.04"
echo "========================================="
echo

# Step 1: Install dependencies
echo "Step 1/9: Installing dependencies..."
sudo apt update
sudo apt install -y openjdk-17-jre-headless postgresql-16 wget curl
echo "✓ Dependencies installed"
echo

# Step 2: Download and install nzyme-node
echo "Step 2/9: Downloading and installing nzyme-node..."
wget -q -O /tmp/nzyme-node.deb https://github.com/nzymedefense/nzyme/releases/download/2.0.0-alpha.17/nzyme-node_ubuntu-2404noble-noarch-2.0.0-alpha.17.deb
sudo dpkg -i /tmp/nzyme-node.deb
echo "✓ nzyme-node installed"
echo

# Step 3: Configure PostgreSQL
clear
echo "Step 3/9: Configuring PostgreSQL..."
echo
read -sp "Enter password for PostgreSQL 'nzyme' user: " NZYME_DB_PASS
echo
echo

sudo -u postgres psql <<EOF
CREATE DATABASE nzyme;
DO \$\$
BEGIN
   IF NOT EXISTS (SELECT FROM pg_catalog.pg_roles WHERE rolname = 'nzyme') THEN
      CREATE USER nzyme WITH ENCRYPTED PASSWORD '$NZYME_DB_PASS';
   END IF;
END
\$\$;
GRANT ALL PRIVILEGES ON DATABASE nzyme TO nzyme;
\c nzyme
GRANT CREATE ON SCHEMA public TO nzyme;
EOF
echo "✓ PostgreSQL configured"
echo

# Step 4: Configure nzyme.conf
clear
echo "========================================="
echo "Step 4/9: Configuring nzyme.conf"
echo "========================================="
echo

# Backup the config file
sudo cp "$NZYME_CONF" "${NZYME_CONF}.bak.$(date +%s)"

# Prompt for configuration values
echo "Enter nzyme node name"
read -p "[Default: nzyme-node-01]: " NZYME_NAME
NZYME_NAME=${NZYME_NAME:-nzyme-node-01}
echo

echo "Enter rest_listen_uri (must start with https://)"
read -p "[Default: $DEFAULT_REST_URI]: " REST_LISTEN
REST_LISTEN=${REST_LISTEN:-$DEFAULT_REST_URI}
while [[ ! "$REST_LISTEN" =~ ^https:// ]]; do
    echo "Error: URL must start with https://"
    read -p "Enter rest_listen_uri: " REST_LISTEN
done
echo

echo "Enter http_external_uri (must start with https://)"
read -p "[Default: $DEFAULT_HTTP_URI]: " HTTP_EXTERNAL
HTTP_EXTERNAL=${HTTP_EXTERNAL:-$DEFAULT_HTTP_URI}
while [[ ! "$HTTP_EXTERNAL" =~ ^https:// ]]; do
    echo "Error: URL must start with https://"
    read -p "Enter http_external_uri: " HTTP_EXTERNAL
done
echo

# Edit nzyme.conf in-place using sed
DB_URL="postgresql://localhost:5432/nzyme?user=nzyme&password=$NZYME_DB_PASS"

# Update general.name
sudo sed -i "s|^  name:.*|  name: $NZYME_NAME|" "$NZYME_CONF"

# Update general.database_path - escape special characters
DB_URL_ESCAPED=$(echo "$DB_URL" | sed 's/[&/\]/\\&/g')
sudo sed -i "s|^  database_path:.*|  database_path: \"$DB_URL_ESCAPED\"|" "$NZYME_CONF"

# Update interfaces.rest_listen_uri
REST_LISTEN_ESCAPED=$(echo "$REST_LISTEN" | sed 's/[&/\]/\\&/g')
sudo sed -i "s|^  rest_listen_uri:.*|  rest_listen_uri: \"$REST_LISTEN_ESCAPED\"|" "$NZYME_CONF"

# Update interfaces.http_external_uri
HTTP_EXTERNAL_ESCAPED=$(echo "$HTTP_EXTERNAL" | sed 's/[&/\]/\\&/g')
sudo sed -i "s|^  http_external_uri:.*|  http_external_uri: \"$HTTP_EXTERNAL_ESCAPED\"|" "$NZYME_CONF"

echo "✓ nzyme.conf configured"
echo

# Step 5: Run database migrations
echo "Step 5/9: Running database migrations..."
sudo nzyme --migrate-database
echo "✓ Database migrations complete"
echo

# Step 6: Enable and start nzyme-node
echo "Step 6/9: Enabling and starting nzyme-node service..."
sudo systemctl enable nzyme
sudo systemctl restart nzyme

echo "✓ nzyme-node service started"
echo
echo "Waiting 10 seconds for nzyme-node to start up..."
sleep 10

# Check if service is running
if sudo systemctl is-active --quiet nzyme; then
    echo "✓ nzyme-node is running"
else
    echo "⚠ Warning: nzyme-node may not be running. Check: sudo systemctl status nzyme"
fi
echo

# Step 7: Wait for user to create tap in web interface
clear
echo "========================================="
echo "IMPORTANT: Manual Step Required"
echo "========================================="
echo
echo "1. Open $HTTP_EXTERNAL in your browser"
echo "2. Complete the initial setup and create your first user"
echo "3. Navigate to: System -> Taps"
echo "4. Create a new tap and click on the name after creation."
echo "5. On this screen you need to Show your Tap Secret and copy it."
echo
echo "Press Enter after you have copied the leader tap secret..."
read -p "" dummy
echo

# Step 8: Get tap configuration
clear
echo "========================================="
echo "Step 7/9: Configuring nzyme-tap"
echo "========================================="
echo
echo "Paste the tap secret from the web interface:"
read -sp "" LEADER_SECRET
echo
echo

echo "Enter leader URI (address of your nzyme-node)"
read -p "[Default: $HTTP_EXTERNAL]: " LEADER_URI
LEADER_URI=${LEADER_URI:-$HTTP_EXTERNAL}
echo

echo "Accept insecure/self-signed TLS certificates?"
read -p "[Default: true]: " ACCEPT_INSECURE
ACCEPT_INSECURE=${ACCEPT_INSECURE:-true}
echo

# Detect network interfaces
echo "Detecting network interfaces..."
echo
sleep 1

# Get all ethernet interfaces (excluding lo, docker, veth, etc.)
ETHERNET_IFACES=$(ip -o link show | awk -F': ' '{print $2}' | grep -v '^lo$\|^docker\|^veth\|^br-\|^virbr' | grep -v '^wl')

# Get all wifi interfaces (typically start with wl or wlx)
WIFI_IFACES=$(ip -o link show | awk -F': ' '{print $2}' | grep '^wl')

echo "Found network interfaces:"
if [ -n "$ETHERNET_IFACES" ]; then
    echo "  Ethernet: $ETHERNET_IFACES"
else
    echo "  Ethernet: None detected"
fi

if [ -n "$WIFI_IFACES" ]; then
    echo "  WiFi: $WIFI_IFACES"
else
    echo "  WiFi: None detected"
fi
echo
sleep 3

# Download and install nzyme-tap
echo
echo "Step 8/9: Downloading and installing nzyme-tap..."
wget -q -O /tmp/nzyme-tap.deb https://github.com/nzymedefense/nzyme/releases/download/2.0.0-alpha.17/nzyme-tap_ubuntu-2404noble-amd64-2.0.0-alpha.17.deb
sudo dpkg -i /tmp/nzyme-tap.deb
echo "✓ nzyme-tap installed"
echo
sleep 2

# Backup tap config if it exists
if [ -f "$TAP_CONF" ]; then
    sudo cp "$TAP_CONF" "${TAP_CONF}.bak.$(date +%s)"
fi

# Edit nzyme-tap.conf in-place
LEADER_SECRET_ESCAPED=$(echo "$LEADER_SECRET" | sed 's/[&/\]/\\&/g')
LEADER_URI_ESCAPED=$(echo "$LEADER_URI" | sed 's/[&/\]/\\&/g')

sudo sed -i "s|^leader_secret = .*|leader_secret = \"$LEADER_SECRET_ESCAPED\"|" "$TAP_CONF"
sudo sed -i "s|^leader_uri = .*|leader_uri = \"$LEADER_URI_ESCAPED\"|" "$TAP_CONF"
sudo sed -i "s|^accept_insecure_certs = .*|accept_insecure_certs = $ACCEPT_INSECURE|" "$TAP_CONF"

sudo sed -i "s|\[ethernet_interfaces\.enp6s0\]|\[ethernet_interfaces.$ETHERNET_IFACES\]|" "$TAP_CONF"
sudo sed -i "s|\[wifi_interfaces\.wlx00c0ca000000\]|\[wifi_interfaces.$WIFI_IFACES\]|" "$TAP_CONF"

# Comment out [wifi_interfaces.wlx00c0ca000001] and following 12 lines
sudo sed -i "/^\[wifi_interfaces\.wlx00c0ca000001\]/,+12 s/^/# /" "$TAP_CONF"


echo "✓ nzyme-tap.conf configured"
echo
sleep 3

# Step 9: Enable and start nzyme-tap
clear
echo "========================================="
echo "Step 9/9: Starting nzyme-tap service"
echo "========================================="
echo
sudo systemctl enable nzyme-tap
sudo systemctl restart nzyme-tap

echo "✓ nzyme-tap service started"
echo

# Final status check
clear
echo "========================================="
echo "Installation Complete!"
echo "========================================="
echo
echo "Service Status:"
sudo systemctl status nzyme --no-pager -l | head -n 5
echo
sudo systemctl status nzyme-tap --no-pager -l | head -n 5
echo
echo "Useful Commands:"
echo "  - View nzyme-node logs: sudo journalctl -u nzyme -f"
echo "  - View nzyme-tap logs:  sudo journalctl -u nzyme-tap -f"
echo "  - Check service status: sudo systemctl status nzyme"
echo "  - Check tap status:     sudo systemctl status nzyme-tap"
echo
echo "Access the web interface at: $HTTP_EXTERNAL"
echo "========================================="
```

* * *

### Nzyme 2.0 alpha caveats

According to Nzyme's release notes, the system has configurable session timeouts with defaults that are Nzyme "fairly aggressive".

Meaning, they're such poor defaults you cannot even use Nzyme as a dashboard - you'll be kicked off.

```sql
nzyme=# SELECT * FROM auth_tenants;
 id |      name      |       description        |         created_at         |         updated_at         |                 uuid                 |           organization_id            | session_timeout_minutes | session_inactivity_timeout_minutes | mfa_timeout_minutes 
----+----------------+--------------------------+----------------------------+----------------------------+--------------------------------------+--------------------------------------+-------------------------+------------------------------------+---------------------
  1 | Default Tenant | The nzyme default tenant | 2025-10-24 17:54:00.674+00 | 2025-10-24 17:54:00.674+00 | 65a61318-de18-452e-a76d-f1a62263db7e | 97990277-55a9-461a-93df-509f0327d96a |                     720 |                                 15 |                   5
(1 row)
```


* * *

#### Nzyme 2.0 alpha token/session is expiring

Nzyme has a session timeout issue. **User experience is expected to end in 15min.**


* * *

##### Official Nzyme 2.0 alpha sorry about the token/session expiring fix

So let's see if we cannot fix this according to how Lennart wanted you to:

Login to Nzyme:
`Authentication & Authorization > Organizations > Default Organization > Tenants > Default Tenant > Edit > Additional options`

Make changes to browser session timeouts in the "Edit Tenant" screen in the collapsed section under "Additional options".

* * *

Yup, that's right. It's as per documentation, errr, well no - it's not documented, it mentioned in a blog post: 
https://www.nzyme.org/news/2023/12/08/release-v200-alpha-6

Congrats, you're supposed to now have gained a system that wont boot you in 15min, but it still boots me. 

So I guess, alpha. Glad I documented this fix.


* * *

#### Fix Nyme 2.0 alpha token/session is expiring in the database

Not keen for a UI fix? Here's the database entries:

```bash
sudo -u postgres psql nzyme
SELECT * FROM auth_tenants;
UPDATE auth_tenants SET session_timeout_minutes = 7200;
UPDATE auth_tenants SET session_inactivity_timeout_minutes = 7255;
\q
```

