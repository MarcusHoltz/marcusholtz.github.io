---
title: Dual Boot Linux and Windows with Hibernation and Encryption
date: 2024-02-08 11:33:00 -0700
categories: [Linux, Install]
tags: [linux, dualboot, windows, hibernation, uefi]
pin: false
image:
  path: /assets/img/header/header--linux--uefi-fixes-install-hibernation-encryption.jpg
  alt: Windows and Linux on dual boot single drive system encrypted with hibernation and working uefi
---

# Dual boot Windows and Linux both encrypted and capable of hibernation

# Emergency Fix for Dual Boot 

Presumably anyone reading this has mostlikly already attempted to dual boot and is here for a fix or information. Please skip to [// end dual boot uefi fix](#// end dual boot uefi fix) if you do not need an emergency fix for dual boot.

Let's review the most common problem after installing a dual boot linux based OS.... UEFI 


## Repair/Remove boot information in UEFI

Presuming working in a Linux enviornment. This could be from a USB boot device or from another partition.


* * *

### Locating the UEFI partition

First, find the UEFI partition to verify all the information we're going to chug through.

1. `lsblk` to show all the devices read by the system

Looking for the device with the boot partition, generally, you'll find a drive with several partitions on it.
In this case, we've found `nvme0n1`.

The UEFI partition is, generally, the first partition on the drive. It is also one of the smallest, with the EFI boot partition sometimes only being 100MB. 

The `/boot` mount often has it's own partition as well, this is to store Kernels and other items required to boot Linux. We do not want this, we're specifically looking for `UEFI`.

So, attempting to find our correct partition, let's mount them up and see what it produces:

2. `sudo mount /dev/nvme0n1p1 /mnt`

3. Now going to look into that folder: `ls /mnt`

`EFI   mach_kernel   System   'System Volume Information'`

Based on the `ls /mnt` list of directories above, we're getting a lot of information.

The one we're looking for, the `EFI` directory.

4. `cd EFI` to go into that directory and find all of our old boot objects. `ls` will show something like:

`Boot   fedora   Microsoft   qubes`

Now we know we've located the UEFI partition and found our EFI boot objects.


* * *

### Making changes to the UEFI boot partition

#### Changing the EFI boot order

1. In the terminal run: `sudo efibootmgr` to see everything bootable.

We're going to locate the offending item, disable it, and reboot.

`efibootmgr` takes the order of the system to boot with the `-b` flag.

So on your list that `efibootmgr` printed, you will see something like:

```bash
Boot0000* Windows
Boot0001* Manjaro
```

To disable Manjaro, we're using the `-A` flag.

The entire command looks like:

2. `sudo efibootmgr -b 0001 -A`

Now when you run `efibootmgr` it should display:

```bash
Boot0000* Windows
Boot0001 Manjaro
```

Without the `*` making the distinction as set-to-boot, it will not appear on GRUB or be loaded.

Reboot.


* * *

#### Remove the EFI boot 

To remove, like forever, the EFI boot entry:

1. `sudo efibootmgr -b 0001 -B`

Please keep in mind to check the number you're removing. DO NOT MAKE A MISTAKE.


* * * 

#### Removing any EFI eating up space

You can also remove any efi boot partitions in your `EFI` directory.

```bash
Boot   fedora   Microsoft   qubes
```

to

```bash
Boot    Microsoft 
```

## // end dual boot uefi fix


# Windows system preperation

This tutorial is written from the example that Windows XX has already been installed on a working Laptop/Desktop already. The Windows install is also using 100% of the physical disk space available on the machine. 

## Booting Windows

Boot into Windows, and make sure you have Administrative privileges available to you.


* * *

### Resize the Windows partition

#### Open Computer Management

There are a varriety of ways to open `Computer Management`, feel free to pick one:

1. Open `compmgmt.msc`.

2. If you prefer using the keyboard, you can press `Windows + X`, followed by `G`. 

3. You can also use Windows' file browser, `explorer`. Open `explorer` and right click on `This PC` in the Navigation pane, and click `Manage`.

4. Now click on `Disk Management` under `Storage` section in the Computer Management window.


* * *

#### Shrinking the Windows partition

To make space for another operating system, we must first resize our prospective drive.

With `Disk Management` open, find the Disk you're doing to resize. Take note of the size of the disk on the left.

With an idea of how much space you need in Windows and how much you can allocate to your new operating system, let's resize this partition.

Refering to the image below:
[!image](image)

1. `Right click` on the partition to resize.

2. Click `Shrink Volume`.

3. Windows will do some math.

4. Choose your new size for the Windows partition and click `Shrink`.

5. Wait, and then look at the new black 'unallocated' section of your Disk. 


* * *
 
### Prepare Install media

Make sure you have a USB flash drive ready with an OS prepared for install. If you dont, I recomend: 

- [DistroWatch](https://distrowatch.com/) - To help find a distribution that meets your needs.

- [Repology](https://repology.org/) - Double check that distribution is going to meet your needs.

- [Ventoy](https://www.ventoy.net/) - Put downloaded Live distributions on a USB inorder to test to ensure it will meet your needs.

- [Rufus](https://rufus.ie/) - Get your new favorite distribution on a USB drive for install.

- [BalinaEtcher](https://etcher.balena.io/) - If your distribution you chose is too esoteric to work, try writing it to USB with this.


### Done with Windows, for now

Shut your computer down, and move all of your prayer statues closer.



# Install Linux for Dual Boot

## BIOS settings

Your BIOS has a ton of different settings that vary by manufacturer. Please look through these as many settings will prevent any of this from working. 

1. UEFI settings - Make sure UEFI is functioning and all hardware supports it.

2. Boot order - Look through your devices boot order, make sure the USB flash drive will boot before Windows. 
   1. Sometimes the boot order displays UEFI OS or just the drive's name. Take note.

3. SecureBoot

One of the most common reasons for an issue, and this is most likely true for Windows 8/10 laptops, is that SecureBoot is enabled. Disable SecureBoot in your BIOS; for my ThinkPad this involves:

- Hit [Enter] to interrupt the normal boot process
- [F1] to enter BIOS setup
- Navigate to "Security" in the sidebar
- Navigate to "Secure Boot" in the main area and turn it off
- [F10] to save & exit

As for whether you should disable SecureBoot, I recommend doing some research into this topic, since SecureBoot provides security benefits you might value.


## Booting from USB flash drive

On most systems, hitting `F12` rapidly after your computer turns on will give you a boot menu. 

If not, go back and check out the BIOS settings if you were unable to boot from the USB flash drive.


## Backing up before Installing your new Operating System

Be sure anything important that may get destroyed has been backed up. This means your Windows files. 

Linux should be able to see your drive, and also have access to the network. Go ahead and try and backup anything you may have forgotten. 



### Concerns? Backup your entire drive

If you’re concerned, feel free to backup your drive before you do anything:

```bash
#on pc to copy
dd bs=16M if=/dev/sda|bzip2 -c|nc serverB.example.net 19000

#on remote storage
nc -l 19000|bzip2 -d|dd bs=16M status=progress | gzip -c > ~/backup_dd_disk.img.gz

#To restore the image back to the disk, the following command can be used:
gunzip -c /storage/images/backup_dd_disk.img.gz | dd of=/dev/sda
```

## Install onto the unallocated space

If you're staring at an install screen, start going through it. 

Generally, there are geographic and language screens first. You have to go through this localization before any 'install' begins. 

1. Install processes vary. Be patient.

2. Fill out everything you can until you get to `Disk Partitioning`.


### Disk setup for Install

Linux needs several partitions, that can be done in a variety of ways, to be created before install. Ubuntu does not seem to have an automatic option to partition the drive alongside Windows, from my personal attempts. 

You will find automatic partitioning in Arch and Red Hat based systems.


#### Distrobution installers

##### Ubuntu based systems

Distros using Ubiquity installer (Ubuntu and it's variants, Kubuntu, MATE, Elementary OS) will let you manually install the OS alongside Windows. This means setting up all system partitions alongside Windows. While the sizes and location of your partitions will be different, I have an excelent write up on how to manually partition (and encrypt) the drive before install.


###### Manual partition drive before install

Here you can find my write up on how to partition and encrypt a drive: [Use LUKS to encrypt your laptop’s data](https://blog.holtzweb.com/posts/using-LUKS-to-encrypt-your-linux-laptop/)


##### Arch based systems

Distros using Calamres installer (Garuda Linux, Manjaro, KDE neon, EndeavourOS, OpenMandriva Lx, and the Live medium of Debian) will let you visually select the partiton to replace. You can do this in the `Partitions` section of the installer.

1. Under the `Partitions` part of the installer, after localization.

2. You will see several icons, and a drop down menu. The drop down menu is to select the disk to install to. Be sure you've got the right disk selected to work with.

3. Click the `Replace partition` icon that looks like two almost empty looking disks with an arrow connecting them.

4. At the bottom of the screen you will see your hard drive's partition table laid out. 

5. Find the dark grey section, under `Current:` and click on it.

6. You will see the secion labeled `After:` change to indicate the selection.

7. Check the `Encrypt system` checkbox directly above the partition table selection and enter your passphrase.

8. At the bottom of the window is the `Boot loader location`. Make sure this is the drive you're currently using.

9. With everything confirmed, hit `Next`.

10. Finish your install.


##### RHEL based systems

Distros using Anaconda installer (Red Hat Enterprise Linux, Oracle Linux, Rocky Linux, AlmaLinux, CentOS, Qubes OS, Fedora) you can have the system automatically allocate the space for the new operating system with a few clicks. 

This is a visual installer, so everything is just icons. You need to click on the section to configure. 

1. Find the `Installation Destination` section of the installer.

2. Make sure the drive in `Drive Selection` you want to install to is checked. 

3. Under `Storage Configuration` select the `Custom` option.

4. Click `Done` in the upper left hand corner of the screen.

5. A new window, `Manual Partitioning` will appear. 

6. Inside this new window, 
   1. Select the `partitioning scheme` to your preference with the drop down menu.
   2. Check the `Encrypt my data.` box.
   3. Then find the blue text at the top, `Click here to create them automatically.`
   4. Click that blue text.

7. You should see your new partition scheme laid out infront of you. 

8. Adjust any of the sizes as needed. The home directory, the root directory, the boot directory, or the efi directory.
   1. The `home directory` will be where users store their files... appimages, python sandboxes, music, downloads, firefox cache, etc. 
   2. The `root directory` is where the system stores its files. These grow in size very little, with the exception of adding something like docker.
   3. The `boot directory` stores everything the system needs to boot. The largest files in here are your Kernel files, they reside after an update as a just-in-case. So this directory can grow in size based on how often your distro pushes released and how often you download them.
   4. The `efi directory` stores the method your computer uses to begin. This is based on the installed distro and grows with each new distro installed.

9. Post adjustments, and space allocation - click the `Done` button.

10. Enter your `passphrase` to decrypt your drive. 
    1.  Remember this password. 
    2.  You cannot open the drive without it. 
    3.  There is no recovery key.

11. You may continue the configuration of the system and `Begin Installation`.


# Setup hibernation

Explain hibernation

Talk about power use


## Run Power Management baseline
`sudo powertop -c -i 10 --html=/home/lockntross/Documents/powertop-power-report.html`






## Restore Hibernation on Fedora

These are the steps I had to take on `Fedora` to be able to close my `ThinkPad` and have it go to `HybridSleep`.


### Swap creation

It's very likly you already have something in play for swap.

`swapon --show`

If this isnt a partition or swap file bigger than your RAM, you're going to need to follow the directions below:

If swap is insufficiently-sized you should disable the existing swap on the running system:

`swapoff /dev/dm-2`

and disable it for future boots by deleting the corresponding entry from `/etc/fstab`

Now to add a sufficiently-sized swap file.


#### Removing Zram0

If you see `zram0` in the list displayed by `swapon --show`, takes these steps to disable it:

`sudo systemctl --all --type swap`

Take note of the zram name.

You can now mask it with the command:

`sudo systemctl mask "dev-zram0.swap"`


#### Set swap, so you can set hibernate

Follow these commands in bash to create a swapfile, and add it to your file systems mounted on boot. Be sure to change the filesize to the amount of ram you have on your system.

These commands were run as root.

`fallocate -l 18G /swapfile`

`chmod 600 /swapfile`

`mkswap /swapfile`

`echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab`

`swapon /swapfile`



* * * 

`ll /dev/disk/by-uuid`


Now if you are using LUKS encryption then select the disk that points to the un-encrypted disk or rather the device mapped by the kernel after it has opened it using your passphrase at the time of boot. In my case it is the one linked to `dm-1`

So the resume parameter should point to the disk which has the swap file. **You don’t need to give the swap file name itself.**

So the final resume parameter is

`resume=/dev/disk/by-uuid/aff8758c-f870-482c-b86f-acb52047868e`

Notice the swap file name is not needed. We have the `resume_offset` parameter for that. To find that just run the `filefrag` command on the swap file like so:

`sudo filefrag -v /swapfile | head -n 4 | tail -n 1 | awk '{print $4}'`

You should get an output like:
`65536..`

Copy out that number and add it to `resume_offset` parameter in grub like so:

`resume_offset=65536`


* * * 

Open the grub config:

`sudo nano /etc/default/grub`

So the final GRUB_CMDLINE_LINUX should be something like:

```bash
GRUB_CMDLINE_LINUX="rd.luks.uuid=luks-2ec7f1a-6f9=b-896-a2-b80e9d2f4 rd.lvm.lv=vgfedora/fedora resume=/dev/disk/by-uuid/03aef3ba-dca1-4cba-a3f5-36c5c0fe948e resume_offset=65536"
```

On UEFI systems you may want to disable the `grub2-osprober`, which is why I have added the line:

`GRUB_DISABLE_OS_PROBER=true`

After all this is done we can make the grub config by

`grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg`

Note: On MBR systems you may need to find out where your `grub.cfg` is located, the above command works for me on UEFI.

And that’s it. Test out hibernate by
`systemctl hibernate`



## dracut

Next you will need to add the resume module to `dracut`.
You can add this into following line into `/etc/dracut.conf`

`add_dracutmodules+=" resume "`


And then re-generate your initramfs using:

`dracut -fv`

Why this is not added by default, and why resume from swap partition works without adding resume to dracut, I will never know. 

Finally, the initramfs, which is managed by dracut. Double check it by running:

`dracut --print-cmdline`

Then reboot.

Then run 

`dracut -f` 

again.

Then test hibernate.

Reboot.

Test again.

Ok should work.



## Restore Hibernate on Ubuntu

This assumes you have a swap partition ready to use (if you are using a swap file you cannot hibernate using this method). 

Install base utilities needed:
`sudo apt install pm-utils hibernate`

Then:
`cat /sys/power/state`
You should see:
`freeze mem disk`
Then run one of the following lines:
`grep swap /etc/fstab`
or
`blkid | grep swap`
Copy the UUID value. You will need it later.

Now let's edit GRUB:
`sudo nano /etc/default/grub`
Change the line that says:
`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`
so that it instead says:
`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=<YOUR_COPIED_UUID>"`
Be careful not to miss the UUID= part.

Then, after saving the file and quitting the text editor, run:
`sudo update-grub`

To test it, run:
`sudo systemctl hibernate`


### Add Hibernate to the Kubuntu Menu 

Paste and Run this command:
```
cat << "EOF" > /etc/polkit-1/localauthority/50-local.d/com.ubuntu.enable-hibernate.pkla
[Re-enable hibernate by default in upower]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=yes

[Re-enable hibernate by default in logind]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions;org.freedesktop.login1.hibernate-ignore-inhibit
ResultActive=yes
EOF
```


## Hibernate on Arch based systems

I have not tried this yet and should check into this.


# No EFI fixes needed

If everything went well, there's no need to use the UEFI fixes mentioned in the begining. 

You should see both your Windows and Linux UEFI choices in your BIOS and or on GRUB. 


# Encrypt your Windows partition

Windows includes BitLocker. It will encrypt your OS drive on the fly, even after installation.

To use BitLocker and encrypt your OS:

1. Boot back into Windows.

2. Open `My Computer`.

3. Right click on your `C` drive (your drive with Windows installed).

4. Choose `Turn on BitLocker`. 

5. Follow the wizard to complete the encryption.

6. Next time you go to boot Windows you will be prompted with a screen to enter your password to unencrypt the Windows partition for use.




