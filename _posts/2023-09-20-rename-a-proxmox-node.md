---
title: Rename Nodes on a Proxmox Cluster
date: 2023-09-20 11:33:00 -0700
categories: [Proxmox]
tags: [proxmox, cluster, hack]
pin: false
image:
  path: /assets/img/header/header--proxmox--rename-proxmox-node.jpg
  alt: Rename Proxmox Nodes without loosing anything
---

# Rename a Proxmox Node

## Renaming a Proxmox Node isnt a right click feature

In order to rename a node, you basically have to move everything around. This isnt an intended feature, and the official documentation basically says to not do it.


## Rename Proxmox Node Script

To help make the node renaming process go smoother -- I have created a simple script to take care of renaming the node. 

> This is a two step process, the script needs ran a second time, after reboot.
{: .prompt-warning }

1. `Copy and execute` the script below.

2. `Follow directions` and prompts within the script.

3. The system will reboot after `Stage 1` is complete.

4. After reboot, get back into the machine, and `re-run the script` again.

5. `Stage 2` will complete, finish, and clean up.

6. You should now see your `new node name` in the web interface.


* * * 

## SCRIPT

```bash
#!/bin/bash
#
# Future to-do:
# Possibly, add section for the edits you could make to /etc/resolv.conf
#
#
#
# This script needs to be run as root
#
if [ "$EUID" -ne 0 ]
  then echo "Please run as root"
  exit
fi
#
# This script takes input from the user for a new hostname, 
# saves this input as NEW_HOSTNAME and the current hostname as OLD_HOSTNAME to a file (for persistance through reboot).
#
if [ -f "/root/pve-nodes-to-rename.tmp" ]; then
    echo -e "\nStage 2 of script begining:"
else
#
#
# RUN SCRIPT IN STAGE 1
#
echo -e "\nPlease enter the node's new name:"
read -s NEW_HOSTNAME
echo -e "** $NEW_HOSTNAME ** will be set for the new name. \nPlease exit now if there was a typo. " && sleep 5; echo -e "Ok, no typo. Resuming..."; sleep 2;
echo "OLD_HOSTNAME=$(hostname)" > /root/pve-nodes-to-rename.tmp && echo -e "\ntemp file created."
echo "NEW_HOSTNAME=$NEW_HOSTNAME" >> /root/pve-nodes-to-rename.tmp && echo -e "hostnames entered into temp file."
sleep 1;
#
# This line reads the file we created into shell variables using a Process Substitution. This takes the output of a process, not just the stout, and reads it into the current shell environment.
. <(cat /root/pve-nodes-to-rename.tmp) && echo -e "shell variables loaded into memory."
sleep 1;
#
# Move the original file to a .bak extension, and replace the current hostname with the entered new node name.
sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/g" /etc/hostname && echo -e "/etc/hostname file sucessfully edited."
#
# Do the same as the hostname with the hosts
sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/g" /etc/hosts && echo -e "/etc/hosts file sucessfully edited."
#
# edit mailname if it exists
[ -e "/etc/mailname" ] && sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/g" /etc/mailname
#
# edit main.cf if it exists
[ -e "/etc/postfix/main.cf" ] && sed -i.bak "s/$OLD_HOSTNAME/$NEW_HOSTNAME/g" /etc/postfix/main.cf
#
# copy config files to new node name and dont send errors to console
cp "/var/lib/rrdcached/db/pve2-node/$OLD_HOSTNAME" "/var/lib/rrdcached/db/pve2-node/$NEW_HOSTNAME" -r  > /dev/null 2>&1;
cp "/var/lib/rrdcached/db/pve2-storage/$OLD_HOSTNAME" "/var/lib/rrdcached/db/pve2-storage/$NEW_HOSTNAME" -r  > /dev/null 2>&1;
cp "/var/lib/rrdcached/db/pve2-$OLD_HOSTNAME" "/var/lib/rrdcached/db/pve2-$NEW_HOSTNAME" -r  > /dev/null 2>&1;
echo -e "rrdcache has been moved to the new node."; sleep 2;
echo -e "\n#################################################################\n                    A reboot will occur,\n      please run this script again after reboot completes.\n#################################################################\n"
REBOOT_INSECS=5
while [[ 0 -ne $REBOOT_INSECS ]]; do
    echo "Rebooting in $REBOOT_INSECS.."
    sleep 1
    REBOOT_INSECS=$[$REBOOT_INSECS-1]
done
sleep 2;
#
# reboot and continue with other half of script
reboot now
fi
#
#
#
# STAGE 2 BEGIN
#
# begin stage 2
echo -e "\nThank you for starting stage 2.\n"
sleep 2;
# read variables into memory
. <(cat /root/pve-nodes-to-rename.tmp) && echo -e "shell variables from before reboot loaded back into memory."
sleep 2;
#
# update storage config
sed -i.bak "s/nodes $OLD_HOSTNAME/nodes $NEW_HOSTNAME/g" /etc/pve/storage.cfg && echo -e "updated storage config."
sleep 1.5;
#
# mv vm configs
mv /etc/pve/nodes/$OLD_HOSTNAME/qemu-server/*.conf /etc/pve/nodes/$NEW_HOSTNAME/qemu-server/ && echo -e "moved VM configs."
sleep 1.5;
#
# mv ct configs
mv /etc/pve/nodes/$OLD_HOSTNAME/lxc/*.conf /etc/pve/nodes/$NEW_HOSTNAME/lxc/ && echo -e "LXC configs have been moved as well.\n"
sleep 3;
#
#
#
# SCRIPT CLEANUP
#
# read variables from ini
. <(cat /root/pve-nodes-to-rename.tmp) && echo -e "Cleanup shall begin, removing ' $OLD_HOSTNAME '."
sleep 3;
#
# Begin file cleanup from using `sed -i.bak`
rm /etc/hostname.bak && rm /etc/hosts.bak
[ -e "/etc/mailname.bak" ] && rm /etc/mailname.bak
[ -e "/etc/postfix/main.cf.bak" ] && rm /etc/postfix/main.cf.bak
rm /var/lib/rrdcached/db/pve2-node/$OLD_HOSTNAME -r  > /dev/null 2>&1;
rm /var/lib/rrdcached/db/pve2-storage/$OLD_HOSTNAME -r  > /dev/null 2>&1;
rm /var/lib/rrdcached/db/pve2-$OLD_HOSTNAME -r  > /dev/null 2>&1;
rm /etc/pve/nodes/$OLD_HOSTNAME -r  > /dev/null 2>&1;
rm /etc/pve/storage.cfg.bak
rm /root/pve-nodes-to-rename.tmp && echo -e "\nCOMPLETE! Temp/Backup files have been removed.\n########################################################\nCheck out new node ' $NEW_HOSTNAME ' on the PVE web interface.\n########################################################\n"
```

Thanks, hope this helps someone.

[Source](https://www.youtube.com/@i12bretro/videos)
