---
layout: post
title: Automated Job Search Review and Alerts Assistant
description: Step-by-step guide to building an automated job search system using ChangeDetection.io, n8n, and Ollama. 
date: 2026-01-21 11:33:00 -0700
categories: [DevOps, Automation]
tags: [homelab, automation, n8n, selfhosted, ollama, ai, docker, changedetection.io, fun, project, guide, hiring]
pin: false
image:
  path: /assets/img/header/header--n8n--job-search-automation-docker-workflow.jpg
  alt: Monitor job boards, filter with AI, get smart notifications.
---
<!--
===TO-DO===
  
  Submit to: https://n8dex.com/
  
  Make gif of the whole thing -- need telegram open, need changedetection.io open, need n8n open

  Crop my screenshots as visuals for each section

-->

# Build Your Own Automated Job Search Alert AI Agent

## Introduction

**What is this project** 

Automatically analyze incoming job postings against your personalized profile and sends smart recommendations to Telegram.

1. Scan new jobs when they appear

2. AI filters jobs into tiers

3. You write applications (AI-written resumes get binned anyway)


* * *

### Tech This Project Uses

**How does this project work**

- ChangeDetection.io monitors job boards

- n8n orchestrates AI analysis via Ollama

- and Telegram delivers smart notifications sorted with a score


* * *

#### Acknowledging Derivative Work

I admit, we're not building anything new.

We just want high-quality alerting. 

But, traditional job alerts and email blasts are slow and noisy.

> Just one quick note, 85% of jobs are filled through networking. So go our there and find your local Linux User Group!


* * *

### Architecture Overview

I admit we're mixing a lot of technology, so let me demonstrate with some ASCII:

```ascii
┌─────────────────────────────────────────────────────────────────────────┐
│                         JOB SEARCH AUTOMATION SYSTEM                    │
└─────────────────────────────────────────────────────────────────────────┘

     ┌──────────────────┐
     │   JOB BOARDS     │
     │  (hiring.cafe)   │
     │  Indeed, LinkedIn│
     └────────┬─────────┘
              │
              │ Monitors every 15min
              │
     ┌────────▼─────────┐
     │ ChangeDetection  │◄──────┐
     │   + Browser      │       │ JavaScript
     │  (Port 5000)     │       │ Rendering
     └────────┬─────────┘       │
              │                 │
              │ Webhook POST    │
              │ (new jobs)  ┌───┴───────────┐
              │             │ Browserless   │
     ┌────────▼─────────┐   │ Chrome        │
     │                  │   │ (Port 3000)   │
     │   n8n Workflow   │   └───────────────┘
     │   (Port 5678)    │
     │                  │
     │  ┌─────────────┐ │
     │  │ User Config │ │ ◄── Your profile, skills,
     │  └──────┬──────┘ │     preferences, scoring
     │         │        │
     │  ┌──────▼──────┐ │
     │  │Parse Webhook│ │ ◄── Clean job text
     │  └──────┬──────┘ │
     │         │        │
     │  ┌──────▼──────┐ │
     │  │AI Analysis  │ │──────┐
     │  └──────┬──────┘ │      │
     │         │        │      │ Ollama API
     └─────────┼────────┘      │ Call
               │               │
               │          ┌────▼─────────┐
               │          │   Ollama     │
               │          │ (Port 11434) │
               │          │              │
               │          │ qwen2.5-     │
               │          │ coder:7b     │
               │          └────┬─────────┘
               │               │
               │◄──────────────┘
               │ AI Response:
               │ FIT SCORE: 8/10
               │ RECOMMENDATION: Apply
               │
     ┌─────────▼─────────┐
     │ Check             │
     │ Recommendation    │
     └─────┬────────┬────┘
           │        │
    Skip   │        │   Apply/Maybe
           │        │
  ┌────────▼────┐ ┌─▼───────────┐
  │ Telegram   │ │  Telegram   │
  │ (Rejected) │ │  (Apply)    │
  │  Channel   │ │   DMs       │
  └────────────┘ └─────────────┘
       │                │
   Low priority    High priority
   notifications   notifications
```


* * *

## Docker Compose Setup

This setup uses docker-compose to get up and running, the images can all be pulled down, but I did not adjust my networking stack. 

This is classic works-on-my-machine documentation. You will have to adjust the networking to match your particular setup. 

This is on a MacVLAN so each container may have it's own Layer 2 connection on the host VLAN.


* * *

### Prerequisites

- Docker and Docker Compose installed
- A domain name (optional, Traefik not included)
- **About 6-8GB free disk space** (Ollama models require 4-5GB each)


* * *

### ChangeDetection.io

ChangeDetection.io needs two containers working together:

1. **changedetection**: The main application that monitors websites
2. **changedetection-browser**: A headless Chrome instance for JavaScript-heavy sites


* * *

### n8n

n8n also needs two containers for this stack:

1. **n8n**: Receives webhooks from ChangeDetection, orchestrates the AI analysis workflow, and sends Telegram notifications
2. **Ollama**: Runs large language models locally to analyze job postings


* * *

### Docker Components Relationship Map

Docker containers? Oh yeah! I have an ASCII picture for that too!!!

```
DOCKER CONTAINERS
═════════════════

┌─────────────────────────────────────────────────────────────┐
│  Docker Network: br1.224 (10.236.224.0/24)                  │
│                                                              │
│  ┌────────────────────┐     ┌──────────────────────┐       │
│  │ changedetection    │────▶│ changedetection-     │       │
│  │ IP: 10.236.224.29  │     │ browser              │       │
│  │ Port: 5000         │     │ IP: 10.236.224.30    │       │
│  │                    │     │ Port: 3000           │       │
│  │ - Monitors URLs    │     │                      │       │
│  │ - Sends webhooks   │     │ - Headless Chrome    │       │
│  │ - Stores state     │     │ - Renders JS sites   │       │
│  └────────┬───────────┘     └──────────────────────┘       │
│           │                                                  │
│           │ Webhook POST                                    │
│           │                                                  │
│  ┌────────▼───────────┐     ┌──────────────────────┐       │
│  │ n8n                │────▶│ ollama               │       │
│  │ IP: 10.236.224.26  │     │ IP: 10.236.224.XX    │       │
│  │ Port: 5678         │     │ Port: 11434          │       │
│  │                    │     │                      │       │
│  │ - Workflow engine  │     │ - AI model runner    │       │
│  │ - Orchestration    │     │ - qwen2.5-coder:7b   │       │
│  │ - Telegram sender  │     │ - Local inference    │       │
│  └────────────────────┘     └──────────────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Internet
                              │
                    ┌─────────▼──────────┐
                    │  Telegram Bot API  │
                    │  (External)        │
                    └────────────────────┘
```


* * *

## Installation: ChangeDetection.io

Setting up ChangeDetection.io using a docker compose file, there are a lot of labels you may not need.

In addition please be aware of the custom docker network.


* * *

### env file

Create a directory for your project and add these files:

**`.env` file:**
```bash
DOMAIN=example.com
SUBDOMAIN=unraid.example.com
```


* * *

### docker-compose.yml for ChangeDetection.io

Replace with your actual domains. If you don't have domains, you can access via IP addresses instead.

**`docker-compose.yml`:**

```yaml
services:
  changedetection:
    image: dgtlmoon/changedetection.io:latest
    container_name: changedetection
    hostname: changedetection
    ports:
      - "5000:5000"
    volumes:
      - /mnt/user/appdata/job_search:/datastore
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.changedetection.loadbalancer.server.port=5000"
      - "traefik.http.routers.changedetection.rule=Host(`jobsearch.${DOMAIN}`) || Host(`changedetection.${SUBDOMAIN}`)"
      - "traefik.http.routers.changedetection.service=changedetection"
      - "traefik.http.routers.changedetection.entrypoints=websecure"
      - "traefik.http.routers.changedetection.tls=true"
      - "traefik.http.routers.changedetection.tls.certresolver=cloudflare"
      - "traefik.http.routers.changedetection.tls.domains[0].sans=*.${DOMAIN}"
      - "traefik.http.routers.changedetection.tls.domains[1].sans=*.${SUBDOMAIN}"
    environment:
      - BASE_URL=http://localhost:5000
      - WEBDRIVER_URL=http://changedetection-browser:3000/webdriver
      - PLAYWRIGHT_DRIVER_URL=ws://changedetection-browser:3000/websocket
      - USE_X_SETTINGS=1
    restart: unless-stopped
    depends_on:
      - changedetection-browser
    networks:
      br1.224:
        ipv4_address: 10.236.224.29

  changedetection-browser:
    image: browserless/chrome:latest
    container_name: changedetection-browser
    hostname: changedetection-browser
    ports:
      - "3000:3000"
    environment:
      - CHROME_FLAGS=--disk-cache-size=0 --disable-application-cache
      - CONCURRENT=10
      - TOKEN=
      - TIMEOUT=30000
      - ENABLE_CORS=true
      - DEFAULT_STEALTH=true
      - ENABLE_DEBUGGER=false
      - MAX_PAYLOAD_SIZE=30000000
      - ENABLE_WEBDRIVER=true
    restart: unless-stopped
    shm_size: 1gb
    networks:
      br1.224:
        ipv4_address: 10.236.224.30

networks:
  br1.224:
    external: true
    name: br1.224
    ipam:
      config:
        - subnet: 10.236.224.0/24
```


* * *

**Key configuration points:**

The entire stack needs:

- **Port 5000**: ChangeDetection.io web interface
- **Port 3000**: Browserless Chrome (internal use only)
- **Playwright driver**: `ws://changedetection-browser:3000/websocket` enables JavaScript rendering
- **Shared network**: Both containers must communicate on the same network


The browserless/chrome container needs sufficient resources:

- **Shared memory**: `1gb` (prevents crashes on complex pages)
- **Concurrency**: `10` simultaneous browser sessions
- **Timeout**: `30000ms` (30 seconds per page load)
- **Stealth mode**: Enabled (helps avoid bot detection)

These settings ensure reliable monitoring of modern job boards that heavily use JavaScript.


* * *

### Network Configuration

This setup uses a custom bridge network (`br1.224`) with static IPs. **You have two options for networking:**

**Option 1: Simple Bridge Network (Recommended for most users)**

If you don't need custom networking, use Docker's default bridge:

```yaml
networks:
  default:
    driver: bridge
```

**Remove the `ipv4_address` lines from each service** and remove the `networks` section at the bottom. Docker will automatically assign IPs.

**Option 2: Custom Network with Static IPs (Advanced)**

If you want static IPs for firewall rules or network management, use the configuration shown in the compose files above. This requires you to:

1. Create the network first: `docker network create --driver=bridge --subnet=10.236.224.0/24 br1.224`
2. Adjust the subnet and IPs to match your network setup
3. Keep the `ipv4_address` and `networks` sections as shown

**For most users, Option 1 (simple bridge) is sufficient.** The tutorial examples use Option 2 because they're on a more complex home network setup.


* * *

### Deploy the Stack

```bash
# Navigate to your project directory
cd /path/to/changedetection

# Start the containers
docker-compose up -d

# Verify they're running
docker ps | grep changedetection
```

You should see both `changedetection` and `changedetection-browser` running.


* * *

### Access the Web Interface

Open your browser:
- **With domain**: `https://jobsearch.example.com`
- **Without domain**: `http://localhost:5000`

You'll see the ChangeDetection.io dashboard—empty for now, but ready to monitor job postings.

We're not going to do anything with it for now, we're going to finish setting up the rest of the docker components, n8n.


* * *

## Installation: n8n and Ollama

n8n means "n-eight-n" → "n[odejsautomatio]n"


* * *

### docker-compose.yml for n8n and Ollama

Create a separate directory for n8n and Ollama (or add to your existing compose file):

**`docker-compose.yml` for n8n + Ollama:**

**IMPORTANT: This example uses custom networking (`network_mode: br1.224`). If you prefer simple networking, replace `network_mode: br1.224` with a standard `networks:` section. See the Network Configuration section above for details.**

```yaml
services:
    n8n:
        container_name: n8n
        network_mode: br1.224  # Change this if using simple bridge networking
        environment:
            - TZ=America/Denver
            - GENERIC_TIMEZONE=America/Denver
            - N8N_SECURE_COOKIE=false
            - EXECUTIONS_MODE=regular
            - N8N_SKIP_WEBHOOK_DEREGISTRATION_SHUTDOWN=true
            - WEBHOOK_URL=http://10.236.224.26:5678  # Adjust to your n8n IP/domain
            - EXECUTIONS_TIMEOUT=-1
            - EXECUTIONS_TIMEOUT_MAX=-1
        volumes:
            - /mnt/user/appdata/n8n:/home/node/.n8n:rw
        restart: unless-stopped
        image: n8nio/n8n

    ollama:
        container_name: ollama
        network_mode: br1.224  # Change this if using simple bridge networking
        environment:
            - TZ=America/Denver
            - OLLAMA_ORIGINS=*
            - OLLAMA_NO_GPU=1
            - OLLAMA_LOAD_TIMEOUT=90m
        volumes:
            - /mnt/user/appdata/ollama:/root/.ollama:rw
        image: ollama/ollama
```

**Alternative for simple bridge networking:**

```yaml
services:
    n8n:
        container_name: n8n
        ports:
            - "5678:5678"
        environment:
            - TZ=America/Denver
            - GENERIC_TIMEZONE=America/Denver
            - N8N_SECURE_COOKIE=false
            - EXECUTIONS_MODE=regular
            - N8N_SKIP_WEBHOOK_DEREGISTRATION_SHUTDOWN=true
            - WEBHOOK_URL=http://localhost:5678  # Adjust to your actual domain/IP
            - EXECUTIONS_TIMEOUT=-1
            - EXECUTIONS_TIMEOUT_MAX=-1
        volumes:
            - /mnt/user/appdata/n8n:/home/node/.n8n:rw
        restart: unless-stopped
        image: n8nio/n8n
        networks:
            - default

    ollama:
        container_name: ollama
        ports:
            - "11434:11434"
        environment:
            - TZ=America/Denver
            - OLLAMA_ORIGINS=*
            - OLLAMA_NO_GPU=1
            - OLLAMA_LOAD_TIMEOUT=90m
        volumes:
            - /mnt/user/appdata/ollama:/root/.ollama:rw
        image: ollama/ollama
        networks:
            - default

networks:
    default:
        driver: bridge
```


* * *

### Key Configuration Points

**n8n settings:**
- `WEBHOOK_URL`: Must match your actual n8n address (adjust IP/domain)
- `EXECUTIONS_TIMEOUT=-1`: Allows long-running AI analysis without timeout
- Port `5678`: Web interface and webhook receiver

**Ollama settings:**
- `OLLAMA_NO_GPU=1`: Uses CPU only (remove if you have GPU)
- `OLLAMA_ORIGINS=*`: Allows n8n to connect
- `OLLAMA_LOAD_TIMEOUT=90m`: Prevents timeout when loading large models


* * *

### Deploy and Verify

```bash
# Start the containers
docker-compose up -d

# Check they're running
docker ps | grep -E 'n8n|ollama'
```


* * *

### Download AI Models

Ollama needs language models to analyze jobs. 

- The 7B model in this guide uses about 4-5GB of disk space and runs well on systems with 8GB+ RAM

>  This uses less RAM than Chrome with 20 tabs open.

* * *

Access the Ollama container and download models:

```bash
# Enter the Ollama container
docker exec -it ollama /bin/bash

# List currently installed models (probably empty)
ollama list

# Pull recommended model for job analysis
ollama pull qwen2.5-coder:7b
```

**Model recommendations:**

- `qwen2.5-coder:7b`: Best balance of quality and speed (recommended, ~4.7GB)
- `qwen2.5-coder:1.5b`: Faster, less RAM, briefer replies, runs on my GTX950 (~1GB)
- `qwen2.5:14b`: Highest quality, requires most resources (~8.5GB)

**Note:** These are the current model names as of January 2026. Check `ollama.com/library` for the latest available models.

The 7B model works well for analyzing job postings and runs on no-gpu systems with 8GB+ RAM.


* * *

### Managing Disk Space

If you download a model that's too large:

```bash
# Remove unwanted models
ollama rm model-name

# Check available space
df -h
```

Models range from 1GB (1.5B) to 8GB+ (14B+). Plan accordingly.


* * *

### Access n8n Interface

Open your browser:
- **With domain**: `https://n8n.example.com`
- **Without domain**: `http://localhost:5678`

First visit will prompt you to create an admin account. Complete the setup—you'll import the workflow in the next section.

Yay! Everything works, now let's get it to work for us how we need it to!!


* * *

## Configuring ChangeDetection.io

Now we'll set up ChangeDetection to monitor job boards and trigger our n8n workflow.


* * *

### Building Target URLs

The example uses `hiring.cafe` with specific search parameters. Build your search URLs by:

1. Going to your target job board
2. Configuring all filters (location, salary, keywords, date range)
3. Copying the complete URL from your browser address bar

**Example: Cattle rancher jobs in the last 24 hours, near Dodge City, KS (100 mile radius):**
```
https://hiring.cafe/?searchState=%7B%22searchQuery%22%3A%22cattle+-sheep%22%2C%22dateFetchedPastNDays%22%3A2%2C%22sortBy%22%3A%22date%22%2C%22maxCompensationLowEnd%22%3A%2244222%22%2C%22locations%22%3A%5B%7B%22id%22%3A%22rhk1yZQBoEtHp_8Uv--X%22%2C%22types%22%3A%5B%22locality%22%5D%2C%22address_components%22%3A%5B%7B%22long_name%22%3A%22Dodge+City%22%2C%22short_name%22%3A%22Dodge+City%22%2C%22types%22%3A%5B%22locality%22%5D%7D%2C%7B%22long_name%22%3A%22Kansas%22%2C%22short_name%22%3A%22KS%22%2C%22types%22%3A%5B%22administrative_area_level_1%22%5D%7D%2C%7B%22long_name%22%3A%22United+States%22%2C%22short_name%22%3A%22US%22%2C%22types%22%3A%5B%22country%22%5D%7D%5D%2C%22geometry%22%3A%7B%22location%22%3A%7B%22lat%22%3A37.7528%2C%22lon%22%3A-100.01708%7D%7D%2C%22formatted_address%22%3A%22Dodge+City%2C+KS%2C+US%22%2C%22population%22%3A27912%2C%22workplace_types%22%3A%5B%22Onsite%22%5D%2C%22options%22%3A%7B%22radius%22%3A100%2C%22radius_unit%22%3A%22miles%22%2C%22ignore_radius%22%3Afalse%2C%22flexible_regions%22%3A%5B%5D%7D%7D%5D%2C%22physicalPositions%22%3A%5B%22Standing%22%5D%7D
```

**Example: Jobs at Agri Beef company:**
```
https://hiring.cafe/?searchState=%7B%22dateFetchedPastNDays%22%3A2%2C%22sortBy%22%3A%22date%22%2C%22maxCompensationLowEnd%22%3A%2244222%22%2C%22companyNames%22%3A%5B%22agri+beef%22%5D%2C%22defaultToUserLocation%22%3Afalse%7D
```

These URLs are long because hiring.cafe includes all search parameters. You may find a job site you're using doesnt. 

ChangeDetection.io inlcudes actions. You can move the mouse, click, login, click a new menu, type words, hit search. Then scrape the page.


* * *

### Creating a New Watch

1. Open ChangeDetection.io web interface (http://localhost:5000 or your domain)
2. Click **"+ Add"** button in top right
3. In the "URL" field, paste your complete job board URL
4. Give it a name (e.g., "Cattle Jobs - Dodge City")
5. Add a tag (e.g., "cattle-rancher" or "agri-beef")
6. Now configure the detailed settings below


* * *

### Request Settings

Navigate to the **Request** section in your watch settings:

**Fetch Method:**
- Dropdown selection: `Playwright Chromium/Javascript via 'ws://changedetection-browser:3000/websocket'`
- This is critical - regular HTTP won't work for JavaScript-heavy sites

**Wait seconds before extracting text:**
- Value: `20`
- This allows the job board to fully load and render all listings via JavaScript


* * *

### Filters and Triggers

Navigate to **Filters and Triggers** section:

**Text filtering - Enable these checkboxes:**
- ✅ **Added lines** - captures new job postings
- ✅ **Replaced/changed lines** - captures updated postings
- ✅ **Remove duplicate lines of text** - prevents duplicate notifications

**Leave unchecked:**
- ❌ Removed lines
- ❌ All other options

This configuration ensures you only get notified about new or changed job listings, not removals or other noise.


* * *

### Fetching Settings

Navigate to **Fetching** section:

**Fetch Method** (yes, it appears twice - set both):
- Select: `Playwright Chromium/Javascript via 'ws://changedetection-browser:3000/websocket'`

**Random jitter seconds ± check:**
- Enable the checkbox
- Value: `1200` (equals 20 minutes of random variance)
- This means checks happen at slightly different times each interval, making the monitoring less predictable and more natural


* * *

### Global Filters

Navigate to **Global Filters**:

**Ignore whitespace:**
- ✅ Enable "Ignore whitespace"
- This prevents false triggers from spacing/formatting changes that don't affect content


* * *

### Notifications Setup

This is where the magic happens - connecting ChangeDetection to n8n:

**URLs field - Add both of these lines:**
```
bot - tgram://11111111111:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/1111111111?parse_mode=Markdown
n8n - form://10.236.224.26:5678/webhook/job-posting
```

**Important adjustments:**
- **Telegram line**: Replace `11111111111:AAAA...` with your actual Telegram bot token and chat ID (or remove this line if you only want n8n notifications)
- **n8n line**: Replace `10.236.224.26:5678` with your actual n8n server address and port
- The webhook path `/webhook/job-posting` must match what you'll configure in n8n (covered in next section)

**Notification Title:**
```
{{watch_title}} {{watch_tag}}
```

This includes the watch name and tag in the notification, helping you identify which search triggered.

**Notification Body:**
```
{{diff_added}}
```

This sends only the new content detected (the actual job posting text), not the entire page.

**Notification format:**
- Select: `Plain Text`
- Do not use HTML or Markdown here, as we'll process it with AI later


* * *

### Understanding the Notification Flow

When ChangeDetection detects a new job:
1. It captures the `{{diff_added}}` content (new text on the page)
2. Sends it to both Telegram (optional) and n8n webhook
3. n8n receives the webhook, passes it to Ollama for AI analysis
4. n8n sends a smart notification to Telegram with the AI's recommendation


* * *

### Test Your Configuration

Before enabling the watch:

1. Click **"Fetch & Extract"** button at the bottom of your watch settings
2. Wait for the preview to load (may take 20-30 seconds with Playwright)
3. Verify you see job listings in the extracted text
4. Check that navigation menus, footers, and ads are NOT captured
5. If you see too much noise, adjust CSS selectors or filters


* * *

### Using Tags for Organization

If monitoring multiple job searches, use tags strategically:

**By job type:**
- Tag: `cattle-rancher`
- Tag: `tech-remote`
- Tag: `construction`

**By company:**
- Tag: `agri-beef`
- Tag: `tyson-foods`

**By location:**
- Tag: `dodge-city`
- Tag: `denver`

Tags help you quickly identify which search triggered which notification.


* * *

### Save and Activate

You must save and mark it active:

1. Scroll to bottom of watch settings
2. Click **"Save"** button
3. In the main dashboard, find your watch
4. Toggle the switch to **"Active"** (green)
5. Set check frequency (e.g., check every 4 hours)

You should see "Last Checked" timestamp update after the first check runs. The system is now live.


* * *

## Setting Up the n8n Workflow

This is where everything comes together. The n8n workflow receives job postings from ChangeDetection, analyzes them with AI, and sends smart notifications to Telegram.


* * *

### Workflow Structure

The workflow has 8 nodes that process jobs in sequence:

1. **Webhook** - Receives data from ChangeDetection
2. **User Config** - Your profile, skills, preferences (customize this!)
3. **Parse Webhook Jobs** - Cleans up the incoming job text
4. **AI Analysis** - Sends to Ollama for intelligent evaluation
5. **Clean Response** - Removes markdown formatting from AI output
6. **Check Recommendation** - Routes jobs based on AI decision
7. **Send to Telegram - Apply Jobs** - Jobs worth applying to
8. **Send to Telegram - Rejected Jobs** - Jobs that don't fit


* * *

### Data Flow Diagram

n8n data ingestion diagram? 

I think we're all sold out; I can check in the back..

Oh wait, here's one right here!

```ascii
NEW JOB POSTED ON WEBSITE
         │
         ▼
┌─────────────────────┐
│ ChangeDetection     │  Playwright Browser
│ detects new text    │  renders JavaScript
└──────────┬──────────┘
           │
           │ Webhook POST with job text
           │ URL: http://n8n:5678/webhook/job-posting
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│              n8n WORKFLOW EXECUTION                  │
├─────────────────────────────────────────────────────────┤
│                                                      │
│  1. WEBHOOK RECEIVES:                               │
│     {                                                │
│       title: "Cattle Jobs - Dodge City",            │
│       message: "(added) Ranch Hand\n$45k..."        │
│     }                                                │
│                                                      │
│  2. USER CONFIG LOADS:                              │
│     - Profile: Ranch Hand, 32, Cheyenne WY          │
│     - Skills: Cattle handling, Equipment...         │
│     - Target salary: $35k-$55k                      │
│     - Fit score factors: +2 for target company...   │
│                                                      │
│  3. PARSE WEBHOOK:                                  │
│     Remove "(added)" tags, clean HTML entities      │
│     Output: "Ranch Hand\n$45k..."                   │
│                                                      │
│  4. AI ANALYSIS:                                    │
│     Send to Ollama with full profile context        │
│     ┌────────────────────────┐                 │
│     │ Ollama processes:          │                 │
│     │ - Job description          │                 │
│     │ - User profile             │                 │
│     │ - Scoring rules            │                 │
│     │ Returns: Analysis + Score  │                 │
│     └────────────────────────────┘                 │
│                                                      │
│  5. CLEAN RESPONSE:                                 │
│     Remove ```markdown``` formatting                │
│                                                      │
│  6. CHECK RECOMMENDATION:                           │
│     IF contains "RECOMMENDATION: Skip"              │
│        → Route to Rejected channel                  │
│     ELSE                                            │
│        → Route to Apply channel                     │
│                                                      │
└──────────────┬───────────┬───────────────────────────┘
               │              │
               ▼              ▼
    ┌──────────────┐   ┌─────────────┐
    │  Telegram    │   │  Telegram   │
    │  -100123...  │   │  123456789  │
    │  (Group)     │   │  (DM)       │
    └──────────────┘   └─────────────┘
```

* * *

### Importing the file for the n8n Workflow

1. Open n8n web interface (http://localhost:5678 or your domain)
2. Click **"Add workflow"** in top right
3. Click the three dots menu (⋮) → **"Import from File"**
4. Copy the complete JSON from the Appendix at the end of this document
5. Paste and click **"Import"**

The workflow will appear with all nodes pre-configured.

**⚠️ CRITICAL: After importing, you MUST update these placeholder values:**
- Both Telegram nodes: Replace `YOUR_TELEGRAM_CHAT_ID` and `YOUR_TELEGRAM_CHAT_ID_FOR_SKIPS` with your actual chat IDs
- AI Analysis node: Verify the Ollama base URL matches your setup
- Webhook node: Ensure the path matches your ChangeDetection configuration


* * *

### Configuring Your Profile

This is the most important customization. Click the **"User Config"** node to edit these values:


* * *

**Personal Information:**
```javascript
profileRole: "Ranch Hand / Cowboy"
profileAge: "32"
profileLocation: "Cheyenne, WY"
salaryMin: "35000"
salaryMax: "55000"
```

Replace these with your actual information.


* * *

**Technical Skills:**

List your actual skills in ranked order. The AI uses this to evaluate job requirements:

```javascript
technicalSkills: "1. Animal Husbandry: Cattle handling, horseback riding, herd management, calving assistance
2. Equipment Operation: Tractors, ATVs, balers, fence stretchers, basic welding
3. Land Management: Fence repair, irrigation, pasture rotation, hay production
4. Livestock Care: Branding, castration, vaccinations, wound treatment, hoof care
5. Horsemanship: Breaking, training, shoeing basics, trailer loading
6. Maintenance: Equipment repair, building maintenance, vehicle upkeep
7. Weather/Seasons: Working in all conditions, drought management, winter feeding"
```


* * *

**Target Job Titles:**

List the job titles you're searching for:

```javascript
targetJobTitles: "1. Ranch Hand / Cattle Ranch Worker
2. Cowboy / Working Cowhand
3. Livestock Handler / Cattle Feeder
4. Farm Laborer / Agricultural Worker
5. Feedlot Worker / Cattle Operations"
```


* * *

**Target Companies:**

Prioritize companies by preference (High/Good/Consider):

```javascript
targetCompaniesHigh: "King Ranch, Deseret Ranches, JA Ranch, Pitchfork Ranch, 6666 Ranch (Four Sixes), Padlock Ranch, YO Ranch, TA Ranch, Cheyenne Cattle Company, Wyoming Cattle Company"

targetCompaniesGood: "Local family ranches, Livestock auctions, Feedlot operations, Guest/Dude ranches with working cattle, Agricultural co-ops, Veterinary ranches, Horse breeding operations, USDA/BLM range management"

targetCompaniesConsider: "Rodeo stock contractors, Horse training facilities, Farm equipment dealers, Agricultural supply companies, Ranch management companies"
```


* * *

**Key Projects:**

Highlight your achievements:

```javascript
keyProjects: "- Managed herd of 200+ head through calving season
- Built and maintained 15+ miles of barbed wire fencing
- Operated heavy equipment for hay harvest (500+ bales/season)
- Trained 3 green horses for ranch work
- Assisted with emergency veterinary care during blizzard conditions"
```


* * *

**Career Preferences:**

Define what you want and don't want:

```javascript
careerPreferencesYes: "Outdoor work, working with animals, hands-on physical labor, working independently or small crew, seasonal variety, ranch/rural living"

careerPreferencesNo: "Office work, desk jobs, strict 9-5 schedules, urban locations, corporate environments"
```


* * *

**Fit Score Factors:**

This controls how the AI scores jobs (point system):

```javascript
fitScoreFactors: "+2 points: High priority ranch/company (King Ranch, Deseret, JA Ranch, Pitchfork, 6666, Padlock, YO Ranch, TA Ranch, Cheyenne/Wyoming Cattle Co)
+1 point: Good fit operation (Family ranches, Feedlots, Working guest ranches, Ag co-ops, Vet ranches, Horse breeding, USDA/BLM)
+2 points: Title matches target roles (Ranch Hand, Cowboy, Livestock Handler, Farm Laborer, Feedlot Worker)
+2 points: Strong skills match (3+ matching: cattle handling, riding, equipment operation, fence/land work, animal care)
+1 point: Rural/ranch location (Wyoming, Montana, Texas, Colorado, Nebraska, South Dakota)
+1 point: Salary in $35k-$55k range (or housing + salary)
+1 point: Livestock/cattle focus, outdoor work
-2 points: Indoor/office only, no animal work
-1 point: Urban location required
-1 point: No housing provided (when needed)"
```


* * *

**Enjoyment Factors:**

Guide the AI on what makes a job enjoyable for you:

```javascript
enjoymentFactors: "- High: Working with cattle/horses, outdoor physical work, ranch setting, seasonal variety
- Medium: Mix of animal work and equipment operation, some fence/maintenance work
- Low: Mostly equipment operation only, limited animal contact, indoor work"
```


* * *

### Configure the Webhook Node

Click the **"Webhook"** node:

**Settings to verify:**
- **HTTP Method**: POST
- **Path**: `job-posting`
- **Respond**: Immediately

The full webhook URL will be: `http://10.236.224.26:5678/webhook/job-posting`

This must match what you configured in ChangeDetection's notification URLs.


* * *

### Configure the AI Analysis Node

Click the **"AI Analysis"** node:

**Ollama Settings:**
- **Model**: `qwen2.5-coder:7b` (or whichever model you downloaded)
- **Credentials**: Click "Create New" and set:
  - **Base URL**: `http://ollama:11434` (if containers share a Docker network) or `http://10.236.224.XX:11434` (replace XX with your Ollama container's IP if using custom networking) or `http://localhost:11434` (if running on same host with port forwarding)

The AI prompt is already configured in the workflow to produce this output format:

```
🎯 Role: [Job Title]
📋 Company: [Company Name]
💰 Salary: [Salary Range or "Not Listed"]
🛠 Skills: [List 3-5 relevant skills from posting]

FIT SCORE: [X/10]
✅ Match: [2-3 brief reasons why good fit]
⚠️ Gaps: [1-2 brief concerns]
Enjoyment: [High/Medium/Low] - [Explanation]

RECOMMENDATION: [Apply/Maybe/Skip] - [Reasoning]
```


* * *

### Setting Up Telegram

You need two Telegram destinations:

**Node: "Send to Telegram - Apply Jobs"**
- **Chat ID**: Your personal Telegram chat ID (positive number like `1234567890`) - **YOU MUST REPLACE THE PLACEHOLDER**
- **Credentials**: Create Telegram bot credentials

**Node: "Send to Telegram - Rejected Jobs"**
- **Chat ID**: A group/channel ID (negative number like `-1001234567890`) - **YOU MUST REPLACE THE PLACEHOLDER**
- **Credentials**: Use same bot credentials

**⚠️ IMPORTANT:** The imported workflow contains placeholder text `YOUR_TELEGRAM_CHAT_ID` and `YOUR_TELEGRAM_CHAT_ID_FOR_SKIPS`. You **must** replace these with your actual Telegram chat IDs or the workflow will fail.


* * *

#### Getting Telegram Credentials

**Create a Telegram Bot:**
1. Open Telegram and message `@BotFather`
2. Send `/newbot` command
3. Follow prompts to name your bot
4. BotFather gives you a token like: `111001010:AAAAAADDAAAAAZZAAAAWWWAAAADDDDAAEE`
5. Save this token

**Get Your Chat ID (for DMs):**
1. Message your bot: "Hello"
2. Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
3. Look for `"chat":{"id":1234567890}` in the JSON response
4. That positive number is your personal chat ID

**Get a Group/Channel ID:**
1. Create a Telegram group or channel
2. Add your bot to the group/channel
3. Send a message in the group
4. Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
5. Look for negative number like `"chat":{"id":-1001234567890}`
6. That's your group/channel ID


* * *

### Add Telegram Credentials to n8n

1. Click either Telegram node
2. Click **"Credential to connect with"** → **"Create New"**
3. Enter your bot token from BotFather
4. Save the credential
5. Both Telegram nodes can use the same credential


* * *

### Testing the Workflow

Before activating:

1. Click **"Execute Workflow"** button (top right)
2. In the Webhook node, click **"Listen for Test Event"**
3. Trigger a test by clicking "Check now" on a ChangeDetection watch
4. Or manually send a POST request to the webhook URL
5. Watch each node execute in sequence
6. Verify the AI analysis appears correctly
7. Check that Telegram messages are sent to correct channels

**Test Input Example** (if testing manually):

You can use the Python test server from the next section to generate sample job postings.


* * *

### Save and Activate

1. Click **"Save"** button (top right)
2. Toggle workflow to **"Active"** (switch in top right)
3. The webhook is now live and ready to receive jobs from ChangeDetection


* * *

### Verify End-to-End Integration

1. Manually trigger your ChangeDetection watch (click "Check now")
2. Wait for it to detect changes
3. Check n8n **"Executions"** tab for new workflow runs
4. Verify Telegram notifications arrive with AI analysis
5. Confirm jobs route to correct channels (Apply vs Rejected)


* * *

## Testing with a Web Server

Before relying on real job boards, test your setup with a controlled environment using a simple Python web server.


* * *

### Why Test This Way?

- **Immediate feedback**: Change the text instantly and see if ChangeDetection catches it
- **No rate limiting**: Test as many times as needed
- **Controlled environment**: Know exactly what changed and when
- **Debug workflow**: Verify the entire pipeline works


* * *

### Create the Test Server

Save this as `test-job-server.py`:

```python
#!/usr/bin/env python3
from http.server import BaseHTTPRequestHandler, HTTPServer

# Your block of text (use triple quotes for multi-line)
TEXT_CONTENT = """
Staff Site Reliability Engineer
United States
$129k-$161k/yr Remote Full Time
Ping Identity: A delivering secure and seamless digital experiences through a cloud identity platform.
Experience in scalable distributed systems, cloud platforms, Go, Docker and Kubernetes, automated deployments, Git teamwork, and CS degree or equivalent.
Go, Docker, Kubernetes, Git, CI/CD
"""

class TextHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.send_response(200)
            self.send_header('Content-type', 'text/plain; charset=utf-8')
            self.send_header('Content-Length', str(len(TEXT_CONTENT)))
            self.end_headers()
            self.wfile.write(TEXT_CONTENT.encode('utf-8'))
        else:
            self.send_response(404)
            self.end_headers()

if __name__ == '__main__':
    server = HTTPServer(('10.236.99.88', 3333), TextHandler)
    print("Serving text at http://10.236.99.88:3333 (Ctrl+C to stop)")
    server.serve_forever()
```

**⚠️ IMPORTANT Configuration:**
- **IP**: `10.236.99.88` - **Replace this with your server's actual IP address.** You can find your IP with `ip addr` (Linux) or `ipconfig` (Windows)
- **Port**: `3333` - Choose any available port on your system
- **Content**: Replace `TEXT_CONTENT` with your test job posting


* * *

### Run the Test Server

```bash
# Make it executable
chmod +x test-job-server.py

# Run the server
./test-job-server.py
```

You should see: `Serving text at http://10.236.99.88:3333 (Ctrl+C to stop)`


* * *

### Configure ChangeDetection to Monitor It

1. In ChangeDetection, click **"+ Add"**
2. Enter URL: `http://10.236.99.88:3333` (replace with your actual IP)
3. Name it: "Test Job Server"
4. **Request settings**: Use `Basic fast Plaintext/HTTP Client` (no Playwright needed for plain text)
5. **Wait seconds**: `0` (instant response)
6. **Filters**: Same as before (Added lines, Replaced lines, Remove duplicates)
7. **Notifications**: Add your n8n webhook URL
8. **Check frequency**: Set to 1 minute for quick testing
9. Save and activate


* * *

### Test the Complete Pipeline

**Step 1: Baseline check**
- Click "Check now" in ChangeDetection
- First check establishes the baseline (no notification)

**Step 2: Make a change**
- Edit `test-job-server.py`
- Modify the `TEXT_CONTENT` (add a new line, change salary, etc.)
- Save the file
- Restart the server: Press `Ctrl+C`, then run `./test-job-server.py` again

**Step 3: Trigger detection**
- Click "Check now" again in ChangeDetection
- Within seconds you should see:
  - ChangeDetection shows "Changes detected"
  - n8n execution appears in "Executions" tab
  - Telegram notification arrives with AI analysis

**Step 4: Verify the analysis**
- Check Telegram for the formatted analysis
- Verify it went to correct channel (Apply vs Rejected)
- Confirm the AI scoring makes sense for the test job



* * *

### Complete Workflow Execution Timeline

Did you want a visual of what was discussed above? 

I actually have a framed picture of one, if you want to come join me in the kitchen.

There it is, next to our 4ft indoor cactus.


* * *

```
TIME: 00:00:00  ChangeDetection check starts
         │
         ▼
      [Fetch page with Playwright]
         │
TIME: 00:00:20  Wait for JavaScript to load (20 seconds)
         │
         ▼
      [Extract text content]
         │
         ▼
      [Compare with previous check]
         │
         ▼──── No changes? ──▶ EXIT (no notification)
         │
         │ Changes detected!
         ▼
TIME: 00:00:25  Send webhook to n8n
         │
         │
┌────────▼──────────────────────────────────────────┐
│ n8n WORKFLOW STARTS                                │
├────────────────────────────────────────────────────┤
│ TIME: 00:00:25  [Webhook] Receive data            │
│         │                                          │
│         ▼                                          │
│ TIME: 00:00:26  [User Config] Load profile        │
│         │                                          │
│         ▼                                          │
│ TIME: 00:00:27  [Parse] Clean job text            │
│         │                                          │
│         ▼                                          │
│ TIME: 00:00:28  [AI Analysis] Call Ollama         │
│         │                                          │
│         │ ┌──────────────────────────────┐       │
│         │ │ Ollama processes request      │       │
│         │ │ Model: qwen2.5-coder:7b       │       │
│         │ │ Time: 5-30 seconds            │       │
│         │ │ (depends on job length)       │       │
│         │ └──────────────────────────────┘       │
│         │                                          │
│ TIME: 00:00:45  [AI Analysis] Receive response    │
│         │                                          │
│         ▼                                          │
│ TIME: 00:00:46  [Clean Response] Format output    │
│         │                                          │
│         ▼                                          │
│ TIME: 00:00:47  [Check] Route by recommendation   │
│         │                                          │
│         ├─────▶ Skip? ──▶ [Telegram: Rejected]    │
│         │                                          │
│         └─────▶ Apply? ──▶ [Telegram: Apply]      │
│                                                    │
│ TIME: 00:00:50  [Telegram] Send notification      │
│                                                    │
└────────────────────────────────────────────────────┘
         │
         ▼
TIME: 00:00:51  Notification arrives on your phone 📱

TOTAL TIME: ~51 seconds from detection to notification
```


* * *

### Troubleshooting

**ChangeDetection doesn't detect changes:**
- Verify the web server is running: `curl http://10.236.99.88:3333`
- Check ChangeDetection can reach the server (network/firewall)
- Look at "Last check" details for errors

**n8n workflow doesn't trigger:**
- Check n8n webhook URL in ChangeDetection notifications
- Verify n8n workflow is Active (not Inactive)
- Look at n8n execution logs for webhook receive confirmation

**Telegram notifications don't arrive:**
- Verify bot token is correct in n8n credentials
- Check chat ID is correct (positive for DMs, negative for groups)
- Ensure bot was added to the group/channel
- Test with n8n's "Test step" button first

**AI analysis is wrong:**
- Check Ollama container is running: `docker ps | grep ollama`
- Verify model is downloaded: `docker exec -it ollama ollama list`
- Review n8n execution logs for Ollama errors
- Adjust the User Config profile to better match test jobs


* * *

## Troubleshooting and Optimization

The only other project I can recomend for this task is:

- https://github.com/Gsync/jobsync


* * *

### Common Issues and Solutions

**Issue: ChangeDetection detects too many false positives**

*Symptoms:* Getting notifications for minor page changes (timestamps, view counts, ads)

*Solutions:*
- Add CSS selectors to isolate job listings
- Enable "Ignore text" filters for known noise patterns
- Increase "Random jitter" to reduce check frequency
- Use "Remove lines containing" to filter unwanted patterns

**Issue: Jobs are being missed**

*Symptoms:* Job appears on site but no notification received

*Solutions:*
- Verify "Wait seconds before extracting" is sufficient (increase to 30-40)
- Check if site uses infinite scroll or "Load more" buttons
- Use browser developer tools to verify content is in page source
- Consider using API endpoint instead of web scraping if available

**Issue: Ollama is slow or timing out**

*Symptoms:* n8n workflow takes minutes to complete or fails

*Solutions:*
- Switch to smaller model: `qwen2.5-coder:1.5b` instead of `7b`
- Enable GPU support in Ollama (remove `OLLAMA_NO_GPU=1`)
- Increase system RAM allocated to Docker
- Increase `OLLAMA_LOAD_TIMEOUT` in docker-compose

**Issue: AI recommendations are inaccurate**

*Symptoms:* Getting "Apply" for poor fits or "Skip" for good opportunities

*Solutions:*
- Review and refine the User Config profile (skills, preferences, scoring)
- Adjust fit score point values to emphasize what matters most
- Add more specific examples to target companies
- Modify the AI prompt to focus on specific criteria
- Review several AI analyses and identify patterns in errors

**Issue: Telegram notifications are noisy**

*Symptoms:* Too many notifications, hard to track what's important

*Solutions:*
- Use separate channels for different job types
- Adjust routing logic: only send 8+ fit scores to Apply channel
- Add summary notifications (daily digest instead of instant)
- Use Telegram's mute feature for Rejected Jobs channel


* * *

### Performance Optimization

**Reduce resource usage:**
- Use CPU-only Ollama for most models (adequate for job analysis)
- Set reasonable check intervals (4-6 hours, not every minute)
- Limit concurrent browser sessions in Browserless Chrome
- Archive old n8n executions: Settings → Executions → Retention

**Speed up processing:**
- Pre-load Ollama models at container startup
- Use keepalive in Ollama API calls to maintain loaded models
- Process jobs asynchronously if monitoring many boards

**Improve accuracy:**
- Regularly update your User Config as skills/preferences change
- Review false positives/negatives weekly and adjust scoring
- A/B test different AI prompts to find best results
- Fine-tune CSS selectors to capture cleaner job text


* * *

### System Maintenance

**Weekly tasks:**
- Review Telegram channels for patterns in AI recommendations
- Check ChangeDetection for any broken watches
- Verify disk space isn't filling up (Ollama models, n8n logs)

**Monthly tasks:**
- Update Docker images: `docker-compose pull && docker-compose up -d`
- Archive or delete old n8n workflow executions
- Review and optimize User Config based on job search results
- Test with new job boards or search parameters

**As needed:**
- Rotate Telegram bot tokens if compromised
- Backup n8n workflows (export JSON)
- Update Ollama models for improved AI quality
- Expand to additional job boards or company searches


* * *

### Scaling Up

**Monitor multiple job boards:**
- Create separate ChangeDetection watches for each board
- Use tags to organize by source, location, or job type
- Consider separate n8n workflows for different industries
- Set staggered check times to distribute load

**Support multiple users:**
- Duplicate the entire stack per user (full isolation)
- Or: Share ChangeDetection/Ollama, separate n8n workflows per user
- Use different Telegram bots/channels per person
- Adjust Ollama concurrency for multiple simultaneous analyses

**Advanced workflows:**
- Auto-apply to jobs scoring 9-10 (with approval queue)
- Generate cover letters using AI based on job description
- Track application status and follow-ups
- Build a personal job database with historical fit scores


* * *

## Conclusion

You now have a fully automated, AI-powered job search assistant that works 24/7 to find opportunities matching your exact profile.


* * *

### What You've Built

✅ **Saves time**: No more manually reviewing hundreds of irrelevant job postings

✅ **Never misses opportunities**: Monitors continuously, even while you sleep

✅ **Makes smarter decisions**: AI analyzes each job against your specific skills and preferences

✅ **Maintains privacy**: Everything runs on your own hardware, no data sent to third parties

✅ **Adapts to you**: Fully customizable profile, scoring, and notification preferences

**System components:**
- **ChangeDetection.io**: Your tireless web watcher, monitoring job boards for new postings
- **n8n**: The orchestration engine connecting everything together
- **Ollama**: Local AI that understands your career goals and evaluates opportunities
- **Telegram**: Instant notifications wherever you are, organized by priority


* * *

### Next Steps

**Immediate:**
1. Let the system run for a week
2. Review the AI recommendations for accuracy
3. Tune the User Config and fit scoring based on results
4. Add more job boards or search variations

**Short-term:**
1. Experiment with different AI models for better analysis
2. Create specialized workflows for different job types
3. Set up daily digest summaries instead of instant notifications
4. Build a tracking system for applications sent

**Long-term:**
1. Expand to monitor company career pages directly
2. Integrate with job application automation
3. Add salary negotiation analysis and recommendations
4. Build historical data to identify hiring patterns and best times to apply


* * *

### Final Thoughts

Job searching is exhausting. This system handles the tedious work—monitoring, filtering, and initial evaluation—so you can focus on what matters: preparing strong applications for truly relevant opportunities.

The time investment (2-3 hours setup) pays off quickly. Most job seekers spend 10-20 hours per week browsing job boards. This system does that work continuously, notifying you only when something actually matches your profile.

Customize it, experiment with it, and make it yours. The stack is open-source and runs entirely under your control. No subscriptions, no data sharing, no limitations.

Happy job hunting! 🎯


* * *

## Appendix: n8n Workflow JSON

Below is the complete n8n workflow that powers the job analysis system. This JSON file contains all 8 nodes pre-configured and ready to import into your n8n instance.


* * *

### What's Included

This workflow contains:
- **Webhook receiver** configured for `/webhook/job-posting` endpoint
- **User Config node** with example ranch hand profile (customize for your needs)
- **Parse Webhook Jobs node** with JavaScript to clean ChangeDetection output
- **AI Analysis node** with comprehensive prompt engineering for job evaluation
- **Clean Response node** to remove markdown formatting
- **Check Recommendation node** to route jobs by AI decision
- **Two Telegram nodes** for Apply and Rejected channels


* * *

### How to Import

1. Copy the entire JSON below
2. Open your n8n interface (http://localhost:5678)
3. Click **"Add workflow"** → Menu (⋮) → **"Import from File"**
4. Paste the JSON and click **"Import"**
5. Configure your credentials:
   - Ollama API credentials (Base URL)
   - Telegram bot token and chat IDs
6. Customize the User Config node with your profile
7. Save and activate the workflow


* * *

### Important Customization Points

After importing, you must customize:

- **User Config node**: Replace all profile data with your actual information (role, skills, salary, preferences)
- **Telegram - Apply Jobs node**: Update Chat ID to your personal Telegram ID
- **Telegram - Rejected Jobs node**: Update Chat ID to your group/channel ID
- **AI Analysis node**: Verify Ollama credentials point to your Ollama instance
- **Webhook node**: Ensure the path matches your ChangeDetection notification URL

The workflow is named "Cowboy Job Search - Telegram - Must Use Single Job" in the example, but you can rename it after import.


* * *

## n8n Job Search with ChangeDetection.io Workflow JSON

Here is the full `Cowboy Job Search` json file.


* * *

### n8n workflow file 🤠

You can download this as file, or just paste it into a newly created workflow inside of n8n.

```json
{
  "name": "Cowboy Job Search - Telegram - Must Use Single Job",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "job-posting",
        "options": {}
      },
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2.1,
      "position": [
        -960,
        32
      ],
      "id": "cc3d86b6-7be8-49d4-ab9a-a882cb5e36ba",
      "name": "Webhook"
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "profileRole",
              "value": "Ranch Hand / Cowboy"
            },
            {
              "name": "profileAge",
              "value": "40"
            },
            {
              "name": "profileLocation",
              "value": "Cheyenne, WY"
            },
            {
              "name": "salaryMin",
              "value": "35000"
            },
            {
              "name": "salaryMax",
              "value": "55000"
            },
            {
              "name": "technicalSkills",
              "value": "1. Animal Husbandry: Cattle handling, horseback riding, herd management, calving assistance\n2. Equipment Operation: Tractors, ATVs, balers, fence stretchers, basic welding\n3. Land Management: Fence repair, irrigation, pasture rotation, hay production\n4. Livestock Care: Branding, castration, vaccinations, wound treatment, hoof care\n5. Horsemanship: Breaking, training, shoeing basics, trailer loading\n6. Maintenance: Equipment repair, building maintenance, vehicle upkeep\n7. Weather/Seasons: Working in all conditions, drought management, winter feeding"
            },
            {
              "name": "targetJobTitles",
              "value": "1. Ranch Hand / Cattle Ranch Worker\n2. Cowboy / Working Cowhand\n3. Livestock Handler / Cattle Feeder\n4. Farm Laborer / Agricultural Worker\n5. Feedlot Worker / Cattle Operations"
            },
            {
              "name": "targetCompaniesHigh",
              "value": "King Ranch, Deseret Ranches, JA Ranch, Pitchfork Ranch, 6666 Ranch (Four Sixes), Padlock Ranch, YO Ranch, TA Ranch, Cheyenne Cattle Company, Wyoming Cattle Company"
            },
            {
              "name": "targetCompaniesGood",
              "value": "Local family ranches, Livestock auctions, Feedlot operations, Guest/Dude ranches with working cattle, Agricultural co-ops, Veterinary ranches, Horse breeding operations, USDA/BLM range management"
            },
            {
              "name": "targetCompaniesConsider",
              "value": "Rodeo stock contractors, Horse training facilities, Farm equipment dealers, Agricultural supply companies, Ranch management companies"
            },
            {
              "name": "keyProjects",
              "value": "- Managed herd of 200+ head through calving season\n- Built and maintained 15+ miles of barbed wire fencing\n- Operated heavy equipment for hay harvest (500+ bales/season)\n- Trained 3 green horses for ranch work\n- Assisted with emergency veterinary care during blizzard conditions"
            },
            {
              "name": "careerPreferencesYes",
              "value": "Outdoor work, working with animals, hands-on physical labor, working independently or small crew, seasonal variety, ranch/rural living"
            },
            {
              "name": "careerPreferencesNo",
              "value": "Office work, desk jobs, strict 9-5 schedules, urban locations, corporate environments"
            },
            {
              "name": "fitScoreFactors",
              "value": "+2 points: High priority ranch/company (King Ranch, Deseret, JA Ranch, Pitchfork, 6666, Padlock, YO Ranch, TA Ranch, Cheyenne/Wyoming Cattle Co)\n+1 point: Good fit operation (Family ranches, Feedlots, Working guest ranches, Ag co-ops, Vet ranches, Horse breeding, USDA/BLM)\n+2 points: Title matches target roles (Ranch Hand, Cowboy, Livestock Handler, Farm Laborer, Feedlot Worker)\n+2 points: Strong skills match (3+ matching: cattle handling, riding, equipment operation, fence/land work, animal care)\n+1 point: Rural/ranch location (Wyoming, Montana, Texas, Colorado, Nebraska, South Dakota)\n+1 point: Salary in $35k-$55k range (or housing + salary)\n+1 point: Livestock/cattle focus, outdoor work\n-2 points: Indoor/office only, no animal work\n-1 point: Urban location required\n-1 point: No housing provided (when needed)"
            },
            {
              "name": "enjoymentFactors",
              "value": "- High: Working with cattle/horses, outdoor physical work, ranch setting, seasonal variety\n- Medium: Mix of animal work and equipment operation, some fence/maintenance work\n- Low: Mostly equipment operation only, limited animal contact, indoor work"
            }
          ]
        },
        "options": {}
      },
      "id": "45d83eee-b514-4660-bac6-9f3d20814139",
      "name": "User Config",
      "type": "n8n-nodes-base.set",
      "typeVersion": 2,
      "position": [
        -752,
        32
      ]
    },
    {
      "parameters": {
        "jsCode": "// ===================================================================\n// PARSE WEBHOOK JOBS - Extract job posting data from incoming webhook\n// ===================================================================\n// Input: Webhook body.message with (into) and (added) tagged lines\n// Output: Clean concatenated job posting text for AI analysis\n// ===================================================================\n\n// Get the message from the webhook (it comes through User Config node)\nconst message = $input.first().json.body?.message || '';\n\nif (!message) {\n  return [{ json: { inputText: 'No message received' } }];\n}\n\n// Process each line (data already has real newlines, not literal \\n)\nconst result = [];\nconst allLines = message.split('\\n');\n\nfor (const line of allLines) {\n  const trimmed = line.trim();\n  \n  // Skip empty lines and (changed) lines\n  if (!trimmed || trimmed.startsWith('(changed)')) {\n    continue;\n  }\n  \n  // If line has (into) or (added), remove the tag and keep the content\n  if (trimmed.startsWith('(into)')) {\n    result.push(trimmed.replace(/^\\(into\\)\\s*/, ''));\n  } else if (trimmed.startsWith('(added)')) {\n    result.push(trimmed.replace(/^\\(added\\)\\s*/, ''));\n  }\n}\n\nif (result.length === 0) {\n  return [{ json: { inputText: 'No job content found' } }];\n}\n\n// Join with newlines and clean up HTML entities\nconst output = result.join('\\n')\n  .replace(/&amp;/g, '&')\n  .replace(/&lt;/g, '<')\n  .replace(/&gt;/g, '>')\n  .replace(/<\\/pre><\\/body><\\/html>/g, ''); // Remove trailing HTML\n\n// IMPORTANT: Pass through ALL the config data from User Config node\n// AND add the parsed inputText\nconst configData = $input.first().json;\n\nreturn [{ \n  json: { \n    ...configData,  // Spread all config values\n    inputText: output  // Add parsed job text\n  } \n}];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -560,
        32
      ],
      "id": "4e0b044c-d844-401f-8bdb-8c06dc6388f9",
      "name": "Parse Webhook Jobs"
    },
    {
      "parameters": {
        "modelId": {
          "__rl": true,
          "value": "qwen2.5-coder:7b",
          "mode": "list",
          "cachedResultName": "qwen2.5-coder:7b"
        },
        "messages": {
          "values": [
            {
              "content": "=# Job Match Analysis\n\n## YOUR PROFILE (Quick Reference)\n**Role:** {{ $json.profileRole }} | **Age:** {{ $json.profileAge }} | **Location:** {{ $json.profileLocation }}\n**Salary Target:** ${{ $json.salaryMin }}-${{ $json.salaryMax }}\n\n**Technical Skills (Ranked by Expertise):**\n{{ $json.technicalSkills }}\n\n**Target Job Titles ({{ $json.profileLocation }} Market Focus):**\n{{ $json.targetJobTitles }}\n\n**Target Companies (Prioritized):**\n- HIGH PRIORITY: {{ $json.targetCompaniesHigh }}\n- GOOD FIT: {{ $json.targetCompaniesGood }}\n- CONSIDER: {{ $json.targetCompaniesConsider }}\n\n**Key Projects:**\n{{ $json.keyProjects }}\n\n**Career Preferences:**\n✅ {{ $json.careerPreferencesYes }}\n❌ {{ $json.careerPreferencesNo }}\n\n---\n\n## INSTRUCTIONS FOR ANALYZING JOB POSTINGS\n\n**CRITICAL: Keep responses SHORT. User viewing on small mobile screen.**\n**CRITICAL: Output PLAIN TEXT ONLY. NO markdown formatting. NO code blocks. NO backticks.**\n**CRITICAL: DO NOT add explanatory notes, disclaimers, or additional commentary after your analysis.**\n**CRITICAL: STOP after the RECOMMENDATION line. DO NOT add \"NOTE:\" or any extra paragraphs.**\n\nThe job posting text will be provided below. Analyze it and respond with ONLY this format (use emoji, keep concise):\n\n\n\n\nOUTPUT FORMAT:\n🎯 Role: [Job Title]\n📋 Company: [Company Name]\n💰 Salary: [Salary Range or \"Not Listed\"]\n🛠 Skills: [List 3-5 relevant skills from posting]\nSummary:\n\n FIT SCORE: [X/10]\n✅ Match: [2-3 brief reasons why good fit, one line each]\n⚠️ Gaps: [1-2 brief concerns, one line each]\n Enjoyment: [High/Medium/Low] - [One sentence explaining why]\n\n RECOMMENDATION: [Apply/Maybe/Skip] - [One sentence reasoning]\n\n**RULES:**\n1. Use emoji at start of each section as shown above\n2. Keep each bullet point to ONE line maximum\n3. Be direct and honest about fit\n4. Focus on skill match first\n5. Consider work environment preferences\n6. Factor in location and remote work preference\n7. Total response should be readable on mobile in one glance\n8. Your final line must be the RECOMMENDATION line - nothing after it\n9. Do not add notes, disclaimers, summaries, or extra commentary\n10. Do not explain the scoring system or methodology\n\n**FIT SCORE FACTORS (calculate by adding points):**\n{{ $json.fitScoreFactors }}\n\n**SCORING GUIDE:**\n- 8-10/10 = Strong match, apply immediately\n- 6-7/10 = Good match, worth applying\n- 4-5/10 = Possible fit, apply if interested\n- 1-3/10 = Poor match, skip unless desperate\n\n**ENJOYMENT FACTORS:**\n{{ $json.enjoymentFactors }}\n\n---\n\n## JOB POSTING TO ANALYZE:\n\n{{ $json.inputText }}"
            }
          ]
        },
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.ollama",
      "typeVersion": 1,
      "position": [
        -416,
        32
      ],
      "id": "5f4f8a29-3b38-4d95-b2c9-2538f3575e3f",
      "name": "AI Analysis"
    },
    {
      "parameters": {
        "jsCode": "// Clean up AI response - remove markdown formatting\nconst content = $input.item.json.content || '';\n\n// Remove code block markers\nlet cleaned = content\n  .replace(/```[a-z]*\\n?/gi, '')  // Remove ```language\n  .replace(/```/g, '')             // Remove closing ```\n  .replace(/`([^`]+)`/g, '$1')     // Remove inline backticks\n  .trim();\n\nreturn {\n  json: {\n    cleanedContent: cleaned\n  }\n};"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -144,
        32
      ],
      "id": "e28d17e6-2cb4-4672-9603-0813d421189b",
      "name": "Clean Response"
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "loose",
            "version": 1
          },
          "conditions": [
            {
              "id": "recommendation-check",
              "leftValue": "={{ $json.cleanedContent }}",
              "rightValue": "RECOMMENDATION: Skip",
              "operator": {
                "type": "string",
                "operation": "contains",
                "singleValue": true
              }
            }
          ],
          "combinator": "and"
        },
        "options": {
          "looseTypeValidation": true
        }
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [
        64,
        32
      ],
      "id": "f0a7bba4-08c5-4624-8a01-d38cd16304b1",
      "name": "Check Recommendation"
    },
    {
      "parameters": {
        "chatId": "YOUR_TELEGRAM_CHAT_ID",
        "text": "={{ $('Webhook').item.json.body.title }}\n{{ $json.cleanedContent }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        240,
        144
      ],
      "id": "d9994740-e86d-4bec-b069-f64de2e06e41",
      "name": "Send to Telegram - Apply Jobs"
    },
    {
      "parameters": {
        "chatId": "YOUR_TELEGRAM_CHAT_ID_FOR_SKIPS",
        "text": "={{ $('Webhook').item.json.body.title }}\n{{ $json.cleanedContent }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        240,
        -96
      ],
      "id": "750a6ec4-ddd4-4784-8e7f-6db279edd3c1",
      "name": "Send to Telegram - Rejected Jobs"
    }
  ],
  "pinData": {},
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "User Config",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "User Config": {
      "main": [
        [
          {
            "node": "Parse Webhook Jobs",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Parse Webhook Jobs": {
      "main": [
        [
          {
            "node": "AI Analysis",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Analysis": {
      "main": [
        [
          {
            "node": "Clean Response",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Clean Response": {
      "main": [
        [
          {
            "node": "Check Recommendation",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Check Recommendation": {
      "main": [
        [
          {
            "node": "Send to Telegram - Rejected Jobs",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Send to Telegram - Apply Jobs",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  },
  "tags": []
}
```


* * *

#### Congratulations - I Hope It Works

Playwright is rendering JavaScript, Ollama is scoring opportunities, and Telegram is queued up to buzz your phone.

None of the nonsense that clogs everyone's inbox.

Your system might be on fire, but hopefully it's monitoring, analyzing, and filtering.

Your infrastructure, your models, your rules. No subscriptions, no API limits, no external dependencies that disappear.

You earned them by creating and using this. Well done.


##### Correctly Customize the Constraints

Now you have actual matches that align with you. 

If your matches dont align, you may have to ask an LLM to customize your `User Config` section.

I asked an LLM to review this website and post what I thought the answers to the `User Config` section should be:

<details>
  <summary>answers</summary>
data:image/png;base64,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
</details>

Sorry about the massive queue of opportunities infront of you now.


* * *

## BONUS

I also wanted to include a simple message and reply - 1 shot.

A simple bot to demonstrate quick replies to simple answers. 

If you need a persistant memory to a chatbot you can access over a messenging platform, you can use [Clawdbot](https://github.com/moltbot/moltbot).


* * *

### Telegram LLM Bot - 1 shot reply

Instructions are included in the n8n workflow.

- Once you publish it, it runs non-stop in a loop
- Checking for new messages every 5 seconds
- Messages are marked as read after the AI generates a response

Good luck & have fun!

```json
{
  "name": "Telegram Bot - Start on Publish - Check Messages Every 5 sec",
  "nodes": [
    {
      "parameters": {
        "url": "=https://api.telegram.org/bot{{ $json.bot_token }}/getUpdates",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "offset",
              "value": "={{ $json.offset }}"
            },
            {
              "name": "timeout",
              "value": 30
            }
          ]
        },
        "options": {}
      },
      "id": "3e1be194-584b-498d-9158-8d4c1dc08e58",
      "name": "2. Get Updates",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        -160,
        144
      ],
      "continueOnFail": true
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict"
          },
          "conditions": [
            {
              "id": "check-msg",
              "leftValue": "={{ $json.result.length }}",
              "rightValue": 0,
              "operator": {
                "type": "number",
                "operation": "gt"
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "id": "02badb1e-4cab-4f4a-8ff6-c27b6451aa13",
      "name": "3. Has Message?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2,
      "position": [
        0,
        112
      ]
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "={{ $json.message.text }}",
        "options": {
          "systemMessage": "You are a helpful AI assistant."
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.8,
      "position": [
        336,
        96
      ],
      "id": "f30be620-2a41-4638-8b82-8f374d62328c",
      "name": "5. Send to LLM"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "new-offset",
              "name": "offset",
              "value": "={{ $('6. Prepare Response + Message Offset').item.json.offset }}",
              "type": "number"
            },
            {
              "id": "keep-token",
              "name": "bot_token",
              "value": "={{ $('1. Set Telegram Token').item.json.bot_token }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "id": "545e61c0-7c15-4c86-8ae0-4ea9e3fcd86d",
      "name": "9. Update Offset Var",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        1152,
        96
      ]
    },
    {
      "parameters": {
        "model": "qwen2.5-coder:7b",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOllama",
      "typeVersion": 1,
      "position": [
        336,
        288
      ],
      "id": "1118be09-8d64-4999-a13b-11ef1cf346a8",
      "name": "Ollama Chat Model",
      "credentials": {
        "ollamaApi": {
          "id": "YOUR_OLLAMA_API_ID_HERE",
          "name": "Ollama account"
        }
      }
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "token",
              "name": "bot_token",
              "value": "YOUR_BOT_TOKEN_HERE",
              "type": "string"
            },
            {
              "id": "offset",
              "name": "offset",
              "value": "={{ $json.offset || 0 }}",
              "type": "number"
            }
          ]
        },
        "options": {}
      },
      "id": "749e5b4a-b469-4a42-82b3-637e7483b67e",
      "name": "1. Set Telegram Token",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        -320,
        192
      ]
    },
    {
      "parameters": {
        "jsCode": "// Extract the first message update\nreturn [{ json: items[0].json.result[0] }];"
      },
      "id": "b06f8f1a-6365-4554-a0ee-05da11c64605",
      "name": "4a. Extract Message",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        192,
        96
      ]
    },
    {
      "parameters": {
        "url": "=https://api.telegram.org/bot{{ $('1. Set Telegram Token').item.json.bot_token }}/getUpdates",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "offset",
              "value": "={{ $('6. Prepare Response + Message Offset').item.json.offset }}"
            }
          ]
        },
        "options": {}
      },
      "id": "2362455a-0717-4d0f-bfca-c58d43af2c9d",
      "name": "8. Confirm Read Telegram Message",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        976,
        96
      ]
    },
    {
      "parameters": {
        "url": "=https://api.telegram.org/bot{{ $('1. Set Telegram Token').item.json.bot_token }}/sendMessage",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "chat_id",
              "value": "={{ $json.chat_id }}"
            },
            {
              "name": "text",
              "value": "={{ $json.llm_response }}"
            }
          ]
        },
        "options": {}
      },
      "id": "c3cfff2f-b4e7-4653-bf69-98332b8edfe3",
      "name": "7. Send AI Reply",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        816,
        96
      ]
    },
    {
      "parameters": {
        "jsCode": "// Get the AI response\nconst llmOutput = $input.first().json.output;\n\n// Get the original message data from Node 4\nconst messageNode = $('4a. Extract Message').first().json;\nconst chatId = messageNode.message.chat.id;\n\n// Prepare clean data for the API call\nreturn [{\n  json: {\n    chat_id: chatId,\n    llm_response: typeof llmOutput === 'object' ? llmOutput.text : llmOutput,\n    offset: $('4a. Extract Message').first().json.update_id + 1,\n    bot_token: $('1. Set Telegram Token').first().json.bot_token\n  }\n}];"
      },
      "id": "7c1de421-e5c9-499c-a47f-4ff7993d038f",
      "name": "6. Prepare Response + Message Offset",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        608,
        96
      ]
    },
    {
      "parameters": {
        "events": [
          "update",
          "activate",
          "init"
        ]
      },
      "type": "n8n-nodes-base.n8nTrigger",
      "typeVersion": 1,
      "position": [
        -432,
        64
      ],
      "id": "aa7e8bc4-3b9d-4420-8a6c-976d793d9617",
      "name": "n8n Trigger"
    },
    {
      "parameters": {},
      "id": "6882277a-8e1a-49ac-885a-69f7b4486ac8",
      "name": "11. Wait 5 Seconds",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        1504,
        304
      ],
      "webhookId": "1809e752-3ac4-426e-bd4e-7e59fa42ed1e"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        144,
        272
      ],
      "id": "1fc685d0-542a-4286-b862-b86af8d1d569",
      "name": "4b. Wait 5 seconds before checking for messages",
      "webhookId": "dd7f73e2-c049-40cb-b7ff-014508bcdb44"
    },
    {
      "parameters": {
        "content": "##    📨 Telegram LLM Bot 🤖\n\n### 📋 Setup Instructions:\n\n1. **Get Bot Token:**\n   - Chat with @BotFather on Telegram\n   - Send `/newbot` and follow instructions\n   - Copy your bot token\n\n2. **Configure Workflow:**\n   - Open \"Set Bot Token\" node\n   - Replace `YOUR_BOT_TOKEN_HERE`\n\n3. **Configure LLM:**\n   - Default: Ollama (qwen2.5-coder:7b)\n   - Or swap to OpenAI/Anthropic/etc\n\n4. **Activate:**\n   - Click \"Activate\" button\n   - Send message to your bot\n   - Get AI response!\n\n\n* * *\n\n### 🔧 Technical Details:\n\n**Offset Management:**\n- Telegram stores updates for 24h\n- We confirm processed updates\n- Prevents re-processing on restart\n\n**Long Polling:**\n- 30s timeout per request\n- Efficient, no missed messages\n- Better than webhooks for simple bots\n\n### ✅ Proper Offset Management\nThis workflow correctly handles Telegram updates to prevent duplicate processing.\n\n**How it works:**\n1. **Poll** - Gets new updates every 5s\n2. **Confirm** - Marks updates as read on Telegram's server\n3. **Process** - Handles each message\n4. **Respond** - Sends LLM reply back\n\n**Key Features:**\n✓ No duplicate messages\n✓ Production-ready\n✓ Efficient polling\n✓ Proper error handling\n\n\n```\nFetch Updates\n   ↓\n   Has messages?\n   ├─ NO → Wait 5s → Loop back\n   └─ YES → Process → LLM → Reply → Loop back\n```",
        "height": 1280,
        "width": 320,
        "color": 4
      },
      "type": "n8n-nodes-base.stickyNote",
      "typeVersion": 1,
      "position": [
        -768,
        -64
      ],
      "id": "32d14f25-46c0-4299-9c02-732496aa3d4c",
      "name": "Sticky Note"
    }
  ],
  "pinData": {},
  "connections": {
    "2. Get Updates": {
      "main": [
        [
          {
            "node": "3. Has Message?",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "3. Has Message?": {
      "main": [
        [
          {
            "node": "4a. Extract Message",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "4b. Wait 5 seconds before checking for messages",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "5. Send to LLM": {
      "main": [
        [
          {
            "node": "6. Prepare Response + Message Offset",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "9. Update Offset Var": {
      "main": [
        [
          {
            "node": "11. Wait 5 Seconds",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Ollama Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "5. Send to LLM",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "1. Set Telegram Token": {
      "main": [
        [
          {
            "node": "2. Get Updates",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "4a. Extract Message": {
      "main": [
        [
          {
            "node": "5. Send to LLM",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "8. Confirm Read Telegram Message": {
      "main": [
        [
          {
            "node": "9. Update Offset Var",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "7. Send AI Reply": {
      "main": [
        [
          {
            "node": "8. Confirm Read Telegram Message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "6. Prepare Response + Message Offset": {
      "main": [
        [
          {
            "node": "7. Send AI Reply",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "n8n Trigger": {
      "main": [
        [
          {
            "node": "1. Set Telegram Token",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "11. Wait 5 Seconds": {
      "main": [
        [
          {
            "node": "1. Set Telegram Token",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "4b. Wait 5 seconds before checking for messages": {
      "main": [
        [
          {
            "node": "1. Set Telegram Token",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": true,
  "settings": {
    "executionOrder": "v1",
    "availableInMCP": false
  },
  "versionId": "c242df8b-1c39-420a-b861-68a47e62e51c",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "5b190c49174778d107cc5d364d3fb35d408aac3422ee5fa91a0e094276d7388f"
  },
  "id": "C3g1W07hyg1IWM6A",
  "tags": []
}
```
