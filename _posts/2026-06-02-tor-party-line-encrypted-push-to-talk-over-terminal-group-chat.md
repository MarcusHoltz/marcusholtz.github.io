---
layout: post
title: Tor Party Line - Encrypted Push-to-Talk Calls With No Phone Number
description: Encrypted push-to-talk voice over Tor hidden services, with a one-button auto-dial walkie-talkie on your Android homescreen.
date: 2026-07-01 11:33:00 -0700
categories: [Privacy, Tor]
tags: [tor, opensource, privacy, onion, security, script, android, termux, voice, encryption, pushtotalk, walkietalkie, selfhosted, guide]
pin: false
image:
  path: /assets/img/header/header--tor--tor-party-line-terminal-phone.jpg
  alt: Tor Party Line - encrypted push-to-talk voice and group calls over Tor hidden services with one-button Android connect
---

# Encrypted push-to-talk voice calls over Tor with a one-button Android walkie-talkie

Let's make a phone call without a phone. No SIM card, no account, no signup, no server rental, no port forwarding. Just a `.onion` address, a shared secret, and a spacebar.

**Tor Party Line** is a single Bash script that turns any Linux box, Mac, Docker host, or Android phone into an encrypted push-to-talk radio that works over the Tor network. Two people can call each other directly, or one person can host a relay and bridge a whole group onto the same line, like the old telephone party lines, but end-to-end encrypted and hidden inside Tor.

The whole project lives here, and everything in this post points back to it:

**GitLab repo: [https://gitlab.com/marcusholtz/tor-party-line](https://gitlab.com/marcusholtz/tor-party-line)**

And yes, at the end of this post we get to the fun part: a **one-button walkie-talkie on your Android homescreen** that auto-dials the party line. Pressing one button is easier than dialing a phone number. Don't let anyone tell you privacy has to be inconvenient.

## OVERVIEW

What is this project? Tor Party Line is a terminal application for encrypted voice over Tor hidden services:

- **Push-to-talk (PTT)**: hold SPACE, talk, release, and your clip is compressed with Opus, encrypted, and delivered. Half-duplex on purpose: Tor latency makes real-time full-duplex unreliable, but complete clips always arrive intact.
- **End-to-end encryption**: AES-256-CBC with PBKDF2 (100k iterations), plus optional HMAC signing of protocol messages. The only thing both sides need to agree on is a shared secret.
- **No infrastructure**: your `.onion` address is your phone number. Tor hidden services punch through NAT, CGNAT, and firewalls automatically.
- **Group calls**: relay mode forwards encrypted blobs between callers. The relay never sees the shared secret and never hears your audio.
- **Auditable**: the entire thing is one Bash file (`partyline.sh`, about 5,600 lines). No binaries, no telemetry, no network calls except through Tor. You can read every line of what you are running.

Under the hood it wires together tools you already trust: `tor` for routing, `opus-tools` for voice compression (roughly 10x reduction), `openssl` for encryption, and `socat` to shove it all through the Tor SOCKS proxy. It runs on Linux, macOS, Android/Termux, and Docker.

## INSTALL

Grab the script from the repo:

```bash
git clone https://gitlab.com/marcusholtz/tor-party-line.git
cd tor-party-line
chmod +x partyline.sh && ./partyline.sh
```

The script checks for its dependencies on first run and installs the missing ones for you (menu option `9` re-runs this any time). The cast of characters:

| Tool | Purpose |
|------|---------|
| `tor` | Anonymous routing and your `.onion` address |
| `opus-tools` | Voice compression |
| `socat` | Transport through the Tor SOCKS proxy |
| `openssl` | AES-256-CBC encryption and HMAC signing |
| `pulseaudio-utils` / `alsa-utils` | Audio capture and playback on Linux |
| `ffmpeg` / `ffplay` | Audio on macOS and Android |
| `qrencode` | QR code so you can share your address by pointing a camera at it |
| `termux-api` | Microphone access on Android |

Prefer containers? There is a compose file in the repo:

```bash
docker compose run --rm partyline
```

Docker mode manages Tor inside the container and stores keys and config in mounted volumes, so your `.onion` address survives container restarts.

## CONFIG

First run does three things: Tor bootstraps (give it 1-3 minutes), your permanent `.onion` address is generated and displayed, and the menu tells you the next required step in yellow. That step is the shared secret.

**The shared secret is the whole security model.** Both sides must enter the same one, and anyone who has it can join your line, so treat it like a password. Menu option `1` sets it interactively, or from the command line:

```bash
./partyline.sh --secret 'mysecret' --save-secret
```

`--save-secret` persists it encrypted at rest (file mode 600) so you never type it again. In Docker, drop it in a git-ignored file instead:

```bash
echo -n 'mysecret' > secrets/shared_secret.txt
chmod 600 secrets/shared_secret.txt
```

To hand the secret to the other caller, don't paste it into a chat app that keeps history forever. Use a one-time self-destructing link like [yopass](https://share.yopass.se), or better, exchange it in person.

Settings follow a sane precedence: built-in defaults, then `.env` (Docker only), then saved config, then CLI flags. The defaults worth knowing:

| Parameter | Default | Note |
|-----------|---------|------|
| `OPUS_BITRATE` | `16` kbps | Lower it for group calls on slow connections |
| `CIPHER` | `aes-256-cbc` | 21 options including ChaCha20 and Camellia |
| `PTT_TOGGLE_MODE` | `0` | `0` = hold-to-talk, `1` = tap-to-toggle |
| `HMAC_AUTH` | `1` | Signs protocol messages, prevents replay |
| `MAX_PTT_SECONDS` | `120` | Hard cap on a single recording |
| `AUTO_LISTEN` | `0` | Start listening the moment Tor boots |

Every flag is documented in the [repo README](https://gitlab.com/marcusholtz/tor-party-line), including `--exclude-nodes` for skipping Tor nodes in specific countries, `--snowflake` for networks that block Tor, and `--single-hop` if you want speed more than you want server anonymity.

## UP AND RUNNING

Before you call anyone, make sure the audio actually works. Silent calls are almost always a wrong audio device, not a broken install. Menu option `2` walks through mic and speaker setup, and `Menu -> 7 -> Audio devices -> Test all outputs` plays a tone through every output it can find until you hear one. There is also a full diagnostic:

```bash
./partyline.sh test
```

Now the button. Push-to-talk defaults to **hold SPACE to record, release to send**. If you'd rather tap once to start and tap again to stop (this is how it always works on Termux, where holding a key is awkward), flip `PTT_TOGGLE_MODE` to `1` in Settings. Once you are in a call, four keys run the whole show:

| Key | Action |
|-----|--------|
| **SPACE** (hold) | Record, send on release |
| **T** | Send an encrypted text message |
| **S** | Mid-call settings (cipher, PTT mode) |
| **Q** | Hang up |

One gotcha worth knowing up front: if the header shows a red dot next to the cipher, the two sides are using different ciphers and decryption is failing silently. Fix it mid-call with `S`.

## USING: CALL, RECEIVE, RELAY

There are only three verbs to learn.

**Receive.** One side has to be listening. From the menu that is option `4`, or from the shell:

```bash
./partyline.sh listen --secret 'shared-secret'
```

Set `--auto-listen` (or `AUTO_LISTEN=1`) and the script goes straight into listening mode as soon as Tor finishes bootstrapping. That plus a saved secret turns a spare machine into an always-on answering post.

**Call.** The other side dials the listener's `.onion` address. Menu option `5`, or:

```bash
./partyline.sh call abc123.onion --secret 'shared-secret'
```

Menu option `3` displays your address as a QR code, which beats reading 56 base32 characters over the phone you are trying to replace.

**Relay.** For a group, one machine hosts the party line (menu option `6`):

```bash
./partyline.sh relay --secret 'group-secret'
```

Everyone else just calls the relay's onion address with the group secret. The relay forwards encrypted blobs and rate-limits flooders, but it cannot decrypt anything: the secret never touches it. Tor bandwidth is the bottleneck, so 2-3 callers is rock solid, 3-5 is good, and past 10 you are pushing your luck. Dropping the Opus bitrate in Settings buys you headroom.

## ANDROID: THE ONE-BUTTON WALKIE-TALKIE

Here is the payoff. When this is done, there is a single icon on your homescreen. You press it, Tor spins up, the script dials your party line with the stored secret, and you are talking. One button. Easier than dialing a number, easier than unlocking most apps.

We use three apps, all free and open source:

| App | What it does | Where it lives |
|-----|--------------|----------------|
| **Termux** | The Linux terminal the script runs in | [F-Droid](https://f-droid.org/en/packages/com.termux/) |
| **Termux:API** | Gives Termux access to the microphone and other Android hardware | [F-Droid](https://f-droid.org/en/packages/com.termux.api/) |
| **Script Runner for Termux** | Manages scripts and gives them homescreen buttons | [IzzyOnDroid repo](https://apt.izzysoft.de/fdroid/index/apk/com.wilixx.termuxscriptrunner) |

Two things to know before you tap a single install button:

**Get Termux from F-Droid, not the Play Store.** The Play Store build of Termux is deprecated and broken. If you don't have F-Droid yet, install it from [f-droid.org](https://f-droid.org/). Termux and Termux:API must come from the same source or their signatures won't match.

**IzzyOnDroid is not in F-Droid by default.** The [IzzyOnDroid repo](https://apt.izzysoft.de/fdroid/) is a separate, well-respected F-Droid-compatible repository, but the stock F-Droid app does not know about it until you add it. You have two ways in:

1. **Add the repo to F-Droid.** In the F-Droid app go to `Settings -> Repositories`, tap `+`, and paste:

   ```
   https://apt.izzysoft.de/fdroid/repo
   ```

   Pull down to refresh the package index and [IzzyOnDroid](https://apt.izzysoft.de/fdroid/) apps show up in search like anything else.

2. **Or just use Droid-ify.** [Droid-ify](https://droidify.app/) is a cleaner F-Droid client that ships with the [IzzyOnDroid repo](https://apt.izzysoft.de/fdroid/) already built in, flip it on under `Repositories` and you are done. If the F-Droid repo settings dance sounds like a chore, this is the easy road.

THEN you can download **Script Runner for Termux**. We will do that in Step 2, Termux itself comes first.

### Step 1: Prep Termux

Install [Termux](https://f-droid.org/en/packages/com.termux/) and [Termux:API](https://f-droid.org/en/packages/com.termux.api/) from F-Droid. Termux:API has no icon of its own and nothing to open, it is a plugin that sits quietly until Termux needs the microphone. Open Termux.

![Open Termux](/assets/img/posts/tor-party-line-1-open-termux.png){: height="250" }

![Termux opens to a shell](/assets/img/posts/tor-party-line-2-termux-opens.png)

Update the package lists and upgrade whatever is stale:

```bash
pkg update
pkg upgrade
```

![Update Termux package lists](/assets/img/posts/tor-party-line-3-update-termux-package-lists.png)

![Package lists updating](/assets/img/posts/tor-party-line-4-updating-termux-package-lists.png)

![Upgrade Termux packages](/assets/img/posts/tor-party-line-5-upgrade-termux-packages.png)

![Upgrade starts](/assets/img/posts/tor-party-line-6-upgrade-of-termux-packages-starts.png)

If the upgrade asks about config files, the default answer is fine, just keep hitting Enter.

![Package update prompts](/assets/img/posts/tor-party-line-7-termux-package-update-prompts.png)

![More package update prompts](/assets/img/posts/tor-party-line-8-termux-package-update-prompts2.png)

Done here for now. Type `exit` and hit Enter to close the session cleanly.

![Exit Termux](/assets/img/posts/tor-party-line-9-to-exit-termux-type-exit-hit-enter.png)

### Step 2: Install Script Runner for Termux

With the [IzzyOnDroid repo](https://apt.izzysoft.de/fdroid/) enabled (via F-Droid's repository settings or [Droid-ify](https://droidify.app/), see above), search for and install [Script Runner for Termux](https://apt.izzysoft.de/fdroid/index/apk/com.wilixx.termuxscriptrunner). It is a GPL-3.0 bridge app that manages scripts, runs them through Termux, and pins them to your homescreen. If search comes up empty, the repo index has not refreshed yet: pull down to refresh and try again.

![Install Script Runner for Termux from IzzyOnDroid](/assets/img/posts/tor-party-line-10-install-script-runner-for-termux.png)

![Open Script Runner for Termux](/assets/img/posts/tor-party-line-11-open-scriptrunner-for-termux.png)

![Welcome screen](/assets/img/posts/tor-party-line-12-script-runner-for-termux-welcome-screen.png)

The setup wizard hands you two commands to run inside Termux. The first grants storage access, the second lets external apps (Script Runner) start Termux commands. Copy the first one:

```bash
termux-setup-storage
```

![Copy the first setup command](/assets/img/posts/tor-party-line-13-copy-paste-setup-script-runner-for-termux-setup.png)

Switch back to Termux and paste it in:

![Switch back to Termux](/assets/img/posts/tor-party-line-13-copy-paste-setup-script-runner-for-termux-switch-back-to-termux.png)

![Paste the first command into Termux](/assets/img/posts/tor-party-line-14-paste-script-runner-for-termux-text-to-termux.png)

![Run it](/assets/img/posts/tor-party-line-15-paste-script-runner-for-termux-text-to-termux2.png)

Now back to Script Runner for the second command:

![Switch back to Script Runner](/assets/img/posts/tor-party-line-16-switch-back-to-script-runner-for-termux.png)

```bash
mkdir -p ~/.termux && echo 'allow-external-apps=true' >> ~/.termux/termux.properties && termux-reload-settings
```

![Copy the second setup command](/assets/img/posts/tor-party-line-17-copy-second-setup-script-runner-for-termux.png)

![Paste it into Termux](/assets/img/posts/tor-party-line-18-paste-script-runner-for-termux-setup-to-termux.png)

Finish the wizard:

![Exit the setup wizard](/assets/img/posts/tor-party-line-19-exit-script-runner-for-termux-setup.png)

Android will ask you to bless the connection between the two apps. Allow both the Termux permission and the Android-level one:

![Grant the Termux permission](/assets/img/posts/tor-party-line-20-script-runner-for-termux-termux-permissions.png)

![Grant the Android permission](/assets/img/posts/tor-party-line-21-script-runner-for-termux-android-permissions.png)

![Script Runner is installed](/assets/img/posts/tor-party-line-22-script-runner-for-termux-is-installed.png)

### Step 3: Get partyline.sh onto the phone

In Script Runner, create a new script:

![Create a new script](/assets/img/posts/tor-party-line-23-script-runner-for-termux-new-script.png)

Open the raw script in your phone's browser, straight from the repo:

```
https://gitlab.com/marcusholtz/tor-party-line/-/raw/main/partyline.sh
```

Select all, copy:

![Open the raw script in the browser](/assets/img/posts/tor-party-line-24-get-the-script-on-your-phone.png)

![Select all and copy the script](/assets/img/posts/tor-party-line-25-copy-the-script-on-your-phone.png)

Paste the whole thing into Script Runner's editor and name it `partyline.sh`:

![Paste the script into Script Runner](/assets/img/posts/tor-party-line-26-paste-in-the-copied-script-to-script-runner-for-termux.png)

![The script is saved](/assets/img/posts/tor-party-line-27-script-is-now-in-script-runner-for-termux.png)

![Verify it was created](/assets/img/posts/tor-party-line-28-verify-creation-in-script-runner-for-termux.png)

### Step 4: First run, install dependencies, set the secret

Open the script's configuration. Set the Interaction Mode to **None (Instant)** and leave **Background Execution** and **Interactive Session** toggled off. The script draws its own menus inside the Termux session, so it does not need Script Runner to hold the terminal open afterward.

![Edit the partyline.sh entry](/assets/img/posts/tor-party-line-29-script-runner-for-termux-edit-new-partyline.sh.png)

![Interaction mode and toggles](/assets/img/posts/tor-party-line-30-turn-off-script-runner-for-termux-interactive-session.png)

Hit the play button:

![Press play on partyline.sh](/assets/img/posts/tor-party-line-31-use-play-button-on-partyline.sh-in-script-runner-for-termux.png)

![Script Runner has started partyline.sh](/assets/img/posts/tor-party-line-34-script-runner-for-termux-has-started-partyline.sh.png)

First launch detects what is missing and offers to install it. Say yes and let it work:

![Dependency check](/assets/img/posts/tor-party-line-35-partyline.sh-dependencies.png)

![Dependencies installing](/assets/img/posts/tor-party-line-36-partyline.sh-dependencies-are-installing.png)

Then the main screen appears, Tor bootstraps, and your permanent `.onion` address is printed at the top. That address is your phone number now, write it down or share the QR code from menu option `3`.

![The Tor Party Line main screen](/assets/img/posts/tor-party-line-37-partyline.sh-mainscreen.png)

Press `1` and set the shared secret. Both parties must use the same one.

![Set the shared secret](/assets/img/posts/tor-party-line-38-partyline.sh-setup-shared-secret-now.png)

![Ready for action](/assets/img/posts/tor-party-line-39-partyline.sh-ready-for-action.png)

That is the one-time setup done. Press `0` to exit cleanly.

![Type 0 to exit](/assets/img/posts/tor-party-line-39-partyline.sh-type-0-to-exit.png)

![Clean shutdown](/assets/img/posts/tor-party-line-40-partyline.sh-exit-quit-completed.png)

### Step 5: The auto-dial button

Now we turn a menu-driven app into a one-press speed dial. Edit the `partyline.sh` entry in Script Runner again:

![Edit partyline.sh in Script Runner](/assets/img/posts/tor-party-line-41-open-script-runner-for-termux-and-edit-partyline.sh.png)

In the **Arguments** field, tell the script to skip the menu and dial straight out to your party line:

```
call --address youronionaddresshere.onion
```

The secret you saved in Step 4 is loaded from disk automatically, so the arguments stay clean and nothing sensitive lands in a launcher shortcut.

![Set the auto-dial arguments](/assets/img/posts/tor-party-line-42-setup-auto-dial-arguments-for-script-runner-for-termux-partyline.sh.png)

Long-press Script Runner's icon (or use its share/pin option) to drop a widget for `partyline.sh` on the homescreen:

![Pin the script to the desktop](/assets/img/posts/tor-party-line-43-pin-script-runner-for-termux-to-desktop-for-1-button-press-access.png)

![Add to home screen](/assets/img/posts/tor-party-line-44-pin-script-runner-for-termux-to-homescreen.png)

![The new button on the homescreen](/assets/img/posts/tor-party-line-45-new-button-added-to-homescreen.png)

And that is it. Press the button: Tor comes up, the hidden service goes active, and the script dials the onion address with your stored secret.

![Auto-connecting to the onion address](/assets/img/posts/tor-party-line-46-auto-connecting-to-onion-address-with-stored-secret-partyline.sh.png)

![Connected to the Tor Party Line](/assets/img/posts/tor-party-line-47-partyline.sh-connected-to-torpartyline.png)

On Termux, push-to-talk is always tap-to-toggle: tap SPACE to start recording, tap again to send. `T` sends encrypted text, `Q` hangs up.

### Step 6: Keep it alive (always-on walkie-talkie)

Android loves killing background processes, and a dead Termux session is a dead phone line. Three fixes:

1. You already installed the [Termux:API app](https://f-droid.org/en/packages/com.termux.api/) back in Step 1. Now install its command-line half inside Termux: `pkg install termux-api`. The app and the package work as a pair, and together they give the script proper microphone access.
2. When Android asks whether Termux can always run in the background, allow it.
3. In Android Settings -> Apps -> Termux -> Battery, set **Unrestricted**. Script Runner's config screen even nags you about this one.

![Allow background access for an always-on walkie-talkie](/assets/img/posts/tor-party-line-48-termux-api-and-background-access-for-always-on-mobile-walkie-talkie.png)

Worried about the battery and data cost of an always-on Tor voice line? Opus at 16 kbps is tiny. A 10 second message is around 20 KB on the wire. Here is the actual data usage after setup and testing:

![Termux data use with partyline.sh installed](/assets/img/posts/tor-party-line-49-termux-data-use-with-partyline.sh-installed.png)

If you want to see it moving, there are two short clips in the repo: a [terminal capture of the script in action](/assets/img/posts/tor-party-line-50-terminal-thumbail-of-script-in-action.mp4) and a [shakycam recording of a live call](/assets/img/posts/tor-party-line-51-shakycam-recording-of-partyline.sh.mp4).

## A FEW HONEST CAVEATS

- **No forward secrecy.** If a shared secret leaks, every call made with it is exposed. Rotate secrets between sensitive conversations.
- **Latency is real.** This is a walkie-talkie, not a phone call. Clips arrive complete but a few seconds late. That is the price of Tor.
- **The relay is blind but not invisible.** It cannot hear you, but it does know how many callers are connected and when they talk.

Everything else, the full flag reference, the security model, vanity `.onion` address generation, Snowflake for Tor-blocked networks, and the troubleshooting guide, lives in the repo:

**[https://gitlab.com/marcusholtz/tor-party-line](https://gitlab.com/marcusholtz/tor-party-line)**

If push-to-talk voice is not your thing, [OnionShare](https://onionshare.org/) covers anonymous file drops and chat over the same hidden service idea, and a self-hosted [Mumble](https://www.mumble.info/) server behind Tor gets you low-latency group voice if you are willing to run real infrastructure. But for "two people, zero servers, one button on a phone", the party line is hard to beat.
