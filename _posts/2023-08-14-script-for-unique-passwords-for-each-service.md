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
# Create a new password for a service or website

The creation of different passwords for every service or website you're using can become a pain.

Using a repeatable method to generate these passwords, while remaining secure, can help speed up password creation and give you a system to finding lost passwords.

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

## What do you need to do to configure this to get it working?

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




