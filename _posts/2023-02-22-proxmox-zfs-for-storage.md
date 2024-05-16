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

## No pool creation needed, Proxmox installed with ZFS root, PBS pools ready

This is the best case. When you've already installed Proxmox and have your ZFS pools ready.

ZFS and Proxmox combined will be taking a chunk out of your storage if you installed Proxmox on ZFS as root. 

You can check available space:

`zfs list -o space rpool`

Then, if you have VMs, you can see how much total space the datastore is using:

`zfs list -o space rpool/data`

And wonder what VM is using the most space? ZFS can tell us where all the datastore's information is located:

`zfs list -t all -r rpool/data`

You can also confirm the mount point correctly with:

`zfs get all` and look for the *mountpoint* in the *property* column



* * *
## ZFS datastores (this is basically like LVM partitions)

You need to tell the pool a place to put the data. It only knows about the group of volumes you created. It doesnt know how to put anything on those volumes.

Now, let's place the disk space in our zfs pools into datastores that we can use...



### Drives available and partition sizes

Note:

#### *Resist the urge to rename your zfs datastores that are being used for VMs.*

Snapshots on VMs (these would sync in a cluster) should be stored on a 'local-zfs' datastore. Using the same name across devices allows Proxmox to sync these stores if there was ever to be a cluster formed. And it allows for uniform naming.



### Partitions Used

Here are the ZFS datastores I use:

`RPOOL` this pool is the default pool that comes with Proxmox on ZFS.

`RPOOL/root` the root filesystem is located at `rpool/ROOT/pve-1` and mounted at `/`

`RPOOL/data` this pool comes with Proxmox on ZFS already. It is used to store the VMs, their snapshots, and backup data.

`RPOOL/faststore` this is the top level datastore for everything that needs 9p into docker

`ROOL/faststore/appdata` my docker application files are stored here, that means config files for most of the docker containers and their persistant storage.

~~`RPOOL/docker` my docker also needs a place to store it's images and instead of letting the VM balloon in size, I've got this directory mapped to `/var/lib/docker/` for docker images.~~

`RPOOL/faststore/fastdata` this is where any data that needs to be on the SSD can be stored that isnt really dedicated to one application or task (if so i need subvolumes).

`RPOOL/web` storage for VirtaMin websites.

`longstore/????` is the ZFS pool for my spinning disks I use for PBS backups. Just like `local-zfs` is the zfspool for `rpool/data`.

`longstore/proxbox-backup` The PBS storage is named `proxbox-backup`. I try and keep this pool used as little as possible, almost exclusivly for backups - to help the drive's lifespan.

`longstore/slowstore` an extra datastore for being able to place any data that needs to be stored and archived but really isnt important and needs rarely accessed.



### Network Mounts Needed

I also mount some network shares from a larger NAS (unraid):

`Cloud` Should the Nextcloud files all live on the larger NAS?

`Videos` These are finished, moved, synced with plex videos. TV, Movies, Youtube, etc.

`Music` This is the area where Airsonic can find the files it serves out.

`Pictures` Here are where that one docker app finds all the cool pictures it can serve out.



## Create ZFS datastores named above

### Do the HDD Datastores first
`zfs create -o compression=on longstore/proxbox-backup`
`zfs create -o compression=zstd longstore/slowstore`

### SSD Datastores
`zfs create -o compression=lz4 rpool/faststore`
`zfs create -o compression=lz4 rpool/faststore/appdata`
`zfs create -o compression=lz4 rpool/faststore/fastdata`
`zfs create -o compression=lz4 rpool/web`


* * *
* * *

## ZFS Partition Example: Using extra space on the end of the disk.
Let's say you DIDNT install Proxmox from the installer, and instead installed from Debian and left some space at the end of your drive for ZFS, and have an extra spinning disk installed for backups.
Where did you leave your partitions at? FIRST, let's find out...
`parted /dev/vda print`
Look underneath the 'end' column, and list that as your START, then do the math for the next size desired.

`parted /dev/sda mkpart primary zfs #from-above 183G` (this is for the localstore pool)
`parted /dev/vda mkpart primary zfs 183G 300G` (this one is the longstore pool)
`parted /dev/vda mkpart primary zfs 300G 100%` (final is the datastore)

Note:
*ZFS modules will not be loaded if you have not rebooted after your inital Proxmox installation.*

`zpool create -f -m /localstore localstore /dev/sda2`
`zpool create -f -m /longstore longstore /dev/vda1`
`zpool create -f -m /datastore datastore /dev/vda2`



### ZFS Example: Adding a RAID with zpool
```
zpool create -f -m /mnt/longstore -o ashift=12 longstore raidz
/dev/disk/by-id/ata-WDC_WD5000AACS-00G8B0_WD-WCAUF0975984 /dev/disk/by-id/ata-WDC_WD5000AACS-00G8B0_WD-WCAUF0550570 /dev/disk/by-id/ata-WDC_WD5002AALX-00J37A0_WD-WMAYU7679039
```



* * *
* * *



## Writing ZFS Partitions in Proxmox/Debian

We talked about ZFS pools, and different datastores for backup. 

Let's impliment those now.



### Writing Partition Changes

#### Blowing out old ZFS pools you dont need or want anymore

If you imported an old ZFS pool to restore some VMs. You can now wipe that ZFS pool and re-create it. 

I did the procedure like [this](https://forum.proxmox.com/threads/correct-procedure-for-zpool-removal.106203/):

I removed the `ZFS datastore` from the `storage` menu after **deactivating** it, by clicking `Edit` and unchecking the `Enabled` button.

I checked `storage.cfg` and the system had removed the mount point.

I ran the `destroy` from clicking on the name of the Node > ZFS > More > Destroy in the Proxmox GUI.

I checked for any pool from shell, making sure I have any ZFS Pool or disk from the `ZFS list` command.

Rebooted the cluster then node by node is removed the disk hardware. Everything went well. Thank you for your info.
 
Special cases:

- you do not care about information there: `zpool destroy <POOLNAME>`

- you want to import the pool on other machine: `zpool export <POOLNAME>`

[Creating and Destroying ZFS Storage Pools](https://docs.oracle.com/cd/E23824_01/html/821-1448/gaypw.html)


## REBOOT
Great idea to reboot right about now.


### Recreating that ZFS pool we destroyed

[To create a ZFS pool](https://wiki.archlinux.org/title/ZFS#Creating_ZFS_pools):

`zpool create -f -m /longstore -o ashift=12 longstore raidz /dev/disk/by-id/ata-WDC_WD5000AACS-00G8B0_WD-WCAUF0975984 /dev/disk/by-id/ata-WDC_WD5000AACS-00G8B0_WD-WCAUF0550570 /dev/disk/by-id/ata-WDC_WD5002AALX-00J37A0_WD-WMAYU7679039`



* * *
* * *



## Quick ZFS Tips


### When creating ZFS Pools and ZFS top level datasets, increment them

Never give two zpools the same name even if they’re in different servers in case there is the off-chance that sometime down the road I’ll need to import two pools into the same system.  

Name your zpool `tank[n]` and create a top level dataset called `ds[n]` where n is unique number across all your pools, just in case you ever have to bring two separate datasets onto the same zpool. 


#### Why? 

I replicate them to each other for backup.  Having that top level `ds[n]` dataset lets me manage ds1 (the primary dataset on the server) completely separately from the replicated dataset (ds2) on stor1. Then the stor2 server replicates the same way. Copy of both datasets. Basically HA server with ZFS. 

```
stor1.something.localdomain
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



### List Snapshots

`zfs list -r -t snapshot -Ho name -S creation`



### Delete all autosnap created snapshots

`zfs list -H -o name -t snapshot | grep autosnap | xargs -n1 zfs destroy`



* * *
* * *


## ZFS Backups

### Using Sanoid to snapshot ZFS

Sanoid works great for snaphotting ZFS automatically, it also handles snapshot sync. 

`/etc/sanoid/sanoid.conf` is the file required to get the thing to work. 

The documentation also says to copy `sanoid.defaults.conf`, lets do that now.

`cp /usr/share/sanoid/sanoid.defaults.conf /etc/sanoid/`

There is a starting template you can use, or find my sample in the repository below. To copy the template file:

`cp /usr/share/doc/sanoid/examples/sanoid.conf /etc/sanoid/`


Then, let's talk about how Sanoid starts. We want to disable the installed cron, and let the systemd service manage it. 

`nano /etc/cron.d/sanoid` and put a `#` infront of the line.



Now, if you wanted to add your owncrontab:

`nano /etc/crontab`

Add this line to allow Sanoid to run every min, and run to your local timezone!! It uses UTC by default. 

` * * * * *  root TZ=/usr/share/zoneinfo/America/Denver /usr/sbin/sanoid --quiet --cron`


But it's not nessesary to change crontab, as the systemd service handles everything on Debian.


Again, we need to change the timezone from UTC by default to your local timezone. To do this, edit the systemd services that run in the background:

`nano /etc/systemd/system/sanoid.service.wants/sanoid-prune.service`

`nano /usr/lib/systemd/system/sanoid.service`

to change the timezone:

 `Environment=TZ=/usr/share/zoneinfo/America/Denver`



Also, if you want to run Sanoid like the cron job above, every min -- you will need to change the timeings for the systemd service:

`nano /usr/lib/systemd/system/sanoid.timer`

Change the systemd timer to run at 00 seconds every minute:

`OnCalendar=*-*-* *:*:00`

Or keep it as every 15min:

`OnCalendar=*:0/15`



```
Systemd Timer OnCalendar Format
---------------------------------------
So the basic format of Oncalnedar event is this -
It is divided into 3 parts -

    * - To signify the day of the week eg:- Sat,Thu,Mon

    *-*-* - To signify the calendar date. Which means it breaks down to - year-month-date.
        2021-10-15 is 15th of October
        *-10-15 means every year at 15th October
        *-01-01 means every new year.

    *:*:* is to signify the time component of the calnedar event. So it is - hour:minute:second

```



Then, after install, it's almost required to run it once and make sure it works.

`TZ=/usr/share/zoneinfo/America/Denver /usr/sbin/sanoid --take-snapshots --verbose`

and then

`sanoid --cron`





#### [My personal setup for using Sanoid to snapshot ZFS](https://github.com/MarcusHoltz/proxmox-automatic-backups/tree/main/sanoid-stuff)





* * *
* * *




## SSH to the remote host from the receiving end

Remote Replication of ZFS Data

`ssh 172.21.8.64 zfs send longstore/data@2022-10-04_Addedstuff | zfs receive -F longstore/slowstore@2022-10-04_Addedstuff`


You can use the zfs send and zfs recv commands to remotely copy a snapshot stream representation from one system to another system. For example:

`zfs send tank/cindy@today | ssh newsys zfs recv sandbox/restfs@today`


This command above sends the `tank/cindy@today` snapshot data and receives it into the `sandbox/restfs` file system. The command also creates a `restfs@today` snapshot on the `newsys` system. In this example, the user has been configured to use ssh on the remote system.



## ZFS receive with -F isnt a bad thing

```
     -F      Force a rollback of the file system to the most recent snap-
             shot before performing the receive operation. If receiving an
             incremental replication stream (for example, one generated by
             "zfs send -R -Fi -iI"), destroy snapshots and file systems
             that do not exist on the sending side.
```

The -F switch comes in handy if you have messed with the destination dataset after it has been received. Once you do any changes to it (including doing something as innocent as a directory listing as this would change atimes), it is no longer in the state it was in after the initial transfer. 

Trying to run a plain zfs receive from an incremental data stream created by the other side's `zfs send -i tank/dataset@old tank/dataset@new` would result in an error. 

In this case you have two options on the receiver side:

- you could either revert to the last snapshot manually using zfs rollback

- or provide the `-F` switch to `zfs receive` to let it handle that for you automatically


The following checks are performed before the -F option is successful:

- If the most recent snapshot doesn't match the incremental source, neither the roll back nor the receive is completed, and an error message is returned.

- If you accidentally provide the name of different file system that doesn't match the incremental source specified in the zfs receive command, neither the rollback nor the receive is completed.



### Receiving a ZFS Snapshot

Keep the following key points in mind when you receive a file system snapshot:

- Both the snapshot and the file system are received.

- The file system and all descendent file systems are unmounted.

- The file systems are inaccessible while they are being received.

- The original file system to be received must not exist while it is being transferred.

- If the file system name already exists, you can use zfs rename command to rename the file system. 



* * *
## SSH ZFS Send/recieve with mbuffer
When deciding to first migrate all the data from one place to another, you'll see that you'll need all the speed. A good trick to boost up the replication performance is to get the data through mbuffer.

It can be used as below. Chunk size and the buffer size can vary and you should play around to see what suits you best.

`zfs send testpool/testdataset@snapshot | mbuffer -s 128k -m 1G | ssh user@backup_host 'mbuffer -s 128k -m 1G | zfs receive testpool_backup/testdataset_backup'`



* * *
* * *




### HTTM

https://github.com/kimono-koans/httm

HTTM allows you to view your ZFS snapshots in the terminal with ease. 


#### HTTM EXAMPLES:

To find and browse through all the files deleted, recursivly:

`httm --deleted=only -i -R .`

`-i` is interfactive, letting you move through the menu and not just print to screen

`-R` is recursive, meaning, search all folders

`--deleted=only` tells it to only show the deleted files [possible values: all, single, only]

`.` is the current working directory




* * *
* * *





## ZFS Performance Tweaks

Some filesystems support an option of disabling atime, meaning that’s one operation that filesystem doesn’t need to track and therefore update on the disk when you’re accessing files (or listing directories). In modern operating systems, there might be hundreds or thousands of files read from each filesystem

Disabling atime is a great way to improve I/O performance on filesystems with lots of small files that are accessed freqently.

Print out a list of all your ZFS datasets:

`zpool list`

To check if atime is on or off:

`zfs get all <dataset_name_here> | grep time`

Ok, so disable that for all of your datasets:

`zfs set atime=off <dataset_name_here>`




### ZFS Performance: RAM consumption

If the ZFS filesystem is the backend for a storage system, then the performance of the filesystem can be increased by tuning its use of physical memory.

To find out how often your ARC cache is being used:

`cat /proc/spl/kstat/zfs/arcstats`

Look for 'hits' that should be a number more relative to how often your arc is being used.

'c' is the target size of the ARC in bytes

'c_max' is the maximum size of the ARC in bytes

'size' is the current size of the ARC in bytes

`arcstat` will also display a relative consumption of the ARC cache.

And for the most maximum amount of information:

`arc_summary`

To make a persistent change to the ARC cache that will stay after a reboot, add the `zfs_arc_max=10737418240` value to `/etc/modprobe.d/zfs.conf` file for a 10GB limit.

BUT none of your changes to modprobe will be picked up, until you run:

`update-initramfs -u -k all`


ZFS will actually shrinks ARC when more memory needed for system, thus I

also use:

```echo 50 >/sys/module/zfs/parameters/zfs_arc_pc_percent```

in order to apply some memory pressure to the rest of the system and use zram more actively.




### Note: If your root file system is ZFS you must update your initramfs every time this value changes:

`update-initramfs -u`

If you are on an EFI system, you will also need to update the kernel list in the EFI boot menu so the updated initial RAM file system is used.

`pve-efiboot-tool refresh`


### Set arc min and max persistent on reboots:

```
$ vim /etc/modprobe.d/zfs.conf
>>>
options zfs zfs_arc_max=34359738368
options zfs zfs_arc_min=17179869184
<<<
update-initramfs -u
 ```
 
### Set arc min max realtime (non-persistent)
```
$ echo 17179869184>> /sys/module/zfs/parameters/zfs_arc_min
$ echo 34359738368>> /sys/module/zfs/parameters/zfs_arc_max
```
 
I also using following command in cronetab once a 24 hours to clear the read-cache and make the RAM available again as ZFS’s ARC (disk cache) is growing in size and Proxmox ARC is not releasing this process automatically:

`echo 3 > /proc/sys/vm/drop_caches`
 
You can test different ARC sizes and run `arc_summary` in cli. 
 
Optional: It is suggested that adding the flag `options zfs zfs_flags=0x10` to the `/etc/modprobe.d/zfs.conf` file may help mitigate the risk of using non-ECC RAM, at the cost of some performance loss (I’m not using this in production, but it is worth mentioning here).
 
After rebooting the system, use the command `arcstat` to make sure the changes have been applied, the value of the last column (c) should be the maximum ARC size you’ve set in /etc/modprobe.d/zfs.conf (expressed in GiB).
 
 




* * *
* * *





## NFS sharing over ZFS

Shares are per dataset, even if the share is the parent of children datasets. You will not be able to access those children. You will, however, be able to see those datasets. 

This is great from a security point-of-view, but I realize it can be a bit annoying. If you don't set up that child for sharing, the client will see the dataset folder, but not be able to access it.

`zfs set sharenfs=on zstorage/VM1data to enable NFS sharing on that dataset`

### To get NFS working with ZFS you need a few tools

https://blog.programster.org/sharing-zfs-datasets-via-nfs


Also too much information: 

https://forum.proxmox.com/threads/virtfs-virtio-9p-plans-to-incorporate.35315/#post-434064





* * *
* * *





## Adding the new longstore/proxbox-backup to PBS so that you can use it to backup proxbox

Connect to PBS

On the left, click the 'add Datastore'

Name can be anything, I just usually reverse whatever the ZFS datastore name is. (E.G - `backup-proxbox`)

Blocking path is the path that comes up when you type `zfs get all`. This should be inherited from the ZFS Pool's original mount point. (E.G. - `longstore/proxbox-backup`)



### Adding the new backup-proxbox PBS storage to PVE for backup location

Open PVE

Datacenter > Storage > 'Add' dropdown menu > 'Proxmox Backup Server'

`ID` is the name of the PBS Name we made above, generally I re-reverse it back to the ZFS datastore's name. (E.G - `proxbox-backup`)

`Server` is THIS server. We're using the host machine for PBS backups. (E.G - `localhost`)

`Username` is here incase you created a new user for PBS above in the post-install section. Enter it here. (E.G - `username@pam`)

`Datastore` is the name you gave the PBS above. Usually reverse whatever the ZFS datastore name is. (E.G - `backup-proxbox`)

`Fingerprint` can be found on the PBS Dashboard. Pretty much right in the middle of the screen is a button that says, 'Show Fingerprint'





### Adding the new ZFS Partitions/Datastores to Proxmox

You need to go to your respective server under 'Datacenter' and head to the 'Disks' area.

From there, find ZFS and click on that. You can see your partitions, but where to add these to Proxmox?


Datacenter > Storage

'add' > ZFS


ID will be the name you see in the left hand column.

Be sure to add the Proxmox Backup Server as well.

Go to the Proxmox Backup Server :8007

On the left hand side of the screen, under the menu, click 'Add Datastore'


Name: backupprox

Blocking Path: /mnt/datastore/backupprox

Change schedule to yearly

Click 'Add'


Now head back to the Proxmox Virtual Environment :8006


Datacenter > Storage

'add' > Proxmox Backup Server


ID: proxbackup

Server: localhost

Username: root@pam

Password: enterrootpassword

Datastore: backupprox









* * *

* * *

## Having trouble?

For a one-time donation you can get one-on-one troubleshooting support for any of my guides/projects. I'll help you fix any issue you may have encountered regarding usage/deployment of one of my guides or projects. More info in my [Github Sponsors profile](https://github.com/sponsors/MarcusHoltz).

<iframe src="https://github.com/sponsors/MarcusHoltz/card" title="By sponsoring me, you're supporting my ongoing work and everything you've come to know me by -- cutting-edge and resilient systems with vigilant monitoring practices." height="225" width="600" style="border: 0;"></iframe>
