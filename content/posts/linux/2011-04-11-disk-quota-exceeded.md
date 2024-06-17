+++
title = "Disk Quota Exceeded?"
date = 2011-04-11T21:11:14+01:00
tags = ["ubuntu", "quotawarn"]
categories = ["Linux"]
published = true
author = "Morten Kjeldgaard"
description = "Utility to send notice to users if they are above disk quota"
keywords = "ubuntu quotawarn"
+++

The disk quota system in Linux is quite old, and if you have a
distributed workstation environment with NFS mounted shares it is not
possible in a convenient way to inform users that they have exceeded
their disk limits.
<!--more-->

So we created a simple disk quota warning system, written in Python,
that works using the notification interface (libnotify). A user who
has exceeded the disk quota is then at regular intervals presented
with a pop-up warning like the one shown in the figure below.

<img src="/posts/linux/warnbox.png" style="margin: 1rem;"/>

The quotawarn system is divided into three parts. Two of them run on
the file server, and one on the users workstation, under the users’
own UID.

On the file server, there is a program called `quotawarn.cron`, which
(duh!) should be run at regular intervals by cron. This program simply
runs repquota and squirrels away the information in a convenient
format. Currently, this is simply a pickled Python dictionary indexed
by UID, but could of course also be a fancier database storing
historical data on each user’s disk usage.

The second program on the server is a CGI script. Using the standard
web interface lets us rely on the access settings in the HTTP server,
thus minimizing the need to program a special security system, and
relieving the burden on the sysadm of learning and maintaining yet
another access control system.

`Quotawarn.cgi` responds to messages from the network, replying to
requests for specific UIDs by transferring the data for that user in
JSON format. Or, optionally, by creating an HTML page with the data of
all users.

On each workstation, the notification script `quotawarn` is started when
the user logs on. The sysadm can achieve this by placing an entry in
`/usr/share/autostart`. The program will ask for data from `quotawarn.cgi`
on the file server, and if the user is under the disk quota limit, the
program will exit without any more noise. If, however, the users disk
quota is exceeded, the program will pop up a notification (shown
below). This will continue at regular intervals (set by the sysadm).

A second notification system, yet to be developed, could hook into
Ubuntu’s `motd` system, and provide the necessary information when the
user logs on via an ssh connection to the network.


<img src="/posts/linux/diagram-small.png" style="margin: 1rem;"/>

I have created a [project on Launchpad][quotawarn] for quotawarn in the hope that
others would like to contribute to the project, and make it more
generally usable. Although an attempt has been made to make the system
general, we have not been able to test it in other settings than our
own. So, if this has whet your appetite, please feel free to branch
the project and contribute!

---

This post was originally authored for my Wordpress blog
[mok0's world][moks-world], reformatted for inclusion in this blog 2024-06-17.


[quotawarn]: https://edge.launchpad.net/quotawarn
[moks-world]: https://mok0.wordpress.com/2011/04/11/disk-quota-exceeded/
