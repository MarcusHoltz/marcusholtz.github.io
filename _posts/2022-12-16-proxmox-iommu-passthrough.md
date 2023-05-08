---
title: Proxmox IOMMU for PCI Passthrough
date: 2022-12-16 11:33:00 -0700
categories: [Proxmox, ProxmoxInstall]
tags: [virtualization, proxmox, iommu, passthrough]
image:
  path: /assets/img/header/header--proxmox--proxmox-iommu-groups.jpg
---

# Edit the host OS

## Edit GRUB

First edit grub to allow IOMMU based on your chipset manufaturer, and have the motherboard split up the IOMMU groups. This is basically the same thing that UnRAID does.

`sudo nano /etc/default/grub`

once inside of grub, comment out the "quiet" one and add this line:

`GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on pcie_acs_override=downstream,multifunction"`

then update grub

`sudo update-grub`

## Edit the modules for the kernel

With this change you add the required modules for IOMMU 

`sudo nano /etc/modules`

add the following:

```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```


## REBOOT

Great idea to reboot right about now.
