---
title: GPG Key Creation, Modification, and Storage
date: 2023-04-22 11:33:00 -0700
categories: [Linux]
tags: [administration, encryption, GPG, cryptography]
pin: false
image:
  path: /assets/img/header/header--gpg--encryption-key-creation-storage-and-use.jpg
  alt: GPG Key Creation and Modification and Storage now with YubiKey
---

# GPG Key Creation, Modification, and Storage

## Install GUI Software

Install software:

`sudo apt -y install kgpg gpa seahorse kleopatra`

Install additional software: 

`sudo apt install faketime xloadimage paperkey dos2unix`


### Check to see how much randomness your computer can create:

`cat /proc/sys/kernel/random/entropy_avail`




## Initialize the gpg system

`gpg -k` This command initializes gpg and creates directory /home/user/.gnupg, it normally displays your entire keyring.

It also creates the first `keybox` in /home/user/.gnupg/pubring.kbx

/home/user/.gnupg/trusted.gpg gpg's trustedb is created as well.

`cd ~/.gnupg`

`wget https://raw.githubusercontent.com/drduh/config/master/gpg.conf`




* * *
# **DISABLE NETWORKING FOR THE REMAINDER OF KEY CREATION**
* * *


## Create the Master key.

The first key we will generate is the master key. 

The master key will be used for **certification** only: to issue sub-keys that are used for encryption, signing and authentication.  [C]

BONUS: If you want to be cool and pick a time to be the EXACT time you created the key/subkeys, then you can use faketime. Keep in mind, faketime uses the local timezone for the creation of the file. If you switch to GMT to create the gpg file.... that would be wise. 

`sudo apt install faketime`

`faketime -f '2021-12-12 12:12:21' gpg --expert --full-generate-key`

`faketime -f '2022-02-22 14:22:22' gpg --expert --full-generate-key`

`gpg --expert --full-generate-key`

We want super duper expert level RSA. Selection #`8`. 

Toggle `Certify` only for this RSA key. Each E,S,A button press sets on/off. 

Set the keysize to `4096`

Valid `0` for infiinity. 

Enter personal information. 

`gpg --list-key`

`gpg --fingerprint --fingerprint`

`gpg --list-sigs`


Take the 'Key fingerprint' hash, located under the creation date. All of it. And remove the spaces. 

Then export this to a quick and easy to use variable: 

`export KEYID=0xFF3E7D88647EBCDB`

And File:

`echo 0xFF3E7D88647EBCDB >keyid`

And the email address to be used with the certificate:

`export EMAILADDRESSUSED=email.address@gmail.com`

`echo "email.address@gmail.com" >emailaddressused`


## Adding a Picture

You might want to add a picture of yourself for completeness. Be sure to have the **metadata stripped off** the photo.... Since the picture is stored in your public key and your public key gets distributed in a lot of places, including sometimes email, it’s best to use a small image to save space.

The image will, on some systems, crap out if you're unable to preview it. To view a preview you need to isntall xloadimage....

`sudo apt install xloadimage`

Use the `gpg --edit-key $KEYID` command. 

At the `gpg>` prompt, enter the command `addphoto` and give GPG the path of the picture you’d like to use: 

`/home/user/Pictures/avatars/photofme.jpg` 

When you’re done, use *save* at the final `gpg>` prompt to save your changes:


If you want to view that picture you will need to install `xloadimage`

`sudo apt install xloadimage`

Now you can view images inside of keys with:

`gpg --list-options show-photos --list-keys email.address@gmail.com`

OR

`gpg --list-options show-photos --fingerprint 0xdc6dc026`

If gpg doesn’t pick the right photo viewer, you can override it with `--photo-viewer 'eog %I'` or similar.



## Start Adding Sub-Keys

Edit the master key to add sub-keys:

`faketime -f '2021-12-12 12:12:21' gpg --expert --edit-key $KEYID`

or

`gpg --expert --edit-key $(cat keyid)`


You will recieve a new prompt, `gpg>`

Let's add our first key with, `addkey`

And we dont have to select `8` just like above, the menu provides a shortcut to the individual key...



## Signing Key 

Select: 

`(4) RSA (sign only)`

Requested keysize `4096` bits


Hit 'Yes'

Once finished you will recieve a new prompt, `gpg>`

Let's add our second key with, `addkey`



## Encryption Key

Select:

`(6) RSA (encrypt only)`

`4096` bits long


Hit 'Yes'

Once finished you will recieve a new prompt, `gpg>`

Let's add our third key with, `addkey`



##### Authentication Key

GPG doesn't provide an authenticate-only key type, so select 

`(8) RSA (set your own capabilities)` and toggle [E,S,A] the required capabilities until the only allowed action is Authenticate.

`gpg> save`



## Master Key OMG Revocation Certificate

Now we generate a revocation certificate file. If your *master* keypair gets lost or stolen, this certificate file is the only way you’ll be able to tell people to ignore the stolen key. This is important, don’t skip this step!

`gpg --output user_$KEYID.gpg-revocation-certificate --gen-revoke my.email@domain.com`


Store the revocation certificate file in a different place than your master keypair (which we’ll export in a later step). You’ll use it to revoke your *master* keypair should you lose access to it. 

**If you only lose access to your laptop keypair, then you’ll revoke those subkeys using the master keypair, not this revocation certificate.**

Import the certificate into your keyring and that will immediately revoke your master key pair.




## Storage for Masterkey/Subkeys

### (Optional) Create Temp Filesystem in RAM

We first create a RAM-based ramfs temporary folder to prevent our keys from being written to permanent storage. we use **ramfs** instead of **tmpfs**/ or /dev/shm/ because ramfs doesn’t write to swap.


```
mkdir /tmp/gpg
sudo mount -t ramfs -o size=1M ramfs /tmp/gpg
sudo chown $(logname):$(logname) /tmp/gpg
gpg --export-secret-subkeys email.address@gmail.com > /tmp/gpg/subkeys
or
gpg --export-secret-subkeys $KEYID > /tmp/gpg/subkeys
```




* * *

SCRIPT I MADE TO TAKE CARE OF IT FOR YOU INCLUDED AT THE BOTTOM OF THIS PAGE

* * *



### Just Make a Directory and Save to That

If you're on a LIVE ISO or other, you can just make a directory as this will all be wiped when the machine looses power...

`mkdir ~/export`



### Find the key fingerprint of each subkey

`gpg --fingerprint --fingerprint`

Now with the keys fingerprint shown we can need to export each individually:


Export is a quick and easy way to reference information: 

`export AUTHKEYID=<LAST 8 DIGITS ON FINGERPRINT>`

And File:

`echo <LAST 8 DIGITS ON FINGERPRINT> >authkeyid`

... do so for each, SIGNINGKEYID, ENCRYPTKEYID



#### -=-=-=- For Reference, Here are mine... -=-=-=- 

```
export EMAILADDRESSUSED=email.address@gmail.com

Main Certify Key: 
7CA20804DF76C8EC194DB1DD153A63F9C3CF70C2
export KEYID=7CA20804DF76C8EC194DB1DD153A63F9C3CF70C2

Signing Key: 
CC4FFBF723C14A410947057D04F61F77ECA8ED49
export SIGNINGKEYID=CC4FFBF723C14A410947057D04F61F77ECA8ED49

Encryption Key : 
EB8609F0B8B6F6100AD1413D9C9884F49842F853
export ENCRYPTKEYID=EB8609F0B8B6F6100AD1413D9C9884F49842F853

Authentication Key: 
18563E061313ACE6DC038C98946D1E3EDBA14FF4
export AUTHKEYID=18563E061313ACE6DC038C98946D1E3EDBA14FF4
```


* * *

The best way to name the files is `<email_in_key>_$KEYID.publicprivate-description-mainkeysubkey.extension.extension`

EXAMPLE:

`email.address@gmail.com_4BDF19B845027EF71CCC871109C4D4913F9A3F3E.public-master_and_subkeys.txt.asc`

.txt.asc is for ASCII armored

.pgp.key is for Binary key

* * *



## What are Subkeys?

OpenPGP supports subkeys, which are like the normal keys, except they're bound to a primary key pair. A subkey can be used for signing or for encryption. The really useful part of subkeys is that they can be revoked independently of the primary keys, and also stored separately from them. 


* * *

## Difference Between `export-secret-keys` and `export-secret-subkeys`

GPG allows you to export your subkeys with with a blank primary key, thereby rendering the secret part of the primary key useless;  using the `gpg --export-secret-subkeys {key-id}` option. 

Its intended use is in generating a full key, say stored on a USB drive, with an additional signing subkey on a dedicated machine, like your laptop. This is a GNU extension to OpenPGP and other implementations & no software can not be expected to successfully import such a key.

It will export dummy packets for the primary key, so effectly only sub keys are exported. This is great if you want to keep your primary key somewhere else, while still having both a signing and encryption subkey.

Verify that `gpg -K` shows a `sec#` instead of just `sec` for your private key. That means the secret key is not really there. See also the presence of a dummy OpenPGP packet in the output of `gpg --export-secret-keys YOURPRIMARYKEYID | gpg --list-packets` 

* * *



## When will I need my primary key again?

- when you sign someone else's key or revoke an existing signature

- when you add a new UID or mark an existing UID as primary (like adding a picture)

- when you create a new subkey

- when you revoke an existing UID or subkey (remove a picture)

- when you change the preferences (e.g., with setpref) on a UID

- when you change the expiration date on your primary key or any of its subkey

- when you revoke or generate a revocation certificate for the complete key

(Because each of these operation is done by adding a new self- or revocation signatures from the private primary key.) 



## How do I use my primary key again?

When you need to use the primary keys, mount the encrypted USB drive.

List keys but use a different home directory (encrypted USB drive( for one command only: 

`gpg --homedir /media/something/.gnupg-alternate --list-keys`

OR

Set the `GNUPGHOME` environment variable, that will set different home directory for the entire session:

`export GNUPGHOME=/media/something`

`gpg -K`

The command should now list your private key with sec and not sec#. 




# Now we can export each of the keys

## AuthKey

---------------

Export the Secret Key used to auth messages, in Binary: 

`gpg --export-secret-keys $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYID.private-auth-mainkey.pgp.key`

Export the Secret Key used to auth messages, in ASCII: 

`gpg --export-secret-keys -a $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYID.private-auth-mainkey.txt.asc`

Export the Subkey's secret key used to auth, in ASCII: 

`gpg --export-secret-subkeys -a $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYID.private-auth-subkey.txt.asc`

And again, Binary, not in ASCII:

`gpg --export-secret-subkeys $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYID.private-auth-subkey.pgp.key`

AuthKey Public Key:

`gpg --export -a $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYID.public-auth-subkey.txt.asc`

And public again, not in ASCII:

`gpg --export $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYID.public-auth-subkey.pgp.key`




## SigningKey

-----------------

Export the Secret Key used to sign messages, in Binary: 

`gpg --export-secret-keys $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYID.private-signing-mainkey.pgp.key`

Export the Secret Key used to sign messages, in ASCII: 

`gpg --export-secret-keys -a $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYID.private-signing-mainkey.txt.asc`

Export the Subkey's secret key used to signing, in ASCII: 

`gpg --export-secret-subkeys -a $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYID.private-signing-subkey.txt.asc`

And again, not in ASCII:

`gpg --export-secret-subkeys $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYID.private-signing-subkey.pgp.key`

SigningKey Public Key: 

`gpg --export -a $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYID.public-signing-subkey.txt.asc`

And public again, not in ASCII:

`gpg --export $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYID.public-signing-subkey.pgp.key`





## EncryptKey

------------------

Export the Secret Key used to encrypt messages, in Binary: 

`gpg --export-secret-keys $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYID.private-signing-mainkey.pgp.key`

Export the Secret Key used to encrypt messages, in ASCII: 

`gpg --export-secret-keys -a $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYID.private-signing-mainkey.txt.asc`

Export the Subkey's secret key used to encrypting, in ASCII: 

`gpg --export-secret-subkeys -a $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYID.private-encryption-subkey.txt.asc`

And again, not in ASCII:

`gpg --export-secret-subkeys $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYID.private-encryption-subkey.pgp.key`

EncryptKey Public Key:

`gpg --export -a $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYID.public-encryption-subkey.txt.asc`

And public again, not in ASCII:

`gpg --export $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYID.public-encryption-subkey.pgp.key`






## Main Certificate Key & MainKey_and_SubKeys

------------------------------------------------------------------------------------

### Export the Master Certificate key, with its subkeys as well

This *file* will be the same as if you were using Kleopatra and hit 'export secret key' -- it would look exactly as the ascii key below:

`gpg --export-secret-subkeys -a $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-master_and_subkeys-subkey.txt.asc`

And again, not in ASCII:

`gpg --export-secret-subkeys $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-master_and_subkeys-subkey.pgp.key`

And again, in super owner format:

`gpg --export-ownertrust > ~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-ownertrust-master-key`





### Export the Main Certificate Signing Key's Master Private Key (the OMG-Keep-It-Safe-Key)

Export the Secret Key used to DO ALL THE THINGS, in Binary: 

`gpg --export-secret-keys $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-master_and_subkeys-mainkey.pgp.key`

Export the Secret Key used to DO ALL THE THINGS, in ASCII: 

`gpg --export-secret-keys -a $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-master_and_subkeys-mainkey.txt.asc`






### Master Key Public Key is last export

This is the public master key that gets transported around the internet:

`gpg --export -a $KEYID > ~/export/$(echo $EMAILADDRESSUSED)_$KEYID.public-master_and_subkeys.txt.asc`

And again, not in ASCII:

`gpg --export $KEYID > ~/export/$(echo $EMAILADDRESSUSED)_$KEYID.public-master_and_subkeys.pgp.key`








## Store it on an encrypted USB in a safe with a paper copy

----------------------------------------------------------------

### PAPERKEY

`paperkey --secret-key ~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-master_and_subkeys-mainkey.pgp.key --output ~/export/$(echo $EMAILADDRESSUSED)_$KEYID.private-master_and_subkeys-paper-key.txt`

Then you can print out this KEY (on the right is the checksum). Wrap the USB Thumb drive for storage in this document, and place a second copy with your regular documents. With some quick OCR you can recover your key. 








## Restore The GPG Key

`cp /path/to/backups/*.gpg ~/.gnupg/` 

or, if you exported the ownertrust

`gpg --import-ownertrust ownertrust-gpg.txt`



If you just copy-pasted the .gnupg folder, you should register keys:

`gpg --import pubring.gpg`

`gpg --import secring.gpg`








## OMG I GOT HACKED or LOST MY KEY

IN CASE OF EMERGENCY

Should the worst happen and your laptop with your special keypair gets lost or stolen (or your special keypair is otherwise compromised), we need to revoke the subkeys on that keypair. 


1. Unlock your safe-deposit box and get your master keypair out.

2. Boot a live USB distro of choice. 

3. Import your master keypair into the live USB’s keyring:

`gpg --import /path/to/user_$KEYID.public-master_and_subkeys.gpg-key /path/to/$(echo $EMAILADDRESSUSED)_$KEYID.private-master-key.pgp.key`

4. Now find your gpg key and use `gpg --edit-key` to interactively revoke your subkeys:

`gpg --fingerprint --fingerprint`

Then export this to a quick and easy to use variable: 

`export KEYID=0xFF3E7D88647EBCDB`

`gpg --edit-key $KEYID`

Or just use the email address attached to the key:

`gpg --edit-key bilbo@shire.org`

Select all the Keys you would like to revoke, in this example -- we'll revoke our subkeys and not our signing certificate.


```
Secret key is available.

pub  4096R/488BA441  created: 2013-03-13  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  4096R/69B0EA85  created: 2013-03-13  expires: never       usage: E
sub  4096R/C24C2CDA  created: 2013-03-13  expires: never       usage: S
[ultimate] (1). Bilbo Baggins <bilbo@shire.org>
[ultimate] (2)  [jpeg image of size 5324]

gpg> key 1

pub  4096R/488BA441  created: 2013-03-13  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub* 4096R/69B0EA85  created: 2013-03-13  expires: never       usage: E
sub  4096R/C24C2CDA  created: 2013-03-13  expires: never       usage: S
[ultimate] (1). Bilbo Baggins <bilbo@shire.org>
[ultimate] (2)  [jpeg image of size 5324]

gpg> key 2

pub  4096R/488BA441  created: 2013-03-13  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub* 4096R/69B0EA85  created: 2013-03-13  expires: never       usage: E
sub* 4096R/C24C2CDA  created: 2013-03-13  expires: never       usage: S
[ultimate] (1). Bilbo Baggins <bilbo@shire.org>
[ultimate] (2)  [jpeg image of size 5324]

gpg> revkey

Do you really want to revoke the selected subkeys? (y/N) 
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
Your decision? 1
Enter an optional description; end it with an empty line:
>
Reason for revocation: Key has been compromised
(No description given)
Is this okay? (y/N) y

You need a passphrase to unlock the secret key for
user: "Bilbo Baggins <bilbo@shire.org>"
4096-bit RSA key, ID 488BA441, created 2013-03-13


You need a passphrase to unlock the secret key for
user: "Bilbo Baggins <bilbo@shire.org>"
4096-bit RSA key, ID 488BA441, created 2013-03-13


pub  4096R/488BA441  created: 2013-03-13  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
This key was revoked on 2013-03-13 by RSA key 488BA441 Bilbo Baggins <bilbo@shire.org>
sub  4096R/69B0EA85  created: 2013-03-13  expires: never       usage: E
This key was revoked on 2013-03-13 by RSA key 488BA441 Bilbo Baggins <bilbo@shire.org>
sub  4096R/C24C2CDA  created: 2013-03-13  expires: never       usage: S
[ultimate] (1). Bilbo Baggins <bilbo@shire.org>
[ultimate] (2)  [jpeg image of size 5324]

gpg> save

```


5. Now that your subkey has been revoked, you have to tell the world about it by distributing your key to a keyserver.

Source:  [Bilbo Baggins GPG Tutorial](https://alexcabal.com/creating-the-perfect-gpg-keypair)

* * *




# KEYSERVERS


## Keybase

Setting up a keybase username lets people easy mode search for you and encrypt things to you. Your entire account is supposed to tie in your social media accounts, thus, even further proving who you are and connecting your digital data trail for threat actors further. 


### Install and Login to Keybase

You must install keybase from their own repository: [Keybase Install Tutorial Here](https://keybase.io/docs/the_app/install_linux)


```
curl --remote-name https://prerelease.keybase.io/keybase_amd64.deb
sudo apt install ./keybase_amd64.deb
run_keybase
```

Once Keybase is installed:

`keybase login`

To import your existing private key:

`keybase pgp select`




### Using Keybase

The point of Keybase is to help you verify the person you want to communicate with is who they say they are. So Keybase let’s users prove who they are by authenticating with their social accounts. For example, search: 

`keybase search sthulb`

Then this will display the following output:

`sthulb twitter:sthulb github:sthulb dns://thulbourn.com`

Now that I have verified this person, if I wanted to encrypt a file for my friend, then I would run:

`keybase encrypt -i secrets.txt -o secrets.txt.asc sthulb`


If you want to encrypt a file for someone who doesn’t use Keybase (e.g. they use their own local GPG installation), then you can export your public/private key from Keybase using the command line tool and then import them into your local GPG so you can utilise GPG to encrypt your data and specify the user’s public key:


```
keybase pgp export -o keybase.public.key
keybase pgp export -s -o keybase.private.key
gpg --import keybase.public.key
gpg --allow-secret-key-import --import keybase.private.key
Notice the use of -s to export the private key
```

Now you can encrypt data via GPG using your Keybase private key. 




### Staying Up To Date

If you’ve pulled keys from a public server, then you should regularly check those keys are still valid and haven’t been compromised. 

`gpg --refresh-keys`





### Other Keyservers

#### Selecting a keyserver and configuring your machine to refresh your keyring.

-----------------------------------------------------------------------------------------------------

These are some keyservers that are often used for looking up keys with `gpg --recv-keys`

These can be queried via https:// (HTTPS) or hkps:// (HKP over TLS) respectively.

```
keybase.io 
keys.openpgp.org
pgp.mit.edu
keyring.debian.org
keyserver.ubuntu.com
attester.flowcrypt.com
zimmermann.mayfirst.org
keys.openpgp.org
keyserver.ubuntu.com
keys.gnupg.net
pgp.mit.edu
keyoxide.org
[Ubuntu Keyserver](https://keyserver.ubuntu.com/): federated, no verification, keys cannot be deleted.
[Mailvelope Keyserver](https://keys.mailvelope.com/): central, verification of email IDs, keys can be deleted.
[keys.openpgp.org](https://keys.openpgp.org/): central, verification of email IDs, keys can be deleted, no third-party signatures (i.e. no Web of Trust support).
```




#### Use the sks keyserver pool, instead of one specific server, with secure connections.

--------------------------------------------------------------------------------------------------------------

Most OpenPGP clients come configured with a single, specific keyserver. This is not ideal because if the keyserver fails, or even worse, if it appears to work but is not functioning properly, you may not receive critical key updates. 

Not only is this a single point of failure, it is also a prime source of leaks of relationship information between OpenPGP users, and thus an attack target.



Therefore, we recommend using the sks keyservers pool. The machines in this pool have regular health checks to ensure that they are functioning properly. If a server is not working well, it will be removed automatically from the pool.


You should also ensure that you are communicating with the keyserver pool over an encrypted channel, using a protocol called hkps. 

Then, to use this keyserver pool, you will need to [download the sks-keyservers.net CA](https://sks-keyservers.net/sks-keyservers.netCA.pem), and save it somewhere on your machine. Please remember the path that you save the file to! Next, you should [verify the certificate’s finger print](https://sks-keyservers.net/verify_tls.php).

* * *

Now, you will need to use the following parameters in `~/.gnupg/gpg.conf`, and specify the full path where you saved the .pem file above:


```
keyserver hkps://hkps.pool.sks-keyservers.net
keyserver-options ca-cert-file=/path/to/CA/sks-keyservers.netCA.pem
```

* * *

Now your interactions with the keyserver will be encrypted via `hkps`, which will obscure your social relationship map from anyone who may be snooping on your traffic. For example, if you do a `gpg --refresh-keys` on a keyserver that is hkp only, then someone snooping your traffic will see **every single key you have in your key ring** as you request any updates to them.

Note: hkps://keys.indymedia.org, hkps://keys.mayfirst.org and hkps://keys.riseup.net all offer this (although it is recommended that you use a pool instead).




#### Ensure that all keys are refreshed through the keyserver you have selected.

----------------------------------------------------------------------------------------------------

When creating a key, individuals may designate a specific keyserver to use to pull their keys from. It is recommended that you use the following option to `~/.gnupg/gpg.conf`, which will ignore such designations:

`keyserver-options no-honor-keyserver-url`

This is useful because (1) it prevents someone from designating an insecure method for pulling their key and (2) if the server designated uses hkps, the refresh will fail because the ca-cert will not match, so the keys will never be refreshed. Note also that an attacker could designate a keyserver that they control to monitor when or from where you refresh their key.




#### Refresh your keys slowly and one at a time.

----------------------------------------------------------------

Now that you have configured a good keyserver, you need to make sure that you are regularly refreshing your keys. The best way to do this on Debian and Ubuntu is to use parcimonie:

`sudo apt-get install parcimonie`

Parcimonie is a daemon that slowly refreshes your keyring from a keyserver over `Tor` -. It uses a randomized sleep, and fresh Tor circuits for each key. The purpose is to make it hard for an attacker to correlate the key updates with your keyring.

**You should not use gpg --refresh-keys or the refresh keys menu item on your email client** because you disclose to anyone listening, and the keyserver operator, the whole set of keys that you are interested in refreshing.






## Using your PGP key to encrypt your system boot process

Check out the YouTube tutorial by Sameer Pasha on UEFI secure boot Kernel Digital Signing and Verification.

[UEFI Secure Boot of Linux](https://www.youtube.com/watch?v=SQ7Ajv2Cnzg)




## Check the creation time of a GPG Key

First you need to get the time, then convert it from Unix to Human. 

To list the creation time of the key:

`gpg -a --export "Heinrich Heine" | gpg --list-packets`


The dates in the key certificate dump are Unix epochs. Convert to human-readable with:
`date -d @1281838967`




## How to Properly Use gpg-agent

### Text Stolen From StackOverflow:

Follow: [http://tr.opensuse.org/SDB:Using_gpg-agent](http://tr.opensuse.org/SDB:Using_gpg-agent)

Following that, my gpg-agent daemon is caching my GnuPG passwords properly now. There was nothing wrong with my setup, just that I didn't know how to test whether my GnuPG passwords is caching properly or not.

Now, I do:

`echo "test" | gpg -ase -r 0xMYKEYID | gpg`

From the site: "Replace 0xMYKEYID with your GnuPG key ID. While running this command, the agent should open a graphical password dialog twice: first for signing or encrypting (gpg -ase)(gpg -ase) then for decryption or signature check (| gpg). From now on, every time GnuPG is used (either from the command line or embedded in a graphical program such as KMail), gpg-agent's password will be passed automatically (until the time-out expires or the graphical interface is closed)."



### Default GPG agent configuration

Paste the following text into a terminal window to create a recommended GPG agent configuration:

And to avoid the caching expiration, I now have set extremely long timeout period:

`nano ~/.gnupg/gpg-agent.conf`


```
enable-ssh-support
ttyname $GPG_TTY
pinentry-program /usr/bin/pinentry-x11
max-cache-ttl 60480000
default-cache-ttl 60480000
#pinentry-program /usr/bin/pinentry
#pinentry-program /usr/bin/pinentry-curses
#pinentry-program /usr/bin/pinentry-tty
#pinentry-program /usr/bin/pinentry-gtk-2
#pinentry-program /usr/bin/pinentry-x11
#pinentry-program /usr/local/bin/pinentry-curses
#pinentry-program /usr/local/bin/pinentry-mac
#pinentry-program /opt/homebrew/bin/pinentry-mac
```



### Replace ssh-agent with gpg-agent

gpg-agent provides OpenSSH agent emulation. To launch the agent for use by ssh use the `gpg-connect-agent /bye` or `gpgconf --launch gpg-agent` commands.

Add the following lines to your shell rc file (~/.zshrc or ~/.bashrc):


```
alias gpg=gpg2
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
gpgconf --launch gpg-agent
```


In Debian, SSH_AUTH_SOCK can be found in ${HOME}/.gnupg/S.gpg-agent.ssh, but if you have gnome-keyring activated it will be in $XDG_RUNTIME_DIR/gnupg/S.gpg-agent.ssh. 

And source your file, source will update arguments in the current shell environment:

`source ~/.bashrc`

Then launch your gpg-agent:

`sudo killall gpg-agent`

`gpg-agent --daemon --enable-ssh-support`


To display your ssh key use:
`ssh-add -L`




### Adding PGP keys to the SSH Keyring

Tell `gpg-agent` which subkey to pass to ssh by adding its “keygrip” to `~/.gnupg/sshcontrol`

Run: `gpg -k --with-keygrip`

You will see a list of the keys you have with a new line added: 'Keygrip  =  '


```
pub   rsa2048/93BDD96B 2017-06-29 [SC]
      D03833D3D52F5FFCCC73452461671825E8DEC139
      Keygrip = 8A6CDC5FCE05A5B251BD8C397B269607534B4702
uid         [ultimate] Big John <big.john@gmail.com>
sub   rsa2048/0424163D 2017-06-29 [E]
      Keygrip = E110250E32B811D45879A66F487CE95BC1906D77
sub   rsa2048/8F228EDB 2017-06-29 [A]
      Keygrip = 32BC5688805A640D495E8A7B41EC78F74E77E098
```

Using the Authorization Key's keygrip:

`echo 32BC5688805A640D495E8A7B41EC78F74E77E098 > ~/.gnupg/sshcontrol`

Confirm key has been added:
`ssh-add -l`

Sidenote. What is a 'keygrip'? As explained by Werner:
    That is a protocol neutral way to identify a public key. It is a hash over the actual public key parameters. It is GnuPG specific but for example, pkcs#15 uses a similar technique. To compute it, you should use the respective Libgcrypt function.


* * *

### Export a Regular Joe SSH Public Key 

It is now (since gpg 2.1) possible to simply extract ssh keys directly using gpg. 

For use with GitHub and other git+ssh providers, add this public key to your account’s SSH keys.:

`gpg --export-ssh-key <key id>!`

With the key id being found from.... `rsa2048/0x93BDD96B`

So the command would be `gpg --export-ssh-key 0x93BDD96B! > ~/.ssh/id_rsa_pgp`

* * *




### Add your new ssh key to your servers

#### Reactivating ssh

Copy your ssh key if you haven’t already. You might need to restart your ssh-agent and re-add you ssh keys to connect to your server  -- or, that's a bunch of bullshit and just login:


```
sudo killall gpg-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

You need to add your private key to the agent for it to work.



#### Adding the keys to the server

First connect to your server :

`ssh sammy@server.com`


On your server, first as a security precaution, temporarily allow password authentication. If anything goes wrong, you will still have access to your server.


`sudo nano /etc/ssh/sshd_config`

Change this line to yes:

`PasswordAuthentication yes`



Then add your ssh keys in `~/.ssh/authorized_keys`

`sudo nano ~/.ssh/authorized_keys`



#### Test the connection
Finally activate your gpg-agent to test your connection.


```
sudo killall gpg-agent
gpg-agent --daemon --enable-ssh-support
source ~/.bashrc

```

Connect your security key and test to see its ssh key:

`ssh-add -L`

Test your connection

`ssh sammy@server.com`

If you are asked for your security key PIN, this is a good indication that the correct SSH key is used.

If the connection does not work:


```
sudo killall gpg-agent
gpg-agent --daemon --enable-ssh-support
```


And retest.

*DONT FORGET*

##### Secure your server

Disable password authentication

`sudo nano /etc/ssh/sshd_config`

Set `PasswordAuthentication no`

Optionally you could remove your other ssh keys from `~/.ssh/authorized_keys`

[Great Tutorial on how to add SSH Auth using a PGP Key](https://ryanlue.com/posts/2017-06-29-gpg-for-ssh-auth)




### Placing your secret key on a new machine

When working with secret keys it's generally preferable not to write them to files and, instead, use SSH to copy them directly between machines using only gpg and a pipe:

`gpg --export-secret-key SOMEKEYID | ssh othermachine gpg --import`


If you must, however, output your secret key to a file please make sure it's encrypted. Here's how to accomplish that using AES encryption using the Dark Otter approach:


```
gpg --output public.gpg --export SOMEKEYID && \
gpg --output - --export-secret-key SOMEKEYID |\
    cat public.gpg - |\
    gpg --armor --output keys.asc --symmetric --cipher-algo AES256
```



## YUIBIKEY (Yubico Yubikey)



### Installing YubiKey Tools on Linux
To get the Yubikey software working we need to satisfy dependencies:
`sudo apt -y install wget gnupg2 gnupg-agent dirmngr cryptsetup scdaemon pcscd secure-delete hopenpgp-tools yubikey-personalization libssl-dev swig libpcsclite-dev`

To install and use the `ykman` utility, python crap has to come with it.

`sudo apt -y install python3-pip python3-pyscard`

`pip3 install PyOpenSSL`
`pip3 install yubikey-manager`

`sudo service pcscd start`
`~/.local/bin/ykman openpgp info`


[Source for additional Software](https://support.yubico.com/hc/en-us/articles/360016649039-Installing-Yubico-Software-on-Linux)

`sudo add-apt-repository ppa:yubico/stable && sudo apt-get update`
`sudo apt install yubikey-manager yubikey-personalization-gui libpam-yubico libpam-u2f libfido*`
And then you can download the actual software you need... but not, as it is an appimage it'll install like shit. 
So first, we must install [AppImageLauncher](https://launchpad.net/~appimagelauncher-team/+archive/ubuntu/stable):
`sudo add-apt-repository ppa:appimagelauncher-team/stable`
Then you can: `sudo apt update` and install `sudo apt install appimagelauncher`

With this glorious POS installed, you can download and run the YubiKey appimage, and it will re-arrange it to a better location and add a .desktop icon. 
BUT BUT BUT, you cannot whitelist apps. So when Joplin starts from AppImage, it tries to re-arrange it and break it. So once AppImageLauncher has run the YubiKey AppImage, REMOVE IT. 

`wget https://developers.yubico.com/yubikey-manager-qt/Releases/yubikey-manager-qt-latest-linux.AppImage` 

RUN. Let AppImageLauncher move the Yubikey Manager appimage.

`sudo apt remove appimagelauncher`

Now you're safe from the bad man. 



#### Using YubiKey
`gpg --card-status`
You should see your card listed.

`gpg --edit-key $KEYID`
Now we are back to where we were before. Able to make changes on the gpg> prompt.

Let's move each of the 3 keys we made to the yubicard. 
`key 1`
You should see an asteresk next to one of the keys. Take a look at the usage: and continue
`keytocard`
Destination slot on the Yubikey:
Appropreate slot on the Yubikey that corresponds to the Key transfering.

You will be prompted to enter a PIN. Default pin is 12345678 for admin changes.

Back to the gpg> prompt. Just type  `key 2` and then  `key 1`. You needed to select and then unselect the keys. 

`keytocard`
Destination slot on the Yubikey.

Do this one more time. 

When all 3 are on the Yubikey you can:
`gpg> quit`
Do not SAVE changes: `N`
Quit without saving? `Y`

If you SAVE, you will loose your main private key. Dont do that. 

##### One Final Backup of the gpg files
`cp -auxf ~/.gnupg/ ~/export/`


### Check and Edit YubiKey Settings
`gpg --list-keys`
We should see the information we saved.

Under "General key info..: "
```
sec#    4096R/0xFF...
ssb>    4096R/0xBE...
ssb>  4096R/0x59
```
`sec#` indicates master key is not available (as it should be stored encrypted offline).


`gpg --card-edit`
Here it tells you about the content of the Yubikey. 

To change the Admin Password, it doesnt have to be digits:
`gpg/card> admin`
`gpg/card> passwd`

Now you can quit out of that.






### Using YubiKey to Break your computer

#### SSH with YubiKey (old method)
`nano ~/.gnupg/gpg-agent.conf`
add the following line:
`enable-ssh-support`

Then you optionally can kill then ssh agent. 

Your User ID may vary, this example is 1001
`ls -lah /var/run/user/1001/gnupg/`

With that info, we can edit our `.bashrc`
Add at the bottom:
`export SSH_AUTH_SOCK=/var/run/user/1001/gnupg/S.gpg-agent.ssh`
SAVE. QUIT.

If the agent was quit above you should be able to see it with: 
`ssh-add -L `

If not, possible reboot. 
Also, make sure your PIN works everytime, someone could intercept that socket file we added above. Unless you set Signature PIN forced. 

`gpg/card> admin`
`gpg/card> forcesig`


#### Setup Your Computer to LOVE the Yubikey
`sudo apt install libpam-u2f`

`mkdir -p ~/.config/Yubico`
(-p says make the stuff underneath me too, incase .config did not exist) 

`pamu2fcfg >  ~/.config/Yubico/u2f_keys`
* * *


#### HOW TO MAKE SUDO USELESS UNLESS YOU #1. Have the YubiKey in a USB slot #2. Touch the YubiKey
`sudo nano /etc/pam.d/sudo `

#under @include common-auth   add the following.....
auth    required    pam_u2f.so

SAVE, CLOSE
* * *


#### HOW TO MAKE LOGIN USELESS WITHOUT YUBIKEY.
`sudo nano /etc/pam.d/gdm-password`
OR
`sudo nano /etc/pam.d/lightdm `
GDM is the more modern choice, could be either.

ONCE OPEN.....
#under @include common-auth   add the following.....
auth    required    pam_u2f.so
* * *


#### TTY LOGIN REQUIRES YUBIKEY  (CTRL + ALT + F2)
`sudo nano /etc/pam.d/login`

ONCE THE FILE IS OPEN......  ctrl+w to look for 'common-auth'
#under @include common-auth   add the following.....
auth    required    pam_u2f.so
* * *


#### YubiKey Required for Remote Server SSH Connection (API ACCESS IS REQUIRED FOR THIS SERVICE) 2FA for Remote Server
```
sudo add-apt-repository ppa:yubico/stable
sudo apt update
sudo apt install libpam-yubico
```

`sudo nano /etc/ssh/authorized_yubikeys`
IN THIS NEW DOCUMENT, ADD THE USER_NAME YOU WANT TO HAVE ACCESS :COLON: THEN TOUCH THE YUBIKEY AND ADD THE FIRST 12 CHARACTERS

`sudo nano /etc/pam.d/sshd `
ADD THE NEXT LINE. THE FIRST THING IN THE FILE.
`auth required pam_yubico.so ID=<CLIENT ID> key=<SECRET> authfile=/etc/ssh/authorized_yubikeys`

https://upgrade.yubico.com/getapikey
EMAIL
*touch YubiKey for the OTP*

Should recieve the CLIENT_ID and KEY once those values were entered for on the website above. 

`sudo nano /etc/ssh/sshd_config`
FIND ctrl+w Challenge
ChallengeResponseAuthentication no
TO
ChallengeResponseAuthentication yes

FIND ctrl+w UsePam
UsePam yes

SAVE. EXIT. RESTART SSH SERVICE.

`sudo nano ~/.ssh/authorized_keys`
COMMENT OUT ANY LINES WITH KEYS THAT HAVE BEEN PUT IN HERE.

**WORKS NOW**
Keep in mind... YubiCloud is the validation server

Yubico provides a validation server with free unlimited access, called YubiCloud. YubiCloud knows the factory configuration of all YubiKeys, and is the "default" validation service used by (for example) `yubico-pam`.  


[This method Is better](https://www.youtube.com/watch?v=bQyL0uzrL3k)
[Or this method](https://gist.github.com/artizirk/d09ce3570021b0f65469cb450bee5e29)



### Yubikey: requiring touch to authenticate
By default the Yubikey will perform key operations without requiring a touch from the user. To require a touch for every SSH connection, use the Yubikey Manager (you’ll need the Admin PIN):
```
sudo apt-get install yubikey-manager
ykman openpgp keys set-touch aut on

```
In case you have an error, try removing and reinserting your Yubikey.

To require a touch for the signing and encrypting keys as well:
```
ykman openpgp keys set-touch sig on
ykman openpgp keys set-touch enc on
```

The Yubikey will blink when it’s waiting for the touch.


    Set the retries for PIN, Reset Code and Admin PIN to 10:
    $ ykman openpgp access set-retries 10 10 10



## FIX YUBIKEY
### Program and upload a new Yubico OTP credential
[Link1](https://www.yubico.com/support/download/yubikey-manager/)
[Link 2](https://support.yubico.com/hc/en-us/articles/360013647680-Resetting-the-OTP-Applications-on-the-YubiKey)
[Link 3](https://upload.yubico.com/)
[Link 4](https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/)









### EXTRA RANDOM STUFF FROM FORUMS

Edit `.bashrc` according to the manual:

[die.net/gpg-agent](https://linux.die.net/man/1/gpg-agent)

You can also add:

gpg-agent `--write-env-file "${HOME}/.gpg-agent-info"`

^^^ Above no longer works ^^^

Try This: The agent is compatible, but no longer exports the environment variables that the older version of gpg expects. If you set those manually it will just work:

`export GPG_AGENT_INFO=${HOME}/.gnupg/S.gpg-agent:0:1`

Or

`GPG_AGENT_INFO=/run/user/$(id -u)/gnupg/S.gpg-agent:0:1`



when starting gpg-agent and then add 


```
if [ -f "${HOME}/.gpg-agent-info" ]; then . "${HOME}/.gpg-agent-info"; export GPG_AGENT_INFO fi 
```


to your .bashrc to detect whether the agent is already running. 


This was from Linode:
File: ~/.bash_profile


```
    if [ -f "${HOME}/.gpg-agent-info" ]; then
         source "${HOME}/.gpg-agent-info"
           export GPG_AGENT_INFO
           export SSH_AUTH_SOCK
           export SSH_AGENT_PID
    else
        eval $( gpg-agent --daemon --write-env-file ~/.gpg-agent-info )
    fi
```

File: ~/.gnupg/gpg-agent.conf


```
write-env-file ~/.gpg-agent-info
```

Restart GPG agent:


```
sudo killall gpg-agent
gpg-agent --daemon --write-env-file ~/.gpg-agent-info --enable-ssh-support
source ~/.gpg-agent-info
```








# Script


```
#!/bin/bash

#
# NAME: backup-gpg-keys.sh
# PATH: $HOME/.gnupg/backup-gpg-keys.sh
# DESC: Backup your keys, sub-keys, public and private
# CALL: Called by shell when needed
# DATE: Dec 30, 2021.
#
# DEPENDS:   sudo apt install paperkey gnupg2
#    Helpful if you already have the system variables defined:
#
# export EMAILADDRESSUSED=
# export KEYID=
# export SIGNINGKEYID=
# export ENCRYPTKEYID=
# export AUTHKEYID=
#
#
################################################################################
# clear the screen


clear
echo "Start of GPG KEYS Backup Script"
sleep 1
cd ~

if [ ! -d "$(echo $HOME)/export" ]
then
    echo "~/export does not exist on your filesystem. I will create it for you."
    mkdir ~/export
    sleep 1
fi
echo "~/export directory available, "; sleep 3; 

#echo "check for paperkey install"; sleep 3;
#command -v paperkey >/dev/null 2&1 || $(echo >&2 $(echo "PAPERKEY NOT FOUND!!!  ---   Install Paperkey:    sudo apt-get -y install paperkey"; ) ); exit 0;
#clear;

echo "we can begin to export each of the keys"; 
sleep 2.5;
echo "now CHECKING if EMAIL and all KEY IDs have been SET ...." 
sleep 2;
echo " .... "; sleep .5; echo " ......... "; sleep .25; echo " ......... ... ... "; sleep .15; echo " ......... ... ...  ... ...  ... ... "; sleep .15; echo " ......... . .. ... ...  ...   ...    ... "; sleep 1.5;
clear
echo ; sleep 2; clear

if [ -z ${EMAILADDRESSUSED+x} ]; then echo "EMAILADDRESSUSED is unset. Please Define one now: "; read EMAILADDRESSUSED; export EMAILADDRESSUED=$EMAILADDRESSUSED; echo "Thank you."; echo; echo "Email address set to '$EMAILADDRESSUSED'"; else echo "'$EMAILADDRESSUSED' is being used for email address..."; fi
echo "$EMAILADDRESSUED will be used for the naming of the file."; echo;
sleep 3.2

if [ -z ${KEYID+x} ]; then echo "KEYID is unset. Please Define one now: "; read KEYID; export KEYID=$KEYID; echo "Thank you."; echo; echo "Main key id set to '$KEYID'"; else echo "'$KEYID' is being used for main key id..."; fi
export KEYIDNAME=${KEYID%???????????????????????????????}; echo "$KEYIDNAME will be used for the naming of the file."; echo;
sleep 3.2

if [ -z ${AUTHKEYID+x} ]; then echo "AUTHKEYID is unset. Please Define one now: "; read AUTHKEYID; export AUTHKEYID=$AUTHKEYID; echo "Thank you."; echo; echo "Auth sub-key set to '$AUTHKEYID'"; else echo "'$AUTHKEYID' is being used for authorization sub-key id..."; fi
export AUTHKEYIDNAME=$(echo ${AUTHKEYID%???????????????????????????????}); echo "$AUTHKEYIDNAME will be used for the naming of the file."; echo;
sleep 3.2

if [ -z ${SIGNINGKEYID+x} ]; then echo "SIGNINGKEYID is unset. Please Define one now: "; read SIGNINGKEYID; export SIGNINGKEYID=$SIGNINGKEYID; echo "Thank you."; echo; echo "Signing sub-key set to '$SIGNINGKEYID'"; else echo "'$SIGNINGKEYID' is being used for signing sub-key id..."; fi
export SIGNINGKEYIDNAME=$(echo ${SIGNINGKEYID%???????????????????????????????}); echo "$SIGNINGKEYIDNAME will be used for the naming of the file."; echo;
sleep 3.2

if [ -z ${ENCRYPTKEYID+x} ]; then echo "ENCRYPTKEYID is unset. Please Define one now: "; read ENCRYPTKEYID; export ENCRYPTKEYID=$ENCRYPTKEYID; echo "Thank you."; echo; echo "Encrypt sub-key set to '$ENCRYPTKEYID'"; else echo "'$ENCRYPTKEYID' is being used for encrypt sub-key id..."; fi
export ENCRYPTKEYIDNAME=$(echo ${ENCRYPTKEYID%???????????????????????????????}); echo "$ENCRYPTKEYIDNAME will be used for the naming of the file."; echo;


echo "====================================================="
echo "            LETS MAKE SOME FILES                     "
echo "====================================================="
sleep 2.5
echo 
echo "File creation under ~/export now begining...."
sleep 3.2
clear












echo "AuthKey ";
echo "--------------- "; sleep 1;
echo "Export the Secret Key used to auth messages, in Binary:"; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-mainkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-mainkey.pgp.key; clear;
echo "AuthKey "; echo "--------------- "; sleep .5;
echo "Export the Secret Key used to auth messages, in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-mainkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys -a $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-mainkey.txt.asc; clear; 
echo "AuthKey "; echo "--------------- "; sleep .5;
echo "Export the Subkey's secret key used to auth, in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-mainkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys -a $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-subkey.txt.asc; clear;
echo "AuthKey "; echo "--------------- "; sleep .5;
echo "And again, Binary, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.private-auth-subkey.pgp.key; clear;
echo "AuthKey "; echo "--------------- "; sleep .5;
echo "AuthKey Public Key: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.public-auth-subkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export -a $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.public-auth-subkey.txt.asc; sleep .5; clear;
echo "AuthKey "; echo "--------------- "; sleep .5;
echo "And public again, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.public-auth-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.public-auth-subkey.pgp.key; sleep .5; clear; 
echo "AuthKey "; echo "--------------- "; sleep 1.55;
echo "AuthKey DONE"; sleep 3.5;
clear

echo "SigningKey ";
echo "----------------- "; sleep 1;
echo "Export the Secret Key used to sign messages, in Binary: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-mainkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-mainkey.pgp.key; clear;
echo "SigningKey "; echo "--------------- "; sleep .5;
echo "Export the Secret Key used to sign messages, in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-mainkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys -a $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-mainkey.txt.asc; clear;
echo "SigningKey "; echo "--------------- "; sleep .5;
echo "Export the Subkey's secret key used to signing, in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-subkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys -a $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-subkey.txt.asc; clear;
echo "SigningKey "; echo "--------------- "; sleep .5;
echo "And again, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.private-signing-subkey.pgp.key; clear;
echo "SigningKey "; echo "--------------- "; sleep .5;
echo "SigningKey Public Key: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.public-signing-subkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export -a $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.public-signing-subkey.txt.asc; sleep .5; clear;
echo "SigningKey "; echo "--------------- "; sleep .5;
echo "And public again, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.public-signing-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export $SIGNINGKEYID >~/export/$(echo $EMAILADDRESSUSED)_$SIGNINGKEYIDNAME.public-signing-subkey.pgp.key; sleep .5; clear;
echo "SigningKey "; echo "--------------- "; sleep 1;
echo "SigningKey DONE"; sleep 3.5;
clear
 
echo "EncryptKey ";
echo "------------------ "; sleep 1;
echo "Export the Secret Key used to encrypt messages, in Binary: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-signing-mainkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-encryption-mainkey.pgp.key; clear;
echo "EncryptKey "; echo "--------------- "; sleep .5;
echo "Export the Secret Key used to encrypt messages, in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-signing-mainkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys -a $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-encryption-mainkey.txt.asc; clear;
echo "EncryptKey "; echo "--------------- "; sleep .5;
echo "Export the Subkey's secret key used to encrypting, in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-encryption-subkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys -a $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-encryption-subkey.txt.asc; clear;
echo "EncryptKey "; echo "--------------- "; sleep .5;
echo "And again, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-encryption-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.private-encryption-subkey.pgp.key; clear;
echo "EncryptKey "; echo "--------------- "; sleep .5;
echo "EncryptKey Public Key: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.public-encryption-subkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export -a $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.public-encryption-subkey.txt.asc; sleep .5; clear;
echo "EncryptKey "; echo "--------------- "; sleep .5;
echo "And public again, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.public-encryption-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export $ENCRYPTKEYID >~/export/$(echo $EMAILADDRESSUSED)_$ENCRYPTKEYIDNAME.public-encryption-subkey.pgp.key; sleep .5; clear;
echo "EncryptKey "; echo "--------------- "; sleep 1;
echo "EncryptKey DONE"; sleep 3.5;
clear

echo "SSHKey ";
echo "------------------ "; sleep 1;
echo "Export your Main GPG key's SSH Key: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.public-SSH-mainkey.txt.ssh"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-ssh-key $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.public-SSH-mainkey.txt.ssh; clear;
echo "SSHKey "; echo "--------------- "; sleep .5;
echo "Export your Auth subkey's SSH Key: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.public-SSH-auth-subkey.txt.ssh"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-ssh-key $AUTHKEYID >~/export/$(echo $EMAILADDRESSUSED)_$AUTHKEYIDNAME.public-SSH-auth-subkey.txt.ssh; clear;
echo "SSHKey "; echo "--------------- "; sleep 1;
echo "SSHKey DONE"; sleep 3.5;
clear

echo "Main Certificate Key & MainKey_and_SubKeys ";
echo "------------------------------------------------------- "; sleep 2.5;
echo "Now exporting the Master Certificate key, "; sleep 1; 
echo "with its subkeys as well "; sleep 2; 
echo "  --  :: NOTE :: --  --  :: NOTE :: --  --  :: NOTE :: --  --  :: NOTE :: --  --  :: NOTE :: --  --  :: NOTE :: --"; sleep 2.25;
echo "This *file* will be the same as if you were using Kleopatra and hit 'export secret key' -- "; sleep 2.5;
echo "it will look exactly as the ascii key file exported below "; sleep 1; 
echo; sleep 4; echo "READY?"; sleep 2; echo; sleep 2;
echo "'export secret key' -- now begining"; sleep 2.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master_and_subkeys-subkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys -a $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master_and_subkeys-subkey.txt.asc; clear; 
echo "'export secret key'"; sleep .5;
echo "And again, not in ASCII: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master_and_subkeys-subkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-subkeys $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master_and_subkeys-subkey.pgp.key; clear;
echo "'export secret key'"; sleep .5;
echo "And again, in super owner format: "; sleep 1.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-ownertrust-master-key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-ownertrust >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-ownertrust-master-key; clear;
echo "Main Certificate Key DONE"; sleep 3.5;
clear

echo "##################################################################################################";
echo "#### Export the Main Certificate Signing Key's Master Private Key (the OMG-Keep-It-Safe-Key) ####"; sleep .5;
echo "##################################################################################################"; sleep 4;
echo "   Export the Secret Key used to DO ALL THE THINGS, in Binary: "; sleep 2.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master_and_subkeys-mainkey.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master-mainkey.pgp.key; echo ;
echo "     Export the Secret Key used to DO ALL THE THINGS, in ASCII: "; sleep 2.55; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master_and_subkeys-mainkey.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export-secret-keys -a $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master-mainkey.txt.asc; clear;
echo "Certificate Key's Private Key DONE"; sleep 3.5;
clear

echo "-----------------------------------------------";
echo "---- Master Key Public Key is last export ----"; sleep .5;
echo "-----------------------------------------------"; sleep 3;
echo "This is the public master key that gets transported around the internet: "; sleep 5; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.public-master_and_subkeys.txt.asc"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export -a $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.public-master.txt.asc; echo ;
echo "        public master key"; sleep 5;
echo "        again, not in ASCII: "; sleep 2; echo "[---- FILENAME BELOW ----]"; 
echo "~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.public-master_and_subkeys.pgp.key"; echo "[/---- FILE CREATED ----/]"; sleep 4.25;
gpg --export $KEYID >~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.public-master.pgp.key; clear;
echo "Master Public Key DONE"; sleep 3.5;
clear

echo "--------------------------------------------------------------------- ";
echo ; sleep .5;
echo "    Store it on an encrypted USB in a fire-safe with a paper copy     "; sleep .5;
echo ; sleep .5;
echo "--------------------------------------------------------------------- "; sleep .5;
sleep 7
echo ;
echo "PAPERKEY CREATION"; sleep 1.5;
paperkey --secret-key ~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master-mainkey.pgp.key --output ~/export/$(echo $EMAILADDRESSUSED)_$KEYIDNAME.private-master-paper-key.txt
echo ;
echo "Now, you can print out this KEY (on the right is the checksum)." 
echo "Wrap the USB Thumb drive, for storage, in this document, "
echo "and place a second copy with your regular documents. "
echo "And hey, if you NEED to ... with some quick OCR "
echo "you can recover your key! "
sleep 10
echo ;
echo " DONE DONE DONE DONE DONE "
echo "   DONE DONE DONE DONE DONE "
echo "     DONE DONE DONE DONE DONE "
echo "       DONE DONE DONE DONE DONE   ---- Thanks for playing backup your GPG Keys! "
echo "     DONE DONE DONE DONE DONE "
echo "   DONE DONE DONE DONE DONE "
echo " DONE DONE DONE DONE DONE "
echo
echo "Tip your waitstaff and remember to check ~/export and verify all files created correctly."
exit 0
```



