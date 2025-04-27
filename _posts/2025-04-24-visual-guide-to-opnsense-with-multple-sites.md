---
layout: post
title: Visual Guide to OPNsense multi-site with HAProxy, Unbound
date: 2025-04-24 11:33:00 -0700
categories: [DevOps, ReverseProxy]
tags: [opnsense, haproxy, splitdns, multisite, reverseproxy, cloud, server, unbound, dnscrypt, security, networking, administration, devops]
pin: false
image:
  path: /assets/img/header/header--opnsense--guide-visual-screenshots-haproxy-unbound-traefik.jpg
  alt: How-to guide with screenshots for setting up HAProxy Proxy Protocol and DNS on OPNsense with multiple sites
---

# Screenshot tutorial to use OPNsense for a Reverse Proxy using multiple domains with splitdns


* * *

# REMOTE LOCATION

## What is the current system these screen shots are from?

I am using two different desktops/browsers/systems but the same OPNsense between them.

### Linux system screenshots

![screenshots-system](/assets/img/posts/201_opnsense-haproxy-screenshots-system.png){: #cool-image }


### OPNSense version screenshot

![screenshots-opnsense-version](/assets/img/posts/202_opnsense-haproxy-screenshots-opnsense-version.png)



## DNSCrypt-Proxy Install

This is a screen shot of the installer page.

### os-dnscrypt-proxy installed

![dnscrypt-proxy-install](/assets/img/posts/203_opnsense-haproxy-dnscrypt-proxy-install.png)



## DNSCrypt-Proxy pick a server

This is the main configuration of DNSCrypt-Proxy.

### Enable DNSCrypt-Proxy and provide a listen address 

![dnscrypt-proxy-enable](/assets/img/posts/204_opnsense-haproxy-dnscrypt-proxy-enable.png)


### Click the 'i' icon to reveal the server list

![dnscrypt-proxy-find-server-list](/assets/img/posts/205_opnsense-haproxy-dnscrypt-proxy-find-server-list.png)


### Known Server list can be found in the help for Server List

![dnscrypt-proxy-found-server-list](/assets/img/posts/206_opnsense-haproxy-dnscrypt-proxy-found-server-list.png)


### Open this in a new page to pick a server

![dnscrypt-proxy-open-server-list](/assets/img/posts/207_opnsense-haproxy-dnscrypt-proxy-open-server-list.png)


### This is a demonstration of the list of servers available for DNSCrypt.

![dnscrypt-server-list](/assets/img/posts/208_opnsense-haproxy-dnscrypt-server-list.png)


### Pick one to meet your needs

![dnscrypt-server-list-selection](/assets/img/posts/209_opnsense-haproxy-dnscrypt-server-list-selection.png)


## Clear and previous DNS

To use our new DNS server we want to be sure any previous DNS entries have been removed or disabled.


### No DNS servers in the general settings

![cleardns-general-settings](/assets/img/posts/210_opnsense-haproxy-cleardns-general-settings.png)


### Dont allow WAN to set DNS settings

![cleardns-general-settings-no-wan-dns](/assets/img/posts/211_opnsense-haproxy-cleardns-general-settings-no-wan-dns.png)


### If you have DNS servers in Unbound's DNS over TLS, be sure they are not Enabled.

![cleardns-unbound-nds-over-tls](/assets/img/posts/212_opnsense-haproxy-cleardns-unbound-nds-over-tls.png)


### Make sure your upstream DNS server is OPNsense

You can set this to your PiHole if you like, or your Zentyal server.

But make sure their upstream DNS is going to be OPNsense.
Dont let them just jet out to the root for domain lookups.

![cleardns-dhcp-with-dns-to-opnsense](/assets/img/posts/213_opnsense-haproxy-cleardns-dhcp-with-dns-to-opnsense.png)


## Setting up Unbound

Unbound will still be the primary DNS provider for OPNsense, we just need to make some changes.


### Enable Unbound and use standard DNS port

![unbound-enable-service](/assets/img/posts/215_opnsense-haproxy-unbound-enable-service.png)


### Dont set your Network Interface to WAN. Remove that.

![unbound-disable-wan-interface](/assets/img/posts/216_opnsense-haproxy-unbound-disable-wan-interface.png)


## Query Forwarding with Unbound

Set the remote resolver with the domain to resolve, set the local resolver (DNSCrypt) and demo that the server ip is on a remote network. Not within our routes.

### This is the main page for Query Forwarding on Unbound

![unbound-query-forwarding-setup](/assets/img/posts/219_opnsense-haproxy-unbound-query-forwarding-setup.png)


### Add a new Server for DNSCrypt

![unbound-query-forwarding-dnscrypt-catch-all](/assets/img/posts/221_opnsense-haproxy-unbound-query-forwarding-dnscrypt-catch-all.png)


### Add an entry for any remote domains and corresponding DNS server

![unbound-query-forwarding-remote-domain-override](/assets/img/posts/222_opnsense-haproxy-unbound-query-forwarding-remote-domain-override.png)


### Looks like that Query Forwarding server is a remote server

![unbound-query-forwarding-told-you-it-was-a-remote-domain-override](/assets/img/posts/223_opnsense-haproxy-unbound-query-forwarding-told-you-it-was-a-remote-domain-override.png)


## Unbound local domain overrides

These are the domains on your network. No remote servers here. Be sure to list the IP Address as the HAProxy connection you want to make, if using a virtual IP or if just applying it to an interface on an OPNsense subnet.


### This is the Unbound overrides page

![unbound-local-domain-overrides](/assets/img/posts/224_opnsense-haproxy-unbound-local-domain-overrides.png)


### Setup a domain override for to a local address

![unbound-local-domain-host-override](/assets/img/posts/225_opnsense-haproxy-unbound-local-domain-host-override.png)


## Wireguard

Quick example setup of my Wireguard:

- Endpoint Addresses

- Public Key

- Allowed IPs


### The wireguard instance page

![wireguard-remote-instances-enabled-wireguard](/assets/img/posts/227_opnsense-haproxy-wireguard-remote-instances-enabled-wireguard.png)


### Create a wireguard instance

![wireguard-remote-create-instance](/assets/img/posts/229_opnsense-haproxy-wireguard-remote-create-instance.png)


### Demo how an instance is used by a peer to connect

![wireguard-remote-instance-being-used-by-peer](/assets/img/posts/230_opnsense-haproxy-wireguard-remote-instance-being-used-by-peer.png)


### The wireguard peer page

![wireguard-remote-peers](/assets/img/posts/231_opnsense-haproxy-wireguard-remote-peers.png)


### Create a wireguard peer

![wireguard-remote-create-peer-to-connect-to](/assets/img/posts/232_opnsense-haproxy-wireguard-remote-create-peer-to-connect-to.png)


### Looks like it works for me

![wireguard-remote-it-works-for-me](/assets/img/posts/233_opnsense-haproxy-wireguard-remote-it-works-for-me.png)


## HAProxy

This is the reverse proxy.



### Install HAProxy

![haproxy-install](/assets/img/posts/234_opnsense-haproxy-haproxy-install.png)


### HAProxy Real Servers Page

![haproxy-real-servers](/assets/img/posts/235_opnsense-haproxy-haproxy-real-servers.png)


### The address of the server with Traefik on it, also the proxy-protocol version.

![haproxy-enable-real-server-add-ip-port](/assets/img/posts/236_opnsense-haproxy-haproxy-enable-real-server-add-ip-port.png)


### Be sure to send-proxy to the receving end

![haproxy-enable-real-server-add-option-send-proxy](/assets/img/posts/237_opnsense-haproxy-haproxy-enable-real-server-add-option-send-proxy.png)


### HAProxy Backend Pools Page

![haproxy-backend-pools](/assets/img/posts/239_opnsense-haproxy-haproxy-backend-pools.png)


### Backend pool creation

Backend pool specifies:

- Protocol Mode: TCP

- Proxy Protocol Version: 2

- Real Servers to use: <yournamehere>

- Options pass-through for config: option tcp-smart-connect


![haproxy-backend-pool-creation-tcp-mode-proxy-protocol-version](/assets/img/posts/241_opnsense-haproxy-haproxy-backend-pool-creation-tcp-mode-proxy-protocol-version.png)


## Options pass-through for config: option tcp-smart-connect

![haproxy-backend-pool-creation-option-tcp-smart-connect](/assets/img/posts/242_opnsense-haproxy-haproxy-backend-pool-creation-option-tcp-smart-connect.png)


## HAProxy Conditions Page

![haproxy-conditions](/assets/img/posts/254_opnsense-haproxy-haproxy-conditions.png)


### Condition sets the domain to look out for.

![haproxy-create-condition-host-contains](/assets/img/posts/256_opnsense-haproxy-haproxy-create-condition-host-contains.png)


### HAProxy Rules Page

![haproxy-rules](/assets/img/posts/263_opnsense-haproxy-haproxy-rules.png)


### Rule specifies what to do when your condition comes up. Use the correct pool for the subdomain.

![haproxy-if-rule-contains-condition-use-backend-pool](/assets/img/posts/265_opnsense-haproxy-haproxy-if-rule-contains-condition-use-backend-pool.png)


### Here is  the HAProxy frontend page, public servers

![haproxy-public-services](/assets/img/posts/276_opnsense-haproxy-haproxy-public-services.png)


### Frontend listen address and type

Frontend is what people connect to.

You need to set the address, and port, this could be a virtual IP.

You also need to be sure the:

- Type: SS/HTTPS (TCP mode)

- Advanced Settings: 

There are a bulk of things to put here. They all help, but really you just need req_ssl_hello_type 1

- Select Rules: Enter the rule tied to the backend for which this front end will go to.

- Save and Apply.

![haproxy-create-public-service-type-tcp](/assets/img/posts/278_opnsense-haproxy-haproxy-create-public-service-type-tcp.png)


### Frontend option pass-through and rules

![haproxy-create-public-service-options-and-rules](/assets/img/posts/280_opnsense-haproxy-haproxy-create-public-service-options-and-rules.png)


## Apply all changes

![haproxy-apply-haproxy-changes](/assets/img/posts/281_opnsense-haproxy-haproxy-apply-haproxy-changes.png)


* * *

# LOCAL LOCATION



## text

![system](/assets/img/posts/101_opnsense-haproxy-system.png){: #cool-image }


## text

![opnsense-version](/assets/img/posts/102_opnsense-haproxy-opnsense-version.png)


## text

![dnscrypt-proxy-install](/assets/img/posts/103_opnsense-haproxy-dnscrypt-proxy-install.png)


## text

![dnscrypt-proxy-enable](/assets/img/posts/104_opnsense-haproxy-dnscrypt-proxy-enable.png)


## text

![dnscrypt-proxy-find-server-list](/assets/img/posts/105_opnsense-haproxy-dnscrypt-proxy-find-server-list.png)


## text

![dnscrypt-proxy-found-server-list](/assets/img/posts/106_opnsense-haproxy-dnscrypt-proxy-found-server-list.png)


## text

![dnscrypt-proxy-open-server-list](/assets/img/posts/107_opnsense-haproxy-dnscrypt-proxy-open-server-list.png)


## text

![dnscrypt-server-list](/assets/img/posts/108_opnsense-haproxy-dnscrypt-server-list.png)


## text

![dnscrypt-server-list-selection](/assets/img/posts/109_opnsense-haproxy-dnscrypt-server-list-selection.png)


## text

![cleardns-general-settings](/assets/img/posts/110_opnsense-haproxy-cleardns-general-settings.png)


## text

![cleardns-general-settings-no-wan-dns](/assets/img/posts/111_opnsense-haproxy-cleardns-general-settings-no-wan-dns.png)


## text

![cleardns-unbound-dns-over-tls](/assets/img/posts/112_opnsense-haproxy-cleardns-unbound-dns-over-tls.png)


## text

![cleardns-dhcp-with-dns-to-opnsense](/assets/img/posts/113_opnsense-haproxy-cleardns-dhcp-with-dns-to-opnsense.png)


## text

![unbound-query-forwarding-setup](/assets/img/posts/119_opnsense-haproxy-unbound-query-forwarding-setup.png)


## text

![unbound-query-forwarding-dnscrypt-catch-all](/assets/img/posts/121_opnsense-haproxy-unbound-query-forwarding-dnscrypt-catch-all.png)


## text

![unbound-query-forwarding-remote-domain-override](/assets/img/posts/122_opnsense-haproxy-unbound-query-forwarding-remote-domain-override.png)


## text

![unbound-query-forwarding-told-you-it-was-a-remote-domain-override](/assets/img/posts/123_opnsense-haproxy-unbound-query-forwarding-told-you-it-was-a-remote-domain-override.png)


## text

![unbound-local-domain-overrides.png](/assets/img/posts/124_opnsense-haproxy-unbound-local-domain-overrides.png.png)


## text

![unbound-local-domain-host-override.png](/assets/img/posts/125_opnsense-haproxy-unbound-local-domain-host-override.png.png)


## text

![wireguard-local-instances-enabled-wireguard](/assets/img/posts/127_opnsense-haproxy-wireguard-local-instances-enabled-wireguard.png)


## text

![wireguard-local-create-instance](/assets/img/posts/129_opnsense-haproxy-wireguard-local-create-instance.png)


## text

![wireguard-local-instance-being-used-by-peer](/assets/img/posts/130_opnsense-haproxy-wireguard-local-instance-being-used-by-peer.png)


## text

![wireguard-local-peers](/assets/img/posts/131_opnsense-haproxy-wireguard-local-peers.png)


## text

![wireguard-local-create-peer-to-connect-to](/assets/img/posts/132_opnsense-haproxy-wireguard-local-create-peer-to-connect-to.png)


## text

![wireguard-local-peer-and-remote-instance](/assets/img/posts/133_opnsense-haproxy-wireguard-local-peer-and-remote-instance.png)


## text

![wireguard-remote-it-works-for-me](/assets/img/posts/134_opnsense-haproxy-wireguard-remote-it-works-for-me.png)


## text

![haproxy-real-servers](/assets/img/posts/135_opnsense-haproxy-haproxy-real-servers.png)


## text

![haproxy-enable-real-server-add-ip-port](/assets/img/posts/136_opnsense-haproxy-haproxy-enable-real-server-add-ip-port.png)


## text

![haproxy-enable-real-server-add-option-send-proxy](/assets/img/posts/137_opnsense-haproxy-haproxy-enable-real-server-add-option-send-proxy.png)


## text

![haproxy-backend-pools](/assets/img/posts/139_opnsense-haproxy-haproxy-backend-pools.png)


## text

![haproxy-backend-pool-creation-tcp-mode-proxy-protocol-version](/assets/img/posts/141_opnsense-haproxy-haproxy-backend-pool-creation-tcp-mode-proxy-protocol-version.png)


## text

![haproxy-backend-pool-creation-option-tcp-smart-connect](/assets/img/posts/142_opnsense-haproxy-haproxy-backend-pool-creation-option-tcp-smart-connect.png)


## text

![haproxy-conditions](/assets/img/posts/154_opnsense-haproxy-haproxy-conditions.png)


## text

![haproxy-create-condition-host-contains](/assets/img/posts/156_opnsense-haproxy-haproxy-create-condition-host-contains.png)


## text

![haproxy-rules](/assets/img/posts/163_opnsense-haproxy-haproxy-rules.png)


## text

![haproxy-if-rule-contains-condition-use-backend-pool](/assets/img/posts/165_opnsense-haproxy-haproxy-if-rule-contains-condition-use-backend-pool.png)


## text

![haproxy-public-services](/assets/img/posts/176_opnsense-haproxy-haproxy-public-services.png)


## text

![haproxy-create-public-service-type-tcp](/assets/img/posts/178_opnsense-haproxy-haproxy-create-public-service-type-tcp.png)


## text

![haproxy-create-public-service-options-and-rules](/assets/img/posts/180_opnsense-haproxy-haproxy-create-public-service-options-and-rules.png)


## text

![haproxy-apply-haproxy-changes](/assets/img/posts/181_opnsense-haproxy-haproxy-apply-haproxy-changes.png)




