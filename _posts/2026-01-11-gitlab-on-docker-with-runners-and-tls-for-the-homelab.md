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

To find the repository that goes along with this guide, visit:

[github.com/MarcusHoltz/docker-gitlab-runner](https://github.com/MarcusHoltz/docker-gitlab-runner)


* * *

## The Architecture at a Glance

Before we dive in, here is how the traffic flows through this stack:

- **Traefik:** Acts as the traffic cop, handling SSL termination and routing.

- **Certbot:** Automatically fetches Wildcard certificates via Cloudflare DNS.

- **GitLab CE:** The core application, running on an internal Docker network.

- **GitLab Runner:** Automatically registers itself to your instance using a helper script.

- **WeatherCICD:** This is the gitlab-ci.yml pipeline that we will run once the project is complete.


* * *

## 1). Configure the Project Environment to Your Requirements

You need to tell the stack a few details. 

- `./certbot/cloudflare.ini` - This file tells Certbot your Cloudflare API Token for DNS-01

- `./secrets/gitlab_root_password.txt` - This file contains our intial password to login to GitLab as root

- `.env` - The `.env` file is where we place every other non-secret value. It contains all the customization within our script.


* * *

### Edit ./certbot/cloudflare.ini

The first step is to leave this write-up and go to another website.

Cloudflare can provide a nameserver for almost any domain, allowing us to use Cloudflare's API to create temporary TXT records for SSL domain validation.

For help with creating a Token in Cloudflare, visit [Cloudflare's docs on creating a token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/) for a quick how-to.

Once you have the token,

- Go inside the `cerbot` folder

- Edit the `cloudflare.ini` file

- Find `dns_cloudflare_api_token`

- Replace `YOUR_CLOUDFLARE_API_TOKEN_HERE` with your token

- Save and quit the file

> Ensure the token has "Zone:DNS:Edit" permissions for your domain.


* * *

### Edit ./secrets/gitlab_root_password.txt

To keep passwords secure, we will use Docker Secrets.

- Go inside the folder named `secrets`

- Edit a file inside called `gitlab_root_password.txt`

- You should see a placeholder password, `HEYOUchangeThisPassword`

- Remove that text and enter your password (your password should be the only text in the file)

- Save and quit the file

> Make sure to paste just your GitLab root password


* * *

### Edit .env

The .env file is were we store everything we want to configure in all of our files (outside of our secrets).

This allows us to make changes in one place throughout the entire project, but keep our keys, tokens, passwords, and secrets safe somewhere else.

- Open your `.env` file

- You must atleast update the following fields:
  - `DOMAIN_NAME`: Your domain (e.g., `example.com`).
  - `GITLAB_SUBDOMAIN`: Usually `gitlab`.
  - `GITLAB_HOST_IP`: The IP address of your Docker host running GitLab

- There are many more fields, change a few more and you may just break something - Good Luck!


* * *

## 2). Spin Up the Stack

With configuration complete, you can bring up the Docker compose project stack.

```bash
docker compose up -d
```


* * *

### What's happening?

* **Certbot** runs first to ensure your SSL certificates exist in `./appdata/certbot`.

* **Traefik** starts listening on ports 80 and 443, but will need rebooted to find the certificates.

* **GitLab** begins its boot sequence.


* * *

#### What can I do?

I would say, to let everything get everything settled, run this command and walk away:

`docker compose up -d && sleep 180 && docker compose up -d && sleep 270 && docker logs -f gitlab_ce`

> **GitLab is heavy.** It can take 5–10 minutes to fully initialize. You can monitor the progress with `docker logs -f gitlab_ce`.


* * *

#### Problems?

See the [Help section](#help) at the end.


* * *

## 3.) Runner Registration Script

Usually, registering a runner is a manual chore of copying tokens. I have automated this with the [register_gitlab_runner.sh](https://github.com/MarcusHoltz/docker-gitlab-runner/blob/main/register_gitlab_runner.sh) script.

Once GitLab is healthy (you can reach the login page), run:

```bash
register_gitlab_runner.sh
```


* * *

### What the register_gitlab_runner.sh script does for you

- **Wait:** It polls the GitLab API until it's actually ready.

- **Auth:** It enters the GitLab container and generates a temporary Personal Access Token.

- **Register:** It fetches a Runner Registration Token and links the `gitlab-runner` container to your instance.

- **Connect:** It configures the runner to use the Docker executor, allowing it to run CI/CD jobs.


* * *

### Login to GitLab and Verify Runner

- Navigate to `https://gitlab.yourdomain.com`.

- Log in with username **root** and the password you put in your secrets file.

- Go to **Admin Area > CI/CD > Runners**. You should see your `homelab-hybrid-runner` online and ready!


* * *

## 4). Your First Project

With your recent sucess of logging into GitLab, we should do something with it.

I have provided a gitlab-ci pipeline to do exactly that!

- Login to GitLab, if not already

- In the upper right hand corner of the screen is a `+` icon, click it

- In this new GitLab menu, click `New project/repository`

- On the new screen click on `Create blank project`

- Enter a `project name`

- Under the **Project URL** use the drop down for `Pick a group or namespace` to select an option (probably just `root`)

- Under **Project Configuration** `uncheck` - Initialize repository with a README

- Click on `Create project`

- On this new screen, with your newly minted repo, head down to the `Add files`

- Click on `HTTPS`

- We want to `Configure the Git repository` for our **WeatherCICD** folder, copy and paste these commands somewhere


* * *

### Git Push the WeatherCICD

With a new reposity to hold our files, we can put the [WeatherCICD](https://github.com/MarcusHoltz/docker-gitlab-runner/tree/main/WeatherCICD) into GitLab.

- Open the folder in the [docker-gitlab-runner](https://github.com/MarcusHoltz/docker-gitlab-runner/tree/main) repository under [WeatherCICD](https://github.com/MarcusHoltz/docker-gitlab-runner/tree/main/WeatherCICD).

- Once inside the `WeatherCICD` folder

- Here you can use the commands we copied from your new repository

- They should look like

```bash
git init --initial-branch=main --object-format=sha1
git remote add origin https://<your-domain>/<user>/<repo>.git
git add .
git commit -m "Initial commit"
git push --set-upstream origin main
```

> Once you git pushed - you should see your new files in GitLab.


* * *

## 5). Weather CI/CD Demo

This project demonstrates GitLab CI/CD pipelines with interactive user input.


* * *

### Getting Weather CI/CD Demo Running

1. **Set up your API key**:
   - Get a free API key from [OpenWeatherMap](https://openweathermap.org/api)
   - Add it in **Settings → CI/CD → Variables**
   - Click **Add variable**
   - Enter the **Key** as `WEATHER_API_KEY`
   - Enter the **Value** with the free API key you got from OpenWeatherMap
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


* * *

## Help


* * *

### 1. Ok, now run the compose file

Deploy the stack using `docker compose up -d`.

**The Novice Trap: The "502 Bad Gateway" Panic**
Immediately after running this, you will likely see a 502 error in your browser. **Don't panic.** GitLab is an "Omnibus" package containing a database, a registry, and a web server; it takes 5–10 minutes to run its internal configuration scripts. If you restart the container during this phase, you risk corrupting the database.

**Understanding Your Config:**

* **External URL:** Your `external_url` in the config must start with `https://`. Even though Traefik handles the encryption, GitLab needs to know it is being served via HTTPS to generate the correct internal links.


* **Port 80 vs 443:** You’ll see `nginx['listen_https'] = false`. This is intentional. Traefik catches the secure traffic on port 443 and passes it to GitLab’s internal port 80. If you enable HTTPS inside GitLab too, you’ll get a port conflict and the deployment will fail.


* **Port 22 Conflict:** If your host machine already uses port 22 for SSH (it usually does), your GitLab SSH will fail. Check if your config maps port 22 to a different host port (like `2424:22`) to avoid this.


* * *

### 2. Super, now run the runner setup script

Now that the instance is alive, run your setup script to connect the "worker" (the Runner).

**The Novice Trap: The "410 Gone" Error**
As of 2025, GitLab has deprecated "Registration Tokens." If you try to use a static token from an old tutorial, you will get a `410 Gone` error.

* **The Fix:** You must create the Runner in the UI first (**Admin > CI/CD > Runners**) to get an **Authentication Token** (prefixed with `glrt-`).

**What the script does for you:**

* **Docker Executor:** It sets the executor to `docker`. This ensures every build job runs in a clean, isolated container.

* **The Socket Mount:** It mounts `/var/run/docker.sock`. This is high-value because it lets your runner spin up other containers, but it means the runner has root-level power over your server. Keep this instance private!


* * *

### 3. Login

You need your password, and it wasn't set in the compose file.

**Retrieve your temporary key:**
GitLab generates a random password on boot. Run this command to see it:
`docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password`.


* * *

### 4. Cool, create your first repo

Click **"New Project"** > **"Create blank project."**

* **Novice Tip:** If you're using this for internal tools, set the visibility to **Private**.
* **Novice Tip:** Don't skip the "Initialize with a README" checkbox; it makes the next step easier to verify.


* * *

### 5. Now connect the gitlab-ci repo

Connect your local code to your new instance using these commands on your screen:

```bash
git remote add origin https://<your-domain>/<user>/<repo>.git
git push -u origin main
```

- **The Novice Trap: SSL Verification Fails**

If your local machine doesn't trust your Let's Encrypt cert yet (or if you are using self-signed certs), your `git push` might fail. 

- **The Fix:** 

Ensure your domain is fully resolved in DNS. If you are on the same network as the server, you may need to add your domain to your local `/etc/hosts` file to bypass "hairpin NAT" issues.


* * *

### Expanded Troubleshooting:


* * *

#### Step 1: Fire Up the Compose File

First, run your `docker-compose.yml`. This file is the "blueprint" for your infrastructure. Here is what is happening under the hood for a novice user:

* **Traefik as the Gateway:** Instead of GitLab managing its own SSL, Traefik sits in front. It talks to **Let's Encrypt** to get your HTTPS certificates automatically.

* **The External URL:** In your config, `external_url` is set to `https://your-domain.com`. Even though Traefik handles the security, GitLab needs to know its public name to generate correct links for your repositories.

* **Internal Networking:** You’ll notice `nginx['listen_https'] = false` and port `80`. This tells GitLab to stay "simple" internally while Traefik handles the "secure" (HTTPS) traffic from the outside world.

* **Persistent Data:** We map volumes (like `/var/opt/gitlab`) to your host machine. This ensures that if the container restarts, your code and users are still there.


* * *

#### Step 2: Run the Runner Setup Script

GitLab Runners are the "workers" that actually run your code tests and builds.

The setup script uses the **new 2025 workflow**. Gone are the days of old "registration tokens." We now use **Authentication Tokens** (starting with `glrt-`) which are more secure and easier to manage across multiple machines.

When you run the registration command:

1. It connects to your new GitLab instance.

2. It identifies itself as a **Docker Executor**.

3. It binds to the `docker.sock`. This is high-value: it lets the runner spin up temporary containers to test your code in a clean environment every time.


* * *

#### Step 3: Login (The "Where is my password?" moment)

GitLab no longer asks you to set a password on the first screen. For security, it generates a random one and hides it inside the container.

To find it, run:
`docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password`.

**Note:** This file is deleted after 24 hours, so your first task should be changing the password in the **User Settings**.


* * *

#### Step 4: Create Your First Repo

Once you are in, click the **"New Project"** button and select **"Create blank project."**

* **Visibility:** Choose "Private" if you're not ready for the world to see your code.

* **Initialize:** You can initialize with a README to see it live immediately.


* * *

#### Step 5: Connect Your Code

To move an existing project into your new instance, use these standard Git commands:

1. **Initialize local Git:** `git init` (if you haven't already).

2. **Add your new home:** `git remote add origin https://your-gitlab-url.com/username/project.git`.

3. **Push everything:** `git push -u origin main`.



