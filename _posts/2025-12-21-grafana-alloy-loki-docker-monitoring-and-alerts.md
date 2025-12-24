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

[Loki](https://grafana.com/docs/enterprise-logs/latest/get-started/overview/) is Grafana's log aggregation system. It's designed to be a way to store your losts, cost-effective and easy to operate.

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

You will see both `container` and `service_name` labels from `__meta_docker_container_name`. This is for maximum Grafana dashboard compatability.

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

https://youtu.be/Xa3mCIdsno4?t=96

* * *

> In this setup, you'll be sending **ALL** of your logs to the Grafana Cloud. Please be aware, all logs for all containers and all logs being read from all files.


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

#### 4). Loki Log Storage Style

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

## The Complete Flow

1. **Container logs to Docker** → Docker daemon captures them
2. **Alloy discovers containers** → Via Docker socket
3. **Alloy applies labels** → Container name, stream, etc.
4. **Alloy tails Traefik logs** → Parses JSON, adds labels
5. **Alloy sends to Loki** → HTTP push to port 3100
6. **Loki indexes and stores** → Labels + compressed chunks
7. **Grafana queries Loki** → LogQL queries via provisioned datasource
8. **Compactor manages retention** → Deletes logs older than 30 days



* * *

## LyrionMediaServer

We have to Add Lyrion to be able to test Grafana.


* * *

### Persistant Data

We are going to keep persistant data inside of ./appdata/<app_name>


```yaml
---
  lyrionmusicserver:
    image: dlandon/lyrionmusicserver
    container_name: LyrionMusicServer
    ports:
      - "9000:9000"
      - "9090:9090"
      - "3483:3483"
      - "3483:3483/udp"
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
      - '${MUSIC_STORAGE}:/music/ultimate:ro'
      - './appdata/lyrion:/config:rw'
    networks:
      br1.232:
        ipv4_address: 10.236.232.138
```


* * *

### Install the PlayLog plugin

Thank you Peter.

  PlayLog (v2.1.49) 

This plugin allows you you to log the tracks you listen to, either automatically or by pressing a few remote control buttons. It provides a web interface for viewing its log, linking to the web for more information about what you've listened to, and downloading XML and M3U playlists of played songs. (Boom, Classic, Slimp3, SoftSqueeze, Squeezebox1, Transporter; limited support for Radio, Receiver, and Touch) 

https://tuxreborn.netlify.app/#slim


Go into the settings

In PlayLog settings:

Under "CURRENT SONG LOGGING":
Select "All tracks"

> "All tracks" will have PlayLog log every track played

Then go to 

Server > Logging

At the top, under the log files destination you will find,  "Advanced Log Settings"

Check the `Save logging settings for use at next application restart` box

Scroll down to: `(plugin.PlayLog) - PlayLog`

and set it to `Debug`

`Save`


* * *

### Add some songs play some songs

To add some songs, you must connect a client to the server.

Get on your phone, download termux, and run `pkg install squeezelite`

Super.

Now you can run: `squeezelite -N my_phone -s 10.236.232.138`

LyrionMusicServer uses multicast and UDP. It's not fun to reverse proxy into a hole. Set a DNS reservation for the address for the server and that should work fine. You're only internal when using LyrionMusicServer anyway.


### Now check output

`docker container logs -f lyrion`


* * *
* * *

## Airsonic example

I will admit, this dashboard is slightly built around my folder structure.

```
/music_folder/Band/[type-of-release]/(date)-album_name/
```

If you're not using this. [Grab a copy of beats](https://docs.linuxserver.io/images/docker-beets/) and use it!


```yaml
paths:
    default: %asciify{$albumartist}/[$albumtype]/($original_year) - $album%aunique{}/$track - $title
    singleton: Non-Album/$artist - $title
    comp: Compilations/%asciify{$albumartist}/($original_year) - $album%aunique{}/$track - $title
    albumtype_soundtrack: Soundtracks/$album/$track $title 
```

Let's get airsonic setup

### air sonic docker compose


```yaml


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
      - "4040:4040"
      - "4041:4041"
      - "1900:1900/udp"
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


### Best Practice: Use Docker Labels + Alloy Discovery

Docker lets you apply **labels to containers**, and Alloy can **use those labels to target the correct container log**.


* * *

### Step 1: Add a Label to the Container

When you run `airsonic-advanced`, add a label:

```bash
docker run \
  --label=alloy.job=airsonic \
  your/airsonic-advanced:image
```

Or in `docker-compose.yml`:

```yaml
services:
  airsonic:
    image: ...
    labels:
      - "alloy.job=airsonic"
```

This adds a Docker label `alloy.job` with value `airsonic` to the container.


* * *

### Step 2: Discover containers with discovery.docker

```ini
discovery.docker "containers" {
  host = "unix:///var/run/docker.sock"
}
```
This discovers all Docker containers and exposes their metadata. The Docker label `alloy.job=airsonic` becomes available as `__meta_docker_label_alloy_job`.


* * *

### Step 3: Filter Using discovery.relabel

This is from ##### Step 4: Finish Step 3

But this is different than ##### Step 4: Finish Step 3
This time, we are going to use `discovery.relabel` to filter for the airsonic-advanced container by name:

```ini
discovery.relabel "airsonic_container" {
    targets = discovery.docker.containers.targets

    // Only keep the airsonic-advanced container
    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/airsonic-advanced"
        action        = "keep"
    }

    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/(.*)"
        target_label  = "container"
    }

    rule {
        source_labels = ["__meta_docker_container_log_stream"]
        target_label  = "stream"
    }
}
```

**What `discovery.relabel` Does:**
- Takes all discovered containers from `discovery.docker`
- First rule: `action = "keep"` means **only keep** containers where `__meta_docker_container_name` matches `/airsonic-advanced`
- Second rule: Removes the leading `/` from container name and creates a `container` label
- Third rule: Adds a `stream` label (stdout/stderr)

**Important:** This uses container NAME matching, not Docker labels. Only containers with the exact name `airsonic-advanced` are processed.


* * *

### Alternative: Using Docker Labels Instead of Names (optional)

If you wanted to filter by Docker labels instead of container names, here's how:

#### 1. Add a label to your docker-compose.yml:

```yaml
services:
  airsonic:
    image: your/airsonic-advanced:image
    labels:
      - "alloy.job=airsonic"
    # ... rest of your config
```


* * *

#### 2. Modify the discovery.relabel rule in config.alloy:

**Change this:**
```ini
discovery.relabel "airsonic_container" {
    targets = discovery.docker.containers.targets

    // Only keep the airsonic-advanced container
    rule {
        source_labels = ["__meta_docker_container_name"]
        regex         = "/airsonic-advanced"
        action        = "keep"
    }

```

**To this:**
```ini
discovery.relabel "airsonic_container" {
    targets = discovery.docker.containers.targets

    // Only keep containers with alloy.job=airsonic label
    rule {
        source_labels = ["__meta_docker_container_label_alloy_job"]
        regex         = "airsonic"
        action        = "keep"
    }

```

**Key changes:**
- `source_labels` changes from `["__meta_docker_container_name"]` to `["__meta_docker_container_label_alloy_job"]`
- `regex` changes from `"/airsonic-advanced"` to `"airsonic"` (no leading slash since it's a label value, not a container name)

**How the label mapping works:**
- Docker label: `alloy.job=airsonic`
- Becomes in Alloy: `__meta_docker_container_label_alloy_job` with value `airsonic`
- The dots (`.`) in the label name get replaced with underscores (`_`) and prefixed with `__meta_docker_container_label_`

**Benefits of using labels:**
- Can target multiple containers with the same label
- More flexible than hardcoding container names
- Easier to manage multiple environments (dev/staging/prod)


* * *

### Step 4: Filter using the relabel from above


* * *

### What `loki.source.docker` Does

Scrapes ONLY airsonic logs from Docker.

```ini
loki.source.docker "airsonic_logs" {
    host       = "unix:///var/run/docker.sock"
    targets    = discovery.relabel.airsonic_container.output
    forward_to = [loki.process.airsonic_enrich.receiver]
}
```

**`targets = discovery.relabel.airsonic_container.output`**
- Gets the filtered container from `discovery.relabel`
- Only the airsonic-advanced container is included

**`forward_to = [loki.process.airsonic_enrich.receiver]`**
- Sends each log line to the processing pipeline
- Goes to `loki.process.airsonic_enrich` (not directly to Loki)

**What it does:**
- Reads logs from the filtered airsonic container
- Forwards each log line to the enrichment pipeline (not directly to Loki)



* * *

### What `loki.process "airsonic_enrich"` Does

This is the part the reads from the logs and sends the data back out as a Loki label.

```ini
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

This is a **multi-stage processing pipeline** that enriches logs:

* * *

### Stage 1: `stage.static_labels`
**Adds static labels to EVERY log line:**
- `job="airsonic"` is applied first, before any other processing

* * *

### Stage 2: `stage.match` for StreamController logs
**Only processes logs matching the selector:**
- Selector: `{job="airsonic"} |~ "StreamController.*listening to"`
- This is a LogQL query that filters which logs enter this stage

**Inside this stage:**
1. **`stage.regex`** - Extracts data using regex:
   - Pattern: `(?P<ip>\\d+\\.\\d+\\.\\d+\\.\\d+): (?P<username>\\w+) listening to`
   - Captures IP address into variable `ip`
   - Example log: `192.168.1.5: user listening to cool-track.mp3`
   - Extracts: `ip="192.168.1.5"`
   - **NEW** Username: `user`

2. **`stage.labels`** - Converts extracted data to labels:
   - Takes the `ip` variable from regex
   - Creates label `client_ip="192.168.1.5"`
   - Also adds static label `log_type="stream"`


* * * 

### Stage 3: `stage.match` for CacheConfiguration logs

**Only processes logs matching the selector:**
- Selector: `{job="airsonic"} |~ "Cache Key:.*\\\\[(?:album|compilation|single|ep)\\\\]"`

**Inside this stage:**
1. **`stage.regex`** - Extracts artist name:
   - Pattern: `Cache Key: (?P<artist>[^/]+)/\\[(?:album|compilation|single|ep)\\]/`
   - Example log: `Cache Key: Pink Floyd/[album]/Dark Side of the Moon`
   - Extracts: `artist="Pink Floyd"`

2. **`stage.labels`** - Converts to labels:
   - Creates label `artist="Pink Floyd"`
   - Adds static label `log_type="cache"`

**Important:** Logs that don't match either stage selector still get the `job="airsonic"` label, but don't get the stage-specific labels.


* * *

### The Full Flow

**Step 1: `discovery.docker "containers"`**
- Connects to Docker and discovers all running containers
- Exposes container metadata including names and log streams

**Step 2: `discovery.relabel "airsonic_container"`**
- Takes all discovered containers from Step 1
- Filters to ONLY containers named `/airsonic-advanced` using `action = "keep"`
- Adds `container` and `stream` labels to the matching container

**Step 3: `loki.source.docker "airsonic_logs"`**
- Reads logs from the filtered airsonic container from Step 2
- Uses Docker's native logging interface (not file-based)
- Forwards each log line to the next stage

**Step 4: `loki.process "airsonic_enrich"` - Stage 1 (static_labels)**
- Receives log entries from Step 3
- Adds label `job="airsonic"` to EVERY log line
- This happens first, before any matching

**Step 5: `loki.process "airsonic_enrich"` - Stage 2 (StreamController match)**
- Only processes logs matching: `{job="airsonic"} |~ "StreamController.*listening to"`
- Uses regex to extract `ip` from matching log lines
- Example: `192.168.1.5: user listening to cool-track.mp3`
  - Extracts: `ip="192.168.1.5"`
- Converts `ip` into labels: `client_ip="192.168.1.5"` and `log_type="stream"`
- Non-matching logs skip this stage

**Step 6: `loki.process "airsonic_enrich"` - Stage 3 (CacheConfiguration match)**
- Only processes logs matching cache entries with album types
- Uses regex to extract `artist` from matching log lines
- Example: `Cache Key: Pink Floyd/[album]/Dark Side of the Moon`
  - Extracts: `artist="Pink Floyd"`
- Converts to labels: `artist="Pink Floyd"` and `log_type="cache"`
- Non-matching logs skip this stage

**Step 7: `loki.write "local"`**
- Receives all processed log entries from Step 4-6
- Sends them to Loki at `http://loki:3100/loki/api/v1/push`

* * *

**In summary:** Discover → Filter by Name → Read Logs → Add Base Labels → Match & Parse Streams → Match & Parse Cache → Send to Loki

* * *


* * *

### Label Results

After processing, logs will have different labels depending on their content:

**All airsonic logs get:**
- `job="airsonic"`
- `container="airsonic-advanced"`
- `stream="stdout"` or `stream="stderr"`

**StreamController logs also get:**
- `client_ip="192.168.1.5"` (extracted IP)
- `log_type="stream"`

**CacheConfiguration logs also get:**
- `artist="Pink Floyd"` (extracted artist)
- `log_type="cache"`

**Other airsonic logs:**
- Only get the base labels (job, container, stream)
- Don't match either stage selector, so skip the extraction stages





* * *

### Summary of Label Hierarchy

#### All Docker Logs
- `container` - Container name (without leading `/`)
- `service_name` - Same as container name
- `container_id` - Docker container ID
- `stream` - stdout or stderr

#### Airsonic Logs (all have these)
- All labels from "All Docker Logs" above
- `job="airsonic"` - Always present

#### Airsonic Stream Logs (subset)
- All labels from "Airsonic Logs" above
- `client_ip` - Extracted IP address
- `log_type="stream"` - Indicates streaming activity

#### Airsonic Cache Logs (subset)
- All labels from "Airsonic Logs" above
- `artist` - Extracted artist name
- `log_type="cache"` - Indicates cache activity

#### Traefik Access Logs
- `log_type="access"`
- `job="traefik"`
- Extracted JSON fields available (but not as labels): `client_ip`, `method`, `path`, `status`


* * *

### Key Differences from Label-Based Discovery

**What this config does NOT use:**
- Docker labels like `alloy.job=airsonic`
- `local.file_match` with Docker discovery targets
- Label-based filtering via `__meta_docker_label_*`

**What this config DOES use:**
- Container name matching (`/airsonic-advanced`)
- `discovery.relabel` with regex filtering
- `loki.source.docker` directly (not file-based)
- Multi-stage processing within `loki.process` blocks

This approach is simpler for single-container targeting but less flexible if you want to label multiple containers for the same pipeline.





## Grafana Alerting

With data, you can setup alerts. Now that we have music playing, we can setup alerts.

Those alerts are already setup for us, using the variables from the .env file. 


* * *

### What is Grafana Alerting

Think of Grafana Alerting as a security guard for your computer. It watches your logs so you can sleep.

**1. The Rule (The Instructions)**
You give the guard a specific list of bad things to look for.

* *Example:* "Wake me up if the music stops."

**2. The Contact Point (The Phone Number)**
You tell the guard where to send the message when they find trouble.

* *Example:* "Send a message to my Telegram app."

**3. The Policy (The Filter)**
You tell the guard how often to bother you.

* *Example:* "Tell me immediately, but if it keeps happening, only remind me every 10 minutes."


* * *

### Notification-Policy.yaml

Here is the explanation for the **Notification Policy**.

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

### WHAT THIS CODE DOES FOR GRAFANA

* **Routing Logic:** It tells Grafana: "When an alert fires, send it to `MusicAlert_bot`."
* **Grouping:** The `group_by` setting ensures that if multiple rules fire at once (e.g., both music servers start playing), they are grouped into a single notification block rather than spamming you with individual messages.
* **Timing Controls:**
* `group_wait: 0s`: Send the alert immediately; don't wait to see if more come in.
* `repeat_interval: 10m`: If the condition persists (music keeps playing), remind me every 10 minutes.


### WHAT THIS CODE DOES FOR THE PROJECT

* **Noise Reduction:** It prevents your phone from buzzing constantly. You get one notification when the music starts, and reminders only if it continues for a long time.
* **Organization:** It groups alerts by their name and folder, keeping your Telegram chat clean and readable.

### DIFFERENT WAYS TO EXPAND OR USE

* **Mute Timings:** Add a `mute_timing` to silence notifications during sleeping hours (e.g., 10 PM - 8 AM).
* **Severity Routing:** You can add nested policies to send "Critical" alerts to Telegram (with sound) and "Warning" alerts to email (silent).


* * *

[Grafana Notification Policy Documentation](https://grafana.com/docs/grafana/latest/alerting/configure-notifications/create-notification-policy/)

* * *


* * *

### ContactPoint-Telegram.yaml

Here is the explanation for the **Telegram Contact Point**.

```yaml
apiVersion: 1
contactPoints:
    - orgId: 1
      name: MusicAlert_bot
      receivers:
        - uid: bsdyv8574b6v
          type: telegram
          settings:
            bottoken: ${MYTGRAM_BOTTOKEN}
            chatid: >
              ${MYTGRAM_CHATID}
            disable_notification: false

```

### WHAT THIS CODE DOES FOR GRAFANA

* **Defines a Receiver:** It creates a destination named `MusicAlert_bot` within Grafana's Alerting system.
* **Integrates Telegram:** It configures the specific Telegram API driver using the `bottoken` and `chatid` variables you defined in your `.env` file.
* **Sets Preferences:** `disable_notification: false` ensures that messages sent to your phone will trigger a sound/vibration (they won't be silent).

### WHAT THIS CODE DOES FOR THE PROJECT

* **Mobile Alerts:** This is the bridge that moves alerts off your server screen and into your pocket.
* **Security:** By using `${MYTGRAM_...}` variables, you are provisioning this sensitive configuration without hardcoding your API tokens into the file.

### DIFFERENT WAYS TO EXPAND OR USE

* **Multiple Channels:** You can add a `discord` or `slack` receiver under the same contact point to send the notification to multiple apps simultaneously.
* **HTML Formatting:** You can enable `parse_mode: HTML` in the settings to send bold or colored text messages to Telegram.

* * *

[Grafana Configure Telegram Documentation](https://grafana.com/docs/grafana/latest/alerting/configure-notifications/manage-contact-points/integrations/configure-telegram/)

* * *


* * *

## LyrionAlert.json

Here is the explanation for the **Lyrion Music Alert (Regex-Based)**.


```sql
// LogQL Query inside the JSON
sum by (title, artist) (
  rate({container=~"(?i)(lyrionmusicserver|lms)"}
    |= "currently playing"
    |~ `(?i)(turd|poop|butt|fart)`
    | regexp `currently playing "(?P<title>[^\t]+)\t(?P<url>[^\t]+)\t(?P<artist>[^\t]+)\t`
  [3s])
)

```

### WHAT THIS CODE DOES FOR GRAFANA

* **Filters Stream:** Selects logs from containers named `lyrionmusicserver` or `lms` (case-insensitive).
* **Keywords Filter:** It specifically looks for log lines containing the words "turd", "poop", "butt", or "fart". (Likely looking for joke songs or specific metadata matches).
* **Regex Parsing:** It uses `regexp` to extract the `title` and `artist` from the raw log line dynamically at query time, because Lyrion logs are unstructured text.

### WHAT THIS CODE DOES FOR THE PROJECT

* **Content Detection:** It triggers an alert specifically when songs with these keywords are played on your Logitech Media Server.
* **Raw Log Parsing:** Unlike Airsonic (which we enriched in Alloy), this demonstrates how to parse "messy" logs directly in Grafana without pre-processing them.

### DIFFERENT WAYS TO EXPAND OR USE

* **Alert on Errors:** Change `|= "currently playing"` to `|= "error"` to detect playback failures instead of specific song titles.
* **Fix Regex:** The regex assumes a tab-separated format (`\t`). If Lyrion updates its log format, this alert will break and need adjustment.

* * *

[Grafana LogQL Regex Documentation](https://grafana.com/docs/loki/latest/query/log_queries/#regular-expression)

* * *


* * *

## AirSonicAlert.json

Here is the explanation for the **Airsonic Music Alert (Label-Based)**.

### CODE EXCERPT

```alloy
// LogQL Query inside the JSON
sum by (asonic_music) (
  rate({job="airsonic"}
    | asonic_music =~ `(?i)(turd|butt|Boat Club|Rick Astley|Nickelback)`
  [3m])
)

```

### WHAT THIS CODE DOES FOR GRAFANA

* **Uses Pre-Computed Labels:** Instead of complex regex, it uses the `asonic_music` label that you created earlier in your Alloy configuration.
* **Artist Matching:** It checks if that label matches the regex list (Rick Astley, Nickelback, etc.).
* **Threshold:** The alert condition checks if this rate is `> 0`, meaning "Is this artist playing right now?".

### WHAT THIS CODE DOES FOR THE PROJECT

* **"Rickroll" Detector:** This alert notifies you immediately if someone plays Rick Astley or Nickelback on your server.
* **Performance:** This query is much faster and lighter on the server than the Lyrion one because Alloy did the hard work of parsing the artist name *before* storing the log.

### DIFFERENT WAYS TO EXPAND OR USE

* **Add Artists:** Simply add `|Taylor Swift` to the list to expand your monitoring.
* **User Tracking:** Add `by (asonic_user)` to the `sum` clause to see *who* is playing these songs in the alert message.

* * *

[Grafana LogQL Label Filter Documentation](https://grafana.com/docs/loki/latest/query/log_queries/#label-filter-expression)

* * *


* * *

## YAML tirade 

To get a string working in YAML, it's a little tricky. `chatid` is required to be a string, but it is a number. So just passing an env_var in, isnt correct.


* * *

### The Solution

Use YAML multiline syntax to force this as a string:

```yaml
yamlapiVersion: 1
contactPoints:
    - orgId: 1
      name: MusicAlert_bot
      receivers:
        - uid: nj9087ns47
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


* * *

### What the Scalar Solution Does

Why this works:

- `The Indentation`: By indenting ${MYTGRAM_CHATID}, the YAML parser treats it as the value of chatid.

- `The Folded Scalar (>)`: The > symbol tells YAML: "Everything indented below this is a string."

- `No Quotes`: Since you aren't using literal " or ' marks, Grafana receives the raw digits but treats them as a string type.

This forces Grafana to treat it as a string instead of a number.


* * *

### Why This Bug Exists

Grafana's YAML parser expands environment variables before type checking, so numbers - even with quotes around them - get parsed as numeric types instead of strings. This is a known bug (#69950) that's been open since June 2023.




