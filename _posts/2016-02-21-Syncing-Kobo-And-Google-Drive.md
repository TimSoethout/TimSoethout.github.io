---
layout: post
category:
tagline:
tags: [ereader, kobo, aura hd, linux, gdrive, epub]
title: Syncing Kobo and Google drive
---
{% include JB/setup %}

I finally made it work again. Some posts ago I wrote about [sending files to my Kobo ereader]({% post_url 2014-04-19-Send-to-kobo %} using email on (http://sendtokobo.com/). Unfortunately this service stopped working some time ago.

I still wanted a workflow where I could put the files I want to read on my e-reader at a later time. After a day of struggling I found out a way that works! I can now dump my files in a Google drive folder and sync them at a later time on my Kobo when connected to the internet. No hassle with connecting the e-reader over USB to my machine, and forgetting to do so when at home.

After a factory reset of my Kobo Aura HD, I installed [this latest and greatest version](https://www.preining.info/blog/2016/01/kobo-firmware-3-19-5761-mega-update-ksm-nickel-patch-ssh-fonts/) of a firmware containing Kobo Start Menu (KSM), [Koreader](https://github.com/koreader/koreader) and telnet/ssh access. Koreader has better support for pdf than the native reader, and KSM allows for easier unix hacks.
An [older post](https://www.preining.info/blog/2015/08/kobo-glohd-firmware-3-17-0-mega-update-ksm-nickel-patch-ssh-fonts/) really helped me figuring out how to connect remotely to the Kobo device using: `telnet $IP_OF_KOBO` with user `root` without password. Previous link also explains how to enable the much safer SSH access instead of telnet.

Next step was inspired by [this gist](https://gist.github.com/wernerb/7864141#file-sync-sh) which uses `wget` to fetch files from Google drive. I updated it to work for me:

{% gist 4c5db37e7bd1a2b4fb49 %}

Feel free to use this file and insert your own Google drive folder ID for a folder which you have set the sharing to "Anyone with the link can view".

The file should go somewhere on the kobo device, I put mine in `/mnt/onboard/.adds/kbmenu_user/scripts/sync.sh` to make sure it comes up in `custom scripts` in the KSM.

Next I ran into another problem because Google uses `https` and the `wget` provided on the Kobo device is not compiled with the appropriate flags to support this.
Fortunately there was [another attempt on Github](https://github.com/spMatti/kobo-wget-sync) doing sync with Google Drive, which actually downloads a version of `wget` that does.
I installed this on my device using the instructions, but probably getting only `wget` from it would have been enough.

Now I have everything to make this workflow happen:

- Drop `epub`/`pdf` file in my shared folder in Google drive.
- Enable wifi on the Kobo
- Hit `synch.sh` in `custom scripts`
- Wait a bit until the downloads are in (the screen flashes)
- And enjoy the files being places in `gdrive`
- (You might need to trigger the library refresh somehow, if you use the native Kobo reader application if you are not using Koreader)
