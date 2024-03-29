---
title: Encrypt your cloud storage with gocryptfs
date: 2023-08-07 11:33:00 -0700
categories: [Linux, storage]
tags: [security, cryptography, passwords, storage, cloud]
pin: false
image:
  path: /assets/img/header/header--gocryptfs--encrypt-files-transparently-fuse.jpg
  alt: Use gocryptfs to transparently encrypt files
---
# Using gocryptfs to store notes

![gocrypt-my-notes-on-taking-notes](/assets/img/posts/gocrypt-my-notes-on-taking-notes.jpg){: #cool-image }


## Would you like to upload files to anywhere and trust their contents will be safe? 

* * * 

- gocryptfs works on Linux, Windows, and kind-of on MacOS

* * *


# gocryptfs

![goscryptfs-example-gui](https://github.com/rfjakob/gocryptfs/raw/master/Documentation/folders-side-by-side.gif){: #gocryptfs-example-gui }

## Gocryptfs transparently encrypts files, using an arbitrary directory as storage for the encrypted files.

> Two directories are involved in mounting Gocryptfs filesystem: the source directory, and the mountpoint. 

- Each file in the mountpoint has a specific file in the source directory that corresponds to it, filenames are encrypted. The file in the mountpoint provides the unencrypted view of the one in the source directory.  

- Files are encrypted using a volume key, which is stored either within or outside the encrypted source directory. **A password is used to decrypt this key.**


> Files *CANNOT* be blank to be encrypted and stored with gocrypt. They must have some data in them.
{: .prompt-info }


* * * 

## gocryptfs installation:

- Debian, Ubuntu: `apt install gocryptfs fuse`

- Windows: [Download the windows only client cppcryptfs](https://github.com/bailey27/cppcryptfs) 

If you really need a GUI in Linux, you can use [SiriKali](https://mhogomchungu.github.io/sirikali/)

* * *


## Encrypting a directory with gocryptfs:


### First, you must create the directory to hold the encrypted contents of gocryptfs.

- Linux: `mkdir ~/Documents/.encrypted-with-gocryptfs`

- Windows: `mkdir %userprofile%\Documents\.encrypted-with-gocryptfs`

* * * 

## Next, use gocryptfs to set our encryption settings and password

### Windows 

When trying to use `cppcryptfs.exe` on Windows, you need to be sure to set values under the 'create' tab.



![cppcryptfs-create-settings](/assets/img/posts/gocrypt-windows-cppcryptfs-create.png){: #cppcryptfs-create-settings }



* * *

Using the command line:

```
cppcryptfsctl.exe --init=%userprofile%\Documents\.encrypted-with-gocryptfs --printmasterkey=%userprofile%\Documents\gocryptfs-masterkey.txt.save.bak --volumename=Gocryptfs --longnamemax 63 --deterministicnames
```

Should produce:

```
Choose a password for protecting your files.
Password:
Repeat:
The gocryptfs filesystem has been created successfully.
```

*Explanation of command:*

- `init` This will create the .conf file specific to your settings. 

- `longnamemax` When 63 for max length in file names is chosen, file names created will be up to 68 characters long.

  * This option is useful when using cloud services that have problems with filenames that are above a certain length.

- `deterministicnames` When the encryption is run in deterministic mode, you can use a utility like rsync to back up the encrypted files, and it will copy only the files that have changed. Also, if your backup utility supports delta-syncing (as rsync does) when working with the unencrypted data, then it will also do delta-syncing with the encrypted data. But this leaks information about identical file names across directories.

- `printmasterkey` Print the unencrypted master key in human-readable form. For example, you could print it and save it in a locked drawer.

  * Recovering from a lost password is possible only if you have printed and saved the unencrypted master key.

  * All changing the password does is change the password used to mount the filesystem. 

  * It does not change the encryption key used to encrypt the data. 

  * This is because the key that is used to encrypt the data is encrypted using a key derived from the password and stored in the config file. 

  * So all the password is used for is to unencrypt the actual encryption key.


* * *

### Linux

* * * 

Using the command line:

```
gocryptfs -init ~/Uploads/ -deterministic-names -longnamemax 63
```

Should produce: 

```
Choose a password for protecting your files.
Password:
Repeat:

Your master key is:

    ff468e11-4b4ae937-c419452a-697291b6-
    380159d6-4bd39756-a5344a7a-92e430ba

If the gocryptfs.conf file becomes corrupted or you ever forget your password,
there is only one hope for recovery: The master key. Print it to a piece of
paper and store it in a drawer. This message is only printed once.
The gocryptfs filesystem has been created successfully.
You can now mount it using: gocryptfs Uploads MOUNTPOINT
```

*Explanation:*

- The master key is printed on the screen. Be sure to copy it somewhere safe. 




* * * 

## Mounting a gocryptfs to an unencrypted path

### There are many options for mounting, but usually, you don't need any. Defaults are fine.



* * * 

### Linux

* * *

```
gocryptfs ~/Documents/.encrypted-with-gocryptfs ~/Documents/UnencryptedMountPoint
```




### Windows 

* * *

```
cppcryptfsctl.exe %userprofile%\Documents\.encrypted-with-gocryptfs %userprofile%\Documents\UnencryptedMountPoint
```




## Auto-mounting on Windows

![gocrypt-automount-windows](/assets/img/posts/gocrypt-automount-windows.jpg){: #gocrypt-automount-windows }


> You **CANNOT** have anything **already mounted** to `R:\` or `Z:\` or you will see *weird issues*.
{: .prompt-danger }

Best, if you can, to just move any drives on those letters before mounting with cppcryptfs.


## Auto-mounting on Linux

Reference: https://wiki.psuter.ch/doku.php?id=encrypted_backups_to_the_cloud

```
/tmp/fstab/a /tmp/fstab/b fuse./usr/local/bin/gocryptfs nodev,nosuid,allow_other,quiet,passfile=/tmp/fstab/pass 0 0

/tmp/cipher /tmp/plain fuse./usr/local/bin/gocryptfs nofail,allow_other,passfile=/tmp/password 0 0
```


## You may need a custom location for the gocryptfs.conf

`gocryptfs.conf` should NOT be backed up with your files, unless you have a **really strong password**.

But it's totally OK to backup `gocryptfs.diriv`


* * * 

## Sync files with Nextcloud

### Windows

If you're using Nextcloud on Windows to sync your files. There are a few changes to make it more ... normal.

- Uncheck: 
  * Nextcloud client > Settings > General > Advanced > "Show sync folders in Explorer's Navigation Pane"

- Keep all files local

![nextcloud-sync-keep-local](/assets/img/posts/nextcloud-sync-keep-local-always.png){: #nextcloud-keep-sync-local }


