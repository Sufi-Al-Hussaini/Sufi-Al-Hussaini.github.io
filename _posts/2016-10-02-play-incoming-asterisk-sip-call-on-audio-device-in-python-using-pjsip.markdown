---
layout: post
title: Play incoming SIP call on audio device 
description: Implemented in python using PJSIP library
date: '2016-10-02 18:59:41'
tags: asterisk banana-pi
permalink: play-incoming-asterisk-sip-call-on-audio-device-in-python-using-pjsip/
---

## Directing SIP call audio to audio device 

***

Recently, I was asked to implement a sort of public announcement system for a banana pi variant running asterisk.

Sure there is the built-in asterisk public announcement system, which will let you make announcements using the phone speakers (if supported and enabled); but the requirement in my case was different. 

Basically, there was to be a designated SIP extension. Users wanting to make a public announcement would call this extension & make their announcement. This was to be played on the banana pi speaker in realtime. 

I started searching online for a solution and came across the PJSIP library.
I couldn't find a ready-made solution and started to code my own using the PJSIP library. 

Here's the code: 
<script src="https://gist.github.com/Sufi-Al-Hussaini/6d00be6316013fdde5e5ed20549ebbef.js"></script>