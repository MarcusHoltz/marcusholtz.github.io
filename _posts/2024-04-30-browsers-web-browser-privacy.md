---
layout: post
title: Web Browser Privacy Presentation
date: 2024-04-30 11:33:00 -0700
categories: [Browsers, Privacy]
tags: [desktop, workstation, browsers, addons, customization, privacy, firefox, brave, security, techtip]
pin: false
image:
  path: /assets/img/header/header--browsers--browser--privacy-presentation.jpg
  alt: Web Browser Privacy
---

# Web Browser Privacy 😎

This presentation attempts to help take back control for the average user and make the web a little more bearable.

* * *

## 🔗 Links to Presentation 📄

* * *

Browser Privacy presentation is available in many different formats. 

👋 **My recomendation is to watch the 📺 video, it'll play and cycle through the slides at a reasonable pace.**


* * *

### Video

- 🎥 [https://is.gd/browserprivacy](https://is.gd/browserprivacy) ⭐


* * *

### Single HTML document

- 🤓 [Link to presentation on Github](/assets/html/browsers-presentation.html)


* * * 

### Powerpoint Slides

- 🖊 [https://is.gd/browserprivacyslides](https://is.gd/browserprivacyslides)



* * *

### Quick markdown export with missing content

- [Just below is a poorly exported text only version](https://blog.holtzweb.com/posts/browsers-web-browser-privacy/#who) **(NOT RECOMENDED)**


* * *

* * *

Again, preferentially, please use one of the choices above for this presentation: 

- [Video](https://is.gd/browserprivacy)

- [HTML](/assets/html/browsers-presentation.html)

- [Powerpoint](https://is.gd/browserprivacyslides)

* * *

* * * 

Below is an export of my powerpoint slides as text, just markdown. 

If you want a mostly plain text based copy -- I would recommend the [HTML](/assets/html/browsers-presentation.html) version of this presentation, as it is text with images. 



* * * 

⚠ ⚠ ⚠ ⚠ ⚠ 

# BROWSERS

⚠ ⚠ ⚠ ⚠ ⚠ 

Marcus Holtz on Web Browser Privacy



## WHO?

I am just a guy. I do not develop a browser.

This is just me, giving my experiences to you.



## WHY?

Are some browsers good? Are some browsers bad?

Nope.

Browsers are not a moral quandary.



## WHAT?

This talk most specifically is about the omnipresent tracking that occurs on the world wide web, with the web being primarily contained to your web browser. 

Please do keep in mind, this talk is specific to the web browser.

Tracking occurs in tandem, outside of the web browser.

Also, the goal here is making the web a little more bearable to navigate and to take back control for the average user.



## SO, HOW TO BEGIN THIS CONVERSATION?

Asking for a friend, why would any individual care about their online privacy?

Is this different than security?

Both keep me safe, right?

Well, what is privacy?



## PRIVACY AND SECURITY - SAME THING?

Source: https://libertytools.io/privacy

Avast as a company is not focused on privacy, they care about security which a lot of the time is not the same.



## What about all the other software I use?

Sorry, you’re right. You’re just as buggered there too.

Here are a few resources for more information about telemetry in applications:

https://github.com/beatcracker/toptout

https://github.com/pluja/awesome-privacy

https://codeberg.org/teaserbot-labs/delightful-humane-design

The goal of this talk is to put you in control.

Understand what data is collected by the tools you use and decide if you want to share it.

Then use methods provided here to opt-in or opt-out.



## Browsers – What Do Browsers Say When They Are Opened?

Source: https://www.scss.tcd.ie/Doug.Leith/pubs/browser_privacy.pdf

Let’s start with…

Opening the application

What happens regarding our privacy if we simply …

1. Open the application.

2. Visit our favorite website.



## Browsers – What Do Browsers Say When They Are Opened?

Source: https://www.scss.tcd.ie/Doug.Leith/pubs/browser_privacy.pdf


### Google Chrome

The start page for Chrome is displayed and a batch of network connections are made, inspection of the content of these connections indicates a device id value is sent in a call to accounts.google.com The URL is now pasted (not typed) into the browser top bar. This generates a request to www.google.com/complete/search with the URL details passed as a parameter.

Also two identifier-like quantities (psi and sugkey). The sugkey value is likely an identifier tied to Chrome itself rather than particular instances of it. The psi value behaves differently however and changes between fresh restarts, it therefore can act as an identifier of an instance of Chrome.

This behavior is reproducible across multiple fresh installs and indicates that user browsing history is by default communicated to Google.

The browser was then closed and reopened. Amongst the connections are some requests that contain data that appear to be persistent identifiers.

One is a request to accounts.google.com/ListAccounts which transmits a cookie that was set during the call to accounts.google.com on initial startup, this cookie acts as a persistent identifier of the browser instance and since is set by the server changing values can 
potentially be linked together by the server.


### Mozilla Firefox

During startup Firefox three identifiers are transmitted to Mozilla: impression id and client id values are sent to incoming.telemetry.mozilla.org, a uaid value sent to Firefox by push.services.mozilla.com via a web socket and echoed back in subsequent web socket messages sent to push.services.mozilla.com 

These three values change between fresh installs of Firefox but persist across browser restarts.

Once startup was complete, the URL was pasted into the browser top bar. This generates no extraneous connections.

The browser was then closed and reopened. Closure results in transmission of data to incoming.telemetry.mozilla.org by a helper ping sender process.

In summary, there appear to be a four identifiers used in the communication with push.services.mozilla.com and incoming.telemetry.mozilla.org, these values also persists across browser restarts.


### Brave

During startup no persistent identifiers are transmitted by Brave.

Calls to go-updater.brave.com contain a sessionid value, similarly to calls to update.googleapis.com in

Chrome, but with Brave this value changes between requests.

Coarse telemetry is transmitted by Brave, and is sent without any identifiers attached.

Once startup was complete, the URL was pasted into the browser top bar.

This generates no extraneous connections.

The browser was then closed and reopened. No data is transmitted on close.

On reopen a subset of the initial startup connections are made but once again no persistent identifiers are transmitted.

In summary, we do not find Brave making any use of identifiers allowing tracking by backend servers of IP address over time, and no sharing of the details of web pages visited with backend servers.



## Browsers – What about _______ browser?

Source: https://privacytests.org

- Pale Moon – uses Goanna instead of Mozilla's Quantum. This makes it a single-process application.

- GNU IceCat – includes additional security features and the GNU LibreJS plugin.

- SeaMonkey – 2006 fork of Firefox, maintaining the XUL plugin architecture.

- Librewolf – modern Firefox fork with modified defaults.

- Brave – Chromium based browser with ad blocking on default.

- Microsoft Edge – Cross platform Chromium based browser.

- Opera – Owned by the communist people’s republic of China.

- Vivaldi – Poweruser and feature rich Chromium based web browser.

- ungoogled-chromium – removing Google components, blobs, and dependency on Google web services.


## What about mobile browsers?

Sorry, that’s a whole other bag.

Here’s a resource for more information:

https://madaidans-insecurities.github.io



## Browsers – Google Chrome Manifest V3

Source: https://developer.chrome.com/docs/extensions/develop/migrate/mv2-deprecation-timeline

Why not discuss Chrome or Chromium as a privacy friendly browser?

Chrome’s API will no longer allow Manifest V2 extensions.

This means total changes to the world's most popular web browser.

Your favorite extensions may stop working at any time.

Google has a stronghold on the web formats that everyone uses.

Let’s discuss Firefox, it’s derivatives, and Brave as alternatives to Chrome.


* * *

## Browsers – Alternative Browsers

During this talk you will be given tools to mitigate the omnipresent tracking that occurs on the world wide web.


### Web Browsing Guide for better Privacy

Locking Down Desktop Browsers:
- Firefox forks
- Chromium forks

## What can I do to prevent my data from being leaked across the browsable internet?

Browsers – What privacy tricks can be used?


### Browsers – Privacy Techniques


#### Privacy through obscurity

- Change as little as possible, only add what is needed.

- Blend in as much as you can.

- Avoid fingerprinting.



##### Anonymity through obfuscation

- Overwhelm the signal with noise.

- Blends anonymity with intervention.

- Confuses trackers as to one's real interests.




## Privacy – Firefox: not great at obscurity


### Privacy through obscurity

All of the different add-ons one can install and preference modifications made to Firefox are inputs that can potentially be used to identify and track you.

Herein lies the catch-22:

Considering the default settings of Firefox are not the best choice for a privacy respecting browser.

The more browser add-ons you install and settings you modify, the more likely you will stand out from the crowd and be easier to track.



#### Firefox obscurity plugins

- uBlock Origin ✔

  - Setup your blocking mode

  - Enable AdGuard URL Tracking Protection

  - Import Actually Legitimate URL Shortener Tool

- Smart Referer ✔

- Skip Redirect ✔

- CanvasBlocker ✔



## Privacy – Brave … Privacy through obscurity

So what addons do I use with Brave?

- NONE

Use Brave as a vanilla browser...

Install and done.

Try to avoid adding more - it differentiates you. Use the same install as thousands of other users.

### Brave automatic obscurity

#### Privacy – Brave privacy through obscurity

Source: https://github.com/brave/brave-browser/wiki/Fingerprinting-Protections

“Brave includes two types of fingerprinting protections, (i) blocking, removing or modifying APIs, to make Brave instances look as similar as possible, and (ii) randomizing values from APIs, to prevent cross session and site linking (e.g. making Brave instances look different to websites each time).”

“Most tools try to make as many browsers look identical as possible … Brave’s system for protecting users against fingerprinting works differently. Instead of trying to make Brave users look identical …, Brave tries to make you look as different as possible, for each website, for each session. This prevents browsers from identifying you when you visit other sites, or when you return to the same site in the future.”


## Privacy – Ad profile obfuscation

### Anonymity through obfuscation

Source: http://ceur-ws.org/Vol-1873/IWPE17_paper_23.pdf

Suppose you check-out (3) books from the library, two on gardening, one on woodworking. The librarian knows something about your interests. The Facebook 'like' button works the same way. Suppose instead you took out every book in the library, and only read the ones you are interested in. The librarian (or anyone with access to the library's records) now cannot tell which books you read, or if you read any of them.

There's no opting out of the surveillance, but there is a way to resist it, to deny those doing the
surveillance any meaningful, valuable, or direct data points.

### Explaining anonymity through obfuscation

Think of it like this - the tracking companies have some information about you – maybe you have a Facebook account, a store card, public records, some unblocked ads on your mobile device, the few trackers that made it past your blockers, etc. That information is valuable companies collect it into data products which they sell on the basis of having some predictive power.

Not giving them more information is what you're trying to do now. You can't get them to forget what they already know about you from various data markets and aggregators. These trade hugely diverse sources of information. Did you register to vote, do you live somewhere with high property values, did your smartphone pass a sensor on a trash can, when, etc.

Suppose you gave them a ton of useless information instead. Accurate data points about you are now drowned in noise. They can filter the noise, but the confidence interval goes down. The value of the data product is diminished. Maybe they also filter out some accurate thing they had gleaned about you. You go from clicking two ads a year to thousands a day. Completely useless new data points every day which they have to store, clean, process, exclude, etc. It would be cheaper to just exclude you from the data product, you bring the average accuracy of their profiles down and cause them work.


## Privacy – Firefox obfuscation - install AdNauseam

Anonymity through obfuscation with ….

AdNauseam 👀

(https://addons.mozilla.org/en-US/firefox/addon/adnauseam)

AdNauseam not only blocks ads, it obfuscates browsing data to resist tracking by ads.

To throw ad networks off your trail AdNauseam “clicks” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance.

The interactive AdVault allows you to visualize and explore the ads that AdNauseam has captured.

On Firefox, you can install AdNauseam with the extensions store in browser.

Note: Firefox’s built in protections wont play well with AdNauseam: disable ‘privacy.trackingprotection.enabled’ in about:config


## AdNauseam on Chromium based browsers

Google has banned AdNauseam from its web store.
Follow these instructions to install it anyway.

Download the latest adnauseam.chromium.zip file from releases in AdNauseam’s GitHub.

(https://github.com/dhowe/AdNauseam)

Extract the zip file to a folder where it can remain after install.

Warning: Do not delete this folder after install or the extension will be disabled

- In the Chrome menu, click Windows > Extensions -- or type chrome://extensions/ in the address bar.

- Make sure the 'Developer Mode' checkbox is ticked

- Click 'Load unpacked extension' and go to the folder from step 2.

- Make sure you select the folder with the name 'adnauseam.chromium' (without a version number)

- CONGRATULATIONS! You have successfully installed AdNauseam.


### You can find the FAQ here.

(https://github.com/dhowe/AdNauseam/wiki/FAQ)

Quick guide on the interface and the per site switches.

(https://github.com/gorhill/uBlock/wiki/Per-site-switches)



## Browsers – Specialty privacy browsers

If you’d like a browser custom to your needs -- you may be building a custom browser.

And if we have to change it, we might as well make it custom to fit our needs.


### Fix Firefox – Firefox obscurity flaws

The default settings of Firefox are not the best choice for a privacy respecting browser.

Many projects prefer to fork Firefox for their needs, for example…


## Best in Class Obscurity: Tor Browser

Tor: gold standard for obscurity

Tor Browser uses the Tor network to protect your privacy and anonymity.

Using the Tor network has two main properties:

- Your internet service provider, and anyone watching your connection locally, will not be able to track your internet activity, including the names and addresses of the websites you visit.

- The operators of the websites and services that you use, and anyone watching them, will see a connection coming from the Tor network instead of your real Internet (IP) address, and will not know who you are unless you explicitly identify yourself.

- In addition, Tor Browser is designed to prevent websites from "fingerprinting" or identifying you based on your browser configuration.



## Fix Firefox – Custom Firefox 

Tor doesn’t leave much room for customization.

How can we customize Firefox to meet our privacy goals?

The default settings of Firefox are not the best choice to be a privacy respecting browser. We can manually modify them to meet our needs.

- Use Firefox Profilemaker to adjust the settings. (https://ffprofile.com)

- An alternative is to download the hardened Arkenfox's user.js – Place this in your Firefox's user.js directory and it’ll fix your browser so everything wont load correctly anymore and settings aren’t the defaults you’re used to. (https://github.com/arkenfox/user.js)

- Or try the different settings on the fly with the Privacy Settings plugin in the toolbar. (https://addons.mozilla.org/en-US/firefox/addon/privacy-settings)

- You can also do it manually (but even that author archived his tutorial in favor of arkenfox). (https://chrisx.xyz/blog/yet-another-firefox-hardening-guide)


## Fix Firefox – Firefox Manual Hardening

Custom Browser – Firefox forks… is it that easy?


### Custom Browser – Introducing a few Firefox forks

Source: https://jm42.github.io/compare-user.js

Today we’re going to cover two different projects that both seek to enhance the privacy of Mozilla’s Firefox browser.

- Customizations stem from the user.js file

  - Betterfox
  - Arkenfox

Browsers have been built around each of these projects that offer different variants for different needs.

- As much privacy as possible - Arkenfox
  - (https://github.com/arkenfox/user.js)

- Privacy without the breakage - Betterfox
  - (https://github.com/yokoffing/Betterfox)


## Custom Browser – Librewolf fork uses arkenfox user.js

Akenfox requires some customization to remove some of the stricter features you may not want.

These features will help you be more private, but you probably don't want them, as they can be breaking.

The Wiki page is required reading. Go to the overrides [common] to make fixes to breaking stuff.

Arkenfox is basically what LibreWolf is built on.

LibreWolf also includes some changes like default DuckDuckGo and uBlockOrigin.

LibreWolf also lets you add checkboxes to a lot of the deeper Arkenfox settings that break a lot of websites.

It also includes the help to these settings, so you can understand what you're doing.

Librewolf has an overrides file. If you have your own custom settings you dont have to worry about updates overwriting your config.

Lots of quality of life improvements overtime.

Do the benefits out weigh the pain? You decide.



## Custom Browser – Firefox forks user.js

Betterfox – This is what you want to pick if you don't want to deal with anything being weird, or websites breaking. Arkenfox is the upstream project for Betterfox.

Betterfox is included in Floorp, with several variations.

Floorp is based on the ESR release of Firefox, meaning Floorp is updated atleast every 4 weeks.

Any settings that are commented out in user.js come with examples.

If you dig through the config you will find details to everything.

You may not be aware of all the settings out there, and a set like this will help you discover them.

Most of the preferences in this will reduce footprint, and disable some features that could be footguns, but at the same time disable some optimizations that aim to reduce cognitive load on users.

Floorp’s GitHub page has "common overwrites“ for more information.



## Custom Browser – Japanese built, Floorp

Floorp key features:

- Strong Tracking Protection: Floorp offers robust tracking protection, safeguarding users from malicious tracking and fingerprinting on the web.

- Flexible Layout: Customize Floorp's layout to your heart's content, including moving the tab bar, hiding the title bar, and more for a personalized browsing experience.

- Switchable Design: Choose from five distinct designs for the Floorp interface, and even switch between OS-specific designs for a unique look

- Regular Updates: Based on Firefox ESR, Floorp receives updates every four weeks, ensuring up-to-date security even before Firefox's releases.

- No User Tracking: Floorp prioritizes user privacy by abstaining from collecting personal information, tracking users, or selling user data, with no affiliations with advertising companies.

- Dual Sidebar: Floorp features a versatile built-in sidebar for webpanels and browsing tools, making it perfect for multitasking and quick access to bookmarks, history, and websites.

- Flexible Toolbar & Tab Bar: Customize your browser with Tree Style Tabs, vertical tabs, and bookmark bar modifications, catering to both beginners and experts in customization.


## Custom Browser – Betterfox explained

Floorp Defaults - By default, Floorp includes a robust tracking blocker, protecting users from a variety of malicious trackers lurking on the web.

Additionally, it provides fingerprinting protection.


### Betterfox settings

- Securefox - Provide sensible security, privacy, and protect user data.

- Default - All the essentials. None of the breakage.

- Fastfox – Priority: speedy browsing. Increase Firefox's browsing speed.

- Peskyfox - Remove annoyances & provide a clean, distraction-free browsing experience.

- Smoothfox – Better scrolling with Microsoft Edge-like smooth scrolling.



## Custom Browser – Tab bar style

- Custom Browser – Vertical tabs
  - Addon Used: https://addons.mozilla.org/en-US/firefox/addon/sidebery


## Custom Browser – Firefox with those design gains.

Source: https://github.com/black7375/Firefox-UI-Fix


Accept no substitutes, except:
WaveFox for more Firefox theming.
(https://github.com/QNetITQ/WaveFox)


## Custom Browser – with uBlock Origin

Source: https://github.com/yokoffing/filterlists#guidelines

Betterfox also has a list of recommended filters for uBlock Origin to help fill in the gaps of the overall browsing experience.

* * *
My personal lists are included below:
```
https://cdn.statically.io/gh/dhowe/AdNauseam/master/filters/adnauseam.txt
https://easylist.to/easylist/easyprivacy.txt
https://secure.fanboy.co.nz/fanboy-annoyance.txt
https://easylist.to/easylist/easylist.txt
https://secure.fanboy.co.nz/fanboy-cookiemonster.txt
https://easylist.to/easylist/fanboy-social.txt
https://filters.adtidy.org/extension/ublock/filters/3.txt
https://filters.adtidy.org/extension/ublock/filters/17.txt
https://malware-filter.gitlab.io/malware-filter/urlhaus-filter.txt
https://raw.githubusercontent.com/yourduskquibbles/webannoyances/master/ultralist.txt
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://big.oisd.nl/
https://winhelp2002.mvps.org/hosts.txt
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt
https://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&showintro=0&mimetype=plaintext
https://adaway.org/hosts.txt
https://raw.githubusercontent.com/DandelionSprout/adfilt/master/AnnoyancesList
```

## Custom Browser – Firefox forks, Floorp, in detail: Recap

Just give it a try, most features are disabled by default so youneed to turn them on, otherwise it's just Firefox + Sidebar.

List of Floorp features:

- Vertical tab + Collapse ✔

- Sidebar ✔

- Change keyboard shortcuts ✔

- Workspace ✔

- Sleeping Tab ✔

- Profile Switcher ✔

- Tab Tiling ✔

If you want to find out more about a setting, use:

`about:about`
(this works in any Firefox browser)

Floorp in relation to Firefox is the same as Vivaldi in relation to Chrome
They are forks aimed at power users with a native support for vertical tabs.


## Don’t take my word for it, here’s everyone else using Floorp!

Source: https://github.com/yokoffing/Betterfox?tab=readme-ov-file#browser-integration

The Gecko Version of Midori uses Floorp.

Source: https://news.itsfoss.com/midori-11

FireDragon is essentially a custom Floorp fork with the gradient KDE Sweet theme and Beautyline icons.
Source: https://forum.garudalinux.org/t/new-firedragon-major-version/34585

Other Projects Using Floorp:

- Waterfox (Sep 2023)

- Pulse (Dec 2021)

- Ghostery Private Browser (Feb 2021)


## Custom Browser – Firefox - Multi-Account-Containers

A Note on Qubes OS:

If you want to surf privately on Qubes OS, the qubes are using Firefox ESR out-of-box.
The Firefox they ship provides no privacy benefit, other than containerization.

So let’s containerize our Firefox!

Using:

- Multi-Account-Containers
by Mozilla

- Temporary Containers
by stoically


## Custom Browser – Firefox - Multi-Account-Containers

What are Containers and how can they help?

Source: https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers

Source: https://addons.mozilla.org/en-GB/firefox/addon/temporary-containers

Containers are a tab/process isolation mechanism in order to separate each new tab/window from each other. This means each Tab gets it’s own resources.

Isolating cookies inside of containers prevents other sites from being able to access them, thus increase both privacy and security in theory.

You can see each tab or group with a different color and icon.

The real power is when you use Multi-Account-Containers for websites you frequent and Temporary Containers for everything else.


Also, side note: Brave will never get Multi-Account-Containers

Source: https://community.brave.com/t/equivalent-of-multi-account-containers-or-temporary-containers-extension-ff/135573/41?page=3



### Custom Browser – Firefox - Unlimited Containers

Source: https://chefkochblog.wordpress.com/2018/04/03/firefox-container-guide/

Some settings for Temporary Containers

- General:
  - Automatic Mode: On
  - Notifications when Temporary Containers are deleted: Off
  - Container Number: Reuse available numbers
  - Delete no longer needed Temporary Containers: After the last tab in it closes

- Isolation > Per Domain:
  - Always open in new Temporary Container: Disabled

- All other settings on this page should be set to “Use Global”.

- Isolation > Global:
  - All settings on this page should be set to: “Different from Tab Domain & Subdomains”.


## Custom Browser – 99.99% is never 100%

You will still have to deal with the occasional error.

Floorp may be the easiest set and forget setup, with little to no intervention needed, but even then

99% perfect still leaves room for 1% of problems.

Knowing your tooling, setup, and configuration can help.

Why is ESR bad?

https://www.mozilla.org/en-US/security/advisories/mfsa2023-40

https://www.pcworld.com/article/2089208/firefox-118-0-1-chrome-0-day-vulnerability-also-affects-firefox.html



## Custom Browsers? – Floorp That. Firefox Ride or Die.

So, just for you – I made a script that sets up a new Firefox profile, just like Floorp.

You can find it here:

https://github.com/MarcusHoltz/Firefox


## That was a lot of information.

Isnt there an easier way, something I can just install and forget - no extra steps.

### Browsers – Brave!

So what addons do I use?

None. I use Brave as my vanilla browser...

Changes will make you appear unique to trackers.

Again, please choose what works best for your needs.


## Browsers – How can I keep track of what opens in what?

How do I manage workflow? Open specific links with respective profiles.

Per Client/URL basis

This solution offers a program to replace the default browser.

Decide which app/profile to open based on the domain or a keyword in the URL.

https://github.com/amirkarimi/browser-hub


## Browsers – Search Engines

Source: https://libertytools.io/privacy

Your search engine does tracking as well…. And watch out for manufactured results.

Let’s discuss some alternate options available.

Bing searches with tracking removed: https://www.ecosia.org



## Browsers – Few Search Engines

Let’s discuss how few options are available.

Source: https://www.searchenginemap.com



## Browsers – Anything else? What? Why do I need a VPN?

Like Tor, a VPN can help with privacy.

Use the same IP address as many other users, hoping to gain a layer of obscurity.

Riseup offers Personal VPN service for censorship circumvention, location anonymization and traffic encryption.

This is a FREE VPN service with comparable speeds to most paid VPN services.

The cost for RiseupVPN to provide this service is approximately $60 USD per person per year.
https://riseup.net/en/vpn



## Be careful

If you're not careful, you can inadvertently reveal way too much information about yourself to websites, companies, and even the web browser makers themselves.

### Websites that Provide Privacy Tests

- coveryourtracks.eff.org

- www.first-party.site

- browserleaks.com

- pbtest.org

- amiunique.org

- arkenfox.github.io/TZP/tzp.html

- github.com/arthuredelstein/privacytests.org/tree/master/scripts

- Fingerprint testing:

- xsinator.com

- panopticlick.eff.org

- www.deviceinfo.me

- ipleak.net



## Privacy – Concerns with who now?

### Privacy – Concerns outside of a browser and search?

Source: https://www.webfx.com/blog/internet/what-are-data-brokers-and-what-is-your-data-worth-infographic/

Surveillance capitalism raises significant concerns about privacy, transparency, fairness, security, and market competition.

Your information is conglomerated from a variety of sources, including social media, telecommunications companies, public records, commercial sources, or simply USB mouse drivers.

These firms then sell that as raw data or an enriched analysis with inferences based on different pseudonymous identifiers.


## Browsers – Privacy Techniques

### WHEN?

### When should anyone even care about privacy?

Source: https://en.wikipedia.org/wiki/Enshittification

Do we even gain anything meaningful or tangible for all of this effort?

Isnt this, like, the same amount of effort and change that reposting some political meme on social media does?


## Maintaining your privacy helps prevent the misuse or abuse of this data by companies or third parties.

### > Lock your car, don’t temp the thieves. <


## Credits

Marcus Holtz
https://www.holtzweb.com

THANKS FOR
JOINING ME

* * *
No Attribution Needed - ShareAlike
NO CC BY-SA NEEDED
