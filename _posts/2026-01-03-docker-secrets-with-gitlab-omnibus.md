---
layout: post
title: Setting up Docker Secrets with GitLab
description: Setting up GitLab Omnibus with Docker Secrets out of an .env file or in Gitlab.rb
date: 2026-01-03 11:33:00 -0700
categories: [DevOps, Monitoring]
tags: [docker, secrets, managment, config, gitlab, homelab, security, customization, logs, guide, self-hosted]
pin: false
image:
  path: /assets/img/header/header--docker--using-docker-secrets-with-gitlab.jpg
  alt: Docker Secrets can be exposed, dont let this happen to you!
---

# Homelab Guide to Setting up Docker Secrets with GitLab

If you're running GitLab in Docker, you've probably stored your root password in an `.env` file. This works, but anyone inside the container or with Docker access can see your credentials in plain text.

This guide was written to help you set up GitLab Omnibus with Docker Secrets stored **outside** of an `.env` file or in `Gitlab.rb`.


## Requirements

Be sure you have docker installed and the compose plugin. If not, I have included a step 0, install docker

This script is for a Fresh Debian LXC to setup a new user, docker, some additional software.

Please configure and run the included script.

```bash


# ==============================================================================
# SCRIPT: New User & System Bootstrap
# DESCRIPTION:
#   This script automates the setup of a new Debian/Ubuntu environment.
#
# USAGE:
#   1. Open the "CONFIGURATION" section below.
#   2. Edit the USERNAME, PASSWORD, and TIMEZONE variables as needed.
#   3. Run as root: sudo ./new-user-script.sh
# ==============================================================================

# ==============================================================================
# CONFIGURATION
# ==============================================================================

# --- Target User Details ---
# If this user exists, we will just install Docker and configure their shell.
# If this user does NOT exist, we will create them using the PASSWORD below.
USERNAME="john"
PASSWORD="myuserpassword"
GROUP="john"
SHELL="/bin/bash"
FULL_NAME="John Hamster"

# --- System Preferences ---
# Timezone used for the Tmux status bar clock
TIMEZONE="America/Denver"

# Default fallback for Debian version if detection fails (e.g., for MX Linux/Parrot)
# As of 2026, 'trixie' is Debian 13 (Stable).
DEFAULT_DEBIAN_CODENAME="trixie"

# --- User Environment Settings (.bashrc) ---
# Skin for Midnight Commander (mc)--
MC_SKIN="nicedark"

# Bash History Settings
HIST_SIZE="10000"
HIST_FILE_SIZE="20000"

# ==============================================================================
# END CONFIGURATION
# ==============================================================================

```


## Three Examples

This repository contains three folders, each demonstrating a different approach. We will be going through each one, in order. Every folder will require edits to run.


* * *

### Example 1). Basic .env Setup

**Folder:** `01-basic-env/`

This is the insecure method. Everything, including your password, lives in the `.env` file and gets loaded into the container environment. The `docker-compose.yml` contains all the GitLab configuration inline using `GITLAB_OMNIBUS_CONFIG`.


* * *

### Example 2). Docker Secrets

**Folder:** `02-docker-secrets/`

Your password moves to `./secrets/gitlab_root_password.txt`. Docker mounts it securely at `/run/secrets/` inside the container, where GitLab reads it using Ruby's `File.read()`. The password never appears in environment variables.

Yes, the password is still stored in plain text, but this is not a [secrets and privileged access management](https://github.com/hashicorp/vault) post. 


* * *

### Example 3). External Configuration File

**Folder:** `03-external-config/`

Extending Docker Secrets by moving all GitLab configuration out of `docker-compose.yml` into the `gitlab-config.rb` file. Your `.env` file expands to include all the tuning parameters, making everything configurable without touching Ruby or YAML files after initial setup.




* * *

## 1). Docker Compose Env Files

Head into the `01-basic-env` folder.

Here we have a `docker-compose.yml` file. It is super messy, we wont pay attention to it for now.

We'll take a look at the `.env` file.


* * *

The `.env` file holds all of the settings you can use inside of the `docker-compose.yml`

Docker loads the environment variables from the file into the docker compose project.

Look for the: 

- GITLAB_HOST_IP

- GITLAB_ROOT_PASSWORD


* * *

### Env File

```ini
# ============================================================================
# GITLAB ACCESS
# ============================================================================

# Hostname for GitLab - use your docker server's IP
GITLAB_HOST_IP=192.168.1.55

# Port to access GitLab web interface on your host machine
GITLAB_PORT=80

# SSH port for Git operations (git clone, push, pull via SSH)
GITLAB_SSH_PORT=2222

# ============================================================================
# GITLAB AUTHENTICATION
# ============================================================================

# Root admin password - CHANGE THIS to a strong password before deploying
GITLAB_ROOT_PASSWORD=HEYOUchangeThisPassword

# ============================================================================
# BACKUP CONFIGURATION
# ============================================================================

# Path on host machine where GitLab backups will be stored
BACKUP_PATH=/mnt/backups/borg/gitlab
```

Please change the `GITLAB_HOST_IP` to that of your Docker host you're running the compose file on, and and the `GITLAB_ROOT_PASSWORD` to that of your choice.


* * *

### Container Env Var from Docker

The `.env` file loads our `.env` file, all of it, into the container for use inside the container.

Let's take a look from the Docker perspective...



![Docker .env secrets exposed](/assets/img/posts/docker_secrets-1.png)



Whoopsie. We've exposed a lot of information most of it not important, except, wait, there's a password. Oh no!



## 2). Seperate Docker Secrets

**What changes**: Remove the password from .env and create a secrets file.


### Env file

As you can see we no longer have our gitlab root password in this file.

But you will still need to edit `GITLAB_HOST_IP` to match your Docker host you're running the compose file on.

```ini

# ============================================================================
# GITLAB ACCESS
# ============================================================================

# Hostname for GitLab - use your server's IP
GITLAB_HOST_IP=192.168.1.55

# Port to access GitLab web interface on your host machine
GITLAB_PORT=80

# SSH port for Git operations (git clone, push, pull via SSH)
GITLAB_SSH_PORT=2222

# ============================================================================
# BACKUP CONFIGURATION
# ============================================================================

# Path on host machine where GitLab backups will be stored
BACKUP_PATH=/mnt/backups/borg/gitlab


```

Where is our password? SHHHHHH. Secret.


* * *

### Secrets folder

A secrets folder allows us to set fine grain permissions ontop an area where we want to keep data safe, and then access only directly from Docker's secrets.

```
.
|-- .env
|-- docker-compose.yml
`-- secrets
    `-- gitlab_root_password.txt
```

And there's our secret. In the `./secrets` folder inside the `gitlab_root_password.txt` file.

```bash
$ cat ./secrets/gitlab_root_password.txt 
HEYOUchangeThisPassword
```

Amazing.


### Compose Changes to Support a Secret File

We were able to do this because we made some changes in the `docker-compose.yml` to support this new secrets file.


* * *

#### Step 1: Docker Secrets File


First step to get the secret into the container is to tell your docker compose project what file you want to use, and what to call this secret. We'll call our secret `gitlab_root_password`

```yaml
secrets:
  gitlab_root_password:
    file: ./secrets/gitlab_root_password.txt
```


* * *

#### Step 2: Put the secret inside the container

The second step is to mount the secret we named in our compose file inside our container. Docker does this by mounting it as a file inside container at: `/run/secrets/` (the location is automatically chosen by Docker)

```yaml
    secrets:
      - gitlab_root_password
```


* * *

#### Step 3: Put the secret into our config

Docker uses yaml for configuration, but GitLab uses Ruby.

So you need to be sure you're putting your secret in correctly.

We're just using a docker compose file, so we have to place the config in this file.

The new addition will read secrets from mounted files via File.read() into the Omnibus config, and then removing any extra lines:

```yaml
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = File.read('/run/secrets/gitlab_root_password').gsub("\n", "")
```


* * *

### Container Env Var from Docker with Secrets File

Now that we have a docker secret we dont see our secret inside the container anymore

Let's take a look...


![Docker secrets file loaded](/assets/img/posts/docker_secrets-2.png)



* * * 

## 3). Secrets in the gitlab.rb

Your docker-compose file can be cumbersome to maintain without adding other software's configuration to it.

We should put the `GITLAB_OMNIBUS_CONFIG` into it's own file.

Luckily there's a file made just for that, `gitlab.rb`

This will additionally allow us to put the rest of the values we saw in the docker-compose file into our .env file and never mess with the `gitlab.rb` after it's been initialy edited. 

But you will still need to edit `GITLAB_HOST_IP` to match your Docker host you're running the compose file on.


```ini
# ============================================================================
# GITLAB ACCESS
# ============================================================================
# Hostname for GitLab - use your server's IP
GITLAB_HOST_IP=192.168.1.55

# Port to access GitLab web interface on your host machine
GITLAB_PORT=80

# SSH port for Git operations (git clone, push, pull via SSH)
GITLAB_SSH_PORT=2222


# ============================================================================
# BACKUP CONFIGURATION
# ============================================================================
# Path on host machine where GitLab backups will be stored
BACKUP_PATH=/mnt/backups/borg/gitlab

```


### gitlab.rb

The new file is the gitlab.rb. We're using that to take the GitLab config out of docker-compose.

The config is packed full of comments to help explain what each part does.







### What does this look like now

Now that we have a docker secret we dont see our secret inside the container and we're not using docker compose to manipulate the docker secret, we're doing it with our gitlab.rb 

Let's take a look...





![Docker secrets loaded using our gitlab.rb](/assets/img/posts/docker_secrets-3.png)








Great job!!




## End


You make sure to keep a password out of your working enviornment. Well done.
























































