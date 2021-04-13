---
layout: post
title: "Optimized Setup for DSI Touchscreen and HDMI Output"
date: 2021-04-09 18:10:00 +0800
categories: [RaspberryPi, Pi3, Touchscreen, HDMI]
---

It took quite a bit of time to get the official Raspberry Pi touchscreen working correctly with my monitor via HDMI at the same time, so I am going to be documenting the important details here.

The primary source of my guide is found [here](https://www.raspberrypi.org/forums/viewtopic.php?p=1504551&sid=50656cbcb593715dbb631ce935544a12#p1504551) though I will pepper the guide with other links that helped me resolve the issue fully.

# The Primary Solution

Add the following to the end of `/boot/config.txt` with only the touchscreen connected:

    dtparam=audio=off
    disable_overscan=1

    enable_uart=off
    start_x=1

    max_framebuffers=2

    #  desktop display will default to the LCD (fb0)
    # This will force the specified display to be the first in the list, i.e. /dev/fb0
    # Actually this is the full set:
    # MAIN_LCD  0
    # AUX_LCD   1
    # HDMI0     2
    # SDTV      3
    # HDMI1 is only availabe on RPi4
    # HDMI1     7
    framebuffer_priority=0

    # increase GPU memory
    gpu_mem=112

Replace the `/usr/share/X11/xorg.conf.d/99-fbturbo.conf` file contents with:

    # This is a minimal sample config file, which can be copied to
    # /etc/X11/xorg.conf in order to make the Xorg server pick up
    # and load xf86-video-fbturbo driver installed in the system.
    # When troubleshooting, check /var/log/Xorg.0.log for the debugging
    # output and error messages.
    #
    # Run "man fbturbo" to get additional information about the extra
    # configuration options for tuning the driver.

    #Section "Device"
    #        Identifier      "Allwinner A10/A13 FBDEV"
    #        Driver          "fbturbo"
    #        Option          "fbdev" "/dev/fb1"
    #        Option          "SwapbuffersWait" "true"
    #EndSection


    Section "Device"
            Identifier      "fbturbo0"
            Driver          "fbturbo"
            Option          "fbdev" "/dev/fb0"

            Option          "SwapbuffersWait" "true"
            Option          "debug" "true"
    EndSection

    Section "Device"
            Identifier      "fbturbo1"
            Driver          "fbturbo"
            Option          "fbdev" "/dev/fb1"

            Option          "SwapbuffersWait" "true"
            Option          "debug" "true"
    EndSection

    Section "Device"
            Identifier      "fbdev0"
            Driver          "fbdev"
            Option          "fbdev" "/dev/fb0"
    EndSection

    Section "Device"
            Identifier      "fbdev1"
            Driver          "fbdev"
            Option          "fbdev" "/dev/fb1"
    EndSection

    Section "Monitor"
        Identifier "Monitor0"
        Option "Primary" "False"
    EndSection

    Section "Monitor"
        Identifier "Monitor1"
        Option "RightOf" "Monitor0"
        Option "Primary" "False"
    EndSection

    Section "Screen"
        Identifier "ScreenTurbo0"
        Monitor "Monitor0"
        Device "fbturbo0"
        Subsection "Display"
        EndSubSection
    EndSection

    Section "Screen"
        Identifier "ScreenTurbo1"
        Monitor "Monitor1"
        Device "fbturbo1"
        Subsection "Display"
        EndSubSection
    EndSection

    Section "Screen"
        Identifier "ScreenDev0"
        Monitor "Monitor0"
        Device "fbdev0"
        Subsection "Display"
        EndSubSection
    EndSection

    Section "Screen"
        Identifier "ScreenDev1"
        Monitor "Monitor1"
        Device "fbdev1"
        Subsection "Display"
        EndSubSection
    EndSection

    Section "ServerLayout"
        Identifier "Multihead"
        Screen  0 "ScreenTurbo0"
        Screen  1 "ScreenTurbo1" rightof "ScreenTurbo0"
        Option  "Xinerama" "true"
    EndSection

    Section "ServerLayout"
        Identifier "Singlehead0"
        Screen  0 "ScreenTurbo0"
    EndSection

    Section "ServerLayout"
        Identifier "Singlehead1"
        Screen  0 "ScreenTurbo1"
    EndSection

    Section "ServerFlags"
        Option "BlankTime"  "0"
        Option "StandbyTime"  "0"
        Option "SuspendTime"  "0"
        Option "OffTime"  "0"
        Option "DefaultServerLayout" "Singlehead0"
    EndSection

Reboot and check that there are two framebuffer devices

    ls /dev/fb*

Connect the HDMI cable to the Pi, and in `raspi-config`, select the autologin to CLI option and reboot.

When tty1 loads up, run:

    startx -- -layout Multihead

All the steps above should result in both the Pi touchscreen and monitor showing an extended desktop.

# Fixes

## Screen orientation

If like me, you bought a case for your Pi3 and touchscreen and found out the default display on it is upside down, add the following into `/boot/config.txt`

    lcd_rotate=2

The HDMI output orientation can be similarly modified by using:

    display_hdmi_rotate

## Automating startup command

After testing out a few options, creating an executable script and autorunning it in tty1 (the default tty when booting without a GUI) on bootup was my personal preference for executing the `x` startup options for using the touchscreen and HDMI connected monitor at the same time.

I first created a file called hdmi in `/usr/local/bin` with the content:

    #!/bin/bash

    startx -- -layout Multihead

Then I made it an executable file with the command:

    sudo chmod +x /usr/local/bin/hdmi

After testing that it works by restarting and executing the script directly in tty1, I added the following lines to `~/.profile`

    # launch GUI for hdmi and lcd
    if [[ "$(tty)" == "/dev/tty1" ]]
        then
            hdmi
    fi

Reboot and if all is well with the universe, the HDMI output should work soon after booting up.

The reason why I did not dump the script into `.bashrc` or `.bash_login` as one user suggested [in the thread](https://www.raspberrypi.org/forums/viewtopic.php?p=1578609&sid=50656cbcb593715dbb631ce935544a12#p1578609) was that the script would execute if I were to login via ssh to the Pi. Even though it would fail to execute and make more of a mess, the extra inefficiency was unnecessary, hence this roundabout process. However, this would be a reasonable solution if you never intend to login via ssh or drop into the other tty sessions.

# Conclusion

There are other tweaks I will be working on to get this Pi with a touchscreen working with my setup. I have already connected it with my monitor via an HDMI switcher so I can easily swap between my desktop and Pi without physically unplugging stuff.

Admittedly, the Raspberry Pi 3 is probably a fair bit outdated by now, however, it is still quite the capable low-powered computer, and hopefully, I can use it for displaying sheet music via Musescore or syncing and reading books via Calibre very soon.