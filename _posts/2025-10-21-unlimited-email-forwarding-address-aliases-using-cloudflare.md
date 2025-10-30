---
layout: post
title: Generating Unlimited Email Alias with Conditional Rules
date: 2025-10-21 11:33:00 -0700
categories: [DevOps, Email]
tags: [cloud, serverless, email, administration, security, customization, privacy, mirror, guide, communication]
pin: false
image:
  path: /assets/img/header/header--email--mail-forwarding-with-cloudflare-email-routing-workers.jpg
  alt: Forwarding Email using Alias with Cloudflare Email Routing Workers
---

# Cloudflare's Email Routing for Generating Unlimited Email Alias with Conditional Rules


## Intro

I have been looking at all the ways to generate forwarding addresses. I wanted to create an email address per account, and not use one overlapping email address.

Here are some of the services available already in that category:

- [AliasVault](https://www.aliasvault.net/)
- [Addy.io](https://addy.io/#pricing)
- [SimpleLogin](https://simplelogin.io/pricing/)
- [MySudo](https://anonyome.com/individuals/mysudo-plans/) 
- [Cloaked.com](https://www.cloaked.com/plans?vid=019a112e-133e-7c3f-af4f-bc3a7060c66b) 
- [ForwardEmail](https://forwardemail.net/en/self-hosted)

Those are great, you can even integrate some into [Bitwarden](https://bitwarden.com/blog/add-privacy-and-security-using-email-aliases-with-bitwarden/), infact [PCWorld](https://www.pcworld.com/article/2217318/how-to-get-unlimited-masked-emails-to-keep-your-inbox-secure-and-private.html) has a whole write-up on it.

But I'm afraid of the idea of being **locked** into a company that **may** one day **disappear**, and **self-hosting** to generate forwarding addresses, just doesnt *vibe*.


* * *

So, I thought - nope, we do this instead:

- [Cloudflare Email Routing](https://www.cloudflare.com/developer-platform/products/email-routing/) using [Cloudflare's Free Email Workers](https://developers.cloudflare.com/email-routing/email-workers/)

Now you can just log into your cloudflare dashboard and edit this script instead of a web UI with buttons.


* * *

## What This Method Allows For

✅ **No vendor lock-in** - Just Cloudflare javascript - make changes in the script

✅ **Unlimited aliases** - Create as many as you want - no need for additional domains

✅ **Free** - Up to 100k emails/day - this is per account and not per domain

✅ **Version Control** - Backup and export rules - config is in script header

✅ **Multiple Functions** - You can create forwarding lists, block spam, and filter emails
 

***

[This script](#the-script-that-needs-edited) basically automates the busy work of sorting your emails efficiently on your domain before they reach your inbox, all controlled by easy-to-edit rules.


* * *

## Steps to Configure Email Forwarding using Alias from Cloudflare's Email Routing with Conditional Rules

> I haven’t gone into email privacy — and yes, that’s a bit ironic for something involving routing with Cloudflare. I do care about privacy and plan to make it a key part of the setup down the line. For now, my main goal is to build a solid, long-term system that runs smoothly and gives me some actual peace of mind.

Here are the steps (with screenshots) needed to get the Cloudflare email worker up and runnnig. 

You will need to configure the script before you paste it in, just a heads up.

There is a section after this screen shot configuration of Cloudflare email workers to help you setup the email worker script for your needs.


* * *

### Step 1. Create a cloudflare account

To create a Cloudflare account:

- Go to the [Sign up page ↗](https://dash.cloudflare.com/sign-up).

- Enter your Email and Password.

- Select Create Account.

- Once you create your account, Cloudflare will automatically send an email to your address to verify that email address.

![Step1-Create-Account](/assets/img/posts/Cloudflare-Forwarding-Email---Step1---Create-Account.png)
*This screenshot demonstrates the Sign up page.*


![Step2-Login-to-Account](/assets/img/posts/Cloudflare-Forwarding-Email---Step2---Login-to-Account.png)
*You are logged in once you see the loading screen*

![Step3-Dashboard-View](/assets/img/posts/Cloudflare-Forwarding-Email---Step3---Dashboard-View.png)
*The Dashboard view page, with the menu on the left*


* * *

### Step 2. Bring your domain into Cloudflare: Either with Nameserver change or DNS entries

To use Cloudflare you need to Onboard or Buy a domain:

- Click on the appropreate button for your option.

- Begin the onboarding process by setting the nameserver.

![Step4-Bring-in-Domain](/assets/img/posts/Cloudflare-Forwarding-Email---Step4---Bring-in-Domain.png)
*The button for Onboarding a domain or Buying a domain*

![Step5-Domain-Imported](/assets/img/posts/Cloudflare-Forwarding-Email---Step5---Domain-Imported.png)
*Instructions to setup the Nameserver with your domain*

* * *

### Step 3. Log into your domain registrar to complete step 2

If you didnt buy your domain with Cloudflare, you may need to login to your registrar to change the nameservers:

- Login to your domain's dashboard

- Look for things like: Enter my own Nameservers, Custom DNS, DNS Management, Edit Nameserver

![Step6-Changing-Nameservers](/assets/img/posts/Cloudflare-Forwarding-Email---Step6---Changing-Nameservers.png)
*Namecheap example screenshot to Add a Nameserver*

* * *

### Step 4. Go to whereever to have to generate the DNS entries for all the MX and DMARC SPF records

Once Cloudflare emails you to verify your nameserver has propigated (up to 24 hours), you can begin to modify your DNS records:

- Cloudflare will check your current records for compatability

- You will need to add MX and TXT records for Cloudflare's Email services (this is available as a guided wizard)

![Step7-Cloudflare-Update-MX-Records](/assets/img/posts/Cloudflare-Forwarding-Email---Step7---Cloudflare-Update-MX-Records.png)
*The guided wizard that imports your DNS records to Cloudflare*

* * *

### Step 5. Login to cloudflare's dashboard (we'll be refering to the "new sidebar" when referencing the menu)

After you've got your DNS setup, you can begin to setup Cloudflare Email Routing:

- Use the menu on the left to locate the `Email` > `Email Routing` page

![Step8-Email-Routing-Menu-Location](/assets/img/posts/Cloudflare-Forwarding-Email---Step8---Email-Routing-Menu-Location.png)
*The menu on the left highlighting the Email Routing page*

![Step9-Email-Routing-Page](/assets/img/posts/Cloudflare-Forwarding-Email---Step9---Email-Routing-Page.png)
*The Email Routing page overview tab*

* * *

### Step 6. email/routing/destination-address

Generate a `destination address` for any address you’re thinking about forwarding to (Gmail accounts):

- Click on `Add destination address`

- Add your email as a Verified Destination

- Click Send verification email

- Check your inbox and click the verification link

- Wait for it to show as "Verified" in Cloudflare

![Step10-Email-Destination-Addresses-Page](/assets/img/posts/Cloudflare-Forwarding-Email---Step10---Email-Destination-Addresses-Page.png)
*The Destination address tab on the Email Routing page*

![Step11-Add-Destination-Addresses](/assets/img/posts/Cloudflare-Forwarding-Email---Step11---Add-Destination-Addresses.png)
*Placeholder Destination addresses that have been properly verified*

* * *

### Step 7. /email/routing/workers

**This is the last step before having to edit a script, to paste into our worker.**

You will need to be sure you have correctly changed the sections of the script to fit your addresses and needs before adding them:

- You can find the [area required to edit this script down below](https://blog.holtzweb.com/posts/unlimited-email-forwarding-address-aliases-using-cloudflare/#basic-configuration-examples)

- If you're ready to create an email worker, click `Create`

- Name it and select `Create my own`


![Step12-Email-Workers-Page](/assets/img/posts/Cloudflare-Forwarding-Email---Step12---Email-Workers-Page.png)
*The Email Workers tab on the Email Routing page*

![Step13-Create-Email-Worker](/assets/img/posts/Cloudflare-Forwarding-Email---Step13---Create-Email-Worker.png)
*Creating an Email Worker*

![Step14-Create-My-Own-Email-Worker](/assets/img/posts/Cloudflare-Forwarding-Email---Step14---Create-My-Own-Email-Worker.png)
*The Create my own option, allowing for custom javascript code*


* * *

### Step 8. /workers/services/edit/email-worker-1/production

**Again, second warning - make sure you have edited the config section of the script before pasting it in** 

You will need to be sure you have correctly changed the sections of the script to fit your addresses and needs before adding them:

- Replace any placeholder email addresses with correct addresses

- You can find more more information in the [area required to edit this script down below](https://blog.holtzweb.com/posts/unlimited-email-forwarding-address-aliases-using-cloudflare/#basic-configuration-examples)

- When the script is ready, open the `Code editor` from the Worker you made

- Erase any code there already and paste in your modified script

- Hit the Deploy button to send into action
- 

![Step15-Replace-Your-Real-Email-Address-in-the-Script](/assets/img/posts/Cloudflare-Forwarding-Email---Step15---Replace-Your-Real-Email-Address-in-the-Script.png)
*Replacing the placeholder address with my real Gmail address*

![Step16-Replace-Your-Custom-Domain-in-the-Script](/assets/img/posts/Cloudflare-Forwarding-Email---Step16---Replace-Your-Custom-Domain-in-the-Script.png)
*Replacing the placeholder domain with the Onboarded Cloudflare domain, smith-family.com*

![Step17-Lets-Edit-Some-Code](/assets/img/posts/Cloudflare-Forwarding-Email---Step17---Lets-Edit-Some-Code.png)
*Back at the Email Routing page, under the Email Workers tab, and a recently created Worker*

![Step18-Remove-the-Template-Code](/assets/img/posts/Cloudflare-Forwarding-Email---Step18---Remove-the-Template-Code.png)
*The default empty worker.js template*

![Step19-Paste-in-the-Modified-Script-with-Replaced-Placeholders](/assets/img/posts/Cloudflare-Forwarding-Email---Step19---Paste-in-the-Modified-Script-with-Replaced-Placeholders.png)
*The pasted in modified script ready to work*

* * * 

### Step 9. email/routing/routes

You need to have a catch all that will forward to your email worker. (DONT click "create route" on your email-worker):

- On the Email Routing page, under Routing Rules

- The Catch-all address area needs to be turned on and edited

- On the Edit catch-all address page, select `Send to a Worker` and select your Worker you made

- Save

![Step20-Adding-an-Email-Routing-Rule-to-Catch-All](/assets/img/posts/Cloudflare-Forwarding-Email---Step20---Adding-an-Email-Routing-Rule-to-Catch-All.png)
*Routing Rules tab on the Email Routing page with Catch-all addresses*

![Step21-Edit-the-Routing-Rule-to-send-Catch-All-to-the-Worker](/assets/img/posts/Cloudflare-Forwarding-Email---Step21---Edit-the-Routing-Rule-to-send-Catch-All-to-the-Worker.png)
*Adding a catch-all address to work with the Email Worker*

* * *

### Step 10. /email/routing/overview

With the tasks complete, you can go back to the overview page to see some changes:

- You should see your destination addresses you made

- Scrolling down you can find Email Routing Summary

- Activity of any emails sent or recieved

- So go send a test email

![Step22-Verify-Destination-Addresses](/assets/img/posts/Cloudflare-Forwarding-Email---Step22---Verify-Destination-Addresses.png)
*Overview of the Email Routing section*

![Step23-Send-Some-Emails-to-be-Routed](/assets/img/posts/Cloudflare-Forwarding-Email---Step23---Send-Some-Emails-to-be-Routed.png)
*Email Routing summary section of the last 7 days*

![Step24-View-the-Email-Activity-Log](/assets/img/posts/Cloudflare-Forwarding-Email---Step24---View-the-Email-Activity-Log.png)
*The Email Routing Activity Log on the Overview Tab*


* * *

### Step 11. /workers/services/view/email-worker-1/production/observability/logs?workers-observability-view=invocations

Now that you sent a test email, did it work? 

- Enable Worker Logs

- Read your Logs

![Step25-Enable-Worker-Logs-in-Observability](/assets/img/posts/Cloudflare-Forwarding-Email---Step25---Enable-Worker-Logs-in-Observability.png)
*Worker Services Page under the Worker created in the Observability tab*

![Step26-View-Your-Worker-Logs-in-Observability](/assets/img/posts/Cloudflare-Forwarding-Email---Step26---View-Your-Worker-Logs-in-Observability.png)
*Observability working and ingesting worker logs that will appear here*

* * * 

### CONGRATS! 

Now you should be all setup to use email alias with Cloudflare Email Routing.

If it didnt work...

- Help is here
- Help is here


* * * 

## Basic Configuration Examples

Here's the section that helps explain what you need to edit. 

* * *

### Summary of How to Apply These Examples

- To make changes, add or modify entries in the `routingRules` array.

- Use the `to` field to target incoming specific recipient email addresses.

- Use the `from` field to filter based on sender email or domain.

- Use the `recipients` array to specify where emails should be forwarded (**your Cloudflare verified destination addresses**).

- Use `block`, `blockKeywords`, or `forwardKeywords` to control which emails get rejected, forwarded, or ignored.

- Each rule should have a friendly `description` for clarity in your logs.

- Deploy this updated script as a Cloudflare Email Worker bound to your domain email routes.

- Monitor logs to check routing success and troubleshoot.



* * *

### Example 1. Email to Email Forwarding

Here you can send an email coming to amazon@yourdomain.com into another recipient's inbox, your-email@gmail.com.

And to stop getting emails from amazon@yourdomain.com - just remove what you added.

```javascript
{
  to: "amazon@yourdomain.com",
  recipients: ["your-email@gmail.com"],
  description: "Amazon account"
}
```


* * *

### Example 2. Block Specific Keywords

Lets say you wanted a better method to filter your messages, you can block phrases or keywords in the subject line.

In this example, your email you use to sign up to social media accounts, social@yourdomain.com, normally forwards to your personal email at, our-email@gmail.com, but you're getting too many messages from social media and there's no way to turn them off. Well, now you can selectivly stop those "follow back xxyy" and "xxyy liked aabb" emails by adding them to the blockedKeywords.

```javascript
{
  to: "social@yourdomain.com",
  recipients: ["your-email@gmail.com"],
  blockKeywords: ["follow back", "liked"],
  description: "Social media"
}
```

Or how about using a specific email just for your Playstation account? No need to worry about store spam anymore!

```javascript
  {
    to: "playstation@yourdomain.com",
    recipients: ["your-email@gmail.com"],
    blockKeywords: ["playstation store", "sale", "offer"],
    description: "PlayStation - no store spam"
  }
```


* * *

### Example 3. Forward Only On Keyword Match

Instead of just blocking a few phrases, you can block every phrase!

Allow only selective keywords or phrases that match to be allowed to be delivered.

```javascript
  {
    to: "steam@yourdomain.com",
    recipients: ["your-email@gmail.com"],
    forwardKeywords: ["security", "login", "password", "purchase"],
    description: "Steam - security and purchases only"
  }
```



* * *

### Example 4. Spam Trap Junk Address  

Then, there's the option of blocking everything.

Use the example email, spamtrap@yourdomain.com, when forced to give email to sketchy sites - it gets sinkholed.

Any emails sent to your spam trap address (spamtrap@yourdomain.com) are automatically blocked.

```javascript
{
  to: "spamtrap@yourdomain.com",
  block: true,
  description: "Block all spam trap emails"
}
```


* * *

### Example 5. Family Priority Inbox  

This example lets us recieve email but only from a specific domain. In this example, it's the family's domain.

We can create a rule that only accepts certain allowed senders for `familymessages@yourdomain.com`. Emails from close family or friends - sending email from the domain trustedfamily.com - will be allowed to arrive on that inbox, and any other sources, like gmail, will be blocked.

```javascript
{
  to: "familymessages@yourdomain.com",
  from: "@trustedfamily.com",  // only emails from this domain forwarded
  recipients: ["your-email@gmail.com"],
  description: "Family emails"
}
```


* * *

### More Configuration Examples Below

There are a lot of different ways you can use this script, so I wanted to include [more examples](#more-configuration-with-scenario-based-examples).

- [Scenario 1: Online Shopping Accounts](#scenario-1-online-shopping-accounts)

- [Scenario 2: Financial Account Separation](#scenario-2-financial-account-separation)

- [Scenario 3: Smart Home & Family Management](#scenario-3-smart-home--family-management)

- [Scenario 4: Gaming & Social Media Isolation](#scenario-4-gaming--social-media-isolation)


Skip to those if you want to learn more. 




* * *
* * *

## How to Edit the Config

I wanted to be sure to give some reference to what is going on, and what everything in the configuration examples mean before we continue.


* * *

### Understanding the Symbols used in Javascript

- `{ }` = A container that holds related information
- `[ ]` = A list of items
- `" "` = Text (always needs quotes around it)
- `,` = Separates items in a list (like "and" between items)
- `//` = A comment/note that the computer ignores


* * *

### Editing Your Settings

#### Step 1. Change the Default Email Address

Find the line:

```javascript
defaultRecipient: "your-email@gmail.com",
```


* * *

Change it to YOUR email:

```javascript
defaultRecipient: "john.smith@gmail.com",
```

- Keep the quotes `" "`

- Keep the comma at the end `,`

- Just replace the text inside the quotes



* * *

#### Step 2. Keywords to Block in Every Email Ever

Find this section:

```javascript
globalBlockKeywords: [
  "viagra", "casino", "lottery", "prince nigeria", 
  "bitcoin mining", "make money fast"
],
```


* * *

Add/remove/change/ keywords:

```javascript
globalBlockKeywords: [
  "pair it up",
  "brand-new iphone",
  "crypto giveaway",
  "click here now"
],
```

- Each keyword in quotes `" "`

- Comma after each one `,` EXCEPT the last

- Keywords are NOT case-sensitive ("VIAGRA" = "viagra")


* * *

#### Step 3. Create an Email Routing Rule

Find the `routingRules:` section. Let's break down what the routingRules look like:

```javascript
routingRules: [
  {
    to: "shopping@yourdomain.com",
    recipients: ["your-email@gmail.com"],
    description: "Shopping accounts"
  },
```

* * *

In the example above:

```javascript
{
  to: "amazon@mydomain.com",                    ← Email address people send TO
  recipients: ["john.smith@gmail.com"],         ← YOUR real email (where it forwards)
  description: "Amazon account"                 ← Note to yourself (optional)
},                                              ← Comma ONLY if there's another rule below 
```


* * *

#### Step 4. Adding Multiple Rules

Let's say you want to set up 3 email addresses. Here's how:

```javascript
routingRules: [
  {
    to: "amazon@mydomain.com",
    recipients: ["john.smith@gmail.com"],
    description: "Amazon purchases"
  },
  {
    to: "netflix@mydomain.com",
    recipients: ["john.smith@gmail.com"],
    description: "Netflix account"
  },
  {
    to: "bank@mydomain.com",
    recipients: ["john.smith@gmail.com", "jane.smith@gmail.com"],
    description: "Bank alerts - sent to both of us"
  }
]
```

**Notice:**

- First two rules have commas `,` after the `}` because another rule follows

- Last rule has NO comma after `}` because it's the last one

- Each rule is wrapped in `{ }` (curly brackets)

- All rules are inside `[ ]` (square brackets)


* * *

## Common Gotchas and What to Look For

Just reviewing everything discussed, incase your config doesnt work **come here**.

* * *

### Mistake 1: Commas Go Between Items, and NOT After the Last One

```javascript
// ❌ WRONG - comma after last item causes errors
recipients: [
  "email1@gmail.com",
  "email2@gmail.com",
  "email3@gmail.com",  ← Remove this comma!
]
```

```javascript
// ✅ CORRECT - comma between items, none after last
recipients: [
  "email1@gmail.com",
  "email2@gmail.com",
  "email3@gmail.com"
]
```


* * *

### Mistake 2: Missing Quotes

```javascript
// ❌ WRONG - no quotes
recipients: [john@gmail.com]
```



```javascript
// ✅ FIXED - quotes around the text
recipients: ["john@gmail.com"]
```

* * *

### Mistake 3: Commas Also in routingRules

Not just when listing between brackets `[]` do you need to be careful of commas `,` but also when listing between each routing rule `{ }`

```javascript
// ❌ WRONG - comma after last item causes errors
routingRules: [
  {
    to: "test@mydomain.com",
    recipients: ["john@gmail.com"],
    description: "Test"
  },  ← Remove this comma! (it's the last rule)
]
```


```javascript
// ✅ FIXED - No comma on last rule
routingRules: [
  {
    to: "test@mydomain.com",
    recipients: ["john@gmail.com"],
    description: "Test"
  }
]
```

* * *

### Mistake 4: Forgetting to Replace "example.com"


```javascript
// ❌ WRONG - You need to change "yourdomain.com"
to: "amazon@yourdomain.com",
```


```javascript
// ✅ FIXED - Using YOUR actual domain
to: "amazon@smith-family.com",
```

* * *

## Quick Checklist Before Running

✅ Changed `defaultRecipient` to your real email?  

✅ All emails in quotes `" "`?  

✅ Commas between items, but NOT after the last one?  

✅ Replaced `@yourdomain.com` with your actual domain?  

✅ All recipient emails are verified in Cloudflare?  



* * *

## More Configuration with Scenario Based Examples


Here are some real-world scenarios that demonstrate the power of this setup. I'll show you exactly what to modify in the `CONFIG` section for each use case.

* * *

### Scenario 1: Online Shopping Accounts

**Goal:** Create unique email addresses for each retailer so you can:

- Track which companies sell your data (spam sources)

- Organize receipts automatically

- Block promotional emails from specific stores


```javascript
routingRules: [
  {
    to: "amazon@yourdomain.com",
    recipients: ["receipts@gmail.com"],
    blockKeywords: ["prime day", "deal of the day", "limited time"],
    description: "Amazon - receipts only, no promos"
  },
  {
    to: "target@yourdomain.com",
    recipients: ["receipts@gmail.com"],
    description: "Target purchases"
  },
  {
    to: "bestbuy@yourdomain.com",
    recipients: ["receipts@gmail.com", "tech-alerts@gmail.com"],
    forwardKeywords: ["shipped", "delivered", "order"],
    description: "Best Buy - only order updates to both emails"
  },
  {
    to: "shopping-test@yourdomain.com",
    recipients: ["junk@gmail.com"],
    description: "Test new stores here first"
  }
]
```

**Real-world logic:**

- Sign up at Amazon with `amazon@yourdomain.com`

- All order confirmations go to your receipts folder

- Promotional emails with "Prime Day" get blocked automatically

- If Amazon sells your email and you get spam, you know exactly who leaked it

- You can block `amazon@yourdomain.com` entirely without affecting other accounts


* * *

### Scenario 2: Financial Account Separation

**Goal:** Keep banking, investments, and crypto separate with different security levels


```javascript
routingRules: [
  {
    to: "chase@yourdomain.com",
    recipients: ["main@gmail.com", "important@gmail.com"],
    forwardKeywords: ["security", "alert", "unusual", "login", "password"],
    description: "Chase bank - urgent alerts to phone & email"
  },
  {
    to: "chase-statements@yourdomain.com",
    recipients: ["statements@gmail.com"],
    description: "Chase - monthly statements only"
  },
  {
    to: "vanguard@yourdomain.com",
    recipients: ["investments@gmail.com"],
    blockKeywords: ["webinar", "market update", "newsletter"],
    description: "Vanguard - trades only, no marketing"
  },
  {
    to: "coinbase@yourdomain.com",
    recipients: ["crypto@protonmail.com"],
    description: "Crypto on separate secure email"
  },
  {
    from: "@irs.gov",
    recipients: ["main@gmail.com", "spouse@gmail.com"],
    description: "IRS emails to both of us"
  }
]
```

**Real-world logic:**

- Only the security alerts from Chase go to your important address

- You use different addresses for login vs. statements (phishing protection)

- Investment newsletters get blocked but trade confirmations come through

- Crypto stays on a completely separate email provider (extra security layer)

- Tax-related emails automatically CC your spouse


* * *


### Scenario 3: Smart Home & Family Management

**Goal:** Handle newsletters, school communications, smart home alerts, and shared family accounts


```javascript
routingRules: [
  {
    to: "school@yourdomain.com",
    recipients: ["parent1@gmail.com", "parent2@gmail.com"],
    forwardKeywords: ["urgent", "emergency", "sick", "incident", "pickup"],
    description: "School - urgent messages to both parents"
  },
  {
    to: "school-newsletter@yourdomain.com",
    recipients: ["family-archive@gmail.com"],
    description: "School newsletters - low priority"
  },
  {
    to: "ring@yourdomain.com",
    recipients: ["parent1@gmail.com"],
    forwardKeywords: ["motion", "doorbell"],
    description: "Ring doorbell alerts"
  },
  {
    to: "nest@yourdomain.com",
    recipients: ["parent1@gmail.com"],
    forwardKeywords: ["smoke", "carbon", "alert"],
    description: "Nest - safety alerts only"
  },
  {
    to: "newsletters@yourdomain.com",
    recipients: ["reading@gmail.com"],
    description: "Substack, Medium, etc."
  },
  {
    to: "spam-trap@yourdomain.com",
    block: true,
    description: "Use this when forced to give email to sketchy sites"
  }
]
```

**Real-world logic:**

- School emergency emails go to both parents immediately

- Regular newsletters go to archive (read when you have time)

- Smart home devices only alert on actual events (not app updates)

- You can give `spam-trap@yourdomain.com` to random sign-up forms that require email


* * *

### Scenario 4: Gaming & Social Media Isolation

**Goal:** Keep gaming, social media, and entertainment separate from important accounts


```javascript
routingRules: [
  {
    to: "steam@yourdomain.com",
    recipients: ["gaming@gmail.com"],
    forwardKeywords: ["security", "login", "password", "purchase"],
    description: "Steam - security and purchases only"
  },
  {
    to: "playstation@yourdomain.com",
    recipients: ["gaming@gmail.com"],
    blockKeywords: ["playstation store", "sale", "offer"],
    description: "PlayStation - no store spam"
  },
  {
    to: "twitter@yourdomain.com",
    recipients: ["social@gmail.com"],
    forwardKeywords: ["mentioned you", "direct message"],
    description: "Twitter - only interactions"
  },
  {
    to: "facebook@yourdomain.com",
    recipients: ["social@gmail.com"],
    blockKeywords: ["suggested", "friend request", "you may know"],
    description: "Facebook - block noise"
  },
  {
    to: "netflix@yourdomain.com",
    recipients: ["entertainment@gmail.com"],
    description: "Netflix account"
  },
  {
    to: "spotify@yourdomain.com",
    recipients: ["entertainment@gmail.com"],
    blockKeywords: ["premium", "upgrade", "offer"],
    description: "Spotify - no upsells"
  }
]
```

**Real-world logic:**

- Gaming platforms generate tons of promotional emails—block them at source

- Social media only notifies you of direct interactions

- Streaming services separated from important email

- If your gaming email gets compromised, no access to banking/critical accounts



* * *

* * *

## The Script That Needs Edited

Here is the code to place in your Email Routing Worker you created, be sure to edit the configuration section first before you deploy it.


```javascript
// ============================================
// CONFIGURATION SECTION - EDIT THIS PART
// ============================================

const CONFIG = {
  // Default forwarding address (fallback)
  // CHANGE THIS TO YOUR REAL EMAIL ADDRESS
  defaultRecipient: "your-main-email@example.com",
  
  // Enable/disable logging to console (visible in Cloudflare dashboard)
  enableLogging: true,
  
  // Log every email (true) or only blocked/errors (false)
  logAllEmails: true,
  
  // Global subject keywords that will BLOCK emails
  globalBlockKeywords: [
    "crypto giveaway", "win a free car", "prince nigeria", "you could have already won", "dont think make money fast"
  ],
  
  // The rules before are checked in order, first match wins
  routingRules: [
    {
      // Route specific email address to specific recipient
      to: "shopping@yourdomain.com",
      recipients: ["your-shopping-email@example.com"],
      description: "Shopping accounts"
    },
    {
      // Multiple recipients example
      to: "important@yourdomain.com",
      recipients: [
        "your-main-email@example.com",
        "your-backup-email@example.com"
      ],
      description: "Important - send to multiple addresses"
    },
    {
      // Block specific keywords for this address
      to: "newsletter@yourdomain.com",
      recipients: ["your-newsletters@example.com"],
      blockKeywords: ["unsubscribe failed", "re-subscribe"],
      description: "Newsletters - block resubscribe attempts"
    },
    {
      // Forward only if subject contains specific keywords
      to: "school@yourdomain.com",
      recipients: ["parent1@gmail.com", "parent2@gmail.com"],
      forwardKeywords: ["urgent", "emergency", "sick", "incident", "pickup"],
      description: "School - urgent messages to both parents"
    },
    {
      // Block all emails to this address
      to: "spam-trap@yourdomain.com",
      block: true,
      description: "Spam trap - block everything"
    }
  ]
};
//
// To find more routingRule examples:
// https://blog.holtzweb.com/posts/unlimited-email-forwarding-address-aliases-using-cloudflare/#more-configuration-with-scenario-based-examples
//
// ============================================
// EMAIL WORKER CODE - NO NEED TO EDIT BELOW
// ============================================

export default {
  async email(message, env, ctx) {
    const startTime = Date.now();
    
    try {
      // Extract email details with fallbacks
      const from = message.from || "unknown@sender.com";
      const to = message.to || "";
      const subject = message.headers.get("subject") || "(no subject)";
      
      // Validate we have required fields
      if (!to) {
        console.error("Missing 'to' address in email");
        message.setReject("Missing recipient address");
        return;
      }
      
      // Log incoming email
      if (CONFIG.enableLogging && CONFIG.logAllEmails) {
        console.log(`[${new Date().toISOString()}] Incoming email:`, {
          from,
          to,
          subject,
          size: message.rawSize
        });
      }
      
      // Check global block keywords in subject
      const subjectLower = subject.toLowerCase();
      for (const keyword of CONFIG.globalBlockKeywords) {
        if (subjectLower.includes(keyword.toLowerCase())) {
          logAction("BLOCKED", "Global keyword match", {
            from, to, subject, keyword
          });
          message.setReject(`Blocked by keyword filter: ${keyword}`);
          return;
        }
      }
      
      // Find matching routing rule
      const matchedRule = findMatchingRule(message, from, to, subject);
      
      if (!matchedRule) {
        // No rule matched, use default recipient
        logAction("FORWARDED", "Default routing", {
          from, to, subject, 
          recipients: [CONFIG.defaultRecipient]
        });
        await message.forward(CONFIG.defaultRecipient);
        return;
      }
      
      // Check if rule blocks this email
      if (matchedRule.block) {
        logAction("BLOCKED", `Rule: ${matchedRule.description}`, {
          from, to, subject
        });
        message.setReject("Blocked by routing rule");
        return;
      }
      
      // Check rule-specific block keywords
      if (matchedRule.blockKeywords) {
        for (const keyword of matchedRule.blockKeywords) {
          if (subjectLower.includes(keyword.toLowerCase())) {
            logAction("BLOCKED", `Keyword in rule: ${matchedRule.description}`, {
              from, to, subject, keyword
            });
            message.setReject(`Blocked by rule keyword: ${keyword}`);
            return;
          }
        }
      }
      
      // Check rule-specific forward keywords (only forward if keyword present)
      if (matchedRule.forwardKeywords && matchedRule.forwardKeywords.length > 0) {
        let hasKeyword = false;
        for (const keyword of matchedRule.forwardKeywords) {
          if (subjectLower.includes(keyword.toLowerCase())) {
            hasKeyword = true;
            break;
          }
        }
        
        if (!hasKeyword) {
          logAction("DROPPED", `No forward keyword match: ${matchedRule.description}`, {
            from, to, subject,
            requiredKeywords: matchedRule.forwardKeywords
          });
          message.setReject("Does not match forward keyword criteria");
          return;
        }
      }
      
      // Forward to recipient(s)
      const recipients = matchedRule.recipients || [CONFIG.defaultRecipient];
      
      // Validate recipients
      if (!recipients || recipients.length === 0) {
        console.error("No valid recipients found");
        message.setReject("No recipients configured");
        return;
      }
      
      logAction("FORWARDED", `Rule: ${matchedRule.description}`, {
        from, to, subject, recipients
      });
      
      // Forward to all recipients
      for (const recipient of recipients) {
        if (!recipient || !recipient.includes("@")) {
          console.error(`Invalid recipient address: ${recipient}`);
          continue;
        }
        await message.forward(recipient);
      }
      
      const processingTime = Date.now() - startTime;
      if (CONFIG.enableLogging) {
        console.log(`Processing completed in ${processingTime}ms`);
      }
      
    } catch (error) {
      // Log errors
      console.error(`[${new Date().toISOString()}] ERROR:`, {
        error: error.message,
        stack: error.stack,
        from: message.from,
        to: message.to
      });
      
      // Reject the message on error (prevents silent failures)
      message.setReject("Internal processing error");
    }
  }
};

// Helper function to find matching routing rule
function findMatchingRule(message, from, to, subject) {
  const subjectLower = subject.toLowerCase();
  
  for (const rule of CONFIG.routingRules) {
    let matches = true;
    
    // Check 'to' field (recipient address)
    if (rule.to && to !== rule.to) {
      matches = false;
    }
    
    // Check 'from' field (can be partial match with @domain.com)
    if (rule.from) {
      if (rule.from.startsWith("@")) {
        // Domain match
        if (!from.toLowerCase().endsWith(rule.from.toLowerCase())) {
          matches = false;
        }
      } else {
        // Exact match
        if (from.toLowerCase() !== rule.from.toLowerCase()) {
          matches = false;
        }
      }
    }
    
    // Check 'subject' field (partial match)
    if (rule.subject) {
      if (!subjectLower.includes(rule.subject.toLowerCase())) {
        matches = false;
      }
    }
    
    if (matches) {
      return rule;
    }
  }
  
  return null;
}

// Helper function for consistent logging
function logAction(action, reason, details) {
  if (!CONFIG.enableLogging) return;
  
  // Always log blocks and errors, optionally log forwards
  if (action === "BLOCKED" || action === "ERROR" || CONFIG.logAllEmails) {
    console.log(`[${new Date().toISOString()}] ${action}: ${reason}`, details);
  }
}
```



