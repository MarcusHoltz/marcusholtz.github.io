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


* * * 

# Bond Mode 3

I know, it sounds like a film in theaters, but it's actually a Linux Bond Mode.


## Linux Bonds

A `bond` allows you take multiple interface and pretend it's one network interface.

* * *

Bonding Mode: 0-6

0 - round robin
1 - active backup
2 - xor (uses same source and dest pair for an interface)
3 - broadcast (sent through all interfaces at once)
4 - IEEE 802.3ad
5- adaptive transmit load balancing
6 - adaptive load balancing

* * * 



## Broadcast Bond

Linux Bond Mode 3, the broadcast bond. This can work functionally the same as a SPAN port, but it is not considered a full copy of the port, as it's `not a Layer 1` connection.

In terms of the OSI network model, both Linux Bond Mode 3 and SPAN ports operate at different layers:

1. **Linux Bond Mode 3 (Broadcast Mode)**:

   - **Layer**: Bond Mode 3 primarily operates at the **Data Link Layer (Layer 2)** of the OSI model.
   
   - **Explanation**: The Data Link Layer is responsible for the transfer of data between adjacent network nodes in a wide area network or between nodes on the same local area network segment. Linux Bond Mode 3, being a feature of the Linux bonding driver, operates at this layer by managing the transmission of data frames between network interfaces.

2. **SPAN Ports (Switched Port Analyzer)**:

   - **Layer**: SPAN ports operate primarily at the **Physical Layer (Layer 1)** and **Data Link Layer (Layer 2)** of the OSI model.
   
   - **Explanation**:
     - **Physical Layer (Layer 1)**: SPAN ports involve physical connections within the network switch or router, where traffic is mirrored or copied from source ports to a designated monitoring port.
     - **Data Link Layer (Layer 2)**: At this layer, SPAN ports deal with the transmission of data frames and the duplication of traffic from specific source ports to a monitoring port for analysis.

**Summary**:

- Linux Bond Mode 3 operates primarily at the Data Link Layer (Layer 2), handling the transmission of data frames between bonded network interfaces.
  
- SPAN ports operate at both the Physical Layer (Layer 1) and the Data Link Layer (Layer 2). They involve physical connections within the network switch or router (Layer 1) and duplicate traffic from specified source ports to a monitoring port for analysis (Layer 2).

While both features serve different purposes, their positioning within the OSI model reflects the layers at which they primarily function in the network architecture.

