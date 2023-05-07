---
title: New PC or New Install - Need to move Proxmox over to a new computer... No Cluster NO PROBLEM!
date: 2022-12-19 11:33:00 -0700
categories: [Proxmox, ProxmoxInstall]
tags: [virtualization, proxmox, storage, backups, mirror, install]
image:
  path: /assets/img/header/header--proxmox--install-proxmox-from-backup.jpg
---

# New PC? New Install? Need to move Proxmox over to a new computer? No Cluster? NO PROBLEM!

Great! You want to re-install proxmox, but use your current... everything.

Maybe you want to replace a failing drive, or make a RAID-6 configuration, whatever the case -- this guide has you covered.




## Save your data
Copy these folders/files before you transfer, they make setting up a new install feel like magic. 
```
/root/
/etc/network/interfaces
/etc/pve/storage.cfg
/etc/sanoid/sanoid.conf
/etc/ssh/sshd_config
/etc/crontab
/etc/hostname
/etc/hosts
/etc/resolv.conf
```



## And save those VMs

VMs need to be backed up to the new machine/install.

Backups on 'local' are stored in `/var/lib/vz/dump/`

Backups for PBS that are taken on localhost and are in the same zfs pool as the VM PBS is backing up... 

example: `/rpool/data/vm/100`

Find your most recent backups above and move them to a safe location that can transfer them back once the new partitions are done.



### (Optional) Mirror your old drive before wiping it.

Take your old drive, and make an exact replica of the drive. 

Just go out to (Microcenter, Best Buy, Amazon) and buy a drive of the same size or larger. Then mirror the drive you're about to wipe to this returnable drive with Clonezilla. 

Now you have an "oopsie, I actually needed that" drive ready.




## Import ZFS pools to the new install

If you have a backup pool that needs imported, do that now by telling ZFS the name of the pool to import:

`zpool import -f nameofpool`

Then you can run `zfs get all` and should see the newly imported pool.




## Renaming a PVE node

You must edit:

`/etc/hosts`

`/etc/hostname`

In the above files replace all occurrences of the *old name* with the *new one*. Ensure that `/etc/hosts` has an entry with the hostname mapped to the IP you want to use as main IP address for this node. 




### Cleanup the old PVE node

Now move the configuration files, as the *pmxcfs* has a few restrictions to ensure consistency you cannot rename non empty folders. Thus if you have VMs or Containers on the node, which is not recommended when changing a nodes name, *you have to recreate the folder structure and copy files per folder level*.

Also copy the contents of: 

`/var/lib/rrdcached/db/pve2-{node,storage}/old-hostname` to `/var/lib/rrdcached/db/pve2-{node,storage}/new-hostname` and remove the old directory. 

`mv /etc/pve/nodes/prox/openvz/* /etc/pve/nodes/proxmox-Xen/openvz/`

