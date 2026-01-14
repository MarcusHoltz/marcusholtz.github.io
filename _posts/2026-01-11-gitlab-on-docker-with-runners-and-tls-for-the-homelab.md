---
layout: post
title: Homelab Optimized GitLab Omnibus with Runner and TLS
description: Get up and running in less than 10 min with a homelab optimized docker GitLab omnibus with TLS and runner for CI/CD.
date: 2026-01-11 11:33:00 -0700
categories: [DevOps, Monitoring]
tags: [homelab, gitops, cicd, selfhosted, gitlabrunner, gitlab, traefik, docker, fun, project, guide, applications]
pin: false
image:
  path: /assets/img/header/header--gitlab--self-hosted-deployment-stack.jpg
  alt: GitLab Omnibus and Runners can be a pain, get your own self-hosted DevOps platform running in this guide!
---

# Gitlab Up and Running on Docker with Runners and TLS in the Homelab

GitLab Omnibus is a massive "all-in-one" platform that bundles databases, web servers, and task runners into a single package.

Getting your own self-hosted DevSecOps platform running doesn't have to be a headache. 

This guide will get you a professional-grade **GitLab CE** instance, secured with **TLS (HTTPS)** via Traefik, a pre-configured **GitLab Runner**, and a **CI/CD pipeline** project, in under 15 minutes.


* * *

## The Architecture at a Glance

Before we dive in, here is how the traffic flows through your stack:

* **Traefik:** Acts as the traffic cop, handling SSL termination and routing.
* **Certbot:** Automatically fetches Wildcard certificates via Cloudflare DNS.
* **GitLab CE:** The core application, running on an internal Docker network.
* **GitLab Runner:** Automatically registers itself to your instance using a helper script.


* * *

## Step 1: Prepare Your Environment

You need to tell the stack who you are. The `.env` file is your single source of truth.

1. **Configure `.env`:** Open your `.env` file and update these key fields:
* `DOMAIN_NAME`: Your domain (e.g., `example.com`).
* `GITLAB_SUBDOMAIN`: Usually `gitlab`.
* `ACME_EMAIL`: Your email for Let's Encrypt alerts.


2. **Set the Root Password:** To keep things secure, we use Docker Secrets.
* Create a folder named `secrets`.
* Create a file inside called `gitlab_root_password.txt` and paste your desired admin password there.


* * *

## Step 2: Spin Up the Stack

With your configuration set, it's time to pull the trigger.

```bash
docker compose up -d

```

### What's happening?

* **Certbot** runs first to ensure your SSL certificates exist in `./appdata/certbot`.
* **Traefik** starts listening on ports 80 and 443.
* **GitLab** begins its boot sequence.

> **GitLab is heavy.** It can take 5–10 minutes to fully initialize. You can monitor the progress with `docker logs -f gitlab_ce`.


* * *

### Problems with docker compose

See the Help section at the end


* * *

## Step 3: Automate Runner Registration

Usually, registering a runner is a manual chore of copying tokens. We've automated this with the `3_register_runner.sh` script.

Once GitLab is healthy (you can reach the login page), run:

```bash
chmod +x 3_register_runner.sh
./3_register_runner.sh
```

**What this script does for you:**

1. **Wait:** It polls the GitLab API until it's actually ready.
2. **Auth:** It enters the GitLab container and generates a temporary Personal Access Token.
3. **Register:** It fetches a Runner Registration Token and links the `gitlab-runner` container to your instance.
4. **Connect:** It configures the runner to use the Docker executor, allowing it to run CI/CD jobs.


* * *

## Step 4: Login and Verify

1. Navigate to `https://gitlab.yourdomain.com`.
2. Log in with username **root** and the password you put in your secrets file.
3. Go to **Admin Area > CI/CD > Runners**. You should see your `homelab-hybrid-runner` online and ready!


* * *

## Step 5: Your First Project

Go to the cidi ddididid directory.

And then 


Click **"New Project"** > **"Create blank project."**

```bash
git remote add origin https://<your-domain>/<user>/<repo>.git
git push -u origin main
```


To move an existing project into your new instance, use these standard Git commands:

1. **Initialize local Git:** `git init` (if you haven't already).


2. **Add your new home:** `git remote add origin https://your-gitlab-url.com/username/project.git`.


3. **Push everything:** `git push -u origin main`.






## Step 6: Weather CI/CD Demo

Once you pushed to a new repo you should see some new files in there.

This project demonstrates GitLab CI/CD pipelines with interactive user input.

There should be some image instructions to go along with this here.


* * *

## Getting Started

1. **Set up your API key**:
   - Get a free API key from [OpenWeatherMap](https://openweathermap.org/api)
   - Add it in **Settings → CI/CD → Variables**
   - Click **Add variable**
   - Enter the **Key** as `WEATHER_API_KEY`
   - Enter the **Value** as `Your API key`
   - Save changes

2. **Run your first pipeline**:
   - Go to **Build → Pipelines**
   - Click **New Pipeline**
   - Fill out the form with your desired location
   - Click **New Pipeline**

3. **Watch the magic happen**:
   - See real-time logs
   - Watch as nothing happens
   - The 'main' branch has manual builds
   - To fix: Make a new branch below
   - Or click: The stuck job card or **run** (play) button in your current pipeline

4. **Make a new Branch**:
   - To push a new README.md automatically to the repository
   - You need to make a new branch not named, 'main' or 'webdav'
   - Go to **Code → Branches**
   - Click **New Branch**
   - Fill out the form with your desired branch name **(be sure to 'create from' `main`)**
   - Click **Create Branch**

5. **Run a pipeline in a branch**:
   - Go to **Build → Pipelines**
   - Click **New Pipeline**
   - Look in the upper left hand corner of this form
   - **Run for branch name or tag**
   - Select the branch you made
   - Fill out the form with your desired location
   - Click **New Pipeline**

6. **Watch your README.md change**:
   - See real-time logs
   - Visit your README.md for the changes





















## Help






## 1. Ok, now run the compose file

Deploy the stack using `docker compose up -d`.

**The Novice Trap: The "502 Bad Gateway" Panic**
Immediately after running this, you will likely see a 502 error in your browser. **Don't panic.** GitLab is an "Omnibus" package containing a database, a registry, and a web server; it takes 5–10 minutes to run its internal configuration scripts. If you restart the container during this phase, you risk corrupting the database.

**Understanding Your Config:**

* **External URL:** Your `external_url` in the config must start with `https://`. Even though Traefik handles the encryption, GitLab needs to know it is being served via HTTPS to generate the correct internal links.


* **Port 80 vs 443:** You’ll see `nginx['listen_https'] = false`. This is intentional. Traefik catches the secure traffic on port 443 and passes it to GitLab’s internal port 80. If you enable HTTPS inside GitLab too, you’ll get a port conflict and the deployment will fail.


* **Port 22 Conflict:** If your host machine already uses port 22 for SSH (it usually does), your GitLab SSH will fail. Check if your config maps port 22 to a different host port (like `2424:22`) to avoid this.



## 2. Super, now run the runner setup script

Now that the instance is alive, run your setup script to connect the "worker" (the Runner).

**The Novice Trap: The "410 Gone" Error**
As of 2025, GitLab has deprecated "Registration Tokens." If you try to use a static token from an old tutorial, you will get a `410 Gone` error.

* **The Fix:** You must create the Runner in the UI first (**Admin > CI/CD > Runners**) to get an **Authentication Token** (prefixed with `glrt-`).



**What the script does for you:**

* **Docker Executor:** It sets the executor to `docker`. This ensures every build job runs in a clean, isolated container.


* **The Socket Mount:** It mounts `/var/run/docker.sock`. This is high-value because it lets your runner spin up other containers, but it means the runner has root-level power over your server. Keep this instance private!



## 3. Login

You need your password, and it wasn't set in the compose file.

**Retrieve your temporary key:**
GitLab generates a random password on boot. Run this command to see it:
`docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password`.

**Trap:** This file is automatically deleted after 24 hours. Login and change your password in **User Settings** immediately.

## 4. Cool, create your first repo

Click **"New Project"** > **"Create blank project."**

* **Novice Tip:** If you're using this for internal tools, set the visibility to **Private**.
* **Novice Tip:** Don't skip the "Initialize with a README" checkbox; it makes the next step easier to verify.

## 5. Now connect the gitlab-ci repo

Connect your local code to your new instance using these commands on your screen:

```bash
git remote add origin https://<your-domain>/<user>/<repo>.git
git push -u origin main
```

**The Novice Trap: SSL Verification Fails**
If your local machine doesn't trust your Let's Encrypt cert yet (or if you are using self-signed certs), your `git push` might fail. 
*   **The Fix:** Ensure your domain is fully resolved in DNS. If you are on the same network as the server, you may need to add your domain to your local `/etc/hosts` file to bypass "hairpin NAT" issues.[3, 12]








### Step 1: Fire Up the Compose File

First, run your `docker-compose.yml`. This file is the "blueprint" for your infrastructure. Here is what is happening under the hood for a novice user:

* **Traefik as the Gateway:** Instead of GitLab managing its own SSL, Traefik sits in front. It talks to **Let's Encrypt** to get your HTTPS certificates automatically.


* **The External URL:** In your config, `external_url` is set to `https://your-domain.com`. Even though Traefik handles the security, GitLab needs to know its public name to generate correct links for your repositories.
* **Internal Networking:** You’ll notice `nginx['listen_https'] = false` and port `80`. This tells GitLab to stay "simple" internally while Traefik handles the "secure" (HTTPS) traffic from the outside world.


* **Persistent Data:** We map volumes (like `/var/opt/gitlab`) to your host machine. This ensures that if the container restarts, your code and users are still there.

### Step 2: Run the Runner Setup Script

GitLab Runners are the "workers" that actually run your code tests and builds.

The setup script uses the **new 2025 workflow**. Gone are the days of old "registration tokens." We now use **Authentication Tokens** (starting with `glrt-`) which are more secure and easier to manage across multiple machines.

When you run the registration command:

1. It connects to your new GitLab instance.
2. It identifies itself as a **Docker Executor**.


3. It binds to the `docker.sock`. This is high-value: it lets the runner spin up temporary containers to test your code in a clean environment every time.



### Step 3: Login (The "Where is my password?" moment)

GitLab no longer asks you to set a password on the first screen. For security, it generates a random one and hides it inside the container.

To find it, run:
`docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password`.

**Note:** This file is deleted after 24 hours, so your first task should be changing the password in the **User Settings**.

### Step 4: Create Your First Repo

Once you are in, click the **"New Project"** button and select **"Create blank project."**

* **Visibility:** Choose "Private" if you're not ready for the world to see your code.


* **Initialize:** You can initialize with a README to see it live immediately.



### Step 5: Connect Your Code

To move an existing project into your new instance, use these standard Git commands:

1. **Initialize local Git:** `git init` (if you haven't already).


2. **Add your new home:** `git remote add origin https://your-gitlab-url.com/username/project.git`.


3. **Push everything:** `git push -u origin main`.

























































































































































