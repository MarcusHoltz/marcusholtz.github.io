---
title: Create a copy of your playlist's music files to a directory
date: 2024-05-21 11:33:00 -0700
categories: [Script, Files]
tags: [administration, techtip, script, rename, files, playlist]
pin: false
image:
  path: /assets/img/header/header--script--copy-m3u-playlist-files.jpg
  alt: Copy your m3u music playlist to a directory
---

# 🎶 Copy music files in a playlist for export 💿

> Copy your mp3s, in order, from an m3u playlist into a directory for export.
{: .prompt-info }

These scripts will: 

- Collect files in your music playlist

- Order the songs

- Copy them into a new directory. 


> Now can burn your collection to a CD, or send the playlist to friends. 👍
{: .prompt-tip }



* * *

## Using copy-m3u-playlist-files-to-directory

The script can, optionally, take two arguments:

- The name of the m3u playlist file.

- The location to place the contents of the playlist. 

> If neither is specified, the script defaults to `playlist.m3u` and `current working directory`.


* * *

# Usage Examples

* * * 


## Download on Linux 

You can download the Bash version of this script here:

```
wget https://raw.githubusercontent.com/MarcusHoltz/copy-m3u-playlist-files/main/copy-m3u-playlist-files-to-directory.sh
```

* * * 

### Running on Bash

Using the bash version of this script, run the script with:

```
bash copy-m3u-playlist-files-to-directory.sh name-of-playlist.m3u /home/user/Music/somefiles
```


* * * 

* * * 

## Download on Windows

You can download the Powershell version of this script to a specified directory with a command like:

```
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/MarcusHoltz/copy-m3u-playlist-files/main/copy-m3u-playlist-files-to-directory.ps1" -OutFile "C:\path\to\destination\somefolder\copy-m3u-playlist-files-to-directory.ps1"
```


* * * 

### Running on Windows Powershell

Using the Powershell version of this script you just downloaded, run the Powershell script with:

```
copy-m3u-playlist-files-to-directory.ps1 name-of-your-playlist.m3u C:\path\to\destination\somefolder\
```


* * * 

* * * 

## Download using Python

To download the Python version, with more Python...

Create and run this python script to download the file:

```
import urllib.request

url = "https://raw.githubusercontent.com/MarcusHoltz/copy-m3u-playlist-files/main/copy-m3u-playlist-files-to-directory.py"
response = urllib.request.urlopen(url)

with open("copy-m3u-playlist-files-to-directory.py", "wb") as file:
    file.write(response.read())
```


* * * 

### Running the Python script with Python

To run the downloaded Python script:

```
python copy-m3u-playlist-files-to-directory.py example-playlist.m3u /path/to/destination
```


* * *

* * *

## Thanks!

You can find more information about this repo at:

[https://github.com/MarcusHoltz/copy-m3u-playlist-files/](https://github.com/MarcusHoltz/copy-m3u-playlist-files/).
