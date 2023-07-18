---
title: Upload a photo using DNS, Download it anywhere
date: 2023-07-17 11:33:00 -0700
categories: [Linux]
tags: [dns, photostorage, cryptography]
pin: false
image:
  path: /assets/img/header/header--dns--upload-download-photo-using-dns-and-base64.png
  alt: Upload a photo using DNS and Download it anywhere
---

# Download Marcus Holtz' photo using DNS


## Would you like to download a picture of Marcus Holtz? NOW YOU CAN!


## Using DNS txt records in Base64 you can download his portrait.

If you want to inspect the script before you run it, it's stored in DNS TXT records as well:

```bash
dig +short TXT marcusInDNS.holtzweb.com | tr -d '"' | tr -d "\n\r" | tr -d [:blank:] | base64 -d
```



### Download and run the script

If you want to just create MarcusHoltz.jpg and if this even works, use the line below:

```bash
wget https://raw.githubusercontent.com/MarcusHoltz/DNS-photo-download/main/downloadmarcusholtz.sh && chmod +x downloadmarcusholtz.sh && /bin/bash downloadmarcusholtz.sh
```



## Explaining the process

Let's just see if we can do this, oh... we can. Let's.

* * * 

This repo is for taking an image, stuffing it into a domain's DNS, and then extracting that image again. 


### Creating a base64 image

What do you want to display? Find an image, but make sure it's SMALL. The only image I found fit in one TXT record was 24 pixel square inside of a 31x31px canvas. So this image will be broken up into many different DNS records that we will combine later.


#### Having found the image to use

- go [search](https://www.ecosia.org/search?method=index&q=base64+encoder) for a base64 encoder

- take your image and convert it to text

- keep your new image as text and save it

- make sure you save it directly from the site you used

    * Windows may format the text differently, so no copy/paste



### Assign 254 character TXT records to incremented subdomains

This was accomplished with Cloudflare and Notepad++. 

* * * 

#### Cloudflare

- Cloudflare allowed me to export my DNS records

- Copy one of my test TXT records ```andsuch.domain.com.    1   IN  TXT   "this is a text record example"``` 

- Into Notepad++, edit it

- Cloudflare also in the same feature allows you to import the edited file back into the DNS Zone


#### Notepad++

Notepad++ or qqNotepad is a great piece of software I like to use for text editing. It allows for quick macros.

I used these macros to aid in changing the Base64 image into a template that could be uploaded back to Cloudflare.

* * * 

##### Changing the copied TXT record in Notepad with macros

Using the macro I just recorded moving over 254 characters, and then a character return. Repeated till end of document.

Then again, with macros, I incremented the DNS number on my subdomain for each line that had been returned. E.G. - ```andsuch1.domain.com.  1   IN  TXT   "f0+hAUXHl9hV50yW0mjilYYZdSMOGqSQBj/b6xnSCB9O1SDwNLjK5rpu"``` 



