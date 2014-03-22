---
layout: post
category: 
tagline: 
tags: [osx, wifi, channel 12, channel 13]
---
{% include JB/setup %}

I found out that my OSX did not see every wifi channel available. See `System Information` > `System Report`, then `Network` > `Wifi`:

```
en0:
  Card Type:	AirPort Extreme  (0x14E4, 0xD1)
  [..]
  Country Code:	US
  Supported PHY Modes:	802.11 a/b/g/n
  Supported Channels:	1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 36, 40, 44, 48, 52, 56, 60, 64, 100, 104, 108, 112, 116, 120, 124, 128, 132, 136, 140
  [..]
```

Due to regulations the allowed wifi channels in the world differ in location. See [wikipedia](https://en.wikipedia.org/wiki/List_of_WLAN_channels#Interference_Concerns).

My adapter has configured itself to `Country Code: US` and `US` does not allow wifi (2.4Ghz) channels 12 and 13, while `NL` for example does. 

The country code is not hardcoded, but it turns out the wifi adapter picks the country code from the first wifi network it discovers. So turning off and on your wifi can make it possible to use more channels.
Some blogs say that changing the locale and location settings and rebooting fixes this issue, but it is really the rebooting that triggers the wifi adapter reset and having luck that makes this work.

The sad news is that there seems not way to control this behaviour. 

