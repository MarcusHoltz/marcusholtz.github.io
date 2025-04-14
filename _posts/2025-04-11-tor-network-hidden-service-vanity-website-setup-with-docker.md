---
layout: post
title: Tor network hidden service with vanity .onion address using Docker
date: 2025-04-11 11:33:00 -0700
categories: [DevOps, Tor]
tags: [tor, opensource, devops, tor, reverseproxy, onion, docker, security, networking, script, privacy]
pin: false
image:
  path: /assets/img/header/header--tor--hidden-services-onion-address-docker.jpg
  alt: Hosting a vanity .onion address on the Tor network using Docker
---

# Using Tor script to create a Tor network hidden service with vanity .onion address and export a service to the internet

## Tor is not hidden

"In our experiments we collected 173667 unique .onion addresses in all using a single Amazon EC2 instances in 1 hour and found 4857 hidden web services online."

- [Out-of-band discovery and evaluation for tor hidden services](https://dl.acm.org/doi/10.1145/2851613.2851798)

- [Tor: Hidden Service Intelligence Extraction](https://rp.os3.nl/2017-2018/p98/report.pdf)


### Hidden Service Discovery

If you were to create a brand-new Tor v3 address, its address is not inherently public, but you might have little time before you're discovered:

- Bots & Crawlers: Hours to a day. Automated services scanning .onion addresses might identify the service pretty quickly.

- Public Discovery (like human users or public listings): Days to weeks, depending on whether the address is actively shared or found.

This is just an estimate, and the actual time can vary based on scanning frequency, network traffic, and how indexed your .onion address becomes.



## Tor to host an .onion service

Now, with the warnings in place, we can begin.

The purpose of this repo/post is to give someone a chance to test out hosting an .onion hidden service.

You can use this to quickly share a service to a friend, client, or even your future self.

> A Tor hidden service does not need your server to have open ports or port forwarding - because it does not accept direct inbound connections from the public internet. Instead, both the client and the hidden service connect outbound to the Tor network, establishing circuits to special relays called introduction and rendezvous points. All communication is routed through these Tor relays, so as long as your server can make outbound connections to the Tor network, it can host a hidden service!
{: .prompt-tip }

* * *

## Tor Hidden Service Tutorial


1. [This Intro](#tor-to-export-a-service-to-the-internet)

2. [1-up Tor Script](#1-up-tor-script)

   1. [Changes the Script makes](#changes-the-script-makes)

3. [Vanity Name Creation](#vanity-name-creation)

   1. [Vanity Name Length](#vanity-name-length)

   2. [Example vanities](#example-vanities)

   3. [How is the vanity generated](#how-is-the-vanity-generated)

4. [What Service to put on Tor](#what-service-to-put-on-tor)

5. [Torrc is important](#torrc-is-important)

6. [The 1-up Tor Script uses two directories](#the-1-up-tor-script-uses-two-directories)

7. [Browsers that find an onion service](#browsers-that-find-an-onion-service)

8. [Want to know more?](#want-to-know-more)

9. [Uninstall](#uninstall)


* * *

## 1-up Tor Script

This script sets up **one** service that will be available through a [Tor .onion address](https://en.wikipedia.org/wiki/.onion).

> This service is only available through the Tor network
{: .prompt-info }

This is intended as a demonstration. I hope you're able to learn and enjoy using.

[Some link to a Github repo that has the script](#)

```bash

Totally has the commands you need to quickly spin up this demonstration here

```
![1-up Tor Onion Address Script for a Tor Hidden Service](https://raw.githubusercontent.com/MarcusHoltz/marcusholtz.github.io/refs/heads/main/assets/img/posts/1-up-tor-script-hidden-onion-service.gif)

* * *

### Script Requirements

This script will need sudo. It is required to set all of the directory permissions correctly. 

You will then need docker installed to run the `docker-compose.yml` file that starts up Tor.

The script is only intended to prepare the environment we're using with docker.



* * *

### Changes the Script makes

The script sets up two directories, a file, and optionally a vanity address.


#### Two Directories

You need the sudo for privs for:

- tor_config/vanity_keys/

and

- tor_data/

> Those directories store the keys that are used for your .onion address. Kept safe from any normal user.



#### A file: torrc

The [torrc file](https://support.torproject.org/glossary/torrc/) contains all the settings Tor uses. 

You need the sudo for privs for:

- tor_config/torrc

> By changing this file we can tell Tor what services we want to serve on a Tor Hidden Service and where to find the corresponding .onion address.


* * *

## Vanity Name Creation

A [vanity address](https://community.torproject.org/onion-services/advanced/vanity-addresses/) is an onion address that starts with a pre-chosen number of characters, usually a meaningful name related to a specific Onion Service. 

For instance, one might try to generate an onion address for the mysitename website and end up with something looking like this:

`mysitenameyx4fi3l6x2gyzmtmgxjyqyorj9qsb5r543izcwymle.onion`

This has some advantages:

- It is easy for Onion Services users to know which site they are about to access.

- It has some branding appeal to site owners.

- It is easy for Onion Services operators to debug their logs and know which services have errors.

- Anyone else is very unlikely to come up with the exact key from the example above, but they may be able to find a similar key - one beginng with the same few letters. 

  - The longer the vanity name length, the less likly it is to have a forgery made.


* * *

### Vanity Name Length

You can only pick something, at max, 7 characters.

> Why?

Let's say you were running this on a Raspberry Pi 2B....

Take a look at the approximate generation time per character for a Raspberry Pi 2B below:

#### Approximate Generation Time per Character Count Chart

```text
Vanity Characters : Approximate Generation Time
1  : <1 second
2  : <1 second
3  : 1 second
4  : 30 seconds
5  : 16 minutes
6  : 8.5 hours
7  : 11.5 days
8  : 1 year
9  : 32 years
10 : 1,024 years
11 : 32,768 years
12 : 1 million years
```



* * *

### Example vanities

So now that we know our upper limit on the amount of letters we can have, take a look at some examples....

Click to expand and take a look at the 6 character example vanities below:

<details>

<summary>6 character example vanity .onion domains</summary>  

- 123456

- nopers

- online

- system

- search

- office

- forums

- mobile

- garden

- nature

- movies

- photos

- social

- future

- people

- estate

- energy

- income

- browse

- create

- report

- global

- agency

- potato

- attack

- wisdom

- stream

- viewer

- status

- screen

- sector

- survey

- secure

- signal

- source

- remote

- direct

- little

- jazzed

- dazzle

- danger

- school

- family
</details>


* * *

### How is the vanity generated

Thanks to the work on the [cathugger/mkp224o](https://github.com/cathugger/mkp224o) repository, we're able to generate vanity address for tor onion v3 (ed25519) hidden services.

- Specifically, the script will run: `docker run ghcr.io/cathugger/mkp224o:master -n 3 <your_vanity_name>`

- It will generate `3` .onion addresses that begin with your vanity name, allowing you to select a favorite.

- The .onion address will be in `tor_config/vanity_keys/`


### Can't I just use my own .onion address

Yes! This script will prompt you to use your own, you just have to provide the path.


#### Instructions for Using Bringing Your Own Vanity Tor Address:

1. Make sure you have all of your files for your .onion address in the same directory:

   - `hostname` - Contains your .onion address

   - `hs_ed25519_secret_key` - Your private key

   - `hs_ed25519_public_key` - Your public key


2. After the script completes, verify your hidden service is correct:

```bash

sudo cat tor_data/hidden_service/hostname

```


* * *

## What Service to put on Tor

You will also need a service to provide to the .onion address. 

This can be anything. It can be another docker container, a python web server on your laptop, your favorite IoT device, whatever!

You will just need to give the script:

- The IP or Hostname of the service you're sending to the Tor network.

- The port for the service to forward over the .onion address.

- ONLY ONE SERVICE!!!    --> tor_data/hidden_service/

> This script is designed for demonstration and as such, there's only one service designed into the script. You can always make multiple services on the same .onion address with different ports, or a new .onion address for every service. But today, only one service.


* * *

## Torrc is important

The `torrc` file lets you define `HiddenServiceDir` and `HiddenServicePort` directives, these tell Tor where to store your service you're sending to the Tor network's keys and what ports to forward, making your .onion site accessible.


* * *

## The 1-up Tor Script uses two directories

File permissions are critical for Tor hidden services:
   - Directories need 700 permissions (drwx------)
   - Key files need 600 permissions (-rw-------)
   - The docker container will adjusts these permissions for you

The Tor user (not root) must own all these files inside the container


* * *

## Browsers that find an onion service

- Use [Brave Browser](https://support.brave.com/hc/en-us/articles/360018121491-What-is-a-Private-Window-with-Tor-Connectivity)

- Use [Tor Browser](https://support.torproject.org/)



* * *

## Want to know more?

Want to know more about this script? How about a breakdown of the script's logic!

* * *

### Take a look at the flow of the script

<details>

<summary>Visual Script Breakdown</summary>  

```text

┌───────────────────────┐
│    check_sudo()       │
│  - Verify privileges  │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ create_directories()  │
│  - tor_config/        │
│  - tor_data/          │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  set_permissions()    │
│  - 755 config         │
│  - 700 data           │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│get_network_settings() │
│  - Collect:           │
│    • HOST_IP          │
│    • HOST_PORT        │
│    • VIRTUAL_PORT     │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│setup_vanity_address() │
└──────────┬────────────┘
           ├────────────────────────────┐
           ▼                            ▼
┌───────────────────────┐      ┌───────────────────────┐
│  Generate New Address │      │  Use Existing Keys    │
│  - mkp224o Docker     │      │  - Validate dir       │
│  - vanity name input  │      │  - Verify key files   │
└──────────┬────────────┘      └──────────┬────────────┘
           ▼                              ▼
┌───────────────────────┐      ┌───────────────────────┐
│  Select From Generated│      │ Copy Existing Keys    │
│  - Display options    │      │  - hostname           │
│  - Validate selection │      │  - secret_key         │
└──────────┬────────────┘      └──────────┬────────────┘
           └────────────┬─────────────────┘
                        ▼
┌─────────────────────────────────────────┐
│      setup_hidden_service_dir()         │
│  - Create hidden_service/               │
│  - Set 700 permissions                  │
└──────────┬──────────────────────────────┘
           ▼
┌───────────────────────┐
│   create_torrc()      │
│  - HiddenServicePort  │
│  - DataDirectory      │
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│  finalize_setup()     │
│  - Set file perms     │
│  - Display hostname   │
│  - Run instructions   │
└───────────────────────┘

```
</details>


* * *

## Copy and Paste the Tor Activation Script

This will need to be in my github repo and diff'd against local git repo.

<details>

<summary>Tor Service Activation Script</summary>  

```bash
#!/bin/bash

set -e

##########################################
## Sudo now and get Directories correct ##
##########################################

#  [main] -- This script cannot continue without sudo
check_sudo() {
    echo "Checking sudo privileges..."
    # Check if sudo is required (i.e., timestamp expired)
    sudo -v &> /dev/null || echo "Sudo password required"
}


#  [main] -- creating directory structure if it doesnt exist
create_directories() {
    echo "Creating directories..."
    sudo mkdir -p tor_config tor_data
}


#  [main] -- setting directory initial permissions
set_permissions() {
    echo "Setting initial permissions..."
    sudo chmod 755 tor_config  # Added sudo here
    sudo chmod 700 tor_data    # Added sudo here
}


#######################################################################
## Prompt for configuration - Frontend Port / Backend service to use ##
#######################################################################

#  [main] -- Function to collect configuration options from user
get_network_settings() {
    # Read-in IP address of the tor backend service
    echo "Please enter the IP address to forward traffic to [default: 127.0.0.1]: "
    read HOST_IP
    HOST_IP=${HOST_IP:-127.0.0.1}

    # Read-in destination port of backend service
    echo "What port on that IP address are you sending tor traffic to [default: 80]: "
    read HOST_PORT
    HOST_PORT=${HOST_PORT:-80}

    # Read-in what port people will use to connect your .onion address
    echo "What is the port for the .onion address people will be hitting on the tor network [default: 80]: "
    read VIRTUAL_PORT
    VIRTUAL_PORT=${VIRTUAL_PORT:-80}
}


####################################
## Setup an .onion vanity address ##
####################################

# Used in [setup_vanity_address] -- This is the work done to get the vanity address in place
generate_vanity_address() {

    # Create the directory for generated keys
    sudo mkdir -p tor_config/vanity_keys

    # Prompt the user for vanity name with length validation
    while true; do
        read -p "Enter a string (less than 7 characters) for your vanity address: " VANITY_NAME

        # Check the length of the input with warning about generation time
        if [[ ${#VANITY_NAME} -lt 7 ]]; then
            echo "Your onion address will begin with $VANITY_NAME"
            echo "Generating 3 addresses... this may take some time depending on the length."
            break
        else
            echo -e "\n-----------------------------------------------------------------\n7 Characters takes a week, and 7 months for 8 characters.\nPlease enter less than 7 characters.\n-----------------------------------------------------------------\n"
        fi
    done

    # Run the mkp224o Docker container to generate keys into the tor_config/vanity_keys directory
    # Using -n 3 to generate multiple addresses to choose from
    docker run --rm -v "$PWD/tor_config/vanity_keys:/keys" ghcr.io/cathugger/mkp224o:master -n 3 -d /keys "$VANITY_NAME"

    # Select an address header
    echo -e "\n-----------------------------------------------------------------\nSelect a vanity address to use:\n-------------------------------"

    # Get all the directories up in here
    mapfile -t onion_directories < <(sudo find tor_config/vanity_keys -mindepth 1 -maxdepth 1 -type d)

    # Check if any addresses were generated
    if [[ ${#onion_directories[@]} -eq 0 ]]; then
        echo "Error: No vanity addresses were generated. Please try again."
        exit 1
    fi

    # Display each directory with its hostname
    for i in "${!onion_directories[@]}"; do
        dir="${onion_directories[i]}"
        hostname=$(sudo cat "$dir/hostname" 2>/dev/null || echo "Unable to read hostname")
        echo "$((i + 1)). $hostname"
    done

    # Ask the user to select a directory
    read -p "Enter the number of the address you want to use: " choice

    # Check if the choice is valid or just give them whatever's available
    if [[ "$choice" -ge 1 && "$choice" -le ${#onion_directories[@]} ]]; then
        selected_directory="${onion_directories[$((choice - 1))]}"
        echo -e "You selected:\n$(sudo cat "$selected_directory/hostname")"
    else
        echo "Invalid selection. Using the first address."
        selected_directory="${onion_directories[0]}"
    fi

    # Double check - create hidden_service directory and set permissions
    setup_hidden_service_dir
    
    # Copy the key files from the chosen onion vanity directory - this requires sudo
    sudo cp "$selected_directory/hostname" tor_data/hidden_service/
    sudo cp "$selected_directory/hs_ed25519_secret_key" tor_data/hidden_service/
    sudo cp "$selected_directory/hs_ed25519_public_key" tor_data/hidden_service/

    # Inform user about the keys now in production
    echo -e "\n\n ::: DONE :::\n\n"; sleep 2;
    clear;
    echo -e "\nVanity address keys configured in:\n\033[1mtor_data/hidden_service/\033[0m"
}


######################################
## Setup an Existing .onion address ##
######################################

# Used in [setup_vanity_address] -- If user chooses to use EXISTING KEY, not a vanity key
use_existing_keys() {
    echo -e "Make sure this directory has allow permissions:\nEnter the directory path where your existing vanity keys are stored WITHOUT the trailing / \n e.g. /home/user/directory1/subdirectory "
    echo ""
    echo "Current directory: $(pwd)"
    echo "Items in current directory: $(ls -m)"
    echo ""
    echo "Please provide full path to folder with files for existing .onion address:"
    read VANITY_DIR

    # Validate the directory exists
    if [[ ! -d "$VANITY_DIR" ]]; then
        echo "Error: Directory $VANITY_DIR not found!"
        exit 1
    fi

# Check for required files with sudo permissions
if ! sudo test -f "$VANITY_DIR/hostname" || \
   ! sudo test -f "$VANITY_DIR/hs_ed25519_secret_key" || \
   ! sudo test -f "$VANITY_DIR/hs_ed25519_public_key"; then
    echo "Error: Missing required key files in $VANITY_DIR!"
    echo "Need: hostname, hs_ed25519_secret_key, and hs_ed25519_public_key"
    exit 1
fi


#    # Check for required files as long as this user has directory permissions
#    if [[ ! -f "$VANITY_DIR/hostname" ]] || [[ ! -f "$VANITY_DIR/hs_ed25519_secret_key" ]] || [[ ! -f "$VANITY_DIR/hs_ed25519_public_key" ]]; then
#        echo "Error: Missing required key files in $VANITY_DIR!"
#        echo "Need: hostname, hs_ed25519_secret_key, and hs_ed25519_public_key"
#        exit 1
#    fi

    # Double check - create hidden_service directory and set permissions
    setup_hidden_service_dir
    
    # Copy the EXISTING key files to the default 'hidden_service' directory - this requires sudo
    sudo cp "$VANITY_DIR/hostname" tor_data/hidden_service/
    sudo cp "$VANITY_DIR/hs_ed25519_secret_key" tor_data/hidden_service/
    sudo cp "$VANITY_DIR/hs_ed25519_public_key" tor_data/hidden_service/

    # Congratulate user about the keys now in production
    echo -e "\n\n ::: DONE :::\n\n"; sleep 2;
}


##########################
## Directory scafolding ##
##########################

# Used in [use_existing_keys]       -- prepare directory to recieve custom vanity .onion address
# Used in [generate_vanity_address] -- prepare directory for an existing key and .onion address
# Helper function to create and set permissions to the hidden_service directory
setup_hidden_service_dir() {
    sudo mkdir -p tor_data/hidden_service
    sudo chmod 700 tor_data/hidden_service
}


########################################
## Setup the Persistant Onion Address ##
########################################

#  [main] -- This is the function that runs all the .onion address key and hostname moving around
setup_vanity_address() {
    echo "Do you want to use a vanity Tor address? (y/n) [default: n]: "
    read USE_VANITY
    USE_VANITY=${USE_VANITY:-n}

    if [[ "$USE_VANITY" == "y" || "$USE_VANITY" == "Y" ]]; then
        echo "Do you want to generate a new vanity address or use existing keys? (generate/existing) [default: generate]: "
        read VANITY_OPTION
        VANITY_OPTION=${VANITY_OPTION:-generate}

        if [[ "$VANITY_OPTION" == "generate" ]]; then
            generate_vanity_address
        else
            use_existing_keys
        fi
    fi
}



#########################
## EDIT THE TORRC FILE ##
#########################

#  [main] -- Creates the torrc configuration - MAKE EDITS TO YOUR TORRC HERE !!!!!!!
create_torrc() {
    # Create torrc configuration if it doesn't exist - EDIT YOUR TORRC AFTER << EOF
    if [[ ! -f tor_config/torrc ]]; then
        echo -e "\nCreating new \033[1mtorrc\033[0m configuration...\n"
        # Edit your torrc here - under this text
        echo "# Tor configuration file
        DataDirectory /var/lib/tor
        
        # Hidden service configuration
        HiddenServiceDir /var/lib/tor/hidden_service/
        HiddenServicePort $VIRTUAL_PORT $HOST_IP:$HOST_PORT
        
        # Log configuration
        Log notice stdout
        
        # Dont promote to directory listings
        HiddenServiceVersion 3" | sudo tee tor_config/torrc > /dev/null

    # Explaining what happened with torrc
        echo -e "\n\033[1mtorrc\033[0m created with hidden service (port ${VIRTUAL_PORT}) pointing to ${HOST_IP}:${HOST_PORT}\n"
    else
        echo -e "\nUsing existing \033[1mtorrc\033[0m configuration.\n"
    fi
}



###########################
## Tor Adddress Printout ##
###########################

#  [main] -- Prints out reminders to the user on what may need to be done next and what was accomplished
finalize_setup() {
    # Set proper permissions for all files
    sudo chmod 700 tor_data/hidden_service 2>/dev/null || true
    sudo find tor_data/hidden_service -type f -exec chmod 600 {} \; 2>/dev/null || true

    # Display setup information
    echo -e "tor_config:\n\033[1m$(realpath tor_config)\033[0m"
    echo ""
    echo -e "tor_data:\n\033[1m$(realpath tor_data)\033[0m"
    echo ""
    echo -e "#########################\nYour onion address:\n\033[31m$(sudo cat tor_data/hidden_service/hostname 2>/dev/null || echo "No hostname file found")\033[0m\n#########################"
    echo ""
    echo ""
    echo -e "################################################\nConfirm your onion address after starting docker:\n\033[31msudo cat tor_data/hidden_service/hostname\033[0m\n################################################"
    echo ""
    echo " -->  Run tor with:   docker compose up -d"
}

################################################
## Bake your recipe - now with Docker Compose ## 
################################################

# #  [main] -- Run Docker Compose
# run_docker_compose() {
#     echo "Environment setup complete... running Docker"
#     docker compose up -d || docker-compose up -d
# }



################################
## Run the Functions in Order ##
################################

# Main function to orchestrate the entire process
main() {
    check_sudo
    create_directories
    set_permissions
    get_network_settings
    setup_vanity_address
    create_torrc
    finalize_setup
}

# Execute main function
main

# # Have a great day! :)
```
</details>


* * *

## Required Docker-Compose.yml file

<details>

<summary>Docker Compose to spinup Tor services</summary>  

```yaml
---
services:
  tor-hidden-service:
    image: alpine:latest
    container_name: tor-hidden-service
    restart: unless-stopped
    networks:  # Explicit network assignment
      - tor_network
    volumes:
      # Mount tor configuration directory to host
      - ./tor_config:/etc/tor
      # Mount tor data directory to host
      - ./tor_data:/var/lib/tor
    command: >
      sh -c "
        # Install tor from Alpine repositories
        apk add --no-cache tor &&
        
        # Ensure proper permissions for directories
        chmod 700 /var/lib/tor &&
        chown -R tor:tor /var/lib/tor &&
        
        # Make sure hidden service directory exists
        mkdir -p /var/lib/tor/hidden_service &&
        chmod 700 /var/lib/tor/hidden_service &&
        chown -R tor:tor /var/lib/tor/hidden_service &&
        
        # Make sure keys have proper permissions
        if [ -f /var/lib/tor/hidden_service/hs_ed25519_secret_key ]; then
          chmod 600 /var/lib/tor/hidden_service/hs_ed25519_secret_key &&
          chown tor:tor /var/lib/tor/hidden_service/hs_ed25519_secret_key;
        fi &&
        
        if [ -f /var/lib/tor/hidden_service/hs_ed25519_public_key ]; then
          chmod 600 /var/lib/tor/hidden_service/hs_ed25519_public_key &&
          chown tor:tor /var/lib/tor/hidden_service/hs_ed25519_public_key;
        fi &&
        
        # Set proper permissions for torrc
        chown root:tor /etc/tor/torrc &&
        chmod 644 /etc/tor/torrc &&
        
        # Run tor as the tor user
        su tor -s /bin/sh -c 'tor -f /etc/tor/torrc'
      "

networks:
  tor_network:
    driver: bridge
    attachable: true
    ipam:
      config:
        - subnet: 192.168.33.0/24  # Fixed private subnet
```
</details>


* * *

## Beefed up torrc

Overzealous idk test it, works for my setup.

<details>

<summary>Torrc file with extra changes</summary>  


```text
DataDirectory /var/lib/tor

### Hidden Service Core Configuration
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 172.21.8.203:80

# Force v3 onion services which have better security properties than v2
HiddenServiceVersion 3

### Critical Security Additions
HiddenServiceSingleHopMode 0             
HiddenServiceNonAnonymousMode 0          

# Anti-fingerprinting measures
AvoidDiskWrites 1                        
DisableDebuggerAttachment 1              
ConnectionPadding 1                      
ReducedConnectionPadding 0               
CircuitPadding 1                         
ReducedCircuitPadding 0                  

# Hardware acceleration introduces fingerprintable artifacts
HardwareAccel 0

# Log configuration - minimal logging for security
Log notice stdout

# Circuit reliability and security settings
NumEntryGuards 4                
HeartbeatPeriod 30 minutes      
NumDirectoryGuards 3            
MaxClientCircuitsPending 32     
KeepalivePeriod 60 seconds      

# Additional security hardening
StrictNodes 1                   
ControlPortWriteToFile ""       
CookieAuthentication 0          

```
</details>


* * *

## Uninstall

How do you stop the Tor network now that you've let it onto your computer? You've let Skynet spread!!! 

1. Go get a high-density polyethylene (HDPE) container. HDPE is a commonly used plastic for robust, leak-proof containers.

2. Fill this up to the brim with high octane gasoline.

3. Dowse your computer in as much gas as possible.

4. Now your computer will run faster, but the Tor network is still on it.

5. Should we: 
 
 - uninstall the work we did
 
 - kill Tor with fire

6. To uninstall, delete the directory you created for this script and demonstration (you may have to use sudo) and run the following to remove the docker container:

  ```bash
  
  docker stop $(docker ps -a | grep tor-hidden-service | awk '{print $1}') 2>/dev/null && docker rm $(docker ps -a | grep tor-hidden-service | awk '{print $1}') 2>/dev/null
  
  ```
