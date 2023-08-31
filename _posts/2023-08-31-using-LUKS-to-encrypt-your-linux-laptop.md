---
title: LUKS Encrypt Linux Laptop
date: 2023-08-14 11:33:00 -0700
categories: [Linux, Security]
tags: [luks, bootloader, scripts, security, cryptography, passwords]
pin: false
image:
  path: /assets/img/header/header--luks--encrypt-linux-drive.jpg
  alt: Create an encrypted volume to mount for your OS
---
# Use LUKS to encrypt your laptop's data

Everyone should have their mobile data encrypted. Your cell phone does this. Why doesnt your laptop?

You must enter your PIN/PASSWORD for using your mobile device, this PIN unlocks the encrypted storage on your mobile device -- allowing you to use it seemlessly. 


## There are several offerings by your favorite closed source operating system:

- Windows has Bitlocker

- MacOS has FileVault


### But Linux has user choice

That's right. You get to decide how to implement this. 

Ubuntu, Debian, Arch, Fedora -- all do it the same. Encrypt the bootloader as well.

Why? Because that's just one more potential attack vector that can be used to break into your laptop.

Well, I'm not Secret Agent 86. No government actor is after my collection of sweet memes.  I just dont want my stuff getting out there if someone grabs my laptop and it ends up at a pawn shop. 


# New Laptop that needs disk encryption

**In this example, we have a new laptop that needs a new Kubuntu install put on it.**

> This does not have to be a new laptop, or even a laptop at all. This process is for DISK ENCRYPTION.
{: .prompt-tip }


* * * 

## What steps will we be performing?

1. Booting from LIVE install media.

2. Create a new partition table.

3. Erase the disk.

4. Create partitions and assign labels.

5. Make filesystems.

6. Encypt our data.

7. Create LV inside of encrypted data.

8. Begin Install.

9. Stuff crypt info into grub to allow LUKS boot.

10. Forget Password.

11. Repeat.

* * * 

## TL;DR -- One more time, what are we doing?

- Boot a live distro

- Create partitions for data

- Encrypt ONLY our operating system & personal data


### Dont forget your password

Final reminder, you must be sure you have this password somewhere... as you cannot get access to this drive without it.

* * *

# Boot from LIVE install media

Once you're booted into the LIVE install media, open a terminal.

- To make typing commands easier and avoiding constant sudo:

```bash
sudo -i
```


## That password you said youd never forget

- Export the password we want to use for our drive's encryption by setting an Environment Variable:

```bash
export SDD_PASS=SuperSecurePassword1234MyMomsLastNameandBirthdayandMyFavoriteColorandMyDogsNameandCamelCase
```


## Wipe that disk

- First step is to create a partition table on our disk:

> This will DESTROY anything on disk. Be sure you're using the correct disk if you have multiple.
{: .prompt-info }

- Make sure you know what disk you're installing to (typically /dev/sda):

```bash
lsblk
```

You should see a list of all of the drives attached to your PC, their mountpoints, and size.

Presumably, this tutorial is going to use `/dev/sda` as the drive we need to install to.


### Begin disk data destruction

- If you're more comfortable making changes using the GUI, there is a graphical option:

```bash
apt update; apt -y install gparted
```

- Otherwise, from terminal:

``` bash
sudo parted /dev/sda mklabel gpt
```


## Partitioning the Disk

#### This script uses the `sgdisk` command to partition the `/dev/sda` disk.

- Create your partitions for install:

```bash
sgdisk --zap-all /dev/sda
sgdisk --new=1:0:+768M /dev/sda
sgdisk --new=2:0:+2M /dev/sda
sgdisk --new=3:0:+256M /dev/sda
sgdisk --new=4:0:0 /dev/sda
sgdisk --typecode=1:8301 --typecode=2:ef02 --typecode=3:ef00 --typecode=4:8301 /dev/sda
sgdisk --change-name=1:/boot --change-name=2:GRUB --change-name=3:EFI-SP --change-name=4:Encrypted-ROOT /dev/sda
sgdisk --hybrid 1:2:3 /dev/sda
sgdisk --print /dev/sda
```

### Let's break down what each part of the script is doing:

   - `sgdisk --zap-all /dev/sda`: This command removes all existing partition data on the `/dev/sda` disk.
   
   - `sgdisk --new=1:0:+768M /dev/sda`: Creates a new partition (Partition 1) of 768MB in size.
   
   - `sgdisk --new=2:0:+2M /dev/sda`: Creates a new partition (Partition 2) of 2MB in size.
   
   - `sgdisk --new=3:0:+256M /dev/sda`: Creates a new partition (Partition 3) of 256MB in size.
   
   - `sgdisk --new=4:0:0 /dev/sda`: Creates a new partition (Partition 4) using the remaining space on the disk.
   
   - `sgdisk --typecode=1:8301 --typecode=2:ef02 --typecode=3:ef00 --typecode=4:8301 /dev/sda`: Sets partition types for each partition.
   
   - `sgdisk --change-name=1:/boot --change-name=2:GRUB --change-name=3:EFI-SP --change-name=4:Encrypted-ROOT /dev/sda`: Changes the names of the partitions for identification.
   
   - `sgdisk --hybrid 1:2:3 /dev/sda`: Sets up the hybrid MBR/GPT scheme.

## Filesystem Formatting

- Create unencrypted filesystems for data to reside in:

```bash
mkfs.ext4 -L boot /dev/sda1
mkfs.vfat -F 16 -n EFI-SP /dev/sda3
```


### Taking a look at each parition and filesytem:

- NOTE: These are the unencrypted filesystems. Our encrypted filesystem can only be made after we have encrypted a partition.

   - `mkfs.ext4 -L boot /dev/sda1`: Formats Partition 1 as an ext4 filesystem with the label "boot".

   - `mkfs.vfat -F 16 -n EFI-SP /dev/sda3`: Formats Partition 3 as a FAT16 filesystem with the label "EFI-SP".


## Disk Encryption

- This is where we give your password to LUKS to encrypt your data.

**PLEASE** be sure you have it saved somewhere.

```bash
echo -n "${SDD_PASS}" | cryptsetup luksFormat /dev/sda4 - 
echo -n "${SDD_PASS}" | cryptsetup open /dev/sda4 ENCRYPTED_DATA - 
```

### OH no. What did I just do?

- Again, only one partition is encrypted. This is where we keep our operating system and personal files.
  
   - `echo -n "${SDD_PASS}" | cryptsetup luksFormat /dev/sda4 -`: Initializes Partition 4 as a LUKS encrypted partition with the password from the `SDD_PASS` variable.
   
   - `echo -n "${SDD_PASS}" | cryptsetup open /dev/sda4 ENCRYPTED_DATA -`: Opens the encrypted partition using the LUKS encryption with the same password.

## Data to Encrypt with Logical Volume Management (LVM)

- This is the partition you will install your operating system to. As this LVM is inside of a LUKS partition, you will need to locate it through `/dev/mapper`

- The only change needed is for system hybernation. Hybernation stores your RAM as encrypted. No matter what state your computer is in, data is encrypted. 

> The setting for SWAP is set to 9GB. Please modify to include `the amount of RAM your system has + 1GB`.
{: .prompt-tip }

```bash
vcreate /dev/mapper/ENCRYPTED_DATA
vgcreate ubuntu-vg /dev/mapper/ENCRYPTED_DATA
lvcreate -L 9G -n swap_1 ubuntu-vg
lvcreate -l 100%FREE -n root ubuntu-vg
```

### Look for swap and modify as needed:

   - `pvcreate /dev/mapper/ENCRYPTED_DATA`: Initializes the encrypted partition as a physical volume for LVM.

   - `vgcreate ubuntu-vg /dev/mapper/ENCRYPTED_DATA`: Creates a volume group named "ubuntu-vg" using the encrypted physical volume.
   
   - `lvcreate -L 9G -n swap_1 ubuntu-vg`: Creates a logical volume named "swap_1" with a size of 9GB within the "ubuntu-vg" volume group.
   
   - `lvcreate -l 100%FREE -n root ubuntu-vg`: Creates a logical volume named "root" using the remaining space within the "ubuntu-vg" volume group.


## Configuring Grub and Crypttab

- Your computer will not be able to access the data on your OS without us telling it how to access it.

> Run the following commands immediately after Ubuntu Installation begins -- when installer is past 1% but before 66%
{: .prompt-info }

```bash
while [ ! -d /target/etc/default/grub.d ]; do sleep 1; done; echo "GRUB_ENABLE_CRYPTODISK=y" > /target/etc/default/grub.d/local.cfg
echo "ENCRYPTED_DATA UUID=$(blkid -s UUID -o value /dev/sda4) none luks,discard" > /target/etc/crypttab
```

- Again, these commands need to be run after the installer begins. This will allow us to modify the temporary install files for grub, before they're moved over to the perminant system.

> The script waits for the existence of the `/target/etc/default/grub.d` directory before proceeding.
  
   - `echo "GRUB_ENABLE_CRYPTODISK=y" > /target/etc/default/grub.d/local.cfg`: Adds a configuration to enable encryption support in the GRUB bootloader.
   
   - `echo "ENCRYPTED_DATA UUID=$(blkid -s UUID -o value /dev/sda4) none luks,discard" >> /target/etc/crypttab`: Adds an entry to the `/etc/crypttab` file to specify the encrypted partition and its UUID.


## Finished with encrypting OS partition and installing OS

- Let's review

The script sets up encryption, partitions, and LVM to prepare the system for installing Ubuntu with encryption enabled. After running this script and performing the Ubuntu installation, you should have an encrypted Ubuntu installation with specific partitioning and configuration settings.


## Concerns. 

If you're concerned, feel free to backup your drive before you do anything:

```bash
#on pc to copy
dd bs=16M if=/dev/sda|bzip2 -c|nc serverB.example.net 19000

#on remote storage
nc -l 19000|bzip2 -d|dd bs=16M status=progress | gzip -c > ~/backup_dd_disk.img.gz

#To restore the image back to the disk, the following command can be used:
gunzip -c /storage/images/sabrent_disk.img.gz | dd of=/dev/sda
```



* * * 

# BONUS POINTS

Why use a password at all? Why not use ... a logo as a password? Easy to find, never easy to guess.

``` 
mkdir /target/etc/luks

cd /target/etc/luks

apt install wget

wget -P /target/etc/luks/Wikimedia-logo.svg.keyfile https://upload.wikimedia.org/wikipedia/commons/8/81/Wikimedia-logo.svg

 
chmod u=rx,go-rwx /target/etc/luks

chmod u=r,go-rwx /target/etc/luks/Wikimedia-logo.svg.keyfile
```

You could have the file on a flashdrive. Like a key to a car, it's required to turn on. 


Find the UUID:  `blkid -t TYPE=vfat -sUUID -ovalue`

* * *

Add the Keyfile to LUKS encryption:

```bash
cryptsetup luksAddKey /dev/sda4 $UUID_device_and_wikimedia-logo.svg.keyfile
```


To open LUKS with the keyfile: 

```bash
cryptsetup luksOpen /dev/sda4 ENCRYPTED_DATA --key-file $UUID_device_and_wikimedia-logo.svg.keyfile
```

For some reason, if your key file destroyed or corrupted, just use the passphrase.


## Keep LUKS mounting a keyfile at boot time

Append the following line to the `/etc/crypttab` file if you want to use the keyfile:

```bash
ENCRYPTED_DATA /dev/sda $UUID_device_and_wikimedia-logo.svg.keyfile luks
```

* * * 

# Fancy boot screen

Also, your bootloader is not encrypted, so you can use Plymouth to make your bootscreen extra special:

[https://github.com/MarcusHoltz/Animated-Boot-Screen-Creator-for-Linux/](https://github.com/MarcusHoltz/Animated-Boot-Screen-Creator-for-Linux/)

![Plymouth-Animated-Boot-Screen](https://raw.githubusercontent.com/MarcusHoltz/Animated-Boot-Screen-Creator-for-Linux/main/images-for-repo/animated-laptop-bootloader.gif)

