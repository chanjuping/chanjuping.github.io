---
layout: post
title: "Connecting to KDE Connect via the CLI Interface"
date: 2021-04-26 +0800
categories: [Linux, KDE Connect, Raspberry Pi 4]
---

This week marks the proper start of me experimenting with i3 on the Raspberry Pi 4 (8GB) as my main machine. To paraphrase a quote I read somewhere long ago I can no longer track down, true beauty can only be found when one is limited. Using the Pi4 as my main machine for just a few days alone has sharpened my understanding of Linux system processes and also improved my ability to read documentation I would usually get on a DuckDuckGo search.

The Pi is not going to win any contests rendering webpages when facing down even my 2013 Zenbook, but when I try as best as possible to remove GUI abstraction layers to run tools directly from the terminal, everything flies just as fast if not faster compared to a GUI interface.

In this instance, I wanted to pair my phone with my Pi via the KDEConnect system. And the best way to get it up and running on a minimalist system is to go the full terminal route.

First, install `kdeconnect`.

> sudo apt install kdeconnect

Next get the path to the `kdeconnectd` executable.

> dpkg-query -L kdeconnect | grep libexec

In my case, I found the file in `/usr/lib/aarch64-linux-gnu/libexec/kdeconnectd`. I launched it thus:

> cd /usr/lib/aarch64-linux-gnu/libexec/
> ./kdeconnect

Help with the tool can be found via:

> kdeconnect-cli --help

With the KDEConnect app running on my phone, I was able to check that my Pi's hostname was being advertised. Since I did not have a proper Plasma desktop running, I decided to initiate a pairing request from the Pi to my phone. I ran a scan in the terminal for available devices:

> kdeconnect-cli -a

And after seeing my phone's ID, I initiated a pairing request, substituting `DeviceIDString` with the output from `kdeconnect-cli -a`:

> kdeconnect-cli -d `DeviceIDString` --pair

Accept the pairing request on your phone, and you are ready to rock!
