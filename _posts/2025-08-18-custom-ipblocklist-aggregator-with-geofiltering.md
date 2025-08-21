---
layout: post
title: Geofiltered IP Address Blocklists Aggregator
date: 2025-08-18 11:33:00 -0700
categories: [Networking, Security]
tags: [security, server, docker, firewall, blocklist, geolocation, iptables, opnsense, pfsense, automated-deployment, ipset, networking, security, ipblocklist, ip-filtering, blocklist-aggregator]
pin: false
image:
  path: /assets/img/header/header--network--country-based-ip-blocklist-aggregator.jpg
  alt: Aggregate IP Blocklists and then Geofilter by Country
---

# Country Based IP Address Internet Blocklist Aggregator

- 📥 Automatically collects IP addresses from the different blocklists you've configured

- 🦺 Combines all the IP addresses into one VLSM list

- 🌐 Filters geographically to show only IP addresses from specific countries you choose

- 🤖 Runs autonomously using GitHub Actions to keep your blocked IP list fresh and updated


* * *

For the repo that goes along with this guide, visit: 

[https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator){:target="_blank"}.

* * *



![The IPblocklist Geofiltered Aggregator Atari Game, Free! A $19.00 value!](/assets/img/posts/aggregator-game-cartridge.jpg)


* * *

- [Country Based IP Address Internet Blocklist Aggregator](#country-based-ip-address-internet-blocklist-aggregator)
  - [What is This Project](#what-is-this-project)
    - [Global Blocklist Sharing](#global-blocklist-sharing)
    - [What Would Our Solution Look Like](#what-would-our-solution-look-like)
    - [What Types of Blocklists are Used](#what-types-of-blocklists-are-used)
    - [How Can I Use an IP Blocklist](#how-can-i-use-an-ip-blocklist)
    - [Defining Features](#defining-features)
      - [Automated Processing](#automated-processing)
      - [Firewall Compatibility](#firewall-compatibility)
      - [Multiple Output Cusomizations](#multiple-output-cusomizations)
  - [How can this be used](#how-can-this-be-used)
    - [How can this be configured](#how-can-this-be-configured)
      - [Customizing your list](#customizing-your-list)
      - [GeoIP Aggregation](#geoip-aggregation)
    - [How does this work](#how-does-this-work)
      - [**⚡ Performance Characteristics**](#-performance-characteristics)
      - [**⚖️ Accuracy vs Performance**](#️-accuracy-vs-performance)
  - [Use Cases and Deployment Scenarios](#use-cases-and-deployment-scenarios)
    - [🌍 Regional Service Protection](#-regional-service-protection)
    - [🛡️ Infrastructure Security](#️-infrastructure-security)
    - [📋 Compliance and Governance](#-compliance-and-governance)
  - [IP Blocklist list](#ip-blocklist-list)
    - [Block list suggestions](#block-list-suggestions)





## What is This Project

I like the idea of fail2ban - I dont like the idea of [Crowdsec capturing all your private connection information](https://discourse.crowdsec.net/t/is-crowdsec-acting-against-european-privacy-regulations/1363){:target="_blank"}.

![Revealing all your private connection information to Crowdsec](/assets/img/posts/aggregator-crowdsec--thumb.png)


* * *

### Global Blocklist Sharing

As an alternative to Crowdsec, sharing ban blocklists globally can work, but only if you use trusted lists and keep updates in sync with upstream.

But alas, this would quickly exceed my firewall's default table size of 1,000,000, and you [do not want to](https://docs.opnsense.org/manual/firewall_settings.html#firewall-adaptive-timeouts){:target="_blank"} go over the number of entries.

![Picture of OPNSense firewall table limits](/assets/img/posts/opnsense--firewall-aliases-nearing-full--thumb.png)


* * *

### What Would Our Solution Look Like

So, we can addresses these challenges by:

- **Aggregating** multiple public blocklists into a single source to compose the more comprehensive list possible

- **Optimizing** using [VLSM](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing){:target="_blank"} to reduce the number of entries

- **Geo-filtering** to focus on specific sections of your network not intended for global use, cutting connections down to countries relevant to your infrastructure  

- **Automating** updates through GitHub Actions for a zero-maintenance operation

> If <your_country_here> is only .5 million addresses, that will give us .5 million of blocklist addresses we can use (presuming firewall default table size of 1,000,000). 


* * *


### What Types of Blocklists are Used

The lists used in the [ipblocklist-geofiltered-aggregator](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator){:target="_blank"} are:

- **IP Address blocklists** with IPs placed into subnets with CIDR notation.

> These go straight into your iptables. Be it, on your firewall, edge gateway, router, proxy, transparent bridge, etc.

- **NOT** `DNS blocklists`, [DNS](https://github.com/hagezi/dns-blocklists) [Blocklists](https://oisd.nl) with dns123names456in789them.net

> DNS Blocklists are the kind that you might use with your PiHole, pfBlockerNG, Squid, AdGuard Home, Power DNS, etc.


* * *



### How Can I Use an IP Blocklist

- Here is a great example from the [Windgate Blog](https://windgate.net) on [How to use ip blocklists with OPNsense](https://windgate.net/opnsense-ip-blocklists-and-geo-ip-block-to-enhance-security-against-malicious-attacks).


- Another resource is [this collection of shell scripts](https://github.com/kravietz/blacklist-scripts) that are intended to block Linux systems and OpenWRT routers by using ip blocklists.


* * *

###  Defining Features

#### Automated Processing

- **⏰ Scheduled Updates**: Runs automatically via GitHub Actions

- **📊 Multiple Source Integration**: Combines various threat intelligence feeds

- **🌐 Geographic Filtering**: Focuses on user-specified countries

- **📐 CIDR Optimization**: Reduces blocklist size through subnet aggregation


#### Firewall Compatibility  

- **🔥 OPNSense/pfSense**: Direct integration with firewall aliases

- **🐧 iptables**: Compatible with Linux-based systems

- **📡 OpenWRT**: Works with router-based implementations

- **🔧 Generic Format**: Standard CIDR notation for broad compatibility


#### Multiple Output Cusomizations

Generated blocklists appear in the `./data/output` directory and include:

- **🌍 Country-specific lists**: Separate files for each configured country

- **📋 Aggregated formats**: Combined lists for multi-country deployments  



![Get your IPBlocklist Aggregator today! Now with Geofiltering, for only $19.00!](/assets/img/posts/aggregator-get-yours-today--450.png)



> A [$19.00 value](https://www.provya.com/12-subscriptions), free! 
{: .prompt-info }

* * *


## How can this be used

This will chew through your Github Actions if you are using a few large lists to combine:

https://docs.github.com/en/get-started/learning-about-github/githubs-plans#github-free-for-personal-accounts

You have a total of 2000 min per account per month.

* * *


### How can this be configured


You need to setup the .env file first.

With a change to the .env file, the Github Actions will run.

You now have output!


* * *

#### Customizing your list

If you're going to customize the list, you should remove the `./data/output` folder, as it will only contain data pertinant to the current setup.

Be sure to remove the `./data/output` folder when you customize the countries. 

- This will ensure you dont include older, unused countries in your new aggreagtion lists.



#### GeoIP Aggregation

All of the information about what IP belongs to what country is pulled in from [Datopian's GeoIP2 IPv4 dataset](https://datahub.io/core/geoip2-ipv4){:target="_blank"}.

That link will provide a **Data Preview** section where you can quickly filter by `country_name` and `country_iso_code`.


* * *


### How does this work


The magic to the script is:

- **PySubnetTree** – A patricia tree based CIDR lookup (from Zeek). 

Example usage:

```python
from SubnetTree import SubnetTree

# Build geographic lookup tree
tree = SubnetTree()
for cidr in us_cidr_list:
    tree[cidr] = True

# Filter IPs by geographic location
us_ips = [ip for ip in all_ips if ip in tree]
```

> This script uses a Patricia trie, Python≥3.9, and it makes the lookups very efficient even with many prefixes. In benchmarks, PySubnetTree is much faster than naive loops.

#### **⚡ Performance Characteristics**

- **💾 Memory Usage**: Modest overhead (tens of MB for ~200k prefixes)

- **🚀 Lookup Speed**: Efficient even with large prefix sets

- **🖥️ Platform Support**: Python ≥3.9 on Unix systems (no Windows support)

- **📦 Installation**: pip-installable C extensions


#### **⚖️ Accuracy vs Performance**

- **🎯 Exact Matching**: No approximation, preserves complete accuracy

- **🌳 Tree Structure**: More memory than flat lists, but faster lookups

- **📚 External Dependencies**: Requires PySubnetTree package installation






* * *

## Use Cases and Deployment Scenarios

### 🌍 Regional Service Protection

Perfect for services that primarily serve specific geographic regions:

- **🛒 E-commerce sites** focusing on domestic markets

- **🏛️ Government services** restricted to national access

- **📺 Regional content delivery** with geographic licensing

- **🏢 Corporate networks** with defined operational territories


### 🛡️ Infrastructure Security

Ideal for hardening network perimeters:

- **🌐 Edge gateway protection** against global threat sources

- **🖥️ Server farm security** with country-based access control

- **🔐 VPN endpoint filtering** for geographic compliance

- **📱 IoT device protection** in constrained environments


### 📋 Compliance and Governance

Supports regulatory requirements:

- **🗄️ Data residency** mandates requiring geographic restrictions

- **🚫 Export control** compliance for sensitive technologies

- **🔒 Privacy regulations** limiting cross-border data flows

- **🏦 Financial services** with jurisdictional operating requirements





* * *

## IP Blocklist list

The `.env` file has many blocklists available for you to look through and uncomment. 


### Block list suggestions

There are many block lists on the internet, it can be a bit tiresome to try...

Dont dig for gold! I'll just lay it down on the ground infront of you:


* * *

[Sakib Mahmud's](https://github.com/sakib-m) - [IP-Prefix-List](https://github.com/sakib-m/IP-Prefix-List)

This repository contains updated IP Prefix list of major Internet Companies. 

> Updated every 24 hours

* * *


[Fnutt Consulting's](https://fnutt.net) - [Open Dynamic Block Lists](https://opendbl.net)

These lists conain blocklists with standalone ip addressess from Fnutt's [exchange points](https://www.peeringdb.com/net/4786).

> Updated every 15 min

* * *


[Sentinel IPS's](https://nomicnetworks.com) - [CINS Army list](https://cinsscore.com)

Leveraging data from our network of Sentinel devices and other trusted InfoSec sources, CINS is a Threat Intelligence database that provides an accurate and timely score for any IP address in the world.

> Updated as attacks appear

* * *


[Proofpoint's](https://www.proofpoint.com) - [Emerging Threats Intelligence](https://rules.emergingthreats.net/fwrules) (you want the `emerging-Block-IPs.txt`)

Emerging Threats is a division of Proofpoint, contributed and maintained by the security community.

> Updated daily

* * *


[FireHOL's](http://firehol.org/) - [Cybercrime IP Feeds](https://iplists.firehol.org)

FireHOL's objective is to create a blacklist that can be safe enough to be used on all systems, with a firewall, to block access entirely, from and to its listed IPs.

> Updated automatically every time any of its IP lists is updated

* * *

[Spamhaus's](https://www.spamhaus.org/) - [DROP lists](https://www.spamhaus.org/blocklists/do-not-route-or-peer)

Spamhaus is weird about this list and is trying to convert from TXT files to JSON, so they just dont give out these links anymore: [DROP list](https://www.spamhaus.org/drop/drop.txt) and [EDROP list](https://www.spamhaus.org/drop/edrop.txt)

> Updated daily

* * *


[Bitwire's](https://github.com/bitwire-it) - [ipblocklist](https://github.com/bitwire-it/ipblocklist)

This project provides aggregated IP blocklists for inbound and outbound traffic, updated every 2 hours. It includes exclusions for major public DNS resolvers to prevent legitimate services from being blocked.

> Updated every 2 hours

* * *


[Romain Marcoux's](https://github.com/romainmarcoux) - [Malicious IP](https://github.com/romainmarcoux/malicious-ip) repository

Aggregation of lists of malicious IP addresses such as scanners and bruteforce, to be blocked in the WAN > LAN direction, integrated into firewalls: FortiGate, Palo Alto, pfSense, IPtables.

> Updated every hour

* * *

[Abuse.ch's](https://abuse.ch/) - [Feodo Tracker](https://feodotracker.abuse.ch/blocklist)

The purpose of the project is to identify botnet command&control servers (C&C) associated with a Feodo malware variant and provide a blocklist so that the community can protect themselves from the threat.

> Updated every 15 min

* * *


- and many more can be found at [MISP Threat Sharing](https://www.misp-project.org/feeds) and at [https://threatfeeds.io](https://threatfeeds.io).


- If you want a web app that can do mostly the aggrigation, you can find [Catcusd](https://github.com/m0zgen/cactusd/). 


* * *








This automated, geo-filtered IP blocklist aggregator provides enterprise-grade security capabilities while remaining completely free and open-source. By focusing on specific countries and optimizing for firewall capacity limits, it delivers targeted protection without the overhead of global-scale blocking solutions. 🎯✨




























![The IPblocklist Geofiltered Aggregator Atari Game](/assets/img/posts/aggregator-game-cartridge--thumb.png)




















































