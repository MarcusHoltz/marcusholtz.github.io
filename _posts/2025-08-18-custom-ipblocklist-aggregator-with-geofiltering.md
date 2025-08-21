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

This tool automatically collects IP addresses from different blocklists and combines them into one big list, then filters out only the IP addresses from specific countries you choose. It runs by itself using GitHub to keep your blocked IP list fresh and updated.


* * *

For the repo that goes along with this guide, visit: [https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator){:target="_blank"}.

* * *






- [Country Based IP Address Internet Blocklist Aggregator](#country-based-ip-address-internet-blocklist-aggregator)
  - [Why this](#why-this)
  - [What are the alternatives](#what-are-the-alternatives)
  - [How can this be used](#how-can-this-be-used)
  - [How does this work](#how-does-this-work)
    - [Python code](#python-code)
  - [Customizing your list](#customizing-your-list)
  - [GeoIP Aggregation](#geoip-aggregation)
  - [IP Blocklist list](#ip-blocklist-list)
    - [Block list suggestions](#block-list-suggestions)
- [Github Blog Post](#github-blog-post)


[![The IPblocklist Geofiltered Aggregator Atari Game](/assets/img/posts/aggregator-game-cartridge--thumb.png)]((/assets/img/posts/aggregator-game-cartridge.jpg))


## Why this

I like the idea of fail2ban, I like the idea of a shared global fail2ban, I dont like [the idea of Crowdsec](https://discourse.crowdsec.net/t/is-crowdsec-acting-against-european-privacy-regulations/1363){:target="_blank"}.

So I decided I should get aggregate all the public block lists I can find, but alas, this would quickly exceed my firewall's default table size of 1000000, and you [do not want to](https://docs.opnsense.org/manual/firewall_settings.html#firewall-adaptive-timeouts){:target="_blank"} go over the number of entries.


![Picture of OPNSense firewall table limits](/assets/img/posts/opnsense--firewall-aliases-nearing-full.png)

So, we need to cut this list down, and refine it by [VLSM](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing){:target="_blank"}.

Cutting it down even more, why block everything across the globe? 
Sections of my network are not intended for global use, but within <your_country_here>'s telecom. 

- If <your_country_here> is only .5 million addresses, that will give us .5 million of blocklist addresses we can use (presuming firewall default table size of 1000000). 

Let's do that.

But I need it to run when I am asleep and be publicly available.

- Enter:  [ipblocklist-geofiltered-aggregator](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator){:target="_blank"} repo.

> A [$19.00 value](https://www.provya.com/12-subscriptions), free!


* * *

## What are the alternatives

The lists in the [ipblocklist-geofiltered-aggregator](https://github.com/MarcusHoltz/ipblocklist-geofiltered-aggregator){:target="_blank"} are IP Address blocklists and are different from [DNS](https://github.com/hagezi/dns-blocklists) [Blocklists](https://oisd.nl) that you might use on your PiHole.

These go straight into your iptables. Be it, on your firewall, edge gateway, router, proxy, transparent bridge, etc.

- Here is a great example from the [Windgate Blog](https://windgate.net) on [How to use ip blocklists with OPNsense](https://windgate.net/opnsense-ip-blocklists-and-geo-ip-block-to-enhance-security-against-malicious-attacks).


- Another resource is [this collection of shell scripts](https://github.com/kravietz/blacklist-scripts) that are intended to block Linux systems and OpenWRT routers by using ip blocklists.

- If you want a web app that can do mostly the aggrigation, you can find [Catcusd](https://github.com/m0zgen/cactusd/). 


* * *

## How can this be used

This will chew through your Github Actions if you are using a few large lists to combine:

https://docs.github.com/en/get-started/learning-about-github/githubs-plans#github-free-for-personal-accounts

You have a total of 2000 min per account per month.



## How does this work


You need to setup the .env file first.

With a change to the .env file, the Github Actions will run.

You now have output!


### Python code





The magic to the script is:

- **PySubnetTree** – A patricia tree based CIDR lookup (from Zeek). 

Example usage:

```python
from SubnetTree import SubnetTree

tree = SubnetTree()
for cidr in us_cidr_list:
    tree[cidr] = True

us_ips = [ip for ip in all_ips if ip in tree]
```

Internally it uses a Patricia trie, so lookup is efficient even with many prefixes. In benchmarks, PySubnetTree is a bit slower than PyTricia but still much faster than naive loops.

It requires Python≥3.9 on *nix (no Windows support). 
Installation is via pip or source (no official PyPI but GitHub has setup).  

Trade-offs: PyTricia/PySubnetTree use extra memory (a tree structure vs a flat list), but for ~200k prefixes it’s modest (tens of MB at most). 
They require installing an external package (not pure stdlib), but both are pip-installable C extensions. 
They preserve exact accuracy (no approximation).







## Customizing your list

If you're going to customize the list, you should remove the `./data/output` folder, as it will only contain data pertinant to the current setup.

Be sure to remove the `./data/output` folder when you customize the countries. 

- This will ensure you dont include older, unused countries in your new aggreagtion lists.



## GeoIP Aggregation

All of the information about what IP belongs to what country is pulled in from [Datopian's GeoIP2 IPv4 dataset](https://datahub.io/core/geoip2-ipv4){:target="_blank"}.

That link will provide a **Data Preview** section where you can quickly filter by `country_name` and `country_iso_code`.







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


and many more can be found at [MISP Threat Sharing](https://www.misp-project.org/feeds) and at [https://threatfeeds.io](https://threatfeeds.io).


* * *















# Github Blog Post


Why all of these countries?

Oh, I own a place there. 

Arent they spread across the world seemingly randomy?

Exactly.


















































































