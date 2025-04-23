---
layout: post
title: OPNsense Unbound Multi-site Split-DNS with DNSCrypt
date: 2025-04-20 11:33:00 -0700
categories: [Networking, DNS]
tags: [opnsense, splitdns, multisite, reverseproxy, unbound, dnscrypt, security, networking, privacy]
pin: false
image:
  path: /assets/img/header/header--opnsense--unbound-dns-crypt-proxy-multisite-splitdns.jpg
  alt: Using OPNSense to serve Multi-site Split-DNS on Unbound and DNSCrypt-Proxy
---


# OPNsense DNS-crypt setup


## Part 1: Why Set Up DNSCrypt-Proxy on OPNsense

You know, you can setup DNS-Crypt on your PiHole too!


But mysetup is as follows: 

```text
Computer > DHCP (from OPNsense) > DNS (Zentyal) > PiHole (container) > OPNsense (internet)
```

Zentyal and PiHole have an interface on most of the VLANS and Subnets. Zentyal does all the caching, and handles Active Directory. PiHole is where I keep most of my block lists. Yes, I could use XXYY, but there are so many great apps/plugins/addons for PiHole. A small button that disables PiHole it for 10 seconds. Glorious. Oh, and then, yeah, OPNsense is the final source of truth. If OPNsense can't find it - it's Internet bound. 


### Why Use DNSCrypt instead of Doh or DoT?

There are a lot of reason we're using DNSCrypt instead of Doh or DoT. But the #1 reason:

- **DNSCrypt is often faster and lighter than the alternatives.**

> Your network can have the speediest upstream gateway, but if your DNS lookups are slow - your whole network will feel laggy.


- DoH and DoT can be more easily blocked or detected by network admins due to their protocol signatures.

> DoH uses standard HTTPS (TCP port 443), and it has very specific request patterns that can be fingerprinted. DoT uses TLS over TCP port 853 — if a connection is made to port 853, a firewall or IDS/IPS can immediately guess it's DoT traffic.


- DNSCrypt supports anonymized relays, client device cloaking, blocklists, and load balancing.

> If you use anonymized DNSCrypt with relays, like Tor, then the DNS resolver never sees your IP address. And the relay doesn’t know what you're asking — so you're functionally private.


- DNSCrypt encrypts DNS queries and also authenticates the responses using cryptographic signatures, ensuring they haven't been tampered with.

> DNSCrypt uses public-key cryptography and ephemeral key exchange, similar to how HTTPS works, but tailored for DNS.



## Part 2: Getting DNSCrypt Functioning


### Step 1: Install DNSCrypt-Proxy on OPNsense

Installing is as simple as visiting an app store for your firewall:

1. `System > Firmware > Plugins`

2. Look for and install "os-dnscrypt-proxy"

3. Wait for the installation to complete - you'll see a success message when finished.


### Step 2: Enable and Configure DNSCrypt-Proxy

1. `Services > DNSCrypt-Proxy > Configuration > General`

2. ✔️ **"Enable DNSCrypt-Proxy"**

3. Set the Listen Addresses to: `127.0.0.1:15353`


### Step 3: Additonal Settings

These next few are up to you. They will restrict what servers you're allowed to use.

   - ✔️ **"Use IPv4 Servers"**

> The setting above will let DNSCrypt-Proxy use IPv4 enabled servers

   - ✔️ **"Use DNSCrypt Servers"**

> Use DNSCrypt Servers is required for DNSCrypt-Proxy to use servers with DNSCrypt protocol enabled


   - ✔️ **"Use DNS-over-HTTPS Servers"**

> Allow DNSCrypt-Proxy to use servers with DNS-over-HTTPS protocol enabled


   - ✔️ **"Require DNSSEC"**

> This enables Domain Name System Security Extensions for additional authentication - like adding wax seals to verify authenticity

   - ✔️ **"Require NoLog"**

> Only use servers that don't log your DNS queries - choose relays that don't keep sender records

   - ✔️ **"Require NoFilter"**

> Only use DNS server without own blacklisting - avoid relays that open your query to remove "junk"

   - 👇 **"Fallback Resolver"**

> You can set this to whatever you prefer:  `185.222.222.222:53` for DNS.SB, or `9.9.9.9:53` for Quad9

   - ✔️ **"Block IPv6"**

> Immediately respond to IPv6-related queries with an empty response for faster resolution when no IPv6 connectivity exists

   - ✔️ **"Cache"**

> Enable a DNS cache to reduce latency and outgoing traffic

   - ✔️ **"Enable query logs"**

> This is useful during the initial setup for troubleshooting purposes:

   - 🦚 **"Server List"** - Try and find a location that [meets your requirements](https://dnscrypt.info/public-servers), and is [geograhically close](https://dnscrypt.info/map/).

> We will go through the process of selecting from the server list below...

   - Click Save to apply all settings




### Step 4: Select the Best Privacy-Focused DNSCrypt Provider

The [link](https://dnscrypt.info/public-servers/) to find these servers is under the help for `Server List` in Services > DNSCrypt-Proxy > Configuration


#### Criteria for Secure Providers

When selecting DNSCrypt providers, consider these criteria for maximum privacy and security:

| Feature | Why It Matters | Top Providers |
|---------|---------------|--------------|
| **No-Log Policy** | Prevents tracking your browsing history | dnscry.pt, Quad9 |
| **DNSSEC Support** | Verifies address authenticity | dnscry.pt, OpenDNS |
| **Multi-Protocol** | Works with all encryption types | CIRA Canadian Shield |
| **Local Jurisdiction** | Avoids problematic data laws | Swiss DNS |


#### Grab a name from the list

Sorry the [https://dnscrypt.info/public-servers/](https://dnscrypt.info/public-servers/) list is kind of long and with weird naming conventions. 

- Generally, find a `dnscry.pt`-`your-area`.

- For maximum privacy, consider selecting multiple providers and enabling provider rotation.

- Click Save after selecting your preferred providers.



## Part 3: Unbound's Upstream DNS Providers

### Step 1: Unbound Query Forwarding for remote self-hosted domains and DNSCrypt as the Primary External DNS Server

This section is for setting as many custom domains as you want. Then, you add a "catch-all" for DNSCrypt to grab anything not in that list (e.g. Internet).

Unbound handles local names so your devices can talk to each other, while DNSCrypt-Proxy encrypts everything going out. You get local access and privacy at the same time.


1. `Services > Unbound DNS > General`

2. ✔️ **"Enable Unbound DNS"**

3. Keep the listen port as `53`

4. Under Network Interfaces, select your specific LAN/VLANs and ⛔ **make sure to exclude "WAN"** ⛔


### Step 2: Setting your upstream DNS

1. `Services > Unbound DNS > Query Forwarding`

2. ⛔ "Use System Nameservers" - Make sure this is unchecked!

3. Click the "+" sign and add the following information, make sure to leave Domain blank, to forward all queries to the specified server.
 
   - Domain: <blank>
 
   - Server IP: 127.0.0.1

   - Server Port: 15353

   - Description: Upstream DNS-Crypt

4. Click Save & Apply


### Step 3: SET UP UPSTREAM REMOTE DNS HERE TOO!!!!!!!!!!!!

This allows you to have Multisite Split DNS. The internal requests still go through the private servers.
This is possible by setting query-forwarding to any of the internal domains you want to reach to your friends' subnet's OPNsense IP. Presumably this other friend has configured their OPNsense in the same way.

You also need to set any remote DNS servers you may have access too.
The idea here is that the remote server handles all of the DNS records.

I feel like querying the remote server is the way to go.

I dont need a copy of all of my friends' DNS records. But when I want to connect to one of their servers - it knows how to find it. Internally, on this subnet. 



## Part 4: Ensure no conflicting DNS entries are present

### Step 1: Remove old DNS and exclusivly use DNSCrypt-Proxy

You have many locations in OPNsense to set the DNS. They will almost all take presidence over DNSCrypt-Proxy. So please be sure they're not being used.

1. Disable DNS server override by DHCP/PPP on WAN:

   - Settings > General

   - Uncheck "Allow DNS server list to be overridden by DHCP/PPP on WAN"

   - Click Save & Apply


2. Did you set DNS servers in your DHCP?

   - Navigate to ISC DHCPv4 > Select an Interface

   - Verify that no DNS Servers are listed under DHCP settings


3. Verify no addional DNS Servers in Unbound:

Double check to be sure you dont have any Unbound DNS Servers set 

   - ` Services > Unbound DNS > DNS over TLS`

   - ⛔ Uncheck any of the servers that may have been enabled.

   - Click Save & Apply



### Step 2: Best Practices for Ongoing Management

Double Check You're Using Redundant, Reliable DNS Resolvers:

   - In Services > DNSCrypt-Proxy > Resolvers, regularly review and update your chosen providers

   - Consider using multiple trusted providers for redundancy




## Source material

YouTube Source: [How to Set Up DNSCrypt-Proxy on OPNsense](https://youtu.be/aK_-FprANl4)

Blog Source: [How to Set Up DNSCrypt-Proxy on OPNsense](https://sysadmin102.com/2025/03/how-to-set-up-dnscrypt-proxy-on-opnsense-protect-encrypt-your-dns-traffic/)


## More to come

This post is part of a larger project demonstrating multi-site split DNS using Unbound and HAProxy to route traffic based on SNI headers. 

The really fun part is most of this traffic being routed is doing so using the proxy protocol. So another reverse proxy, Traefik, with proxy protocol setup, is waiting on the other end.
