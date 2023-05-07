---
title: Proxmox Backup Server
date: 2022-12-16 11:33:00 -0700
categories: [Proxmox, ProxmoxBackupServer]
tags: [virtualization, proxmox, storage, backups, install]
image:
  path: /assets/img/header/header--proxmox--proxmox-backup-server.jpg
---

# Install Proxmox Backup Server

(This was all done earlier, but is a requirement for installing PBS)

Make sure you have added the repo: 

`wget http://download.proxmox.com/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg`

then:

edit `/etc/apt/sources.list` and add:

`deb http://download.proxmox.com/debian/pbs buster pbs-no-subscription`


* * * 

Update then install both client and server:

`apt update; apt upgrade -y`

`apt-get install proxmox-backup`


- [Instructions about the whole PBS thing](https://pbs.proxmox.com/docs/installation.html)

- [The real PBS instruction manual](https://www.proxmox.com/en/downloads/category/documentation-pbs)


* * * 
## Create a new PBS user

list current users:

`proxmox-backup-manager user list`

You can add a new user with the user create subcommand below or through the web interface, under the User Management tab of `Configuration -> Access Control`.

`proxmox-backup-manager user create dancecommander@pam --comment "Administrator that can samba"`

Newly created users do not have any permissions. 

You can manage permissions via Configuration -> `Access Control -> Permissions` in the web interface. Likewise, you can use the acl subcommand below to manage and monitor user permissions from the command line. 

`proxmox-backup-manager acl update / Admin --auth-id dancecommander@pam`

Once complete we can list the permissions with: 

`proxmox-backup-manager acl list`
