---
layout: post
title: OpenWRT for COMFAST Routers
description: LEDE project supports COMFAST devices
date: '2017-03-10 20:33:49'
tags: openwrt comfast lede
---

## OpenWrt support for COMFAST routers is scarce, but you could use LEDE instead.

If you came here looking for [OpenWrt](https://openwrt.org/) firmware for [COMFAST](http://en.comfast.com.cn/) routers, you probably haven't read this [OpenWrt forum post](https://forum.openwrt.org/viewtopic.php?id=58973).
COMFAST (like many Chinese companies), showed little regard for the GPL, and this pissed off a lot of OpenWrt devs. 

### OpenWrt support for COMFAST
Although it isn't clear if this is the sole reason why OpenWrt support for COMFAST routers is scarce, but surely support for COMFAST routers *is* scarce and there isn't much being done to add support for these devices.

The other problem with COMFAST is that on their default (vendor supplied) firmware, ssh password has been changed. And, their web UI update doesn't accept OpenWrt firmwares. It only accepts COMFAST firmwares. 

#### The LEDE project
The good news is that the [LEDE project](https://lede-project.org/) (also based on OpenWrt) supports quite a few COMFAST models. And a lot of new COMFAST models are essentially clones of earlier ones. (The CF-E325n for example is a CF-E316N v2 clone.)

You can find prebuilt LEDE firmwares [here](https://downloads.lede-project.org/releases/17.01.0/targets/ar71xx/generic/). 

### Flashing the firmware
* I suggest you get yourself a USB TTL serial cable. You'd have to open up the COMFAST router and connect the cable to the Tx, Rx and ground pins, to be able to see the boot logs. 
* The bootloader, by default, checks for a `firmware_auto.bin` tftp server on a predefined IP. You can view this in the boot logs. 
* Then assign your dev machine this static predefined IP and setup a tftp server on your dev machine. 
* Rename the firmware to `firmware_auto.bin`, and serve it on the tftp server. 
* Now reset the router.

If all goes well, the device should connect to tftp server, download and flash the firmware. It may take about a minute for the device to show up on the network.

