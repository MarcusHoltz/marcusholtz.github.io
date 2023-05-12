---
layout: post
title: "TTY0 Tmux Window Rotate and Cool Terminal Dashboards"
date: 2021-02-10 22:00:00 -0600
categories: Linux
tags: tools tmux dashboard tutorial
image:
  path: /assets/img/header/header--tmux-dashboard.jpg
  alt: Cool Tmux and Terminal Dashboards
---
# Cool Tmux and Terminal Dashboards

![tmux window rotate](/assets/img/posts/tmux--window-rotate.gif){: #cool-image }

These scripts present examples of dashboards and how to make a rotating Tmux windows. A TTY0 screen can be used (or over SSH) to visualize data and rotate through these Tmux windows.

* * * 
## Cool terminal info for your host system
If you need status visuals 24/7, and you want them directly on the TTY of the primary host, then [this repo](https://github.com/MarcusHoltz/tmux-screen-rotate) is just for you!

# Add Tmux
Add Tmux with mouse on, save capabilities, copy, and plugin manager: 

`clear; echo -e "INSTALL TMUX and FRIENDS\n\n"; sleep 1; sudo apt install -y tmux git xsel; sleep 3; clear; echo -e "Addding plugin manager to tmux: \n\n"; sleep 1; git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm; touch ~/.tmux.conf; tmux source-file ~/.tmux.conf; sleep 2; clear; printf 'Now creating the tmux.conf\n\n'; sleep 2; bash -c "echo -e '# 720 no scope pane switch\nset -g mouse on\n\n# List of plugins\n'set -g @plugin \'tmux-plugins/tpm\''\n'set -g @plugin \'tmux-plugins/tmux-sensible\''\n'set -g @plugin \'tmux-plugins/tmux-yank\''\n'set -g @plugin \'tmux-plugins/tmux-resurrect\''\n\n# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)\n'run \'~/.tmux/plugins/tpm/tpm\'' '" > ~/.tmux.conf; sleep 3; tmux source ~/.tmux.conf; clear; printf 'To finish the job, you must open\n__tmux__\n\nand then hit \n__CTRL + b__\n\nthen within 2 seconds hit\n_I_ (capital I)\n ... this will install the plugin manager.\n\n'; sleep 1;`

*Please make sure* you finish adding the plugin manager by opening tmux: `tmux` and then keying in `CTRL + b` (that's lowercase 'b') and then within 2 seconds, key in `I` (that's a capital 'i').

### Add to Tmux config
Now we can add a shortcut to Tmux to launch this script when we hit a designated key.

Open `nano ~/.tmux.conf`


And paste this after the mouse input:
```shell
# Rotate windows on Tmux with capital C to run script
bind-key C run-shell -b 'tmuxrotate -rotate #{session_name}'
```

If you're not patient running it the first time, you may accidently spawn another script. Wait for the first window to rotate (12 seconds).



## Review changes
Currently, we have installed for our local user tmux and tmux plugin manager. Tmux Plugins sensible, yank, and resurrect were installed by starting tmux and using the default prefix (ctrl + b) and then within 2 seconds, pressing `I` (that's a capital 'i').

After that, editing the .tmux.conf file for the local user.

.tmux.conf was edited to be able to initiate rotating through our differnt Tmux windows with the capital `C` character. 


If you have reviewed all the changes, and feel they were accomplished successfully -- we can move on to the fancy stuff.




* * *
## Tmux on tty1 host fancy
###  Tmux autorotate script - rotate that cool terminal info

You can use Tmux to roate between dashboard pannels on your TTY.


To get this up, we need to create a few files. First, tmuxrotate.

Open a text editor with the filename tmuxrotate: 

`nano tmuxrotate`

Open that new text file and place the following in:

```shell
#!/bin/bash
rotate(){
        file=/tmp/mytmux.$session
        if [ -f "$file" ]
        then rm "$file"
        else touch "$file"
             while [ -f "$file" ] && tmux next-window -t "$session"
             do     sleep 12
             done
        fi
}
case $1 in
-rotate)shift
        session=${1?session name}
        rotate ;;
esac
```

and copy it into a PATH that tmux can find:

`sudo mv tmuxrotate /usr/bin/tmuxrotate`

Now, make the file executable:

`sudo chmod +x /usr/bin/tmuxrotate`

* * *


###  Tmux countdown
To make this look snazzy we need a few more files.

Open a text editor with the filename tmuxcountdown: 

`nano tmuxcountdown`

Open that new text file and place the following in:

```shell
#!/bin/bash
seconds=20
start="$(($(date +%s) + $seconds))"
while [ "$start" -ge `date +%s` ]; do
    time="$(( $start - `date +%s` ))"
    printf '%s\r' "$(date -u -d "@$time" +%H:%M:%S)"
done
```

and copy it into a PATH that tmux can find:

`sudo mv tmuxcountdown /usr/bin/tmuxcountdown`

Now, make the file executable:

`sudo chmod +x /usr/bin/tmuxcountdown`



* * *

###  Tmux autostart
This is the file that triggers the whole script shabang. 

> PLEASE REPLACE: username_for_docker_server@192.168.8.31
{: .prompt-info }

The way I have this setup is without SSH keys. So if you have them installed, forget needing `sshpass` ... but if you dont, Let's install: `sudo apt install -y sshpass`

Open a text editor with the filename tmuxautostart: 

`nano tmuxautostart`

Open that new text file and place the following in:

```shell
#!/bin/bash
tmux new-session -d -n btm
tmux send-keys -t :btm "btm -f --autohide_time --hide_table_gap --mem_as_value --network_use_bytes" Enter
tmux new-window -n clock
# alternative to the big figlet lol cat clock is: `tty-clock -s -S -c -b -r -B`
tmux send-keys -t :clock "~/.start-lolcat-clock" Enter
tmux split-window -t :clock -v -l 62%
# apt install ncal
tmux send-keys -t :clock "while true; do clear; cal | lolcat; date | lolcat; sleep 21600; done" Enter
tmux split-window -t :clock -v -l 80%
tmux send-keys -t :clock "while true; do clear; ~/.get-random-quotes; clear; cat /tmp/quote | figlet -f standard -c -w 180 | lolcat; sleep 208; done" Enter
# an alternative to curling for your quotes is to install github.com/mubaris/motivate and just use a local collection
tmux new-window -n weather
tmux send-keys -t :weather "while true; do clear; curl wttr.in/denver; sleep 900; done" Enter
tmux new-window -n lazydocker
sleep 1;
tmux send-keys -t :lazydocker "clear;  echo -e '\n   You have 20 seconds\n         ---\n       hit CTRL-b \n         then\n       capital C\n         ---\n' &&  bash tmuxcountdown; sleep 3 && clear; ssh-keyscan $1 >> ~/.ssh/known_hosts; sshpass -p passwordtotheserver ssh -o StrictHostKeyChecking=no -t username_for_docker_server@192.168.8.31 'lazydocker'" Enter
tmux attach
```

and copy it into a PATH that tmux can find:

`sudo mv tmuxautostart /usr/bin/tmuxautostart`

Now, make the file executable:

`sudo chmod +x /usr/bin/tmuxautostart`



* * *
## All the toys

I have a bunch of extra scripts that will start other panes inside of Tmux autorotate.

You need to install a few files and setup some scripts.

First, install `sudo apt install -y lolcat figlet toilet ncal` to get a few to work.



* * *
### Get Random Quotes script
Open a text editor with the filename .get-random-quotes:

Open that new text file and place the following in:

`nano .get-random-quotes`

Open that new text file and place the following in:

```shell
#!/bin/bash
curl https://api.quotable.io/random | grep -o -P '(?<="content":).*(?=,"tags")' | sed 's/,"author":/\n  -- /g' > ${TMPDIR:-/tmp}/quote
clear
```

Save and quit.

Now, make the file executable:

`chmod +x .get-random-quotes`



* * *
### LOLcat clock script
Open a text editor with the filename .start-lolcat-clock:

`nano .start-lolcat-clock`

```shell
#!/bin/bash
watch -t -c -n 1 'echo " $(date +"%I:%M.%S") " | 'figlet' -c -k -W -w 180 -f standard' | /usr/games/lolcat -f -F .005 -p .15 -a -d 5 -s 10
```

Save and quit.

Now, make the file executable:

`chmod +x .start-lolcat-clock`



* * *
### Start the application 

You can set an alias to `bash /usr/bin/tmuxautostart`

Add your alias to your .bashrc:

`nano ~/.bashrc`

```
alias tmuxrotate='bash /usr/bin/tmuxautostart'
```

Save and quit.





* * *
## Other Applications

### Cool terminal info for your host system
[A customizable cross-platform graphical process/system monitor for the terminal.](https://github.com/ClementTsang/bottom)

To install latest version:

`curl -s https://api.github.com/repos/ClementTsang/bottom/releases/latest | grep "browser_download_url.*amd64.deb" | cut -d : -f 2,3 | tr -d \" | wget -i - && sudo dpkg -i bottom*.deb && mkdir -p ~/.local; mkdir -p ~/.local/installed && mv bottom*.deb ~/.local/installed`



* * *
### Cool terminal info for docker
[A simple terminal UI for both docker and docker-compose.](https://github.com/jesseduffield/lazydocker)

To install latest version:
`cd ~; wget https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh && sed -i '4s/.*/DIR="${DIR:-"\/usr\/local\/bin"}"/' install_update_linux.sh && sudo chmod +x ~/install_update_linux.sh && sudo ./install_update_linux.sh && mkdir -p ~/.local; mkdir -p ~/.local/installed && mv install_update_linux.sh ~/.local/installed/lazydocker_updater.sh`


#### Quick Breakdown of the command above:
Here is what the One Liner does: 

- Download the script:

`wget https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh`

- modify the first line:

`sed -i '4s/.*/DIR="${DIR:-"\/usr\/local\/bin"}"/' install_update_linux.sh`

```shell
# allow specifying different destination directory
DIR="${DIR:-"/usr/local/bin"}"
```

- then run the script:

`sudo chmod +x ~/install_update_linux.sh && sudo ./install_update_linux.sh && mkdir -p ~/.local/installed && mv install_update_linux.sh ~/.local/installed/lazydocker_updater.sh` 

We first changed the install directory, to allow lazydocker to be accessable by the whole system (e.g. - root, or user 1000). Then moved the install file to a folder archiving everything installed on the system.


* * *
## With hopes to add:
### - [WTF](https://github.com/wtfutil/wtf/)
- - Wtfutil is the personal information dashboard for your terminal, providing at-a-glance access to your very important but infrequently-needed stats and data.

### - [Sampler](https://github.com/sqshq/sampler)
- - Sampler works with shell commands execution, visualization and alerting. Configured with a simple YAML file.


* * *
* * *
## Bonus Points:
> [Create some new Tmux windows to rotate through, add some screensavers, and an aquarium.](https://blog.holtzweb.com/posts/terminal-screensavers/)
{: .prompt-info }
