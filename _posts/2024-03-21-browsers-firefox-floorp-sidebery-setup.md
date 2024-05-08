---
layout: post
title: Browser addons, Sidebery, and Floorp settings
date: 2024-03-21 11:33:00 -0700
categories: [Browsers, Sidebery]
tags: [desktop, workstation, browsers, addons, customization, floorp, firefox, sidebery, techtip]
pin: false
image:
  path: /assets/img/header/header--browsers--addons-and-sidebery.jpg
  alt: Addons for Gecko based browsers
---

# Sidebery addon setup inside Floorp browser

I use the Sidebery addon with the Floorp browser as my primary means of browsing. Sidebery brings with it some extra features that need some customization. 


![Floorp with Sidebery addon, using vertical tabs](/assets/img/posts/FloorpVerticaltabs1.gif)


* * *

## Using Sidebery tabs as bookmarks

Sidebery can go from `tabs --> bookmarks --> tabs`.

> You can use your [bookmark sync](https://floccus.org/) to sync Sidebery tabs between devices as well.


![Floorp with Sidebery addon, saving current tabs as bookmarks and then back again](/assets/img/posts/FloorpSideberyTabs2Bookmarks.gif)



* * * 

## Setting up Sidebery Snapshots

* * * 

Sidebery also lets you save your current tabs to disk in JSON and MD format, at a specific time interval or with manual creation. 

> These files can then be synced between devices for a self-hosted cloud history/tab sync between devices.


* * * 

### File and folders location

To setup Sidebery's Snapshots: 

1. Open the `Sidebery Settings` page.

2. Scroll to the `Snapshots` section.

3. Under, `Auto export (on every snapshot)` select `on`.

4. Find the text box in the area labeled `Path with file name`.

5. Edit the `text box` for the Path with file name to match the Example naming below.


* * * 

### Example naming

To provide uniqueness to each browser being synced: 

- Include the `hostname` of the client machine along with the `profile name`.

1. You will need to edit the `text box` using the method for syncing above.

2. Sidebery wants to use the `~/Downloads` folder for it's use. Stick with this default folder.

3. Be sure to add a `sub-folder` inside the default `Sidebery` folder representing your `hostname`.

4. When ready, edit `Path with filename` using the examples below: 

`~/Downloads/Sidebery/BuddyPC.default/buddypc.default--%Y.%M.%D-%h.%m`

`~/Downloads/Sidebery/Thunktop.default-release/thunktop.default-release--%Y.%M.%D-%h.%m`

> Include the `hostname` of the client machine along with the `profile name` (seconds were excluded from the filename).


* * * 

## Saving increments

How big did you want your directory for each browser to get before you started to rotate out the files? 

How many files until syncthing starts to get clogged up by this share, or the inodes fill up on the NAS?

8 days of storage  
       @    
10 min intervals
       =    
1152 files per browser


* * *

### Setting auto-snapshots saving interval and limit

1. In the settings, under Snapshots, find `Auto-snapshot interval`. 
> This will determine how often a file is generated with all the tabs in the browser.

2. Set the `Auto-snapshots interval` to `10 minutes`.

3. Down, at the end, set the `Snapshot limit` to `days`.

4. And set the `Snapshots limit` to `8 days`.
> This will determine how many snapshots are kept around before they are removed and replaced by newer snapshots.


* * * 

## Syncthing sync snapshots

Once the snapshots are saved to a location on disk, they can be synced across devices. 

Syncing my tabs allows me to take a look back at what I was doing and import that to another browser or profile in a different workstation. 



### Syncthing Diagram

```
      S y n c t h i n g         
    ┌────────┐  ┌───────────┐   
    │ Floorp │  │           │   
    │ +addon │  │  N A S    │   
    │Sidebery│  │ (remote)  │   
    │   on   │  └───────────┘   
    │   PC   │     ▲            
    └────────┴─────┘            
```

Syncthing keeps all of the tabs for each workstation and browser profile the same across all my devices. 


* * *

## Save tabs, multiple ways

If you require a 64-bit version of all 9's, 99.9999999999999999 (that's 10 billion years)...


### Tab Session Manager Addon

- You can also use [Tab Session Manager](https://addons.mozilla.org/en-US/firefox/addon/tab-session-manager/) to backup your browsing history and tab sessions in the Downloads folder.

- And you can have these [sync](https://syncthing.net/) to your VPS, NAS, FTP, as well. 


* * *

### Firefox Sync

- But if you want to use Firefox's native syncing, you can do that too and [host your own sync server](https://github.com/mozilla-services/syncserver).

  - Self-hosting is not a requirement. Mozilla's own syncing service is free. But It is NOT recommended to use if you have if you have more than 30-100 containers. Mozilla enforces limits on the [amount of data](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/storage/sync#storage_quotas_for_sync_data) each extension is allowed to store in the sync area.


- If you're using Firefox's Sync Client you can also use the [ffsclient](https://github.com/Mikescher/firefox-sync-client) commandline-utility to list/view/edit/delete entries in a firefox-sync account.

  - [ffsclient](https://github.com/Mikescher/firefox-sync-client) can be used to access bookmarks, passwords, forms, tabs, history or custom data.


* * *

## Vertical Tabs from scratch 

If you want the slow-motion self-bake version of vertical tabs, check out [Sidetabs](https://addons.mozilla.org/en-US/firefox/addon/sidetabs/) by Jeb Nicholson.

Otherwise, you can continue to the Floorp customization below to enable vertical tabs:


* * * 

## Floorp vertical tabs custom changes


The following images demonstrate some of the customizations done to the default Floorp install.

(click thumbnails below for full, larger image)

* * * 


[![Floorp general settings - part 1](/assets/img/posts/FloorpGeneral1_thumb.png)](/assets/img/posts/FloorpGeneral1.png)

Floorp General Settings - 1/2

* * *

[![Floorp general settings - part 2](/assets/img/posts/FloorpGeneral2_thumb.png)](/assets/img/posts/FloorpGeneral2.png)

Floorp General Settings - 2/2

* * *

[![Floorp design settings - part 1](/assets/img/posts/FloorpDesign1_thumb.png)](/assets/img/posts/FloorpDesign1.png)

Floorp Design Settings - 1/2

* * *

[![Floorp design settings - part 2](/assets/img/posts/FloorpDesign2_thumb.png)](/assets/img/posts/FloorpDesign2.png)

Floorp Design Settings - 2/2

* * *


## Sidebery CSS changes

Here are the color styles I use for my Sidebery:

```css
#root.root {--toolbar-bg: #161923     ;}
#root.root {--frame-bg: #1A1B26 ;}
```


* * *

## uBlock Origin addon

[uBlock Origin](https://ublockorigin.com/) is an open-source, cross-platform browser extension for content filtering — primarily aimed at neutralizing privacy invasion in an efficient, user-friendly method.

> uBlock Origin requires lists to operate.


* * *

### uBlock Origin lists

There are different types of filter lists that people use across the internet.

The lists I use are specific for Ublock Origin.


#### uBlock Origin with Betterfox's Recommended Filter lists

There is **no better guide** than [`yokoffing`'s `Betterfox` wiki for filterlists](https://github.com/yokoffing/filterlists).


* * *

##### Yokoffing's Guidelines

1) Prevent overblocking by applying the law of [diminishing returns](https://web.archive.org/web/20230719033252/https://pmctraining.com/site/wp-content/uploads/2018/04/Law-of-Diminishing-Returns-CHART.png) (always blocking more ≠ better blocking experience).

2) Aim for [efficiency](https://brave.com/the-mounting-cost-of-stale-ad-blocking-rules/) without sacrificing quality (use sane, quality resources).

3) Implement the [minimum](https://reddit.com/r/uBlockOrigin/wiki/index#wiki_which_filter_lists_should_i_select.3F) number of useful lists (avoid redundancy and bloat when possible).

> [Subscribe uBlock Origin to Yokoffing's lists](https://github.com/yokoffing/filterlists?tab=readme-ov-file#privacy) (use the 'subscribe' link for each list) 


#### My personal block lists

My block lists imported into uBlock Origin are below - but

Be **sure** you have **read** and **understand** [`yokoffing`'s `Betterfox` wiki for filterlists](https://github.com/yokoffing/filterlists) first!

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

* * *

#### Import all of the mentioned uBlock Origin lists

For quick import, you can use my entire uBlock Origin config - it has been exported and uploaded for your convenience.

- [Download my entire uBlock Origin config](/assets/json/ublock.json) 👍


* * *

## Multi-Account-Containers addon

What are Containers and how can they help?

> Containers are a tab/process isolation mechanism in order to separate each new tab/window from each other.
> This means each Tab gets it’s own resources.

The **real power** is when you use [`Multi-Account-Containers`](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers) for websites you frequent and [`Temporary Containers`](https://addons.mozilla.org/en-US/firefox/addon/temporary-containers/) for everything else.


* * * 

## Temporary Containers addon

**Unlimited Containers** are now at our disposal with [Temporary Containers](https://addons.mozilla.org/en-US/firefox/addon/temporary-containers/).



### Settings for Temporary Containers

To make the Temporary Containers addon more useful, try and change the defaults to the following:

* * *

In the `General` section:

- Automatic Mode: `On`

- Notifications when Temporary Containers are deleted: `Off`

- Container Number: `Reuse available numbers`

- Delete no longer needed Temporary Containers: `After the last tab in it closes`

* * *

Under the `Isolation` tab, find `Global`:

- All settings on this page should be set to: `“Different from Tab Domain & Subdomains”`

* * *

In the `Isolation` section, look for `Per Domain`:

- Always open in new Temporary Container: `Disabled`

- All other settings on this page should be set to `“Use Global”`

* * *


#### Import the above mentioned Temporary Containers config

For quick import, you can use my Temporary Containers config - it has been exported and uploaded for your convenience.

- [Download the Temporary Containers config](/assets/json/temporary_containers.json) 👍


* * *

> For more information about [`Multi-Account-Containers`](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers) and [`Temporary Containers`](https://addons.mozilla.org/en-US/firefox/addon/temporary-containers/) visit the [Firefox Container Guide](https://chefkochblog.wordpress.com/2018/04/03/firefox-container-guide/).


* * *

## Auto Tab Discard: Options

In the Auto Tab Discard plugin, I find these options to work the best for me:

- Discard inactive tabs after `137` minutes (zero to disable) when the number of inactive tabs exceeds `8`

- `checkbox` Change favicon of discarded tabs

- `checkbox` Do not discard a tab if it can display desktop notifications

- Tabs with the following hostnames or regular expression rules are not being discarded:
  - `pastebin.com, galeapps.gale.com, gale.udemy.com, dash.cloudflare.com, one.dash.cloudflare.com, mattermost.sofree.us, jsfiddle.net, my.nocix.net`

- `checkbox` Store YouTube's timestamp before discarding

- `uncheck` Open FAQs page on updates


* * *

### Import the Auto Tab Discard config from above

For quick import, you can use the Auto Tab Discard config below - it has been exported and uploaded for your convenience.

- [Download the Auto Tab Discard config](/assets/json/auto_tab_discard.json) 👍


* * *

## CanvasBlocker: 	Settings

CanvasBlocker is a great plugin, but it can quickly ruin your browsing experience. I would recomend avoiding `Stealth settings` or `Maximum protection`. 

- To run the auto-configure wizard: 
  - `Settings > Presets > Open`

- If you run the Wizard, choose:
  - `Convenient settings` and `reCAPTCHA exception`

- Adding, `Stealth settings` or `Maximum protection` will slow down the browser. 
  - Choose the presets based on your needs. 



* * *

### Import the CanvasBlocker config from above

For quick import, you can use the CanvasBlocker config below - it has been exported and uploaded for your convenience.

- [Download this CanvasBlocker config](/assets/json/canvas_blocker.json) 👍


* * *

## Firefox Addons I Keep Installed

- [AdNauseam](https://addons.mozilla.org/en-US/firefox/addon/adnauseam/)
> Blocks ads & obfuscates browsing data by “clicking” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance (comes packaged with uBlock Origin). 

- [Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)
> Creating dark themes for websites on the fly. 

- [Sidebery](https://addons.mozilla.org/en-US/firefox/addon/sidebery/)
> Vertical tabs tree and bookmarks in sidebar.

- [Auto Tab Discard](https://addons.mozilla.org/en-US/firefox/addon/auto-tab-discard/)
> Automatically discard inactive tabs to free up resources

- [Temporary Containers](https://addons.mozilla.org/en-US/firefox/addon/temporary-containers/)
> Open tabs, websites, and links in automatically managed disposable containers which isolate the data websites store.

- [Facebook Container](https://addons.mozilla.org/en-US/firefox/addon/facebook-container/)
> Isolating your Facebook identity into a separate browser profile.

- [Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
> Carve out a separate box for each of your online lives.

- [Containers Helper](https://addons.mozilla.org/en-US/firefox/addon/containers-helper/)
> Provides container searching, deleting, modifying, duplicating, and URLs used -- on a per-container or bulk basis. 

- [FastForward](https://addons.mozilla.org/en-US/firefox/addon/fastforwardteam/)
> Use FastForward to skip annoying URL "shorteners".

- [floccus](https://addons.mozilla.org/en-US/firefox/addon/floccus/)
> Sync your bookmarks across browsers via Nextcloud, WebDAV or Google Drive.

- [I still don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/istilldontcareaboutcookies/)
> Get rid of cookie warnings from almost all websites!

- [nightTab](https://addons.mozilla.org/en-US/firefox/addon/nighttab/)
> New tab page accented with a chosen colour.

- [OneTab](https://addons.mozilla.org/en-US/firefox/addon/onetab/)
> Convert tabs to a list and reduce browser memory.

- [CanvasBlocker](https://addons.mozilla.org/en-US/firefox/addon/canvasblocker/)
> Alters some JS APIs to prevent fingerprinting.

- [Private Bookmarks](https://addons.mozilla.org/en-US/firefox/addon/webext-private-bookmarks/)
> Enables a password-protected bookmark folder.

- [Redirector](https://addons.mozilla.org/en-US/firefox/addon/redirector/)
> Automatically redirects to user-defined urls on certain pages

- [refined-h264ify](https://addons.mozilla.org/en-US/firefox/addon/refined-h264ify/)
> Improves YouTube playback performance by using your desired codecs.

- [Request Control](https://addons.mozilla.org/en-US/firefox/addon/requestcontrol/)
> An extension for controlling requests.

- [TTSFox](https://addons.mozilla.org/en-US/firefox/addon/ttsfox/)
> Text to Speech on Firefox. It use OS build-in TTS engines, you can use it even when offline.

- [Ecosia](https://addons.mozilla.org/en-US/firefox/addon/ecosia-the-green-search/)
> Adds Ecosia.org as the default search engine to your browser.

- [DuckDuckGo Lite Search](https://addons.mozilla.org/en-US/firefox/addon/ddg-lite-search-provider/)
> Adds DuckDuckGo Lite as a search engine for a more lean web-search experience.



* * * 

### Firefox Addons Disabled, but installed

- [JShelter](https://addons.mozilla.org/en-US/firefox/addon/javascript-restrictor/)
> JShelter controls the APIs provided by the browser. 

- [uBO-Scope](https://addons.mozilla.org/en-US/firefox/addon/ubo-scope/)
> A tool to measure your 3rd-party exposure score for web sites you visit.


* * *

## Script to Install Vertical Tabs, Betterfox, and Addons

The script below will turn a vanilla Firefox profile into one that resembles my Floorp setup.

All the `plugins` mentioned above will be installed along with the `user.js` and `userChrome.css` changes.

> You will still need to configure `uBlock Origin` and all `Multi-Account` and `Temporary Containers`, as [mentioned above](https://blog.holtzweb.com/posts/browsers-firefox-floorp-sidebery-setup/#ublock-origin-addon).


* * *

### Firefox New Profile Creation Script

You can find the [Firefox New Profile Creation Script](https://raw.githubusercontent.com/MarcusHoltz/Firefox/main/ffNewProfile.sh) in 👆 [this repository](https://github.com/MarcusHoltz/Firefox/). 👆

> You will need to download the script and then run it.


![Firefox Profile Install Script](/assets/img/posts/firefox_profile_install.gif)





* * *


* * *

## Other Firefox Addons Organized by Category:

* * *

=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  AD BLOCKER


[AdNauseam](https://addons.mozilla.org/en-US/firefox/addon/adnauseam/)
> Blocks ads & obfuscates browsing data by “clicking” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance. 

[I don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/i-dont-care-about-cookies/)
> Get rid of cookie warnings from almost all websites!

[Universal Bypass](https://addons.mozilla.org/en-US/firefox/addon/universal-bypass/)
> Automatically skips annoying link shorteners.

[Skip Redirect](https://addons.mozilla.org/en-US/firefox/addon/skip-redirect/)
> Go to the final url from the intermediary url.

[uBlock Origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)
> An efficient wide-spectrum content blocker (comes packaged with AdNauseam).

[Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)
> Automatically learns to block invisible trackers.

[Privacy Possum](https://addons.mozilla.org/en-US/firefox/addon/privacy-possum/)
> Reducing and falsifying the data gathered by tracking companies.



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  CLEAN TEXT


[Copy PlainText](https://addons.mozilla.org/en-US/firefox/addon/copy-plaintext/)
> Copy Plain Text without any formatting.

[Pure URL](https://addons.mozilla.org/en-US/firefox/addon/pure-url/)
> Removes all garbage from URLs. 

[Neat URL](https://addons.mozilla.org/en-US/firefox/addon/neat-url/)
> Remove garbage from URLs.

[ClearURLS](https://addons.mozilla.org/en-US/firefox/addon/clearurls/)
> Removes tracking elements from URLs.

[Cookie-Editor](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor/)
> Create, edit and delete a cookie for the current tab




=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  ISOLATION


[Google Container](https://addons.mozilla.org/en-US/firefox/addon/google-container/)
> Isolating your Google identity into a separate browser profile.

[Facebook Container](https://addons.mozilla.org/en-US/firefox/addon/facebook-container/)
> Isolating your Facebook identity into a separate browser profile.

[Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
> Carve out a separate box for each of your online lives.

[Containerise](https://addons.mozilla.org/en-US/firefox/addon/containerise/)
> Simple and easy way to add rules for a domain or subdomain in a dedicated container.

[Containers Helper](https://addons.mozilla.org/en-US/firefox/addon/containers-helper/)
> Power user container searching, deleting, modifying, duplicating, on a per-container or bulk basis. 

[First Party Isolation](https://addons.mozilla.org/en-US/firefox/addon/first-party-isolation/)
> FPI works by separating cookies on a per-domain basis.

[FoxyProxy Standard](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/)
> Advanced proxy management tool that completely replaces Firefox's limited proxying capabilities.

[Proxy SwitchyOmega](https://addons.mozilla.org/en-US/firefox/addon/switchyomega/)
> Manage and switch between multiple proxies quickly & easily.

[Profile Switcher for Firefox](https://addons.mozilla.org/en-US/firefox/addon/profile-switcher/)
> Create, manage and switch between browser profiles seamlessly.

> NOTE: !!! As of writing, [Profile Switcher is broken](https://addons.mozilla.org/en-CA/firefox/addon/profile-switcher/reviews/?score=1) but [fixes are posted in Github](https://github.com/null-dev/firefox-profile-switcher/issues/106#issuecomment-2008582801).
{: .prompt-warning }

> Also, Profile Switcher needs [additional software](https://github.com/null-dev/firefox-profile-switcher-connector/releases) to function. After installation, it will prompt you based on your OS and Firefox install path.
{: .prompt-info }




=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  PRIVACY


[Bypass Paywalls](https://github.com/iamadamdev/bypass-paywalls-chrome/blob/master/README.md)
> Bypass paywalls for selected sites.

[CSS Exfil Protection](https://addons.mozilla.org/en-US/firefox/addon/css-exfil-protection/)
> Guard your browser against CSS Exfil attacks.

[LocalCDN](https://addons.mozilla.org/en-US/firefox/addon/localcdn-fork-of-decentraleyes/)
> Emulates Content Delivery Networks.

[Don't track me Google](https://addons.mozilla.org/en-US/firefox/addon/dont-track-me-google1/)
> Removes Google's link-conversion/tracking feature. 

[Searchonymous2](https://addons.mozilla.org/en-US/firefox/addon/searchonymous2/)
> Search anonymously on Google while staying logged in.

[Font Fingerprint Defender](https://addons.mozilla.org/en-US/firefox/addon/font-fingerprint-defender/)
> Reports a random fake value for fonts installed. 

[WebGL Fingerprint Defender](https://addons.mozilla.org/en-US/firefox/addon/webgl-fingerprint-defender/)
> Defending against WebGL fingerprinting by reporting a fake value.

[Trocker](https://addons.mozilla.org/en-US/firefox/addon/trockerapp/)
> An email Tracker Blocker.

[Private Bookmarks](https://addons.mozilla.org/en-US/firefox/addon/webext-private-bookmarks/)
> Enables a password-protected bookmark folder.

[Smart HTTPS](https://addons.mozilla.org/en-US/firefox/addon/smart-https-revived/)
> Changes HTTP protocol to HTTPS, and if loading encounters error, reverts it back to HTTP.

[History AutoDelete](https://addons.mozilla.org/en-US/firefox/addon/history-autodelete/)
> Control your history! Auto delete history by date, use, and domain.




=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  BETTERMENT


[Custom Search Engine](https://addons.mozilla.org/en-US/firefox/addon/custom-search-engine/)
> Define custom search engines and use it from address bar of the browser. 

[HackBar V2](https://addons.mozilla.org/en-US/firefox/addon/hackbar-free/)
> Press F12 to open hackbar. Ctrl + Enter to execute.

[Joplin Web Clipper](https://addons.mozilla.org/en-US/firefox/addon/joplin-web-clipper/)
> Capture and save web pages and screenshots from your browser to Joplin. 

[Reddit Enhancement Suite](https://addons.mozilla.org/en-US/firefox/addon/reddit-enhancement-suite/)
> Suite of tools to enhance your Reddit browsing experience.

[Social Fixer](https://addons.mozilla.org/en-US/firefox/addon/socialfixer/)
> Fixes Facebook annoyances.

[Refined GitHub](https://addons.mozilla.org/en-US/firefox/addon/refined-github-/)
> Simplifies the GitHub interface and adds many useful features.

[Mailvelope](https://addons.mozilla.org/en-US/firefox/addon/mailvelope/)
> Adds missing encryption and decryption features to the user interface of common webmail providers. 

[PX: Viewport Dimensions](https://addons.mozilla.org/en-US/firefox/addon/px-viewport-dimensions/)
> Displays the viewport dimensions when a browser window is resized.

[PiHole Browser Extension](https://addons.mozilla.org/en-US/firefox/addon/pihole-browser-extension/)
> Browser extension to quickly control your PiHole.

[DownThemAll!](https://addons.mozilla.org/en-US/firefox/addon/downthemall/)
> Select, queue, sort, rename and run your downloads faster.

[Bookmark search plus 2](https://addons.mozilla.org/en-US/firefox/addon/bookmark-search-plus-2/)
> Search for both bookmarks and folders, displaying the exact location in the bookmark tree.

[Checkmarks](https://addons.mozilla.org/en-US/firefox/addon/checkmarks-web-ext/)
> Importing a new profile? Checkmarks sorts, formats, and loads bookmarks favicons.

[External Application Button](https://addons.mozilla.org/en-US/firefox/addon/external-application/)
> A highly customizable external application button and context-menu items

> Also, External Application Button needs [additional software](https://github.com/andy-portmen/native-client) to function. After installation, it will prompt you based on your OS, and you will need to extract the files to an install path then run the `install` file.
{: .prompt-info }

> This will install NodeJS, and on Windows, change some registry entries. `../node/x64/node install.js`
{: .prompt-warning }




=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  SESSION


[Tab Session Manager](https://addons.mozilla.org/en-US/firefox/addon/tab-session-manager/)
> Auto-save sessions at any cost. 

[Auto Tab Discard](https://addons.mozilla.org/en-US/firefox/addon/auto-tab-discard/)
> Automatically reduces the memory load of open—but inactive—tabs.

[OneTab](https://addons.mozilla.org/en-US/firefox/addon/onetab/)
> Convert tabs to a list and reduce browser memory.

[Copy/Paste and Save tabs list](https://addons.mozilla.org/en-US/firefox/addon/paste-site-list)
> Copy every tab opened, save, and paste a list of urls to open in new tabs (alternative to OneTab).

[Sidebery](https://addons.mozilla.org/en-US/firefox/addon/sidebery/)
> Vertical tabs tree and bookmarks in sidebar.

[Wallabagger](https://addons.mozilla.org/en-US/firefox/addon/wallabagger/)
> Add pages to wallabag, a self-hosted web archive you can read later.

[Readeck](https://addons.mozilla.org/en-US/firefox/addon/readeck/)
> Save the readable content of web pages. See it as a bookmark manager and a read later tool.

[Shiori](https://addons.mozilla.org/en-US/firefox/addon/shiori_ext/)
> Hosted bookmark manager intended as a simple clone of Pocket. 

[Bookmark search plus 2](https://addons.mozilla.org/en-US/firefox/addon/bookmark-search-plus-2/)
> Know what folder that bookmark is in. Search for both bookmarks and folder and find the exact location.

[Save my tabs!](https://addons.mozilla.org/en-US/firefox/addon/save-all-my-tabs/)
> Save only current open tabs as bookmarks in a folder at a designated interval.




=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  STREAM/VIDEO


[BlockTube](https://addons.mozilla.org/en-US/firefox/addon/blocktube/)
> YouTube Video blocker.

[Improve YouTube!](https://addons.mozilla.org/en-US/firefox/addon/youtube-addon/)
> Adds stuff to YouTube.

[SponsorBlock](https://addons.mozilla.org/en-US/firefox/addon/sponsorblock/)
> Skip over sponsors, intros, outros, subscription reminders.

[h264ify](https://addons.mozilla.org/en-US/firefox/addon/h264ify/)
> Makes YouTube stream H.264 videos instead of VP8/VP9 videos.

[Alternate Player for Twitch.tv](https://addons.mozilla.org/en-US/firefox/addon/twitch_5/)
> Alternate player of live broadcasts for Twitch.tv website.

[BetterTTV](https://addons.mozilla.org/en-US/firefox/addon/betterttv/)
> Enhances Twitch with new features, emotes, and more.




=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

* * *

###  BEAUTIFICATION


[Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)
> Creating dark themes for websites on the fly. 

[nightTab](https://addons.mozilla.org/en-US/firefox/addon/nighttab/)
> New tab page accented with a chosen colour.

[Adaptive Tab Bar Color](https://addons.mozilla.org/en-US/firefox/addon/adaptive-tab-bar-colour/)
> Changes the color of Firefox tab bar to match the website theme.

[Matte Black (Purple)](https://addons.mozilla.org/en-US/firefox/addon/matte-black-purple/)
> A modern dark / Matte Black theme with a purple accent color.

[Black & Purple mood](https://addons.mozilla.org/en-US/firefox/addon/black-purple-mood/)
> Simple Black and Purple theme, dark and moody just the way I like it.





* * *

* * *

## Additional Web Browsers and Addons:


## Basilisk

------------------------------------------------

### Install

You have to download a zip from their website and then extract it to where you'd like the application 'installed' to. 


#### Addons for Basilisk

[Photonic](https://addons.basilisk-browser.org/addon/photonic/)
> Port of the Firefox (Quantum) default theme, "Photon", to Pale Moon.

[Decentraleyes](https://addons.basilisk-browser.org/addon/decentraleyes/)
> Localizes large third parties used for content delivery. 

[I don't care about cookies](https://addons.basilisk-browser.org/addon/i-dont-care-about-cookies/)
> Removes annoying cookie warnings.

[Pure URL](https://addons.basilisk-browser.org/addon/pureurl4pm/)
> Automatically strips garbage URL tracking parameters.

[Swarth](https://addons.basilisk-browser.org/addon/swarth/)
> Modifies web pages to use a dark color scheme.

[uBlock Origin (Legacy)](https://github.com/gorhill/uBlock-for-firefox-legacy/releases)
> Efficient ad blocker add-on.

[ηMatrix](https://addons.basilisk-browser.org/addon/ematrix/)
> ηMatrix will break the majority of the websites you visit.


* * *

## Chromium

------------------------------------------------

### Install De-Googled Chromium

Please review and follow any install instructions here: [https://github.com/ungoogled-software/ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium)



#### Addons for Chromium

[New Tab Draft](https://chrome.google.com/webstore/detail/new-tab-draft/nmfjkeiebceinkbggliapgfdjphocpdh)
> Simply turn your New Tab page into a clean space to take notes. 

[PixelBlock](https://chrome.google.com/webstore/detail/pixelblock/jmpmfcjnflbcoidlgapblgpgbilinlem)
> PixelBlock is a Gmail extension that blocks people from tracking when you open their emails.

[The Marvellous Suspender](https://chrome.google.com/webstore/detail/the-marvellous-suspender/noogafoofpebimajpfpamcfhoaifemoa)
> Prevent the RAM consumption of unused pages. 

[Tabs Outliner](https://chrome.google.com/webstore/detail/tabs-outliner/eggkanocgddhmamlbiijnphhppkpkmkl)
> Fusion of tabs manager with a tree like personal information organizer.

[EditThisCookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg)
> You can add, delete, edit, search, protect and block cookies!

[streamkeys](https://chrome.google.com/webstore/detail/streamkeys/ekpipjofdicppbepocohdlgenahaneen)
> Global hotkeys for online music players including support for media keys.

[Width and Height Display](https://chrome.google.com/webstore/detail/width-and-height-display/hhcddohiohbojnfdmfpbbhiaompeiemo)
> Displays the inner width and innner height of the screen.

[Undo Closed Tabs Button](https://chrome.google.com/webstore/detail/undo-closed-tabs-button/ieehkmoiljghfkejgahoheemdjpdinml)
> Easily undo closed tabs via your browser's toolbar popup!



