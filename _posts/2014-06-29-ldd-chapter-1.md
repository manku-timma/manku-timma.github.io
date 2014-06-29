---
layout: post
title:  Linux Device Drivers
date:   2014-06-29 16:50:00
tags: linux, device drivers
---

I started reading [LDD3][ldd3]. Here are some notes.

A device driver implements mechanisms to support user policies about devices.
The important choices to be made by a driver writer are:

- allow device to be opened multiple times
- decide how to handle concurrent requests
- allow sync and async apis

Some of the functionality can be in a userland library and some in a userland
config program. Not everything needs to be in the kernel driver.

[ldd3]: http://lwn.net/Kernel/LDD3/
