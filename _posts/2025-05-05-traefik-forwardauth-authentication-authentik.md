---
layout: post
title: Authentik on a MacVLAN bridge with Traefik
date: 2025-05-05 11:33:00 -0700
categories: [DevOps, Authentication]
tags: [server, opensource, devops, authentication, reverseproxy, traefik, macvlan, docker, security, networking, authentik, forwardauth, oidc, oauth]
pin: false
image:
  path: /assets/img/header/header--traefik--authentik-macvlan-setup-oauth-oidc-forwardauth.jpg
  alt: MacVLAN bridge for Traefik reverse proxy to communicate with Authentik
---

# Setting Up Bridged Docker MacVLAN Network for Authentik to Access Traefik


I want to keep Traefik on it's own IP, but now I need [host to container communication](https://blog.holtzweb.com/posts/docker-macvlan-for-traefik-analytic-dashboards/#step-4-enable-host-to-container-communication) to allow Authentik to pass tokens.

The OAuth flow requires bidirectional communication between services that may be on different networks (Traefik), that OAuth token exchange requires multiple HTTP requests between the services and Authentik.


* * *

To facilitate this: 

- Traefik has it's own IP address on host network via a MacVLAN.

- The host's IP is used for all other containers, standard to how docker normally operates.


* * *

For the repo that goes along with this guide, visit https://github.com/MarcusHoltz/Authentik-Traefik-MacVLAN.



* * *

## Demo

![demo](#)



* * *

## Install Script

The only requirement is Debian 12. 

The rest of the script covers all materials needed to have:

- MacVLAN on the host interface 

- Docker MacVLAN for Traefik

- MacVLAN bridge for Authentik

- Traefik's access logs with original source headers

- Analytic dashboard with Promtail/Loki/Grafana


* * *

### System Requirements

Make sure this is a Debian 12 system.


* * *

### Script Features

- **Automated Traefik + MacVLAN Setup:** Configures Traefik with a MacVLAN Docker network for simplified reverse proxy.

- **Docker Installation** (Optional): Installs Docker if not already present on the system.

- **Automatic Network Information:** Detects and stores host network details (IP, gateway, subnet).

- **Systemd-Networkd Integration:** Generates and applies systemd network configuration files for MacVLAN bridge support.

- **ifupdown networking disable**: Disables ifupdown2 networking if systemd-networkd is enabled.

- **Context-Aware Configuration**: Adapts its configuration steps based on whether it is running before or after a system reboot, ensuring proper setup in either scenario.

- **LXC virtualization check**: Will alter the script depending on the virtual environment.

- **Dynamic IP Assignment:** Automatically assigns an IP address to the Traefik MacVLAN for access.

- **Docker Network Management:** Creates `proxy2traefik` for Docker container communication and `traefik2host` for Traefik's personal MacVLAN, along with isolated networks for all databases.

- **Update .env File**: Save all collected variables and Authentik secret key.

- **Configuration File Management:** Verifies, downloads, and sets up necessary configuration files for Traefik, Promtail, and Grafana.

- **Docker Compose Deployment:** Deploys the entire Authentik and Traefik stack using Docker Compose.

- **Informational Output**: Provides post-configuration instructions, including DNS record setup and application access URLs, to guide the user.



* * *

### MacVLAN WARNING!!

* * *

> ::Information about MacVLAN:: **All ports** are **exposed** by MacVLAN.
> This is fine when Traefik is only serving `80` and `443`, but this setup includes port `8080`.
> Additionally, you may want to secure your Traefik metric endpoints, like, `/metrics` or `/stats` with an ipWhitelist.



* * *

## Setup with Install Script

To run the script from the command line directly:

```bash

source <(curl -fsSL https://raw.githubusercontent.com/MarcusHoltz/Authentik-Traefik-MacVLAN/refs/heads/main/traefik_macvlan_bridge.sh)

```

