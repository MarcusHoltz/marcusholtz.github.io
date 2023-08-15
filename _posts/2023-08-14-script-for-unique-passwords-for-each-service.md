---
title: Unique passwords for every service or website
date: 2023-08-14 11:33:00 -0700
categories: [Linux]
tags: [hash, security, cryptography, passwords]
pin: false
image:
  path: /assets/img/header/header--script--unique-password-hashing.jpg
  alt: Create a new password for every service or website you visit
---
# Let's hash some information into a password

> #### The creation of different passwords for every service or website you're using can become a pain.
>
> - You have to keep them unique and very different.
> - Access without password manager is a pain.
>
>  *What if* there was a better **way?**

Using a repeatable method to generate these passwords, while remaining secure, can help speed up password creation and give you a system to finding lost passwords.
{: .prompt-tip }

> Please keep in mind that hashing is a one way operation. 

- All of the hashes generated are stored within the .passwords folder

- The only hash that isnt available is the original plain text version.

- The original plain text of our hash is stored in a passworded zip (the password used is the one chosen inside the script).


```
########################################################
#####          What does this script do?           #####
########################################################
## Inspired by: https://github.com/pashword/pashword ##
## This script hopes to create a hashed password     ##
## that cannot be found in a pre-generated table     ##
#######################################################
```

#### What do you need to do to configure this to get it working?
{: .prompt-info }

```
#######################################################
## This script requires 1GB of free RAM              ##
## Be sure you have installed:                       ##
##          zip, openssl, scrypt, xxd                ##
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

## Visual representation of each hash step:
```
  ┌────────┐     ┌────────┐      ┌─────────┐
  │        │     │        │      │         │
  │ User   │     │        │      │Password │
  │        ├────►│Combined├─────►│         ├────┐
  │ Input  │     │ User   │      │         │    │
  │        │     │  Input │      │  TXT    │    │
  │        │     │        │      │  FORMAT │    │
  └────────┘     └────────┘      └─────────┘    │
                                                │
┌───────────────────────────────────────────────┘
│
│  ┌─────────┐     ┌──────────┐      ┌────────────┐
│  │         │     │          │      │            │
│  │Password │     │Password  ├─────►│ Password   │
│  │         ├────►│          │      │            │
└─►│         │     │          │      │    +       ├─┐
   │SHA3-512 │     │ Base64   │      │            │ │
   │         │     │          │      │  S a l t   │ │
   │         │     │          │      │            │ │
   └─────────┘     └──────────┘      └────────────┘ │
                                                    │
┌───────────────────────────────────────────────────┘
│
│ ┌──────────┐   ┌─────────┐   ┌──────────┐
│ │          │   │         │   │          │
│ │Password  │   │ Password│   │ Password │
└►│          ├──►│         ├──►│          ├─────┐
  │          │   │         │   │          │     │
  │ Scrypt   │   │         │   │  H E X   │     │
  │  BINARY  │   │   XXD   │   │ format   │     │
  │          │   │         │   │          │     │
  └──────────┘   └─────────┘   └──────────┘     │
                                                │
┌───────────────────────────────────────────────┘
│
│  ┌───────────┐        ┌──────────────────────────┐
│  │           │        │                          │
│  │ Password  │        │    PASSWORD              │
│  │           │        │    Finished Hashing      │
└─►│           ├───────►│                          │
   │ SHA3-512  │        │                          │
   │           │        │ Original Text in         │
   │           │        │ .password-service.txt.zip│
   └───────────┘        └──────────────────────────┘
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
## This script requires 1GB of free RAM              ##
## Be sure you have installed:                       ##
##          zip, openssl, scrypt, xxd                ##
#######################################################
## What directory do you want to                     ## 
## store your password files in?                     ##
#######################################################
PASSWORD_FILES_LOCATION=~/Documents/.passwords
#######################################################
# Make sure that directory exists, or create it
if [ -d "~/Documents/.passwords" ] 
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
echo -e "Not too old...\n"; sleep .5; echo -e "Next, with the **first letter capitalized**, \nEnter the name of the **Website or Service** you're creating a password for:"
read -s PASWRD1_service
echo -e "\nBe sure that first character was a capital letter!"; sleep 1
echo -e "This script will insert a colon now     :     \n"
echo -e "Now enter your username, tied to the service above:"
read -s PASWRD1_username
echo -e "Last Step.\n\nEnter a password that you can remember:"
read -s PASWRD1_password
echo -e "\nGreat!"; sleep .5; echo -e "We're done entering our information!"
sleep 3;
echo -e "\n********************************************************\nLet's combine these into a string to pass into openssl\n********************************************************"; sleep 1;
PASWRD1_full=${PASWRD1_phrase}${PASWRD1_age}${PASWRD1_service}:${PASWRD1_username}${PASWRD1_password}
echo -n $PASWRD1_full > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt
# Password up that raw text file with the original combination of information we used, for reference.
zip --junk-paths -P ${PASWRD1_password} ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt.zip ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt > /dev/null 2>&1; rm ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-txt
echo -n $PASWRD1_full | openssl dgst -sha3-512 | sed 's/^[^ ]* //' > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3
sleep 1
echo -e "Initial .password-${PASWRD1_service}-sha3 created!\n"
echo -n ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3 | base64 > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64
# Appending some salt to our base64 hash we just made
cp ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64 ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt
echo -n ${PASWRD1_service}${PASWRD1_username} >> ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt
#scrypt only outputs a crazy binary blob. It cannot copy and paste into a password field.
# The key derivation result (scrypt_key) is converted to a hexadecimal string using xxd -p -c 1000. The -p flag specifies plain hexdump, and -c 1000 ensures that the output doesn't wrap to the next line. This makes the Scrypt key readable and usable in subsequent concatenation and hashing operations.
export PASWRD1_password
# Setting "--logN 20 -r 8 -p 1" uses up about 1Gig of temp RAM space.
# Setting "-M 1G" uses up about 1Gig of temp RAM space. 
scrypt enc --logN 20 -r 8 -p 1 --passphrase env:PASWRD1_password ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt-scryptbinary
# The xxd command allows you to create a hex dump from a file.
echo -n ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}-sha3-base64-salt-scryptbinary | xxd -p -c 1000000 > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex
# Take that hex back to a shorter form, sha3-512
echo -n .password-${PASWRD1_service}sha3-base64-salt-scrypthex | openssl dgst -sha3-512 | sed 's/^[^ ]* //' > ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512
echo -e "Finished ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512 created"
sleep 5
echo -e "==========================================="
cp ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512 ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}
echo -e "\nYour password for ${PASWRD1_service} is:"
cat ${PASSWORD_FILES_LOCATION}/.password-${PASWRD1_service}sha3-base64-salt-scryptbinary-hex-sha3-512
unset PASWRD1_password
```

## Beating Password Requirements

If there's a special character requirement, usually just go with the first character in the upper left corner of the keyboard, begining with '1' and '!'.

So a 'number' and 'special character' requirement would look like:

`1!a5i0dj68dhg59458fjhd`

Or if there's a 'two number' and 'one special character (baring !.<>$&())' requirement:

`12@a5i0dj68dhg59458fjhd`

The same goes for capitalization requirements, just capitalizae the first letter in the password -- as required. 

`!1A5i0dj68dhg59458fjhd`

If you have to truncate the password:

The hash is shortened from the left to the right. 

Example:

`abcdefghijklmn` --> `abcdefg`

also, max character requirements. If there's a 16 char max, make it a 15 char password, (N - 1).  

Thanks! Good Luck!
