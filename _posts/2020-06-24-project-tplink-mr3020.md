---
title:  "Project: TPLINK MR3020"
layout: post
---

In which I feebly hack on a cheap TPLINK MR3020 router.

## Project description

I  bought this router because I wanted to play around with some hardware. I'd seen a few people flashing it with OpenWRT via easily installed UART headers, and overall it seemed like a pretty tame base to mess with.

I expected to drop into a root shell within about 10 minutes of unbooxing it, naturally.

It turned out that I got the new version, which has tiny unsoldered pads (TP1 and TP2 below) for the UART connection rather than holes for pins.

![My board](../assets/images/20200617_200710.jpg?raw=true)

I'm no stranger to soldering but I was concerned that the size of the pads would lead me to bodge the job and somehow break things.

I booted up the router and checked out the web UI. There's a firmware upload feature, so I decided a more fun plan would be to backdoor the firmware, upload it, and see how much trouble I could get up to that way, and *then* to solder some connectors onto the tiny pads. If I broke it at that point, no harm done.

## Project goals

- Write usermode backdoor
- Write kernel module with basic backdoor
- Extend kernel module with extra special rootkit functionality (optional)
- Solder stuff on to the tiny pins and drop into a shell that way
- Vulnerability hunt!
- Find a cool final form for it ([like turning it into an OpenVPN router](http://blog.prototypecreations.net/2017/05/17/tp-link-tl-mr3020-portable-openvpn-router/)) and set it up

## Why it's cool

I'm currently super interested in a) what's possible with hardware hacking, b) deeply understanding IoT/embedded internals, c) returning to the bosom of C programming, d) learning how to write rootkits/kernel modules, and e) learning how to find vulnerabilities and exploit them! 

This little project allows me to pursue all of this at once in a pretty cohesive way, and the hardware is quite a versatile base for making something fun at the end, too.

## Documentation and stuff
Some relevant links for this device:

- [Firmware](https://www.tp-link.com/au/support/download/tl-mr3020/)
- [GPL code](https://www.tp-link.com/au/support/download/tl-mr3020/#GPLCode)
- [SoC doc](http://download.villagetelco.org/hardware/MT7620/MT7628%20datasheet.pdf)
