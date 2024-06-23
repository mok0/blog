+++
title = "Sacrosanct Linux Feature Dies"
date = 2012-03-23T19:07:13+01:00
tags = ["ubuntu"]
categories = ["Linux"]
published = true
author = "Morten Kjeldgaard"
keywords = "ubuntu virtual terminal"
+++

I've been using Linux since 1996. Since then, the OS has undergone an
amazing development, and all distributions provide an impressive
high-resolution graphical interface. However, one feature has remained
sacrosanct: the 6 virtual terminals. In the old days, when you had to
provide timings for the video card and manually edit the xfree86
config file, it was easy to mess up the graphics display. But then,
CTRL-Alt-F1 to the rescue! It was ALWAYS possible to get a terminal
and consequently access to the operating system. And, in addition,
most of the GUI versions of system setup programs had a TUI analogue,
that could be run from the 80 x 24 terminal.

Until now. Ubuntu 12.04 BREAKS the virtual terminals on many older
video cards, because it insists in using frame buffer mode, presumbly
to provide fancy, meaningless, silly graphics for the boot screen.
This is what my virtual terminals looks like now:

<figure>
    <img src="/posts/linux/ubuntu-12-04-virtual-terminal.jpg", style="width: 400px; margin: 1rem;" />
</figure>

This is a scandal, no more, no less! Breaking the virtual terminals,
that ALWAYS have been available, no matter what video card you had in
your computer, breaks the promise that you always can obtain a console
to control Linux. Simply a very, very bad design decision.

---
This post was originally written for my wordpress blog [MOK's world][moks-world]
and included here on 2024-06-17.

[moks-world]: https://mok0.wordpress.com/2012/03/23/sacrosanct-linux-feature-dies/
