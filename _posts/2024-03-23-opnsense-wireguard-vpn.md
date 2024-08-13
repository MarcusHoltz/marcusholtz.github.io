---
layout: post
title: OPNsense as a WireGuard Server with Site-to-Site VPN
date: 2024-03-23 11:33:00 -0700
categories: [Networking, VPN]
tags: [opnsense, workstation, splitdns, vpn, wireguard, remote]
pin: false
image:
  path: /assets/img/header/header--opnsense--wireguard-VPN.jpg
  alt: Setting up OPNsense router as a WireGuard VPN
---

# WireGuard on OPNsense 

There are multiple scenarios where WireGuard can be used, but they all require different configs for that setup.

This writeup should enable a single user (Roadwarrior) and further along the line, a Site-to-Site VPN.


## WireGuard then OPNsense

This tutorial discusses the setup of WireGuard first.

Please **hang on** till we're done with WireGuard.

**None of this will work** without the client, peer setup, and also changing the **OPNsense router and firewall**.


### Tutorial Steps

1. [This Intro](#wireguard-then-opnsense)

2. [Client Device Setup](#wireguard-on-the-peers-client-machine)

3. WireGuard

   1. [WireGuard Configuration](#wireguard-tunnel-subnet-interface)

   2. [WireGuard Site-to-Site VPN](#setting-up-wireguard-on-each-instance-of-opnsense-for-site-to-site)

4. OPNsense

   1. [WireGuard Interface](#configure-a-wireguard-interface-on-opnsense)
   
   2. [OPNsense rules for WireGuard](#create-a-wireguard-outbound-nat-rule)
      
      1. [Outbound NAT](#create-a-wireguard-outbound-nat-rule)
      
      2. [Inbound WAN](#create-firewall-rules-on---wan)

      3. [WireGuard Group](#create-firewall-rules-on---wireguard)

5. Site-to-Site VPN Router/Firewall

   1. [OPNsense Firewall configuration](#site-to-site-wireguard-wan-connection)

   2. [OPNsense Router configuration](#routing-different-subnets-across-wireguard)

6. [Getting the Peer Client's Device Connected](#setting-up-the-client-software)


* * *

## WireGuard intro

To setup WireGuard first, you must understand it's conceptual overview.


1. It's not a client/server VPN setup.

2. **Both sides are a peer.** 

3. So, in essence - you need to setup private and public keys for each side. 
    (*Technically you will derive the public key from the private key.*)

4. No dynamic IP assignment (or very little), each client has a fixed IP.

5. WireGuard associates tunnel IP addresses with public keys and remote endpoints. 


### Cryptokey Routing

At the heart of WireGuard is a concept called `Cryptokey Routing`, which works by associating `public keys` with a list of tunnel `IP addresses` that are `allowed` inside the WireGuard tunnel. 

**Each WireGuard network interface has a private key and a list of peers.**


* * *

## WireGuard conceptual overview example

* * * 


### Someone sending a packet


1. When `WireGuard` needs to `send` a packet, it looks for the IP address in its Local Addresses Subnet. 

2. Let's say, this packet is meant for `192.168.30.8`. 

3. Your computer's WireGuard `looks` for which peer that is. 

4. Okay, it's for peer `ABCDEFGH`. 
(Or if it's not for any configured peer, drop the packet.)

5. `Encrypt` entire IP packet using peer **ABCDEFGH**'s `public key`.

6. Now where do I `send` peer **ABCDEFGH**'s encrypted packet?

7. What remote `endpoint` was listed for peer **ABCDEFGH**?

8. The endpoint is `listed` as *216.58.211.110* port *53133*.

9. Send encrypted data over the `Internet` to *216.58.211.110:53133*.


* * *

### WireGuard receiving a packet

1. When an interface for `WireGuard` `receives` a packet, this could be from port forwarding or an open interface, it attempts to identify it.

2. The WireGuard interface checks the source IP address and port to determine `which` `peer` the packet is from.

3. Once the peer is identified, WireGuard looks up the corresponding `key` associated with that peer from its internal configuration. 

4. It then begins to `decrypt` the information that was sent.

5. WireGuard then uses **ABCDEFGH**'s key to verify the `authenticity` of the packet. 

6. With the accepted, `verified` packet - WireGuard will now remember that peer's most recent Internet `endpoint`.

7. Once all that has been completed, WireGuard will `look` at the packet.

8. It can see the `plain-text` packet from someone on the *192.168.30.X* subnet.

9. A final verification takes place as WireGuard looks at the packet from *192.168.30.X* to verify that `IP` is even `allowed` to be sending us packets.

10. If the `association` is successful, the packets are `allowed` to pass through the VPN tunnel.


* * * 

#### Summarize WireGuard routing

```text
Some App -> WireGuard -> Destination IP in tunnel -> Public key for peer holding that IP -> peer's most recent Internet endpoint
```


* * *

# PART II - The client device setup

* * * 

## WireGuard on the peer's client machine

**WireGuard is an exchange of keys.** Your OPNsense firewall's WireGuard cannot connect with a peer it doesnt have a key for. So, go ahead and skip this step if you're doing a [WireGuard Site-to-Site VPN](#wireguard-tunnel-subnet-interface).

There are many ways to do this, we'll use **COPY/PASTE**.


### What is your machine?

This is the WireGuard `peer client`'s software `connecting` back to `OPNsense`. Here's a few that I can mention:


* * * 

#### Android

- [The official WireGuard app for Android](https://apt.izzysoft.de/fdroid/index/apk/com.wireguard.android) - The official app includes an auto-updater, this is against F-Droid policy, and you will find this app at the IzzyOnDroid Repository. 

- [WireGuard Tunnel](https://f-droid.org/packages/com.zaneschepke.wireguardautotunnel/) - An alternative client app for WireGuard with additional features, available on F-Droid.


* * * 

#### iOS

- [The official WireGuard iOS App Store app](https://apps.apple.com/us/app/wireguard/id1441195209) - iOS's official WireGuard app for iOS 15.0 or later. Mostly feature parity with Android.


* * * 

#### MacOS

- [The official WireGuard MacOS App Store app](https://apps.apple.com/us/app/wireguard/id1451685025) - Apple's Mac App Store's WireGuard app for macOS 12.0 or later. This app allows users to manage and use WireGuard tunnels.


* * * 

#### Windows

- [The official WireGuard for Windows app](https://www.wireguard.com/install/) - [This installer](https://download.wireguard.com/windows-client/wireguard-installer.exe) is the only official and recommended way of using WireGuard on Windows.


* * * 

#### Linux

- [KDE](https://userbase.kde.org/System_Settings/Connections/en) - Since Plasma 5.15, Plasma support WireGuard VPN tunnels, when the appropriate Network Manager plugin is installed.

- [Ubuntu](https://github.com/UnnoTed/wireguird) - WireGuard gtk gui for linux 

- [Ubuntu Server](https://forum.level1techs.com/t/self-hosted-vpn-with-wireguard/160861) - quick forum guide with official ppa

- [Debian Server](https://www.wireguard.com/quickstart/) - official quickstart documentation

- [Arch](https://wiki.archlinux.org/title/wireguard) - arch wiki

- [Raspbian](https://wireguard.how/client/raspberry-pi-os/) - WireGuard.how's guide for Raspbian OS Bullseye 

- [RHEL](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/assembly_setting-up-a-wireguard-vpn_configuring-and-managing-networking) - Red Hat's official documentation on WireGuard


* * *

## WireGuard client key generation

There are many different clients listed above. The concept below remains the same for each of them.


### Generate a private key for this peer's client

#### Use your GUI

You can use any of the GUI clients to hit a button to generate a `Private Key` and a `Public Key`. 

*Copy the Public Key.*


##### Windows - Example

In this example we'll be using the Official Windows WireGuard client.

1. Heading over to the client machine. 

2. Open the WireGuard client application.

3. Use the `Add Tunnel` drop down arrow and select `Add Empty Tunnel`.

4. You will have an `[Interface]` with a `Private Key = ` .... **DONT MESS WITH IT**

5. Copy the `Public Key` at the top, including the equals sign.

6. Dont copy anything else. 

More information can be [found here](https://introserv.com/docs/wireguard-windows-setup/).


* * *

#### Use the terminal

You can also use any terminal client to do basically the same with `wg genkey`.

##### Linux Terminal - Example

This quick `script` will `generate` the `keys` needed to `/etc/wireguard`, and print them to the screen:

```bash
if [ "$EUID" -ne 0 ]; then echo "Please re-run as root." && sleep 2 && echo "Application Exiting..." && sleep 30 && exit 1; fi && echo -e "\n\nSetting up public & private key in /etc/wireguard\n" && wg genkey | tee /etc/wireguard/$HOSTNAME.private.key | wg pubkey > /etc/wireguard/$HOSTNAME.public.key && chmod 600 /etc/wireguard/$HOSTNAME.private.key /etc/wireguard/$HOSTNAME.public.key && echo "Private Key:" && cat /etc/wireguard/$HOSTNAME.private.key && echo -e "\nPublic Key (copy this):" && cat /etc/wireguard/$HOSTNAME.public.key;
```


* * * 

### WireGuard peer client - review

- You will need to have the `public key` from your client copied.

> At this point we should have a public and private key generated for our client. 


* * * 

# PART III - WireGuard Configuration

* * * 


## WireGuard on OPNsense

### Install WireGuard on OPNsense

WireGuard is now Kernel level in OPNsense. There is no need to download a package anymore.


* * * 

### WireGuard tunnel subnet interface

This is the configuration for the tunnel address of the OPNsense endpoint, the *"server"*.

`Instance` is the WireGuard interface's subnet.

1. `VPN ‣ WireGuard ‣ Settings ‣ Instances`

2. New (Click on the + symbol)

3. Enabled

4. Name - `wgopn1-memestor`
> These names wont be seen anywhere outside of this config screen. BUT, you will see an interface name when you go to assign an interface. 

5. Public Key - Hit the cog. You will see a series of characters with an equals sign that will always appear at the end.
 
7. Keep hitting the cog until a public key with a series of numbers and letters appears without any special characters (except the =, it indicates the end of the key).

8. Listen Port - `51820`

9. Tunnel Address - `10.2.2.1/24`

10. Peers - Blank for now

11. Save

12. Apply


* * * 

### WireGuard client endpoint as a peer

#### WireGuard peer info

`VPN ‣ WireGuard ‣ Settings ‣ Peers ‣ Allowed IPs` - These are what IP addresses are going to be permitted over the tunnel.

You can send and the server will receive it, but it will do nothing and send nothing back... UNLESS you have the IP of the Endpoint (in it's `[Interface]` `Address = ` section) on the Allowed IPs list for the endpoint.

This is why you have to configure every client that wants to connect to this firewall/WireGuardserver. (Unless they're sharing certificates.)

That client should have created the private/public key pair, and you will paste the public key.


* * *

#### WireGuard peer creation - Peer Generator

Using the generator, you will not need the public key set earlier, it is defined in the generator. The peer generator will also load in the correct `Address` for you, but the rest needs set. If you're doing a WireGuard Site-to-Site VPN go ahead and [skip this step](#setting-up-wireguard-on-each-instance-of-opnsense-for-site-to-site).

1. `VPN ‣ WireGuard ‣ Settings ‣ Peer Generator`

2. Select the `Instance` you would like to use, in this example it was `wgopn1-memestor`.

3. `Endpoint` - Specify how to reach the instance, usually the public address of this firewall. (e.g. my.endpoint.local:51820)

4. `Name` - thinkpad

5. `Public key` - Set by the generator and copied out of this page in the 'config' below.

6. `Address` - Should be automatically set and incremented for each peer in the 'instance' subnet.
> If you're using the same certificate for multiple clients, then you will need to increase the CIDR subnet for more IP addresses.

7. `Allowed IPs` - List the networks on the other side of WireGuard that we're allowed to pass through. (e.g. - 10.2.2.0/24, 172.23.10.0/24)

7. `DNS Servers` - Give the address of the router, on that network's 'Allowed IPs' subnet.

8. `Config` - Here lies the generated WireGuard config. Copy this text and save it to for your client, 'WireGuard-memestor.conf'
> You will NOT get a chance to copy the 'Private Key' again, as it will only appear on here this screen, now. Refresh, clears it - Save, clears it. 


![OPNsense and WireGuard automatic peer generator](/assets/img/posts/OPNsense-and-Wireguard--PEER-Generator.png)


* * *

#### WireGuard peer creation - Manual Creation

If you're using the 'peer generator' instructions above, feel free to skip this section. This page is where you setup each individual key connecting to WireGuard, and is why we were required to setup the client, for the key generation earlier. Again, if you're doing a WireGuard Site-to-Site VPN go ahead and [skip this step and head below](#setting-up-wireguard-on-each-instance-of-opnsense-for-site-to-site).

1. `VPN ‣ WireGuard ‣ Settings ‣ Peers`

2. New (Click on the + symbol)

3. `Name` - thinkpad

4. `Public key` - Paste in the `Public Key` from the client machine you generated earlier.
> If you forgot, you will need to go to the client you're using and copy the Public Key.

5. `Pre-shared Key` — `Optional`, and may be omitted. This option adds an additional layer of symmetric-key cryptography for post-quantum resistance. You will need to take this key from OPNsense to the client. Write it down or copy it.

6. `Allowed IPs` - `10.2.2.0/24, 172.23.10.0/24`
>  `Allowed IPs` adds a route inside of opnsense to the "allowed IPs" subnet over WireGuard's local profile's `Tunnel Address`.

7. Endpoint address - `coolserver.dyndns.net`
> How do you reach the server you're connecting to? 

8. Endpoint port - `51820`

9. Instances - Should have the name of the tunnel subnet we made earlier, `wgopn1-memestor`.

10. Keepalive interval - 25

11. Save

12. Apply




* * * 

### Finish WireGuard initial config - resetting services

The save and apply are meaningless, as WireGuard never resets the service to load the new configuration. 

You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings ‣ General` or go to the `Dashboard` and `reset services` from there. 



* * * 

### WireGuard configuration - review

Reviewing what should be completed at this point.

- You should have an `Instance` setup with a `tunnel address` that you can use.

- There should also be a `Peer` with the `public key` that was generated from your client, client being the remote machine you're using to connect back to OPNsense. 

- On the same `Peer`, you should also have an `IP` set for your OPNsense's peer within the Instance's `subnet``.



* * * 

## Caveats

* * *


### Allowed IPs

Generally, it is important to keep the subnets small on the endpoints, especially when using multiple endpoints.

- Allowed IPs sets routes for you, using WireGuard. 
> If you want to talk to that network after you're connected, list the subnet here.

- Route all traffic over the connection (including the internet):
> `AllowedIPs = 0.0.0.0/0`


#### Allowed IP Explained

In other words, when sending packets, the list of Allowed IPs behaves as a sort of routing table, and when receiving packets, the list of Allowed IPs behaves as a sort of access control list.
   
   - In sending direction this list behaves like a routing table.

   - In receiving direction it serves as Access Control List.


* * * 

### One Side Needs a Static IP

> If you use **hostnames** in the Endpoint Address, WireGuard will only resolve them once when you start the tunnel. If both sites have dynamic Endpoint Addresses set, the tunnel will stop working if a site receive a new WAN IP lease from the ISP. 
{: .prompt-info }

> To mitigate this you'd need to check DNS resolve the IP addresses for the system, then restart WireGuard if there was a new IP.
{: .prompt-warning }


* * * 

### Behind a NAT? Probably. Set keepalive!

> If a site/instance/peer is behind NAT, a keepalive has to be set on the site behind the NAT. The keepalive should be set above, if you followed the tutorial, to 25 seconds as stated in the official WireGuard docs. It keeps the UDP session open when no traffic flows, preventing the WireGuard tunnel from becoming stale because the outbound port changes. Tailscale, Zerotier, Netbird all do the same thing.
{: .prompt-info }



* * *

## IP addressing with WireGuard on OPNsense

* * *


#### OPNsense's subnetting example:

```
Tunnel Address  >   Allowed IPs      >   OPNsense [Interface] address
```

#### WireGuard subnetting example:

```
   Instances    >     Peers          >   Address on the Client's Interface
       ^                ^                           ^
     Entire       Any Subnet to           Single IP that resides on both
     Subnet       connect between         the Peer's and Instance's subnet
```

**Each WireGuard network `interface` has a `private key` and a `list of peers`.**

**Each WireGuard `peer` on OPNsense must have the client's `public key` and match the tunnel's `Allowed IPs`**



* * * 

# PART IV - WireGuard Site-to-Site VPN

* * * 

**You may skip this section** if you do not require a site-to-site WireGuard VPN, and you are strictly `using` this as a roadwarrior way to `remote devices` into your OPNsense router.


* * * 

## Doing an OPNsense Site-to-Site WireGuard VPN

The peer generator didnt help site-to-site WireGuard VPN config generation at all.

- You need a seperate Instance on BOTH locations for a Site-to-Site VPN over WireGuard. 💾👍

- You need to connect those Instances with a respective peer on both sites. 😀-😀

- You need to have a Wireguard firewall rule set, for those interfaces to allow traffic in both directions across their respective interfaces. 🔥🧱


## Setting up WireGuard on each Instance of OPNsense for Site-to-Site

The following example covers an IPv4 Site to Site WireGuard Tunnel between two OPNsense Firewalls with public IPv4 addresses on their WAN interfaces. You will connect the (*Site A LAN*) `172.16.0.0/24` to the (*Site B LAN*) `192.168.0.0/24` using the (*WireGuard Transfer Network*) `10.2.2.0/24`. (*Site A Public IP*) is `203.0.113.1` and (*Site B Public IP*) is `203.0.113.2`. On the (*WireGuard Tunnel Network*) the tunnel address for (*Site A WireGuard*) is `10.2.2.1/24` and the tunnel address for (*Site B WireGuard*) is `10.2.2.2/24`.


### Table of all addresses and interfaces used 

There are a lot of confusing segments in this tutorial. I have adapted this table to the information being used. 


` T A B L E ___  O F ___  A D D R E S S E S`

|      Address        |     IP         |
|---------------------|----------------|
| WireGuard Network   |  10.2.2.0/24   |
| Site A - WireGuard  |  10.2.2.1/24   |
| Site B - WireGuard  | 10.2.2.2/24    |
| Site A - LAN        | 172.16.0.0/24  |
| Site B - LAN        | 192.168.0.0/24 |
| Site A - Public WAN | 203.0.113.1    |
| Site B - Public WAN | 203.0.113.2    |


* * *

### Site A - Meme Storage Bunker HQ's Server - Instance setup

This is, presumably, the OPNsense router you've already been configuring from above - as this is the meme bunker. 
We can ignore these steps if you've already got the first `Instance` from above already setup, as these steps are primarily the same.

1. `VPN ‣ WireGuard ‣ Settings ‣ Instances`

2. New (Click on the + symbol)

3. Enable the `advanced mode` toggle in the upper corner.

4. Name - `wgopn1-memestor`

5. Public Key - `Generate` with “Generate new keypair” cog looking button.

6. Copy (and label) this `public key` somewhere, as we will need it shortly.

7. Tunnel Address - `10.2.2.1/24`

8. Save

9. Apply


![WireGuard interface creation on the meme storage bunker's OPNsense firewall](/assets/img/posts/OPNsense-and-Wireguard--HOME--instance-creation.png)


* * *

### Site B - Sarah's Flower Shop Server - Instance setup

Back in another Instance of OPNsense, we are going to follow mostly the same steps.

1. Get off the Airplane in Detroit.

2. Call Sarah and see where her Flower Shop is at.

3. Arrive at Flower Shop, and login to OPNsense server.

4. Visit: `VPN ‣ WireGuard ‣ Settings ‣ Instances`

5. New (Click on the + symbol)

6. Enable the `advanced mode` toggle in the upper corner.

7. Name - `wgopn2-flwrstor`

8. Public Key - `Generate` with “Generate new keypair” cog looking button.

9. Copy this `public key` somewhere, as we will need it shortly.

10. Tunnel Address - `10.2.2.2/24`

11. Save

12. Apply


![WireGuard interface creation for Sarah's Flower Shop OPNsense Server remote connection](/assets/img/posts/OPNsense-and-Wireguard--REMOTE--instance-creation.png)


* * *

### Site A - MSB HQ's Server - Peer server setup

This OPNsense is back in the original WireGuard Instance that was made. 

The Meme Storage Bunker HQ's Server, `wgopn1-memestor`.
Site A Instance with Tunnel Address of `10.2.2.1/24`.

This part is where you will setup each individual key connecting to WireGuard, and is why the `public key` of each site's Instance was copied somewhere handy.


1. `VPN ‣ WireGuard ‣ Settings ‣ Peers`

2. New (Click on the + symbol)

3. Enable the `advanced mode` toggle in the upper corner.

4. Name - `wgopn2-flwrstor`

5. Public Key - Insert the public key of the instance from `wgopn2-flwrstor`.
> Remember, when you were in Detroit? You setup an OPNsense WireGuard Instance at Sarah's Flower Shop. 

6. Allowed IPs - `10.2.2.2/32 192.168.0.0/24`
> You are allowing the static IP of Sarah's Flower Shop Server's WireGuard Instance's tunnel.

7. Endpoint Address - This is set to the public IP of the WireGuard Instance we're connecting to, `203.0.113.2`.

8. Instances - Select the Instance to connect this Peer to, `wgopn1-memestor`.

9. Save

10. Apply


![WireGuard peer certificate creation for meme storage bunker](/assets/img/posts/OPNsense-and-Wireguard--HOME--adding-peer.png)


* * *

### Site B - SFS' Server - Peer server setup

Back in Detroit, at our other OPNsense server in Sarah's Flower Shop - setup is primarily the same.

Sarah's Flower Shop Server, `wgopn2-flwrstor`.
The Site B Instance with Tunnel Address of `10.2.2.2/24`.

Again, setup the key, IPs, and Instance connecting to MSB HQ's WireGuard.


1. `VPN ‣ WireGuard ‣ Settings ‣ Peers`

2. New (Click on the + symbol)

3. Enable the `advanced mode` toggle in the upper corner.

4. Name - `wgopn1-memestor`

5. Public Key - Insert the public key of the instance from `wgopn1-memestor`.
> Remember, when you were in Detroit? You setup an OPNsense WireGuard Instance at Sarah's Flower Shop. 

6. Allowed IPs - `10.2.2.1/32 172.16.0.0/24`
> You are allowing the static IP of Sarah's Flower Shop Server's WireGuard Instance's tunnel.

7. Endpoint Address - This is set to the public IP of the WireGuard Instance we're connecting to, `203.0.113.1`.

8. Instances - Select the Instance to connect this Peer to, `wgopn2-flwrstor`.

9. Save

10. Apply


![WireGuard peer certificate creation for Sarah's Flower Shop](/assets/img/posts/OPNsense-and-Wireguard--REMOTE--adding-peer.png)


* * * 

### WireGuard site-to-site setup review

A lot of this was the same as the inital Roadwarrior setup in the beginning. The difference with site-to-site it's between two OPNsense servers and not a peer client.


1. You should have an `Instance` setup on both OPNsense servers.

2. Each WireGuard Instance should have a unique `tunnel address` on the same subnet.

3. A `Peer` was added with the `public key` that was generated from the server we're going to connect to's Instance.

4. The `Peer` needs to have each subnet from the other `Site` listed in the `Allowed IPs`.

5. On the same `Peer`, the `Endpoint Address` needs to point to the other server's connectable IP.

6. `Peers` and `Instances` must also `belong` to each other with the `drop down` selecting each, respectively.



* * * 

### Finish - by restarting services

The save and apply are meaningless, as WireGuard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings ‣ General` or go to the `Dashboard` and `reset services` from there. 




* * * 

# PART V - WireGuard Interface

* * * 


## Configure a WireGuard Interface on OPNsense

This allows separation of the firewall rules for each WireGuard instance (`wgX` device). 


* * * 

### Assign an interface to WireGuard 

Please note, if you have not enabled the WireGuard service the interface creation will fail.

1. Go to `Interfaces ‣ Assignments`

2. At the bottom is the `Assign a new interface` section. 

3. In the dropdown next to `Device`, select the WireGuard device you created.

4. Add a description (eg wgopn1memestor)
> This is what will be visible under Interfaces on the menu.

1. Click the `Add` button, then click `Save`.


* * * 

### Enable new WireGuard interface

1. Click on your `new interface`'s description under the `Interfaces menu`.

2. Once on this new screen, we need to Enable the interface.

1. Enable - `Checked`

2. Lock - `Checked`

3. IPv4 Configuration Type - `None`
> There is no need to configure IPs on the interface. The tunnel address(es) specified in the Instance configuration for your server will be automatically assigned to the interface once WireGuard is restarted

4. Save

5. Apply


> When assigning interfaces, gateways can be added to them. This is useful if balancing traffic across multiple tunnels is required or in more complex routing scenarios. To do this, go to `System ‣ Gateways ‣ Configuration` and add a new gateway. Choose the relevant WireGuard interface and set the Gateway to dynamic. 
{: .prompt-info }



* * * 

### Finish WireGuard interface by resetting WireGuard services

The save and apply are meaningless, as WireGuard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings ‣ General` or go to the `Dashboard` and `reset services` from there. 

`Unbound DNS` requires a `reload` of Unbound DNS's services to get the new WireGuard interface added. 








* * * 

# PART VI - OPNsense rules for WireGuard

* * * 

This area is where the networking configuration begins. 

> This may be a good time to go make a tea ☕ and grab a snack.


We are starting to shape where our traffic can and cannot go. 

Please note, much of this can be changed for preference and is not ridged. 


* * *

## Create a WireGuard outbound NAT rule

### Detailing outbound NAT changes

This step is only necessary if you intend to `allow` client peers to `access` IPs `outside` of the local IPs/subnets behind `OPNsense`.

Think `VPN provider`, do you want to allow `wgopn1memestor` clients to forward their Internet traffic over your network?

WireGuard subnets that you want to have internet access need their subnets in `Firewall ‣ NAT ‣ Outbound`.


#### Edits made to NAT in brief:

- `Interface`: WAN

- `Source`: whatever the `Local Tunnel` subnet is set to.


* * *

### Details of outbound NAT rule

> Note: Changes made to the rule are highlighted as `code blocks`, areas that were not modified are written as standard text. 
{: .prompt-info }


1. Go to `Firewall ‣ NAT ‣ Outbound`

2. Select `Hybrid outbound NAT rule generation` at the top, if it is not already selected.

3. `Save` and then `Apply` changes

4. Click `Add` to add a new rule (Click on the + symbol)

5. Interface - `WAN`

6. TCP/IP Version - `IPv4` or IPv6 (as applicable)

7. Protocol - any

8. Source invert - Unchecked

9. Source address - Select the network of our new interface: `wgopn1memestor net`

10. Source port - any

11. Destination invert - Unchecked

12. Destination address - any

13. Destination port - any

14. Translation / target - Interface address

15. Description - `Allow traffic from wgopn1memestor to outbound LAN/Internet`

16. Save

17. Apply




* * *

## Setup firewall rules on OPNsense for WireGuard

This will involve two steps.

1. Creating a firewall rule on the WAN interface to allow clients to connect to the OPNsense WireGuard server.

2. Creating a firewall rule to allow access by the clients to whatever IPs on the local network they are intended to have access to.


* * *

### Create firewall rules on - WAN

Letting in your port you made for WireGuard opens your firewall up. You now have a hole in your network, on the port you choose. 

> Be aware, [UDP hole-punching](https://en.wikipedia.org/wiki/UDP_hole_punching) VPNs like Tailscale, Zerotier, Netbird all work without this requirement.

Now that you're aware of the risks and alternatives, let's begin:

1. Go to `Firewall ‣ Rules ‣ WAN`

4. Click `Add` to add a new rule (Click on the + symbol)

3. Action - Pass

4. Quick - Checked

5. Interface - WAN

6. Direction - in

7. Protocol - `UDP`

8. Source / Invert - Unchecked

9. Source - any

10. Destination / Invert - Unchecked

11. Destination - `WAN address`

12. Destination port range - Select `(other)`. The number to enter is probably the default, `51820`, but check the WireGuard port you set in the Instance configuration on an earlier step.

13. Description - `WireGuard in WAN allow`

14. Save

15. Apply


* * *

### Create firewall rules on - WireGuard

The firewall rule outlined below will need to be configured on the automatically created `WireGuard group` that appears once the Instance configuration is enabled and WireGuard is started. 

You will also need to manually specify the subnet for the `tunnel`. 

> You can also define an `alias` (via `Firewall ‣ Aliases`) for any IPs/`subnet` that you want to use. 


1. Go to `Firewall ‣ Rules ‣ WireGuard (Group)`

2. Click `Add` (Click on the + symbol) to add a new rule.

3. Action - Pass

4. Quick - Checked

5. Interface - `WireGuard (Group)` or an alias

6. Direction - in

7. TCP/IP Version - `IPv4` or IPv4+IPv6 (as applicable)

8. Protocol - any

9. Source / Invert - Unchecked

10. Source - `WireGuard (Group) net` or an alias

11. Destination / Invert - Unchecked

12. Destination - Specify the IPs or subnet that client peers `should` be able to `access`. You can use an alias here too.

13. Destination port range - any

14. Description - `Any WireGuard interface can access these networks`

15. Save

16. Apply


![WireGuard firewall rules](/assets/img/posts/OPNsense-and-Wireguard--FIREWALL--interface-traffic.png)


* * *

### Normalization rules for WireGuard

By creating normalization rules, you ensure that IPv4 TCP can pass through the WireGuard tunnel without fragmentation of traffic. Otherwise you could get working ICMP and UDP, but some encrypted TCP sessions **will refuse to work**.

1. Go to `Firewall ‣ Settings ‣ Normalization`

2. Click `Add` (Click on the + symbol) to add a new rule.

3. Interface - `WireGuard (Group)`

4. Direction - `Any`

5. Protocol - any

6. Source - any

7. Destination - any

8. Destination port - any

9. Description - `WireGuard MSS Clamping IPv4`

10. Max mss - `1380` (default); it’s 40 bytes less than your WireGuard MTU

11. Save the rule

12. Apply


* * * 

## COMPLETE

* * *

### Working WireGuard Site-to-Site VPN with OPNsense

You should now have a working WireGuard VPN. 

- The `instance` has been made, this created our `interfaces`, and we enabled them. 

- The WireGuard `peer` is connecting to an `Endpoint` addresss and port -- which is an open WAN port set in the firewall.

- The server's instance `Public Key` is set for other machine's connecting 'peer'.

- `Allowed IPs` reflect the traffic that is allowed to pass through the tunnel.

- `Firewall` rules are in place to allow the WireGuard interface onto the router.

> This traffic can come from anywhere and go anywhere. We should further restrict this...


* * * 

# PART VII - Site-to-Site VPN - OPNsense firewall configuration

* * * 


**You may skip this section** if you do not require firewalling for a site-to-site WireGuard VPN, or you are strictly `using` this as a way to `remote devices` into your OPNsense router's subnet.


* * *

## Site-to-Site WireGuard WAN connection

Think of this section like `two peers` connecting to each other.

You will be editing the firewall settings for the `WAN connection` between the two WireGuard clients. 

> For reference, [link to table](https://blog.holtzweb.com/posts/opnsense-wireguard-vpn/#setting-up-wireguard-on-each-instance-of-opnsense-for-site-to-site) of all addresses used in this writeup.


* * *

### Site A - WAN Firewall setup - Meme Storage Bunker HQ's Server

Back at the bunker, we need to add a new rule to allow incoming WireGuard traffic from Site B (Sarah's Flower Shop).


1. Go to `Firewall ‣ Rules ‣ WAN` 

2. Click `Add` (Click on the + symbol) to add a new rule.

3. Action - Pass

4. Interface - WAN

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - `UDP`

8. Source - `Single host or Network` and set this to the remote site's WAN address: `203.0.113.2`

9. Destination - `WAN address` set this to the WAN address to allow on WAN from our remote source.

10. Destination port - `51820`

11. Description - `Allow WireGuard from remote Site B to this Site A`


* * *

### Site B - WAN Firewall setup - Sarah's Flower Shop Server

The same step needs to be taken, but with the WAN addresses reversed - allow incoming WireGuard traffic from Site A (Meme Storage Bunker HQ).


1. Go to `Firewall ‣ Rules ‣ WAN`

2. Click `Add` (Click on the + symbol) to add a new rule.

3. Action - Pass

4. Interface - WAN

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - UDP

8. Source - `Single host or Network` and set this to the remote site's WAN address: `203.0.113.1`

9. Destination - `WAN address` set this to the WAN address to allow a WAN connection from our remote source.

10. Destination port - `51820`

11. Description - `Allow WireGuard from remote Site A to this Site B`

12. Press Save and Apply.



* * *

## Verify WireGuard connection on Site A and Site B

### Load new WireGuard configuration

Ensure you are able to reset the WireGuard service to load the **new configuration**. 

- Go to `VPN ‣ WireGuard ‣ Settings` on both sites and `check` and `uncheck` the `Enable WireGuard` and press `Apply`.

- Go to the `Dashboard` on both sites and `reset services` from there. 


### Check WireGuard Logs

To verify any of this is working correctly, go to `VPN ‣ WireGuard ‣ Diagnostics`. 

You should see Send and Received traffic and Handshake should be populated by a number. This happens as soon as the first traffic flows between the sites.

> If you see this, your tunnel is now up and running.



* * * 

# PART VIII - Site-to-Site VPN - OPNsense router configuration

* * * 


## Routing different subnets across WireGuard

Different subnets separate networks from communicating with each other. The firewall also stops these networks.

Currently, your two LANs cannot see each other. 

We're going to add firewall rules to allow these two sites to communicate like they were in the same room.

You can use this method to connect your home to a VPS in the cloud, or you mom's house to your house.

> For reference, [link to the table](https://blog.holtzweb.com/posts/opnsense-wireguard-vpn/#setting-up-wireguard-on-each-instance-of-opnsense-for-site-to-site) of Site A and Site B LAN addresses used in this writeup.


* * * 

## Site A - Router Pass Traffic - Bunker HQ

> Please make sure you do not have overlapping subnets.


* * * 

### Allow traffic between Site A LAN Net and Site B LAN Net

The first firewall rule will make sure our Bunker HQ (172.16.0.0/24) can reach Sarah (192.168.0.0/24).

1. Go to OPNsense Site A

2. Open `Firewall ‣ Rules ‣ LAN` and `add` a new rule.

   1. Note: `Change for preference`. The network you want to share with WireGuard may be different than `LAN`, please modify the name of the Interface to match your network.

3. Action - Pass

4. Interface - `LAN`

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - Any

8. Source - `172.16.0.0/24`

9. Source port - Any

10. Destination - `192.168.0.0/24`

11. Destination port - Any

12. Description - `Allow LAN on Site A to remote LAN Site B`

13. Press Save and Apply.


* * * 

### Allow traffic from WireGuard Site B LAN Net to Site A LAN Net

The second firewall rule is through the WireGuard tunnel. It allows Sarah's LAN to reach the Bunker HQ's LAN.

1. Go to OPNsense Site A 

2. Open `Firewall ‣ Rules ‣ WireGuard (Group)` and `add` a new rule.

3. Action - Pass

4. Interface - `WireGuard (Group)`

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - Any

8. Source - `192.168.0.0/24`

9. Source port - Any

10. Destination - `172.16.0.0/24`

11. Destination port - Any

12. Description - `Allow LAN on remote Site B to LAN on Site A`

13. Press Save and Apply.



* * * 

## Site B - Router Pass Traffic - Sarah's Shop

> Please make sure you do not have overlapping subnets.


* * * 

### Allow traffic between Sarah's Site B LAN Net and Site A LAN Net

1. Go to OPNsense Site B 

2. Open `Firewall ‣ Rules ‣ LAN` and `add` a new rule.

   1. Note: `Change for preference`. The network you want to share with WireGuard may be different than `LAN`, please modify the name of the Interface to match your network.

3. Action - Pass

4. Interface - `LAN`

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - Any

8. Source - `192.168.0.0/24`

9.  Source port - Any

10. Destination - `172.16.0.0/24`

11. Destination port - Any

12. Description - `Allow LAN on Site B to remote LAN Site A`

13. Press Save and Apply.



* * *

### Allow traffic from WireGuard Site A LAN Net to Sarah's Site B LAN Net

1. Go to OPNsense Site B 

2. Open `Firewall ‣ Rules ‣ WireGuard (Group)` and add a new rule.

3. Action - Pass

5. Interface - `WireGuard (Group)`

6. Direction - In

7. TCP/IP Version - IPv4

8. Protocol - Any

9. Source - `172.16.0.0/24`

10. Source port - Any

11. Destination - `192.168.0.0/24`

12. Destination port - Any

13. Description - `Allow LAN on remote Site A to LAN on Site B`

14. Press Save and Apply.

> Now both sites have full access to the LAN of the other Site through the WireGuard Tunnel. For additional networks just add more Allowed IPs to the WireGuard Endpoints and adjust the firewall rules to allow the traffic.
{: .prompt-info }


* * *

* * *

## Alt route - no firewall route

You can try and route without any rules. You want the wireguard subnet to be routed in its entirety to and from that gateway so that access can be established and mapped by your router without the need to add firewall rules. 

1. Go to `System ‣ Gateways ‣ Configuration`

2. Create a new gateway.

3. Assign the interface.

4. Make sure you have set the IP of your remote wireguard's gateway's IP, the tunnel address, in our example above it was Site B to remote Site A: `10.2.2.1/24`.

5. With a new gateway up we can point a route there.

6. Head to `System ‣ Routes ‣ Configuration`.

7. Create a new route.

8. Make the network address your WireGuard subnet.

9. Set the gateway dropdown to the one you defined earlier.

10. Open `Firewall ‣ Settings ‣ Advanced ‣ Static route filtering`.

11. Check the `Bypass firewall rules for traffic on the same interface` checkbox.



### The rest of the Firewall stuff you know

1. Click on Firewall -> Rules -> WireGuard

2. Then on the + ADD button. 

3. Select `Single host or Network` as `source` and 

4. Enter the `IP range` of the `WireGuard network` and its subnet mask.

5. Save.

6. Looking back..

> Firewall -> Rules -> WireGuard, you should see -- under `Source` the IP Address of the Internal subnet you're using for you're WireGuard network you made under VPN -> WireGuard -> Local.


* * *

* * *

### Finish - by resetting services

The save and apply are meaningless, as WireGuard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings ‣ General` or go to the `Dashboard` and `reset services` from there. 



* * * 

# PART IX - The peer client

* * * 


## Setting up the client software

**So the client needs:**

- Their Client Config

- Endpoint config of the "server"
  - Public IP of the Endpoint "server"
  - Public Key of the Endpoint "server"


* * *

### Example: Official Windows WireGuard client

That client software we started up and then left out in the cold, hungry for input. 
Let's go do something with that client now.

1. Make sure the SAME client software is open, with the same `Public Key` that was copied earlier. No exit and starting over.

2. Copy this into your client config for tunnel generation:

```ini
[Interface]
PrivateKey = #somenumber#
Address = 10.10.10.2/32
DNS = #Internally Routed DNS. This is on the subnet you're VPNing into, example 172.23.55.254#

[Peer]
PublicKey = #HEYWAIT!---WeDontHaveThisYet---#
AllowedIPs = 0.0.0.0/0 #This is the subnet youre VPNing into, so this would be, example 172.23.55.0/24#
Endpoint = edge.sub.domain.com:51820
```

3. That's right! We still have to get the `Public Key` of our other confidant, the Server we're connecting to.

4. Moving over to the web interface, under `VPN ‣ WireGuard ‣ Settings ‣ Local`

5. Edit the `Local Profile's Configuration` that we created earlier, named `wgopn1-memestor` and copy the `Public Key` making sure to get **THE ENTIRE THING** including the `=`

6. Paste this into the config section of the client, under `[peer]`

7. Save




### Example commented wg0.conf

THERE ARE A BUNCH OF STEPS FOR CREATING CONFIGS.

THERE ARE SEVERAL AUTO GENERATING ONES ONLINE

HERE IS ANOTHER EXAMPLE:


```ini
[Interface]
Address = <Configured client IP>/<Netmask> // For example the IP "10.11.0.20/32"
PrivateKey = <Private Key of the client>

[Peer]
PublicKey = <Public Key of the OPNsense WireGuard instance>
AllowedIPs = <Networks to which this client should have access>/<Netmask>
             // For example "10.11.0.0/24, 192.168.1.0/24"
             //               |             |
             //               +--> The network area of the OPNsense WireGuard VPNs
             //                             |
             //                             +--> Network behind the firewall
Endpoint = <Public IP of the OPNsense firewall>:<WireGuard Port>
```



## Future Updates - help

If any of this is out of date, send me a pull request. I would love the chance to update this if there's a change. 

At the time of writing **OPNsense is at version 24.1.4**




* * *

* * *

## Having trouble?

For a one-time donation you can get one-on-one troubleshooting support for any of my guides/projects. I'll help you fix any issue you may have encountered regarding usage/deployment of one of my guides or projects. More info in my [Github Sponsors profile](https://github.com/sponsors/MarcusHoltz).

<iframe src="https://github.com/sponsors/MarcusHoltz/card" title="By sponsoring me, you're supporting my ongoing work and everything you've come to know me by -- cutting-edge and resilient systems with vigilant monitoring practices." height="225" width="600" style="border: 0;"></iframe>
