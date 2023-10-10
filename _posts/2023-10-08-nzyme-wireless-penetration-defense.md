---
title: nzyme - wireless monitoring system
date: 2023-10-08 11:33:00 -0700
categories: [Networking, WiFi]
tags: [security, wireless, WPA2, telemetry, hack]
pin: true
image:
  path: /assets/img/header/header--wireless--nzyme-wifi-penetration-defense.jpg
  alt: wireless penetration defense nzyme
---

# Monitor your Wireless Networks with nzyme

Detect and physically locate threats using an easy to build and deploy sensor system.


## Script

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
### Check distro, version, and if ran as root
#######################################################
read -d . VERSION < /etc/debian_version
if [[ "${VERSION[0]}" == "11" || "${VERSION[0]}" == "12" ]]; then
    if [ "$EUID" -ne 0 ]
         then echo "Please run with sudo permissions or as root"
         exit 1
    fi
else
  echo -e "Debian is not one of the required versions: 11 (Bullseye) or 12 (Bookworm).\nExiting the script.\n\n"
  exit 1
fi
###
#######################################################
### Architecture, IP, first WiFi device found, if none found assign foo
#######################################################
export MY_SYS_PROC_TYPE=$(dpkg --print-architecture)
export MY_IP=$(ip a | grep 'inet ' | awk '{print $2;}' | tail -n +2 | cut -d/ -f1)
export MY_WIFI=$(ip -br l | awk '$1 !~ "lo|vir|eth|ens|enp" { print $1}')
[ -z "$MY_WIFI" ] && export MY_WIFI=wlan-card-not-found
###
#######################################################
### Check if config file exists, possibly previously run -- ask if they need help
#######################################################
if [ -f "/etc/nzyme/nzyme.conf" ]; then
  echo "Is your configuration is not working... the config file exists."
  read -p "Would you like a last ditch help script? (yes/no): " help_response
  if [ "$help_response" = "yes" ]; then
    echo "Please remember, you can access the web interface at: http://$MY_IP"
    read -p "Do you want to run the nzyme-fix-it-on-reboot script? (yes/no): " fix_response
    if [ "$fix_response" = "yes" ]; then
      echo "Running the nzyme-fix-it-on-reboot script..."
      sudo printf "sleep 20; sudo systemctl stop nzyme; sudo systemctl status nzyme; sudo systemctl daemon-reload; sudo ifconfig $MY_WIFI down; sudo iwconfig $MY_WIFI mode monitor; sudo ifconfig $MY_WIFI up; sudo setcap cap_net_raw,cap_net_admin=eip /usr/lib/jvm/java-1.11.0-openjdk-$MY_SYS_PROC_TYPE/bin/java; sudo systemctl start nzyme; sudo systemctl status nzyme;\n" | sudo tee -a /etc/nzyme/nzyme-reboot.sh
      sudo chmod 755 /etc/nzyme/nzyme-reboot.sh
      crontab -l | { cat; echo "@reboot /etc/nzyme/nzyme-reboot.sh"; } | crontab -
      exit 0
    else
      echo "You can re-run this script anytime to reinstall or run the fix-it-on-reboot script."
      exit 0
    fi
  else
    echo "Please remember, you can access the web interface at: http://$MY_IP"
  fi
fi
###
#######################################################
### User input for db password / web password
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
### Wrap this as a function so we can control output later
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
### Take the function's output and send it somewhere else
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
### Congratulations, we're done!
#######################################################
echo -e "\nTest after reboot with: \ntail -n 200 /var/log/nzyme/nzyme.log"
echo -e "\n######################################\n## Install complete, reboot needed  ##\n######################################"
```
