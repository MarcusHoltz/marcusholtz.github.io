---
title: Proxmox Post-Install User Creation
date: 2023-02-03 11:33:00 -0700
categories: [Proxmox, ProxmoxInstall]
tags: [proxmox, linux, usermanagement, install]
image:
  path: /assets/img/header/header--proxmox--proxmox-postinstall-user-creation.jpg
---

# Post Install Proxmox 

Not using LDAP? Let's use PAM (Pluggable Authentication Modules), in its little corner of the /etc directory.



## Connect to the Proxmox VE web interface

Connect to the admin web interface (http**s**://youripaddress:8006). If you have a fresh install and didn't add any users yet, you should use the root account with your linux root password, and select "PAM Authentication" to log in.


* * * 
## Commands to create a new Super Administrator for logging in instead of using root

Assuming you added your user from the Proxmox Datacenter web interface, get to a terminal and type:

`pveum groupadd CustomSuperUsers -comment "System Administrators That Can Dance Belong Here"`

The command above **Generates The Following Line:** 

`group:CustomSuperUsers:System Administrators That Can Dance Belong Here:`


* * *
Cool, now we have a CustomSuperUsers group created. Now for The Secret Sauce ⟨™⟩ command below: 

`pveum aclmod / -group CustomSuperUsers -role Administrator` < - -  The Secret Sauce

The secret sauce command above **Generates The following Line:** 

`acl:1:/:@CustomSuperUsers:Administrator:`


* * *
Yay, now our CustomSuperUsers really are superusers, or, administrators. The last bit, is to add a user to that group:

`pveum usermod dancecommander@pam -group CustomSuperUsers --comment "Administrator that can samba"`

**Adds User with the following Line:** 

`group:CustomSuperUsers:dancecommander@pam:System Administrators That Can Dance Belong Here:`


* * *
Everything should be complete for us to use a user we created instead of root to accomplish most tasks. 

We can verify that with: `cat /etc/pve/user.cfg`
* * *




