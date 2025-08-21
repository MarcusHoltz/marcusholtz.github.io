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

Automated IP blocklist aggregation with geolocation-based country filtering, Docker ready, and twice daily runs via GitHub Actions.

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

## What is This Project

I like the idea of fail2ban - I dont like the idea of [Crowdsec capturing all your private connection information](https://discourse.crowdsec.net/t/is-crowdsec-acting-against-european-privacy-regulations/1363){:target="_blank"}.

![Revealing all your private connection information to Crowdsec](/assets/img/posts/aggregator-crowdsec--thumb.png)


* * *

### Global Blocklist Sharing

As an alternative to Crowdsec, sharing ban blocklists globally can work, but only if you use trusted lists and keep updates in sync with upstream.

But alas, this would quickly exceed my firewall's default table size of 1,000,000, and you [do not want to](https://docs.opnsense.org/manual/firewall_settings.html#firewall-adaptive-timeouts){:target="_blank"} go over the number of entries.

![Picture of OPNSense firewall table limits](/assets/img/posts/opnsense--firewall-aliases-nearing-full--thumb.png)


* * *

### What Does The Solution Look Like

So, we can addresses this challenge by:

- **Aggregating** multiple public blocklists into a single source to compose the more comprehensive list possible

- **Optimizing** using [VLSM](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing){:target="_blank"} to reduce the number of entries

- **Geo-filtering** to focus on specific sections of your network not intended for global use, cutting connections down to countries relevant to only your infrastructure  

- **Automating** updates for a zero-maintenance operation

> Let's say "yourcountryhere" consists of .5 million addresses, that gives .5 million blocklist addresses to use (presuming firewall default table size of 1,000,000). 


* * *


### What Types of Blocklists are Used

The lists used in the [ipblocklist-geofiltered-aggregator](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator){:target="_blank"} are:

- **IP Address blocklists** with IPs placed into subnets with CIDR notation.

> These go straight into your iptables. Be it, on your firewall, edge gateway, router, proxy, transparent bridge, etc.

- **NOT** DNS blocklists, [DNS](https://github.com/hagezi/dns-blocklists) [Blocklists](https://oisd.nl) have dns123names456in789them.net

> DNS Blocklists are the kind that you might use with your PiHole, pfBlockerNG, Squid, AdGuard, Power DNS, etc.


* * *



### How Can I Use an IP Blocklist

- Here is a great example from the [Windgate Blog](https://windgate.net) on [How to use ip blocklists with OPNsense](https://windgate.net/opnsense-ip-blocklists-and-geo-ip-block-to-enhance-security-against-malicious-attacks).


- Another resource is [this collection of shell scripts](https://github.com/kravietz/blacklist-scripts) that are intended to block Linux systems and OpenWRT routers by using ip blocklists.


* * *

###  Defining Final Feature Set

The features I wanted in this project are:


#### Automated Processing

- **Scheduled Updates**: Automated updates that the user can set. Runs automatically via GitHub Actions.

- **Multiple Source Integration**: Combines various IP blocklist feeds into a de-duplicated minimized VLSM list.

- **Geographic Filtering**: Lets the user specific as few or as many countries to filter the aggregated blocklist through.

- **CIDR Optimization**: Reduces the total blocklist size through subnet aggregation in all blocklist generations.


#### Firewall Compatibility  

- **OPNSense/pfSense**: This must work with firewall aliases on M0n0wall-based firewalls.

- **iptables**: The lists generated need to be compatible with Linux-based deployments as well. 

- **OpenWRT**: Support with other router-based operating systems

- **Generic Format**: Standard CIDR notation for broad compatibility


#### Multiple Output Cusomizations

Generated blocklists appear in the `./data/output` directory and include:

- **Country-specific lists**: Separate files for each configured country

- **Aggregated formats**: Combined lists for multi-country deployments  


* * *


![Get your IPBlocklist Aggregator today! Now with Geofiltering, for only $19.00!](/assets/img/posts/aggregator-get-yours-today--450.png)

> A [$19.00 value](https://www.provya.com/12-subscriptions), free! 
{: .prompt-info }


* * *

## Using the ipblocklist-geofiltered-aggregator

### First Step - Install

1. **Sign in** to GitHub and navigate to [this repository](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator).

2. Click the **"Use this template"** button (in the upper right corner).

3. Select **Create a new repository**. Enter a name (e.g., `my-eu-badip-blocklist`), and confirm.

4. Your new repository is now independent — it will not share commit history with the original.

5. You can immediately begin editing or configuring it for your own multi-country IP aggregation project.

> The **"Use this template"** button on GitHub allows you to quickly create a new, independent repository pre-populated with the project's files and structure. Your new repository won't inherit commit history from the template. This is perfect for your personal blocklist repo.

*Usage is below for steps on running this repository with Github Actions in your new IP aggregation project.*


* * *

### Second Step - Set Your Permissions

6. **Enable Actions**: Go to Settings > Actions > General > Workflow permissions

7. **Set Permissions**: Select "Read and write permissions", click "Save".


* * *

### Third Step - Configure Your Cron

8. **Adjust cron**, it is how often your aggregator runs in `.github/workflows/ip-aggregation.yml` 

This repository chew through your Github Actions if you are using large lists to combine with many countries. 

You have a [total of 2000 min](https://docs.github.com/en/get-started/learning-about-github/githubs-plans#github-free-for-personal-accounts) per account per month.


* * *

### Fourth Step - How can this be configured

You need to setup the `.env` file.

9. **Configure Environment**: Edit `.env` file with your desired sources and countries

10. **Your Favorite Blocklists**: Load as many blocklists as you like, just make sure the line starts with `LIST1_`, `LIST2_`, `LIST3_`, etc.

11. **Multiple Countries**: Countries can be modified the same way, `COUNTRY_ISO_CODE_1`, `COUNTRY_NAME_1`, `COUNTRY_ISO_CODE_2`, `COUNTRY_NAME_2`, etc.

12. **Find Country Codes**: You can find your country codes in the [geoip2-ipv4 spreadsheet](https://datahub.io/core/geoip2-ipv4)

With a change to the `.env` file, the Github Actions will run.

You now have output!


* * *

#### Customizing your list

If you're going to customize the list, you should remove the `./data/output` folder, as it will only contain data pertinent to the current setup.

Be sure to remove the `./data/output` folder when you customize the countries. 

- This will ensure you dont include older, unused countries in your new aggreagtion lists.


* * *

#### GeoIP Aggregation

All of the information about what IP belongs to what country is pulled in from [Datopian's GeoIP2 IPv4 dataset](https://datahub.io/core/geoip2-ipv4){:target="_blank"}.

That link will provide a **Data Preview** section where you can quickly filter by `country_name` and `country_iso_code`.

This project assumes you are already doing country-based blocking. The best method for country-based blocking is to inverse match. Anything not matching the countries you are accepting traffic from, blocked. Then filter with this project's custom blocklists.

* * *


### How does this work

Most of the script is written in Python.

The magic to the script is:

- **PySubnetTree** – A patricia tree based CIDR lookup (from Zeek). 

Example usage:

```python
from SubnetTree import SubnetTree

# Build geographic lookup tree
tree = SubnetTree()
for cidr in us_networks:
    tree[cidr] = True

# Filter IPs by geographic location with O(log n) magic
us_ips = [ip for ip in all_ips if ip in tree]
```

> This script uses a Patricia trie, Python≥3.9, and it makes the lookups very efficient even with many prefixes. In benchmarks, PySubnetTree is much faster than naive loops.


* * *

## Differernt Use Cases and Deployment Scenarios

### Regional Service Protection

Perfect for services that primarily serve specific geographic regions:

- **E-commerce sites** focusing on domestic markets

- **Government services** restricted to national access

- **Regional content delivery** with geographic licensing

- **Corporate networks** with defined operational territories


### Infrastructure Security

Ideal for hardening network perimeters:

- **Edge gateway protection** against global threat sources

- **Server farm security** with country-based access control

- **VPN endpoint filtering** for geographic compliance

- **IoT device protection** in constrained environments


### Compliance and Governance

Supports regulatory requirements:

- **Data residency** mandates requiring geographic restrictions

- **Export control** compliance for sensitive technologies

- **Privacy regulations** limiting cross-border data flows

- **Financial services** with jurisdictional operating requirements





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





![The IPblocklist Geofiltered Aggregator Atari Game](/assets/img/posts/aggregator-game-cartridge-with-price--thumb.png)

