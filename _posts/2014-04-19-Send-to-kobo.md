---
layout: post
category: 
tagline: 
tags: [ereader, kobo, aura hd, linux]
---
{% include JB/setup %}

A while ago I bought my first e-reader. The [Kobo Aura HD](https://www.kobo.com/koboaurahd). It is a very nice device with a clear screen and it turned out to run some kind of linux.

You can copy a file with name `KoboRoot.tgz` to the `.kobo` directory when mounted and as soon as you unmount and disconnect the device, it will copy the contents of the file into the root file system of the ereader. Thus there is a way to make changes to your device!

Some time ago I even managed to install an ssh server on the ereader. See https://wikisec.free.fr/mobile/kobo.html#ssh

One of the things I miss on my Kobo is an easy way to send files to the device. You must either connect the reader and copy over the file or run a Calibre server and browse to it on the webbrowser on the device. This is ofter far to much work especially if you want to send smaller texts such as blog posts and news articles to your device.

More recently I stumbled upon [a website](https://sendtokobo.com/) that uses the method described above to let the device receive messages from an email-adress. I signed up and am trying it out now. This would be an ideal functionally to make my ereader more useful!