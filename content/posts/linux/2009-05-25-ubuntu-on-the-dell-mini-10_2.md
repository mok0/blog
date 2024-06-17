+++
title = "Ubuntu on the Dell Mini 10 (2)"
date = 2009-05-25T21:20:13+02:00
tags = ["ubuntu"]
categories = ["Linux"]
published = true
author = "Morten Kjeldgaard"
description = "Ubuntu on the Dell-10 mini"
keywords = "ubuntu"
+++

Great news! The Dell Mini 10 now runs on Ubuntu 9.04 at full nominal
resolution of 1024  x  576!

What has happened is that the [Ubuntu Mobile Team][ubuntu-mobile] has
compiled and packaged kernel modules, X.org drivers, libraries to
interface to kernel DRM services, etc. etc. for the Poulsbo chipset
and made them available on their PPA.

Remember from my previous post, that my Mini 10 was running using the
VESA driver for X. All I had to do to switch to the psb driver was to
create the file `/etc/apt/sources.list.d/ubuntu-mobile.list` with the
following content:

```text
deb http://ppa.launchpad.net/ubuntu-mobile/ppa/ubuntu jaunty main
deb-src http://ppa.launchpad.net/ubuntu-mobile/ppa/ubuntu jaunty main
```

Then,

```shell
apt-get update
apt-get install xserver-xorg-video-psb
```

and reboot! The Dell Mini 10 then came alive -- after a bit of
thinking -- with the display at the correct resolution, and the little
display just looked stunningly bright, crisp and sharp!

This is really great news to the many owners of Dell Mini 10 and 12 --
as well as other Poulsbo based netbooks -- that want to run Ubuntu
Jaunty.

Kudos and many thanks to the Ubuntu Mobile Team for their effort and a
job well done!

---

This post was originally authored for my Wordpress blog
[mok0's world][moks-world], reformatted for inclusion in this blog 2024-06-15.


[ubuntu-mobile]: https://edge.launchpad.net/~ubuntu-mobile
[moks-world]: https://mok0.wordpress.com/2009/05/25/ubuntu-on-the-dell-mini-10-2/
