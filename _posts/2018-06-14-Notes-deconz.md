---
title: Notes DeCONZ
layout: post
category: null
tagline: null
tags:
  - deconz
---

{% include JB/setup %}

Here are some notes and command lines I used to set DeConz up on my Odroid C2. Mainly for my backup, but if someone needs more info, I'm glad to try and remember how I did it.

## Deconz

```sudo apt-get install libqt5serialport5 libqt5websockets5 libqt5sql5 sqlite3```

```sudo dpkg --ignore-depends=wiringpi:armhf -i deconz-2.05.29-qt5.deb```

Unfortunately apt will not forget the dependency, and keep complaining.

```
The following packages have unmet dependencies:
 deconz:armhf : Depends: wiringpi:armhf but it is not installable
```

Fix: ```sudo nano /var/lib/dpkg/status```
Remove `wiringpi` from Depends (search for this in this document)

```sudo nano /usr/lib/systemd/system/deconz.service```
or `/lib/systemd/system/deconz.service` (in newer versions of deconz?)
- rename user to root (no pi) (TODO should use odroid or specific user for this)
- comment out (#) capabilities (will give error on start, don't know why, too old version of systemd?)

```systemctl daemon-reload```
```journalctl -e -u deconz```

## Home Assistant

https://github.com/ggravlingen/pytradfri/blob/master/script/install-coap-client.sh
Required for pytradfri