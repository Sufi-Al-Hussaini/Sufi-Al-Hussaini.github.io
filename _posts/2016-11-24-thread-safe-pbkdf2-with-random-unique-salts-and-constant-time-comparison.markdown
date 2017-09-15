---
layout: post
title: Thread safe PBKDF2 with random, unique salts and constant-time comparison
description: A simple C library to store/retrieve your passwords in/from a data store securely 
date: '2016-11-24 19:06:00'
tags: pbkdf2
---

## Thread safe PBKDF2 with random, unique salts and constant-time comparison. 

***

Based on [JP Mens' C implementation](https://github.com/jpmens/mosquitto-auth-plug/blob/master/pbkdf2-check.c) which is itself inspired by [Simon Sapin's scheme](https://exyr.org/2011/hashing-passwords/).

If you're looking for a ready to use C library to store/retrieve your passwords in/from a data store securely, then this library is for you.

##### Code and usage
The code is available on [github](https://github.com/Sufi-Al-Hussaini/pbkdf2-sha256-salt).
OpenSSL is a dependancy and must be installed.

For sample usage look at `example1.c` in the `examples/` directory. Use the `Makefile` provided to build the example statically.
Use CMake to build as a shared library.


##### Credits
My contributions to this project are limited to identifying a [bug concerning thread-safety](https://github.com/jpmens/mosquitto-auth-plug/issues/134) in JP Mens' code and providing a header & sample with openssl locking for thread safe usage; so all credit to JP Mens, Simon Sapin & Kungliga Tekniska Hgskolan (author of `base64.c/h`).