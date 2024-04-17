---
title: Unique passwords for every service or website
date: 2023-08-14 11:33:00 -0700
categories: [Linux, Security]
tags: [hash, scripts, security, cryptography, passwords]
pin: true
image:
  path: /assets/img/header/header--script--unique-password-hashing.jpg
  alt: Create a new password for every service or website you visit
---
# Let's hash some information into a password

**The creation of different passwords for every service or website you're using can become a pain.**

> Using a repeatable method to generate these passwords, while remaining secure, can help speed up password creation and give you a system to finding lost passwords.
{: .prompt-tip }

> Please keep in mind that hashing is a one way operation. 

- The hash generated is stored within the .passwords folder inside of a passworded zip file (the password used is the one chosen inside the script).

- The original plain text of the hash is stored in a passworded zip.

- At no point is anything unencrypted written to disk.


```
########################################################
#####          What does this script do?           #####
########################################################
## Inspired by: https://github.com/pashword/pashword ##
## This script hopes to create a hashed password     ##
## that cannot be found in a pre-generated table     ##
#######################################################
```

> Please read script information.
{: .prompt-info }

```
#######################################################
## This script requires 135MB of free RAM            ##
## Be sure you have installed:                       ##
##       zip openssl argon2 gocryptfs fuse           ##
#######################################################
## What directory do you want to                     ## 
## store your password files in?                     ##
#######################################################
This is the only area in the script that would need
            user input for editing
--------------------------------------------------------
PASSWORD_FILES_LOCATION=~/Documents/.passwords
ENCRYPTED_PASSWORD_FILES_LOCATION=~/Documents/.passencrypted-on-disk
```


> Everything else prompts for input to generate the password hashes.


* * * 

## Visual representation of what the script does:
```
  ┌────────────────────────────────────────────────────┐
  │                                                    │
  │ gocryptfs directory created and password encrypted │
┌─┤                                                    │
│ └────────────────────────────────────────────────────┘
│
│
│    ┌────────────┐     ┌───────────┐     ┌────────────┐
│    │            │     │           │     │ ┌────────┐ │
│    │            │     │           │     │ │  I N   │ │
│    │ U S E R    │     │           │     │ │ memory │ │
└───►│            ├────►│ Combined  ├────►│ │Password│ ├─┐
     │            │     │  U S E R  │     │ └────────┘ │ │
     │  I N P U T │     │ I N P U T │     │  TXT       │ │
     │            │     │           │     │  FORMAT    │ │
     └────────────┘     └───────────┘     └────────────┘ │
                                                         │
┌────────────────────────────────────────────────────────┘
│
│  ┌─────────────┐  ┌────────────┐  ┌────────────────────┐
│  │ ┌─────────┐ │  │ ┌────────┐ │  │    unmounted       │
│  │ │  I N    │ │  │ │  I N   │ │  │    gocryptfs       │
│  │ │ memory  │ │  │ │ memory │ │  │    ──────────      │
│  │ │Password │ │  │ │Password│ │  │                    │
│  │ └─────────┘ │  │ └────────┘ │  │   Encrypted Zip    │
│  │             │  │ S A L T    ├─►│      Created       │
│  │             ├─►│    +       │  │                    │
└─►│  SHA3-384   │  │  Argon2    │  │     txt & hash     │
   │   H A S H   │  │  H A S H   │  │    files inside    │
   └─────────────┘  └────────────┘  └───────────────────┬┘
                                                        │
  ┌───────────────────────────────────────────────────┐ │
  │                                                   │ │
  │ Finished  Argon2  Hash  is  reprinted  on  screen │◄┘
  │                                                   │
  └───────────────────────────────────────────────────┘
```



# Script


```
#!/bin/bash
########################################################
###################   NOTES    #########################
##   -  Mnemonic for remembering argon2 prefs         ##
##         13       +      4     =    17              ##
##         13 interations, 4 cores, N^17 memory       ##
########################################################
########################################################
#####          What does this script do?           #####
########################################################
## Inspired by: https://github.com/pashword/pashword ##
## This script hopes to create a hashed password     ##
## that cannot be found in a rainbow table           ##
#######################################################
## This script requires 136MB of free RAM            ##
## Be sure you have installed:                       ##
##       zip openssl argon2 gocryptfs fuse           ##
#######################################################
## What directory do you want to                     ##
## store your password files in?                     ##
#######################################################
PASSWORD_FILES_LOCATION=~/Documents/.passwords
ENCRYPTED_PASSWORD_FILES_LOCATION=~/Documents/.passencrypted-on-disk
#######################################################
#######################################################
#### == Begin script - no need to edit anything == ####
#######################################################
### Directories are created, using an array and loop:
password_directories=("$PASSWORD_FILES_LOCATION" "$ENCRYPTED_PASSWORD_FILES_LOCATION")
## Loop through password_directories, if current_directory doesnt exist, create it.
for current_directory in "${password_directories[@]}"; do
    if [ ! -d "$current_directory" ]; then
        mkdir -p "$current_directory"
    fi
done
###
#######################################################
#### Benchmark for sleep length assignments
#######################################################
### Check if VAR_MIPS_SPEED is in one of the specified ranges
### Then, assign it an appropriately opinionated value
VAR_MIPS_SPEED=$(lscpu | grep -oP "BogoMIPS:\s+\K\w+")
if ((VAR_MIPS_SPEED >= 0 && VAR_MIPS_SPEED <= 3000)); then
  ENCRYPTION_SPEED=SLOW
elif ((VAR_MIPS_SPEED >= 3001 && VAR_MIPS_SPEED <= 5999)); then
  ENCRYPTION_SPEED=OK
elif ((VAR_MIPS_SPEED >= 6000 && VAR_MIPS_SPEED <= 9000)); then
  ENCRYPTION_SPEED=GOOD
else
  ENCRYPTION_SPEED=GREAT
fi
### Define an associative array to map ENCRYPTION_SPEED opinionated values to sleep durations
declare -A sleep_times
sleep_times["SLOW"]=12
sleep_times["OK"]=6
sleep_times["GOOD"]=4
sleep_times["GREAT"]=2
###
#######################################################
### The purpose of this function is to read user input and store it in a variable specified as an argument to the function.
#######################################################
function read_data {
  local data_name="$1"
  echo -e "Enter data for $data_name to use in this hash:"
  read -s $data_name
  declare -g $data_name
}
###
#######################################################
### Do we even need directories, or is this a quick hash?
#######################################################
read -p "Do you want a quick hash?  (Y/n)   - e.g. (this will not store a password) " hashOrNot
hashOrNot="${hashOrNot:-Y}"
if [[ $hashOrNot == "Y" || $hashOrNot == "y" ]]; then
echo -e "*************************************************************\nAnswer each prompt to generate a hash.\n\n(Example for Group: seniorclass, mymom, personalstuff)"
    read_data GROUP
    read_data AGE
    read_data WEBSITE_or_SERVICE
    read_data USERNAME
    read_data ZIP_PASSWORD
    clear
        PASWRD1_full=${GROUP}${AGE}${WEBSITE_or_SERVICE}:${USERNAME}${ZIP_PASSWORD}
        PASWRD1_salt=${GROUP}${WEBSITE_or_SERVICE}${USERNAME}
    echo -e "Your hashed password is:"
        echo -n $PASWRD1_full | openssl dgst -sha3-384 | echo -n $(awk '{print $2}') | argon2 ${PASWRD1_salt} -id -e -t 13 -m 17 -p 4 -l 32 | sed 's/.*\$//'
exit;
fi
###
#######################################################
### gocrypt fuse based stacked filesystem for encrypted data at rest
#######################################################
### Check if gocrypt folder has been initialized
### Run `gocryptfs -init` if no config file exists
### Nag screen for the user along with sleep timer
### Include for errors like: already mounted, incorrect password
umount $PASSWORD_FILES_LOCATION 2>/dev/null
if [ ! -e "$ENCRYPTED_PASSWORD_FILES_LOCATION/gocryptfs.conf" ]; then
    echo -e "****************************************************\nTo begin, please encrypt your passwords directory.\n              Use a STRONG password.\nAnd write down your MASTER KEY, when it appears.\n****************************************************"
    echo -e "Enter password to encrypt/decypt your passwords folder ( $PASSWORD_FILES_LOCATION ) with:"
    gocryptfs -init $ENCRYPTED_PASSWORD_FILES_LOCATION -deterministic-names -longnamemax 63
if [ $? -ne 0 ]; then
    echo "Please re-run script. The passwords you typed did not match. The process has failed."
  exit;
else
  echo -e "\n\nBe SURE you have WRITTEN DOWN or COPIED your MASTER KEY.\n\n"
fi
sleep 1; read -t 30 -p "... I am going to wait for only 30 seconds ...             (skip ahead with ENTER)"; echo "";
fi
###
### If a gocryptfs folder has been initialized
### Mount gocryptfs folder unencrypted to a read/write location
echo -e -n "**************************************************************\n   Access your on disk gocryptfs encrypted passwords folder\n**************************************************************\nEnter Password:"
gocryptfs $ENCRYPTED_PASSWORD_FILES_LOCATION $PASSWORD_FILES_LOCATION 2>/dev/null
if [ $? -ne 0 ]; then
    echo "Password incorrect. Please re-run script. This process has failed."
  exit;
else
    echo -e "\n"
fi
###
#######################################################
### Begin reading data from the user for a hash
#######################################################
# I wish I could restore cursor position before sending welcome banner
echo -e "**************************************************************************\n     Welcome to password generator, please follow instructions below\n**************************************************************************\nAnswer each prompt to generate a hash."
echo -e "\n(Example of a Group: seniorclass, mymom, personalstuff)"
    read_data GROUP
    read_data AGE
    read_data WEBSITE_or_SERVICE
    read_data USERNAME
    read_data ZIP_PASSWORD
        PASWRD1_full=${GROUP}${AGE}${WEBSITE_or_SERVICE}:${USERNAME}${ZIP_PASSWORD}
        PASWRD1_salt=${GROUP}${WEBSITE_or_SERVICE}${USERNAME}
echo -e "\n**********************************************************************\n   Let's pass this data into our hash  (may take up to 30 seconds)\n**********************************************************************"; sleep 1;
### If the group directory already exists compliment the user, or create the directory
if [ -d ${PASSWORD_FILES_LOCATION}/.${GROUP} ]
then
    echo "I am glad you like this script."
else
    mkdir -p ${PASSWORD_FILES_LOCATION}/.${GROUP}
fi
###
#######################################################
### Use a fifo to store data while being hashed and zipped
#######################################################
### Use a fifo (a named pipe), instead of writing to disk
### We'll be piping to stdin directly. The only gotcha is
### You have to pipe the data to fifo, to the background.
### You need this data to remains in memory for as long
### As the hashing takes before it can be encrypted.
#######################################################
### This `.txt` file created below is the plain text of the answers originally given above in the read_data function
mkfifo ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME}.txt && echo -n $PASWRD1_full > ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME}.txt &
sleep .5
### The file containing the finished hash is named without the `.txt`
mkfifo ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME} && echo -n $PASWRD1_full | openssl dgst -sha3-384 | echo -n $(awk '{print $2}') | argon2 ${PASWRD1_salt} -id -e -t 13 -m 17 -p 4 -l 32 | sed 's/.*\$//' > ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME} &
### Sleep for as long as the hash takes, to keep our file open in the pipe
sleep ${sleep_times["$ENCRYPTION_SPEED"]};
### ZIP up both files, `.txt` and hash, with the password that was chosen above.
zip --fifo --junk-paths -u -P ${ZIP_PASSWORD} ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}.zip ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME}.txt ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME} > /dev/null 2>&1; rm ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME}.txt ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME}
### Zip time is very fast
sleep .5;
### Zip is complete, files are now stored
### Remove all files used from memory
rm ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME}.txt ${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}--${USERNAME} > /dev/null 2>&1;
sleep .5
#######################################################
### Unmount unencypted gocryptfs folder and notify user
#######################################################
echo -e "\n*************************\n**********DONE!**********\n*************************"; sleep .5;
echo -e "An archive of your password was created.\nPlease take care to keep this gocrypt folder properly encrypted."; sleep .5;
umount $PASSWORD_FILES_LOCATION 2>/dev/null
echo -e "\nYour passwords folder is currently encrypted, and has a passworded zip\nwith 'the zip password' that you set earlier.\n"; sleep 1;
echo -e "You can access the encrypted passwords folder again with the command:"
echo -e "gocryptfs $ENCRYPTED_PASSWORD_FILES_LOCATION $PASSWORD_FILES_LOCATION"
echo -e "\nYour hashed and unhashed information is stored in:"
echo -e "${PASSWORD_FILES_LOCATION}/.${GROUP}/.password-${WEBSITE_or_SERVICE}.zip\n"
### Password is hashed one more time for display in the terminal
echo -e "Your final hashed password is:"
echo -n $PASWRD1_full | openssl dgst -sha3-384 | echo -n $(awk '{print $2}') | argon2 ${PASWRD1_salt} -id -e -t 13 -m 17 -p 4 -l 32 | sed 's/.*\$//'

```

- Script can be found [here](https://raw.githubusercontent.com/MarcusHoltz/password-hash-script/main/generate-password.sh), on my Github account.



# Explaining the script above

```
############################################
###      Why did I pick those values?    ###
############################################
# Phrase - helps the user keep these passwords organized in their head. It's repeatable through multiple passwords.
# Age - random number data, but also, reminds the user to change their password yearly. If you try and hash 23 and it doesnt work, hash 22, and it does -- that reminds you to update the hash to this years age. 
# Service - Very important. This helps name all the files, is also used in the salt.
# Username - Should be different for each site. Also used for the salt.
#################################################
```


## In memory, never on disk

**Use a fifo (a named pipe), instead of writing to disk.** 

To do this, we'll be piping to stdin directly. 

The only gotcha is when the data is piped to fifo it needs to run in the background. We need this data to remain in memory for as long as the hashing takes -- before it can be encrypted.

Then just remove the file and it's gone from memory, never on disk.



### SHA3-384 --> Argon2

User values are hashed, so one-way-encryption, first using SHA3-384 and then using the Argon2 algorithm the second time. This is to ensure maximum bruteforce and dictionary attack protection.

SHA3-384 was chosen due to Argon2's input size.



### Argon2 input size

128 minus 1 characters are supported in command line utility for Argon2. *So this means we have to use something smaller than 512 bits.*

- SHA3-384's hash is 384 bits long. 

- This will give us a nice long hash to send to our key derivation function.

Just for reference, maximum input length for bcrypt is 72 characters.



## Argon2 settings

With encryption, understand...  yes, you can get 50x more performance out of tweaking settings. 
But that's like saying there's a differnce between $500 trillion and $10 trillion. Either would be fine for my needs. 

I dont "NEED" 50x the original $10 trillion. It's almost pointless after your first $10 trillion.


### A hash that works for our system

You need to make a hash value that works for your system. Are you on 256 cores with 2TB of ram? Or on a Raspbery Pi? 

Best way to set these values is to consider them as parameters affecting computational costs:

- `-p` to decide how many threads you can run without delaying other processes on the CPU

- `-k` or `-m` to set how much memory you can assign to the hashing set

- `-t` number of iterations, set this as high as the other two values will allow


#### Example hash values for argon2

* * *

Defaults for `argon2` are:
- Type:           Argon2i
- Parallelism:    1
- Memory:         4096 KiB
- Iterations:     3
- Hash length:    32

* * *

`Bitwarden` default parameters are:
`-p 4 -k 62500 -t 3`
- Parallelism:    4
- Memory:         64 MB
- Iterations:     3

* * *

These example parameters provided by [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#argon2id) are equivalent in the defense they provide. 
The only difference is a trade off between CPU and RAM usage.

```
    m=47104 (46 MiB), t=1, p=1 (Do not use with Argon2i)
    m=19456 (19 MiB), t=2, p=1 (Do not use with Argon2i)
    m=12288 (12 MiB), t=3, p=1
    m=9216 (9 MiB), t=4, p=1
    m=7168 (7 MiB), t=5, p=1
```

* * *

Example parameters for `this script` are:
`-id -p 4 -m 17 -t 13 -l 32`
- Type:           Argon2id
- Parallelism:    4
- Memory:         134.218 MB
- Iterations:     13
- Hash length:    32

The value chosen above, `-id -p 4 -m 17 -t 13 -l 32` is half the time of: `-id -p 4 -m 18 -t 11 -l 32`

You can use the latter if you need harder encryption.

* * *

#### Explaining Argon2 parameters in detail

##### Iteration count

You need to keep your `iterations` **higher than `10`** to keep entropy in the millions of years.

- `-t` > 10


##### Length of Hash

In this example, the last value on our parameters, we're using 32 bits. This creates a hash like MD5 or any other:

- `-l 32`


##### Memory used during the hash

kB and GB are like the metric system, but this software wants kibibytes. Directly under 1GB is 976562 kibibytes (a whole GB is 976562.5 kibibytes).  1 Gigabyte is equal to (10^9 / 2^10) kibibytes. You can specify this with: `-k 976562`

To specify memory in KiB use the `-m` flag. This uses 2^N KiB so, `-m 20` would be about one gigabyte (24576 bytes more), and `-m 18` is 268.44 MB.

* * *

`-id -p 64 -m 20 -t 256 -l 32`

The command above, would take my CPU **10 minutes** to generate!

* * *

`-id -p 8 -m 20 -t 16 -l 32`

These parameters are more reasonable, taking less than **1 minute**.

* * *

But, the maximum I want is, maybe, *3 seconds*.

So we'll use **13 iterations, 134mb of memory, and 4 threads**. 

The example parameters of `-p 4 -m 17 -t 13` took my CPU (Passmark score of 5500) *3.2 seconds* to generate.

* * *

#### Running an example argon2 hash

Let's make an argon2 hash and demonstrate how this can be used:

On the command line: 

```bash
echo pants | argon2 somefantasticsalt -id -p 4 -m 17 -t 13 -l 32
```

Will give you a lot of information printed on the screen, but the actual hash comes after the last dollar sign `$`.

You can use CyberChef to find this same value, if you're unable to install Argon2:

[https://gchq.github.io/CyberChef/#recipe=Argon2](https://gchq.github.io/CyberChef/#recipe=Argon2(%7B'option':'UTF8','string':'somefantasticsalt'%7D,13,131072,4,32,'Argon2id','Encoded%20hash')&input=cGFudHM)

You can also [combine operations in CyberChef](https://gchq.github.io/CyberChef/#recipe=SHA3('384')Argon2(%7B'option':'UTF8','string':'somefantasticsalt'%7D,13,131072,4,32,'Argon2id','Encoded%20hash')&input=cGFudHM) to re-create this password hash in your webbrowser:


For more information, I've included argon2's help below:

```
       -id             Use Argon2id instead of Argon2
       -t N            Sets the number of iterations to N (default = 3)
       -m N            Sets the memory usage of 2^N KiB (default 12)
       -k N            Sets the memory usage of N KiB (default 4096)
       -p N            Sets parallelism to N threads (default 1)
       -l N            Sets hash output length to N bytes (default 32)
       -e              Output only encoded hash
```

* * *

### Beating Password Requirements


![make-password-entropy-not-rules](/assets/img/posts/password-rules-not-entropy.jpg){: #make-password-entropy-not-rules}


Sometimes the hashed password doesnt work out of the box. We need to modify it to be accepted into the password rules for the service or website we're trying to login to. If the hashed password, for some reason, doesnt meet the requirements on the website you're trying to use it with -- here are my suggestions:

* * *

If there's a special character requirement, usually just go with the first character in the upper left corner of the keyboard, beginning with '1' and '!'.

So a 'number' and 'special character' requirement would look like:

`1!a5i0dj68dhg59458fjhd`

Or if there's a 'two number' and 'one special character (baring !.<>$&())' requirement:

`12@a5i0dj68dhg59458fjhd`

The same goes for capitalization requirements, just capitalize the first letter in the password -- as required. 

`!1A5i0dj68dhg59458fjhd`

If you must remove any offending characters to the password rules, remove all as required.

`aidjdhgfjhd`

If you have to truncate the password:

The hash is shortened keeping the characters from the left and shortening farthest to the right. 

Example:

`abcdefghijklmn` --> `abcdefg`

also, max character requirements. If there's a 16 char max, make it a 15 char password, (N - 1).  

Thanks! Good Luck!


* * *

### This script switched from scrypt to argon2id ???!!

Why did I do this?

- [Argon2](https://en.wikipedia.org/wiki/Argon2) is a newer key derivation function and I find it more fun to use.

- [Bitwarden](https://community.bitwarden.com/t/recommended-settings-for-argon2/50901) and [KeePass](https://keepass.info/help/base/security.html) both use Argon2 as their encryption mechanism. 

- [OpenSSL](https://github.com/openssl/openssl/pull/12256) is soon supporting Argon2 and will reduce the requirements of this script. 




### What did the old script look like? 

```
#############################################################################################
##    These are the old scrypt methods I used, I no longer need them with Argon2           ##
#############################################################################################
## export PASWRD1_password
## echo -n $PASWRD1_full > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt
## zip --junk-paths -P ${PASWRD1_password} ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt.zip ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt > /dev/null 2>&1; rm ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt
## echo -n $PASWRD1_full | openssl dgst -sha3-512 | sed 's/^[^ ]* //' > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3
## sleep 1
## echo -e "Initial .password-${PASWRD1_service}-sha3 created!\n"
## echo -n ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3 | base64 > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64
## Appending some salt to our base64 hash we just made
## cp ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64 ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt
## echo -n ${PASWRD1_service}${PASWRD1_username} >> ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt
## # scrypt only outputs a crazy binary blob. It cannot copy and paste into a password field. So we're going to stop using it. K.THX.BYE.
## # The key derivation result (scrypt_key) is converted to a hexadecimal string using xxd -p -c 1000. The -p flag specifies plain hexdump, and -c 1000 ensures that the output doesn't wrap to the next line. This makes the Scrypt key readable and usable in subsequent concatenation and hashing operations.
## export PASWRD1_password
## # Setting "--logN 20 -r 8 -p 1" uses up about 1Gig of temp RAM space.
## # Setting "-M 1G" uses up about 1Gig of temp RAM space.
## scrypt enc --logN 20 -r 8 -p 1 --passphrase env:PASWRD1_password ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt-scryptbinary
## # The xxd command allows you to create a hex dump from a file.
## echo -n ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt-scryptbinary | xxd -p -c 1000000 > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex
## # Take that hex back to a shorter form, sha3-512
## echo -n .password-${PASWRD1_service}sha3-base64-salt-scrypthex | openssl dgst -sha3-512 | sed 's/^[^ ]* //' > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512
## echo -e "Finished ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512 created"
## sleep 5
## echo -e "==========================================="
## cp ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512 ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}
## echo -e "\nYour password for ${PASWRD1_service} is:"
## cat ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512
## unset PASWRD1_password
#############################################################################################
## Visual representation of old scrypt methods:
##
##  ┌────────┐     ┌────────┐      ┌─────────┐
##  │        │     │        │      │         │
##  │ User   │     │        │      │Password │
##  │        ├────►│Combined├─────►│         ├────┐
##  │ Input  │     │ User   │      │         │    │
##  │        │     │  Input │      │  TXT    │    │
##  │        │     │        │      │  FORMAT │    │
##  └────────┘     └────────┘      └─────────┘    │
##                                                │
##┌───────────────────────────────────────────────┘
##│
##│  ┌─────────┐     ┌──────────┐      ┌────────────┐
##│  │         │     │          │      │            │
##│  │Password │     │Password  ├─────►│ Password   │
##│  │         ├────►│          │      │            │
##└─►│         │     │          │      │    +       ├─┐
##   │SHA3-512 │     │ Base64   │      │            │ │
##   │         │     │          │      │  S a l t   │ │
##   │         │     │          │      │            │ │
##   └─────────┘     └──────────┘      └────────────┘ │
##                                                    │
##┌───────────────────────────────────────────────────┘
##│
##│ ┌──────────┐   ┌─────────┐   ┌──────────┐
##│ │          │   │         │   │          │
##│ │Password  │   │ Password│   │ Password │
##└►│          ├──►│         ├──►│          ├─────┐
##  │          │   │         │   │          │     │
##  │ Scrypt   │   │         │   │  H E X   │     │
##  │  BINARY  │   │   XXD   │   │ format   │     │
##  │          │   │         │   │          │     │
##  └──────────┘   └─────────┘   └──────────┘     │
##                                                │
##┌───────────────────────────────────────────────┘
##│
##│  ┌───────────┐        ┌──────────────────────────┐
##│  │           │        │                          │
##│  │ Password  │        │    PASSWORD              │
##│  │           │        │    Finished Hashing      │
##└─►│           ├───────►│                          │
##   │ SHA3-512  │        │                          │
##   │           │        │ Original Text in         │
##   │           │        │ .password-service.txt.zip│
##   └───────────┘        └──────────────────────────┘
##
#############################################################################################
```


* * *

### List of other sites that do the same thing

- [Spectre](https://spectre.app/)

- [LessPass](https://www.lesspass.com/)

- [Bookmark inside of Browser](javascript:var masterpw='uA,dTNMQ';var%20j%3D8%2Ck%3D%22%22%3B%20function%20l(b%2Cd)%7Bb%5Bd%3E%3E5%5D%7C%3D128%3C%3C24-d%2532%3Bb%5B(d%2B64%3E%3E9%3C%3C4)%2B15%5D%3Dd%3Bfor(var%20a%3DArray(80)%2Cc%3D1732584193%2Ce%3D-271733879%2Ch%3D-1732584194%2Cf%3D271733878%2Ci%3D-1009589776%2Cm%3D0%3Bm%3Cb.length%3Bm%2B%3D16)%7Bfor(var%20r%3Dc%2Cs%3De%2Ct%3Dh%2Cu%3Df%2Cv%3Di%2Cg%3D0%3Bg%3C80%3Bg%2B%2B)%7Ba%5Bg%5D%3Dg%3C16%3Fb%5Bm%2Bg%5D%3A(a%5Bg-3%5D%5Ea%5Bg-8%5D%5Ea%5Bg-14%5D%5Ea%5Bg-16%5D)%3C%3C1%7C(a%5Bg-3%5D%5Ea%5Bg-8%5D%5Ea%5Bg-14%5D%5Ea%5Bg-16%5D)%3E%3E%3E31%3Bvar%20w%3Dn(n(c%3C%3C5%7Cc%3E%3E%3E27%2Cg%3C20%3Fe%26h%7C~e%26f%3Ag%3C40%3Fe%5Eh%5Ef%3Ag%3C60%3Fe%26h%7Ce%26f%7Ch%26f%3Ae%5Eh%5Ef)%2Cn(n(i%2Ca%5Bg%5D)%2Cg%3C20%3F1518500249%3Ag%3C40%3F1859775393%3Ag%3C60%3F-1894007588%3A-899497514))%2Ci%3Df%2Cf%3Dh%2Ch%3De%3C%3C30%7Ce%3E%3E%3E2%2Ce%3Dc%2Cc%3Dw%7Dc%3Dn(c%2Cr)%3Be%3Dn(e%2Cs)%3Bh%3Dn(h%2Ct)%3Bf%3D%20n(f%2Cu)%3Bi%3Dn(i%2Cv)%7Dreturn%5Bc%2Ce%2Ch%2Cf%2Ci%5D%7Dfunction%20o(b)%7Bfor(var%20d%3D%5B%5D%2Ca%3D(1%3C%3Cj)-1%2Cc%3D0%3Bc%3Cb.length*j%3Bc%2B%3Dj)d%5Bc%3E%3E5%5D%7C%3D(b.charCodeAt(c%2Fj)%26a)%3C%3C24-c%2532%3Breturn%20d%7Dfunction%20n(b%2Cd)%7Bvar%20a%3D(b%2665535)%2B(d%2665535)%3Breturn(b%3E%3E16)%2B(d%3E%3E16)%2B(a%3E%3E16)%3C%3C16%7Ca%2665535%7D%20function%20p(b)%7Bfor(var%20d%3D%22%22%2Ca%3D0%3Ba%3Cb.length*4%3Ba%2B%3D3)for(var%20c%3D(b%5Ba%3E%3E2%5D%3E%3E8*(3-a%254)%26255)%3C%3C16%7C(b%5Ba%2B1%3E%3E2%5D%3E%3E8*(3-(a%2B1)%254)%26255)%3C%3C8%7Cb%5Ba%2B2%3E%3E2%5D%3E%3E8*(3-(a%2B2)%254)%26255%2Ce%3D0%3Be%3C4%3Be%2B%2B)d%2B%3Da*8%2Be*6%3Eb.length*32%3Fk%3A%22ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789%2B%2F%22.charAt(c%3E%3E6*(3-e)%2663)%3Breturn%20d%7Dwindow.removeElement%3Dfunction(b%2Cd)%7Bdocument.body.removeChild(document.getElementById(b))%3Bdocument.body.removeChild(document.getElementById(d))%7D%3B%20function%20q(b)%7Bif(b!%3D%3D%22%22%26%26b!%3D%3Dnull)%7Bvar%20d%3Ddocument.location.href.match(%2F%5E%5B%5E%3A%5D%2B%3F%3A%5C%2F%5C%2F%5C%2F%3F(www%5Cd%3F%5C.)%3F(%5B%5E%5C%2F%5D%2B)%2F)%5B2%5D%2Ca%2Cd%3D(a%3Dd.match(%2F%5B%5E.%5D%2B(%5C.(aero%7Carpa%7Casia%7Cbiz%7Ccat%7Ccom%7Ccoop%7Cco%7Cedu%7Cgov%7Cinfo%7Cint%7Cjobs%7Cmil%7Cmobi%7Cmuseum%7Cname%7Cnet%7Corg%7Cpro%7Ctel%7Ctravel%7Cxxx))%3F(%5C.%5Ba-z%5D%7B2%7D)%3F%24%2F))%3Fa%5B0%5D%3Ad%3Ba%3D%7B%22wikipedia.org%22%3A%2F%5E(wiki(%5Bpm%5Dedia%7Cbooks%7Csource%7Cquote%7Cnews%7Cspecies)%7Cmediawiki%7Cwiktionary)%5C.org%24%2F%2C%22example.com%22%3A%2Fexample%5C.(com%7Corg%7Cnet)%2F%7D%3Bfor(var%20c%20in%20a)d.match(a%5Bc%5D)%26%26(d%3Dc)%3Bb%3Dp(l(o(b%2B%22%3A%22%2Bd)%2C(b%2B%22%3A%22%2Bd).length*j)).substr(0%2C8)%3BString.prototype.a%3D%20function(a%2Cb)%7Breturn%20this.substr(0%2Ca)%2Bb%2Bthis.substr(a%2Bb.toString().length)%7D%3Bc%3Db.charCodeAt(0)%3Ba%3Dc%25b.length%3Bb.match(%2F%5Cd%2F)%7C%7C(b%3Db.a(a%2Cc%2510))%3Bvar%20e%3Dtrue%2Ch%3D%5B%22mecanto.com%22%2C%22tvtropes.org%22%5D%2Cf%3Bfor(f%20in%20h)if(d.match(h%5Bf%5D))%7Be%3Dfalse%3Bbreak%7De%26%26(f%3Db.length-a-1%2Cf%3D%3D%3Da%26%26f%2B%2B%2Cb%3Db.a(f%2CString.fromCharCode(32%2Bc%2515)))%3Bc%3Df%3D0%3Ba%3Ddocument.forms%3Be%3Dfalse%3Bfor(f%3D0%3Bf%3Ca.length%3Bf%2B%2B)if(a%5Bf%5D.id!%3D%22HMP%22)%7Bh%3Da%5Bf%5D.elements%3Bfor(c%3D0%3Bc%3Ch.length%3Bc%2B%2B)%7Bvar%20i%3Dh%5Bc%5D%3Bif(i.type%3D%3D%3D%22password%22%7C%7Ci.type%3D%3D%3D%22text%22%26%26i.name.match(%2Fp(ass%7Cw(or)%3Fd%3F)%2Fi))i.value%3D%20b%2Ci.focus()%2Ce%3Dtrue%7D%7De%7C%7Cwindow.prompt(%22Your%20password%20for%20%22%2Bd%2B%22%20is%22%2Cb)%7D%7Dwindow.hmp%3Dq%3B%20if(typeof%20masterpw%3D%3D%3D%22undefined%22%7C%7Cmasterpw%3D%3D%3D%22%22)%7Bvar%20x%3Ddocument.createElement(%22div%22)%3Bx.setAttribute(%22id%22%2C%22overlay%22)%3Bx.setAttribute(%22style%22%2C%22height%3A100%25%3B%20width%3A100%25%3B%20position%3Afixed%3B%20left%3A0%3B%20top%3A0%3B%20z-index%3A1%20!important%3B%20background-color%3Ablack%3B%20opacity%3A%200.75%3B%22)%3Bdocument.body.appendChild(x)%3Bvar%20y%3Ddocument.createElement(%22div%22)%3By.setAttribute(%22id%22%2C%22hashMypAssBox%22)%3By.setAttribute(%22style%22%2C%22padding%3A10px%3B%20background-color%3A%20%23dfdfdf%3B%20-moz-border-radius%3A5px%3B%20-webkit-border-radius%3A5px%3Bborder-radius%3A5px%3B%20position%3Afixed%3B%20top%3A30%25%3B%20left%3A40%25%3B%20z-index%3A2!important%3B%22)%3By.innerHTML%3D%20'%3Cform%20action%3D%22%22%20method%3D%22post%22%20style%3D%22margin%3A0%22%20id%3D%22HMP%22%3E%3Clabel%20for%3D%22passwordHashMypAssword%22%3EEnter%20your%20master%20password%3C%2Flabel%3E%3Cbr%3E%3Cinput%20type%3D%22password%22%20id%3D%22passwordHashMypAssword%22%20width%3D%22100%22%20%2F%3E%3Cbr%3E%3Cinput%20type%3D%22submit%22%20name%3D%22send%22%20value%3D%22submit%22%20onclick%3D%22javascript%3Ahmp(this.previousSibling.previousSibling.value)%3B%20removeElement(%5C'hashMypAssBox%5C'%2C%5C'overlay%5C')%3B%22%3E%3Ca%20href%3D%22javascript%3AremoveElement(%5C'hashMypAssBox%5C'%2C%5C'overlay%5C')%22%20title%3D%22Close%20HashMypAss%20Box%22%20class%3D%22closeButton%22%20style%3D%22float%3Aright%3B%20font-size%3Axx-small%3B%20position%3Arelative%3B%20top%3A1em%3B%22%3E(close)%3C%2Fa%3E%3C%2Fform%3E'%3B%20document.body.appendChild(y)%3Bdocument.getElementById(%22passwordHashMypAssword%22).focus()%7Delse%20q(masterpw)%3B)
