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


![Floorp with Sidebery addon, using vertical tabs](/assets/img/posts/FloorpVerticaltabs1.gif)


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

## Firefox Addons Installed

- [AdNauseam](https://addons.mozilla.org/en-US/firefox/addon/adnauseam/)
> Blocks ads & obfuscates browsing data by “clicking” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance. 

- [Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)
> Creating dark themes for websites on the fly. 

- [Sidebery](https://addons.mozilla.org/en-US/firefox/addon/sidebery/)
> Vertical tabs tree and bookmarks in sidebar.

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

- [AudioContext Fingerprint Defender](https://addons.mozilla.org/en-US/firefox/addon/audioctx-fingerprint-defender/)
> Defending against AudioContext fingerprinting by reporting a fake value.

- [Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)
> Automatically learns to block invisible trackers.

- [Privacy Possum](https://addons.mozilla.org/en-US/firefox/addon/privacy-possum/)
> Reducing and falsifying the data gathered by tracking companies.

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

- [Black & Purple mood](https://addons.mozilla.org/en-US/firefox/addon/black-purple-mood/)
> Simple Black and Purple theme, dark and moody just the way I like it.



* * * 

### Firefox Addons Disabled, but installed

- [JShelter](https://addons.mozilla.org/en-US/firefox/addon/javascript-restrictor/)
> JShelter controls the APIs provided by the browser. 

- [uBO-Scope](https://addons.mozilla.org/en-US/firefox/addon/ubo-scope/)
> A tool to measure your 3rd-party exposure score for web sites you visit.



* * *

## Other Firefox Addons Organized by Category:

* * *

=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  AD BLOCKER

======================

* * *


[AdNauseam](https://addons.mozilla.org/en-US/firefox/addon/adnauseam/)
> Blocks ads & obfuscates browsing data by “clicking” blocked and hidden ads, polluting your data profile and injecting noise into the economic system that drives online surveillance. 

[I don't care about cookies](https://addons.mozilla.org/en-US/firefox/addon/i-dont-care-about-cookies/)
> Get rid of cookie warnings from almost all websites!

[Universal Bypass](https://addons.mozilla.org/en-US/firefox/addon/universal-bypass/)
> Automatically skips annoying link shorteners.

[Skip Redirect](https://addons.mozilla.org/en-US/firefox/addon/skip-redirect/)
> Go to the final url from the intermediary url.

[uBlock Origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)
> An efficient wide-spectrum content blocker.

[Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)
> Automatically learns to block invisible trackers.

[Privacy Possum](https://addons.mozilla.org/en-US/firefox/addon/privacy-possum/)
> Reducing and falsifying the data gathered by tracking companies.



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  CLEAN TEXT

======================

* * *


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

###  ISOLATION

======================

* * *


[Google Container](https://addons.mozilla.org/en-US/firefox/addon/google-container/)
> Isolating your Google identity into a separate browser profile.

[Facebook Container](https://addons.mozilla.org/en-US/firefox/addon/facebook-container/)
> Isolating your Facebook identity into a separate browser profile.

[Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
> Carve out a separate box for each of your online lives.

[First Party Isolation](https://addons.mozilla.org/en-US/firefox/addon/first-party-isolation/)
> FPI works by separating cookies on a per-domain basis.



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  PRIVACY

======================

* * *


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



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  BETTERMENT

======================

* * *


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

[Mailvelope](https://addons.mozilla.org/en-US/firefox/addon/mailvelope/)
> Adds missing encryption and decryption features to the user interface of common webmail providers. 

[PX: Viewport Dimensions](https://addons.mozilla.org/en-US/firefox/addon/px-viewport-dimensions/)
> Displays the viewport dimensions when a browser window is resized.



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  SESSION

======================

* * *


[MySessions](https://addons.mozilla.org/en-US/firefox/addon/my-sessions/)
> Auto-save sessions at any cost. 

[Auto Tab Discard](https://addons.mozilla.org/en-US/firefox/addon/auto-tab-discard/)
> Automatically reduces the memory load of open—but inactive—tabs.

[OneTab](https://addons.mozilla.org/en-US/firefox/addon/onetab/)
> Convert tabs to a list and reduce browser memory.



=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--=--

###  STREAM/VIDEO

======================

* * *


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

###  BEAUTIFICATION

======================

* * *


[Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)
> Creating dark themes for websites on the fly. 

[nightTab](https://addons.mozilla.org/en-US/firefox/addon/nighttab/)
> New tab page accented with a chosen colour.

[Matte Black (Purple)](https://addons.mozilla.org/en-US/firefox/addon/matte-black-purple/)
> A modern dark / Matte Black theme with a purple accent color.



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



