---
layout: post
title: Unbound Views in OPNsense to Resolve Domains by VLAN/Subnet
date: 2025-09-27 11:33:00 -0700
categories: [Networking, Router]
tags: [networking, security, server, workstation, opnsense, splitdns, multisite, firewall, DNS, unbound]
pin: false
image:
  path: /assets/img/header/header--opnsense--unbound-dns-views-resolve-domains-by-subnet.jpg
  alt: Unbound views allows for easy domain resolution based on subnet or interface
---

# Configure Unbound DNS in OPNsense for Subnet Based Domain Resolution

Running multiple VLANs in your home or lab can be a headache — especially when it comes to DNS. If you’ve got a server with interfaces on multiple VLANs/subnets, OPNsense’s Unbound DNS doesn’t respond automatically based on the interface or subnet. By default, it gives clients **all the IPs** it knows for that hostname.

That means your gaming PC on VLAN 112 might resolve `unraid.myhouse.home.arpa` to an IP on VLAN 88, which won’t work.


* * *

## Unbound Views

Unbound has a feature called **views**. With views, you can tell Unbound:

- “If a client comes from subnet `192.168.112.0/21`, use this view.”

- “If a client comes from subnet `192.168.88.0/21`, use that view.”

Each view can have its own DNS records, so the same hostname can return **different IPs depending on the source VLAN**.

For example:

- Clients on VLAN 112 → `unraid.myhouse.home.arpa = 192.168.112.12`

- Clients on VLAN 88 → `unraid.myhouse.home.arpa = 192.168.88.12`

This **solves DNS leaking** all IPs problem once and for all.


* * *

### Unbound Views Script

If you only have one or two VLANs, you could hand-write the Unbound config file with views. But if you’ve got multiple VLANs and several hosts, the config quickly becomes **tedious and error-prone**.

That’s why I built a script to do it automatically. Instead of learning Unbound’s config syntax line-by-line, you just define your VLANs and hosts in a simple text file — and the script generates the correct configuration for OPNsense.

You can find the — [🔭 generate-unbound-views.sh script](#generate-unbound-viewssh-script) — script below.


* * *

## Using DNS Views

When running multiple VLANs and subnets in OPNsense, you may need Unbound DNS to return different IP addresses depending on the interface or client network.

For example:

- You have a host `unraid` with addresses on multiple VLANs.

- You want clients on VLAN 112 (`192.168.112.0/21`) to resolve `unraid.myhouse.home.arpa` to `192.168.112.12`.

- You want clients on VLAN 88 (`192.168.88.0/21`) to resolve that same hostname to `192.168.88.12`.

By default, OPNsense/Unbound will happily return all addresses for `unraid` to everyone — which isn’t what we want.

This is where views come in. PowerDNS, some CISCO stuff, and BIND all have views available. Infact, BIND introduced views. BIND is available in OPNsense, but today we're just covering Unbound.


* * *

### The Problem with Old Documentation

If you’ve searched forums or blogs, you’ve probably seen instructions like:

> “Paste your custom config under **Services → Unbound DNS → Advanced**.”

That’s no longer possible. As of **OPNsense 21.1**, the custom configuration fields were removed ([announcement here](https://forum.opnsense.org/index.php?topic=23929.msg114008#msg114008)).

#### New Production Friendly Method

The new approach is to place your Unbound config snippets into:

```
/usr/local/etc/unbound.opnsense.d/
```

That’s documented in the [OPNsense manual](https://docs.opnsense.org/manual/unbound.html#advanced-configurations), but it doesn’t explain *how* to use Unbound **views**.For that, you need the upstream Unbound docs:

- [Tags & Views](https://unbound.docs.nlnetlabs.nl/en/latest/topics/filtering/tags-views.html)

- [Unbound access-control-view](https://nlnetlabs.nl/documentation/unbound/unbound.conf/#access-control-view)


* * *

## Step-by-Step: Configuring Views

We’re going to tell Unbound:

- **Which subnets** belong to which views

- **What local records** each view should return


* * *

Instructions will walk through setting this up with two VLANs:

- VLAN 112: `192.168.112.0/21` → `unraid = 192.168.112.12`

- VLAN 88: `192.168.88.0/21` → `unraid = 192.168.88.12`


* * *

### 1. SSH into OPNsense and install an editor

```sh
pkg install nano
```

### 2. Create a custom Unbound config file

```sh
nano /usr/local/etc/unbound.opnsense.d/vlan-views.conf
```

### 3. Add the configuration

```yaml
# Server defines which subnets use which views
server:
    access-control-view: 192.168.112.0/21 "vlan112_view"
    access-control-view: 192.168.88.0/21 "vlan88_view"

# View for VLAN 112 (Gaming)
view:
    name: "vlan112_view"
    local-zone: "myhouse.home.arpa." transparent
    local-data: "unraid.myhouse.home.arpa. IN A 192.168.112.12"

# View for VLAN 88 (Work)
view:
    name: "vlan88_view"
    local-zone: "myhouse.home.arpa." transparent
    local-data: "unraid.myhouse.home.arpa. IN A 192.168.88.12"
```

> 🔎 Replace `myhouse.home.arpa` with your own local domain.

### 4. Validate the config

```sh
configctl unbound check
```

### 5. Restart Unbound

```sh
configctl unbound restart
```

(Optional: reboot if you want to be extra sure it persists.)

### 6. Verify

The config won’t appear in `/var/unbound/unbound.conf` directly — that file is generated dynamically. But your custom file will be included automatically.

Check with:

```sh
dig @192.168.112.254 unraid.myhouse.home.arpa
dig @192.168.88.254 unraid.myhouse.home.arpa
```

Clients in VLAN 112 should get `192.168.112.12`, while VLAN 88 clients should get `192.168.88.12`.


* * *

Now you can:

- Control DNS responses per subnet/VLAN

- Keep multiple interfaces for the same hostname cleanly separated

- Avoid Unbound’s default behavior of returning *all* addresses

This is extremely useful if you’re running a host that has multiple IPs across different networks, but you want clients to resolve to the “closest” or correct one automatically.


* * *

## Automating Unbound Views with a Script

Doing this by hand works fine for one or two VLANs. But if you have several VLANs and multiple hosts (`unraid`, `nas`, `plex`…), editing configs quickly becomes painful.

That’s why I wrote a script — [🔭 generate-unbound-views.sh script](#generate-unbound-viewssh-script) — that automates everything.


* * *

### How It Works

The script takes an input file where each line is:

```txt
VLAN_ID,SUBNET,HOSTNAME,IP_ADDRESS
```


*For example*:


```txt
# Gaming VLAN
112,192.168.112.0/21,unraid,192.168.112.12
112,192.168.112.0/21,nas,192.168.112.15

# Work VLAN
88,192.168.88.0/21,unraid,192.168.88.12
88,192.168.88.0/21,nas,192.168.88.15
```

> Run the script, and it generates the full Unbound config.


* * *

### Running the Script

#### Step 1. Download the generate-unbound-views.sh script

Copy and paste the script to your OPNsense box, adding execute privs:

```sh
nano /root/generate-unbound-views.sh
chmod +x /root/generate-unbound-views.sh
```

#### Step 2. VLAN_ID,SUBNET,HOSTNAME,IP_ADDRESS

Create your VLAN definitions in a file:

```sh
nano /root/vlans.txt
```

#### Step 3. Use the script and your input file

Generate the Unbound config:

```sh
./generate-unbound-views.sh vlans.txt /usr/local/etc/unbound.opnsense.d/vlan-views.conf
```

#### Step 4. Does it still work

Validate and restart Unbound:

```sh
configctl unbound check
configctl unbound restart
```

#### Step 5. Did you change anything

Test resolution from each VLAN:

```sh
dig unraid
```

> Clients on VLAN 112 will see the `.112.12` address, while VLAN 88 clients will see `.88.12`.


* * *

## generate-unbound-views.sh script


> **Remember:** Put all of your VLANs, Subnets, hostnames, IPs in a simple comma seperated text file for input.


* * *

```bash
#!/bin/sh
#
# generate-unbound-views.sh
# 
# Purpose: Generate Unbound VLAN-specific DNS views configuration from an input file
# Usage: ./generate-unbound-views.sh input.txt output.conf
# 
# Input file format (CSV-style, comments allowed with #):
# VLAN_ID,SUBNET,HOSTNAME,IP_ADDRESS
# 
# Example input.txt:
# # Gaming VLAN
# 112,192.168.112.0/24,unraid,192.168.112.12
# 112,192.168.112.0/24,nas,192.168.112.15
# # Work VLAN
# 88,192.168.88.0/24,unraid,192.168.88.12
# 88,192.168.88.0/24,nas,192.168.88.15

set -e  # Exit on any error

# ============================================================================
# Configuration
# ============================================================================
DOMAIN_SUFFIX="myhouse.home.arpa"
DEFAULT_OUTPUT="/usr/local/etc/unbound.opnsense.d/vlan-views.conf"

# ============================================================================
# Functions
# ============================================================================

usage() {
    cat << EOF
Usage: $0 <input_file> [output_file]

Generate Unbound VLAN-specific DNS views configuration.

Arguments:
    input_file   - CSV file with VLAN definitions (required)
    output_file  - Output configuration file (default: ${DEFAULT_OUTPUT})

Input file format (CSV with optional comments):
    VLAN_ID,SUBNET,HOSTNAME,IP_ADDRESS

Example input file:
    # Gaming VLAN
    112,192.168.112.0/24,unraid,192.168.112.12
    112,192.168.112.0/24,nas,192.168.112.15
    # Work VLAN  
    88,192.168.88.0/24,unraid,192.168.88.12
    88,192.168.88.0/24,nas,192.168.88.15

Example usage:
    $0 vlans.txt
    $0 vlans.txt /tmp/test-views.conf

After generation:
    configctl unbound check    # Validate configuration
    configctl unbound restart  # Apply changes
EOF
    exit 1
}

log_info() {
    echo "[INFO] $*" >&2
}

log_error() {
    echo "[ERROR] $*" >&2
}

validate_input() {
    local input_file="$1"
    
    if [ ! -f "$input_file" ]; then
        log_error "Input file not found: $input_file"
        exit 1
    fi
    
    if [ ! -r "$input_file" ]; then
        log_error "Input file not readable: $input_file"
        exit 1
    fi
    
    # Check if file has any valid data lines
    if ! grep -qE '^[^#]' "$input_file" 2>/dev/null; then
        log_error "Input file contains no valid data (only comments or empty)"
        exit 1
    fi
}

# ============================================================================
# Main Script
# ============================================================================

# Check arguments
if [ $# -lt 1 ]; then
    usage
fi

INPUT_FILE="$1"
OUTPUT_FILE="${2:-$DEFAULT_OUTPUT}"

log_info "Input file: $INPUT_FILE"
log_info "Output file: $OUTPUT_FILE"
log_info "Domain suffix: $DOMAIN_SUFFIX"

# Validate input file
validate_input "$INPUT_FILE"

# Create temporary file for processing
TMP_FILE=$(mktemp) || {
    log_error "Failed to create temporary file"
    exit 1
}

# Ensure temp file cleanup on exit
trap 'rm -f "$TMP_FILE"' EXIT INT TERM

# ============================================================================
# Parse input and build data structures
# ============================================================================
log_info "Parsing input file..."

# Arrays to store unique VLANs and their data
# Format: vlan_id:subnet:hostname1:ip1:hostname2:ip2...
VLAN_DATA=""

while IFS=',' read -r vlan_id subnet hostname ip_address; do
    # Skip comments and empty lines
    echo "$vlan_id" | grep -qE '^\s*#' && continue
    echo "$vlan_id" | grep -qE '^\s*$' && continue
    
    # Trim whitespace
    vlan_id=$(echo "$vlan_id" | tr -d ' \t')
    subnet=$(echo "$subnet" | tr -d ' \t')
    hostname=$(echo "$hostname" | tr -d ' \t')
    ip_address=$(echo "$ip_address" | tr -d ' \t')
    
    # Validate fields
    if [ -z "$vlan_id" ] || [ -z "$subnet" ] || [ -z "$hostname" ] || [ -z "$ip_address" ]; then
        log_error "Invalid line (missing fields): $vlan_id,$subnet,$hostname,$ip_address"
        continue
    fi
    
    # Store data - append to existing VLAN or create new entry
    if echo "$VLAN_DATA" | grep -q "^$vlan_id:$subnet:"; then
        # Append to existing VLAN
        VLAN_DATA=$(echo "$VLAN_DATA" | sed "s|^$vlan_id:$subnet:\(.*\)$|$vlan_id:$subnet:\1:$hostname:$ip_address|")
    else
        # New VLAN entry
        if [ -z "$VLAN_DATA" ]; then
            VLAN_DATA="$vlan_id:$subnet:$hostname:$ip_address"
        else
            VLAN_DATA="$VLAN_DATA
$vlan_id:$subnet:$hostname:$ip_address"
        fi
    fi
    
done < "$INPUT_FILE"

if [ -z "$VLAN_DATA" ]; then
    log_error "No valid data parsed from input file"
    exit 1
fi

# ============================================================================
# Generate configuration file
# ============================================================================
log_info "Generating Unbound configuration..."

cat > "$TMP_FILE" << 'EOF_HEADER'
# Unbound VLAN-specific DNS Views Configuration
# Generated by generate-unbound-views.sh
# DO NOT EDIT MANUALLY - Regenerate using the script
#
# Purpose: Return VLAN-specific IP addresses based on which subnet
# the DNS query originates from.

# ============================================================================
# SERVER CLAUSE
# ============================================================================
# CRITICAL: access-control-view MUST be inside a server: clause.
# These directives are server-level configuration and will be ignored or
# cause errors if placed at the root level of the config file.
server:
    # Map source subnets to their respective views
    # Format: access-control-view: <subnet> "<view_name>"
    # Note: View names MUST be quoted
    
EOF_HEADER

# Generate access-control-view entries
echo "$VLAN_DATA" | while IFS=':' read -r vlan_id subnet rest; do
    cat >> "$TMP_FILE" << EOF
    # VLAN ${vlan_id} - ${subnet}
    access-control-view: ${subnet} "vlan${vlan_id}_view"
    
EOF
done

cat >> "$TMP_FILE" << 'EOF_MID'

# ============================================================================
# VIEWS - One per VLAN
# ============================================================================
EOF_MID

# Generate view entries
echo "$VLAN_DATA" | while IFS= read -r line; do
    [ -z "$line" ] && continue
    
    # Extract vlan_id and subnet (first two colon-separated fields)
    vlan_id=$(echo "$line" | cut -d: -f1)
    subnet=$(echo "$line" | cut -d: -f2)
    
    cat >> "$TMP_FILE" << EOF

# ============================================================================
# VIEW FOR VLAN ${vlan_id}
# ============================================================================
# Subnet: ${subnet}
# A view is a named local-zone tree that can be assigned to specific clients.
view:
    # Unique name for this view (must match access-control-view above)
    name: "vlan${vlan_id}_view"
    
    # Local zone type: "transparent"
    # - transparent: Answers local-data queries here, allows other queries to
    #   resolve normally (e.g., DHCP registrations, other host overrides)
    # - static: Would ONLY answer from local-data defined here; everything 
    #   else would get NXDOMAIN (too restrictive for most use cases)
    local-zone: "${DOMAIN_SUFFIX}." transparent
    
    # A records for this VLAN
EOF

    # Parse and add all hostnames for this VLAN
    # Format: vlan:subnet:hostname1:ip1:hostname2:ip2...
    # Start from field 3 (after vlan and subnet)
    field_num=3
    num_fields=$(echo "$line" | awk -F: '{print NF}')
    
    while [ $field_num -le $num_fields ]; do
        hostname=$(echo "$line" | cut -d: -f${field_num})
        next_field=$((field_num + 1))
        
        if [ $next_field -le $num_fields ]; then
            ip=$(echo "$line" | cut -d: -f${next_field})
            
            if [ -n "$hostname" ] && [ -n "$ip" ]; then
                cat >> "$TMP_FILE" << EOF
    local-data: "${hostname}.${DOMAIN_SUFFIX}. IN A ${ip}"
EOF
            fi
        fi
        
        # Move to next pair (skip 2 fields: hostname and ip)
        field_num=$((field_num + 2))
    done
    
    echo "" >> "$TMP_FILE"
done

# Add footer with instructions
cat >> "$TMP_FILE" << 'EOF_FOOTER'

# ============================================================================
# VALIDATION & RESTART
# ============================================================================
# After editing the input file and regenerating:
# 1. Validate: configctl unbound check
# 2. Restart:  configctl unbound restart
# 3. Test:     nslookup <hostname>.<domain> (from each VLAN)
#
# To regenerate this file, edit your input file and run:
#     ./generate-unbound-views.sh <input_file>
# ============================================================================
EOF_FOOTER

# ============================================================================
# Install the configuration
# ============================================================================

# Check if output directory exists
OUTPUT_DIR=$(dirname "$OUTPUT_FILE")
if [ ! -d "$OUTPUT_DIR" ]; then
    log_error "Output directory does not exist: $OUTPUT_DIR"
    log_error "This script is intended for OPNsense with Unbound installed"
    exit 1
fi

# Backup existing file if it exists
if [ -f "$OUTPUT_FILE" ]; then
    BACKUP_FILE="${OUTPUT_FILE}.backup.$(date +%Y%m%d_%H%M%S)"
    log_info "Backing up existing configuration to: $BACKUP_FILE"
    cp "$OUTPUT_FILE" "$BACKUP_FILE"
fi

# Move temp file to final location
mv "$TMP_FILE" "$OUTPUT_FILE"
chmod 644 "$OUTPUT_FILE"

log_info "Configuration generated successfully: $OUTPUT_FILE"
log_info ""
log_info "Next steps:"
log_info "  1. Review the configuration: cat $OUTPUT_FILE"
log_info "  2. Validate: configctl unbound check"
log_info "  3. Apply: configctl unbound restart"
log_info ""

# Summary of what was generated
VLAN_COUNT=$(echo "$VLAN_DATA" | wc -l | tr -d ' ')
log_info "Summary:"
log_info "  - VLANs configured: $VLAN_COUNT"
log_info "  - Domain suffix: $DOMAIN_SUFFIX"

echo "$VLAN_DATA" | while IFS=':' read -r vlan_id subnet rest; do
    # Count hostnames (every other field after subnet)
    host_count=$(echo "$rest" | awk -F: '{print int((NF+1)/2)}')
    log_info "  - VLAN $vlan_id ($subnet): $host_count host(s)"
done

exit 0

```


* * *

## More Unbound View Examples

Let me give another example and expand on these views a bit more, based on the concept here: https://forum.opnsense.org/index.php?topic=40247.0


You can fine grain redirect specific subnets, for example work people cannot use Copilot or watch restricted YouTube content:


```txt
# GLOBAL: Define which subnets get which views
server:
    access-control-view: 192.168.112.0/21 "vlan112_view"
    access-control-view: 192.168.88.0/21 "vlan88_view"

# VIEW: VLAN 112 (Gaming) - No filters
view:
    name: "vlan112_view"
    local-zone: "myhouse.home.arpa." transparent
    local-data: "unraid.myhouse.home.arpa. IN A 192.168.112.12"

# VIEW: VLAN 88 (Work) - With redirect filters
view:
    name: "vlan88_view"
    local-zone: "myhouse.home.arpa." transparent
    local-data: "unraid.myhouse.home.arpa. IN A 192.168.88.12"

    # Bing
    local-zone: "bing.com" redirect
    local-data: "bing.com CNAME nochat.bing.com"

    # CoPilot
    local-zone: "copilot.microsoft.com" redirect
    local-data: "copilot.microsoft.com CNAME cdp.copilot.microsoft.com"

    # YouTube
    local-zone: "www.youtube.com" redirect
    local-data: "www.youtube.com CNAME restrict.youtube.com"
    local-zone: "m.youtube.com" redirect
    local-data: "m.youtube.com CNAME restrict.youtube.com"
    local-zone: "youtubei.googleapis.com" redirect
    local-data: "youtubei.googleapis.com CNAME restrict.youtube.com"
    local-zone: "youtube.googleapis.com" redirect
    local-data: "youtube.googleapis.com CNAME restrict.youtube.com"
    local-zone: "www.youtube-nocookie.com" redirect
    local-data: "www.youtube-nocookie.com CNAME restrict.youtube.com"
```
