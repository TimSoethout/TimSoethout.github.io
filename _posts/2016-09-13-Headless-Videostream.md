---
title: Headless Videostream
layout: post
category: null
tagline: null
tags:
  - chromecast
  - atom cpu
  - Videostream
---

{% include JB/setup %}

I have a file server with some media on it that I want to stream to my chromecast. Unfortunately the server is an old 32bit atom server with not too much power. It is even to slow for streaming using Plex Media Server.

I am using a [Chrome Extension/App VideoStream](https://chrome.google.com/webstore/detail/videostream-for-google-ch/cnciopoikihiagdjbjpnocolokfelagl) on a Macbook to watch on my Chromecast, while having the media files mounted over AFP.
This is workable but not ideal because I need to have the laptop running, the mount ready, Chrome with VideoStream running and sometimes reindex all my video's.

I found a way to run this on my simple low-powered Ubuntu server.

### 1. Install google-chrome

NB. Use google-chrome, chromium had problems using the Chromecast for me.

I used the latest i386 version that I could find:

```
wget http://bbgentoo.ilb.ru/distfiles/google-chrome-stable_48.0.2564.116-1_i386.deb
sudo dpkg -i google-chrome-stable_48.0.2564.116-1_i386.deb
sudo apt-get -f install
```

The last line was to force the dependencies to be installed correctly.

### 2. Install VideoStream

Now I connected to my server using `ssh -X` to enable X-forwarding. I could start Chrome now and install the Extension from https://chrome.google.com/webstore/detail/videostream-for-google-ch/cnciopoikihiagdjbjpnocolokfelagl . On the first start it will also install the Chromecast extension for you.

Now setup the connection to the mobile VideoStream app (on Android for me).

### 3. Run it Headless

Install xvfb to be able to run Chrome headless.

```
sudo apt install xvfb
```

And run it! The app-id is the VideoStream app id and also the same as in the URL in the Chromestore.

```
xvfb-run google-chrome --app-id=cnciopoikihiagdjbjpnocolokfelagl > /dev/null &
```


Extra:
This might help to kill it again.

```
killall Xvfb
```
