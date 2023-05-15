---
title: ZFS for Proxmox Storage
date: 2023-02-22 11:33:00 -0700
categories: [Proxmox, ProxmoxStorage]
tags: [proxmox, storage, ZFS, server]
image:
  path: /assets/img/header/header--proxmox--proxmox-storage-zfs.jpg
  alt: Proxmox storage with ZFS
---



# Using ZFS with Proxmox Storage has never been easier!

## Quick ZFS Tips

### When creating ZFS Pools and ZFS top level datasets, increment them
Never give two zpools the same name even if they’re in different servers in case there is the off-chance that sometime down the road I’ll need to import two pools into the same system.  
Name your zpool `tank[n]` and create a top level dataset called `ds[n]` where n is unique number across all your pools, just in case you ever have to bring two separate datasets onto the same zpool. 
#### Why? 
I replicate them to each other for backup.  Having that top level `ds[n]` dataset lets me manage ds1 (the primary dataset on the server) completely separately from the replicated dataset (ds2) on stor1. Then the stor2 server replicates the same way. Copy of both datasets. Basically HA server with ZFS. 
```
stor1.b3n.org
 | - tank1/ds1
 | - tank2/ds2 (replicated)
```
Then he creates sub-datasets for all of his different priority backups. 
Highest rank > Backed up to both servers, a usb drive, and offsite backups to cloud server and his buddy's server. 
"It’s backed up to CrashPlan offsite, rsynced to a friend’s remote server, snapshots are replicated to a local ZFS server, plus an annual backup to a local hard drive for cold storage.  3 copies onsite, 2 copies offsite, 2 different file-system types (ZFS, XFS) and 3 different backup technologies (CrashPlan, Rsync, and  ZFS replication) ."
`tank1/ds1/highestrank/photos`
Seciond highest rank > Backed up movies and music, etc. 
"For this dataset I want at least 2 copies, everything is backed up offsite to CrashPlan and if I have the space local ZFS snapshots are replicated to a 2nd server giving me 3 copies."
`tank1/ds1/secondhighest/media`




