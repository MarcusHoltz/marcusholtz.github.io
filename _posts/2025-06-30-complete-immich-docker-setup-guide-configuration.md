---
layout: post
title: Immich Guide for Docker on UnRAID - Installation + Config
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

For the repo that goes along with this guide, visit: [https://github.com/MarcusHoltz/immich-setup](https://github.com/MarcusHoltz/immich-setup){:target="_blank"}.

* * *


## Hi, I'm new to Immich

Hello. Welcome. 

I presume you're here because you want out of a proprietary cloud-based photos/videos/memories system.

[Immich](https://www.reddit.com/r/immich/){:target="_blank"} is a good Google Photos or iCloud Photos alternative, and still allows sharing between multiple users. It may just be easy enough for your parents.

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

1. [Make changes outside of Immich, import your existing folders](https://immich.app/docs/guides/external-library/){:target="_blank"}

2. [Transfer your current folders full of photos to an Immich library](https://immich.app/docs/features/command-line-interface/){:target="_blank"}

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

- `My meme folders` - not in Immich. (use [Meme-search](https://github.com/neonwatty/meme-search){:target="_blank"})

- `All of my documentation screen shots` - not in Immich. (use [Holtzweb Blog](https://blog.holtzweb.com/){:target="_blank"})

- `All of my receipts, manuals, invoices, pdfs` - not in Immich. (use [Papra](https://github.com/papra-hq/papra){:target="_blank"} or [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx){:target="_blank"})

- `Smokie's Birthday Photos` - in Immich.

Yes, you could send all that to Immich, and have it available to search -- but **this** is a *shared Immich instance* for family, not a general dumping ground.

Each user will have the same folder structure. Any organization is done by Immich in [albums](https://github.com/simulot/immich-go#from-folder-sub-command){:target="_blank"}.



* * *

## First step: Install

We will be using Immich on an UnRAID server, no point of having the service up if the files are unavailable.

Immich has a very good [Immich on Unraid: Docker-Compose](https://immich.app/docs/install/unraid) write-up for us to use. 

If you have not read the official documentation from Immich above, do that now - or go ahead and use the one I made [below](#copy-my-files-make-a-few-edits-easy-mode)...


### Copy my files, make a few edits, easy mode

You can find the files we will be using, the same I use on my UnRAID server, in my:

- [UnRAID Immich Setup](https://github.com/MarcusHoltz/immich-setup/tree/main/unraid-immich-compose) folder.

> To setup - if you're not using UnRAID - the [docker-compose.yml](https://github.com/MarcusHoltz/immich-setup/blob/main/unraid-immich-compose/compose.manager/projects/immich/docker-compose.yml) and [env](https://github.com/MarcusHoltz/immich-setup/blob/main/unraid-immich-compose/compose.manager/projects/immich/env) files will work just fine.


* * *

### UnRAID Requirements: Part 1

To get this up and running, we need additonal software in UnRAID to support how this is set-up.

The easiest way is to install the [Docker Compose Manager](https://github.com/dcflachs/plugin-repository/blob/master/compose.manager.xml){:target="_blank"} from the UnRAID Community Applications.

You can find out more about [UnRAID's Unofficial Docker Compose Manager Plugin](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager){:target="_blank"}.

- Once it's installed, you can find it under `Plugins`.

- Add New Compose Stack, and name it `immich`.

> This will create a new folder under:
`/boot/config/plugins/compose.manager/projects/immich/`


* * *

### UnRAID Immich Setup Downloader: Copy the Github Files

If you haven’t copied the [Immich Setup: Install Immich on UnRAID Compose Github](https://github.com/MarcusHoltz/immich-setup/tree/main/unraid-immich-compose){:target="_blank"} repo yet, you'll need to get a few files:

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

If you are using [UnRAID's Docker Compose Manager Community Application](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager){:target="_blank"}:

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

If you are using [UnRAID's Docker Compose Manager Community Application](https://forums.unraid.net/topic/114415-plugin-docker-compose-manager){:target="_blank"} this is a nice feature to have.

- Make sure the stack_name is `immich`

- Your `docker-compose.override.yml` file needs to be in the same directory as the `docker-compose.yml` file.

- The `docker-compose.override.yml` file should be located at: `/boot/config/plugins/compose.manager/projects/immich/docker-compose.override.yml`


* * *

### Auto-Start Immich On Boot

Immich will fail, as the network has not fully come up yet. YMMV.


* * *

#### UnRAID Requirements: Part 2

##### Install Userscripts on UnRAID

To fix the Immich stack startup we're using the [User Scripts](https://github.com/Squidly271/user.scripts/blob/master/plugins/user.scripts.plg){:target="_blank"} plugin.

You can find out more about [The Community Application: User Scripts](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts){:target="_blank"}.

Make sure it is installed before continuing.


* * *

#### Using User Scripts for Immich Delay

I have this in my User Scripts and it runs at the start of the array.

My docker-compose stack name is `immich`. The rest should be copy and paste.

This script waits 100 seconds and then updates & restarts the docker-compose stack so it can see the network.

It then proceeds to do the same to NetBird. 

If you are using [UnRAID's User Scripts Community Application](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts){:target="_blank"} this is a nice feature to have.

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

There is even a Python based GUI you could use: [Immich-Go GUI](https://github.com/shitan198u/immich-go-gui){:target="_blank"}


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

[Immich-stack](https://github.com/majorfi/immich-stack){:target="_blank"} is designed to [automatically group similar photos](https://majorfi.github.io/immich-stack/getting-started/quick-start/){:target="_blank"} into [stacks](https://immich.app/docs/api/create-stack/){:target="_blank"} within the Immich photo management system. Its primary purpose is to help users organize large photo libraries by stacking related images—such as burst shots, [similar filenames](https://majorfi.github.io/immich-stack/api-reference/environment-variables/?h=criteria#custom-criteria_1){:target="_blank"}, or images taken in quick succession—into logical groups for easier browsing and management



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

Sorry, let me explain this a little. Their [wiki](https://majorfi.github.io/immich-stack/){:target="_blank"} isnt too easy to go by.


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

3. Cousin stores all of his photos as RAW/CR2 files (psst - tell them there's a [Lightroom Immich plugin](https://blog.fokuspunk.de/lrc-immich-plugin/){:target="_blank"}).

Yeah. **Lossy compression may be deemed acceptable.** 

Original files will always be backed up and stored off site. But if you really need compression with no loss in image data, try [immich-upload-optimizer](https://github.com/miguelangel-nubla/immich-upload-optimizer){:target="_blank"}.


* * *

### 1. Compressing Video Files Over a Specific Size

Grandma and her train rides... If you have several people uploading large video files that are tapping your storage space as outliers, compress them.

I have a script that will:

- Compress videos above a certain size.

- Backup the original video.

That should take care of any concerns about people gobbling up space with a few large video files. 

The original video can be stored off site, or on a slower storage medium. It doesnt really need to be on the Immich server, heck, it doesnt really need it exist at all. Delete it. It's up to you.

You can find the `inplace_mp4_optimizer.sh` script in the [compress2largeVIDEOS](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeVIDEOS){:target="_blank"} folder in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/){:target="_blank"}.

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

1. Have [Docker](https://docs.docker.com/get-docker/){:target="_blank"} Ready

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

If you haven’t copied the [Immich Setup: compress2largeVIDEOS](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeVIDEOS){:target="_blank"} folder yet, you'll need to get a few files:

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

- **Docker not installed:** To fix, [Install Docker](https://docs.docker.com/get-docker/){:target="_blank"} or [Docker Destkop](https://www.docker.com/products/docker-desktop/){:target="_blank"}.


* * *

### 2. Compressing Image Files Over a Specific Size

Uncle and his darkroom skills... Has certain files that are way larger than the rest of the media that sits on the server, compress them.

I have a script that will:

- Compress images above a certain size.

- Backup the original image.

That should take care of any concerns about people gobbling up space with large images. 

The original image can be stored off site, or on a slower storage medium. It doesnt, really need to be on the Immich server, heck, it doesnt really need it exist at all. Delete it. It's up to you.

You can find the `inplace_jpg_optimizer.sh` script in the [compress2largeIMAGES](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeIMAGES){:target="_blank"} folder in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/){:target="_blank"}.

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

1. Have [Docker](https://docs.docker.com/get-docker/){:target="_blank"} Ready

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

If you haven’t copied the [Immich Setup: compress2largeIMAGES](https://github.com/MarcusHoltz/immich-setup/tree/main/compress2largeIMAGES){:target="_blank"} folder yet, you'll need to get a few files:

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

- **Docker not installed:** To fix, [Install Docker](https://docs.docker.com/get-docker/){:target="_blank"} or [Docker Destkop](https://www.docker.com/products/docker-desktop/){:target="_blank"}.


* * *

### 3. Compressing CR2 Files down to JPEG

I made a script to import a family member's CR2 library. 

We're going to presume the family member's CR2 library will remain on their prem, maintained by them, but we all want to see their photos in Immich.

So this script assumes you're at a remote location, prepping content to import back to your Immich server - but to do so later with Immich-GO or Immich-CLI.

So, the script assumes you're outputting not to Immich, but to a folder, or external hard drive, or network resource, whatever. You will need somewhere to store these **new** files.

You can find the `cr2jpeg.sh` script in the [batchCR2intoJPEG](https://github.com/MarcusHoltz/immich-setup/tree/main/batchCR2intoJPEG){:target="_blank"} folder in my [Immich Setup Repo](https://github.com/MarcusHoltz/immich-setup/){:target="_blank"}.


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

If you haven’t copied the [Immich Setup: batchCR2intoJPEG](https://github.com/MarcusHoltz/immich-setup/tree/main/batchCR2intoJPEG){:target="_blank"} folder yet, you'll need to get a few files:

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

Why JetBird? Because NetBird's mobile client is closed source, and doesnt offer the option to register users with a key - OIDC login is hardcoded into the official Netbird mobile app — even when self-hosted.

So if you want NetBird on mobile you must [have an OIDC ready to go](https://blog.holtzweb.com/posts/traefik-forwardauth-authentication-authentik){:target="_blank"}, and expect to have anyone connecting use their OIDC account to login.


* * *

#### DNS Entries Will Not Work With JetBird

If you just want to use registration keys to set up mobile users, you will need to use JetBird. But, the opensource client JetBird, [will not be able to send DNS entries to your clients](https://codeberg.org/bg443/JetBird/issues/57){:target="_blank"}. That's a big problem.

You will have to have public DNS entries for everyone to reach. This defeats the purpose of having something like Netbird, it might as well just be Wireguard.

Just a little FYI when trying to decide how to pick a client or setup your DNS. If you wanted to use registration keys with mobile users, you will give up NetBird's use of DNS Nameservers.


* * *

### NetBird vs Headscale

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
