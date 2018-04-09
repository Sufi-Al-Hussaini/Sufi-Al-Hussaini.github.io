---
layout: post
title: A CLI tool for bulk provisioning of Grandstream IP phones
description: Without the need to manually create a cfg.xml file
date: '2017-12-23 16:16:23'
tags: tool ip-phone bulk-provisioning php sip 
---

## A CLI tool for automatic discovery and bulk provisioning of Grandstream IP phones. 
The script takes an IP range as input, then crawls it looking for Grandstream IP phones and is capable of automatically determining the phone model. It then configures the phone with requested settings based on this information.

***

Currently 3 models are supported: 
* GXP1628
* GXP2170
* GXV3240

The script works by using Grandstream's HTTP API, which associates every setting on the web UI with a short-code (eg `P64` for `timezone`).  

#### Note: 
Grandstream HTTP API isn't documented and what you see here is a essentially a reverse-engineering approach. Therefore, there are no guarantees that this will work for you or that this will continue to work in the future. 

**Read the code first and use with caution!** 

<script src="https://gist.github.com/Sufi-Al-Hussaini/7d40e4ff17d8f65bf35f6146d3e55e27.js"></script>
