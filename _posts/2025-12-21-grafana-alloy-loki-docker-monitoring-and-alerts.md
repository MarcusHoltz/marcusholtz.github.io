---
layout: post
title: Grafana, Alloy, Loki using Docker alerting on our favorite song
description: A quick guide to Grafana, Alloy, Loki using Docker
date: 2025-12-21 11:33:00 -0700
categories: [DevOps, Monitoring]
tags: [cloud, monitoring, homelab, security, customization, logs, mirror, guide, applications, docker, traefik, fun]
pin: false
image:
  path: /assets/img/header/header--grafana--alloy-loki-stack-docker-homelab-monitoring.jpg
  alt: Grafana-alloy-loki-on-docker-for-homelab-monitoring-explained
---


# Homelab Guide to Monitoring Docker Logs and Log Files

When you're running containerized applications, you need to understand what's happening inside your stack. 

This article was written to help you get better understanding of logging and observability.

If you want to try this out yourself on system prepared for you, check out the free labs available for Grafana at [killercoda.com](https://killercoda.com/het-tanis/course/Linux-Labs/102-monitoring-linux-logs).


* * *

## Homelab Monitoring Stack: Alloy, Loki, and Grafana

This article uses three tools for monitoring:

- `Grafana Alloy` for log collection
- `Loki` for log storage
- `Grafana` for visualization

By the end of this guide, you'll understand what each part does.

**BONUS**: `Traefik` is also used in this stack.

* * *

### What Is Alloy?

Alloy is Grafana's modern log collector. It is [replacing Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/), for [Alloy's OTEL support](https://grafana.com/oss/alloy-opentelemetry-collector/).

Think of it as building pipelines with three stages:

1. **Pull Data** - Discover and collect logs
2. **Process Data** - Parse, label, and transform
3. **Send Data** - Forward to destinations

**Important Syntax Gotchas:**
- Alloy uses HCL (HashiCorp Configuration Language)
- Comments use `//` not `#`
- Commas, even after the last item in the list


* * *

### What Is Loki?

[Loki](https://grafana.com/docs/enterprise-logs/latest/get-started/overview/) is Grafana's log aggregation system. It's designed to be a way to store your logs, cost-effective and easy to operate.

Unlike traditional log systems, Loki doesn't index log content. Instead:

1. **Index Labels Only** - Stores metadata like service name, host, job
2. **Compress Logs** - Keeps raw log lines compressed

**Key Design Principles:**
- Only label dimensions you'll query by (service, environment, host)
- Don't add high-cardinality labels (user IDs, request IDs, timestamps)
- Use LogQL to filter log content at query time
- Logs are stored in chunks and kept in object storage

**Storage Modes:**
- `filesystem` - Simple, good for single-node setups
- `s3` or `gcs` - Production-ready, scalable storage


* * *

### What Is Grafana?

Grafana is your [visualization dashboard](https://docs.huihoo.com/grafana/img/animated_gifs/drag_drop.gif). It queries Loki (and other data sources) to display logs, metrics, and handles alerts all in one place.

Key concepts:

1. **Data Sources** - Connect to Loki, Prometheus, etc.
2. **Dashboards** - Visual panels showing your data


* * *

### The Complete Alloy, Loki, and Grafana Stack

1. **Container logs to Docker** → Docker daemon captures them
2. **Alloy discovers containers** → Via Docker socket
3. **Alloy applies labels** → Container name, stream, etc.
4. **Alloy tails Traefik logs** → Parses JSON, adds labels
5. **Alloy sends to Loki** → HTTP push to port 3100
6. **Loki indexes and stores** → Labels + compressed chunks
7. **Grafana queries Loki** → LogQL queries via provisioned datasource
8. **Compactor manages retention** → Deletes logs older than 30 days


* * *

## Part 0: Setup the Lab Enviornment

Be sure you have copied the required files from the github repo.

Before we can spin up docker, there are things we need to do:

- Run: `1_permissions_init_for_project.sh`

- Edit: `.env`

- Edit: `./traefik/traefik.yml`

- Review: `docker network`


* * *

### 1_permissions_init_for_project.sh script

For ease of use, just run the `sudo bash 1_permissions_init_for_project.sh` script that will set permissions for you.

There are two permissions that need set, as we'll be using files outside of the container - on the host.

- `acme.json` needs `600` in the `./traefik` directory

- `data` directory must be owned by UID `10001` in the `./loki` directory


* * *

### .env file

The .env file stores all of the variables we will use throughout. Please review and make changes (instructions for Telegram are included when we get there).

The sections include:

- `Docker Compose information` for `DOMAIN` and `Let's Encrypt`
- `Cloudflare API` for Let's Encrypt DNS-01 certificates
- Your `Grafana Cloud` information
- The location of your `Music` for generating logs
- `Telegram` for Grafana alerts needs the `token` and `chatid`


* * *


### `traefik.yml` in the traefik directory

This file is our static traefik config. It should never need updating, except for this one thing.

- At the bottom of the file, under the "Certificate Resolvers", you will find `email:` and will have `"your_email_address@some_email_provider.com"`
- Change the `"your_email_address@some_email_provider.com"` to an email address you have access to (this is for LE renewals)


* * *

### Docker Network

So this write-up uses an external docker network. That network is a macvlan that allows each container to get their own address on the network connected to the parent interface (eth0), with the name of the bridged docker network being (br1.232).

`docker network create -d macvlan --subnet=10.236.232.0/24 --gateway=10.236.232.254 -o parent=eth0 br1.232`

Please adjust the settings for the network used throughout for your own use.


* * *

### Starting Each Part 1-5

#### Working Folder Workflow

**The Core Idea**: You'll have a base folder (`0-Setup-Lab-Environment`) that stays untouched, and a working folder that gets deleted and recreated for each step.

##### Initial Setup (Do This Once)

1. Complete everything in your `0-Setup-Lab-Environment` folder:
   - Run `1_permissions_init_for_project.sh`
   - Edit `.env` with your details
   - Update the email in `./traefik/traefik.yml`
   
2. Create the Docker network (see the blog for the command)

##### For Each Lab Step

1. Create a new working folder
2. Copy everything from `0-Setup-Lab-Environment` into it
3. Add the files for your current step
4. Run `1_permissions_init_for_project.sh` again
5. Run `docker-compose up`

##### Moving to the Next Step

1. Delete your working folder completely
2. Repeat the process above with the new step's files

**Why?** This keeps your base configuration clean and gives you a fresh start for each step. You're always working with `0-Setup-Lab-Environment` + current step files, nothing more.

**Important**: Never edit `0-Setup-Lab-Environment` directly after initial setup. Always work in your temporary working folder.


* * *

## Part 1: Alloy Reads - Docker Socket Logs


Alloy is the first step, it allows us to take a log file, or a unix socket, read from it, and transform it, before sending it off for log ingestion and storage.


* * *

### 1). Alloy Access to Docker Socket

First, to get Alloy to read all the logs in docker, you must give it access to the docker socket.

* * *

You can add the Alloy container to the docker socket in your `docker-compose.yml` file using a volume mount. 

Here's how to modify your `docker-compose.yml`:

```yaml
alloy:
  image: grafana/alloy:latest
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
```

Don't forget to also mount the Docker socket (`/var/run/docker.sock`) as a **read-only** volume so Alloy can only access Docker's API to read container logs.


* * *

### 2). Get Alloy to Discover Docker Containers

Let's look at how reading docker logs works in our `config.alloy` file.

In the `config.alloy` file, we will start with container discovery. 

The Docker socket is a Unix socket that allows processes to communicate with the Docker daemon, and Alloy uses this to discover all running containers on your system. 

Here's the discovery block:

```ini
// Discover running Docker containers
discovery.docker "containers" {
    host = "unix:///var/run/docker.sock"
}
```

This connects to Docker and discovers all running containers. 

The exported field `discovery.docker.containers.targets` contains the list of discovered containers.

This is also only possible because of our `docker-compose.yml` file, where we mount the Docker socket into the Alloy container.

You can now view the list of all of your targets and their fields in Alloy:

Access it on the Alloy Web UI: `http://your-alloy-host:12345/component/discovery.docker.containers`

* * *

[Grafana Alloy Docker Discovery Documentation](https://grafana.com/docs/alloy/latest/reference/components/discovery/discovery.docker/)

* * * 


### 3). Use Alloy to Clean Up Labels

Discovering containers is only the first step. 

We need to transform the raw metadata that Docker provides into useful labels that we can query later. 

This is where the relabeling process comes in.

Grafana Alloy's `discovery.relabel` block takes the raw Docker metadata and creates structured labels. For example, Docker provides the container name as `__meta_docker_container_name` with a leading slash, like (`/traefik`). Let's fix that, it's easier to refer to contianers with the `/` removed:

```ini
discovery.relabel "containers" {
    targets = discovery.docker.containers.targets

    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/(.*)"
        target_label  = "container"
    }
}
```

Each `rule` block transforms labels, meaning, each rule transforms Docker's internal metadata into labels you can actually use when searching logs. The regex pattern `/(.*)` captures everything after the leading slash, it strips the leading slash, giving us clean container names. Then the exported field is `discovery.relabel.containers.output`.

You can now view the list of all of your *new* `target_labels` and their fields in Alloy (they will be at the bottom):

Access it: `http://your-alloy-host:12345/component/discovery.relabel.containers#Arguments-rule_0`


* * * 

[Grafana Alloy Discovery Relabeling](https://grafana.com/docs/alloy/latest/reference/components/discovery/discovery.relabel/)

* * * 


### 4). Finish Step 3

Now that you understand the concept, let's clean up the rest of those fields and make them all presentable.

```ini
// Add proper labels to discovered containers
discovery.relabel "containers" {
    targets = discovery.docker.containers.targets

// You should already have this rule in, add the ones under it
    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/(.*)"
        target_label  = "container"
    }
    rule {
        source_labels = ["__meta_docker_container_log_stream"]
        target_label  = "stream"
    }
    rule {
        source_labels = ["__meta_docker_container_id"]
        target_label  = "container_id"
    }
}
```

This block accepts the list of discovered Docker targets from the previous component (`discovery.docker.containers.targets`).

Docker setups with Promtail (now Alloy) often export the label, `container`, as it is a longstanding convention for the raw container name (e.g., "Plex"),

But, `service_name` is gaining traction for Kubernetes or multi-service setups.

You will see both `container` and `service_name` labels from `__meta_docker_container_name`. This is for maximum Grafana dashboard compatibility.

This ensures your logs arrive in Loki/Grafana with standard, readable tags like `container` and `container_id` rather than obscure internal variables.


* * *

#### Put a Filter in Your Rules (optional)

This is an example. If you had a container you didnt want to include in the logs, you can add a rule to drop specific containers.

```ini
rule {
  source_labels = ["__meta_docker_container_name"]
  regex         = "noisy_container_.*"
  action        = "drop"
}
```

* * *


### 5). Alloy Send Logs to Collector

Finally, the actual log collection happens through the `loki.source.docker` block. Alloy's configuration component takes the discovered targets and begins streaming their logs from the discovered containers to Loki. It acts as a bridge, reading lines as they are written and immediately pushing them to the `loki.write.local` component.

```ini
// Scrape logs from Docker containers - send to local
loki.source.docker "docker_logs" {
    host       = "unix:///var/run/docker.sock"
    targets    = discovery.relabel.containers.output
    forward_to = [loki.write.local.receiver]
}
```

Notice how the targets come from our relabeling output, `discovery.relabel.containers`. This means every log line will automatically have the labels we configured.


* * *

### 6). Alloy Sends to Local Loki

Let's tie together how logs actually flow from Alloy to Loki. 

We've seen how Alloy collects logs from Docker, but the final step is writing them to Loki for storage and querying.

The final destination for logs is Loki and the `loki.write` component, it points to Loki's endpoint address. This creates an input point (specifically `loki.write.local.receiver`) that other components in the configuration can forward their log data to.

```ini
loki.write "local" {
    endpoint {
        url = "http://loki:3100/loki/api/v1/push"
    }
}
```


* * *

[Grafana Alloy has documentation on using Docker as a Loki Source](https://grafana.com/docs/alloy/latest/reference/components/loki/loki.source.docker/)

* * *


* * * 

## Part 2: Alloy Reads - Files (Traefik Access Logs)


Beyond container stdout and stderr logs, we often need to collect structured application logs from files. In this example, we're using Traefik as a reverse proxy.

Traefik will write access logs in JSON format to a file. This gives us rich information about every HTTP request hitting our infrastructure.


* * *


### 0). Quick Reminder: Working Folder Setup

1. Delete your current working folder
2. Create a new working folder
3. Copy `0-Setup-Lab-Environment` into it
4. Add `2-Alloy-Reads-Files-Traefik-Access-Logs` files
5. Run `1_permissions_init_for_project.sh`
6. Run `docker-compose up -d`

When moving to the next step: delete the working folder and repeat this process.

* * *

### 1). Docker - Traefik Mount for Access Logs

In the `docker-compose.yml` file, there needs to be a section to tell Traefik to export logs to the host so we can read them outside of the container.

You will find `"./traefik/access-logs:/opt/access-logs"` in your `docker-compose.yml` file that send our logs to our current directory under `traefik` and inside of `access-logs`.

```yaml
  traefik:
    image: traefik:latest
    container_name: traefik
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./traefik/access-logs:/opt/access-logs"
```


* * *


### 2). Traefik Config - Access Logs Type and Location

In your static `traefik.yml` config, you will need to be sure you're passing the same path, and telling what kind of log format you need.


```yaml
################################################################
# Access Logging
################################################################
accessLog:
  filePath: "/opt/access-logs/access.json"
  format: json
  fields:
    defaultMode: keep
    headers:
      defaultMode: keep
      names:
        User-Agent: keep
        Referer: keep
        Forwarded: keep
```


* * *

### 3). Alloy Docker Volume to Access Traefik Logs in Alloy

This snippet from the `docker-compose.yml` file creates a target pointing to the access log file. 

In the `docker-compose.yml`, mount the Traefik access logs directory into the Alloy container so it can read these files:

```yaml
alloy:
  image: grafana/alloy:latest
  volumes:
  - "./traefik/access-logs:/var/log:ro"
```


* * *

### 4). Access the Access Logs in Alloy

In the `config.alloy` file, the process of reading file-based logs is slightly different from reading Docker logs. 

First, we need to tell Alloy where to find the log files. The `local.file_match` component generates a target for our file discovery:

```ini
local.file_match "traefik_access_logs" {
    path_targets = [{
        __path__ = "/var/log/access.json",
    }]
}
```

This creates a target, `traefik_access_logs`, pointing to Traefik's JSON access log file that we mounted as a volume from our host to our Alloy container.

* * *

[Grafana Alloy local file documentation](https://grafana.com/docs/alloy/latest/reference/components/local/local.file_match/)

* * *


### 5). Read the File In

Once Alloy knows where the file is, the `loki.source.file` component can begin tailing it, similar to how the `tail -f` command works:

```ini
loki.source.file "traefik_access" {
    targets    = local.file_match.traefik_access_logs.targets
    forward_to = [loki.process.traefik_labels.receiver]
}
```

Our new Loki file source, `traefik_access`, tails the file and forwards new lines to `traefik_labels` for processing, the next stage in the pipeline - `loki.process` acts as a middleware layer that modifies the log metadata before storage.

Notice that instead of forwarding directly to Loki, we're forwarding to that `loki.process` processing stage first. This is because Traefik's JSON logs contain structured data that we want to extract into labels for Loki.

* * *

[Grafana Alloy Loki Source File Documentation](https://grafana.com/docs/alloy/latest/reference/components/loki/loki.source.file/)

* * *


### 6). Parse JSON and Add Labels

The `loki.process` block below parses the JSON and creates labels. Traefik writes fields like ClientHost, RequestMethod, and DownstreamStatus in its JSON logs, and we want these as queryable labels in Loki:

```ini
// Add labels to Traefik access logs - send raw JSON to Loki
loki.process "traefik_labels" {
    forward_to = [loki.write.local.receiver]

// Drop the filename label (not needed with single file)
stage.label_drop {
    values = ["filename"]
}

// Add a static label so dashboard queries work
    stage.static_labels {
        values = {
            host     = "localhost",
            job      = "traefik",
            log_type = "access",
        }
    }
}

```

In the block above,

- We are forwarding `traefik_labels` - which was first assigned from our local file match, `traefik_access_logs`, with output delivered to us from `traefik_access`, then forwarded onto the `loki.process`, `traefik_labels`, above.

- `stage.label_drop` Removes the `filename` label to reduce index cardinality.

- `stage.static_labels` adds static labels to identify these as Traefik access logs. 


* * *

- [local.file_match](https://grafana.com/docs/alloy/latest/reference/components/local/local.file_match/)

- [loki.source.file](https://grafana.com/docs/alloy/latest/reference/components/loki/loki.source.file/)

- [loki.process](https://grafana.com/docs/alloy/latest/reference/components/loki/loki.process/)

* * *


* * *

### You now have file logs and socket logs

One detail to note: the forward configuration, `forward_to`, appears in multiple places because we have multiple sources. 

Both Docker logs and Traefik logs ultimately forward to `loki.write.local.receiver`. 

This receiver has `local` in it because you can change where logs go by modifying just the `loki.write` block, and all your sources automatically use the new destination.


* * *

## Part 3: Sending Logs to Grafana Cloud

You can send your data to the cloud for safe-backup, sure.

But the real reason, is Grafana's AI integration to the data. It can craft outliers and alerts with a single prompt.

Want to send data to Grafana Cloud? Alloy makes this easy through its forwarding configuration.

In the `config.alloy` file, you'll see commented-out blocks for Grafana Cloud. The configuration file was designed so you can enable or disable cloud logging simply by uncommenting those specific lines. 

Basic Authentication (User ID + API Key) is also required in the `.env` file.


* * *

If you've never used Grafana Cloud, you need an account. 

And probably a quick introduction, this video shows how to import local to cloud:

[Grafana Learning Journeys: How to Send Logs to Grafana Cloud Using Alloy](https://youtu.be/Xa3mCIdsno4?t=96)

* * *

> In this setup, you'll be sending **ALL** of your logs to the Grafana Cloud. Please be aware, all logs for all containers and all logs being read from all files.


* * *

### Step 0. Setup Reminder

Dont forget to delete your current working folder, and copy everything again!

```
Create working folder → Copy 0-Setup-Lab-Environment → Add step files → Run permissions script → Run docker-compose
```

(Delete working folder before moving to next step)

* * *

### Step 1. Create a Grafana Cloud account

You must have an account to use Grafana Cloud, go ahead and sign up now.

- [https://grafana.com/auth/sign-up/create-user](https://grafana.com/auth/sign-up/create-user)


* * *

### Step 2. After account creation - find details here

You need to create and org/Grafana Cloud stack

You can create a token if you go to this page:

- [https://<your_assigned_url>.grafana.net/a/grafana-collector-app/alloy/installation](https://<your_assigned_url>.grafana.net/a/grafana-collector-app/alloy/installation)

- Once you create a token, you will see an "Install and run Grafana Alloy" section.

- This has all of your `env_var` in place you need. Copy that and go to the `.env` file. Paste and replace.


* * *

### Step 3. Add your details assigned to the .env file

You need to enter your username and password for Alloy to be able to export to the cloud.

Again, if using the Grafana Cloud, you will need to uncomment the lines required.

Username and password are stored in an .env file so you never commit those.

```alloy
// loki.write "grafana_cloud" {
//     endpoint {
//         url = env("GCLOUD_HOSTED_LOGS_URL")
//         basic_auth {
//             username = env("GCLOUD_HOSTED_LOGS_ID")
//             password = env("GCLOUD_RW_API_KEY")
//         }
//     }
// }
```

### Step 4. Uncommenting the cloud endpoint isnt enough! 

You must also add the destination for your `forward_to` lists:

To finish enabling cloud logging, you would uncomment the `grafana_cloud` block above and then add its receiver to your forwarding configurations. 

For example, the Docker logs would be updated from:

```ini
forward_to = [
    loki.write.local.receiver,
]
```

To include both destinations:

```ini
forward_to = [
    loki.write.grafana_cloud.receiver,  # Cloud
    loki.write.local.receiver,          # Local
]
```

This means Alloy sends every log line to multiple destinations simultaneously, **this also goes for your Traefik access logs**.

* * *

## Part 4: Sending Grafana Cloud Metrics

Beyond logs, Alloy can also collect and forward metrics. In our configuration, we're specifically collecting Alloy's own metrics so we can monitor the health of our logging pipeline itself. This self-monitoring is crucial because if your logging system fails, you need to know about it!

(You can use your current,`3-Sending-Data-to-Grafana-Cloud`, working folder)


* * *

### 1). Alloy Exporter for Prometheus

The metrics collection starts with the self-monitoring exporter:

```ini
prometheus.exporter.self "alloy" { }
```

This component exposes **only** Alloy's internal metrics in Prometheus format.


* * *

### 2). Alloy Prometheus Scraper

Next, we configure a scraper on ourself that periodically collects data  metrics from ourself (uncomment to enable):

```ini
alloyprometheus.scrape "alloy" {
    targets         = prometheus.exporter.self.alloy.targets
    scrape_interval = "60s"
    forward_to      = [
        // prometheus.remote_write.grafana_cloud.receiver,
        // prometheus.remote_write.local.receiver,
    ]
}
```

The `scrape_interval` of sixty seconds means Alloy checks its own metrics every minute. 


* * *

### 3). Alloy prom grafana cloud endpoint

In the `config.alloy` file, you'll see commented-out blocks for Grafana Cloud prometheus server. The configuration is designed so you can enable or disable cloud logging simply by uncommenting specific lines. Here's the Grafana Cloud Prometheus endpoint configuration:

You need to enter your username and password.

Username and password are stored in an .env file so you never commit those.


```ini
// prometheus.remote_write "grafana_cloud" {
//     endpoint {
//         url = env("GCLOUD_HOSTED_METRICS_URL")
//         basic_auth {
//             username = env("GCLOUD_HOSTED_METRICS_ID")
//             password = env("GCLOUD_RW_API_KEY")
//         }
//     }
// }
```

Notice the URL is different from the Loki endpoint. 

Grafana Cloud separates logs and metrics into different cloud hosted service enpoint urls, each optimized for its data type. 

The username here is your hosted metrics ID, which is different from your hosted logs ID.


* * *

### 4). Alloy prom local endpoint

If you're running your own Prometheus instance locally, you can uncomment and configure the local endpoint:

```ini
// prometheus.remote_write "local" {
//     endpoint {
//         url = "http://prometheus:9090/api/v1/write"
//     }
// }
```


* * *

### 5). Alloy Sends to Local Loki

Now that we're out of the cloud, let's tie together - how logs actually flow from Alloy to Loki, one more time.

We've seen how Alloy collects logs from Docker and files, but the final step is writing them to Loki for storage and querying.

The final destination for logs is Loki and the `loki.write` component, it points to Loki's endpoint address.

```ini
loki.write "local" {
    endpoint {
        url = "http://loki:3100/loki/api/v1/push"
    }
}
```

### 6). Loki Collection Endpoint - IP or DNS

This is optional choice is configured in the docker-compose file.

You can set the IP address to match to a static IP assigned in the `docker-compose.yml`, like the config above, let the internal Docker DNS hostname resolution handle it. **Make sure you have loki as the container name, or use a static IP**

```yaml
loki:
  image: grafana/loki:latest
  container_name: loki
  ports:
    - "3100:3100"
  networks:
    br1.232:
      ipv4_address: 10.236.232.146
```

- Using a static IP on a Docker network makes the configuration more predictable. You could also use the container name `loki` instead of the IP.

- Port 3100 inside the container to port 3100 on the Docker network. 

- When Alloy sends logs to Loki, it's not just sending raw text. Remember all those labels we created during discovery and processing? Alloy bundles those labels with each log line, and Loki indexes them.

* * *

[Grafana Alloy Loki Write endpoint documentation](https://grafana.com/docs/alloy/latest/reference/components/loki/loki.write/)

* * *


* * *

## Part 5: Loki the endpoind

Loki also goes by the collector, the compressor, the reciever, the accepter, the listener, etc. It has a lot of things it does, but is still very easy to use. 

Loki takes logs.

(You can use your current working folder, all deployments have the same loki configuration)


* * *

### 1). How Loki Sets Where It's Listening

Moving to the Loki side of our stack, we need to configure where Loki accepts incoming logs. This is defined in the `loki-config.yaml` file under the server section:

```yaml
server:
  http_listen_port: 3100
  grpc_listen_port: 9096
  http_server_read_timeout: 5s
  http_server_write_timeout: 10s
  log_level: info
```

- The `http_listen_port` of `3100` is the standard Loki port and matches what we configured in Alloy. Loki actually exposes multiple endpoints on this port. The `/loki/api/v1/push` endpoint that Alloy uses is for writing logs, while `/loki/api/v1/query` and `/loki/api/v1/query_range` are for reading logs (which Grafana uses).

- The `grpc_listen_port` on port `9096` and is used forfor internal gRPC communication, which some clients use instead of HTTP. In our simple setup, we're not using it, but it's available if needed.

- The `http_server_read_timeout` limits how long Loki waits to receive the complete request, protecting against slow clients. 

- The `http_server_write_timeout` limits how long Loki spends sending a response, preventing queries that take too long from tying up resources.


* * *

[Loki Server Configuration Documentation](https://www.google.com/search?q=https://grafana.com/docs/loki/latest/configure/server/)

* * *


### 2). Loki Docker Compose Volumes

Loki's storage configuration determines everything from where log data lives to how it's organized and accessed.

In this section we'll look at file storage for loki logs.


#### Configuring Loki Data on the Host

Loki has to store this data somewhere, if you dump it all in docker volume  - I will cry.

Please dont make me cry, please export this data for the host system to have available.

* * *

In our `docker-compose.yml`, we **mount a host directory** to persist this data, but we need to amke sure to keep permissions correct:

```yaml
volumes:
  - "./loki/data:/loki"
  - "./loki/config:/etc/loki"
```

This means all the log data written to `/loki` inside the container is actually stored in `./loki/data` on your host machine. If the Loki container restarts, your logs are safe because they're outside the container filesystem.


* * *

#### Permissions for Loki Data on the Host

Exporting the data above requires correct permissions.

They must be **set for the container**, now that these files reside on the host.

The `1_permissions_init_for_project.sh` script creates and sets proper permissions for these directories:

```bash
LOKI_DATA_DIR="./loki/data"
LOKI_UID=10001
LOKI_GID=10001

mkdir -p "$LOKI_DATA_DIR/rules"
mkdir -p "$LOKI_DATA_DIR/chunks"
mkdir -p "$LOKI_DATA_DIR/wal"

chown -R "$LOKI_UID:$LOKI_GID" "$LOKI_DATA_DIR"
```

Loki runs as user ID `10001` inside the container, so these directories must be owned by that user ID. The script creates the necessary subdirectories and sets ownership before starting Loki for the first time.


* * *

### 3). Loki Log File Storage Location

In our `loki-config.yaml`, the storage configuration uses the filesystem mode:

```yaml
common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
```

- This defines the storage path and tells Loki to store data on the local disk (`filesystem`) inside `/loki/chunks`.

- The `path_prefix` sets the base directory for all Loki data. Within this, we have separate directories for different types of data. 

- The `chunks_directory` is where actual log data gets written. Loki compresses logs into chunks, which are immutable blocks of data that can be efficiently stored and queried.

- `replication_factor: 1` is set because you are running a single instance. It won't try to copy data to other nodes.


* * *

### 4). Loki Log Storage Style

The storage schema configuration tells Loki how to organize this data:

```yaml
schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h
```

This configuration is particularly important. 

- The `store: tsdb` means Loki uses its Time Series Database index format, which is optimized for time-based queries. 

- The `period: 24h` means Loki creates a new index file every twenty-four hours. This makes it efficient to drop old data and keeps index files at manageable sizes.


* * *

[Lokie Storage Configuration Documentation](https://grafana.com/docs/loki/latest/configure/storage/)

* * *



* * *

### 5). How Loki Keeps Logs

Back to the config file.

Log retention is a balance between storage costs and regulatory or operational requirements.

The retention configuration starts with the compactor:

```yaml
compactor:
  working_directory: /loki/compactor
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
  retention_delete_worker_count: 150
  delete_request_store: filesystem
```

The compactor runs every ten minutes (controlled by `compaction_interval`) and performs two key tasks: 
- It merges small chunks into larger ones for efficiency.
- It deletes old data based on retention rules. 

Just incase there is a problem the `retention_delete_delay` of two hours provides a safety buffer.
- When the compactor marks data for deletion, it waits two hours before actually removing it, giving you time to recover if you realize you need that data.


* * *

### 6). How Long Loki Keeps Logs

The actual retention period is set in the limits configuration:

```yaml
limits_config:
  retention_period: 720h  # 30 days
  reject_old_samples: true
  reject_old_samples_max_age: 168h
```

The `retention_period` of 720 hours means thirty days. 

**Any logs older than this will be deleted during compaction.**

The `reject_old_samples` setting prevents clients from writing logs with timestamps older than `reject_old_samples_max_age` (one week). 

**This protects against scenarios where a log collector has been offline and tries to push a huge backlog of old logs all at once.**

- Logs older than 30 days are deleted
- Compactor runs every 10 minutes
- 2-hour safety buffer before actual deletion
- Rejects logs older than 7 days at ingestion


* * *

[Loki Retention Documentation](https://grafana.com/docs/loki/latest/operations/storage/retention/)

* * *


### 7). Loki Operational Limits

This provides stability for loki, and ensures that one noisy container (spamming logs) cannot take down the entire logging system.

These limits also control resource usage:

```yaml
  ingestion_rate_mb: 10
  ingestion_burst_size_mb: 20
  max_query_series: 100000
  max_query_parallelism: 32
```

- The ingestion rate, `ingestion_rate_mb: 10`, caps the incoming log volume at 10 Megabytes per second. This limits how fast logs can be written, preventing a single container from overwhelming Loki. 

- The `max_query_series` stops users from running massive queries (like "show me all logs for all time") that would crash the server by consuming all resources.

- These are reasonable defaults, but you may need to adjust them based on your log volume and query patterns.

* * *

### 8). Loki with GeoIP data

We're collecting labels from Docker, from Traefik access logs, and potentially from GeoIP lookups.
The default Loki limits are quite restrictive, so we've increased them to accommodate our rich labeling strategy. 

```yaml
  max_label_names_per_series: 30
  max_label_name_length: 1024
  max_label_value_length: 2048
```

- By increasing `max_label_names_per_series` to 30, you enable the complex parsing in Alloy. Defaults are often too low for this.

- The `max_label_...` settings are tuned up to allow for rich metadata (like long URLs, complex User-Agents, or GeoIP data) without truncation errors.

Without these increased limits, Loki would reject logs that have too many labels or labels that are too long.

* * *

[Loki Limits Documentation](https://www.google.com/search?q=https://grafana.com/docs/loki/latest/configure/limits_config/)

* * *


### Additional Fixes That Might Be Useful

Here are some adustments you can make to your `loki-config.yaml`, if your setup requires it:


- If you have high traffic, you might need to bump `ingestion_rate_mb` to 50 or 100 to avoid "429 Too Many Requests" errors.

- You can add `query_timeout: 1m` here to automatically kill dashboard queries that hang for too long.


* * *

* * *

## Part 6: Grafana Configuration

Are we there yet? No, so if you have to use the bathroom we can make a quick stop now.


(You can use your current working folder, all deployments have at least one Grafana dashboard)


* * *

### 1). How Grafana Looks for Datasources

When Grafana starts, it needs to know where to find its data sources like Loki. Rather than configuring these manually through the UI, we use Grafana's provisioning system to automatically configure datasources when the container starts.

In our `docker-compose.yml`, we mount a provisioning directory into Grafana:

```yaml
volumes:
  - "./grafana/provisioning/:/etc/grafana/provisioning"
```

Grafana looks for YAML files in specific subdirectories under `/etc/grafana/provisioning`. The structure follows a convention:

```
/etc/grafana/provisioning/
  ├── datasources/
  ├── dashboards/
  ├── notifiers/
  ├── alerting/
  └── plugins/
```

Each subdirectory corresponds to a different type of Grafana configuration. When Grafana starts, it scans these directories and automatically provisions whatever it finds. This is incredibly powerful for infrastructure-as-code approaches because your Grafana configuration lives in version control alongside your application code.

The environment variables in Grafana's configuration also prepare it for this:

```yaml
environment:
  - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
```

This tells Grafana where to look for provisioning files. Although this is the default location, setting it makes the configuration clearer and easier to troubleshoot with docker volume mounts.

Grafana's provisioning system watches these directories every 10 second and will reload when a file changes, no need to  restart the container.


* * *

### 2). How Grafana Finds Loki and Sets the UID

The datasource configuration is where Grafana learns how to connect to Loki. Our configuration lives in `ds.yaml`:

```yaml
apiVersion: 1
datasources:
- name: Loki
  type: loki
  uid: lokithedatasourceuid
  access: proxy 
  orgId: 1
  url: http://loki:3100
  basicAuth: false
  isDefault: true
  version: 1
  editable: false
```

Let's break down each field. The `name` is what appears in Grafana's datasource dropdown. The `type: loki` tells Grafana this is a Loki datasource, which determines what query interface Grafana shows and how it communicates with the backend.

The `uid` (unique identifier) is particularly important. This identifier is used in dashboard JSON definitions to reference this specific datasource. If you import a dashboard that was built against a Loki datasource with UID `lokithedatasourceuid`, Grafana will automatically connect the dashboard panels to this datasource. This makes dashboards portable across Grafana instances.

The `url` points to Loki's HTTP API at our static IP and port. The `access: proxy` setting means Grafana acts as a proxy for queries. When you view a dashboard, your browser sends queries to Grafana, and Grafana forwards them to Loki. This is better than direct access because it means Loki doesn't need to be accessible from users' browsers, and Grafana can cache and optimize queries.

The `isDefault: true` setting makes this the default datasource for new panels. When you create a new panel in a dashboard, it automatically selects this Loki instance. The `editable: false` setting prevents users from modifying this datasource through the UI, which helps maintain consistency in production environments.

This datasource file needs to be placed in the correct location for Grafana to find it. Based on our docker-compose configuration, it should be at:

```
./grafana/provisioning/datasources/ds.yaml
```

When Grafana starts, it reads this file and automatically creates the datasource connection to Loki. You'll see it immediately available in the datasource list without any manual configuration.


* * *

### 3). How Grafana Provisions Dashboards

Dashboards are the visual interface where you query and display your logs. Like datasources, dashboards can be provisioned automatically using configuration files.

The dashboard provisioning configuration is in `dashboard.yaml`:

```yaml
apiVersion: 1
providers:
  - name: "default"
    orgId: 1
    folder: ""
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    options:
      path: /etc/grafana/provisioning/dashboards
```

This configuration creates a provider that tells Grafana to look for dashboard JSON files in the specified path. The `type: file` means dashboards are loaded from files on disk rather than from a database or API.

The `updateIntervalSeconds: 10` setting is interesting. Grafana checks this directory every ten seconds for new or modified dashboard files. This means you can drop a new dashboard JSON file into the directory, and within ten seconds, it appears in Grafana without restarting anything.

The `disableDeletion: false` setting allows dashboards to be deleted through the UI. If this were true, any dashboard from this provider would be read-only and couldn't be deleted, which is useful in production environments where you want to prevent accidental deletion of important dashboards.

The `folder: ""` setting means dashboards appear at the root level of Grafana's dashboard list. You could set this to a folder name like "Production Monitoring" to organize dashboards automatically.

To actually provision dashboards, you would place JSON files in the configured path. Based on our docker-compose volumes, that would be:

```
./grafana/provisioning/dashboards/
```

Any JSON file you place there gets loaded as a dashboard. Dashboard JSON files can be exported from Grafana's UI or created programmatically. They're large JSON documents that describe every panel, query, and visualization in the dashboard.

Here's what makes this powerful: you can version control your dashboards alongside your application code. When you deploy a new version of your application, you can deploy updated dashboards at the same time. The dashboards automatically reference our Loki datasource by its UID, so everything connects seamlessly.

* * *

### 4). Adding New Dashboards

You can always [search for Grafana Dashboards](https://grafana.com/grafana/dashboards/) that other's have made public. No support provided.

You can edit:

- Name: `Container Log Dashboard`

- Folder: `Dashboards`

- Unique identifier (UID): `ghNnYnbt`

You **must** edit:

- Loki: `Select a Loki data source`

- `Import`


* * *

## Part 7: LyrionMediaServer Setup

To demonstrate our monitoring stack in action, we need an application that generates interesting logs. LyrionMediaServer (formerly Logitech Media Server) is a music streaming server that logs every track played, making it perfect for testing our Grafana alerting system.


* * *

### Persistant Data

We are going to keep persistant data inside of ./appdata/<app_name>

* * *

### 0.) Setup Working Folder First

1. Delete your current working folder
2. Create a new working folder
3. Copy `0-Setup-Lab-Environment` into it
4. Add `2-Alloy-Reads-Files-Traefik-Access-Logs` files
5. Run `1_permissions_init_for_project.sh`
6. Run `docker-compose up -d`

When moving to the next step: delete the working folder and repeat this process.


* * *

### 1). Configure LyrionMediaServer in Docker Compose

The LyrionMediaServer container needs access to your music files and a place to store its configuration. Add this service to your `docker-compose.yml`:


```yaml
---
  lyrionmusicserver:
    image: dlandon/lyrionmusicserver
    container_name: LyrionMusicServer
    ports:
      - "9000:9000"    # Web interface
      - "9090:9090"    # CLI interface
      - "3483:3483"    # SlimProto (TCP)
      - "3483:3483/udp" # SlimProto (UDP)
    env_file:
      - .env
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.lyrionmusicserver.loadbalancer.server.port=9000"
      - "traefik.http.routers.lyrionmusicserver.rule=Host(`lms.${DOMAIN}`) || Host(`lyrion.${SUBDOMAIN}`)"
      - "traefik.http.routers.lyrionmusicserver.entrypoints=websecure"
      - "traefik.http.routers.lyrionmusicserver.tls=true"
      - "traefik.http.routers.lyrionmusicserver.tls.certresolver=cloudflare"
      - "traefik.http.routers.lyrionmusicserver.service=lyrionmusicserver"
      - "traefik.http.routers.lyrionmusicserver.tls.domains[0].sans=*.${DOMAIN}"
      - "traefik.http.routers.lyrionmusicserver.tls.domains[1].sans=*.${SUBDOMAIN}"
    volumes:
      - '${MUSIC_STORAGE}:/music:ro'
      - './appdata/lyrion:/config:rw'
    networks:
      br1.232:
        ipv4_address: 10.236.232.138
```

That was a lot above, what does it all mean:

- Port 9000 is the web interface where you manage your music library
- Ports 3483 (TCP and UDP) are used by Squeezebox clients to connect to the server
- The ipv4_address is set to `10.236.232.138` this is a macvlan address on our host network
- The music directory is mounted as read-only (`:ro`) since the server only needs to read files
- Configuration is stored in `./appdata/lyrion` on your host for persistence


Access the web interface at `http://10.236.232.138:9000` to verify it's running. 
The first-time setup wizard will guide you through adding your music library, located at `/music`.


* * *

### 2). Install the PlayLog Plugin

At time of writing, you will need PlayLog.

This plugin allows you you to log the tracks you listen to, either automatically or by pressing a few remote control buttons. It provides a web interface for viewing its log, linking to the web for more information about what you've listened to, and downloading XML and M3U playlists of played songs.


#### Step 1: Install Plugin from Settings
First we need to install PlayLog:
- Click on the menu and find the **Settings** area
- Click on **Server**
- On the new page, click the drop down menu at the top
- Under **plugins** in the drop down menu, find **Manage Plugins**
- Click on **Manage Plugins** in the drop down menu
- Use the **search** field in the upper right corner
- **playlog** should bring up what we need
- Check the box for **PlayLog**
- Click **Save Settings** at the bottom of the page
- Restart the LyrionMediaServer container when prompted

#### Step 2: Configure PlayLog
Once the container restarts, go back to Settings > Server:
- In the drop down menu, click **PlayLog settings** (under Plugins section)
- Under **Current Song Logging**, select **All tracks** (every single track played generates a log entry)
- Click **Save Settings**


#### Step 3: Enable Debug Logging
To get the detailed log format we need for parsing:
- Go to **Settings** → **Server Settings** → **Logging**
- After clicking on **Logging** in the drop down menu, you should be on a new page.
- Check the box: **Save logging settings for use at next application restart**
- Scroll down to find `(plugin.PlayLog) - PlayLog` in the list
- Change its level from `WARN` to `DEBUG`
- Click **Save Settings**

The PlayLog plugin is now configured and will write play events to the Docker logs.

* * *

[PlayLog Plugin Documentation](https://tuxreborn.netlify.app/#slim)

* * *


### 3). Connect a Squeezelite Client

LyrionMediaServer needs at least one client connected to actually play music and generate logs. Squeezelite is a lightweight software player that runs on almost any platform.

**For Linux/Termux:**

If you have a spare Android phone, or tablet, you can use Termux (a Linux terminal emulator) and run Squeezelite directly on it.

Once Termux is installed, open it and run:

```bash
pkg update
pkg install squeezelite
```

Start the player:

```bash
squeezelite -N my_phone -s 10.236.232.138
```

- The `-N my_phone` flag sets the player name (this will appear in the LyrionMediaServer web interface)
- The `-s 10.236.232.138` flag points to your LyrionMediaServer's IP address
- Replace the IP with your actual server address from the docker-compose configuration

**For Desktop Clients:**

If you prefer a graphical interface, you can download one of these clients:

- [Squeezelite-X](https://sourceforge.net/projects/lmsclients/files/squeezelitex/) (Windows): Full-featured player with GUI
- [SqueezePad](https://apps.apple.com/us/app/squeezepad/id380003002) (iPad): Touch-optimized controller and player
- [MacOS](https://lyrion.org/getting-started/mac-install/) (MacOS): If you [downloaded](https://lyrion.org/downloads/) Lyrion, once installed it will insert a menu into your menu bar right-hand side
- [Melodeon](https://flathub.org/en/apps/io.github.cdrummond.melodeon) (Flatpak): Qt5/6 wrapper around MaterialSkin rendered in QWebEngine
- [Squeezer](https://f-droid.org/packages/uk.org.ngo.squeezer/) (Android): I use it to control playback from my Android Wear device
- [Lyrion](https://f-droid.org/en/packages/com.craigd.lmsmaterial.app/) (Android): Beautiful execution of a WebView wrapper for accessing a Lyrion Music Server instance using MaterialSkin
- [Squeezelite](https://f-droid.org/en/packages/org.lyrion.squeezelite/) (Android): If you dont want to use Termux and would like a GUI


All clients will automatically discover your LyrionMediaServer if they're on the same network. Note that LyrionMediaServer uses multicast UDP for discovery, so if your clients are on a different network segment, you'll need to manually specify the server IP address.


* * *

### 4). Add Music and Test Playback

Before we can generate logs, we need music in the library:

#### Step 1: Add Music Files
- Place some MP3s files in your `${MUSIC_STORAGE}` directory
- Dont have any MP3s? You can always find some new music at [OCRemix](https://ocremix.org/)
- Once you have your music in place
- In the LyrionMediaServer web interface, go to **Settings** → **Basic Settings**
- Under **Music Folder**, click **Scan** to index your files

#### Step 2: Start Playing
- On your Squeezelite client device, you should see "my_phone" appear in the LyrionMediaServer web interface
- Select a song from your library
- Press play

#### Step 3: Verify Logs Are Generated
Watch the Docker logs to confirm PlayLog is working:

```bash
docker logs -f LyrionMusicServer
```

You should see log entries like:

```
[XX-XX-XX XX:XX:XX.XXXX] Slim::Plugin::PlayLog::Plugin::logTrack (XXX) currently playing "Track Title	file:///.../music.mp3	Artist Name	Album Name"
```

This log format is what we'll parse in our Alloy configuration. The fields are tab-separated (`\t`) in the order: title, file path, artist, album.

#### Didnt work?

- Help is here!

If you don't see these logs, verify:
- PlayLog is set to log "All tracks"
- Debug logging is enabled for `plugin.PlayLog`
- The LyrionMediaServer container was restarted after changing logging settings

* * *


* * *

## Part 8: Airsonic Advanced Setup

While LyrionMediaServer gives us basic music playback logs, it's mainly intended for around the house streaming.

Airsonic Advanced provides a more feature-rich music streaming web-server.

This section demonstrates a different logging pattern: instead of parsing unstructured text logs like we do with LyrionMediaServer, we'll extract metadata from Airsonic's file paths and cache operations.

(You can use your current working folder, `5-Lyrion-Airsonic-Grafana-Alerts`)


* * *

### What Is Airsonic Advanced?

Airsonic Advanced is a fork of the Airsonic music server, which itself is a fork of Subsonic. It's a self-hosted music streaming server with a web interface, mobile apps, and support for transcoding, playlists, and user management.

For our logging stack, Airsonic is interesting because:
- It logs every song access through its cache system
- We can parse artist and album information from standardized directory structures

* * *

### A Note About Music Organization

This configuration assumes your music follows a specific directory structure:

```
/music_folder/Artist/[album-type]/(year) - album_name/track_number - track_title.mp3
```

For example:
```
/music/Pink Floyd/[album]/(1973) - The Dark Side of the Moon/01 - Speak to Me.flac
/music/Led Zeppelin/[compilation]/(1990) - Remasters/05 - Whole Lotta Love.mp3
```

If your music isn't organized this way, I highly recommend using [Beets](https://beets.io/), a command-line tool that automatically organizes and tags your music library. The Docker image from LinuxServer.io makes this easy:

```bash
docker run -v /your/music:/music ghcr.io/linuxserver/beets
```

Configure Beets with this path format in `~/.config/beets/config.yaml`:

```yaml
paths:
    default: %asciify{$albumartist}/[$albumtype]/($original_year) - $album%aunique{}/$track - $title
    singleton: Non-Album/$artist - $title
    comp: Compilations/%asciify{$albumartist}/($original_year) - $album%aunique{}/$track - $title
    albumtype_soundtrack: Soundtracks/$album/$track $title
```

This folder structure is the best way to store your complete music archive. 

* * *

[Beets Documentation](https://beets.readthedocs.io/)


* * *

### 1). Configure Airsonic in Docker Compose

Add the Airsonic Advanced service to your `docker-compose.yml`:

```yaml
services:
  airsonic-advanced:
    image: airsonicadvanced/airsonic-advanced:latest
    container_name: airsonic-advanced
    environment:
      - TZ=America/Denver
      - CONTEXT_PATH=/
      - JAVA_OPTS=-Xms256m -Xmx512m
    env_file:
      - .env
    ports:
      - "4040:4040"     # WebUI
      - "4041:4041"     # WebUI-HTTPS
      - "1900:1900/udp" # Upnp
    volumes:
      - './appdata/airsonic-advanced:/var/airsonic:rw'
      - '${MUSIC_STORAGE}:/var/music/:ro'
      - '${MUSIC_STORAGE}/_podcasts:/var/podcasts:rw'
      - '${MUSIC_STORAGE}/_playlists:/var/playlists:rw'
    labels:
      - "alloy.job=airsonic"
      - "traefik.enable=true"
      - "traefik.http.services.airsonic.loadbalancer.server.port=4040"
      - "traefik.http.routers.airsonic.rule=Host(`airsonic.${DOMAIN}`) || Host(`music.${SUBDOMAIN}`)"
      - "traefik.http.routers.airsonic.entrypoints=websecure"
      - "traefik.http.routers.airsonic.tls=true"
      - "traefik.http.routers.airsonic.tls.certresolver=cloudflare"
      - "traefik.http.routers.airsonic.tls.domains[0].sans=*.${DOMAIN}"
      - "traefik.http.routers.airsonic.tls.domains[1].sans=*.${SUBDOMAIN}"
    networks:
      br1.232:
        ipv4_address: 10.236.232.156
```


Configuration details:

- Port 4040 is the main web interface
- Port 4041 is used for HTTPS if configured
- Port 1900 (UDP) is for UPnP/DLNA discovery
- The Docker label `alloy.job=airsonic` is important - we'll use this to filter logs in Alloy
- Configuration persists in `./appdata/airsonic-advanced`
- Music is mounted read-only; podcasts and playlists are read-write

Access the web interface at `http://10.236.232.156:4040`. The default credentials are:
- Username: `admin`
- Password: `admin`

Change these immediately after logging in for the first time.


* * *

#### Best Practice: Use Docker Labels for Alloy Discovery

Docker lets you apply **labels to containers**, and Alloy can **use those labels to target the correct container log**.

In our example, in `docker-compose.yml` above:

```yaml
services:
  airsonic:
    image: ...
    labels:
      - "alloy.job=airsonic"
```

This adds a Docker label `alloy.job` with value `airsonic` to the container.



* * *

### 2). Configure Alloy to Discover Airsonic Container

Now we move to the Alloy side to configure log collection. Unlike our earlier general Docker discovery configuration, **we want to create a dedicated pipeline just for Airsonic logs**.

In your `config.alloy` file, we'll use a two-step approach: first discover all Docker containers, then filter to only the Airsonic container using a relabel rule.

#### Step 1: Discover Airsonic Using Relabeling

This is our configuration for `config.alloy`:

```ini
// Discover and filter for Airsonic container
discovery.relabel "airsonic_container" {
    targets = discovery.docker.containers.targets

    // Only keep the airsonic-advanced container
    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/airsonic-advanced"
        action        = "keep"
    }

    // Remove the leading slash from container name
    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/(.*)"
        target_label  = "container"
    }

    // Add stream label (stdout/stderr)
    rule {
        source_labels = ["__meta_docker_container_log_stream"]
        target_label  = "stream"
    }
}
```

This relabeling configuration filters the discovered Docker containers to only include `airsonic-advanced`. The `action = "keep"` means any container that doesn't match the regex is dropped from this pipeline.

The second and third rules clean up the container name (removing the leading `/` that Docker adds) and add a stream label to distinguish stdout from stderr logs.


* * *

#### Step 1 Alternative: Using Docker Labels Instead of Names

The configuration above filters by container name. If you prefer to use Docker labels (which a way better idea. It's is more flexible), you can filter by the `alloy.job=airsonic` label we added in the `docker-compose.yml` file.

Replace the first rule above with:

```ini
rule {
    source_labels = ["__meta_docker_container_label_alloy_job"]
    regex         = "airsonic"
    action        = "keep"
}
```

Docker labels are converted to Alloy metadata with the pattern: `__meta_docker_container_label_<label_name>`. Since our label is `alloy.job`, the dots are converted to underscores, giving us `__meta_docker_container_label_alloy_job`.

This approach has advantages:
- You can apply the same label to multiple containers
- You can easily enable/disable monitoring by adding/removing the label
- It's more explicit about which containers are being monitored


* * *

### 4). Configure Alloy to Read Airsonic Logs

With our filtered container targets ready, we can now configure the log source:

```ini
// Read logs from Airsonic container
loki.source.docker "airsonic_logs" {
    host       = "unix:///var/run/docker.sock"
    targets    = discovery.relabel.airsonic_container.output
    forward_to = [loki.process.airsonic_enrich.receiver]
}
```

This `loki.source.docker` component reads logs from the Docker socket, but only for containers that passed through our relabel filter. Notice we're not forwarding directly to Loki - instead, logs go to `loki.process.airsonic_enrich` for enrichment.

This is different from our general Docker log collection because we want to extract additional metadata from Airsonic's logs before storing them.


* * *

### 5). Parse Airsonic Logs and Add Labels

This is where the magic happens. Airsonic's cache logs contain file paths, and we can parse the artist name directly from those paths using regex. This creates searchable labels in Loki without storing duplicate data.

Add this processing pipeline to `config.alloy`:

```ini
//================================================================
// AIRSONIC MUSIC LOGS
// Take logs from Airsonic-Advanced into labels
//================================================================

// Filter Docker discovery for airsonic container (by name for now)
discovery.relabel "airsonic_container" {
    targets = discovery.docker.containers.targets

    // Only keep a containers with label: "alloy.job=airsonic"
    rule {
        source_labels = ["__meta_docker_container_label_alloy_job"]
        regex         = "airsonic"
        action        = "keep"
    }

    rule {
        source_labels = ["__meta_docker_container_log_stream"]
        target_label  = "stream"
    }
}

// Scrape ONLY airsonic logs from Docker
loki.source.docker "airsonic_logs" {
    host       = "unix:///var/run/docker.sock"
    targets    = discovery.relabel.airsonic_container.output
    forward_to = [loki.process.airsonic_enrich.receiver]
}

// Parse and enrich airsonic logs with IP, username, and artist info
loki.process "airsonic_enrich" {
    forward_to = [loki.write.local.receiver]

        // Add static job label FIRST (always applied)
        stage.static_labels {
            values = {
                job = "airsonic",
            }
        }

        // Extract IP from StreamController logs
        stage.match {
            selector = "{job=\"airsonic\"} |~ \"StreamController.*listening to\""

            stage.regex {
                expression = "(?P<ip>\\d+\\.\\d+\\.\\d+\\.\\d+): (?P<username>\\w+) listening to"
            }

            stage.labels {
                values = {
                    asonic_ip = "ip",
                    asonic_user  = "username",
                    log_type  = "stream",
                }
            }
        }

        // Extract artist from CacheConfiguration logs
        stage.match {
            selector = "{job=\"airsonic\"} |~ \"Cache Key:.*\\\\[(?:album|compilation|remix|single|ep)\\\\]\""

            stage.regex {
                expression = "Cache Key: (?P<artist>[^/]+)/\\[(?:album|compilation|remix|single|ep)\\]/"
            }

            stage.labels {
                values = {
                    asonic_music   = "artist",
                    log_type = "cache",
                }
            }
        }
}
```

Let's break down what each stage does:


* * *

#### Stage 1: Static Labels
The `stage.static_labels` block adds `job="airsonic"` to every log line that enters this pipeline. This happens first, before any matching or parsing.


* * *

#### Stage 2: Stream Log Parsing
The first `stage.match` block looks for logs containing `StreamController` and `listening to`. These are generated when a user starts playing a track. An example log looks like:

```
192.168.1.100: johndoe listening to /var/music/Pink Floyd/[album]/(1973) - Dark Side/01 - Speak to Me.flac
```

The `stage.regex` extracts:
- `ip` - The client's IP address (192.168.1.100)
- `username` - The Airsonic username (johndoe)

The `stage.labels` block converts these into Loki labels:
- `asonic_ip="192.168.1.100"`
- `asonic_user="johndoe"`
- `log_type="stream"`


* * *

#### Stage 3: Cache Log Parsing
The second `stage.match` block processes cache access logs. These are generated when Airsonic reads file metadata. 

An example log:

```
Cache Key: Pink Floyd/[album]/(1973) - Dark Side of the Moon/01 - Speak to Me.flac
```

The regex extracts the artist name (`Pink Floyd`) from the file path. The `[^/]+` pattern means "capture everything up to the first forward slash", which corresponds to our artist folder.

The regex also validates the album type is one of: album, compilation, remix, single, or ep. This prevents false matches on other file paths.

The extracted data becomes:
- `asonic_music="Pink Floyd"`
- `log_type="cache"`

Logs that don't match either stage selector still get the `job="airsonic"` label but skip the extraction stages. This includes error logs, startup messages, and other operational logs.


* * *

### 6). Understanding the Label Hierarchy

After processing, your logs will have different labels depending on their type. Here's the complete label hierarchy:

**All Airsonic Logs:**
- `job="airsonic"` - Always present
- `container="airsonic-advanced"` - From discovery

**Stream Logs (user playback):**
- All labels from "All Airsonic Logs"
- `asonic_ip` - Client IP address
- `asonic_user` - Username
- `asonic_music` - Artist name

This labeling strategy lets you write precise queries in Grafana:
- `{job="airsonic"}` - All Airsonic logs
- `{job="airsonic", log_type="stream"}` - Only playback events
- `{job="airsonic", asonic_music="Pink Floyd"}` - Only Pink Floyd songs accessed
- `{job="airsonic", asonic_user="admin"}` - Only admin's activity


* * *

### 7). Verify Airsonic Log Collection

After adding this configuration to Alloy and restarting the container, verify it's working:

#### Step 1: Check Alloy UI
Navigate to `http://your-alloy-host:12345/component/loki.process.airsonic_enrich`

You should see:
- Metrics showing logs processed
- The pipeline stages listed
- Any errors if the regex isn't matching

#### Step 2: Play a Song in Airsonic
Go to your Airsonic web interface and play a song.

#### Step 3: Query in Grafana
Open Grafana and go to Explore. Select your Loki datasource and run:

```
{job="airsonic"} | asonic_music != ""
```

You should see logs with the `asonic_music` label populated with artist names. If the label is empty or missing, check:
- Your music folder structure matches the expected format
- The regex in the `stage.regex` block matches your actual log format
- The cache logs are actually being generated (they may take a moment after playback starts)


* * *

## Part 9: Grafana Alerting System

Now that we have music streaming logs flowing into Loki with rich labels, we can configure Grafana's alerting system to notify us when specific songs or artists are played. This demonstrates the complete monitoring pipeline: logs → parsing → storage → alerting → notification.

Grafana's alerting system consists of three components that work together:
1. **Alert Rules** - Define what conditions trigger an alert
2. **Contact Points** - Define where to send notifications
3. **Notification Policies** - Define routing and timing behavior

We'll configure all three to send Telegram notifications when certain music is played.

* * *

### How Grafana Alerting Works

Before diving into configuration, it helps to understand the alert flow:


#### Step 1: Evaluation
Grafana periodically runs LogQL queries defined in alert rules. These queries check your Loki data for specific conditions.

#### Step 2: State Change
When a query result crosses a threshold, the alert state changes from "Normal" to "Alerting". This state change is what triggers the next steps.

#### Step 3: Notification Policy Matching
The alert is evaluated against notification policies. These policies determine which contact point receives the alert and control timing behaviors like grouping and repeat intervals.

#### Step 4: Contact Point Execution
The selected contact point sends the notification to its configured destination (Telegram, email, Slack, etc.).

#### Step 5: Repeat and Resolution
If the alert condition persists, notifications repeat according to the policy. When the condition clears, a resolution notification can optionally be sent.


* * *

### 1). Configure Telegram Bot for Notifications

Before we can send alerts to Telegram, we need to create a bot and get its credentials. This is a one-time setup process.

#### Step 1: Create a Telegram Bot
- Open Telegram and search for `@BotFather`
- Send the command `/newbot`
- Follow the prompts to choose a name and username for your bot
- BotFather will respond with a token like `112233445:AAQQqTtvv11gGHJXxfFtESEOsaAcKsSBlaDWin`
- Save this token - you'll need it for the Grafana configuration

#### Step 2: Get Your Chat ID
- Search for `@userinfobot` in Telegram
- Send it any message
- It will reply with your user ID (a number like `123456789`)
- This is your chat ID for personal messages

#### Step 3: If you want to send alerts to a group
- Create a group and add your bot to it
- Add `@userinfobot` to the group temporarily
- The bot will show the group's chat ID (it will be negative, like `-987654321`)
- Remove `@userinfobot` after getting the ID

#### Step 4: Add Credentials to .env File
Add these lines to your `.env` file:

```bash
MYTGRAM_BOTTOKEN=112233445:AAQQqTtvv11gGHJXxfFtESEOsaAcKsSBlaDWin
MYTGRAM_CHATID=123456789
```

Replace the values with your actual bot token and chat ID. These environment variables keep your credentials out of the configuration files.


* * *

### 2). Create Telegram Contact Point

Contact points define where Grafana sends notifications. We'll create one for Telegram using the provisioning system so it's automatically configured when Grafana starts.

We will be using the file `./grafana/provisioning/alerting/ContactPoint-Telegram.yaml`:

```yaml
apiVersion: 1
contactPoints:
  - orgId: 1
    name: MusicAlert_bot
    receivers:
      - uid: telegram_music_alerts
        type: telegram
        settings:
          bottoken: ${MYTGRAM_BOTTOKEN}
          chatid: >
            ${MYTGRAM_CHATID}
          disable_notification: false
          disable_web_page_preview: false
          protect_content: false
        disableResolveMessage: true
```

Let's break down this configuration:

#### Contact Point Identification
- `name: MusicAlert_bot` - This is how you reference this contact point in notification policies
- `uid: telegram_music_alerts` - A unique identifier for this specific receiver

#### Telegram Settings
- `bottoken` - References your bot token from the .env file
- `chatid` - Your personal or group chat ID (see the note below about the `>` syntax)
- `disable_notification: false` - Messages will trigger sound/vibration on your phone
- `disable_web_page_preview: false` - Telegram will show previews for any URLs in messages
- `protect_content: false` - Allows forwarding and saving messages

#### Alert Behavior
- `disableResolveMessage: true` - When an alert clears, Grafana won't send a "resolved" notification. This prevents notification spam when music stops playing.


* * *

**Important: The ChatID YAML Syntax**

You'll notice the `chatid` uses special YAML syntax:

```yaml
chatid: >
  ${MYTGRAM_CHATID}
```

The `>` symbol is a YAML folded scalar. This is necessary because of a bug in Grafana's YAML parser (issue #69950). The chat ID is a number, but Telegram's API requires it as a string. Without the folded scalar syntax, Grafana interprets the environment variable as a number and the API call fails.

The indentation matters - the `${MYTGRAM_CHATID}` line must be indented relative to `chatid:`. This forces YAML to treat the entire value as a string type, even though it contains only digits.

This is not the only solution, but it's the cleanest way to handle numeric values that need to be strings without hardcoding quotes (which would prevent environment variable expansion).

* * *

[Grafana Telegram Configuration Documentation](https://grafana.com/docs/grafana/latest/alerting/configure-notifications/manage-contact-points/integrations/configure-telegram/)

* * *


### 3). Create Notification Policy

Notification policies control the routing and timing of alerts. They determine which contact point receives each alert and how often notifications repeat.

We will be using the file `./grafana/provisioning/alerting/Notification-Policy.yaml`:

```yaml
apiVersion: 1
policies:
  - orgId: 1
    receiver: MusicAlert_bot
    group_by:
      - grafana_folder
      - alertname
    group_wait: 0s
    group_interval: 5m
    repeat_interval: 10m
```

#### Routing Configuration
- `receiver: MusicAlert_bot` - All alerts matching this policy go to our Telegram contact point
- This is the root policy, so it applies to all alerts by default

#### Grouping Configuration
- `group_by: [grafana_folder, alertname]` - Alerts are grouped by their folder location and rule name
- This means if both the Airsonic and Lyrion alerts fire simultaneously, they'll be sent as separate notifications
- Without grouping, you'd get one notification per label combination, which could be dozens of messages

#### Timing Configuration
- `group_wait: 0s` - Send notifications immediately, don't wait to collect more alerts into the group
- `group_interval: 5m` - If more alerts join this group, wait 5 minutes before sending an update
- `repeat_interval: 10m` - If the alert condition persists, resend the notification every 10 minutes

This timing configuration is tuned for music alerts where:
- You want immediate notification when someone starts playing monitored songs
- You don't want spam if they listen to multiple songs in a row (hence the 5-minute grouping)
- You want periodic reminders if they're binge-listening (every 10 minutes)


* * *

#### Extending This Policy

You can add nested policies for more complex routing. For example, to send critical alerts to Telegram with sound and warning alerts via email:

```yaml
policies:
  - orgId: 1
    receiver: MusicAlert_bot
    routes:
      - receiver: MusicAlert_bot
        matchers:
          - severity = critical
        group_wait: 0s
      - receiver: email_team
        matchers:
          - severity = warning
        group_wait: 5m
```

You can also add mute timings to silence notifications during specific hours:

```yaml
mute_time_intervals:
  - name: sleep_hours
    time_intervals:
      - times:
        - start_time: '23:00'
          end_time: '08:00'
```

Then reference it in your policy:

```yaml
policies:
  - orgId: 1
    receiver: MusicAlert_bot
    mute_time_intervals:
      - sleep_hours
```

* * *

[Grafana Notification Policy Documentation](https://grafana.com/docs/grafana/latest/alerting/configure-notifications/create-notification-policy/)

* * *


### 4). Create Alert Rule for Lyrion Music

Alert rules define the actual conditions that trigger notifications. We'll create two rules: one for LyrionMediaServer and one for Airsonic. These demonstrate different approaches to log parsing.

The LyrionMediaServer alert uses regex to parse unstructured text logs at query time. We will be using the file `./grafana/provisioning/alerting/LyrionAlert.json`:

> Sorry I didnt include the JSON for the alerting, it was too large. Please find it in the repo link.

* * *

#### Understanding the LogQL Query Above

```sql
sum by (title, artist) (
  rate({container=~"(?i)(lyrionmusicserver|lms)"}
    |= "currently playing"
    |~ `(?i)(nickelback|creed|insane clown posse|limp bizkit|crazy potato)`
    | regexp `currently playing "(?P<title>[^\t]+)\t(?P<url>[^\t]+)\t(?P<artist>[^\t]+)\t`
  [3s])
)
```

Let's break this down line by line:

* * *

##### Line 1: Label Filter
- `{container=~"(?i)(lyrionmusicserver|lms)"}` - Select logs from containers named "lyrionmusicserver" or "lms"
- The `(?i)` makes the match case-insensitive
- The `=~` operator means "matches this regex"

##### Line 2: Line Filter
- `|= "currently playing"` - Only logs containing this exact phrase
- This is a fast filter that runs before regex parsing
- Line filters are much faster than regex, so always use them when possible

##### Line 3: Content Filter
- `|~ (regex pattern)` - Only logs matching this regex pattern
- Looks for song titles or artists containing our monitored bands
- The list includes: Nickelback, Creed, Insane Clown Posse, Limp Bizkit, Crazy Potato

##### Line 4: Parse and Extract
- `| regexp` - Parse the log line and extract named groups
- `(?P<title>[^\t]+)` - Capture the song title (everything up to the first tab)
- `(?P<artist>[^\t]+)` - Capture the artist name (after the second tab)
- These become labels we can use in aggregations

##### Line 5: Rate Calculation
- `[3s]` - Calculate the rate over a 3-second window
- This converts log line counts into events per second


* * *

### 5). Create Alert Rule for Airsonic Music

The Airsonic alert uses pre-parsed labels from our Alloy configuration, making the query much simpler and more efficient.

We will be using the file `./grafana/provisioning/alerting/AirsonicAlert.json`:

> Sorry I didnt include the JSON for the alerting, it was too large. Please find it in the repo link.

* * *

#### Understanding the LogQL Query

This query is much simpler than the Lyrion one because:

```sql
sum by (asonic_music) (
  rate({job="airsonic"}
    | asonic_music =~ `(?i)(nickelback|creed|fred durst|Rick Astley|Limp Bizkit)`
  [3m])
)
```

##### Line 1: Label Filter
- `{job="airsonic"}` - Select only Airsonic logs
- This is a label we added in our Alloy `stage.static_labels` block

##### Line 2: Label Matcher
- `| asonic_music =~ (regex)` - Filter on the pre-parsed artist label
- This label was extracted by our Alloy `stage.regex` from the cache file paths
- No regex parsing needed at query time - the work was already done by Alloy

##### Line 3: Rate and Aggregation
- `[3m]` - Calculate rate over a 3-minute window (longer than Lyrion because Airsonic logs are less frequent)
- `sum by (asonic_music)` - Group by artist name


##### Why This Is More Efficient

The Lyrion query must:
1. Search log content for keywords
2. Parse each matching log with regex
3. Extract title and artist at query time
4. Then aggregate the results

The Airsonic query:
1. Filter by job label (indexed, very fast)
2. Filter by artist label (also indexed)
3. Aggregate

By doing the parsing in Alloy, we've moved the expensive work from query time (when you're viewing dashboards) to ingestion time (when logs arrive). This makes dashboards faster and reduces load on Loki.

The tradeoff is that you need to know what you want to extract before logs arrive. You can't retroactively add labels to old logs. The Lyrion approach is more flexible - you can change the regex in your query anytime - but it's slower.


* * *

### 6). Test the Alert System

Now for the fun part - testing if everything works:

**Step 1: Verify Contact Point**
- In Grafana, go to **Alerting** → **Contact points**
- Find "MusicAlert_bot" in the list
- Click **Test** to send a test message to your Telegram
- You should receive a test notification within a few seconds

If the test fails:
- Check your bot token is correct in the `.env` file
- Verify you've started a conversation with your bot in Telegram (send it any message first)
- For group chats, ensure the bot was added before you got the chat ID

**Step 2: Play a Monitored Song**
- Open either LyrionMediaServer or Airsonic
- Play a song by one of the monitored artists (Nickelback, Creed, etc.)
- Wait up to 30 seconds for the alert to evaluate
- Check your Telegram for a notification

The notification should include:
- The alert title
- A summary showing which artist/song triggered it
- A timestamp
- Links back to Grafana

**Step 3: Verify Alert State in Grafana**
- Go to **Alerting** → **Alert rules**
- The triggered alert should show state "Alerting" with a red background
- Click on the alert to see its evaluation history and labels

If the alert doesn't fire:
- Check the query returns data in **Explore** (use the same LogQL query from the alert)
- Verify your logs are being collected (check `{job="airsonic"}` or `{container=~"lyrion.*"}`)
- Look at Grafana's logs for evaluation errors: `docker logs grafana`

* * *

### 7). Customizing Alert Behavior

Now that the basic system works, here are some ways to customize it:

**Add More Artists:**
Simply edit the regex pattern in the JSON files. For example, to also monitor for Taylor Swift:

```
|~ `(?i)(nickelback|creed|taylor swift|fred durst)`
```

**Alert on Specific Users:**
Modify the Airsonic query to include the user label:

```sql
sum by (asonic_music, asonic_user) (
  rate({job="airsonic", asonic_user="johndoe"}
    | asonic_music =~ `(?i)(nickelback|creed)`
  [3m])
)
```

**Change Notification Frequency:**
Edit `Notification-Policy.yaml`:
- Reduce `repeat_interval` to 5m for more frequent reminders
- Increase `group_interval` to 10m to batch more alerts together

**Add Severity Levels:**
Create separate alerts with different severity labels, then route them to different contact points:

```yaml
routes:
  - receiver: telegram_urgent
    matchers:
      - severity = critical
  - receiver: email_team
    matchers:
      - severity = warning
```

**Mute During Sleep Hours:**
Add mute timings to the notification policy:

```yaml
mute_time_intervals:
  - name: night_hours
    time_intervals:
      - times:
        - start_time: '22:00'
          end_time: '07:00'
        weekdays: ['saturday', 'sunday']
```

* * *

## Final Monitoring and Alerting Flow

Let's trace a single log line through the entire system to see how all the pieces connect:

**Step 1: Music Plays**
A user plays "Photograph" by Nickelback in Airsonic.

**Step 2: Airsonic Logs**
Airsonic writes a cache access log to stdout:
```
Cache Key: Nickelback/[album]/(2005) - All The Right Reasons/01 - Photograph.flac
```

**Step 3: Docker Captures**
The Docker daemon captures this stdout log from the container.

**Step 4: Alloy Discovers**
The `discovery.docker` block finds the Airsonic container.

**Step 5: Alloy Filters**
The `discovery.relabel` block matches the container name and keeps it for processing.

**Step 6: Alloy Reads**
The `loki.source.docker` block reads the log line from Docker.

**Step 7: Alloy Parses**
The `loki.process` pipeline:
- Adds label: `job="airsonic"`
- Matches the cache pattern
- Extracts: `asonic_music="Nickelback"`
- Adds label: `log_type="cache"`

**Step 8: Alloy Sends**
The processed log with all labels is sent to Loki at port 3100.

**Step 9: Loki Stores**
Loki indexes the labels and compresses the log line for storage.

**Step 10: Grafana Queries**
Every 30 seconds, Grafana runs the alert query:
```sql
{job="airsonic"} | asonic_music =~ `(?i)(nickelback|...)`
```

**Step 11: Condition Triggers**
The query returns a rate > 0, so the condition evaluates to true.

**Step 12: Policy Routes**
The notification policy matches the alert and selects the "MusicAlert_bot" contact point.

**Step 13: Telegram Sends**
The contact point uses the Telegram API to send a message to your phone.

**Step 14: You React**
Your phone buzzes with: "Airsonic playback detected: Nickelback"

And that's the complete flow from music playback to mobile notification, demonstrating the power of modern observability stacks for both serious monitoring and fun use cases like this.

* * *

- [Grafana Alerting Overview](https://grafana.com/docs/grafana/latest/alerting/)

- [LogQL Query Language Documentation](https://grafana.com/docs/loki/latest/query/)

- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)

* * *
