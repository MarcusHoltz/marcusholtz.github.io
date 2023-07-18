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


## Using DNS txt records in Base64 you can download a photo.

If you want to inspect the script before you run it, it's stored in DNS TXT records as well:

```bash
dig +short TXT marcusInDNS.holtzweb.com | tr -d '"' | tr -d "\n\r" | tr -d [:blank:] | base64 -d
```



### Download and run the script

If you want to just create MarcusHoltz.jpg and see if this even works, use the line below:

```bash
wget https://raw.githubusercontent.com/MarcusHoltz/DNS-photo-download/main/downloadmarcusholtz.sh && chmod +x downloadmarcusholtz.sh && /bin/bash downloadmarcusholtz.sh
```



## Explaining the process

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

- Copy one of my test TXT records ```andsuch.domain.com.    1   IN  TXT   "this is a text record example"``` 

- Into Notepad++, edit it

- Cloudflare also in the same feature allows you to import the edited file back into the DNS Zone


#### Notepad++

Notepad++ or qqNotepad is a great piece of software I like to use for text editing. It allows for quick macros.

I used these macros to aid in changing the Base64 image into a template that could be uploaded back to Cloudflare.

* * * 

##### Changing the copied TXT record in Notepad with macros

Using the macro I just recorded moving over 254 characters, and then a character return. Repeated till end of document.

Then again, with macros, I incremented the DNS number on my subdomain for each line that had been returned. E.G. - ```andsuch1.domain.com.  1   IN  TXT   "f0+hAUXHl9hV50yW0mjilYYZdSMOGqSQBj/b6xnSCB9O1SDwNLjK5rpu"``` 

Keep in mind the total TXT records you made, we'll need this number later on.


#### Don't forget, to upload this new modified text file to Cloudflare

Cloudflare will give you an error message if anything isnt semantically correct.



* * *

### Recombining the DNS TXT records into an image

* * *


#### Now that everything is Base64

You can convert base64 back with the command ```base64 -d```. Base64 takes standard input, so you'll need to echo or pipe something to it. 



#### Create a loop in a shell script using dig

I probably should have explained this earlier, but we'll be using dig to grab these DNS entries. 



##### Let me explain the shell script we'll be using to recombine the DNS TXT records.

Remember when I said we'd need the total number of TXT records you made? 

- Replace the **"N"** in ```for i in {1..N}``` with the total number of TXT records created.

- TXT DNS from dig have quotations around them, we also need to remove these quotes.

- Each TXT record is then put into a temp file, and line by line the original Base64 is copied.

- After all the DNS TXT records have been copied in order, you must remove the line breaks that seperate them and combine into a single line.

- Then convert the single line of text in the temp file from Base64 to an image.

- Confirm the image is created to the user.

- Clean up any temporary files that might exist




#### Shell Script to recombine the DNS Records

```bash
for i in {1..N}
do
dig +short TXT andsuch$i.domain.com @9.9.9.9 | tr -d '"' >> workingpicture.tmp
done
cat workingpicture.tmp | tr -d "\n\r" | base64 -d > MyPhoto.jpg
echo -e "\nMyPhoto.jpg created\n"
rm workingpicture.tmp showmethescript.sh showmeportrait.sh >/dev/null 2>&1
```


### Stuffing your shell script into a DNS TXT record

That script is pretty large, it wont meet the 255 character limit that we have for TXT records. But most DNS ZONE files will take something up to 2,048 characters, but it will still split them up.

To fix this you must remove the spaces in these larger 255+ character TXT records. That's why the ```tr -d [:blank:]``` was used above to retrieve the script.



#### Stuffing your shell script into a single print line

Having everything in a print statement allows it to be sent to a file for later use.

Make sure you escape all the special characters you might run into when creating the printf command.

```bash
printf "for i in {1..N}\ndo\ndig +short TXT andsuch\$i.domain.com @9.9.9.9 | tr -d '\"' >> workingpicture.tmp\ndone\ncat workingpicture.tmp | tr -d \"\\\\n\\\\r\" | base64 -d > MyPhoto.jpg\necho -e \"\\\\MyPhoto.jpg created\\\n\"\nrm workingpicture.tmp showmethescript.sh showmeportrait.sh >/dev/null 2>&1"
```


##### Stuff your double stuffed shell script into base64 format

Now that you have a printf command that will output your script, convert that to base64 to prevent plain text dns records for sitting around, and to save space ;-)

I usually use the subdomain without the number after it. So in our example it would be - ```andsuch.domain.com.  1   IN  TXT   "fEACkQAAEAUXHlHyxj50yW0mjilYLwj8j/b6xnSCB9O1kj8NLjK5rpu28WC3iCs"``` 

And your base64 TXT record will be much much longer than this example, but now you've got your script to bring down and combine all of your DNS TXT records for this subdomain into an image. 

Pretty cool.



## Enjoy a photo from a DNS TXT record. 

Well, you should now be looking at your photo.


### Reviewing the steps taken


### Converted from image -> base64 -> dns txt -> shell buffer -> base64 temp text file -> image

Thanks for looking at this project. Hope you get some practical use out of it ;-)
