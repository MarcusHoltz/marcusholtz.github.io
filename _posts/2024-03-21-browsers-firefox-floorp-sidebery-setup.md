---
layout: post
title: Browser Addons, Sidebery, and Floorp settings
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


![Floorp with Sidebery addon, using vertical tabs](/assets/img/posts/FloorpVerticaltabs1.gif){: #cool-image }


* * * 

## File and folder convention

Sidebery wants to use the `~/Downloads` folder for it's use. Stick with the default folder.

When making the selection to use the default `~/Downloads/Sidebery` folder.

The change is to include the hostname of the client machine along with the profile name.


* * * 

### Example naming convention

1. **Path with filename**: 

`~/Downloads/Sidebery/BuddyPC.default/buddypc.default-2024.03.21-19.14.json`

`~/Downloads/Sidebery/Thunktop.default-release/thunktop.default-release-2023.11.31-08.15.json`

> (Seconds were excluded from the filename.)


* * * 

## Saving increments

1. **Auto-snapshots interval**: `10 minutes`

2. **Snapshots limit**: `8 days`



* * * 

## Syncthing sync snapshots

Once the snapshots are saved to a location on disk, they can be synced across devices. 

Syncing my tabs allows me to take a look back at what I was doing and import that to another browser, or profile in a different workstation. 



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

## Floorp custom changes


The following images demonstrate some of the customizations done to the default Floorp install.


[![Floorp general settings - part 1](/assets/img/posts/FloorpGeneral1_thumb.png)](/assets/img/posts/FloorpGeneral1.png)

* * *

[![Floorp general settings - part 2](/assets/img/posts/FloorpGeneral2_thumb.png)](/assets/img/posts/FloorpGeneral2.png)

* * *

[![Floorp design settings - part 1](/assets/img/posts/FloorpDesign1_thumb.png)](/assets/img/posts/FloorpDesign1.png)

* * *

[![Floorp design settings - part 2](/assets/img/posts/FloorpDesign2_thumb.png)](/assets/img/posts/FloorpDesign2.png)

* * *



## Firefox Addons Installed

- [AdNauseam](https://addons.mozilla.org/en-US/firefox/addon/adnauseam/)
> Blocks ads & obfuscates browsing data by “clicking” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance. 


Creating dark themes for websites on the fly. 

- [Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)


Vertical tabs tree and bookmarks in sidebar.

- [Sidebery](https://addons.mozilla.org/en-US/firefox/addon/sidebery/)


Open tabs, websites, and links in automatically managed disposable containers which isolate the data websites store.

- [Temporary Containers](https://addons.mozilla.org/en-US/firefox/addon/temporary-containers/)


Isolating your Facebook identity into a separate browser profile.

- [Facebook Container](https://addons.mozilla.org/en-US/firefox/addon/facebook-container/)


Carve out a separate box for each of your online lives.

- [Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)


Provides container searching, deleting, modifying, duplicating, and URLs used -- on a per-container or bulk basis. 

- [Containers Helper](https://addons.mozilla.org/en-US/firefox/addon/containers-helper/)


Use FastForward to skip annoying URL "shorteners".

- [FastForward](https://addons.mozilla.org/en-US/firefox/addon/fastforwardteam/)


Sync your bookmarks across browsers via Nextcloud, WebDAV or Google Drive.

- [floccus](https://addons.mozilla.org/en-US/firefox/addon/floccus/)


Get rid of cookie warnings from almost all websites!

- [I still don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/istilldontcareaboutcookies/)


New tab page accented with a chosen colour.

- [nightTab](https://addons.mozilla.org/en-US/firefox/addon/nighttab/)


Convert tabs to a list and reduce browser memory.

- [OneTab](https://addons.mozilla.org/en-US/firefox/addon/onetab/)


Alters some JS APIs to prevent fingerprinting.

- [CanvasBlocker](https://addons.mozilla.org/en-US/firefox/addon/canvasblocker/)


Defending against AudioContext fingerprinting by reporting a fake value.

- [AudioContext Fingerprint Defender](https://addons.mozilla.org/en-US/firefox/addon/audioctx-fingerprint-defender/)


Automatically learns to block invisible trackers.

- [Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)


Reducing and falsifying the data gathered by tracking companies.

- [Privacy Possum](https://addons.mozilla.org/en-US/firefox/addon/privacy-possum/)


Enables a password-protected bookmark folder.

- [Private Bookmarks](https://addons.mozilla.org/en-US/firefox/addon/webext-private-bookmarks/)


Automatically redirects to user-defined urls on certain pages

- [Redirector](https://addons.mozilla.org/en-US/firefox/addon/redirector/)


Improves YouTube playback performance by using your desired codecs.

- [refined-h264ify](https://addons.mozilla.org/en-US/firefox/addon/refined-h264ify/)


An extension for controlling requests.

- [Request Control](https://addons.mozilla.org/en-US/firefox/addon/requestcontrol/)


Text to Speech on Firefox. It use OS build-in TTS engines, you can use it even when offline.

- [TTSFox](https://addons.mozilla.org/en-US/firefox/addon/ttsfox/)


Adds Ecosia.org as the default search engine to your browser.

- [Ecosia](https://addons.mozilla.org/en-US/firefox/addon/ecosia-the-green-search/)


Adds DuckDuckGo Lite as a search engine for a more lean web-search experience.

- [DuckDuckGo Lite Search](https://addons.mozilla.org/en-US/firefox/addon/ddg-lite-search-provider/)


Simple Black and Purple theme, dark and moody just the way I like it.

- [Black & Purple mood](https://addons.mozilla.org/en-US/firefox/addon/black-purple-mood/)



* * * 

### Firefox Addons Disabled, but installed


JShelter controls the APIs provided by the browser. 
[JShelter](https://addons.mozilla.org/en-US/firefox/addon/javascript-restrictor/)

A tool to measure your 3rd-party exposure score for web sites you visit.
[uBO-Scope](https://addons.mozilla.org/en-US/firefox/addon/ubo-scope/)



* * *

## Other Firefox Addons Organized by Category:

=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  AD BLOCKER
======================
Blocks ads & obfuscates browsing data by “clicking” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance. 
[AdNauseam](https://addons.mozilla.org/en-US/firefox/addon/adnauseam/)

Get rid of cookie warnings from almost all websites!
[I don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/i-dont-care-about-cookies/)

Automatically skips annoying link shorteners.
[Universal Bypass](https://addons.mozilla.org/en-US/firefox/addon/universal-bypass/)

Go to the final url from the intermediary url.
[Skip Redirect](https://addons.mozilla.org/en-US/firefox/addon/skip-redirect/)

An efficient wide-spectrum content blocker.
[uBlock Origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)

Automatically learns to block invisible trackers.
[Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)

Reducing and falsifying the data gathered by tracking companies.
[Privacy Possum](https://addons.mozilla.org/en-US/firefox/addon/privacy-possum/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  CLEAN TEXT
======================
Copy Plain Text without any formatting.
[Copy PlainText](https://addons.mozilla.org/en-US/firefox/addon/copy-plaintext/)

Removes all garbage from URLs. 
[Pure URL](https://addons.mozilla.org/en-US/firefox/addon/pure-url/)

Remove garbage from URLs.
[Neat URL](https://addons.mozilla.org/en-US/firefox/addon/neat-url/)

Removes tracking elements from URLs.
[ClearURLS](https://addons.mozilla.org/en-US/firefox/addon/clearurls/)

Create, edit and delete a cookie for the current tab
[Cookie-Editor](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  ISOLATION
======================
Isolating your Google identity into a separate browser profile.
[Google Container](https://addons.mozilla.org/en-US/firefox/addon/google-container/)

Isolating your Facebook identity into a separate browser profile.
[Facebook Container](https://addons.mozilla.org/en-US/firefox/addon/facebook-container/)

Carve out a separate box for each of your online lives.
[Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)

FPI works by separating cookies on a per-domain basis.
[First Party Isolation](https://addons.mozilla.org/en-US/firefox/addon/first-party-isolation/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  PRIVACY
======================
Bypass paywalls for selected sites.
[Bypass Paywalls](https://github.com/iamadamdev/bypass-paywalls-chrome/blob/master/README.md)

Guard your browser against CSS Exfil attacks.
[CSS Exfil Protection](https://addons.mozilla.org/en-US/firefox/addon/css-exfil-protection/)

Emulates Content Delivery Networks.
[LocalCDN](https://addons.mozilla.org/en-US/firefox/addon/localcdn-fork-of-decentraleyes/)

Removes Google's link-conversion/tracking feature. 
[Don't track me Google](https://addons.mozilla.org/en-US/firefox/addon/dont-track-me-google1/)

Search anonymously on Google while staying logged in.
[Searchonymous2](https://addons.mozilla.org/en-US/firefox/addon/searchonymous2/)

Reports a random fake value for fonts installed. 
[Font Fingerprint Defender](https://addons.mozilla.org/en-US/firefox/addon/font-fingerprint-defender/)

Defending against WebGL fingerprinting by reporting a fake value.
[WebGL Fingerprint Defender](https://addons.mozilla.org/en-US/firefox/addon/webgl-fingerprint-defender/)

An email Tracker Blocker.
[Trocker](https://addons.mozilla.org/en-US/firefox/addon/trockerapp/)

Enables a password-protected bookmark folder.
[Private Bookmarks](https://addons.mozilla.org/en-US/firefox/addon/webext-private-bookmarks/)

Changes HTTP protocol to HTTPS, and if loading encounters error, reverts it back to HTTP.
[Smart HTTPS](https://addons.mozilla.org/en-US/firefox/addon/smart-https-revived/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  BETTERMENT
======================
Define custom search engines and use it from address bar of the browser. 
[Custom Search Engine](https://addons.mozilla.org/en-US/firefox/addon/custom-search-engine/)

Press F12 to open hackbar. Ctrl + Enter to execute.
[HackBar V2](https://addons.mozilla.org/en-US/firefox/addon/hackbar-free/)

Capture and save web pages and screenshots from your browser to Joplin. 
[Joplin Web Clipper](https://addons.mozilla.org/en-US/firefox/addon/joplin-web-clipper/)

Suite of tools to enhance your Reddit browsing experience.
[Reddit Enhancement Suite](https://addons.mozilla.org/en-US/firefox/addon/reddit-enhancement-suite/)

Fixes Facebook annoyances.
[Social Fixer](https://addons.mozilla.org/en-US/firefox/addon/socialfixer/)

Adds missing encryption and decryption features to the user interface of common webmail providers. 
[Mailvelope](https://addons.mozilla.org/en-US/firefox/addon/mailvelope/)

Displays the viewport dimensions when a browser window is resized.
[PX: Viewport Dimensions](https://addons.mozilla.org/en-US/firefox/addon/px-viewport-dimensions/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  SESSION
======================
Auto-save sessions at any cost. 
[MySessions](https://addons.mozilla.org/en-US/firefox/addon/my-sessions/)

Automatically reduces the memory load of open—but inactive—tabs.
[Auto Tab Discard](https://addons.mozilla.org/en-US/firefox/addon/auto-tab-discard/)

Convert tabs to a list and reduce browser memory.
[OneTab](https://addons.mozilla.org/en-US/firefox/addon/onetab/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  STREAM/VIDEO
======================
YouTube Video blocker.
[BlockTube](https://addons.mozilla.org/en-US/firefox/addon/blocktube/)

Adds stuff to YouTube.
[Improve YouTube!](https://addons.mozilla.org/en-US/firefox/addon/youtube-addon/)

Skip over sponsors, intros, outros, subscription reminders.
[SponsorBlock](https://addons.mozilla.org/en-US/firefox/addon/sponsorblock/)

Makes YouTube stream H.264 videos instead of VP8/VP9 videos.
[h264ify](https://addons.mozilla.org/en-US/firefox/addon/h264ify/)

Alternate player of live broadcasts for Twitch.tv website.
[Alternate Player for Twitch.tv](https://addons.mozilla.org/en-US/firefox/addon/twitch_5/)

Enhances Twitch with new features, emotes, and more.
[BetterTTV](https://addons.mozilla.org/en-US/firefox/addon/betterttv/)



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--
###  BEAUTIFICATION
======================
Creating dark themes for websites on the fly. 
[Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)

New tab page accented with a chosen colour.
[nightTab](https://addons.mozilla.org/en-US/firefox/addon/nighttab/)

A modern dark / Matte Black theme with a purple accent color.
[Matte Black (Purple)](https://addons.mozilla.org/en-US/firefox/addon/matte-black-purple/)



* * *

## Additional Web Browsers and Addons:


## Basilisk
------------------------------------------------
### Install
You have to download a zip from their website and then extract it to where you'd like the application 'installed' to. 

#### Addons for Basilisk Install
Port of the Firefox (Quantum) default theme, "Photon", to Pale Moon.
[Photonic/](https://addons.basilisk-browser.org/addon/photonic/)

Localizes large third parties used for content delivery. 
[Decentraleyes/](https://addons.basilisk-browser.org/addon/decentraleyes/)

Removes annoying cookie warnings.
[I don't care about cookies](https://addons.basilisk-browser.org/addon/i-dont-care-about-cookies/)

Automatically strips garbage URL tracking parameters.
[Pure URL](https://addons.basilisk-browser.org/addon/pureurl4pm/)

Modifies web pages to use a dark color scheme.
[Swarth](https://addons.basilisk-browser.org/addon/swarth/)

Efficient ad blocker add-on.
[uBlock Origin (Legacy)](https://github.com/gorhill/uBlock-for-firefox-legacy/releases)

ηMatrix will break the majority of the websites you visit.
[ηMatrix](https://addons.basilisk-browser.org/addon/ematrix/)


* * *

## Chromium
### Install De-Googled Chromium


#### Addons
Simply turn your New Tab page into a clean space to take notes. 
[New Tab Draft](https://chrome.google.com/webstore/detail/new-tab-draft/nmfjkeiebceinkbggliapgfdjphocpdh)

PixelBlock is a Gmail extension that blocks people from tracking when you open their emails.
[PixelBlock](https://chrome.google.com/webstore/detail/pixelblock/jmpmfcjnflbcoidlgapblgpgbilinlem)

Prevent the RAM consumption of unused pages. 
[The Marvellous Suspender](https://chrome.google.com/webstore/detail/the-marvellous-suspender/noogafoofpebimajpfpamcfhoaifemoa)

Fusion of tabs manager with a tree like personal information organizer.
[Tabs Outliner](https://chrome.google.com/webstore/detail/tabs-outliner/eggkanocgddhmamlbiijnphhppkpkmkl)

You can add, delete, edit, search, protect and block cookies!
[EditThisCookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg)

Global hotkeys for online music players including support for media keys.
[streamkeys](https://chrome.google.com/webstore/detail/streamkeys/ekpipjofdicppbepocohdlgenahaneen)

Displays the inner width and innner height of the screen.
[Width and Height Display](https://chrome.google.com/webstore/detail/width-and-height-display/hhcddohiohbojnfdmfpbbhiaompeiemo)

Easily undo closed tabs via your browser's toolbar popup!
[Undo Closed Tabs Button](https://chrome.google.com/webstore/detail/undo-closed-tabs-button/ieehkmoiljghfkejgahoheemdjpdinml)

