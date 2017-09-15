---
layout: post
title: Port fowarder using MiniUPnP
description: Simple port forwarding program implemented using the miniUPnP library
date: '2016-10-28 19:48:59'
tags: upnp miniupnp
permalink: port-fowarder-using-miniupnp/
---

## For a lot of applications today (especially in the IOT and embedded domain), port forwarding is almost unavoidable. Thankfully, there's a way to automate it.

***

### UPnP

The Universal Plug and Play protocol (UPnP) provides a feature to automatically install instances of port forwarding in residential Internet gateways. UPnP defines the Internet Gateway Device Protocol (IGD) which is a network service by which an Internet gateway advertises its presence on a private network via the Simple Service Discovery Protocol (SSDP). An application that provides an Internet-based service may discover such gateways and use the UPnP IGD protocol to reserve a port number on the gateway and cause the gateway to forward packets to its listening socket.<sup>[1](#UPnP-wikipedia)</sup>

---

### MiniUPnP

The MiniUPnP project offers software which supports the UPnP Internet Gateway Device (IGD) specifications. 
This project offers the following softwares:

* MiniUPnP client (`miniupnpc`) - an UPnP IGD control point
* MiniUPnP daemon (`miniupnpd`) - an implementation of a UPnP IGD + NAT-PMP / PCP gateway
* SSDP managing daemon (`minissdpd`): Designed to work with miniupnpc, miniupnpd, minidlna, etc.
* `miniupnpc-async`: Proof of concept for a UPnP IGD control point using asynchronous (non blocking) sockets.
* `miniupnpc-libevent`: UPnP IGD control point using [libevent2](http://libevent.org/).

The MiniUPnP client (`miniupnpc`) and daemon (`miniupnpd`) are portable and should work on any POSIX system.<sup>[2](#MiniUPnP-github)</sup>

---

#### Example

I have modified the MiniUPnP client (`upnpc.c`) to demonstrate how the MiniUPnP library can be used to setup a port forward.
The code is readable and self-explanatory, thanks to MiniUPnP's intuitive API.

<script src="https://gist.github.com/Sufi-Al-Hussaini/a509f055a08539f7aca4f018d7220f3c.js"></script>

---

##### References:

<a name="UPnP-wikipedia">1. </a>[ Wikipedia: UPnP](https://en.wikipedia.org/wiki/Universal_Plug_and_Play)
<a name="MiniUPnP-github">2. </a>[ Github: MiniUPnP](https://github.com/miniupnp/miniupnp)
