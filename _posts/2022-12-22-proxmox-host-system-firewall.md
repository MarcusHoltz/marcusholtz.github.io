---
title: Proxmox Server Host System's Firewall
date: 2022-12-22 11:33:00 -0700
categories: [Proxmox, ProxmoxNetworking]
tags: [virtualization, proxmox, networking, bridge, firewall, fail2ban]
image:
  path: /assets/img/header/header--proxmox--proxmox-server-host-firewall.jpg
  alt:  Host System Firewall
---

# Proxmox Networking Security


## Firewall

Install UFW & Fail2Ban:

`sudo apt install ufw fail2ban`

:NOTE: Be sure you have installed ifupdown2 before here, or it may brake the connection.

* * * 

In a basic firewall, denying all incoming traffic and allowing outgoing traffic is a good place to start:

`ufw default deny incoming`

`ufw default allow outgoing`



Next, you want to open up any services you wish to be available to the internet:

`sudo ufw allow 47979/tcp comment 'tcp port 47979 for SSH'`

`sudo ufw allow 38365/udp comment 'udp port 38365 for OpenVPN'`

`sudo ufw allow 8006/tcp  comment 'tcp port 8006 over HTTPS for Proxmox'`

`sudo ufw enable`

`sudo ufw status numbered`

`sudo systemctl enable ufw`

Verify with: 

`lsof -i`

* * *

This is just the start. You will need to open many more ports used by Proxmox:

See the [Proxmox VE Administration Guide -- Ports used by Proxmox VE](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_ports_used_by_proxmox_ve)

* * *


### Problems with the Firewall? 

Disable:

`ufw disable`


Wipe:

`ufw reset`


* * *

### What's going on now that there's a wall up?

To see what services are running behind the firewall on local ports:

`sudo ss`

`sudo ss -anpt`

-s all sockets (listening & connected)

-n shows port numbers (not just service names)

-p process

-t tcp sockets (HTTPS/SSH)


`sudo nmap -sS <public ip address>`







## Fail2Ban

Fail2Ban's config resides in : `/etc/fail2ban/fail2ban.conf` BUT

Place your config inside of `/etc/fail2ban/jail.local` it takes precedence.


### Fail2Ban config

Add the following:

```
echo "# UFW is our Firewall. By default, it will use iptables to ban IPs ">>/etc/fail2ban/jail.local
echo "[DEFAULT]">>/etc/fail2ban/jail.local
echo "banaction = ufw">>/etc/fail2ban/jail.local
echo -n "ignoreip = 127.0.0.1/8 ">>/etc/fail2ban/jail.local
getent hosts mydomain.lan | awk '{ print $1 }'>>/etc/fail2ban/jail.local
echo "# Fail2Ban configuration fragment for Open-SSHD">>/etc/fail2ban/jail.local
echo "[sshd]">>/etc/fail2ban/jail.local
echo "enabled = true">>/etc/fail2ban/jail.local
echo "banaction = iptables-multiport">>/etc/fail2ban/jail.local
echo "port = 47979">>/etc/fail2ban/jail.local
echo "logpath = /var/log/auth.log">>/etc/fail2ban/jail.local
echo "maxretry = 10">>/etc/fail2ban/jail.local
echo "findtime = 43200">>/etc/fail2ban/jail.local
echo "bantime = 86400">>/etc/fail2ban/jail.local
echo "# Fail2Ban configuration fragment for Proxmox">>/etc/fail2ban/jail.local
echo "[proxmox]">>/etc/fail2ban/jail.local
echo "enabled = true">>/etc/fail2ban/jail.local
echo "port = https,http,8006">>/etc/fail2ban/jail.local
echo "filter = proxmox">>/etc/fail2ban/jail.local
echo "logpath = /var/log/daemon.log">>/etc/fail2ban/jail.local
echo "maxretry = 5">>/etc/fail2ban/jail.local
echo "findtime = 43200">>/etc/fail2ban/jail.local
echo "bantime = 86400">>/etc/fail2ban/jail.local
echo "# Fail2Ban configuration fragment for OpenVPN">>/etc/fail2ban/jail.local
echo "[openvpn]">>/etc/fail2ban/jail.local
echo "enabled  = true">>/etc/fail2ban/jail.local
echo "port     = 38365">>/etc/fail2ban/jail.local
echo "protocol = udp">>/etc/fail2ban/jail.local
echo "filter   = openvpn">>/etc/fail2ban/jail.local
echo "logpath  = /var/log/openvpn.log">>/etc/fail2ban/jail.local
echo "maxretry = 5">>/etc/fail2ban/jail.local
echo "findtime = 43200">>/etc/fail2ban/jail.local
echo "bantime = 86400">>/etc/fail2ban/jail.local
echo "[Definition]">>/etc/fail2ban/filter.d/proxmox.conf
echo "failregex = pvedaemon\[.*authentication is now a failure; rhost=<HOST> user=.* msg=.*">>/etc/fail2ban/filter.d/proxmox.conf
echo "ignoreregex =">>/etc/fail2ban/filter.d/proxmox.conf
echo "[INCLUDES]">>/etc/fail2ban/filter.d/openvpn.conf
echo "before = common.conf">>/etc/fail2ban/filter.d/openvpn.conf
echo "[Definition]">>/etc/fail2ban/filter.d/openvpn.conf
echo "failregex =%(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} TLS Auth Error:.*">>/etc/fail2ban/filter.d/openvpn.conf
echo "           %(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} VERIFY ERROR:.*">>/etc/fail2ban/filter.d/openvpn.conf
echo "           %(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} TLS Error: TLS handshake failed.*">>/etc/fail2ban/filter.d/openvpn.conf
echo "           %(__hostname)s ovpn-server.*: TLS Error: cannot locate HMAC in incoming packet from \[AF_INET\]<HOST>:[0-9]{4,5}">>/etc/fail2ban/filter.d/openvpn.conf
echo "# Final Gotcha - ">>/etc/fail2ban/jail.local
echo "# UFW Firewall with Fail2Ban will only work if in /etc/fail2ban/action.d you have a file called ufw.conf">>/etc/fail2ban/jail.local
echo "# https://raw.githubusercontent.com/fail2ban/fail2ban/0.11/config/action.d/ufw.conf">>/etc/fail2ban/jail.local
```

#### openvpn.conf, proxmox.conf were created inside of /etc/fail2ban/filter.d/

Fire up fail2ban:

`systemctl start fail2ban`

`sudo systemctl enable fail2ban`

`sudo systemctl restart fail2ban`


[Explaining how to use Fail2Band regex for anything in your logs](https://www.linode.com/docs/security/using-fail2ban-to-secure-your-server-a-tutorial/)


* * *

### Fail2Ban Recovery

Lockout Recovery:

In the event that you find yourself locked out of your server due to fail2ban, you can still gain access. To do this, enter the following command:

`iptables -n -L`

Look for your IP address in the source column of any fail2ban chains. 

To remove your IP address from a jail, you can use the following command, replacing IP and jailname with the IP address and name of the jail that you’d like to unban:


`fail2ban-client set jailname unbanip 192.168.4.5`

or just stop the service `fail2ban-client stop`


* * *

##### More Fun Firewall Ideas

Dont even DROP or REJECT packets. Just accept them all. Attackers will have no clue what is actually open and what is not -- unless they manually identify the connections.

Realistically, you could probably get away with accepting the connection for unknown ports, blackholing the data, then closing the connection after a timeout. However, that will cause problems with legitimate users if their application makes a connection to one of these ports.




