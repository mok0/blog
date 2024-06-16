---
title: Ubuntu on the Dell mini 10 (1)
date: 2009-05-22 12:46:18
categories: ['Linux']
tags: ['ubuntu']
published: true
---

I recently purchased a Dell Mini 10 notebook. I've been wanting to get
a netbook for some time, and I was planning to one of the Asus EEE
series. However, looking at one at the local computer store, I found
it a bit "plastic-y". I've always heard that Dell computers have a
good build quality, so one morning, on a whim, I ordered a Dell. I
told the salesman that I had no use for Windows XP. He said there was
no price on it, but gave me a discount of around 26 Euro. My suspicion
is that Microsoft allows Dell to install XP for free.

Now that I have it, I'm a bit disappointed in the Dell's build
quality. I find the Mini 10 a bit plastic-y too. But then, I'm used to
the solid feel of my PowerBook G4, and in build quality, nothing comes
close to Apple I guess.

Apart from that, the Mini 10 has some really nice features, among
others I really like the touchpad without buttons. From the Mac, I am
used to using gestures on the touchpad, and I can reveal that it works
nearly perfectly under Ubuntu 9.04. I also really like the keyboard,
which is about the same size as my [Cherry 4100 keyboard][cherry]
4100 keyboard</a> that I use on my workstations. In fact, everything
works... except the graphics... but more on that later.

Initially, I was confused by the fact that Ubuntu recommends that you
install the UNR version on all netbooks, but that is for the i386
architecture only, and I wanted to use lpia, because it's optimized
for the Atom processor and has been reported to run better, and give
better battery life.

So instead of UNR, I chose to download the [MID version of
Ubuntu][ubuntu-mid] which installs lpia architecture packages.

I put the image file on an USB key following the [instructions on the
download page][instructions]. I then booted up the Mini 10 with the
pre-installed XP -- just to make sure that it worked. When the XP
installation came to the point where you have to accept the license,
it felt REALLY good to hit the "No" button :-). The XP install
process then rebooted the Mini 10 from the USB key.

So I installed the Ubuntu-MID version, zapping XP with 2 ext4
partitions `/` and `/home` (plus 5Gb swap). Everything went very
smooth and the Mini 10 sprung to life.

The MID version by default installs a very simple window manager; it
seems to work fine but was unfamiliar to me (and after all, the Mini
10 is not MID device) so I installed ubuntu-desktop. It works very
well. (I am actually a Kubuntu user, but KDE4 is not at all suited for
the limited resolution on the notebook.)

Every worked out of the box: WiFi, bluetooth, sound, even the little
camera. BUT, the machine runs at resolution 800 x 576 so images and
fonts look a bit "squished". The Mini 10 is based on the Poulsbo
chipset and Intel's GMA-500 graphics chip. There's been a lot of
chatter on the net on the lack of Linux drivers from Intel for these
chips. You can google and see for yourself. The situation for Linux is
pretty bad, since the Poulsbo chipset is appearing in more and more
netbook computers.

To cut a long story short, X does not recognize the GMA-500 graphics
chip, so it uses the VESA driver. The native resolution on the Mini 10
is 1024 x 576, but the VESA driver refuses to use that resolution. I
have tried various things, i.e. specifying the modeline, but to no
avail. X always returns to 800 x 576. So for now, I've settled and am
using that resolution. Another thing is that there's no acceleration
and glxgears runs pathetically slow (25-30 fps) but the Mini 10 is
perfectly usable as a netbook despite of this.

There's an [effort to get drivers for the Poulsbo chipset][poulsbo] in
shape, hopefully that will result in something soon.

- - -

This post was originally authored for my Wordpress blog
[mok0's world][moks-world], reformatted for inclusion in this blog 2015-05-15.

- - -
[cherry]: https://www.cherry-world.com/g84-4100
[ubuntu-mid]: https://canonical.com/blog/ubuntu-mid-edition-804-achieves-its-first-public-release
[instructions]: https://help.ubuntu.com/community/Installation/FromImgFiles
[poulsbo]: https://blueprints.edge.launchpad.net/ubuntu-mobile/+spec/poulsbo-packaging
[moks-world]: https://mok0.wordpress.com/2009/05/22/ubuntu-on-the-dell-mini-10/
