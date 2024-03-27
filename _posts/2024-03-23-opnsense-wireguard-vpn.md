---
layout: post
title: OPNsense as a Wireguard VPN
date: 2024-03-23 11:33:00 -0700
categories: [Networking, VPN]
tags: [opnsense, workstation, split-dns, VPN, wireguard, remote]
pin: false
image:
  path: /assets/img/header/header--opnsense--wireguard-VPN.jpg
  alt: Setting up OPNsense router as a Wireguard VPN
---

# Wireguard on OPNsense 

There are multiple scenarios where Wireguard can be used, but they all require different configs for that setup.

This writeup should enable a single user (Roadwarrior) and further along the line, a Site-to-Site VPN.


## Wireguard then OPNsense

This tutorial discusses the setup of Wireguard first. 

Please **hang on** till we're done with Wireguard.

**None of this will work** without also changing the `OPNsense firewall` settings. 

### Tutorial Steps

1. Client stuff

2. Wireguard stuff

3. Firewall stuff


* * *

## Wireguard background

To setup Wireguard first, you must understand it's conceptual overview.


1. It's not a client/server VPN setup.

2. **Both sides are a peer.** 

3. So, in essence - you need to setup private and public keys for each side. 
    (*Technically you will derive the public key from the private key.*)

4. No dynamic IP assignment (or very little), each client has a fixed IP.

5. WireGuard associates tunnel IP addresses with public keys and remote endpoints. 


### Cryptokey Routing

At the heart of WireGuard is a concept called `Cryptokey Routing`, which works by associating `public keys` with a list of tunnel `IP addresses` that are `allowed` inside the Wireguard tunnel. 

**Each Wireguard network interface has a private key and a list of peers.**


* * *

## Wireguard conceptual overview example

* * * 


### Someone sending a packet


1. When `Wireguard` needs to `send` a packet, it looks for the IP address in its Local Addresses Subnet. 

2. Let's say, this packet is meant for `192.168.30.8`. 

3. Your computer's Wireguard `looks` for which peer that is. 

4. Okay, it's for peer `ABCDEFGH`. 
(Or if it's not for any configured peer, drop the packet.)

5. `Encrypt` entire IP packet using peer ABCDEFGH's `public key`.

6. Now where do I `send` peer ABCDEFGH's encrypted packet?

7. What remote `endpoint` was listed for peer ABCDEFGH?

8. The endpoint is `listed` as 216.58.211.110 port 53133.

9. Send encrypted data over the `Internet` to 216.58.211.110:53133.


* * *

### Wireguard recieving a packet

1. When an interface for `Wireguard` `recieves` a packet, this could be from port forwarding or an open interface, it attempts to identify it.

2. The Wireguard interface checks the source IP address and port to determine `which` `peer` the packet is from.

3. Once the peer is identified, WireGuard looks up the corresponding `key` associated with that peer from its internal configuration. 

4. It then begins to `decrypt` the information that was sent.

5. Wireguard then uses ABCDEFGH's key to verify the `authenticity` of the packet. 

6. With the accepted, `verified` packet - Wireguard will now remember that peer's most recent Internet `endpoint`.

7. Once all that has been completed, Wireguard will `look` at the packet.

8. It can see the `plain-text` packet from someone on the 192.168.30.X subnet.

9. A final verification takes place as Wireguard looks at the packet from 192.168.30.X to verify that `IP` is even `allowed` to be sending us packets.

10. If the `association` is successful, the packets are `allowed` to pass through the VPN tunnel.


* * * 

#### Summarize Wireguard routing

```text
Some App -> Wireguard -> Destination IP in tunnel -> Public key for peer holding that IP -> peer's most recent Internet endpoint
```


* * *

# PART II - The peer client device setup

* * * 

## Wireguard on the peer's client machine

**Wireguard is an exchange of keys.** Your OPNsense firewall's Wireguard cannot connect with a peer it doesnt have a key for.

There are many ways to do this, we'll use **COPY/PASTE**.


### What is your machine?

This is the WireGuard `peer client`'s software `connecting` back to `OPNsense`. Here's a few that I can mention:


* * * 

#### Android

- [The official WireGuard app for Android](https://apt.izzysoft.de/fdroid/index/apk/com.wireguard.android) - The official app includes an auto-updater, this is against F-Droid policy, and you will find this app at the IzzyOnDroid Repository. 

- [WireGuard Tunnel](https://f-droid.org/packages/com.zaneschepke.wireguardautotunnel/) - An alternative client app for WireGuard with additional features, available on F-Droid.


* * * 

#### iOS

- [The official WireGuard iOS App Store app](https://apps.apple.com/us/app/wireguard/id1441195209) - iOS's official WireGuard app for iOS 15.0 or later. Mostly feature pairity with Android.


* * * 

#### MacOS

- [The official WireGuard MacOS App Store app](https://apps.apple.com/us/app/wireguard/id1451685025) - Apple's Mac App Store's WireGuard app for macOS 12.0 or later. This app allows users to manage and use WireGuard tunnels.


* * * 

#### Windows

- [The official WireGuard for Windows app](https://www.wireguard.com/install/) - [This installer](https://download.wireguard.com/windows-client/wireguard-installer.exe) is the only official and recommended way of using WireGuard on Windows.


* * * 

#### Linux

- [KDE](https://userbase.kde.org/System_Settings/Connections/en) - Since Plasma 5.15, Plasma support Wireguard VPN tunnels, when the appropriate Network Manager plugin is installed.

- [Ubuntu](https://github.com/UnnoTed/wireguird) - wireguard gtk gui for linux 

- [Ubuntu Server](https://forum.level1techs.com/t/self-hosted-vpn-with-wireguard/160861) - quick forum guide with official ppa

- [Debian Server](https://www.wireguard.com/quickstart/) - official quickstart documentation

- [Arch](https://wiki.archlinux.org/title/WireGuard) - arch wiki

- [Raspbian](https://wireguard.how/client/raspberry-pi-os/) - wireguard.how's guide for Raspbian OS Bullseye 

- [RHEL](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/assembly_setting-up-a-wireguard-vpn_configuring-and-managing-networking) - Red Hat's official documentation on WireGuard


* * *

## WireGuard client key generation

There are many different clients listed above. The concept below remains the same on each of them.


### Generate a private key for this peer's client

#### Use your GUI

You can use any of the GUI clients to hit a button to generate a `Private Key` and a `Public Key`. Copy the Public Key.


##### Windows Example

In this example we'll be using the Official Windows WireGuard Client.

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

##### Linux Terminal Example

This quick `script` will `generate` the `keys` needed to your `/etc/wireguard` directory, and print them to the screen:

```bash
if [ "$EUID" -ne 0 ]; then echo "Please re-run as root." && sleep 2 && echo "Application Exiting..." && sleep 30 && exit 1; fi && echo -e "\n\nSetting up public & private key in /etc/wireguard\n" && wg genkey | sudo tee /etc/wireguard/$HOSTNAME.private.key | sudo wg pubkey > /etc/wireguard/$HOSTNAME.public.key && sudo chmod 600 /etc/wireguard/$HOSTNAME.private.key /etc/wireguard/$HOSTNAME.public.key && echo "Private Key:" && cat /etc/wireguard/$HOSTNAME.private.key && echo -e "\nPublic Key (copy this):" && cat /etc/wireguard/$HOSTNAME.public.key;
```


* * * 

### WireGuard peer client review

- You will need to have the `public key` from your client copied.

> At this point we should have a public and private key generated for our client. 


* * * 

# PART III - WireGuard Configuration

* * * 


## Wireguard on OPNsense

### Install Wireguard on OPNsense

Wireguard is now Kernel level in OPNsense. There is no need to download a package anymore.


* * * 

### Wireguard tunnel subnet interface

This is the configuration for the tunnel address of the OPNsense endpoint, the *"server"*.

`Instance` is the WireGuard interface's subnet.


1. `VPN > WireGuard > Settings > Instances`

2. New (Click on the + symbol)

3. Enabled

4. Name - `wgopn1-memestor`
> These names wont be seen anywhere outside of this config screen. BUT, you will see the interface name when you go to assign an interface. 

5. Public Key - Hit the cog. You will see a series of characters with an equals sign that will always appear at the end.
 
7. Keep hitting the cog until a public key with a series of numbers and letters appears without any special characters.

8. Listen Port - `51820`

9. Tunnel Address - `10.2.2.1/24`

10. Peers - Blank for now

11. Save

12. Apply


* * * 

### WireGuard client endpoint as a peer



`Peer` > `Allowed IPs`

You can send and the server will recieve it, but it will do nothing and send nothing back... UNLESS you have the IP of the Endpoint (in it's `[Interface]` `Address = ` section) on the Allowed IPs list for the endpoint.

This is why you have to configure every client that wants to connect to this firewall/Wireguardserver. (Unless they're sharing certificates.)

`Name` should be the description of who's connecting. Example: `holtzhouse-windows-buddypc`

`Allowed IPs` what IP addresses are going to be permitted over the tunnel.

Enter the address you want the client to have, along with the matching subnet.

If you entered an internet IP, only the computer on the internet with that IP address would be allowed access.

To finish this, you must import the key from the Client you're using to connect with.

That client will create the private/public key pair and you will paste in the public key.
















This page is where you setup each individual key connecting to WireGuard, and is why we were required to setup the client, for the key generation earlier.

1. `VPN > WireGuard > Settings > Peers`

2. New (Click on the + symbol)

3. `Name` - thinkpad

4. Public key - Paste in the `Public Key` from the client machine you generated earlier.
> If you forgot, you will need to go to the client you're using and copy the Public Key.

5. Pre-shared Key — `Optional`, and may be omitted. This option adds an additional layer of symmetric-key cryptography for post-quantum resistance. You will need to take this key from OPNsense to the client. Write it down or copy it.

6. Allowed IPs - `10.2.2.22/32`
> If you're using the same certificate for multiple clients, then you will need to increase the CIDR subnet for more IP addresses.

7. Endpoint address - Leave the endpoint address on this OPNsense **server** `instance` with the `static IP` address `empty`.
> The *peer client* with the dynamic IP will then be the initiator, and the *peer server* with the static IP will be the responder.

8. Endpoint port - Should also remain `blank`.

9. Instances - Should have the name of the tunnel subnet we made earlier, `HomeWireGuard`.

10. Keepalive interval - 25

11. Save

12. Apply

**`Allowed IPs` adds a route inside of opnsense to the "allowed IPs" subnet over WireGuard's local profile's `Tunnel Address`.** 


* * * 

### Finish - by reseting services

The save and apply are meaningless, as wireguard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings > General` or go to the `Dashboard` and `reset services` from there. 



* * * 

### WireGuard configuration review

1. You will need to have an `Instance` setup with a `tunnel address` that you can use.

2. There should also be a `Peer` with the `public key` that was generated from your client, client being the remote machine you're using to connect back to OPNsense. 

3. On the same `Peer`, you should also have a `static IP` set for your OPNsense's peer within the Instance's subnet.



* * * 

## Caveates

* * *


### AllowedIPs

Generally, it is important to keep the subnets small on the endpoints, especially when using multiple endpoints - to prevent hopping.

But if your public IP roams (xfinity, comcast) you will need:

`AllowedIPs = 0.0.0.0/0`

This will ensure any Source IP can connect to that interface if it can autheticate correctly. 


#### Allowed IP Explained

In other words, when sending packets, the list of Allowed IPs behaves as a sort of routing table, and when receiving packets, the list of Allowed IPs behaves as a sort of access control list.
   
   - In sending direction this list behaves like a routing table.

   - In receiving direction it serves as Access Control List.


* * * 

### One Side Needs a Static IP

> If you use hostnames in the Endpoint Address, Wireguard will only resolve them once when you start the tunnel. If both sites have dynamic Endpoint Addresses set, the tunnel will stop working if a site receive a new WAN IP lease from the ISP. 
{: .prompt-warning }

> To mitiage this you'd need to check DNS resolve the IP addresses for the system, then restart WireGuard if there was a new IP.
{: .prompt-info }


* * * 

### Behind a NAT? Probably. Set keepalive!

> If a site/instance/peer is behind NAT, a keepalive has to be set on the site behind the NAT. The keepalive should be set above, if you followed the tutorial, to 25 seconds as stated in the official wireguard docs. It keeps the UDP session open when no traffic flows, preventing the wireguard tunnel from becoming stale because the outbound port changes. Tailscale, Zerotier, Netbird all do the same thing.
{: .prompt-warning }



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

**You may skip this section** if you do not require a site-to-site WireGuard VPN, and you are strictly `using` this as a way to `remote devices` into your OPNsense router.


* * *

## Setting up WireGuard on each Instance of OPNsense for a Site to Site Config

The following example covers an IPv4 Site to Site Wireguard Tunnel between two OPNsense Firewalls with public IPv4 addresses on their WAN interfaces. You will connect the *Site A LAN Net* `172.16.0.0/24` to the *Site B LAN Net* `192.168.0.0/24` using the *Wireguard Transfer Net* `10.2.2.0/24`. *Site A Public IP* is `203.0.113.1` and *Site B Public IP* is `203.0.113.2`.


* * *

### Site A - Meme Storage Bunker HQ's Server - Instance setup

This is, presumably, the OPNsense router you've already been configuring from above - as this is the meme bunker. 
We can ignore these steps if you've already got the first `Instance` from above already setup, as these steps are primarily the same.

1. `VPN > WireGuard > Settings > Instances`

2. New (Click on the + symbol)

3. Enable the `advanced mode` toggle in the upper corner.

4. Name - `wgopn1-memestor`

5. Public Key - `Generate` with “Generate new keypair” cog looking button.

6. Copy (and label) this `public key` somewhere, as we will need it shortly.

7. Tunnel Address - `10.2.2.1/24`

8. Save

9. Apply


* * *

### Site B - Sarah's Flower Shop Server - Instance setup

Back in another Instance of OPNsense, we are going to follow mostly the same steps.

1. Get off the Airplane in Detroit.

2. Call Sarah and see where her Flower Shop is at.

3. Arrive at Flower Shop, and login to OPNsense server.

4. Visit: `VPN > WireGuard > Settings > Instances`

5. New (Click on the + symbol)

6. Enable the `advanced mode` toggle in the upper corner.

7. Name - `wgopn2-flwrstor`

8. Public Key - `Generate` with “Generate new keypair” cog looking button.

9. Copy this `public key` somewhere, as we will need it shortly.

10. Tunnel Address - `10.2.2.2/24`

11. Save

12. Apply



* * *

### Site A - MSB HQ's Server - Peer server setup

This OPNsense is back in the original WireGuard Instance that was made. 

The Meme Storage Bunker HQ's Server, `wgopn1-memestor`.
Site A Instance with Tunnel Address of `10.2.2.1/24`.

This part is where you will setup each individual key connecting to WireGuard, and is why the `public key` of each site's Instance was copied somewhere handy.


1. `VPN > WireGuard > Settings > Peers`

2. New (Click on the + symbol)

3. Enable the `advanced mode` toggle in the upper corner.

4. Name - `wgopn1-memestor`

5. Public Key - Insert the public key of the instance from `wgopn2-flwrstor`.
> Remember, when you were in Detroit? You setup an OPNsense WireGuard Instance at Sarah's Flower Shop. 

6. Allowed IPs - `10.2.2.2/32 192.168.0.0/24`
> You are allowing the static IP of Sarah's Flower Shop Server's WireGuard Instance's tunnel.

7. Endpoint Address - This is set to the public IP of the WireGuard Instance we're connecting to, `203.0.113.2`.

8. Instances - Select the `Site B` Instance that was created earlier, `wgopn2-flwrstor`.

9. Save

10. Apply


* * *

### Site B - SFS' Server - Peer server setup

Back in Detroit, at our other OPNsense server in Sarah's Flower Shop - setup is primarily the same.

Sarah's Flower Shop Server, `wgopn2-flwrstor`.
The Site B Instance with Tunnel Address of `10.2.2.2/24`.

Again, setup the key, IPs, and Instance connecting to MSB HQ's WireGuard.


1. `VPN > WireGuard > Settings > Peers`

2. New (Click on the + symbol)

3. Enable the `advanced mode` toggle in the upper corner.

4. Name - `wgopn2-flwrstor`

5. Public Key - Insert the public key of the instance from `wgopn1-memestor`.
> Remember, when you were in Detroit? You setup an OPNsense WireGuard Instance at Sarah's Flower Shop. 

6. Allowed IPs - `10.2.2.1/32 172.16.0.0/24`
> You are allowing the static IP of Sarah's Flower Shop Server's WireGuard Instance's tunnel.

7. Endpoint Address - This is set to the public IP of the WireGuard Instance we're connecting to, `203.0.113.1`.

8. Instances - Select the `Site A` Instance that was created earlier, `wgopn1-memestor`.

9. Save

10. Apply



* * * 

### WireGuard site-to-site setup review

A lot of this was the same as the inital Roadwarrior setup in the begining. The difference with site-to-site it's between two OPNsense servers and not a peer client.


1. You should have an `Instance` setup on both OPNsense servers.

2. Each WireGuard Instance should have a unique `tunnel address` on the same subnet.

3. A `Peer` with the `public key` that was generated from each Instance.

4. The `Peer` needs to have each subnet from the other `Site` listed in the `Allowed IPs`.

5. On the same `Peer`, the `Endpoint Address` needs to point to the other server's connectable IP.

6. `Peers` and `Instances` must also be `belong` to eachother with the `drop down` selecting each.



* * * 

### Finish - by restarting services

The save and apply are meaningless, as wireguard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings > General` or go to the `Dashboard` and `reset services` from there. 




* * * 

# PART V - WireGuard on OPNsense

* * * 


## Configure a WireGuard Interface on OPNsense

This allows separation of the firewall rules of each WireGuard instance (each wgX device). 


* * * 

### Assign an interface to WireGuard 

Please note, if you have not enabled the WireGuard service the interface creation will fail.

1. Go to Interfaces ‣ Assignments

2. In the dropdown next to “New interface:”, select the WireGuard device (wg1 if this is your first one)

3. Add a description (eg HomeWireGuard)
> This is what will be visible under Interfaces on the menu.

4. Click the `add` button, then click Save


* * * 

### Enable new WireGuard interface

Select your new interface under the Interfaces menu.

Configure it as follows (if an option is not mentioned below, leave it as the default):

1. Enable - Checked

2. Lock - Checked

3. IPv4 Configuration Type - None
> There is no need to configure IPs on the interface. The tunnel address(es) specified in the Instance configuration for your server will be automatically assigned to the interface once WireGuard is restarted

4. Save

5. Apply


> When assigning interfaces, gateways can be added to them. This is useful if balancing traffic across multiple tunnels is required or in more complex routing scenarios. To do this, go to `System ‣ Gateways ‣ Configuration` and add a new gateway. Choose the relevant WireGuard interface and set the Gateway to dynamic. 
{: .prompt-info }


* * *

### Create a WireGuard outbound NAT rule

This step is only necessary (if at all) to allow client peers to access IPs outside of the local IPs/subnets behind OPNsense.

1. Go to Firewall ‣ NAT ‣ Outbound

2. Select “Hybrid outbound NAT rule generation” if it is not already selected, and click Save and then Apply changes

3. Click Add to add a new rule

4. Interface - WAN

5. TCP/IP Version - IPv4 or IPv6 (as applicable)

6. Protocol - any

7. Source invert - Unchecked

8. Source address - Select the assigned an interface or an alias (eg HomeWireGuard net )

9. Source port - any

10. Destination invert - Unchecked

11. Destination address - any

12. Destination port - any

13. Translation / target - Interface address

14. Description - Add one if you wish to







* * * 

### Finish WireGuard interface by reseting WireGuard services

The save and apply are meaningless, as wireguard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings > General` or go to the `Dashboard` and `reset services` from there. 

`Unbound DNS` requires a `reload` of Unbound DNS's services to get the new Wireguard interface added. 



* * *

## Setup firewall rules on OPNsense for WireGuard

This will involve two steps - first creating a firewall rule on the WAN interface to allow clients to connect to the OPNsense WireGuard server, and then creating a firewall rule to allow access by the clients to whatever IPs they are intended to have access to.


### Create firewall rules on WAN


1. Go to Firewall ‣ Rules ‣ WAN

2. Click Add to add a new rule

3. Action - Pass

4. Quick - Checked

5. Interface - WAN

6. Direction - in

7. Protocol - UDP

8. Source / Invert - Unchecked

9. Source - any

10. Destination / Invert - Unchecked

11. Destination - WAN address

12. Destination port range - The WireGuard port specified in the Instance configuration in Step 2

13. Description - Add one if you wish to

14. Save

15. Apply



### Create firewall rules on WireGuard

The firewall rule outlined below will need to be configured on the automatically created WireGuard group that appears once the Instance configuration is enabled and WireGuard is started. 

You will also need to manually specify the source IPs/subnet(s) for the tunnel. It’s probably easiest to define an alias (via Firewall ‣ Aliases) for those IPs/subnet(s) and use that. 

Although this is generally not recommended.


1. Go to Firewall > Rules > WireGuard (Group)

2. Click Add (Click on the + symbol) to add a new rule.

3. Action - Pass

4. Quick - Checked

5. Interface - Whatever interface you are configuring the rule on (eg HomeWireGuard ) - see note below

6. Direction - in

7. TCP/IP Version - IPv4 or IPv4+IPv6 (as applicable)

8. Protocol - any

9. Source / Invert - Unchecked

10. Source - Select the assigned an interface or an alias (eg HomeWireGuard net )

11. Destination / Invert - Unchecked

12. Destination - Specify the IPs that client peers should be able to access, eg “any” or specific IPs/subnets

13. Destination port range - any

14. Description - Add one if you wish to

15. Save

16. Apply


### Make normalization rules for WireGuard

By creating normalization rules, you ensure that IPv4 TCP can pass through the Wireguard tunnel without being fragmented. Otherwise you could get working ICMP and UDP, but some encrypted TCP sessions will refuse to work.

1. Go to Firewall ‣ Settings -> Normalization 

2. Press + to create one new normalization rule.

3. Interface - WireGuard (Group)

4. Direction - Any

5. Protocol - any

6. Source - any

7. Destination - any

8. Destination port - any

9. Description - Wireguard MSS Clamping IPv4

10. Max mss - 1380 (default) or 1372 if you use PPPoE; it’s 40 bytes less than your Wireguard MTU

11. Save the rule





* * * 

# PART VI - OPNsense router configuration for Site-to-Site VPNs

* * * 





* * *

### Site A - Meme Storage Bunker HQ's Server - Firewall setup


1. Go to Firewall ‣ Rules ‣ WAN 

2. add a new rule to allow incoming wireguard traffic from Site B.

3. Action - Pass

4. Interface - WAN

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - UDP

8. Source - 203.0.113.2

9. Destination - 203.0.113.1

10. Destination port - 51820

11. Description - Allow Wireguard from Site B to Site A


### Normalization rules for Site A's WireGuard

By creating the normalization rules, you ensure that IPv4 TCP can pass through the Wireguard tunnel without being fragmented. Otherwise you could get working ICMP and UDP, but some encrypted TCP sessions will refuse to work. 

1. Go to Firewall ‣ Settings -> Normalization 

2. Press + to create one new normalization rule.

3. Interface - WireGuard (Group)

4. Direction - Any

5. Protocol - any

6. Source - any

7. Destination - any

8. Destination port - any

9. Description - Wireguard MSS Clamping Site A

10. Max mss - 1380 (default); it’s 40 bytes less than your Wireguard MTU

11. Save the rule








* * *

### Site B - Sarah's Flower Shop Server - Firewall setup


1. Go to Firewall ‣ Rules ‣ WAN 

2. add a new rule to allow incoming wireguard traffic from Site A.

3. Action - Pass

4. Interface - WAN

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - UDP

8. Source - 203.0.113.1

9. Destination - 203.0.113.2

10. Destination port - 51820

11. Description - Allow Wireguard from Site A to Site B

12. Press Save and Apply.




### Normalization rules for Site B's WireGuard

By creating the normalization rules, you ensure that IPv4 TCP can pass through the Wireguard tunnel without being fragmented. Otherwise you could get working ICMP and UDP, but some encrypted TCP sessions will refuse to work. 

1. Go to Firewall ‣ Settings -> Normalization 

2. Press + to create one new normalization rule.

3. Interface - WireGuard (Group)

4. Direction - Any

5. Protocol - any

6. Source - any

7. Destination - any

8. Destination port - any

9. Description - Wireguard MSS Clamping Site B

10. Max mss - 1380 (default); it’s 40 bytes less than your Wireguard MTU

11. Save the rule





### Enable Wireguard on Site A and Site B

Go to VPN ‣ WireGuard ‣ Settings on both sites and Enable WireGuard

Press Apply and check VPN ‣ WireGuard ‣ Diagnostics. You should see Send and Received traffic and Handshake should be populated by a number. This happens as soon as the first traffic flows between the sites.

Your tunnel is now up and running.









* * * 

# PART VII - OPNsense firewall configuration for Site-to-Site VPNs

* * * 


## Allow traffic between Site A LAN Net and Site B LAN Net


1. Go to OPNsense Site A Firewall ‣ Rules ‣ LAN A add a new rule.

2. Action - Pass

3. Interface - LAN A

4. Direction - In

5. TCP/IP Version - IPv4

6. Protocol - Any

7. Source - 172.16.0.0/24

8. Source port - Any

9. Destination - 192.168.0.0/24

10. Destination port - Any

11. Description - Allow LAN Site A to LAN Site B

12. Press Save and Apply.



* * * 


1. Go to OPNsense Site A Firewall ‣ Rules ‣ Wireguard (Group) add a new rule.

2. Action - Pass

3. Interface - Wireguard (Group)

4. Direction - In

5. TCP/IP Version - IPv4

6. Protocol - Any

7. Source - 192.168.0.0/24

8. Source port - Any

9. Destination - 172.16.0.0/24

10. Destination port - Any

11. Description - Allow LAN Site B to LAN Site A

12. Press Save and Apply.



* * *


## Allow traffic between Site B LAN Net and Site A LAN Net

1. Go to OPNsense Site B Firewall ‣ Rules ‣ LAN A add a new rule.

2. Action - Pass

3. Interface - LAN B

4. Direction - In

5. TCP/IP Version - IPv4

6. Protocol - Any

7. Source - 192.168.0.0/24

8. Source port - Any

9. Destination - 172.16.0.0/24

10. Destination port - Any

11. Description - Allow LAN Site B to LAN Site A

12. Press Save and Apply.



* * *


1. Go to OPNsense Site B Firewall ‣ Rules ‣ Wireguard (Group) 

2. add a new rule.

3. Action - Pass

4. Interface - Wireguard (Group)

5. Direction - In

6. TCP/IP Version - IPv4

7. Protocol - Any

8. Source - 172.16.0.0/24

9. Source port - Any

10. Destination - 192.168.0.0/24

11. Destination port - Any

12. Description - Allow LAN Site A to LAN Site B

13. Press Save and Apply.

> Now both sites have full access to the LAN of the other Site through the Wireguard Tunnel. For additional networks just add more Allowed IPs to the Wireguard Endpoints and adjust the firewall rules to allow the traffic.
{: .prompt-info }



















## Edit the OPNSense firewall 

Add the normal open ports that you normally would. 
Also be sure
Wireguard subnets that you want to have internet acess need their subnets in: Firewall:NAT:Outbound

`Interface`: WAN
`Source`: whatever the `Local Tunnel` subnet is set to.



### OPNsense interface assignment

`Interfaces > Assignments`

1. Use the drop down and the `Description` box to name your new WireGuard instance, example `WG1_home`

2. `Add (Plus symbol)`

3. `Save`

* * *

You should now see your interface on the `Interfaces` list (left menu).

1. Click on the interface you named.

2. Once on the new page, `Enable Interface`.

3. `Save`

4. `Apply`




### OPNsense Firewall Rules

Let in your port you made for WireGuard.

1. Source is `WAN Address`

2. Dest is that port 

rest you knoe




### The rest of the Firewall stuff you knoe

1. Click on Firewall -> Rules -> WireGuard

2. Then on the + ADD button. 

3. Select `Single host or Network` as `source` and 

4. Enter the `IP range` of the `WireGuard network` and its subnet mask.

5. Save.

6. Looking back..

> Firewall -> Rules -> Wireguard, you should see -- under `Source` the IP Address of the Internal subnet you're using for you're Wireguard network you made under VPN -> Wireguard -> Local.














* * * 

# PART VI - The peer client

* * * 


## Setting up the client software

### Example: Official Windows WireGuard client

That client software we started up and then left out in the cold, hungry for input. 
Let's go do something with that client now.

1. Make sure the SAME client software is open, with the same `Public Key` that was copied earlier. No exit and starting over.

2. Copy this into your client config for tunnel generation:

```ini
[Interface]
PrivateKey = #somenumber#
Address = 10.10.10.2/32
DNS = #Internally Routed DNS. This is on the subnet you're VPNing into, example 172.23.1.254#

[Peer]
PublicKey = #HEYWAIT!---WeDontHaveThisYet---#
AllowedIPs = 0.0.0.0/0 #This is the subnet youre VPNing into, so this would be, example 172.32.1.0/24#
Endpoint = edge.sub.domain.com:51820
```

3. That's right! We still have to get the `Public Key` of our other confidant, the Server we're connecting to.

4. Moving over to the web interface, under `VPN > WireGuard > Settings > Local`

5. Edit the `Local Profile's Configuration` that we created earlier, named `HomeWireGuard` and copy the `Public Key` making sure to get **THE ENTIRE THING** including the `=`

6. Paste this into the config section of the client, under `[peer]`

7. Save





## Finish - by reseting services

The save and apply are meaningless, as wireguard never resets the service to load the new configuration. You must be sure to either `check` and `uncheck` the `Enable WireGuard` button in `Settings > General` or go to the `Dashboard` and `reset services` from there. 







In addition to AllowedIPs, you may also need to specify Endpoints in the client config as well:

`Endpoint = 192.95.5.69:51820`

So the client needs:

- Their Client Config

- Endpoint config of the "server"
  - Public IP of the Endpoint "server"
  - Public Key of the Endpoint "server"



* * *

So the last step using the WireGuard settings is to specifiy our clients that will be connecting to the different WireGuard services listening for connections. In this case we've only created one, `HomeWireGuard`




















************************
************************
Now you have to go back to the front end thing and use the drop down to select whatever you named your endpoint
************************
************************

edg0lxc+f5Vq+XKp/qPkfbkTBURfPYAjJat6zmKrJEw=


`THERE ARE A BUNCH OF STEPS FOR CREATING CONFIGS.` 
`THERE ARE SEVERAL AUTO GENERATING ONES ONLINE`
`BUT I SHOULD HAND WRITE THE STEPS HERE`

```
[Interface]
PrivateKey = MEwQUyAhO2HayVt6P6VId6i3tFwVydZGNKE8e5xxr3o=
Address = 172.23.55.69/32
DNS = 172.23.1.254

[Peer]
PublicKey = 0CP43e6zkyxJk61ZJmt2DIaj8Arq6cYESuXTX55aBng=
AllowedIPs = 172.23.1.0/24
Endpoint = opnsense.nocix.holtzweb.com:38365

```


Dude's LAN address:
10.122.2.178

I'm missing: Peer 1 created

Sending Handshake initirion to peer one.



RKMiB4La6jFHYG2pjn3TsCI02zPfVVlN6vQYeJDUqHc=



## Example wg0.conf 2024


```
[Interface]
Address = <Configured client IP>/<Netmask> // For exaple the IP "10.11.0.20/32"
PrivateKey = <Private Key of the client>

[Peer]
PublicKey = <Public Key of the OPNsense Wireguard instance>
AllowedIPs = <Networks to which this client should have access>/<Netmask>
             // For example "10.11.0.0/24, 192.168.1.0/24"
             //               |             |
             //               +--> The network area of the OPNsense WireGuard VPNs
             //                             |
             //                             +--> Network behind the firewall
Endpoint = <Public IP of the OPNsense firewall>:<WireGuard Port>
```
























































* * *



2024-03-24-opnsense-wireguard-vpn.md
---

Goals:

Wireguard setup:

- Road Warrior: this means client to network. Some machine, at some location wants onto your network. Could be a cell phone, laptop, or workstation -- it wants to remote in.
	- Screen shots of client device setup, on Linux, Windows, MacOS, and Android
- Site to Site: connecting two computer networks together. This requires knowing routing on the other end, and letting your system know the new gateway to route through. 
	- Screen shot second OPNsense setup at SFS headquarters.
https://forum.level1techs.com/t/infrastructure-series-wireguard-site-to-site-tunnel/168766

- Setup a VPS to wireguard to and from:
https://forum.level1techs.com/t/self-hosted-vpn-with-wireguard/160861
https://www.thomas-krenn.com/en/wiki/Ubuntu_Desktop_as_WireGuard_VPN_client_configuration
https://www.youtube.com/watch?v=yDgpBC7c1uY
- Devices_on_your_network_that_get_tunneled through an external VPN provider like Windscribe based on their network subnet and or IP address - This is also a wireguard config.
	- Possibly buy a month to make sure this works? Can I try out a new provider to taste that... say, mulvad?



