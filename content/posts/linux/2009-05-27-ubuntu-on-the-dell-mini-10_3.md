+++
title = "Ubuntu on the Dell Mini 10 (3)"
date = 2009-05-27T16:09:13
tags = ["ubuntu"]
categories = ["Linux"]
published = false
author = "Morten Kjeldgaard"
description = "Ubuntu on the Dell-10 mini"
keywords = "ubuntu"
+++

A few people have asked if the psb X driver is stable running under
jaunty. It is indeed... rock stable. So now that's established, let's
move on to something else.

I really want the Dell to be a "mini-laptop" so I originally installed
the ubuntu-desktop package. But I've felt that the response was a bit
sluggish. The menus were just a few fractions of a second to appear,
they kind of "rolled" on instead of just appearing etc. Not much, but
enough to make it annoying.

So a few days ago, I installed Xubuntu, and it's brilliant! It made a
true, noticeable difference in response. The Mini 10 now appears
really snappy! Xubuntu makes use of the XFCE4 window manager, it's
lightweight compared to the Gnome environment, but still comes with a
bunch of applications like terminal, text editor, etc. etc. I am truly
impressed with the amazing installation the Xubuntu team has created!

I must admit, I am a bit shameful, because I never thought of Xubuntu
as a "real" distribution... just one that could be used on really old
and slow hardware. Boy was I wrong!

The default setup of the Xubuntu desktop, as designed by the Xubuntu
team, is almost exactly like the default Gnome desktop in Ubuntu. I
bet you could exchange someones desktop and (s)he would hardly notice
any difference. All the functionality is there, and it's just as
elegant. An additional bonus for those who don't like Ubuntu's human
theme with its brown/beige colors, Xubuntu's default color scheme is
pretty bluish, very light on the eyes.

I've been running with Xubuntu for a few days now. One thing that I've
noticed -- also when running ubuntu-desktop -- is that the way the
default desktop is set up, with panels at the top and bottom of the
screen is not well suited for the Mini 10's wide 16:9 display. What
you really need is screen real estate in the vertical direction,
because you tend to scroll a lot, for example in Firefox. On my
Kubuntu workstation, I have a 27" Samsung SyncMaster wide screen, and
there I can use the width of the screen to have two applications
running side-by-side. But the Mini 10's monitor is really too tiny to
do that.

Another thing is that the desktop looks exactly like that of a
workstation, with everything scaled down to a tiny size. It looks
neat, but for most people -- and especially netbook users who are not
familiar with Linux -- it's probably not the best setup.

So, I have played a littlebit today with reorganizing the desktop
layout, to make it something of an in-between of UNR ([Ubuntu Netbook
Remix][unr]) )and the standard desktop. Here is what I've come up
with:

<figure>
   <img src="/posts/linux/dell-mini-10-xubuntu-screenshot.png" alt="Xubuntu running a la UNR">
   <figcaption><i>Xubuntu running a la UNR</i></figcaption>
</figure>

The panels now appear on the left and right edges of the screen, where
there is plenty of real estate. The left panel is "controlling". At
the top is the "Applications" menu, then the snapshot applet I used to
make the screenshot (really doesn't belong there). Third and fourth
from the top is "Places" and "Help", which are also on the standard
top panel. Next is the desktop switcher (you really need lots of
desktops with this small screen). At the bottom is the applet to hide
all the application windows, so you can get to the launcher icons on
the root.

The right panel is "informational". From the top, a clock, a weather
applet, the notification window (with Ubuntu One, battery, wifi and
bluetooth monitors), and finally at the bottom (not visible on the
screenshot) I've put the icon box showing running apps on that
desktop.

The root window has some applications grouped in "Network", "Office"
and "System" areas. With a customized wallpaper image with labels and
squares this could be elaborated even further. I have no need for that
though :-).

I am very pleased with the setup as it has developed so far. I think
Xubuntu, installed with lpia architecture packages, and with a
simplified desktop theme is very close to the ideal setup for a
netbook. Perhaps something to consider for the Xubuntu team? An XUNRR
package? (Xubuntu UNR Revisited :-))

[unr]: https://wiki.ubuntu.com/UNR


This post was originally authored for my Wordpress blog
[mok0's world][moks-world], reformatted for inclusion in this blog 2024-06-15.


[moks-world]: https://mok0.wordpress.com/2009/05/27/ubuntu-on-the-dell-mini-10-3/
