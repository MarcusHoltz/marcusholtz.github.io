---
title: Using NAT and SPAN with Proxmox
date: 2022-12-30 11:33:00 -0700
categories: [Proxmox, ProxmoxNetworking]
tags: [virtualization, proxmox, networking, NAT, SPAN]
image:
  path: /assets/img/header/header--proxmox--proxmox-nat-and-span.jpg
  alt: Using NAT and SPAN with Proxmox
---


# NAT - Network Address Translation

In Proxmox, a NAT configuration can only be done through a CLI.

IP forwarding must be allowed in order for NAT to work. By default, it is not enabled. The following steps show how to enable IP forwarding, then configure NAT for a network interface:

Use a text editor to open the `/etc/sysctl.conf` file:

`nano /etc/sysctl.conf`

Uncomment the following line:

`net.ipv4.ip_forward=1`

`Save the file and reboot to activate the change.`

Open the network interface configuration with a text editor:

`nano /etc/network/interfaces`

Add and remove the configurations from an interface as follows:

```
auto vmbr0
iface vmbr0 inet static
address 192.168.10.1
netmask 255.255.255.0
bridge_ports none
bridge_stp off
bridge_fd 0

post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables –t nat –A POSTROUTING –s '192.168.10.0/24' –o eth0 –j MASQUERADE
post-down iptables –t nat –D POSTROUTING –s '192.168.10.0/24' –o eth0 –j MASQUERADE
```

Based on the configuration made in the previous section, all traffic on the vmbr0 virtual interface passes through eth0 , while all the IP information of VMs remains hidden behind the vmbr0 bridge.




Edit file: `/etc/default/ufw`


Adding: `DEFAULT_FORWARD_POLICY="ACCEPT"`





# SPAN port script

```
#!/bin/bash

# proxmox-seconiontap.sh

## What Interface Do You Want This to Dump to (mirror to)? ##
## :NOTE: ## Any traffic arriving on the interface you pick is dropped ##
## tl;dr - dont pick an interface you're using to connect with ##
## What Interface Do You Want This to Dump to (mirror to)? ##
SPANTO=tap100i2

## What bridge do you want to sniff on? ##
## :NOTE: ## The span interface above must reside on the bridge being sniffed ##
## What bridge do you want to sniff on? ##
BRIDGEMIRROR=vmbr0

SECONIONLOG=/root/network-ovs-mirror_creation.log

date >> $SECONIONLOG

echo "##############################" >> $SECONIONLOG

echo "Clearing any existing mirrors...\n"

ovs-vsctl clear bridge $BRIDGEMIRROR mirrors

echo "Sniffing on $BRIDGEMIRROR... dumping to $SPANTO\n"

# Command below taken from: https://docs.openvswitch.org/en/latest/faq/configuration/
ovs-vsctl -- --id=@p get port $SPANTO \
-- --id=@m create mirror name=span-of-$BRIDGEMIRROR-onto-$SPANTO select-all=true output-port=@p \
-- set bridge $BRIDGEMIRROR mirrors=@m

echo "\nBridge created above sucessfully, check logs for more information.\n"

ovs-vsctl list Mirror >> $SECONIONLOG

echo "##############################" >> $SECONIONLOG

```

