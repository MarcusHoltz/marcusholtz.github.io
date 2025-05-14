---
layout: post
title: Turning Network Services Off At Specified Time
date: 2025-05-12 11:33:00 -0700
categories: [DevOps, Authentication]
tags: [server, turnitoff, devops, authentication, security, networking, authentik, opnsense, openwrt, oauth]
pin: false
image:
  path: /assets/img/header/header--traefik--turn-off-at-hour-authentik-opnsense-openwrt.jpg
  alt: Turn Your Network Services Off When They're Not Needed
---





# 5pm turn off services when business hours end or while asleep at home



## Homelab

Your homelab should boast, at max, 80% uptime.

If you're asleep - unable to respond, why have services running?

- WiFi turns off

- Restrictive Firewall Rules in Place

- Disconnect any remote VPNs

- Disable app login


This concept can be expanded beyond the homelab.



## Reverse Proxy business case with Traefik

In the next example we'll do **smart routing** based on business hours.

Let's say you run a pizzeria's website. 

They're not accepting orders while the business is closed.

You could use a reverse proxy to forward `/orders/`

to a non-app container, a static page, mitigating attack surface, and AI crawler expense.


A great idea for managing resources effectively without sacrificing the customer experience. 




## Follow Along System

This guide is using:

- OpenWRT firmware

- OPNSense

- Authentik: https://blog.holtzweb.com/posts/traefik-forwardauth-authentication-authentik/

In addition, you will need:

- [An application](https://docs.goauthentik.io/docs/add-secure-apps/applications/manage_apps) you can bind a policy to. This post uses Portainer.




## Turn your WiFi Off

All of your security cameras are corded and powered, riiiight? I mean, you know how easy it is to knock out a wifi signal. A motivated person would easily disable a wireless camera system, and you knew better!

Less noise.
Stop those background devices (smart TVs, IoT, phones) from endlessly talking overnight.

Everyone is at home, asleep. 
Less radiation in our house, the better. Turn the wifi off. 

Your business has employees home for the evening.
Less power consumption and chance for a threat actor to do recon.

Reduced screen time.
Turning off wifi can help reduce late-night screen time and encourage better routine and sleep habits.


## OpenWRT as the access point


### Step 1: Update OPKG package list

First thing is to update the package list

![Update OpenWRT list of OPKG packages](turnoffservices--openwrt-1.png)


Wait as the package manager updates

![OPKG update command is running to ensure a up-to-date list of packages](turnoffservices--openwrt-2.png)


### Step 2: Search OpenWRT software package list

Use the `filter` search bar to search for `wifi`

Install `wifischedule`

Install `luci-app-wifischedule`

![Filter for wifi to find wifischedule](turnoffservices--openwrt-3.png)

### Step 3:Using Wifi Schedule on OpenWRT

Under `Services > wifischedule`

Go to the `Global Settings` section

Make sure that `Enable Wifi Schedule` is checked

Head down to `Schedule events`

![wifischedule on OpenWRT needs enabled](turnoffservices--openwrt-4.png)

Under `Schedule events`

At the bottom of the screen, you will see an empty field and an `Add` button

You must enter the name of your event to continue, then hit the `Add` button

Again, at the bottom, enter the name of the schedule you'd like to create, example - `BUSINESSHOURS` or `WEEKEND`

Select the `Day(s) of the week` you'd like to schedule

Select the `Start WiFi` time

Select the `Stop WiFi` time

Be sure to `Enable mode` to continue

![Add your wifischedule name, date, and time](turnoffservices--openwrt-5.png)


### Step 4: View your added schedule in cron

You can verify that you entered everything correctly by checking OpenWRT's crontab

Head over to `System > Scheduled Tasks`

Here you can see the time, days, and the script being executed for those respective times. Cool.

![Look at OpenWRT crontab to be sure our configuration worked](turnoffservices--openwrt-6.png)










## Change OPNSense firewall rules based on a schedule

You may also want to lock down the same VLAN on your router that is on your Access Point we just turned off.

You may want to use this concept a different way, and restrict network access or internet for certain subnets or vlans. 

"No internet after dinnertime."

"Sorry, guest Wifi only routes connections during vistor hours."

Or, possibly, you have a close-to-air-gapped network that runs manual updates at a specific time, and restrict the internet gateway to that window of time.



## OpenWRT firewall schedules

### Step 1: Create a new firewall schedule

To create a new schedule:

- `Firewall > Settings > Schedules`

- Click on the `orange plus` in the upper right corner


![Find firewall schedules under firewall settings](turnoffservices--opnsense-firewall-0.png)


### Step 2: Add the new firewall schedule

- Enter a `name` (This is a one time thing, you cannot change this)

- You can change the `description`

- Select the `days` you want this to be active

- Adjust the `start time` and `stop time` for your firewall rule

- Click on `Add Time` to create the schedule

- You will see your new schedule at the bottom of the page

- Hit `Save` at the bottom


![Configure your firewall schedule for your needs](turnoffservices--opnsense-firewall-1.png)



### Step 3: Adding or Editing a firewall rule to schedule

To add a new firewall rule, or edit an old rule:

- `Firewall > Rules`

- You can see an existing firewall rule that has been configured with a `Schedule`

- You should also see this rule has an `X` icon to indicate it is a `Blocking` firewall rule

- Additionally, the `Schedule` and `Description` are both `crossed out`, to indicate this rule is not active.

![OPNsense firewall rules page](turnoffservices--opnsense-firewall-2.png)


- Click on the `orange plus` sign to make a new rule or the `pencil icon` to edit an existing rule.

![Adding or Editing a firewall rule](turnoffservices--opnsense-firewall-3.png)



### Step 4: Configuring an OPNSense firewall rule to schedule

On this new page there is only one section that schedules the rule, but we'll go over the rule in the screen shot as well:

- Action: `Block`

- Disabled: `Disable this rule` (your rule will turn itself on and off as needed)

- Select the `interface` that this rule is for

- `any` for all. This is a block everything rule, any 2 any.

![Creating a blocking firewall rule any to any](turnoffservices--opnsense-firewall-4.png)


Scrolling down the page:

- Description: `Defines the rule we're creating`

- **Schedule:** `Enter the name of the schedule you made` on [Step 2](#).

- `Save`

![Add the schedule created in step 2 to the firewall rule](turnoffservices--opnsense-firewall-5.png)












## Creating an OPNSense crontab action

If there isnt a crontab action already made for a service you need to modify, you can make it yourself.

In this example we're going to be making an action for Wireguard, to start and stop the service.


## OpenWRT custom Wireguard action cron schedule

### Step 1: Let's look at the script

We're going to add a `.conf` file to OPNSense's `action.d` directory for it's `services`.

First let's look at this script we're going to make and stuff in this folder.

Looking from a console directly on OPNSense:

- You can see the directory the file needs to be in: `/usr/local/opnsense/service/conf/actions.d/`

- You can also see the output of the file: `actions_wireguard--start.conf`

![OPNSense console viewing actions_wireguard--start.conf](turnoffservices--opnsense-wireguard-6.png)


### Step 2: SSH in and paste the files required

This may be difficult to read, so let's SSH in and view this outside of a console terminal:

- In the `/usr/local/opnsense/service/conf/actions.d/` directory

- Both `actions_wireguard--start.conf` and `actions_wireguard--stop.conf` are the files we'll be making

* * *

#### actions_wireguard--start.conf

```ini
[start]
command:/usr/local/opnsense/scripts/Wireguard/wg-service-control.php
parameters: start %s
type:script
message: start wireguard instance %s
description:Z Turn on WireGuard
```

* * *

#### actions_wireguard--stop.conf

```ini
[stop]
command:/usr/local/opnsense/scripts/Wireguard/wg-service-control.php
parameters: stop %s
type:script
message: stop wireguard instance %s
description:Z Stop on WireGuard
```

![OPNSense SSH remote connection viewing actions_wireguard--start.conf](turnoffservices--opnsense-wireguard-7.png)


### Step 3: Create actions in cron

With those files in place:

- `System > Settings > Cron`

![OPNSense web interface for cron](turnoffservices--opnsense-wireguard-1.png)


- Click on the `orange plus` to add a new cron job

![Adding a new crontab to OPNSense](turnoffservices--opnsense-wireguard-2.png)


### Step 4: Setup the cron job for Wireguard

#### Using cron time format

You will need to add a cronjob for both the `on`and the `off` of the Wireguard service.

This screenshot section only demonstrates adding a single service. **Please remember to add both!**

- `Enable` the checkbox for this job

- You can configure `Minutes` and `Days of the month` and `Months` and `Days of the Week` the same way as cron.

  - If you need any help with your cron syntax, check out [https://crontab.guru](https://crontab.guru).


![Using cron format to setup a cronjob](turnoffservices--opnsense-wireguard-3.png)

#### Input the Wireguard interface

Please note, the action takes an input. 

- Under `Parameters` you will need to input your Wireguard interface

  - The first Wireguard interface you make is `wg0` so try that if you dont know what to put.

![Adding a parameter to the cron action](turnoffservices--opnsense-wireguard-4.png)


#### Select the command to run

- `Command` will be the **description** of the **action** in the directory **actions.d** you made earlier on [Step 2](#).


![Selecting the action of the cron tab](turnoffservices--opnsense-wireguard-4.png)







## Disable Web Application Login

Let's say you want to restrict your web applications. 

You want to disable the authorization of specific web application based on the time.

That's what we're here for.

## Authentik behind Traefik

This demonstration uses Authentik to provide the authorization and custom script management that we need to make this work.


### Step 1: Enter Application Policy Binding

Authentik has added a lot, this is one of those newer things - you can create and bind a policy to an application with one button.

This guide is written assuming you already have a working Application. 

If you do not, please see: [Ibracorp](https://docs.ibracorp.io/authentik/authentik/docker-compose/traefik-forward-auth-single-applications#authentik-config), [Helge Klein](https://helgeklein.com/blog/authentik-authentication-sso-user-management-password-reset-for-home-networks/#authentik-initial-configuration), [Jim's Garage (YouTube)](https://youtu.be/enwFWELCYJo)

- `Authentik Web UI > Applications > Applications`

![Authentik applications location in the menu](turnoffservices--authentik-03.png)


This example is going to use Portainer as the `app requiring authentication` before use.

- Click on your `APPNAME` or hit the `pencil icon`

![Enter the application to bind a policy to](turnoffservices--authentik-04.png)

- Once on the new page, at the top

- Click on `Policy/Group/User/Bindings`

![Authentik Application user bindings page in the menu](turnoffservices--authentik-05.png)

- With the new tab open

- Click on `Create and bind Policy`

![Application create and bind policy button](turnoffservices--authentik-06.png)

- The `new policy` page will appear 

- Click on `Expression Policy`

- Click `Next`

![Expression Policy creation for an Application on Authentik](turnoffservices--authentik-07.png)

- Think of the `name` you want to use to identify this python script. 

- This name will also appear on the `Authentik rejection page` as to why the user wasnt allowed in.

- In this example, `Sorry, Business Hours Only (8am - 5pm)`

![New Policy Name for the Expression Policy created for an Application on Authentik](turnoffservices--authentik-08.png)


- Under `Policy-specific settings` is where you can enter your python

- You can put the Python code in the `Expression` code block section

- This example is going to use a Python expression to only allow logins during business hours (8am to 5pm) and block access at all other times:

```python
from datetime import datetime
current_hour = datetime.now().hour
# Return True (allow) if time is between 8am and 5pm (17:00)
# Return False (deny) at all other times
return 8 <= current_hour < 17
```

This expression will:

- Allow access between `8:00am` and `4:59pm`

- Deny access from `5:00pm` until `7:59am` the `next day`

- Click `Next` when done

![Python code for returning datetime](turnoffservices--authentik-09.png)

The last page is for assigning the Policy, Enabling, Inverting, Ordering, and Assigning a Pass/Dont

- `Policy` should be whatever policy name you write in earlier

- `Enabled` this should be checked

- Scroll to the bottom

![On a binding for a policy you must enable and select the policy](turnoffservices--authentik-10.png)

We dont want people to login if the current_hour is before 8am or after 5pm.

- `Failure result` should be `dont pass`.

- Click `Finish`

![What is the result of the bound expression - pass or dont pass](turnoffservices--authentik-11.png)

You can review what you just created on the page you arrived at earlier

- `Applications > Applications > Application_Name > Policy / Group / User Bindings`

- Click on `Edit Binding`, just for good fun.

![Authentik Applications page now has the Bound Policy we created](turnoffservices--authentik-12.png)

Just to test out our rule, as it's still business hours where I'm at...

- Edit the Binding and Update it to Invert the Result

- Click `Negate result`

- Click `Update`

![Negate the result of an Application's Bound Policy](turnoffservices--authentik-13.png)

- Now try and use Authentik login to your test application.

![Using Authentik to login to an OAuth page](turnoffservices--authentik-14.png)

- This page is blocking us from logging in!

- The page is displaying `Policy binding 'Binding from Portainer to Policy`

- Then in the same line it lists our Expression Policy's name used to identify the python script envoking this rule `Sorry - Business Hours Only (8am - 5pm)`

- The value returned is then displayed for the user `returned result 'False'`

![A blocked authentication from Authentik for attempting to login after hours](turnoffservices--authentik-15.png)

But none of this is real if we cant prove it to our boss.

- `Events > Logs` 

![Authentik logs page is under events and logs](turnoffservices--authentik-16.png)

Once one the page for event logs, we need to search for the failed login.



![Authentik logs deeper dive, exploring logs](turnoffservices--authentik-17.png)

If you've located the offending login, hit the arrow down button to drill down and expand the action.


![Drilling down into Authentik logs with an event](turnoffservices--authentik-18.png)


Here is the information of the bad person. Oh no!!!

- The website they tried to sign in to

- The device they were using

- The identity of the person and the account used

![Authentik Log information about the offending user](turnoffservices--authentik-19.png)


You can view and edit the Policy you made, incase you need to change the hours

- `Customization > Policies`

![Location of custom expression policies in Authentik](turnoffservices--authentik-21.png)


To edit the policy:

- Find the policy you want to edit

- Click on the `pencil icon`



![To make changes to the policy click the pencil icon](turnoffservices--authentik-22.png)



Here you can adjust the policy in any way you want,

- In this example, adjust the time of day between 8am and 5pm


![A sloppy example of how to change the policy, this is from 8am to 5pm](turnoffservices--authentik-23.png)


## Congratulations









