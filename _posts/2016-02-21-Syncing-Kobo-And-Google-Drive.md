---
layout: post
category:
tagline:
tags: [ereader, kobo, aura hd, linux, gdrive, epub]
title: Syncing Kobo and Google drive
---
{% include JB/setup %}

I finally made it work again. Some posts ago I wrote about [sending files to my Kobo ereader](/2014/04/19/Send-to-kobo/) using email with http://sendtokobo.com/. Unfortunately this service stopped working some time ago.

After a factory reset of my Kobo Aura HD, I installed [this latest and greatest version](https://www.preining.info/blog/2016/01/kobo-firmware-3-19-5761-mega-update-ksm-nickel-patch-ssh-fonts/) firmware containing Kobo Start Menu (KSM), [Koreader](https://github.com/koreader/koreader) and telnet/ssh access. Koreader has better support for pdf than the native reader, and KSM allows for easier unix hacks.
After reading an [older post](https://www.preining.info/blog/2015/08/kobo-glohd-firmware-3-17-0-mega-update-ksm-nickel-patch-ssh-fonts/) I found out how to easier connect with: `telnet $IP_OF_KOBO` using user `root` without password. Previous link also explains how to enable the much safer ssh access.

After this I found [this gist](https://gist.github.com/wernerb/7864141#file-sync-sh) which uses `wget` to fetch files from Google drive and updated it to work for me:
https://gist.github.com/TimSoethout/4c5db37e7bd1a2b4fb49
Feel free to use this file and insert your own Google drive folder ID for a folder which you have set the sharing to "Anyone with the link can view".

The file should go somewhere on the kobo device, I put mine in `/mnt/onboard/.adds/kbmenu_user/scripts/sync.sh` to make sure it comes up in `custom scripts` in the KSM.

Next I ran into another problem because Google uses `https` and the `wget` provided on the Kobo device is not compiled with the appropriate flags to support this.
Another attempt on github doing some sync with Google drive actually downloads a version that does: https://github.com/spMatti/kobo-wget-sync
I installed this, but probably getting only `wget` from it would have been enough.

Now I have everything to make this workflow happen:

- Drop `epub`/`pdf` file in my shared folder in Google drive.
- Enable wifi on the Kobo
- Hit `synch.sh` in `custom scripts`
- Wait a bit until the screen flashes and the downloads are in
- And enjoy the files being places in `gdrive`
