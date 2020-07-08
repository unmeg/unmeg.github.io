---
title:  "DIMELAB: ZJ-024 ATX Power Supply Breakout Adapter"
layout: post
---
![ZJ-024](../assets/images/20200708_105853.jpg?raw=true)

No router project update this week as I've been consumed by some regular engineering stuff for uni.  However, I finally received this little breakout board in the mail today and I thought I'd list the running output voltages for anyone who might be considering it.

## Numbers

Attached to a 550W PENQUIN power supply, I get:

+12V: 12.10V

-12V: -11.24V (6.3% error)

+5V: 5.18V (3.6% error)

+5V(standby): 5.08V (1.6% error)

+3.3V: 3.33V

It seems nice and stable. I watched it on the meter for a minute, and then powered a blinky test circuit for a while. Got a perfect, constant 9V off a voltage divider and the 12V rail.

![ZJ-024](../assets/images/20200708_104645.gif?raw=true)

## Verdict

This thing cost me $8 (including shipping) and an old ATX I had stashed in a drawer. Build quality is good, it functions well enough (most rails within 5% of the goal). Winner.
