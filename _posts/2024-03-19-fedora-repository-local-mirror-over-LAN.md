---
title: Create local cache of Fedora mirrors, update systems over LAN
date: 2024-03-19 11:33:00 -0700
categories: [Linux, LocalRepository]
tags: [security, administration, cloud, fedora, server, techtip, mirror, networking, repository]
pin: false
image:
  path: /assets/img/header/header--linux--fedora-local-repo-over-network.jpg
  alt: Local mirror of Fedora repos on LAN to save bandwidth
---

# Use a cached dnf mirror of Fedora repositories and update systems over the local network to save bandwidth


>  Limited bandwidth, no internet access, just trying to be green? Cache your files! 
{: .prompt-tip }



### Deploying Fedora on all of the computers across a workspace/home/office, updates pile up quickly. 

> Let's update our systems as efficiently as possible, with a local repository. 



## Steps involved

1. Install `python3-dnf-plugin-local`

2. Setup a repository. 

3. Share that repository across the network.

4. Repeat.



### Why is this method best?

- No Squid *SSL bumping*. 

- No Nginx *forward proxy*. 

- Just a plugin for dnf. 

I have done this several ways, and find using the dnf plugin to be the easiest. There's even an [Ansible task](https://github.com/buckaroogeek/ansible-role-fedora-dnf-local-plugin/blob/main/tasks/main.yaml) already for it.



## Creating the initial local repository

>  You have to create a repository for the local cache to store it's metadata in. The `createrepo_c` package will be installed as a dependency. 
{: .prompt-tip }



### Choose a location for your local repository. 

**Choose where you're keeping the data.**

>  This is possibly the most important step. 
{: .prompt-info }

1. **On the network.**
  - This can be any network share you can mount, NFS/SMB/SSH/whatever

2. **On your system.**
  - You need a location to mount the network share to.



### Make folder, mount network share, and create local repository

>  In this example, I will be storing the locally mirrored Fedora repo in `/srv/fedoraLocalRepo` and mounting a samba share.
{: .prompt-info }


* * * 

Please update all of these names according to your personal setup. 
You may need to change the SMB server, share, credentials, and mount point.


1. `sudo mkdir -p /srv/fedoraLocalRepo`

2. `sudo mount -t cifs -o guest,dir_mode=0777,file_mode=0777 //172.21.8.12/toshiba_n300/z_Fedora_Updates /srv/fedoraLocalRepo`

3. `sudo createrepo /srv/fedoraLocalRepo`


>  The local caching Fedora repository should now be setup and ready for use across the network.
{: .prompt-tip }


* * *

## Installing python3-dnf-plugin-local 

Install the required plugin to create a locally caching dnf mirror:

`sudo dnf install python3-dnf-plugin-local`


### Configuring the dnf local plugin

The configuration for the dnf local plugin is stored in: `/etc/dnf/plugins/local.conf`

We must defines where on the local filesystem the plugin will keep the RPM repository. Open and edit `local.conf`, and add the location of your local repository you created. 

Add the line `repodir = /srv/fedoraLocalRepo`:

```
[main]
enabled = true
# Path to the local repository.
# repodir = /var/lib/dnf/plugins/local
repodir = /srv/fedoraLocalRepo
```


* * *

## Testing the _dnf_local plugin

Remove any caching in dnf so we can run a fresh update against our new local repo:

`sudo rm -r /var/cache/dnf`

Install something that is not on your system, `dnf install htop`

You should see the program update, and use _dnf_local as a repository.

Check inside your local repository for the files downloaded: `ls /srv/fedoraLocalRepo/`

```
htop-3.3.0-1.fc39.x86_64.rpm  hwloc-libs-2.10.0-1.fc39.x86_64.rpm  repodata
```

> Great! Our binaries are now cached inside of a local repository we can share across the network.


* * *

### Instead of autofs, Systemd

We can start this mount as a service using Systemd. No need to install anything.

Using automount will also auto-un-mount the share as well. 

It will only be mounted when in use.

> Please note: the unit name should match the mount point. So, if the mount point is `/mnt/foo` the unit name should be `mnt-foo`.mount and `mnt-foo`.automount -- you cannot use-dashes-in-folder-names
{: .prompt-warning }


### Create Systemd unit files


* * *

#### SystemD mount file

Open the new Systemd mount file:

`sudo nano /etc/systemd/system/srv-fedoraLocalRepo.mount`

Enter the following (please update any information according to your personal setup. You may need to change the SMB server, share, credentials, and mount point): 

```
[Unit]
Description=mount fedora local repo on unraid

[Mount]
What=//172.21.8.12/toshiba_n300/z_Fedora_Updates
Where=/srv/fedoraLocalRepo
Type=cifs
Options=rw,file_mode=0777,dir_mode=0777,uid=1000,user=guest,password=guest
DirectoryMode=0777

[Install]
WantedBy=multi-user.target
```


* * *

#### SystemD automount file

Make sure the mount is automounted:

`sudo nano /etc/systemd/system/srv-fedoraLocalRepo.automount`

Enter the following: 

```
[Unit]
Description=automount my share

[Automount]
Where=/srv/fedoraLocalRepo
TimeoutIdleSec=360

[Install]
WantedBy=multi-user.target
```


* * *

### Enable and start the service

If everything was entered correctly, we can now start the mount and enable to for next reboot:

`sudo systemctl daemon-reload`

`sudo systemctl enable srv-fedoraLocalRepo.automount --now`


* * *

## Permanently mount the fedoraLocalRepo network share

With everything working correctly, we can now make sure our network mount attaches at boot.


* * *

### Modify fstab for autofs similar functions

> Please note that systemd's automount feature is separate from the method provided by the autofs package.  The method used by the autofs package does not use entries in `/etc/fstab`.
{: .prompt-warning }

The most notable difference between the default behaviors of SystemD's `x-systemd.automount` and the `autofs` package is:  

- The `autofs` package defaults to timing out (*umounting*) an automount **after** approximately **10 minutes** of idleness.  

- However, `x-systemd.automount` *defaults* to **never timing out**.
  - This can introduce potential points of failure.


* * *

#### Changing fstab with x-systemd.mount

More information about the updates being made can be found within the [systemd man page](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html).

- `sudo nano /etc/fstab` to edit the `fstab` file.

- Example `fstab` file with *no* edit, just network mount:
  - - `//172.21.8.12/toshiba_n300/z_Fedora_Updates /srv/fedoraLocalRepo cifs guest,dir_mode=0777,file_mode=0777 0 0`


* * *

#### Confirming automount

To confirm the default behavior of automounting:

- `x-systemd.automount`

- Example `fstab` file with `automount` edit:
  - `//172.21.8.12/toshiba_n300/z_Fedora_Updates /srv/fedoraLocalRepo cifs guest,dir_mode=0777,file_mode=0777,x-systemd.automount 0 0`


* * *

##### Speedy mount on boot

The default device timeout is 90 seconds, so a disconnected device will make your boot take 90 seconds longer to be mount before giving up. To fix:

- For devices: `x-systemd.device-timeout=7`

- For our mounts: `x-systemd.mount-timeout=7`

- Example `fstab` file with `timeout` edits:
  - `//172.21.8.12/toshiba_n300/z_Fedora_Updates /srv/fedoraLocalRepo cifs guest,dir_mode=0777,file_mode=0777,x-systemd.automount,nofail,x-systemd.device-timeout=7,x-systemd.mount-timeout=7 0 0`

> The `nofail` option is best combined with the `x-systemd.device-timeout` option.  

> Append a unit to your timeout. You can use: `s`, `min`, `h`.
{: .prompt-info }


* * *

##### Automatic unmount

You may also specify an idle timeout for a mount with the `x-systemd.idle-timeout` flag.  

- `x-systemd.idle-timeout=6min`

- Example `fstab` file with `unmounting` edits:
  - `//172.21.8.12/toshiba_n300/z_Fedora_Updates /srv/fedoraLocalRepo cifs guest,dir_mode=0777,file_mode=0777,x-systemd.automount,nofail,x-systemd.device-timeout=7s,x-systemd.mount-timeout=7s,x-systemd.idle-timeout=6min 0 0`

This will make systemd `unmount` the mount after it has been idle for `6 minutes`. 

> 6 min is equal the `360 seconds` set in `TimeoutIdleSec` under the `srv-fedoraLocalRepo.automount` file.
{: .prompt-info }


* * *

##### Restart system services for changes

Run `systemctl daemon-reload` after editing your fstab.

and then one, or both, of the following:

```
sudo systemctl restart remote-fs.target
sudo systemctl restart local-fs.target
```


* * *

## Using _dnf_local plugin on all the machines

Now you must do this all over again.

1. Make the `/srv/fedoraLocalRepo` folder to mount to.

2. Edit your `/etc/fstab` file to mount a network share to this location.

3. Install `python3-dnf-plugin-local`

4. Edit the configuration plugin to use this new location: `/etc/dnf/plugins/local.conf`

5. Add the line `repodir = /srv/fedoraLocalRepo` to `local.conf`

> Great! This machine will now also look inside the shared network mirror of our cached local Fedora repository to find any binaries it may need to update. 


* * *

## Adding automatic updates 

The main advantage of automating the updates is that machines are likely to get updated more quickly, more often, and more uniformly than if the updates are done manually. 
So while you should still be cautious with any automated update solution, in particular on production systems, it is definitely worth considering, at least in some situations. 


1. Install automatic updates:

`dnf install dnf-automatic`


2. Edit the file:

`/etc/dnf/automatic.conf`

3. Make sure this machine starts on an odd cycle, so they dont update at the same time.
Maximal random delay before downloading. Note that, by default, the systemd timers also apply a random delay of up to 1 hour.

Increase the `random_sleep` time in seconds: 

```
random_sleep = 1980
```

4. Once you are finished with configuration, execute: 

`systemctl enable --now dnf-automatic.timer`


5. Check status of dnf-automatic: 

`systemctl list-timers *dnf-*`




**Dnf and Yum in Fedora has GPG key checking enabled by default.**

Assuming that you have imported the correct GPG keys, and still have gpgcheck=1 in your /etc/dnf/dnf.conf for dnf or /etc/yum.conf for yum, 
then we can at least assume that any automatically installed updates were not corrupted or modified from their original state. 

Using the GPG key checks, there is no known way for an attacker to generate packages that your system will accept as valid and any data corruption during download will be caught. 


## Alternative thing

You could also run this command every day:
`dnf updateinfo --secseverity Critical` 
This will check for critical security updates.  



You can then choose to do full update once in a while: 
`sudo dnf offline-upgrade`


* * *

* * *

## Explaining steps taken


So the best idea is to:

use this python script to store all downloaded files in a local directory.

That local directory should be a NFS share on a fileserver everyone can access.

So you make the directory for the local files.

Open a share to a server into that directory, like an unraid share with read-write permissions.

You will then have to do this to every machine you want to be able to update using the shared network 'local' files.

Once you have a share, install the python script.

Edit the `/etc/dnf/plugins/local.conf` to set the `repodir` to the unraid share directory.

Try not to run updates in parallel, but serially. 



## Has this been done before?

This is similar to Debian's [AptCacherNG](https://wiki.debian.org/AptCacherNg). AptCacherNG, however, does [not support Fedora](https://www.unix-ag.uni-kl.de/~bloch/acng/html/distinstructions.html#hints-fccore). 



## What can I do about Flatpak updates as well?

So, Flatpak has no automatic method of caching downloads across a network and sharing them. 


## Adding a Flatpak remote for local storage

Just like with Fedora updates, you can set up Flatpak to utilize a SMB share for storing downloaded binaries. This is a great way to save data usage and streamline updates across multiple machines. 


### Mount the SMB share for Flatpak

I would ask you to refer to the section above about creating shares and mounting them. In this example, we're storing Flatpak updates in `/srv/fedoraLocalFlatpak/`


Flatpak uses [OSTree](https://ostreedev.github.io/ostree/introduction/) for managing application images. By default, it stores its repositories in `/var/lib/flatpak/`, but you can change this location by setting the `FLATPAK_USER_DIR` environment variable.


### Modify Flatpak's system directory

The environment variable `FLATPAK_SYSTEM_DIR` tells Flatpak where to put binaries. 

Run this line, then add it to your system-wide profile configuration file (e.g., /etc/profile) to make it persistent across reboots:

```bash
export FLATPAK_SYSTEM_DIR=/srv/fedoraLocalFlatpak/
```


### Initialize Flatpak's new repo

The `FLATPAK_SYSTEM_DIR` should be changed, you will need to initialize Flatpak.


```bash
flatpak --system remote-add --no-gpg-verify --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

and then begin the update:

```bash
flatpak --system update
```

This command updates the Flatpak repositories at the system level, from the configured repositories.



### Changing default Flatpak repositories


1. Add SMB share as a system-wide repository

```bash
flatpak remote-add --no-gpg-verify --system smb-repo /path/to/mount/flatpak/repo
```


2. Add Flathub as a user repository

```bash
flatpak remote-add --no-gpg-verify flathub https://flathub.org/repo/flathub.flatpakrepo
```

3. Prioritize repositories

By default, Flatpak will prioritize the user repository over the system repository. To change this behavior and prioritize the system repository:

```bash
flatpak config --set --system --default-remotes smb-repo,flathub
```


With this configuration, Flatpak will first look for updates in the system-wide repository (SMB share). If the application is not found or needs updating, Flatpak will then fall back to Flathub to download the updates directly from there.




### Installing Flatpak's from the new repo

```bash
flatpak --system install flathub org.mozilla.firefox
```

When you run this command, Flatpak will fetch the Firefox application from the Flathub repository and install it on your system. 
Since we've configured Flatpak to use the SMB share for storing downloaded binaries (FLATPAK_SYSTEM_DIR), the downloaded files will be stored on the SMB share rather than being downloaded individually on each machine. 



## Flatpak airgapped sneakerwear updates

* * *

Flatpak does support [installing from sideload repos](https://docs.flatpak.org/en/latest/flatpak-command-reference.html).

> This is, again, not automatic. You will need to create an offline version of each Flatpak, and then install that Flatpak through a sideloaded repo. 


* * *

An example command from [the documentation for using USB Drives](https://docs.flatpak.org/en/latest/usb-drives.html) for sneakerwear is:

- Load the Flatpak onto the NAS:

`flatpak create-usb /media/user/FLATPAKS org.gnome.gedit`

- Install the Flatpak onto NAS connected machines requiring updates:

`flatpak update --sideload-repo=/media/user/FLATPAKS/.ostree/repo flathub org.gnome.gedit`


* * *

This can be accomplished by seeing what Flatpacks need updates:

`sudo flatpak remote-ls --updates`

1. On the initial update machine:
> You will need a script to copy the **Application ID** and go through each Application ID after it has been updated, package it for USB,

2. On the secondary update machines:
> You need to check what updates are needed, and iterate through them checking to see if the **--sideload-repo** has the file available. If not, just install normally:

`flatpak install --sideload-repo=/media/user/FLATPAKS/.ostree/repo flathub org.gnome.gedit`



### Flatpak custom installation

If all of your machines are on the same server, storage, or network -- and they never move. You could just install Flatpak apps in one location, no need for updating.

- [Using a custom installation](https://docs.flatpak.org/en/latest/flatpak-command-reference.html#flatpak-installation)

Also, **System-wide remotes** can be statically preconfigured by dropping flatpakrepo files into `/etc/flatpak/remotes.d/`.

- [More information about Flatpak remotes](https://man7.org/linux/man-pages/man5/flatpak-remote.5.html)

- [How Flatpak install/updates are processed](https://github.com/flatpak/flatpak/issues/5365)

- [Example adding Flathub repos](https://superuser.com/questions/1747288/why-cant-i-add-flatpak-repos)

- [Even more information about Flatpak repos](https://github.com/flatpak/flatpak/blob/main/doc/flatpak-repo.xml)

- [Even more information about Flatpak remotes](https://github.com/flatpak/flatpak/blob/main/doc/flatpak-remote.xml)


* * *

Flatpakrepo files can also be placed in `/usr/share/flatpak/remotes.d/` and `/etc/flatpak/remotes.d/` to statically preconfigure system-wide remotes. 

Such files must use the `.flatpakrepo` extension. If a file with the same name exists in both, the file under `/etc</` will take precedence.

```ini
[Installation "extra"]
Path=/location/of/sdcard
DisplayName=Extra Installation
StorageType=network
```

Possible `StorageType`: network, mmc, sdcard, harddisk.




# Sources:

- [fedoraforum](https://forums.fedoraforum.org/showthread.php?324608-Creating-a-local-mirror-for-an-internal-network)

- [fedoramagazine](https://fedoramagazine.org/use-the-dnf-local-plugin-to-speed-up-your-home-lab/)

- [dataswamp](https://dataswamp.org/~solene/2023-04-05-lan-cache-flatpak.html)

