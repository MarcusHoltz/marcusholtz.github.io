---
layout: post
title: "Terminal Screensavers - Screen saver for your shell"
date: 2022-06-02 22:00:00 -0600
categories: Linux
tags: techtip terminal simple oneliner
pin: true
image:
  path: /assets/img/header/header--terminal-screensaver-asciiquarium.jpg
---

![Asciiquarium](/assets/img/posts/terminal-screensaver--asciiquarium.gif){:.cool-image}


# Single Line / Copy & Paste Screensavers
You can find these, and plenty more at mewbies.com: [Best collection of Terminal Based screensavers](http://mewbies.com/acute_terminal_fun_02_view_ascii_art_ansi_art_and_movies_on_the_terminal.htm)

* * *
## Cursor Bounce

Cursor will bounce off the walls of your terminal. Try resizing your terminal while its in the middle of the terminal.

`x=1;y=1;xd=1;yd=1;while true;do if [[ $x == $LINES || $x == 0 ]]; then xd=$(( $xd *-1 )) ; fi ; if [[ $y == $COLUMNS || $y == 0 ]]; then yd=$(( $yd * -1 )) ; fi ; x=$(( $x + $xd )); y=$(( $y + $yd )); printf "\33[%s;%sH" $x $y; sleep 0.02 ;done `



* * * 
## Rainbow Cursor Worm

Cursor leaves a rainbow trail behind it.

`a=1;x=1;y=1;xd=1;yd=1;while true;do if [[ $x == $LINES || $x == 0 ]]; then xd=$(( $xd *-1 )) ; fi ; if [[ $y == $COLUMNS || $y == 0 ]]; then yd=$(( $yd * -1 )) ; fi ; x=$(( $x + $xd )); y=$(( $y + $yd )); printf "\33[%s;%sH\33[48;5;%sm \33[0m" $x $y $(($a%199+16)) ;a=$(( $a + 1 )) ; sleep 0.001 ;done`



* * *
## Terminal Screen Saver

Full terminal color gradient effects. The modulus operations keep the colors cycling a bit and makes sure it repeats itself after producing some horizontal lines. Probably could be reworked to be a bit better, this is an exercise left to the hacker.

`j=0;a=1;x=1;y=1;xd=1;yd=1;while true;do for i in {1..2000} ; do if [[ $x == $LINES || $x == 0 ]]; then xd=$(( $xd *-1 )) ; fi ; if [[ $y == $COLUMNS || $y == 0 ]]; then yd=$(( $yd * -1 )) ; fi ; x=$(( $x + $xd )); y=$(( $y + $yd )); printf "\33[%s;%sH\33[48;5;%sm . \33[0m" $x $y $(( $a % 8 + 16 + $j % 223 )) ;a=$(( $a + 1 )) ; done ; x=$(( x%$COLUMNS + 1 )) ; j=$(( $j + 8 )) ;done`



* * * 
## Blue Matrix

will output horizontally then scroll your screen down and repeat. 


`while [ 1 -lt 2 ]; do i=0; COL=$((RANDOM%$(tput cols)));ROW=$((RANDOM% $(tput cols)));while [ $i -lt $COL ]; do tput cup $i $ROW;echo -e \
"\033[1;34m" $(cat /dev/urandom | head -1 | cut -c1-1) 2>/dev/null \
; i=$(expr $i + 1); done; done`



* * *
# Downloaded Terminal Screensavers 
* * * 
## pipes.sh

Animated Windows 95 esque pipes terminal screensaver.

`cd /usr/local/bin;wget https://raw.githubusercontent.com/pipeseroni/pipes.sh/master/pipes.sh;chmod 755 pipes.sh`



* * * 
## NCmatrix - Network Code Matrix
Cmatrix is mad popular terminal screensaver ... wouldnt it be great if that actually did something?
You can attach colors to  based on incoming and outgoing packets of the network interface.
[https://github.com/mayfrost/NCMatrix](https://github.com/mayfrost/NCMatrix)



* * * 
## CACA-UTILS:
Cacademo displays  ASCII art effects with animated transitions: metaballs, moire pattern of con-centric circles, old school plasma, Matrix-like scrolling.
`apt install caca-utils`
To view the programs (binaries) included with caca-utils:
`dpkg -L caca-utils|grep -i /usr/bin/`
- /usr/bin/cacaview   - ASCII image browser (renders images)
- /usr/bin/cacafire   - libcaca's demonstration applications
- /usr/bin/cacaserver - telnet server for libcaca
- /usr/bin/cacaplay   - play libcaca animation files
- /usr/bin/cacademo   - libcaca's demonstration applications
- /usr/bin/img2txt    - convert images to various text-based coloured files
- (If you installed -0.99.beta18 also cacaclock - Text-mode clock display
CACA - VIEW MOVIES IN LINUX TERMINAL AS ASCII ART
 Yes you can watch movies in your Linux terminal, but you need VLC to do so. `apt install vlc`
 `cvlc --loop --no-audio -V caca Animation.mp4`
 To quit a video in loop, `CTRL+Q`
 
 Mplayer will also display this in terminal. `apt install mplayer`
 `mplayer -vo caca Animation.mp4`



* * * 
## WEAVE
Using Bash's Arithmetic Evaluation to enable the weaving
`cd /usr/local/bin;wget https://raw.githubusercontent.com/pipeseroni/weave.sh/master/weave.sh;chmod 755 weave.sh`

`Ctrl+l` to clear screen. Some example variables to run weave:

- `./weave.sh '(((x + y) % 2))'`
- `./weave.sh '(((x % 5 + y % 11) % 2))'`
- `char_v='|' char_h='-' ./weave.sh`
And:
- `./weave.sh '(( $(echo "v=s(($x-1)*2*4*a(1)/$W);scale=0;$H-$H*(v+1)/2 == $y" | bc -l) ))'`



* * *
# PearlBased Terminal Screensavers
* * *
## ASCIIQuarium

You can preview your own live ASCIIQuarium at [asciiquarium.live](https://github.com/kilimnik/asciiquarium.live).



### Install Pre-requisets:

`apt install libcurses-perl build-essential`

`cpan -i Term::Animation`

The first time running cpan you will be prompted with many questions. Hit Enter key to all questions to select its default.



#### Next, copy this URL and download it:

`wget http://search.cpan.org/CPAN/authors/id/K/KB/KBAUCOM/Term-Animation-2.6.tar.gz`

`tar -zxvf Term-Animation-2.6.tar.gz`

`cd Term-Animation-2.6/`

`perl Makefile.PL && make && make test`

`make install`



#### Clean up:

`rm Term-Animation-2.6.tar.gz`

`rm Term-Animation-2.6/ -rf`


### Install mission Bundle in Pearl:

Invoke the CPAN shell by entering the following command at a system shell prompt:

`% perl -MCPAN -eshell`

With the new prompt, all you have to do is enter:

`install Bundle::LWP`


## Now you can install ASCIIQuarium:

`wget http://www.robobunny.com/projects/asciiquarium/asciiquarium_1.1.tar.gz`

`tar -zxvf asciiquarium_1.1.tar.gz`

`cd asciiquarium_1.1/`

`cp asciiquarium /usr/local/bin`

`chmod 0755 /usr/local/bin/asciiquarium`

`cd ../;rm asciiquarium_1.1/ -rf`

`rm asciiquarium_1.1.tar.gz`



## Weatherspect (broken but cool scene)

The requirements for WeatherSpect are the same as ASCIIQuarium.

To use the Weather Thing also install: 

`cpan -i Weather::Underground`

But it doesnt work. 

`wget http://www.robobunny.com/projects/weatherspect/weatherspect_v1.11.tar.gz`

`tar -zxvf weatherspect_v1.11.tar.gz && cd weatherspect_v1.11/`

`cp weatherspect /usr/local/bin && chmod 0755 /usr/local/bin/weatherspect`

`cd ../;weatherspect_v1.11/ -rf`

`rm weatherspect_v1.11.tar.gz`



* * *
## Making it a command

1. Editing .bashrc or .profile to make the screen saver work on a command you may edit .bashrc or .profile (if you are using ubuntu) using one of your favourite text editor here we are using vi:

`vi ~/.bashrc`

2. Put the following content in `.bashrc` if you are using Ubuntu `.profile`

```
    function run_scr(){
       j=0;a=1;x=1;y=1;xd=1;yd=1;while true;do for i in {1..2000} ; do if [[ $x == $LINES || $x == 0 ]]; then xd=$(( $xd *-1 )) ; fi ; if [[ $y == $COLUMNS || $y == 0 ]]; then yd=$(( $yd * -1 )) ; fi ; x=$(( $x + $xd )); y=$(( $y + $yd )); printf "\33[%s;%sH\33[48;5;%sm . \33[0m" $x $y $(( $a % 8 + 16 + $j % 223 )) ;a=$(( $a + 1 )) ; done ; x=$(( x%$COLUMNS + 1 )) ; j=$(( $j + 8 )) ;done &
    main=$!
    tput smso
    echo "Press any key to return \c"
    tput rmso
    oldstty=`stty -g`
    stty -icanon -echo min 1 time 0
    dd bs=1 count=1 >/dev/null 2>&1
    stty "$oldstty"
    echo
    clear
    clear
    kill $main
    }
    alias screensaver=run_scr
```

3. Close the session and open it again, or simply reload the .bashrc configuration

4. To enable to screen saver type the following command string:
    `screensaver`

5. To stop it press any key or `**CTRL+C**`

NOTE: These calls are not compatible with all linux distros. 

[Source for copy paste screensaver one liners](http://web.archive.org/web/20140315023348/http://thelinuxnotes.com/post/screen-saver-for-text-terminal#playing-with-cursor-and-color)


* * * 
## Automatic Terminal Screensaver

This github repo has a setup to run a screensaver every XX time alotment. This is basically a screensaver!

[https://github.com/xiongchiamiov/terminal-screensaver](https://github.com/xiongchiamiov/terminal-screensaver)


* * *
## Collection
[Best collection of Terminal Based screensavers](http://mewbies.com/acute_terminal_fun_02_view_ascii_art_ansi_art_and_movies_on_the_terminal.htm)
 
 
 
* * *
## Bonus Points:
> Install Unreal Tournament and have BOTS play eachother:
{: .prompt-info }
[Text Mode Unreal Tournament](http://icculus.org/~chunky/ut/aaut/)
