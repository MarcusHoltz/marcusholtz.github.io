---
layout: post
title: Immich Docker on UnRAID/: Installation /+ Config
date: 2025-06-30 11:33:00 -0700
categories: [Networking, UnRAID]
tags: [backup, networking, immich, storage, communication, security, photos, sharing, privacy, de-google, cloud, family, docker, unraid, newbie, guide]
pin: false
image:
  path: /assets/img/header/header--immich--self-hosted-photo-storage-setup.jpg
  alt: Setup Immich Secure Photo Storage and Sharing with Family
---

# Complete Immich Setup Guide for Docker on UnRAID with Post Installation Scripts

Here are all of the things I had to do to get Immich at a point where users, and myself, could start to enjoy using it.


* * *

## Hi, I'm new to Immich

Hello. Welcome. 

I presume you're here because you want out of a proprietary cloud-based photos/videos/memories system.

[Immich](https://www.reddit.com/r/immich/) is a good Google Photos or iCloud Photos alternative, and still allows sharing between multiple users. It may just be easy enough for your parents.

It is not just a good iCloud/Google Photos replacement, in many aspects, it does much more.


* * *

## Getting Immich Up and Running

What all does this tutorial cover?

- [Immich Setup Scripts](#immich-setup-scripts)
  - [First step: Install](#first-step-install)
    - [Immich Docker Compose Stack](#immich-docker-compose-stack)
  - [Second step: Immich Import](#second-step-immich-import)
    - [Importing Folders as Albums](#importing-folders-as-albums)
  - [Third step: Immich stack](#third-step-immich-stack)
    - [Duplicates with Immich stack](#duplicates-with-immich-stack)
  - [Fourth step: Immich compress](#fourth-step-immich-compress)
    - [1. Compressing Video Files Over a Specific Size](#1-compressing-video-files-over-a-specific-size)
    - [2. Compressing Image Files Over a Specific Size](#2-compressing-image-files-over-a-specific-size)
    - [3. Compressing CR2 Files down to JPEG](#3-compressing-cr2-files-down-to-jpeg)
  - [Fifth step: Connect Remotely with Netbird](#fifth-step-connect-remotely-with-netbird)
    - [Understanding Immich Network Architecture](#understanding-example-setup-network-architecture)
  - [🎉 Congratulations! 🎉](#-congratulations-)

Let's go!


* * *

## Immich: The Line in the Sand

How Immich works, you need to pick one of these two:

1. [Make changes outside of Immich, import your existing folders](https://immich.app/docs/guides/external-library/)

2. [Transfer your current folders full of photos to an Immich library](https://immich.app/docs/features/command-line-interface/)

This sounds like they're the same thing, right? NO!


* * *

### 1. Import your folders full of photos

The first option, `Make changes outside of Immich, import your existing folders`, does not move your files. They stay where they're at and appear in Immich. 

> This option is referred to as an External Library in Immich.


* * *

### 2. Transfer your photos to an Immich library

The second option, `Transfer to an Immich library`, is the one this tutorial is using. The whole point of me moving to Immich is to not have to rely on folders again, but be able to find things directly through Immich. My folders will still be there, but present as `Albums` inside of Immich. 

> You will need to import your photos to Immich. This is an extra process as you're sending data into Immich, not just pointing to an External Library.


* * *

### Why transfer files to an Immich library?

I am going to transfer photos to an Immich library, letting Immich handle all of the organization of my files. 

We won't ever need to create new folders to sort our media; all of it is available in Immich, just search it.

This is the start of the entire process. How will you store your photos?

**Why manage External Libraries for family and friends, when you could transfer those files from shared folders to a centralized location?**

I will be importing into the system Immich uses to organize files. The folder structure can be changed in the future and Immich can re-folder your files.

- `My meme folders` - not in Immich.

- `All of my documentation screen shots` - not in Immich.

- `All of my receipts, manuals, invoices, pdfs` - not in Immich.

- `Smokie's Birthday Photos` - in Immich.

Yes, you could send all that to Immich, and have it available to search -- but **this** is a *shared Immich instance* for family, not a general dumping ground.

Each user will have the same folder structure. Any organization is done by Immich in [albums](https://github.com/simulot/immich-go#from-folder-sub-command).


* * *

## First step: Install

I have most of my long term and speedy storage on UnRAID. 

Immich will reside on the UnRAID server, no point of having the service up if the files are unavailable.

[Immich](https://immich.app/docs/overview/welcome) has a very good [Unraid starter template](https://immich.app/docs/install/unraid) for us to use.

This write-up assumes you've already had a go at that. If not, give the [Unraid: Community Applications Template - Docker-Compose Method](https://immich.app/docs/install/unraid) official documentation a read.

I will be using that as a base for the files in this tutorial. You can find the files we will be using, the ones I use on my UnRAID server, in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/tree/main/unraid-immich-compose).

To setup - if you're not using UnRAID - the [docker-compose.yml](https://github.com/MarcusHoltz/immich-setup/blob/main/unraid-immich-compose/compose.manager/projects/immich/docker-compose.yml) and [env](https://github.com/MarcusHoltz/immich-setup/blob/main/unraid-immich-compose/compose.manager/projects/immich/env) files will work just fine.


### Use UnRAID for Immich

I have most of my long term and speedy storage on UnRAID. 

So, to me, it made sense to setup Immich on the UnRAID server, no point of having the service up if the files are unavailable.

You can find the files I have used on the UnRAID server in this [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/unraid-immich-compose).


* * *

### UnRAID Requirements: Part 1

To get this up and running, we need additonal software in UnRAID to support how this is set-up.

The easiest way is to install the [Docker Compose Manager](https://github.com/dcflachs/plugin-repository/blob/master/compose.manager.xml) from the UnRAID Community Applications.

You can find out more about [UnRAID's Unofficial Docker Compose Manager Plugin](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager).

- Once it's installed, you can find it under `Plugins`.

- Add New Compose Stack, and name it `immich`.

> This will create a new folder under:
`/boot/config/plugins/compose.manager/projects/immich/`


* * *

### UnRAID Immich Setup Downloader: Copy the Github Files

If you haven’t copied the [Immich Setup: Install Immich on UnRAID Compose Github](https://github.com/MarcusHoltz/immich-setup/tree/main/unraid-immich-compose) repo yet, you'll need to get a few files:

- `docker-compose.yml` - in the `compose.manager` folder. **REQUIRED** - The main configuration file that defines all services and their relationships. Deploy this file to launch your complete Immich infrastructure.

- `env` - in the `compose.manager` folder. **REQUIRED** - Contains your required environment variables. Configure your storage locations and database credentials here.

- `docker-compose.override.yml` - in the `compose.manager` folder. This is an optional file for making custom modifications without editing the main compose file, for icons and web addresses.

- `Immich-docker-compose` - in the `user.scripts` folder. Unraid script for automated start and management of the Immich stack.


* * *

#### Bash/ZSH Script to Download UnRAID Immich Setup

If you have not dowloaded anything yet, here is a `Bash` script to download all the required files for an UnRAID Immich Setup:

```bash

BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/unraid-immich-compose/" && for file in "user.scripts/scripts/Immich-docker-compose" "compose.manager/projects/immich/docker-compose.yml" "compose.manager/projects/immich/env" "compose.manager/projects/immich/docker-compose.override.yml"; do curl -O "$BASE_URL$file"; done

```


* * *

#### Powershell Script to Download UnRAID Immich Setup

On Windows, here is a `Powershell` script to download all the required files for an UnRAID Immich Setup:

```powershell

$BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/unraid-immich-compose/"; @("user.scripts/scripts/Immich-docker-compose","compose.manager/projects/immich/docker-compose.yml","compose.manager/projects/immich/env","compose.manager/projects/immich/docker-compose.override.yml") | ForEach-Object { Invoke-WebRequest -Uri "$BASE_URL$_" -OutFile ".\$(Split-Path $_ -Leaf)" }

```


* * *

### Immich Docker Compose Stack

This is what lies inside the docker compose stack I decided to use:

- `Immich Server` - Webserver handling requests

- `Immich Machine Learning` - The sweet juice Immich pours into my computer

- `Reddis` - In-memory database, for speedy lookups

- `Postgres` - For that good olde database feel

- `Immich Public Proxy` - Unused, but available

- `Prometheus` - The official tutorial for Immich included this, so I left it

- `Grafana` - If it aint Kabana, it's Grafana

- `Pgadmin4` - Edit the database before the kids come home

- `Immich Kiosk` - Turn your photos into a Screensaver

Now that you know what's included in the stack, let's get this Docker Compose file ready to go!



#### Compose File on UnRAID

A `docker-compose.yml` file contains all of the programs in the stack.

If you are using [UnRAID's Docker Compose Manager Community Application](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager):

- Make sure the stack_name is `immich`

- The `docker-compose.yml` file should be located at: `/boot/config/plugins/compose.manager/projects/immich/docker-compose.yml`

Additionally, you require one more file... 



#### Env File

This stack relies on environment variables. An environment variable file is typically named `.env` and must be placed in the same directory as the compose file.

This stack also relies on Environment Varriables to help set some of the configuration information, but the environment variable file is named `env`, without the `period`. This is how the plugin is written.

- Make sure the stack_name is `immich`

- Your `env` file needs to be in the same directory as the `docker-compose.yml` file.

- The `env` file should be located at: `/boot/config/plugins/compose.manager/projects/immich/env`


* * *

#### Docker-Compose Additional UnRAID Overrides

UnRAID special labels with Docker that help the web interface display addional information. These labels define elements like the WebUI URL, container icons, and descriptions that appear in the Unraid dashboard. By including these labels in a `docker-compose.override.yml` file, you can make Immich services integrate seamlessly with Unraid's management interface, accessible through the GUI.

If you are using [UnRAID's Docker Compose Manager Community Application](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager) this is a nice feature to have.

- Make sure the stack_name is `immich`

- Your `docker-compose.override.yml` file needs to be in the same directory as the `docker-compose.yml` file.

- The `docker-compose.override.yml` file should be located at: `/boot/config/plugins/compose.manager/projects/immich/docker-compose.override.yml`


* * *

### Auto-Start Immich On Boot

Immich will fail, as the network has not fully come up yet. YMMV.


* * *

#### UnRAID Requirements: Part 2

##### Install Userscripts on UnRAID

To fix the Immich stack startup we're using the [User Scripts](https://github.com/Squidly271/user.scripts/blob/master/plugins/user.scripts.plg) plugin.

You can find out more about [The Community Application: User Scripts](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/).

Make sure it is installed before continuing.


* * *

#### Using User Scripts for Immich Delay

I have this in my User Scripts and it runs at the start of the array.

My docker-compose stack name is `immich`. The rest should be copy and paste.

This script waits 100 seconds and then updates & restarts the docker-compose stack so it can see the network.

It then proceeds to do the same to NetBird. 

If you are using [UnRAID's User Scripts Community Application](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/) this is a nice feature to have.

- Make sure the directory you're in is `/boot/config/plugins/user.scripts/scripts/`

- Your `Immich-docker-compose` file needs to be in the `/boot/config/plugins/user.scripts/scripts/` directory.

- Alternativly, you can use the GUI: `Settings` > `User Scripts` > `Add New Script` > `Immich-docker-compose` > `click on cog next to name` > `Edit Script` > `Paste`

   ```bash
   #!/bin/bash
   cd /boot/config/plugins/compose.manager/projects/immich
   sleep 100
   docker compose down
   docker compose pull
   docker compose up -d
   sleep 60
   docker container restart NetBird-Client
   ```


* * *

### ASCII Flow of the Immich Docker stack


<details>

<summary>Visual Breakdown of the Docker Compose file</summary>

```text
# Immich Docker Stack Logic Flow

## User Interaction Layer

USER ACCESS POINTS:
┌─────────────────┬──────────────────┬─────────────────┐
│ Web Interface   │ Admin Tools      │ Display Mode    │
│ Port 2283       │ Port 8888        │ Port 3000       │
│ (Photo Mgmt)    │ (DB Admin)       │ (Kiosk)         │
└─────────────────┴──────────────────┴─────────────────┘


## Core Service Dependencies & Execution Flow

### 1. Foundation Services (Start First)

REDIS (172.21.8.113)
├── Function: In-memory caching for speed
├── Health Check: redis-cli ping
└── Status: Always restart

DATABASE (172.21.8.114) 
├── Image: tensorchord/pgvecto-rs:pg14
├── Function: PostgreSQL with vector extensions
├── Environment Variables:
│   ├── POSTGRES_PASSWORD: ${DB_PASSWORD}
│   ├── POSTGRES_USER: ${DB_USERNAME}
│   └── POSTGRES_DB: ${DB_DATABASE_NAME}
├── Volume Mount: ${DB_DATA_LOCATION}:/var/lib/postgresql/data
├── Health Check: pg_isready + checksum validation
└── Command: postgres with vectors.so preload


### 2. Core Processing Services (Start After Dependencies)

IMMICH-SERVER (172.21.8.111:2283)
├── Depends On: redis + database
├── Volume Mounts:
│   ├── ${UPLOAD_LOCATION}:/usr/src/app/upload
│   ├── ${IMPORT_LOCATION}:/usr/src/app/import
│   └── /etc/localtime:/etc/localtime:ro
├── Function: Main web server handling HTTP requests
└── Health Check: Enabled

IMMICH-MACHINE-LEARNING (172.21.8.110)
├── Volume Mount: model-cache:/cache
├── Function: AI processing for photo analysis
├── Network: Internal communication only
└── Health Check: Enabled


## Data Flow Logic

### Photo Upload Process

1. USER UPLOADS → Immich Server (Port 2283)
                       ↓
2. Server writes to → ${UPLOAD_LOCATION} volume
                       ↓
3. Metadata stored → PostgreSQL Database
                       ↓
4. Cache updates → Redis
                       ↓
5. ML analysis → Machine Learning Service


### Photo Import Process

1. Files exist in → ${IMPORT_LOCATION}
                       ↓
2. Server scans → Import directory
                       ↓
3. Processing → Same as upload flow (steps 2-5)


## Monitoring & Administration Stack

### Observability Services

PROMETHEUS (172.21.8.117:9090)
├── Config: ${PRO_DATA_LOCATION}/prometheus.yml
├── Function: Metrics collection
└── Volume: prometheus-data:/prometheus

GRAFANA (172.21.8.118:3000)
├── Function: Metrics visualization
├── Command: -disable-reporting
└── Volume: grafana-data:/var/lib/grafana


### Database Administration

PGADMIN (172.21.8.119:8888)
├── Environment:
│   ├── PGADMIN_DEFAULT_EMAIL: los_emails@gmail.com
│   └── PGADMIN_DEFAULT_PASSWORD: passwardos
├── Function: PostgreSQL web interface
└── Volume: pgadmin-data:/var/lib/pgadmin


## Extended Services

### Public Access

IMMICH-PUBLIC-PROXY (172.21.8.116:3033)
├── Points to: http://172.21.8.111:2283
└── Function: External access proxy


### Kiosk Display Mode

IMMICH-KIOSK (172.21.8.120:3000)
├── API Connection: KIOSK_IMMICH_URL: "http://172.21.8.111:2283"
├── Authentication: KIOSK_IMMICH_API_KEY (from user5@gmail.com)
├── Display Settings:
│   ├── Time: 12-hour format, MM/DD/YYYY
│   ├── Refresh: Every 25 seconds
│   ├── Theme: fade
│   └── Layout: single
├── Content Filtering:
│   ├── Person Filter: 3 specific person IDs
│   ├── Show archived: false
│   └── Album order: random
└── Image Display:
    ├── Fit: contain
    ├── Effect: smart-zoom (120%)
    └── Metadata: owner, album, person, time, date, description, exif, location


## Network Architecture

All services communicate via br0.8 network (172.21.8.0/24)

Service IP Allocation:
├── Machine Learning: .110
├── Immich Server: .111 (main entry point)
├── Redis: .113
├── Database: .114
├── Public Proxy: .116
├── Prometheus: .117
├── Grafana: .118
├── PgAdmin: .119
└── Kiosk: .120


## Configuration Dependencies

Required .env file variables:
├── UPLOAD_LOCATION (photo storage)
├── IMPORT_LOCATION (import source)
├── DB_DATA_LOCATION (database files)
├── PRO_DATA_LOCATION (prometheus config)
├── IMMICH_VERSION (container version)
├── DB_PASSWORD, DB_USERNAME, DB_DATABASE_NAME
└── IMMICH_TELEMETRY_INCLUDE



## Execution Logic Summary

**WHY**: Photo management with AI analysis, monitoring, and display capabilities
**HOW**: Containerized microservices with shared storage and network communication
**FLOW**: Database/Cache → Core Services → Extended Features → User Interfaces
```

</details>


* * *

## Second step: Immich Import

Now that you have Immich up and running, you need to load some content into Immich.

Without using External Libraries, you need to find a way to upload your images to Immich.

When importing your already saved folders into Immich you will need an API already generated and ready for each user.

This will be easy when using your cell phone, just login, the mobile app asks you what folders and BAM they're on the server.

> But what are we going to do with our old photo archive and Google Takeout photos?


* * *

### Importing Folders as Albums

Make sure you're using albums properly. 

This is how your phone will work too. Whatever folder your photos reside it, an album will be created.

So on a standard Android phone, you will have at-least one Album called `Camera`. The DCIM folder has the Camera folder, the standard folder for dumping your Camera apps photos into.

After that things like, `Downloads`, `Telegram Images`, `Telegram Video`, `Telegram Documents`, etc it's all up to you.

But make sure, if you're doing the desktop sync, that these folders make sense in a logical manner, and accompany the albums you're using on your mobile device.


* * *

### Immich-GO

Immich-GO is a single binary you can slap anywhere and then start importing.

It is soooooo easy. You could be on a remote server with a remote mount, no problem. Slap that binary in there, issue the command - done. No changes to the remote machine, nothing installed.

There is even a Python based GUI you could use: [Immich-Go GUI](https://github.com/shitan198u/immich-go-gui)


* * *

### All of the commands are on --dry-run

All of the commands you see below have the `--dry-run` flag active on them.

**There will be no changes made.**

You must remove the `--dry-run` part of the command to make changes. What you see on your screen will only be a representation of the changes that can be made.


* * *

#### Immich-GO (expects subfolders)

This will work in the current directory, assuming you have your folders there and the immich-go binary. But it will also "join folders" - what this means is if you have a bunch of subdirectories `/some/stuff/here/for/my/specific/purpose` - there are no sub-albumns in Immich. You need to flatten that structure. If you flatten it, what do you want it to look like? `some-stuff-here-for-my-specific-purpose`. And pictures in the `/some/stuff/here` directory will be in the album, `some-stuff-here`. The album-path-joiner flag at then end takes `/` and replaces them with `-` ... this is the same as `sed 's/\//-/g'`


```bash

MY_IMMICH_API_KEY=Xf9Lm2QzT8VwJpN0bYRsCk5HaHaHad7Ue3xWjF4gZt1Ao; \
./immich-go upload from-folder --dry-run \
  --pause-immich-jobs=FALSE \
  --api-key $MY_IMMICH_API_KEY \
  --server http://172.21.8.111:2283 \
  --recursive . \
  --folder-as-album FOLDER \
  --album-path-joiner "-"

```


* * *

#### Immich-GO (Google Takeout)

Here is another example, let's import our photos from a recent Google Takeout we downloaded, but oh no. 

We're on a remote Windows machine - download the binary!


```bash

.\immich-go.exe upload from-google-photos `
  --server http://172.21.8.111:2283 `
  --api-key Mh3ZyWf0RbJ7SnUoQXkT1eP6Cvg8LDAjVslI9Giggle2p `
  --dry-run `
  --sync-albums `
  --include-trashed `
  --include-unmatched `
  --pause-immich-jobs=FALSE `
  "O:\Testimg_import3\*.zip"

```


* * *

#### Immich-CLI (lets you name the album)

I understand if you wanted to take your time importing your files, and only wanted to use officially supported tools. 

Here is an example to import only the files in the current directory, and to give them a specific album name, in this case: `birthday2024`

```bash

MY_IMMICH_API_KEY=Tf2m9WzH8bpYndik7RVeAZr4DH3LwtMluLggk6TXe; \
IMMICH_ALBUM_NAME=birthday2024; \
docker run -it -v "$(pwd)":/import:ro \
  -e IMMICH_INSTANCE_URL=http://172.21.8.111:2283 \
  -e IMMICH_API_KEY=$MY_IMMICH_API_KEY \
  ghcr.io/immich-app/immich-cli:latest \
  upload --dry-run --album-name $IMMICH_ALBUM_NAME \
  -c 5 --recursive .

```


* * *

#### Immich-CLI (Google Takeout)

You can even use it to import your Google Takeout photos, see example below:

```bash

docker run --rm -it \
  --name immich_cli \
  --network br0.8 \
  --ip 172.21.8.112 \
  -v "/mnt/user/Mobile Backups/Testimg_import3/:/import:ro" \
  -e IMMICH_INSTANCE_URL=http://172.21.8.111:2283 \
  -e IMMICH_API_KEY=Kv6e4NpD1qtLsgow3PKmSYm7ZT8JxpLurXxxd2RCf \
  ghcr.io/immich-app/immich-cli:latest \
  upload from-google-photos \
    -dry-run \
    -include-trashed \
    -include-unmatched \
    -create-albums \
    -c 2 \
    /import/takeout-20250614TZ-001.zip \
    /import/takeout-20250614TZ-1-001.zip

```


* * *

## Third step: Immich stack

[Immich-stack](https://github.com/majorfi/immich-stack) is designed to [automatically group similar photos](https://majorfi.github.io/immich-stack/getting-started/quick-start/) into [stacks](https://immich.app/docs/api/create-stack/) within the Immich photo management system. Its primary purpose is to help users organize large photo libraries by stacking related images—such as burst shots, [similar filenames](https://majorfi.github.io/immich-stack/api-reference/environment-variables/?h=criteria#custom-criteria_1), or images taken in quick succession—into logical groups for easier browsing and management



### Duplicates with Immich stack

Stacking your images is basically how you "keep" your duplicates together. Even if they are smaller size, less image resolution, have poor color density, but hey -- gotta keep 'um all.

Immich has a feature to help you inspect every duplicate and decide to trash it, stack it, or just plain keep it. It's pretty good at deciding to trash the older, or smaller in size photo.


* * *

#### Immich-Stack Introduction

You run immich-stack as a command-line tool or via Docker. It connects to your Immich server using an API key and processes your photo library according to the criteria you specify.

The real beauty of immich-stack is it lets you specify how your files have been named, then stack them accordingingly. Let me give you an example:


* * *

### Example Immich-Stack Command

Here is an example of an immich-stack command, please note the `--critera` section, explained below:

```bash

immich-stack \
--criteria '[{"key":"originalFileName","split":{"delimiters":["+", "."],"index":0}}]' \
--parent-filename-promote ",+" \
--dry-run \
--api-key Tu4p1OgS6tUrl3scA4r3m31OIaGVrwrLvrGtl75PCg \
--api-url http://172.21.8.111:2283

```



#### Immich-Stack Command Explained

Sorry, let me explain this a little. Their [wiki](https://majorfi.github.io/immich-stack/) isnt too easy to go by.


- `--criteria:` Specifies how to group photos. In this example, it splits the `originalFileName` on `+` and `.` and uses the first segment, (`+`), as the grouping key. This is useful for stacking images that share a common base filename (e.g., burst shots like `IMG_1234+1.JPG`, `IMG_1234+2.JPG`).

- `--parent-filename-promote:` Controls which photo in a stack is promoted as the parent. The value `",+"`  will give a preference for filenames without a `+` sign.

- `--dry-run:` Performs a simulation without making changes, so you can review what would happen.

- `--api-key:` Your Immich API key for authentication.

- `--api-url:` The URL to your Immich server's API endpoint.


* * *

### Immich-GO Stacking

Immich-Stack is best with defined delimiters and parent promotion. If you're not using it for that, then there's no real reason to use immich-stack.

You're just letting it go find duplicates or similar photos and stack them, you can use Immich-GO.

Stack photos using immich-go is done with the `stack` command, which connects to your Immich server and groups related photos together based on the options for stacking below:



#### Options for Immich-GO Stacking

- `--server` (or `-s`): Your Immich server address (required).

- `--api-key` (or `-k`): Your Immich API key (required).

- `--dry-run`: Simulate actions without making changes.

- `--manage-burst`: Manage burst photos (options: NoStack, Stack, StackKeepRaw, StackKeepJPEG).

- `--manage-raw-jpeg`: Manage RAW+JPEG pairs (options: NoStack, KeepRaw, KeepJPG, StackCoverRaw, StackCoverJPG).

- `--manage-heic-jpeg`: Manage HEIC+JPEG pairs (options: NoStack, KeepHeic, KeepJPG, StackCoverHeic, StackCoverJPG).

- `--manage-epson-fastfoto`: Stack Epson FastFoto scans with the corrected scan as the cover.


* * *

##### Immich-GO Stacking Example: Stack RAW+JPEG pairs with the RAW file as the cover

```bash

MY_IMMICH_API_KEY=Ab3Tg5kPzQ9CwL8WbYdRsH3VvDf7GxF1Zh6JmK9A3t4L; \
immich-go stack --server=http://172.21.8.111:2283 \
  --api-key=$MY_IMMICH_API_KEY \
  --dry-run \
  --manage-raw-jpeg=StackCoverRaw

```


* * *

##### Immich-GO Stacking Example: Stack burst photos**

```bash

MY_IMMICH_API_KEY=Jq5KnX2Bf7WsRp9VtHgYzDdQm8ZtUv0Cv4JsLwP3a1BoK; \
immich-go stack --server=http://172.21.8.111:2283 \
  --api-key=$MY_IMMICH_API_KEY \
  --dry-run \
  --manage-burst=Stack

```


* * *

## Fourth step: Immich compress

Having everyone in the family jump on your Immich server as their primary means of backup (Google Photos alternative). Then you may have a server filling up very fast.

Let's address three "hypothetical" situations:

1. Grandma has several a 5GB video of her favorite train rides. 

2. Uncle uses 50MB jpegs for edits of his best photos.

3. Cousin store photos with his RAW/CR2 files present (psst - tell them there's a [Lightroom Immich plugin](https://blog.fokuspunk.de/lrc-immich-plugin/)).

Yeah. **Lossy compression may be deemed acceptable.** 

Original files will always be backed up and stored off site. But if you really need compression with no loss in image data, try [immich-upload-optimizer](https://github.com/miguelangel-nubla/immich-upload-optimizer).


* * *

### 1. Compressing Video Files Over a Specific Size

Grandma and her train rides... If you have several people uploading large video files that are tapping your storage space as outliers, compress them.

I have a script that will:

- Compress videos above a certain size.

- Backup the original video.

That should take care of any concerns about people gobbling up space with a few large video files. 

The original video can be stored off site, or on a slower storage medium. It doesnt really need to be on the Immich server, heck, it doesnt really need it exist at all. Delete it. It's up to you.

You can find the `inplace_mp4_optimizer.sh` script in the [compress2largeVIDEOS](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeVIDEOS) folder in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/).

That information can also be found below:


* * *

#### In-Place MP4 Optimizer Introducton

- **This script will:** Scans a directory of MP4 videos, recompresses each file that meets certain condition, keep all metadata, and back up originals.

- **To use this script:** Just run this in your media folder with Docker.

- **What happens:** Replaces the original .mp4 with the optimized version. The original file is saved in a backup folder, and details of each processed file are logged. 

- **What to configure:** `CRF`, `PRESET` and `SIZE_THRESHOLD_KB` - Constant Compression Factor, Preset the speed of compression, and min size of files to compress.


* * *

#### What exactly does In-Place MP4 Optimizer Do

- **Finds all mp4/MP4 files in your folder tree above a certain size (default: 5250KB)**

- **Optimizes the files in-place, meaning, same filename and same location**

- **Preserves ALL metadata (EXIF, GPS, etc) and keeps the original modified date**

- **Backs up the original file (with folder structure) to an `originals/` directory**

- **Shows progress as it works**

- **Skips files it already processed if interrupted (resumes cleanly)**


* * *

#### In-Place MP4 Optimizer Quick Instructions

1. Have [Docker](https://docs.docker.com/get-docker/) Ready

2. Put your media in a folder, e.g. `./media`

3. Copy the provided `docker-compose.yml`, `Dockerfile`, and `inplace_mp4_optimizer.sh` into that folder

4. Edit `docker-compose.yml` to change the size threshold or quality:

- `PROCESS_DIR`: Folder to process (default: `./`)

- `CRF`: Output MP4 quality (default: 21, range: 1-30)

   ```
   MP4 Quality ---
   - CRF 18-20: Very high quality – Almost indistinguishable from the original video. File size is noticeably larger than 21.
   - CRF 21: Default "good quality" – Best balance between quality and file size. This is the default for many tools like FFmpeg.
   - CRF=21       # Default CRF video compression value is 21
   ```

- `PRESET` – The FFmpeg preset for speed vs. compression (default `"slow"`).  Common values include `medium`, `slow`, `slowest`.

- `SUFFIX` – Optional text to append to backup filenames, to avoid confusion if using a flat filesystem.

- `SIZE_THRESHOLD_KB`:  Minimum file size (in kilobytes) to consider for optimization (default `5250` KB).  Files below this size are skipped entirely.

5. Run the container and have it remove itself when complete:

   ```bash

   docker compose run --rm mp4-optimizer

   ```

6. Done!

   - Optimized JPGs are now in place

   - Originals are in `originals/`  

   - Progress is shown as it works


* * *

#### In-Place MP4 Optimizer Downloader: Copy the Github Files

If you haven’t copied the [Immich Setup: compress2largeVIDEOS](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeVIDEOS) folder yet, you'll need to get a few files:

- `docker-compose.yml` - The main configuration file that defines all services and their relationships. Deploy this file to launch your video compression journey.

- `inplace_mp4_optimizer.sh` - Contains the logic that creates optimized and preserved MP4 files.

- `Dockerfile` - This will put an image together of all the tooling we need to complete the `inplace_mp4_optimizer.sh` script.


* * *

##### Bash/ZSH Script to Download In-Place MP4 Optimizer

If you have not dowloaded anything yet, here is a `Bash` script to download all the required files for the In-Place MP4 Optimizer script, ran in Docker:

```bash

BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/compress2largeVIDEOS/" && for file in "docker-compose.yml" "inplace_mp4_optimizer.sh" "Dockerfile"; do curl -O "$BASE_URL$file"; done

```


* * *

##### Powershell Script to Download In-Place MP4 Optimizer

Again, on Windows, if you have not dowloaded anything yet here is a `Powershell` script to download all the required files for the In-Place MP4 Optimizer script, ran in Docker:

```powershell

$BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/compress2largeVIDEOS/"; @("docker-compose.yml","inplace_mp4_optimizer.sh","Dockerfile") | ForEach-Object { Invoke-WebRequest -Uri "$BASE_URL$_" -OutFile ".\$(Split-Path $_ -Leaf)" }

```


* * *

#### In-Place MP4 Optimizer FAQ

**Q: Will this overwrite my MP4 files?**  
A: Yes, but it moves the original to `originals/` first, preserving the folder structure.

**Q: What if the process is interrupted?**  
A: It keeps a log and will skip already optimized files on the next run.

**Q: Will it change the quality of my videos?**  
A: The script uses CRF 21 (high quality) by default. You can adjust this with the `CRF` environment variable (lower = higher quality).

**Q: What metadata is preserved?**  
A: All metadata is preserved in the optimized file using exiftool.

**Q: Will it change the timestamp on my files?**  
A: Only optimized files get their modified date set to the original's modified date. Files not optimized are untouched.

**Q: How do I change the size threshold or quality settings?**  
A: Set environment variables: `SIZE_THRESHOLD_KB` for minimum file size, `CRF` for quality (18-28 range), and `PRESET` for encoding speed.

**Q: What video formats are supported?**  
A: Only MP4 files are processed. The script looks for `.mp4` and `.MP4` extensions.

**Q: Can I customize the backup file naming?**  
A: Yes, set the `SUFFIX` environment variable to add a custom suffix, otherwise it uses timestamps.

**Q: How much space will I save?**  
A: Typically 30-50% file size reduction with minimal quality loss, depending on the original encoding.

**Q: What happens to audio tracks?**  
A: Audio is copied without re-encoding to preserve quality and processing speed.


* * *

#### In-Place MP4 Optimizer Example Output

Once you run the script, it will display all of the set variables and then begin to process files. This will look something like the Example Output below:

<details>

<summary>Example Output of the inplace_mp4_optimizer.sh</summary>


```bash
$ ./inplace_mp4_optimizer.sh

In-Place MP4 Optimizer (with Originals Backup, Progress, mtime Copy, and Temp Log)
==================================================================================
Processing directory: /workdir
JPEG Quality: 85
Optimization flags: -optimize -progressive
Size threshold: 5250KB (5376000 bytes)
Originals backup directory: /workdir/originals
Files converted log file: /workdir/.mp4_files_optimized--keepme.log
Errors and unconverted file: /workdir/.mp4_files_optimized--errors.log

Loaded temp log with 3 previously processed files

Scanning for MP4 files...
Total MP4 files found: 15
Files below 5250KB (untouched): 7
Files already processed: 3
Files to optimize: 5

Suffix for originals: (timestamp-based)
Only files larger than 5250KB will be optimized.
Optimized files will have their mtimes set to their original modified date.
Originals will be moved to: /workdir/originals

WARNING: This will modify your original MP4 files in-place!
Make sure you have backups if needed.

Starting optimization of 5 files...

[1/5] (20%)
Optimizing: /workdir/vacation_video.mp4
   Time for vacation_video.mp4 set to 202312151430.25
✓ Optimized: vacation_video.mp4 (15240KB → 8950KB, saved 41%)
   Logged to processing history

[2/5] (40%)
Optimizing: /workdir/conference_recording.mp4
   Time for conference_recording.mp4 set to 202401081245.15
✓ Optimized: conference_recording.mp4 (22100KB → 14800KB, saved 33%)
   Logged to processing history

[3/5] (60%)
Optimizing: /workdir/family_reunion.mp4
   Time for family_reunion.mp4 set to 202312201600.45
✓ Optimized: family_reunion.mp4 (18750KB → 11200KB, saved 40%)
   Logged to processing history

[4/5] (80%)
Optimizing: /workdir/presentation_demo.mp4
   Time for presentation_demo.mp4 set to 202401151030.12
✓ Optimized: presentation_demo.mp4 (9860KB → 6420KB, saved 35%)
   Logged to processing history

[5/5] (100%)
Optimizing: /workdir/sports_highlights.mp4
   Time for sports_highlights.mp4 set to 202312281900.33
✓ Optimized: sports_highlights.mp4 (12300KB → 7850KB, saved 36%)
   Logged to processing history

Processing complete!
- Optimized: 5 files
- Failed: 0 files
- Total in log: 8 files

All done! MP4 files larger than 5250KB have been optimized in-place.
Optimized files have had their mtimes set to their original modified date.
Originals have been moved to: /workdir/originals
Converted files logged at: /workdir/.mp4_files_optimized--keepme.log
```

</details>


* * *

#### In-Place MP4 Optimizer Troubleshooting

- **Not enough space:** Make sure you have room for the `originals/` backup folder, which will contain copies of all original files.

- **Permissions:** Run as a user with read/write access to your photo folder.

- **"Error compressing" messages:** Check that your MP4 files aren't corrupted. Try playing them in a video player first.

- **"Error copying metadata" messages:** This usually means ExifTool failed. Check that ExifTool is properly installed and the file isn't corrupted.

- **Files not being processed:** Check that files are above the size threshold (default 5250KB) and not already in the log file.

- **Slow processing:** The default preset is "slow" for better compression. Set `PRESET=medium` or `PRESET=fast` for faster processing.

- **Quality too low:** Decrease the CRF value (e.g., `CRF=18` for higher quality) or increase it (e.g., `CRF=28`) for smaller files.

- **Script stops with "Directory does not exist":** Make sure the `PROCESS_DIR` path is correct and accessible.

- **Docker not installed:** To fix, [Install Docker](https://docs.docker.com/get-docker/) or [Docker Destkop](https://www.docker.com/products/docker-desktop/).


* * *

#### Details About the In-Place MP4 Optimizer

I have included a details section if you wanted a deeper dive into how this script was made, and it's function.


* * *

##### Tools Used

Here are the tools and utilities the In-Place MP4 Optimizer uses:


###### Core Video Processing

- **FFmpeg** - The main tool for MP4 recompression using H.264 codec with configurable CRF and preset values

- **ExifTool** - Used to copy all metadata from the original file to the optimized version


###### System Utilities

- **find** - Locates MP4 files recursively in the directory

- **stat** - Gets file sizes and modification times (with fallback for different systems)

- **md5sum/md5** - Generates file hashes for tracking processed files

- **date** - Handles timestamps for logging and file modification times

- **touch** - Sets file modification times back to original values

- **mv/cp** - File operations for backups and temporary files

- **mkdir** - Creates backup directory structure

- **wc** - Counts lines in log files


###### Shell Features

- **Bash** - The script is written for Bash shell

- Various bash built-ins like `read`, `while`, `if`, `echo`, etc.


* * *

##### Step-by-Step Inside the Script

Here’s what happens step by step:

- We capture the original file’s mtime (`get_file_mtime`) so we can restore it later.

- A **temporary backup** `${mp4_file}.tempbackup` is made by copying the original.  This serves two purposes: it preserves the original content, and it provides the source for metadata (since `ffmpeg` might strip some metadata when re-encoding).

- The script then runs `ffmpeg` quietly (`-loglevel quiet`) to re-encode the video using `libx264` with the specified `CRF` and `PRESET`.  Audio is copied as-is (`-c:a copy`).  The `-map_metadata 0` flag attempts to carry over metadata as well.

- If `ffmpeg` succeeds, `exiftool` is run to copy **all** metadata tags from the backup file into the new `.tmp.mp4`.  This ensures things like camera info, subtitles, and other tags are preserved.

- Next, `backup_original "$mp4_file"` moves the original file into the `originals/` directory.  (See **Backups and Naming** below.)

- The optimized file `.tmp.mp4` is then renamed to the original filename, effectively replacing it.

- We use `set_file_mtime` to restore the original modification timestamp on the new file, so it looks unchanged in terms of date.

- Finally, the script records details in the log.  It computes a hash of the new file, notes the original and new sizes, and appends a line to the temp log file via `add_to_temp_log`.  Then it removes the temporary backup copy.


* * *

##### Configuration (Environment Variables)

The script’s behavior can be customized via environment variables.  Each has a default value (shown here with code comments), and you can override them by exporting before running the script or by prefixing the command. For example, to change the directory or CRF value. The relevant lines from the script are:

```bash
PROCESS_DIR="${PROCESS_DIR:-/workdir}"
JPEG_QUALITY="${JPEG_QUALITY:-85}"
SIZE_THRESHOLD_KB="${SIZE_THRESHOLD_KB:-5250}"
SIZE_THRESHOLD=$((SIZE_THRESHOLD_KB * 1024))
ORIGINALS_DIR="$PROCESS_DIR/originals"
TEMP_LOG_FILE="$PROCESS_DIR/.mp4_files_optimized--keepme.log"
ERROR_LOG_FILE="$PROCESS_DIR/.mp4_files_optimized--errors.log"
SUFFIX="${SUFFIX:-}"
CRF="${CRF:-21}"           # Default CRF value is 21
PRESET="${PRESET:-slow}"    # Default preset value is "slow"
```

* * *

- `PROCESS_DIR` – The root directory to scan for MP4 files (default `/workdir`).

- `SIZE_THRESHOLD_KB` – Minimum file size (in kilobytes) to consider for optimization (default 5250 KB).  Files below this size are skipped entirely.

- `CRF` – The FFmpeg Constant Rate Factor for compression (default 21; higher = more compression but lower quality).

- `PRESET` – The FFmpeg preset for speed vs. compression (default `"slow"`).  Common values include `fast`, `medium`, `slow`.

- `SUFFIX` – Optional text to append to backup filenames.  If unset, the script will use a timestamp in the filename instead.

The other variables are derived from these (e.g. `SIZE_THRESHOLD` in bytes, `ORIGINALS_DIR` for backups, log file paths).


* * *

##### How Files Are Selected

When you run the script, it first **scans** the target directory for MP4 files and decides which ones to optimize.  This involves two checks:

1. **Size threshold:** The file must be larger than the `SIZE_THRESHOLD_KB`.  This avoids wasting time on tiny files.

2. **Not already optimized:** The script keeps a log (`.mp4_files_optimized--keepme.log`) of files it has processed (including their hash and size).  If a file’s path, hash, and size all match an entry in that log, it is considered “already processed” and will be skipped.


* * *

A function called `count_mp4_files()` walks through the directory (using `find`) and prints a summary.  For example:

```bash
count_mp4_files() {
    local total_found=0 below_threshold=0 already_processed=0 count=0
    while IFS= read -r -d '' file; do
        if [[ "$file" == *"/originals/"* ]]; then continue; fi
        ((total_found++))
        if file_above_threshold "$file"; then
            if is_file_processed "$file"; then
                ((already_processed++))
            else
                ((count++))
            fi
        else
            ((below_threshold++))
        fi
    done < <(find "$PROCESS_DIR" -type f \( -iname "*.mp4" -o -iname "*.MP4" \) -print0)
    echo "Total MP4 files found: $total_found"
    echo "Files below ${SIZE_THRESHOLD_KB}KB (untouched): $below_threshold"
    echo "Files already processed: $already_processed"
    echo "Files to optimize: $count"
}
```

> Here `is_file_processed()` reads the log file and checks for matching entries, so any file already optimized (and unchanged) will be skipped.  This log-based check is a safety mechanism to prevent redoing work.


* * *

##### The Optimization Pipeline

Once the script has the list of files to optimize, it processes them one by one. The high-level flow for **each** file is:

1. **Record original info:** Save the file’s original modification time (`get_file_mtime`), and create a temporary backup copy (`.tempbackup`) of the file.

2. **Recompress:** Run `ffmpeg` to compress the video into a new temporary file (`.tmp.mp4`), using H.264 (`libx264`), the chosen CRF, and preset. Audio streams are copied.

3. **Copy metadata:** Run `exiftool` on the new file to copy all metadata (tags) from the original backup.

4. **Backup original:** Move the original file into an `originals/` subfolder, optionally appending the given `SUFFIX` or a timestamp to its name.  This ensures the original is preserved.

5. **Replace and restore timestamp:** Move the optimized temp file into the original file’s place, and set its modification time back to the original timestamp.

6. **Log results:** Compute a hash and sizes of the new file, and write a line to the log file. Print a summary of space savings.


* * *

Below are excerpts from the core function `optimize_mp4_file()` in the script:

```bash
optimize_mp4_file() {
    local original_mtime=$(get_file_mtime "$mp4_file")
    echo "Optimizing: $mp4_file"
    # Copy original to a temp backup (for metadata and fallback)
    cp "$mp4_file" "${mp4_file}.tempbackup"
    local original_size=$(get_file_size_bytes "${mp4_file}.tempbackup")

    # Recompress with ffmpeg
    if ffmpeg -i "$mp4_file" -map_metadata 0 -c:v libx264 -crf "$CRF" -preset "$PRESET" \
              -c:a copy "${mp4_file}.tmp.mp4"; then
        # Copy metadata from original to recompressed file
        if exiftool -TagsFromFile "${mp4_file}.tempbackup" -all:all "${mp4_file}.tmp.mp4" -overwrite_original; then
            local new_size=$(get_file_size_bytes "${mp4_file}.tmp.mp4")

            # Move original to backup location
            backup_original "$mp4_file"

            # Replace original with optimized file
            mv "${mp4_file}.tmp.mp4" "$mp4_file"
            set_file_mtime "$mp4_file" "$original_mtime"

            # Log the operation
            local compressed_hash=$(get_file_hash "$mp4_file")
            add_to_temp_log "$mp4_file" "$compressed_hash" "$original_size" "$new_size"
            rm "${mp4_file}.tempbackup"
            ...
```

After a successful optimization, the script prints a message with the saved space.

In case of an error (either `ffmpeg` fails or metadata copying fails), the script logs an error message to `$ERROR_LOG_FILE` and cleans up the temp files, then continues to the next file.


* * *

##### Backups, Logging, and Timestamp Restoration

Let me break down the next bit into three different sections.


* * *

###### Backups

The original (uncompressed) files are moved into an `originals/` directory under `PROCESS_DIR`, preserving the subdirectory structure.  The naming of the backup file depends on the `SUFFIX` variable:

* If `SUFFIX` is set (for example `"_orig"`), the backup is named `basename_SUFFIX.mp4`.
* If `SUFFIX` is empty, the script appends a Unix timestamp: e.g. `basename_1638316800.mp4`.

The code handling this is:

```bash
if [[ -n "$SUFFIX" ]]; then
    backup_name="${base}${SUFFIX}.$ext"
else
    timestamp=$(date +%s)
    backup_name="${base}_${timestamp}.$ext"
fi
```

This ensures unique backup names (incrementing a counter if needed).

For example, if you have `video.mp4`, a backup might be `video_orig.mp4` or `video_1638316800.mp4` in `originals/`.


* * *

###### Logging

The script maintains a tab-delimited log file `.mp4_files_optimized--keepme.log` inside `PROCESS_DIR`.  Each entry records the file path, a hash, sizes, and timestamp.  This log serves two purposes:

* It provides a history of what was done (which can help with auditing or undoing changes if needed).
* The script uses it to **skip** files already optimized.  The function `is_file_processed()` reads this log and checks if a file’s path, hash, and size match a logged entry.  If so, the file is left untouched.

For example, `is_file_processed()` works roughly like this:

```bash
is_file_processed() {
    local current_hash=$(get_file_hash "$file")
    local current_size=$(get_file_size_bytes "$file")
    while IFS='|' read -r logged_file logged_hash logged_size ...; do
        if [[ "$logged_file" == "$file" && "$logged_hash" == "$current_hash" && "$logged_size" == "$current_size" ]]; then
            return 0  # Already processed
        fi
    done < "$TEMP_LOG_FILE"
    return 1
}
```


If a file matches, `is_file_processed` returns success and the script skips that file.  This prevents wasting time recompressing the same file twice (assuming it hasn’t changed).


* * *

###### Timestamp restoration

Because compressing a file would normally update its modification time, the script captures the original `mtime` before processing and then applies it back to the new file with `touch -t`.  This makes the optimized file appear to have the same date as before.


* * *

##### Safety Mechanisms and Warnings

Several safeguards are in place:

- **Size threshold:** By default, very small MP4 files (under \~5 MB) are skipped.  You can adjust `SIZE_THRESHOLD_KB` to your needs.

- **Log-based skip:** Already-processed files are detected via the log, so re-running the script won’t touch them again unless their contents change.

- **Backups:** Originals are not deleted outright; they are moved to `originals/` for safekeeping.

- **Dry-run notice:** The script prints a clear warning before proceeding:

  > `WARNING: This will modify your original MP4 files in-place! Make sure you have backups if needed.`

- **Error logging:** Any errors in compression or metadata copying are written to `.mp4_files_optimized--errors.log` in the processing directory.

Together, these ensure you don’t lose data inadvertently and can undo or re-run safely if needed.


* * *

##### Running the Script by Itself (No Docker - Examples)

Assuming the script file is named `inplace_mp4_optimizer.sh`, make it executable and run it like this:

```bash
chmod +x inplace_mp4_optimizer.sh
PROCESS_DIR="/path/to/videos" ./inplace_mp4_optimizer.sh
```

By default, it will process all `*.mp4` files in `/path/to/videos` larger than 5250 KB.  If you want to adjust settings, you can set environment variables on the command line. For example:

- **Use a different CRF and preset:** `CRF=23 PRESET=fast ./inplace_mp4_optimizer.sh`

- **Lower the size threshold to 1000 KB (1 MB):**
  `SIZE_THRESHOLD_KB=1000 ./inplace_mp4_optimizer.sh`

- **Add a suffix to backups:** `SUFFIX="_old" ./inplace_mp4_optimizer.sh`

You can also combine them, e.g.:

```bash
PROCESS_DIR="/videos" CRF=25 PRESET=medium SIZE_THRESHOLD_KB=2000 ./inplace_mp4_optimizer.sh
```

When you run it, the script will echo its configuration (the values of `PROCESS_DIR`, `CRF`, etc.) and then show progress messages for each file, like `[1/10] (10%) Optimizing: example.mp4`, followed by a checkmark and savings on success. At the end, it summarizes how many files were optimized, how many failed, and how many are listed in the log.

Check out the MP4 optimization pipeline diagram below for a visual overview.


* * *

##### User and Machine Flow of the In-Place MP4 Optimizer script

<details>

<summary>Visual Script Breakdown</summary>


```
MP4 OPTIMIZER SCRIPT - LOGIC FLOW DIAGRAM
=========================================

USER PERSPECTIVE                    |  MACHINE LOGIC
------------------------------------|----------------------------------
                                    |
User runs script                    |  Parse environment variables:
   ./inplace_mp4_optimizer.sh       |  - PROCESS_DIR (default: /workdir)
                                    |  - SIZE_THRESHOLD_KB (default: 5250)
                                    |  - CRF (default: 21)
                                    |  - PRESET (default: slow)
                                    |  - SUFFIX (optional)
                                    |
User sees initialization info:      |  Validate PROCESS_DIR exists
- Processing directory              |     └─ If not: EXIT with error
- CRF Quality setting              |
- Encoder preset                   |  Initialize log files:
- Size threshold                   |  - Create .mp4_files_optimized--keepme.log
- Backup directory location        |  - Create .mp4_files_optimized--errors.log
                                    |
                                    |  Load existing temp log (if exists)
                                    |     └─ Count previously processed files
                                    |
User sees "Scanning for MP4s..."   |  Scan directory recursively:
                                    |     find PROCESS_DIR -type f *.mp4/MP4
                                    |
User sees file count summary:       |  For each MP4 file found:
- Total MP4 files found            |  ┌─ Skip if in /originals/ directory
- Files below threshold            |  ├─ Check file size vs threshold
- Files already processed          |  ├─ Calculate MD5 hash
- Files to optimize                |  ├─ Check if already in temp log
                                    |  └─ Categorize file status
                                    |
User sees warning about            |  Build optimization queue:
in-place modification              |     └─ Only files > threshold + not processed
                                    |
                                    |  IF no files to optimize:
                                    |     └─ Display "No files to optimize"
                                    |        and EXIT
                                    |
User sees optimization progress:    |  FOR EACH file in optimization queue:
[1/5] (20%)                        |
"Optimizing: filename.mp4"         |  ┌─ Get original file mtime
                                    |  ├─ Create temp backup copy
                                    |  ├─ Get original file size
                                    |  │
                                    |  ├─ Run ffmpeg compression:
                                    |  │   ffmpeg -i input -c:v libx264 
                                    |  │   -crf CRF -preset PRESET output.tmp
                                    |  │
                                    |  ├─ IF ffmpeg succeeds:
                                    |  │   ├─ Copy metadata with exiftool
                                    |  │   ├─ Move original to backup location
                                    |  │   ├─ Move compressed file to original location
                                    |  │   ├─ Restore original mtime
                                    |  │   ├─ Calculate new hash
                                    |  │   ├─ Log to temp file
                                    |  │   └─ Clean up temp backup
                                    |  │
                                    |  └─ IF ffmpeg fails:
                                    |      ├─ Log error to error file
                                    |      └─ Clean up temp files
                                    |
User sees success/failure messages: |
"✓ Optimized: file.mp4             |  Calculate and display:
(5120KB → 3840KB, saved 25%)"      |  - Original size vs new size
"Time for file.mp4 set to..."      |  - Percentage saved
"Logged to processing history"      |  - Confirm mtime restoration
                                    |  - Confirm logging
                                    |
OR                                  |
"✗ Error compressing: file.mp4"     |  Write error to error log file
                                    |
User sees final summary:            |  Count final results:
"Processing complete!"              |  - Total files processed
"- Optimized: X files"              |  - Total failures
"- Failed: Y files"                 |  - Total entries in log
"- Total in log: Z files"           |
                                    |
User sees completion message:       |  Display final status and locations:
"All done! MP4 files larger than   |  - Threshold reminder
5250KB have been optimized..."      |  - Backup directory location
"Originals moved to: /path/originals"|  - Log file location
"Converted files logged at: ..."    |

KEY DECISION POINTS:
===================

FILE PROCESSING LOGIC:
- Skip if in /originals/ subdirectory
- Skip if size <= SIZE_THRESHOLD_KB (default 5250KB)
- Skip if hash+size already in temp log (already processed)
- Process only files meeting all criteria

BACKUP STRATEGY:
- Move original to /originals/ with suffix or timestamp
- Ensure unique backup filename (increment counter if needed)
- Preserve directory structure in backup location

OPTIMIZATION PROCESS:
- Use libx264 codec with configurable CRF and preset
- Copy audio stream without reencoding (-c:a copy)
- Preserve all metadata using exiftool
- Maintain original file modification time
- Log success/failure for tracking

ERROR HANDLING:
- Directory validation at startup
- ffmpeg failure handling
- exiftool metadata copy failure handling
- File system operation error handling
- Cleanup of temporary files on failure
```

</details>


* * *

### 2. Compressing Image Files Over a Specific Size

Uncle and his darkroom skills... Has certain files that are way larger than the rest of the media that sits on the server, compress them.

I have a script that will:

- Compress images above a certain size.

- Backup the original image.

That should take care of any concerns about people gobbling up space with large images. 

The original image can be stored off site, or on a slower storage medium. It doesnt, really need to be on the Immich server, heck, it doesnt really need it exist at all. Delete it. It's up to you.

You can find the `inplace_jpg_optimizer.sh` script in the [compress2largeIMAGES](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeIMAGES) folder in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/).

That information can also be found below:


* * *

#### In-Place JPG Optimizer Introducton

- **This script will:** Shrink big JPGs in-place, keep all metadata, and back up originals.

- **To use this script:** Just run this in your photo folder with Docker.

- **What happens:** All big JPGs are optimized, originals are moved to an `originals/` backup, and you get to see the progress.

- **What to configure:** `JPEG_QUALITY` and `SIZE_THRESHOLD_KB` - Compression level, and min size of files to compress.


* * *

#### What exactly does In-Place JPG Optimizer Do

- **Finds all JPG/JPEG files in your folder tree above a certain size (default: 5250KB)**

- **Optimizes the files in-place, meaning, same filename and same location**

- **Preserves ALL metadata (EXIF, GPS, etc) and keeps the original modified date**

- **Backs up the original file (with folder structure) to an `originals/` directory**

- **Shows progress as it works**

- **Skips files it already processed if interrupted (resumes cleanly)**


* * *

#### In-Place JPG Optimizer Quick Instructions

1. Have [Docker](https://docs.docker.com/get-docker/) Ready

2. Put your photos in a folder, e.g. `./photos`

3. Copy the provided `docker-compose.yml`, `Dockerfile`, and `inplace_jpg_optimizer.sh` into that folder

4. Edit `docker-compose.yml` to change the size threshold or quality:

- `PROCESS_DIR`: Folder to process (default: `./`)

- `JPEG_QUALITY`: Output JPG quality (default: 85, range: 1-100)

- `SIZE_THRESHOLD_KB`: Only optimize files *larger* than this (default: 5250)

5. Run the container and have it remove itself when complete:

   ```bash

   docker compose run --rm jpg-optimizer

   ```

6. Done!

   - Optimized JPGs are now in place

   - Originals are in `originals/`  

   - Progress is shown as it works


* * *

#### In-Place JPG Optimizer Downloader: Copy the Github Files

If you haven’t copied the [Immich Setup: compress2largeIMAGES](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeIMAGES) folder yet, you'll need to get a few files:

- `docker-compose.yml` - The main configuration file that defines all services and their relationships. Deploy this file to launch your video compression journey.

- `inplace_jpg_optimizer.sh` - Contains the logic that creates optimized and preserved JPG files.

- `Dockerfile` - This will put an image together of all the tooling we need to complete the `inplace_jpg_optimizer.sh` script.


* * *

##### Bash/ZSH Script to Download In-Place JPG Optimizer

If you have not dowloaded anything yet, here is a `Bash` script to download all the required files for the In-Place JPG Optimizer script, ran in Docker:

```bash

BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/compress2largeIMAGES/" && for file in "docker-compose.yml" "inplace_jpg_optimizer.sh" "Dockerfile"; do curl -O "$BASE_URL$file"; done

```


* * *

##### Powershell Script to Download In-Place JPG Optimizer

Again, on Windows, if you have not dowloaded anything yet here is a `Powershell` script to download all the required files for the In-Place JPG Optimizer script, ran in Docker:

```powershell

$BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/compress2largeIMAGES/"; @("docker-compose.yml","inplace_jpg_optimizer.sh","Dockerfile") | ForEach-Object { Invoke-WebRequest -Uri "$BASE_URL$_" -OutFile ".\$(Split-Path $_ -Leaf)" }

```


* * *

#### In-Place JPG Optimizer FAQ

**Q: Will this overwrite my photos?**  
A: Yes, but it moves the original to `originals/` first, preserving the folder structure.

**Q: What if the process is interrupted?**  
A: It keeps a log and will skip already optimized files on the next run.

**Q: What if I want to keep all metadata?**  
A: All metadata is preserved in the optimized file.

**Q: Will it change the timestamp on my files?**  
A: Only optimized files get their modified date set to the original's modified date. Files not optimized are untouched.

**Q: How do I change the size threshold or quality?**  
A: Edit `docker-compose.yml` and set `SIZE_THRESHOLD_KB` or `JPEG_QUALITY` as you wish.


* * *

#### In-Place JPG Optimizer Example Output

```
Scanning for JPG files...
Total JPG files found: 412
Files below 5250KB (untouched): 309
Files to optimize: 103

Starting optimization of 103 files...

[1/103] (0%)
Optimizing: ./IMG_1234.JPG
✓ Optimized: IMG_1234.JPG (8123KB → 4210KB, saved 48%)
 → Set mtime for IMG_1234.JPG to 202406221530.12
```

...some time passes...

```text
[103/103] (100%)
Processing complete!
- Optimized: 103 files
- Failed: 0 files

All done! JPG files larger than 5250KB have been optimized in-place.
Optimized files have had their mtimes set to their original modified date.
Originals have been moved to: ./originals
```


* * *

#### In-Place JPG Optimizer Troubleshooting

- **Not enough space:** Make sure you have room for the `originals/` backup.

- **Permissions:** Run as a user with read/write access to your photo folder.

- **Docker not installed:** To fix, [Install Docker](https://docs.docker.com/get-docker/) or [Docker Destkop](https://www.docker.com/products/docker-desktop/).



* * *
* * *

#### Details About the In-Place JPG Optimizer

I have included a details section if you wanted a deeper dive into how this script was made, and it's function.


* * *

##### Tools Used

- `djpeg` and `cjpeg` – from a JPEG toolkit (e.g. [libjpeg-turbo](https://github.com/libjpeg-turbo/libjpeg-turbo) or [MozJPEG](https://github.com/mozilla/mozjpeg)).  The script pipes each image through `djpeg` (to decode) into `cjpeg` (to re-encode) with options `-optimize -progressive`.  These flags produce smaller final JPEGs (the `-optimize` option “is worth using when you are making a ‘final’ version”).  *(Note: piping can strip EXIF data, so the script uses [exiftool](#) below to restore metadata.)*

- `exiftool` – for copying metadata.  After recompression the script runs:

- `md5sum` (or `md5`) – for hashing files.  The script uses `md5sum` (Linux) or `md5 -q` (macOS) to detect if an image has already been processed.


* * *

##### Log File

The script keeps a history log of processed files (`$PROCESS_DIR/.jpg_files_optimized--keepme.log`).  This log stores one line per image, with fields separated by `|`:

```
filepath|hash|compressed_size|date|original_size|compressed_size
```

After each successful optimization, the script appends a line via the `add_to_temp_log` function:

```bash
add_to_temp_log "$jpg_file" "$compressed_hash" "$original_size" "$new_size"
```

This function calculates the current date and writes the entry: e.g.

```
/path/to/image.jpg|d41d8cd98f00b204e9800998ecf8427e|123456|2025-06-25 23:00:00|234567|123456
```

(Last two columns are original and new sizes in bytes.)

The log is also used to check if a file was already processed: before re-optimizing, the script compares the current file’s hash and size to the log entries.  If a match is found, that file is skipped as “already processed”.


* * *

##### File Locking

To prevent multiple instances from running at once (which could corrupt the log or compete for files), the script uses a lock file.  It does this by redirecting a file descriptor and using `flock`:

```bash
LOCKFILE="/tmp/.jpg_optimizer.lock"
exec 200>"$LOCKFILE"
flock -n 200 || {
  echo "Another instance is already running. Exiting." >&2
  exit 1
}
```

Here `exec 200> "$LOCKFILE"` opens file descriptor 200 on the lockfile, and `flock -n 200` obtains an exclusive non-blocking lock.  If locking fails, the script exits.  This is a common Bash pattern for locking.  A `cleanup()` function is hooked via `trap` on `INT`, `TERM`, and `EXIT` to remove the lock on exit.


* * *

##### Scanning and Threshold Check

The script builds its list of images with `find`.  In the function `get_files_to_optimize`, it runs:

```bash
find "$PROCESS_DIR" -type f \( -iname "*.jpg" -o -iname "*.jpeg" \) -print0 | while IFS= read -r -d '' file; do
    # Skip files under the backups folder
    if [[ "$file" == *"/originals/"* ]]; then continue; fi
    # Only select if above size threshold and not already in log
    if file_above_threshold "$file" && ! is_file_processed "$file"; then
        echo "$file"
    fi
done
```

This ensures only JPEGs in (or below) `PROCESS_DIR` are considered, skipping any in the `originals` backup subdirectory.  The helper `file_above_threshold` checks `stat` to compare the file’s size to the threshold in bytes.  Small files are skipped (counted as “untouched”), and already-logged files are ignored.

A summary function `count_jpg_files` prints counts of total files found, below-threshold, already processed, and to-be-optimized.  This gives an overview before the actual optimization run.


* * *

##### Backup Originals

Before overwriting any JPEG, the script moves the original file into the backup directory (`$ORIGINALS_DIR`) using the `backup_original` function.  This function:

- Strips off the base `PROCESS_DIR` path to determine a relative path.

- Creates a mirrored directory under `originals/`.

- Builds a new filename: either appending the `SUFFIX` (if set) or adding a UNIX timestamp.

- Ensures uniqueness by appending a counter if that filename already exists.

- Finally `mv`s the file to the backup location, preserving the full relative folder structure, and echoes the backup path.

> For example, `image.jpg` might be backed up as `image_1624567890.jpg` in `originals/`, or `image_suffix.jpg` if `SUFFIX=_suffix`. This way you never lose the uncompressed original.  (The script logs the original path in the log as well.)


* * *

##### Optimization Pipeline

For each file to optimize, the script runs `optimize_jpg_file`.  Here’s an excerpt of that code:

```bash
# Copy original to a temp location for metadata and fallback
cp "$jpg_file" "$temp_backup"

# Recompress JPG with cjpeg for optimization
if djpeg "$temp_backup" | cjpeg -quality "$JPEG_QUALITY" $JPEG_OPT_FLAGS > "$temp_file"; then
    # Copy all metadata from original to recompressed
    if exiftool -TagsFromFile "$temp_backup" -all:all "$temp_file" -overwrite_original > /dev/null; then
        local new_size=$(stat -c%s "$temp_file")
        # Move original to backup dir
        backup_original "$jpg_file"
        # Replace original with optimized
        mv "$temp_file" "$jpg_file"
        # Restore original modification time
        set_file_mtime "$jpg_file" "$original_mtime"
        # Log the compression
        local compressed_hash=$(get_file_hash "$jpg_file")
        add_to_temp_log "$jpg_file" "$compressed_hash" "$original_size" "$new_size"
        ...
        echo "✓ Optimized: $(basename "$jpg_file") (saved ${percent_savings}% )"
    fi
else
    echo "✗ Error recompressing: $jpg_file"
fi
```

So, for each image, the script:

1. **Copy to temp**: The original is first `cp`ed to `$file.tempbackup`, so we have it safe for metadata and size checks.

2. **Recompress**: It pipes `djpeg` into `cjpeg` with the quality and flags, outputting a new `.tmp` file.  (This effectively decodes the JPEG to raw pixels and re-encodes it.)  As noted, piping can drop EXIF metadata, so…

3. **Restore metadata**: The script runs `exiftool -TagsFromFile original backupfile.jpg` to copy all metadata into the new file.  This ensures EXIF/IPTC data is retained.

4. **Check success**: If the above steps succeed, it then computes the new file’s size and hash.

5. **Backup original**: The original `$jpg_file` is moved into the backup directory by calling `backup_original`.

6. **Replace and touch**: The optimized temp file replaces the original (`mv` into its place), and the script uses `touch -t` to set its modification time back to the original date.  (This preserves the original timestamp.)

7. **Log entry**: Finally it logs the operation (file path, hash, old size, new size, date) using `add_to_temp_log`.


After each file, the script prints a message like “✓ Optimized: image.jpg (1234KB → 987KB, saved 20%)” based on the size savings.  If any step fails (recompression or metadata copy), it prints an error and cleans up temp files.

These steps are run in a loop over all found files.  The `process_all_jpgs` function collects the file list, then iterates with progress counters, printing `[N/M (P%)]` before each file. At the end it summarizes how many succeeded/failed and total log entries.

Check out the ASCII flow diagram below to help illustrate this pipeline from scanning → compressing → backing-up → logging.


* * *

##### ASCII Flow of the In-Place JPG Optimizer script


<details>

<summary>Visual Script Breakdown</summary>

```
IN-PLACE JPG OPTIMIZER - EXECUTION FLOW
═══════════════════════════════════════════════════════════════════════════════════════════════

START
  │
  ├─ PROCESS LOCKING & INITIALIZATION
  │   ├─ Create lockfile (/tmp/.jpg_optimizer.lock)
  │   ├─ Check for existing instance ──[RUNNING]──► EXIT "Already running"
  │   │                                    │
  │   │                                   [OK]
  │   │                                    ▼
  │   ├─ Set trap for cleanup (INT/TERM/EXIT signals)
  │   └─ Initialize environment variables:
  │       ├─ PROCESS_DIR=/workdir
  │       ├─ JPEG_QUALITY=85
  │       ├─ SIZE_THRESHOLD=5250KB (5,376,000 bytes)
  │       ├─ ORIGINALS_DIR=/workdir/originals
  │       └─ TEMP_LOG_FILE=.jpg_files_optimized--keepme.log
  │
  ├─ VALIDATION & SETUP
  │   ├─ Check if PROCESS_DIR exists ──[NO]──► EXIT ERROR
  │   │                                  │
  │   │                                 [YES]
  │   │                                  ▼
  │   ├─ init_logs() - Create/touch temp log file
  │   └─ Load existing processing history
  │
  ├─ FILE DISCOVERY & ANALYSIS
  │   ├─ Scan for JPG/JPEG files (case-insensitive)
  │   ├─ Skip files in /originals/ directory
  │   ├─ For each file found:
  │   │   ├─ Check file_above_threshold() (>5250KB)
  │   │   ├─ Check is_file_processed() (hash+size match in log)
  │   │   └─ Categorize file for processing
  │   │
  │   └─ Display file count summary:
  │       ├─ Total JPG files found: X
  │       ├─ Files below 5250KB (untouched): Y
  │       ├─ Files already processed: Z
  │       └─ Files to optimize: W
  │
  ├─ PRE-PROCESSING VALIDATION
  │   ├─ Check if files_to_optimize > 0 ──[NO]──► EXIT "No files to optimize"
  │   │                                     │
  │   │                                    [YES]
  │   │                                     ▼
  │   └─ Display processing warnings and backup info
  │
  ├─ MAIN PROCESSING LOOP
  │   │
  │   └─ For each file to optimize:
  │       │
  │       ├─ Display progress: [N/Total] (X%)
  │       │
  │       ├─ optimize_jpg_file() PROCESS:
  │       │   │
  │       │   ├─ Capture original mtime: get_file_mtime()
  │       │   ├─ Create temp backup: file.jpg.tempbackup
  │       │   ├─ Record original_size for statistics
  │       │   │
  │       │   ├─ RECOMPRESSION PIPELINE:
  │       │   │   ├─ djpeg tempbackup | cjpeg -quality 85 -optimize -progressive
  │       │   │   └─ Output to: file.jpg.tmp
  │       │   │
  │       │   ├─ METADATA PRESERVATION:
  │       │   │   ├─ exiftool -TagsFromFile tempbackup -all:all file.jpg.tmp
  │       │   │   └─ Check success ──[FAIL]──► Cleanup & Return Error
  │       │   │                        │
  │       │   │                       [OK]
  │       │   │                        ▼
  │       │   ├─ BACKUP & REPLACEMENT:
  │       │   │   ├─ backup_original() - Move original to /originals/
  │       │   │   │   ├─ Create backup directory structure
  │       │   │   │   ├─ Generate unique backup filename:
  │       │   │   │   │   ├─ With SUFFIX: basename_SUFFIX.jpg
  │       │   │   │   │   └─ With timestamp: basename_timestamp.jpg
  │       │   │   │   └─ Handle filename conflicts with counter
  │       │   │   │
  │       │   │   ├─ mv file.jpg.tmp → file.jpg (replace original)
  │       │   │   └─ set_file_mtime() - Restore original timestamp
  │       │   │
  │       │   ├─ LOGGING & STATISTICS:
  │       │   │   ├─ Calculate compression savings (KB & percentage)
  │       │   │   ├─ Generate file hash: get_file_hash()
  │       │   │   ├─ add_to_temp_log() with pipe-delimited format:
  │       │   │   │   └─ filepath|hash|size|date|original_size|compressed_size
  │       │   │   └─ Display: "✓ Optimized: file.jpg (XKB → YKB, saved Z%)"
  │       │   │
  │       │   └─ Cleanup temp files & return success
  │       │
  │       └─ Update progress counter
  │
  ├─ FINAL STATISTICS & CLEANUP
  │   ├─ Display processing summary:
  │   │   ├─ Files optimized: X
  │   │   ├─ Files failed: Y
  │   │   └─ Total in log: Z
  │   │
  │   └─ cleanup() function (via trap):
  │       ├─ Remove lockfile
  │       └─ Release file lock
  │
  └─ END

PROCESSING DECISION TREE
════════════════════════
For each JPG file found:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│  File: example.jpg                                                              │
│  │                                                                              │
│  ├─ In /originals/ directory? ──[YES]──► SKIP (avoid processing backups)       │
│  │                                │                                             │
│  │                               [NO]                                           │
│  │                                ▼                                             │
│  ├─ Size > 5250KB? ──[NO]──► SKIP (below threshold)                            │
│  │                    │                                                         │
│  │                   [YES]                                                      │
│  │                    ▼                                                         │
│  ├─ Already processed? ──[YES]──► SKIP (hash+size match in log)                │
│  │   (hash + size check)  │                                                     │
│  │                       [NO]                                                   │
│  │                        ▼                                                     │
│  └─ ADD TO OPTIMIZATION QUEUE                                                   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

TEMP LOG FILE STRUCTURE
═══════════════════════
File: .jpg_files_optimized--keepme.log
Format: filepath|hash|size|date|original_size|compressed_size

Example entries:
/workdir/photo1.jpg|a1b2c3d4e5f6|2048000|2024-01-15 14:30:25|3145728|2048000
/workdir/subdir/photo2.jpg|f6e5d4c3b2a1|1536000|2024-01-15 14:31:12|2621440|1536000

BACKUP DIRECTORY STRUCTURE
═══════════════════════════
Original: /workdir/subdir/photo.jpg
Backup:   /workdir/originals/subdir/photo_1642251825.jpg
                                    └─ timestamp or suffix

TOOLS & UTILITIES USED
═══════════════════════
┌─ Image Processing ─┐    ┌─ Metadata Handling ─┐    ┌─ File Operations ─┐
│ • djpeg (decode)    │    │ • exiftool (copy)    │    │ • find (discover)  │
│ • cjpeg (encode)    │    │ • date (timestamps)  │    │ • stat (file info) │
│ • Quality: 85       │    │ • touch (set mtime)  │    │ • mv/cp (move/copy)│
│ • -optimize flag    │    │ • md5sum/md5 (hash)  │    │ • flock (locking)  │
│ • -progressive flag │    │                      │    │ • trap (cleanup)   │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘

HELPER FUNCTIONS USED:
═════════════════════
• get_file_hash() - MD5 hash or size_mtime fallback
• get_file_size_bytes() - Cross-platform file size
• is_file_processed() - Check against temp log
• file_above_threshold() - Size comparison
• get_file_mtime() / set_file_mtime() - Timestamp handling
• backup_original() - Move to originals directory
• add_to_temp_log() - Pipe-delimited logging


SAFETY FEATURES
═══════════════
├─ Process Locking: Prevents multiple instances
├─ Original Backup: All originals moved to /originals/ before modification
├─ Metadata Preservation: Full EXIF data copied to optimized files
├─ Timestamp Preservation: Original modification times restored
├─ Processing History: Permanent log prevents duplicate processing
├─ Error Handling: Failed operations don't affect originals
├─ Unique Filenames: Backup naming prevents overwrites
└─ Signal Trapping: Clean exit on interruption

```

</details>


* * *

### 3. Compressing CR2 Files down to JPEG

I made a script to import a family member's CR2 library. 

We're going to presume the family member's CR2 library will remain on their prem, maintained by them, but we all want to see their photos in Immich.

So this script assumes you're at a remote location, prepping content to import back to your Immich server - but to do so later with Immich-GO or Immich-CLI.

So, the script assumes you're outputting not to Immich, but to a folder, or external hard drive, or network resource, whatever. You will need somewhere to store these **new** files.

You can find the `cr2jpeg.sh` script in the [batchCR2intoJPEG](https://github.com/MarcusHoltz/immich-setup/tree/main/batchCR2intoJPEG) folder in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/).


* * *

#### CR2intoJPEG Introducton

- **This script will:** Convert Canon `CR2` RAW files to optimized JPEGs, compress large JPG files above 5250KB, copy MP4 videos, and set proper timestamps on all files based on their EXIF data while maintaining directory structure and avoiding duplicate processing.

- **To use this script:** Set the `SRC_ROOT` and `DST_ROOT` environment variables to your input and output directories respectively, then run the script in a Docker container with those directories mounted as volumes.

- **What happens:** The script scans your source directory, **converts** `CR2` files to `JPEG` format, compresses JPG files larger than 5250KB while leaving smaller ones unchanged, copies MP4 files as-is, preserves all metadata, sets file timestamps based on photo EXIF data, and logs everything to prevent reprocessing the same files.

- **What to configure:** Modify the `JPEG_QUALITY` variable (default 85) for different compression levels, adjust `SIZE_THRESHOLD` (default 5,376,000 bytes = 5250KB) to change when JPG files get compressed, and customize the SRC_ROOT and DST_ROOT environment variables for your specific input and output paths.


* * *

#### What CR2intoJPEG will do

If you run the script, it is because you need the following features:

* Convert your `.CR2` to optimized lossy`.JPG`

* Compresses `.JPG`s above a specific size (>5.25MB)

* Copies all `.JPG`s & `.MP4`s

* Preserves folders & EXIF timestamps

* Avoids reprocessing files (uses a log)


* * *

#### CR2JPEG - Step 1: Copy the Github Files

If you haven’t copied the [Immich Setup: batchCR2intoJPEG](https://github.com/MarcusHoltz/immich-setup/tree/main/batchCR2intoJPEG) folder yet, you'll need to get a few files:

- `docker-compose.yml` - The main configuration file that defines all services and their relationships. Deploy this file to launch your video compression journey.

- `cr2jpeg.sh` - Contains the logic that creates optimized and preserved JPG files.

- `Dockerfile` - This will put an image together of all the tooling we need to complete the `cr2jpeg.sh` script.


* * *

##### Bash/ZSH Script to Download CR2JPEG

If you have not dowloaded anything yet, here is a `Bash` script to download all the required files for the CR2 --> JPEG script ran in Docker:

```bash

BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/batchCR2intoJPEG/" && for file in "docker-compose.yml" "cr2jpeg.sh" "Dockerfile"; do curl -O "$BASE_URL$file"; done

```


* * *

##### Powershell Script to Download CR2JPEG

One more time, on Windows, if you have not dowloaded anything yet here is a `Powershell` script to download all the required files for the CR2 --> JPEG script ran in Docker:

```powershell

$BASE_URL="https://raw.githubusercontent.com/MarcusHoltz/immich-setup/main/batchCR2intoJPEG/"; @("docker-compose.yml","cr2jpeg.sh","Dockerfile") | ForEach-Object { Invoke-WebRequest -Uri "$BASE_URL$_" -OutFile ".\$(Split-Path $_ -Leaf)" }

```


* * *

#### CR2JPEG - Step 2: Organize Your Files

You'll need two directories:

- One **input directory** (e.g., `/home/you/photos_input`)

- One **output directory** (e.g., `/home/you/photos_output`)

The input directory should contain folders/files with `.CR2`, `.JPG`, `.MP4` content.


* * *

#### CR2JPEG - Step 3: Modify docker-compose.yml for Your Purposes

The docker-compose.yml file contains many variables that can be configured to fit your needs. Please look into them:

- `JPEG_QUALITY`: This is the compression factor. How much to compress your images?  (85-90 is typical for Google Pixel/GCam)

- `SIZE_THRESHOLD`: Set this to the minimum size you'd like to compress (in bytes)

Sorry about the bytes, here's a conversion table to help!

* * *

##### Byte File Size Conversion Table: **Bytes to Kilobytes and Megabytes**

| **Bytes**            | **Kilobytes (KB)** | **Megabytes (MB)** |
| -------------------- | ------------------ | ------------------ |
| **1,000 Bytes**      | 1 KB               | 0.00097656 MB      |
| **10,000 Bytes**     | 9.7656 KB          | 0.00953674 MB      |
| **50,000 Bytes**     | 48.828 KB          | 0.0476837 MB       |
| **100,000 Bytes**    | 97.656 KB          | 0.095367 MB        |
| **500,000 Bytes**    | 488.281 KB         | 0.476837 MB        |
| **1,000,000 Bytes**  | 976.562 KB         | 0.976562 MB        |
| **5,000,000 Bytes**  | 4,882.812 KB       | 4.768371 MB        |
| **10,000,000 Bytes** | 9,765.625 KB       | 9.536743 MB        |


* * *

#### CR2JPEG - Step 4: Try CR2intoJPEG out

You can just run the script in Linux, it will do as detailed above, but you will need to specifiy the input and output directories.

```bash

SRC_ROOT=/path/to/your/test_input DST_ROOT=/path/to/test_output ./cr2jpeg.sh

```


* * *

##### Now Try CR2intoJPEG out with Docker

This script was originally built to run in a Docker container, why install software on your Laptop you're only using the script once?

If you want, you can use the dockerized version - just make sure to:

1. Put your photos in a folder (e.g., `/home/yourname/photos_input`)

2. Make an output folder (e.g., `/home/yourname/photos_output`)

3. Run Docker Compose



* * *

##### Run the CR2intoJPEG Container

If you get these three files in the working directory, you can run:


* * *

###### 1. Build the Docker Image

`docker-compose build`


* * *

###### 2. Run the Processing

Then run the script with:

`docker-compose run --rm photo-processor`


* * *

###### 3. Change the script? Re-create the container (optional)

If you modify the script, you will need to load the script back into the image, and re-run the container.

`docker-compose up --build --force-recreate`


* * *

###### 4. The CR2JPEG Script Will Have Done

- Logged actions to `/output/processed_files.log`

- Output files to be imported into Immich in the same subdirectory structure as in `/input`


* * *

##### ASCII Flow of the CR2intoJPEG script


<details>

<summary>Visual Script Breakdown</summary>

```text
CR2JPEG BATCH PROCESSOR - EXECUTION FLOW
═══════════════════════════════════════════════════════════════════════════════════════════════

START
  │
  ├─ Initialize Environment
  │   ├─ Set src_root (/input) & dst_root (/output)
  │   ├─ Create log files (processed_files.log, .progress_counter)
  │   └─ Set JPEG_QUALITY=85, SIZE_THRESHOLD=5250KB
  │
  ├─ Validate Input Directory
  │   └─ Check if src_root exists ──[NO]──► EXIT ERROR
  │                                   │
  │                                  [YES]
  │                                   ▼
  ├─ File Discovery & Counting
  │   ├─ Scan for CR2 files ──► count_cr2_files()
  │   ├─ Scan for JPG files ──► count_jpg_files()
  │   ├─ Scan for MP4 files ──► count_mp4_files()
  │   └─ Check processed_files.log for already processed files
  │
  ├─ Display Summary
  │   ├─ Show file counts by type
  │   ├─ Show already processed count
  │   └─ Show processing strategy
  │
  ├─ PROCESSING PHASE
  │   │
  │   ├─ CR2 FILES PROCESSING ──[if total_cr2 > 0]
  │   │   │
  │   │   └─ For each CR2 file:
  │   │       ├─ Check if already_processed() ──[YES]──► Skip
  │   │       │                                   │
  │   │       │                                  [NO]
  │   │       │                                   ▼
  │   │       ├─ Create output directory structure
  │   │       ├─ dcraw -c -w file.cr2 | cjpeg → output.jpg
  │   │       ├─ exiftool: Copy ALL metadata CR2→JPG
  │   │       ├─ set_file_timestamp() using EXIF DateTimeOriginal
  │   │       ├─ add_to_log() & update_progress()
  │   │       └─ Display: "✓ Converted: file.cr2 → file.jpg"
  │   │
  │   ├─ JPG FILES PROCESSING ──[if total_jpg > 0]
  │   │   │
  │   │   └─ For each JPG file:
  │   │       ├─ Check if already_processed() ──[YES]──► Update timestamp only
  │   │       │                                   │
  │   │       │                                  [NO]
  │   │       │                                   ▼
  │   │       ├─ Check file_above_threshold() (5250KB)
  │   │       │   │
  │   │       │   ├─[LARGE FILE >5250KB]──► COMPRESSION PATH
  │   │       │   │   ├─ djpeg | cjpeg → temp file
  │   │       │   │   ├─ exiftool: Copy metadata
  │   │       │   │   ├─ set_file_timestamp()
  │   │       │   │   ├─ Calculate & show compression savings
  │   │       │   │   └─ Display: "✓ Compressed: XKB → YKB (saved Z%)"
  │   │       │   │
  │   │       │   └─[SMALL FILE ≤5250KB]──► COPY ONLY PATH
  │   │       │       ├─ cp -p (preserve timestamps)
  │   │       │       ├─ set_file_timestamp() using EXIF
  │   │       │       └─ Display: "✓ Copied with timestamp update"
  │   │       │
  │   │       └─ add_to_log() & update_progress()
  │   │
  │   └─ MP4 FILES PROCESSING ──[if total_mp4 > 0]
  │       │
  │       └─ For each MP4 file:
  │           ├─ Check if already_processed() ──[YES]──► Skip
  │           │                                   │
  │           │                                  [NO]
  │           │                                   ▼
  │           ├─ Create output directory structure
  │           ├─ cp -p (copy with preserved timestamps)
  │           ├─ set_file_timestamp() if EXIF available
  │           ├─ add_to_log() & update_progress()
  │           └─ Display: "✓ Copied: file.mp4 → file.mp4"
  │
  ├─ FINAL STATISTICS
  │   ├─ Read counters from temp files
  │   ├─ Display processing summary:
  │   │   ├─ CR2 files converted: X
  │   │   ├─ JPG files compressed & optimized: Y
  │   │   ├─ JPG files with timestamp only: Z
  │   │   ├─ Already processed files (timestamp updated): W
  │   │   └─ MP4 files copied: V
  │   └─ Clean up temporary counter files
  │
  └─ END

PROGRESS TRACKING SYSTEM
────────────────────────
┌─ .progress_counter file ─┐    ┌─ processed_files.log ─┐
│ Current: 15/42 (35%)     │    │ /input/IMG_001.CR2     │
│ Updates after each file  │    │ /input/IMG_002.JPG     │
└─────────────────────────┘    │ /input/VID_001.MP4     │
                               │ ... (full file paths)  │
                               └─────────────────────────┘

TOOLS USED IN PROCESSING
────────────────────────
┌─ CR2 → JPG Conversion ──┐    ┌─ JPG Optimization ──┐    ┌─ Metadata & Timestamps ─┐
│ • dcraw (RAW decoder)   │    │ • djpeg (decompress) │    │ • exiftool (copy EXIF)  │
│ • cjpeg (JPEG encoder)  │    │ • cjpeg (recompress) │    │ • date (timestamp conv) │
│ • Quality: 85           │    │ • -optimize          │    │ • touch (set mtime)     │
│ • -progressive flag     │    │ • -progressive       │    │ • stat (get file size)  │
└─────────────────────────┘    └─────────────────────┘    └─────────────────────────┘

DOCKER INTEGRATION
──────────────────
Environment Variables:
├─ SRC_ROOT=/input (mounted volume)
├─ DST_ROOT=/output (mounted volume)
└─ Container has all required tools pre-installed
```

</details>


* * *

## Fifth step: Connect Remotely with Netbird

NetBird is an open-source hole-punching zero-trust networking platform that lets your devices form a secure mesh VPN. 


In my example setup, the Immich server (at 172.21.192.111) and my mobile devices join a NetBird policy restricted network. 

I have a Docker container on the Immich server acts as a *routing peer*, forwarding traffic between NetBird and a reverse proxy that has access to my home Immich server. 

Let me draw a picture:

### Understanding Example Setup Network Architecture

Here is the path we want our mobile devices to take to access Immich resources:

```
Mobile Clients (Remote) 
    ↓ (connect to Netbird VPN)
Netbird Network (100.64.0.0/10)
    ↓
"Local" Reverse Proxy (172.21.8.254)
    ↓
"Local" Immich Server (172.21.192.111)
```


* * *

#### Netbird? VPN? What's about go to on here?

So, the beauty of a lot of the new Wireguard VPNs is holepunching. Only one side needs to be able to accept a connection from the internet, then the reply from that connection will create a hole in a stateful firewall to allow persistant traffic in, provided there's a keep alive packet every now and then.

A connection can be made into a network without opening any sort of public port.


* * *

### Netbird Setup - in a brief

Setting this up in Netbird is fairly straight forward, we will go through it in detail, but the jist is:

- Add Network
  - Enter name of network

- Add Resource
  - Enter name of resource
  - Enter subnet to share
  - Create destination group

- Create Policy
  -  Select or Create source generating traffic
  -  Destination will be the group made above
  -  Upperright, protocol: TCP
  -  Ports: 80

- Add Routing Peer    
  - Select a connected peer that can provide this subnet to share

- DNS Nameservers
  - Enter the domain for Immich, point to router's IP, port 53
    - Make sure you're using your router for DNS as well as Reverse Proxy


* * *

### Connecting Mobile Clients to Immich via NetBird 

In this example, we'll be setting up Jetbird on our Mobile devices to connect to a free tier of Netbird's hosted service. Our Immich instance will be behind a Reverse Proxy, which we'll connect to with our own hosted DNS, through a container on our network with Netbird.


* * *

#### NetBird Peer Verification

- **Check and verify a green dot on your peers:**  
  - Log in to your NetBird dashboard at `app.netbird.io`.
  - Go to the `Peers` section.
  - Confirm your mobile clients and Docker container (hosting Immich) are listed as connected peers.
  - Note the NetBird-assigned IP for each peer (especially the Docker container running Immich).


* * *

#### Create a Network for Immich Access

The first step is going to be creating a name of a network so we can add resources, routes, policy, etc.

  - In the NetBird dashboard, go to `Networks` on the left hand menu.
  - Click the `Add Network` button.
  - Name it something like `immich-home-net-access`.
  - Enter a Description with more detail, like when you made this, why you made this, how you plan to use this, whom shall be using this network, where it is intended to reach, etc.
  - Commit and click the `Add Network` button at the bottom.


* * *

#### Create a Resource for Netbird on your Network

We need to add a Resource (Subnet) to your Network, so external clients can find it

  - If a pop-up wizard style button doesnt appear,
  - In the new network, click `Add Resource`.
  - Name: `immich-home`.
  - Description: `Route to local network for Immich access`
  - Address: Enter your home subnet (e.g., `172.21.8.0/24` if your home LAN is in this range).
  - Destination Groups: Click on this field, then click into the search bar that appears at the top of this new menu. You should see a blinking cursor next to the magnifing glass. You can now type in the name to restrict this resource down to. This is the group for your Docker container (e.g. immich-connector).
  - Enabled: ✅
  - Click the `Add Resource` button.


* * *

#### Setting Up Network Route to the Reverse Proxy

Without a policy, the resource will not be accessible by any peers. Create a policy to control access to this resource.

  - If a pop-up wizard style button doesnt appear,
  - Under `Access Control` you can find `Policies` once on that page, click `Add Policy`.
  - Source Groups: Select the group(s) containing your mobile clients (or create these groups if needed).
  - Destination Groups: The group for the resource you just created (`immich-connector`) should already be set, if not, set it.
  - Upper right corner: `TCP` this is the protocol for the connection.
  - Ports: `80` you can now enter the ports for the protocol to allow passed through.
  - Name and everything else should be automatically generated based on the Destination Group you set earlier (`immich-connector`).
  - Click the `Save` button.


* * *

#### Adding DNS to the Reverse Proxy for our Immich Server

I dont want to have to go back into Netbird and change any DNS information if I change my reverse proxy.

So, I will set the domain, IP, and port address to that of the Router. It can find those resources for me, and I can easily set that at the "local" site.

- Click on `DNS` to expand and find `Nameservers`.
- Under `Nameservers` click on `Add Nameserver`
- We will be adding my router, it is a `Custom DNS` server.
- Click on `Add Nameserver` it is at the top, kind of greyed out.
- Enter the IP address of your router on the subnet you set as a resource (e.g. `172.21.8.254`) along with the Port of `53`.
- Distribution Groups: Enter the groups that will be trying to find the Immich server (`Mobile Phones`).
- Select the `Domains` tab at the top.
- Click on `Add Domain` it is at the top, kind of greyed out.
- Enter the domain that you have set on the router with a DNS override pointing to your reverse proxy that points to your Immich server (e.g. `immich.domain.com`).


* * *

### Jetbird

Why JetBird? Because NetBird's mobile client is closed source, and doesnt offer the option to register users with a key - OIDC login is hardcoded into the mobile app — even when self-hosted -- so if you want 100% of the NetBird features you must [have an OIDC ready to go](https://blog.holtzweb.com/posts/traefik-forwardauth-authentication-authentik).


* * *

#### DNS Entries Will Not Work With JetBird

If you use the opensource client JetBird, [you will not be able to send DNS entries to your clients](https://codeberg.org/bg443/JetBird/issues/57). That's a problem.

You will have to have public DNS entries for everyone to reach.

Just a little FYI when trying to decide how to pick a client or setup your DNS.


* * *

### 🔁 NetBird vs Headscale

| Feature                | NetBird                            | Headscale                                        |
| ---------------------- | ---------------------------------- | ------------------------------------------------ |
| Open Source            | ✅ Yes                              | ✅ Yes                                            |
| Self-host UI           | ✅ (Web UI included)                | ❌ Only CLI — 3rd-party UIs                       |
| Identity Required      | ✅ Yes (OIDC mandatory for clients) | ❌ No login required — devices are pre-authorized |
| Android support        | ✅ Yes (OIDC locked)                | ✅ Works with Tailscale Android (no login)        |
| Popularity / Ecosystem | 🟨 Medium (rising)                 | 🟩 Very large (Tailscale-compatible)             |
| Dev Simplicity         | 🟨 OIDC config needed              | 🟩 Easy with SSH or simple auth                  |
| WireGuard Backend      | ✅ Yes                              | ✅ Yes                                            |


* * *

## 🎉 Congratulations! 🎉

You've officially installed and configured Immich with remote access, that's an achievement! 

Now you have a powerful self-hosted solution for managing your photos and media, and you've also taken control of your own environment. 

There’s nothing quite like seeing something you’ve set up from the ground up start working smoothly.

Now, go ahead and enjoy the pics of your labor. 
