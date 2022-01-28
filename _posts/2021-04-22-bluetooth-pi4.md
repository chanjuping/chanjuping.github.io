---
layout: post
title: "Bluetooth Fix for Some Raspberry Pi 4 devices (Probably Resolved)"
date: 2021-04-22 +0800
categories: [Linux, Bluetooth, Raspberry Pi 4]
---

Update 20210515: Yesterday the upgrade to Ubuntu 21.04 was made available for my setup, and after upgrading, this solution seems to have been obsoleted! I had to re-pair my mouse initially after saying goodbye to Groovy Gorrila, but after that, no more need to run esoteric commands to get Bluetooth working. Hooray!

A very unexpected problem cropped up when I received my new Raspberry Pi 4 8GB device. I could no longer pair, let alone connect to my Logitech MX Ergo. A surprising development considering how relatively reliable the connectivity was with my 2 GB variant of the Pi 4 devices.

Many hours of searching later, I found the solution [here](https://github.com/RPi-Distro/pi-bluetooth/issues/8#issuecomment-467668132) which indicates a potential race condition during bootup.

A follow up comment suggesting modification of the bthelper@service file no longer seemed to work, so I instead simplified the commands to run into a function in `.bashrc`:

```shell

function bt() {
        sudo hciconfig hci0 down
        sudo systemctl restart bluetooth
}

```

A bit of an ugly workaround while we wait for the patches to roll in. One can only assume that QA testing was with physically connected mice and keyboards so this problem escaped attention for the newer devices.
