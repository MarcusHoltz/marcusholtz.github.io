---
title: Unique passwords for every service or website
date: 2023-08-14 11:33:00 -0700
categories: [Linux, Security]
tags: [hash, scripts, security, cryptography, passwords]
pin: false
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

- At no point is anything unecrypted written to disk.


```
########################################################
#####          What does this script do?           #####
########################################################
## Inspired by: https://github.com/pashword/pashword ##
## This script hopes to create a hashed password     ##
## that cannot be found in a pre-generated table     ##
#######################################################
```

> #### Please read script information.
{: .prompt-info }

```
#######################################################
## This script requires 270MB of free RAM            ##
## Be sure you have installed:                       ##
##          zip, openssl, argon2                     ##
#######################################################
## What directory do you want to                     ## 
## store your password files in?                     ##
#######################################################
This is the only area in the script that would need
            user input for editing
--------------------------------------------------------
PASSWORD_FILES_LOCATION=~/Documents/.passwords
```


> Everything else prompts for input to generate the password hashes.


* * * 

## Visual representation of what the script does:
```
     ┌────────────┐     ┌───────────┐     ┌────────────┐
     │            │     │           │     │            │
     │            │     │           │     │ ┌────────┐ │
     │            │     │           │     │ │  I N   │ │
     │ U S E R    │     │           │     │ │ memory │ │
     │            ├────►│ Combined  ├────►│ │Password│ ├─┐
     │            │     │  U S E R  │     │ └────────┘ │ │
     │  I N P U T │     │ I N P U T │     │  TXT       │ │
     │            │     │           │     │  FORMAT    │ │
     └────────────┘     └───────────┘     └────────────┘ │
                                                         │
┌────────────────────────────────────────────────────────┘
│
│  ┌─────────────┐  ┌────────────┐  ┌─────────────────────────┐
│  │             │  │            │  │                         │
│  │ ┌─────────┐ │  │ ┌────────┐ │  │       ┌────────┐        │
│  │ │  I N    │ │  │ │  I N   │ │  │       │  I N   │        │
└─►│ │ memory  │ ├─►│ │ memory │ ├─►│       │ memory │        │
   │ │Password │ │  │ │Password│ │  │       │Password│        │
   │ └─────────┘ │  │ └────────┘ │  │       └────────┘        │
   │  SHA3-384   │  │  Argon2    │  │ Encrypted Zip Created   │
   │   H A S H   │  │  H A S H   │  │ txt & hash files inside │
   └─────────────┘  └────────────┘  └────────────────────┬────┘
                                                         │
 ┌───────────────────────────────────────────────────┐   │
 │                                                   │   │
 │ Finished  Argon2  Hash  is  reprinted  on  screen │◄──┘
 │                                                   │
 └───────────────────────────────────────────────────┘
```



# Script


```
#!/bin/bash
########################################################
#####          What does this script do?           #####
########################################################
## Inspired by: https://github.com/pashword/pashword ##
## This script hopes to create a hashed password     ##
## that cannot be found in a rainbow table           ##
#######################################################
## This script requires 270MB of free RAM            ##
## Be sure you have installed:                       ##
##          zip, openssl, argon2                     ##
#######################################################
## What directory do you want to                     ##
## store your password files in?                     ##
#######################################################
PASSWORD_FILES_LOCATION=~/Documents/.passwords
#######################################################
# Make sure that directory exists, or create it
if [ -d ${PASSWORD_FILES_LOCATION} ]
then
    echo "directory exists already"
else
    mkdir -p ${PASSWORD_FILES_LOCATION}
fi
# Done with directory concerns
echo -e "\n**************************************************************************\nWelcome to password generator, please follow instructions below\n**************************************************************************"
echo -e "Enter password phrase used to identify what group this goes in:\n(Example: seniorclass, mymom, personalstuff)"
read -s PASWRD1_phrase
echo -e "Great!\n"; sleep .5; echo -e "Now enter your age:"
read -s PASWRD1_age
echo -e "Is this a marketing survey, or a password generator...\n"; sleep .5; echo -e "Next, with the **first letter capitalized**, \nEnter the name of the **Website or Service** you're creating a password for:"
read -s PASWRD1_service
echo -e "\nBe sure that first character was a capital letter!"; sleep 1
echo -e "This script will insert a colon now     :     \n"
echo -e "Now enter your username, tied to the service above:"
read -s PASWRD1_username
echo -e "Last Step.\n\nEnter a password that you can remember:"
read -s PASWRD1_password
echo -e "\nGreat!"; sleep .5; echo -e "We're done entering our information!"
sleep 3;
echo -e "\n**********************************************************************\n   Let's pass this data into our hash  (may take up to 30 seconds)\n**********************************************************************"; sleep 1;
# Record the variables we need later on
PASWRD1_full=${PASWRD1_phrase}${PASWRD1_age}${PASWRD1_service}:${PASWRD1_username}${PASWRD1_password}
PASWRD1_salt=${PASWRD1_service}${PASWRD1_username}
# Use a fifo (a named pipe), instead of writing to disk -- we'll be piping to stdin directly. The only gotcha is you have to pipe the data to fifo, to the background, so you need this data to remains in memory for as long as the hashing takes before it can be encrypted.
# .txt file created are the plain text answers that were given above, for reference
mkfifo ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username}.txt && echo -n $PASWRD1_full > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username}.txt &
sleep 2;
# the file containing the hash is named without the .txt
mkfifo ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username} && echo -n $PASWRD1_full | openssl dgst -sha3-384 | sed 's/.*[[:space:]]//' | argon2 ${PASWRD1_salt} -id -e -t 16 -m 18 -p 8 -l 32 | sed 's/.*\$//' > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username} &
sleep 17;
# ZIP up both files, original and hash, with the password that was chosen above.
zip --fifo --junk-paths -u -P ${PASWRD1_password} ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}.zip ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username}.txt ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username} > /dev/null 2>&1; rm ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username}.txt ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username}
sleep 3;
# remove files in memory
rm ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username}.txt ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}--${PASWRD1_username} > /dev/null 2>&1;
echo ""; sleep 2;
echo -e "\n*************************\n**********DONE!**********\n*************************"; sleep .5;
echo -e "An archive of your password was created.\nPlease take care to properly encrypt this folder."; sleep .5;
echo -e "It is currently only encrypted as a zip,\nwith 'the password that you can remember' you gave earlier.\n"; sleep 1;
echo -e "Your hashed and unhashed information is stored in:"
echo -e "${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}.zip\n"
echo -e "Your final hashed password is:"
echo -n $PASWRD1_full | openssl dgst -sha3-384 | sed 's/.*[[:space:]]//' | argon2 ${PASWRD1_salt} -id -e -t 16 -m 18 -p 8 -l 32 | sed 's/.*\$//'
```


# Explaining the script above

```
###########################################
###      Why did I pick those values?    ##
###########################################
# Phrase - helps the user keep these passwords organized in their head. It's repeatable through multiple passwords.
# Age - random number data, but also, reminds the user to change their password yearly. If you try and hash 23 and it doesnt work, hash 22, and it does -- that reminds you to update the hash to this years age. 
# Service - Very important. This helps name all the files, is also used in the salt.
# Username - Should be different for each site. Also used for the salt.
###########################################
```



## In memory, never on disk

**Use a fifo (a named pipe), instead of writing to disk.** 

To do this, we'll be piping to stdin directly. 

The only gotcha is when the data is piped to fifo it needs to run in the background. We need this data to remain in memory for as long as the hashing takes -- before it can be encrypted.
Then just remove the file and it's gone from memory, never on disk.



## SHA then Argon2

User values are hashed, so one-way-encryption, first using SHA3-384 and then using the Argon2 algorithm the second time. This is to ensure maximum bruteforce and dictionary attack protection.

SHA3-384 was chosen due to Argon2's input size.



## Argon2 input size

128 minus 1 characters are supported in command line utility for Argon2. *So this means we have to use something smaller than 512 bits.*

- SHA3-384's hash is 384 bits long. 

- This will give us a nice long hash to send to our key derivation function.

Just for reference, maximum input length for bcrypt is 72 characters.


## Argon2 settings

You need to keep your iterations higher than 10 to keep entropy in the millions of years:  `Argon2 - t=10, m=512.000, p=4       is      John the Ripper: 1 Password/second `

```
       -id             Use Argon2id instead of Argon2
       -t N            Sets the number of iterations to N (default = 3)
       -m N            Sets the memory usage of 2^N KiB (default 12)
       -k N            Sets the memory usage of N KiB (default 4096)
       -p N            Sets parallelism to N threads (default 1)
       -l N            Sets hash output length to N bytes (default 32)
       -e              Output only encoded hash
```

The only reason for a minimum password length is to prevent easy to guess passwords. There is no purpose for a maximum length.

A maximum length specified on a password field should be read as a SECURITY WARNING. Any sensible, security conscious user must assume the worst and expect that this site is storing your password literally (i.e. not hashed).

Setting maximum password length less than 128 characters is discouraged by OWASP. Yet, atleast 10% of the time I have to create a password... character limits. This is the reality, not the ideal.

* * *

In this example, we're using 32 bits. This creates a hash like MD5 or any other:

`-l 32`

kB and GB are like the metric system, but this software wants kibibytes. Directly under 1GB is 976562 kibibytes (a whole GB is 976562.5 kibibytes).  1 Gigabyte is equal to (10^9 / 2^10) kibibytes. You can specify this with: `-k 976562`

To specify memory in KiB use the `-m` flag. This uses 2^N KiB so, `-m 20` would be about one gigabyte (24576 bytes more), and `-m 18` is 268.44 MB.

`-id -e -t 256 -m 20 -p 64 -l 32`

The command above, would take my CPU 10 minutes to generate!

* * *

The maximum I want is, maybe, 30 seconds of patience. So we'll use 16 iterations, 270mb of memory, and 8 threads. 

This combination took my CPU (Passmark score of 5500) 7 seconds to generate.

Be sure to use the config below:

```bash
-id -e -t 16 -m 18 -p 8 -l 32
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

The hash is shortened from the left to the right. 

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
