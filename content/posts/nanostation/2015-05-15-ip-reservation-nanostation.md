---
title: Reservation of IP address on Nanostation M5
date: 2015-05-15T12:30:13+02:00
published: false
categories:
  - nanostation
author:
  - Morten Kjeldgaard
---

> Note: From AirOS version 5.6.9 and forward, the following is no
> longer necessary, because you can reserve IP addresses directly from
> the Nanostation's web interface. 2017-01-11.

The Nanostation M5 from [Ubiquity](https://www.ubnt.com/) has a
comprehensive and beautiful web GUI interface that enables
customization of a large number of settings. Unfortunately,
reservation of IP addresses for specific hosts on the local network in
the Nanostation's DHCP server is not possible via the GUI. This is a
bit strange, because the GUI does allow port-forwarding, an option
which is a bit useless without the possibility to reserve an IP
address on the LAN to forward traffic to.

Instead, you need to specify the DHCP reservations manually by making
a small change in the Nanostation's operating system. Log on to the
Nanostation from a terminal:

    ssh admin@192.168.50.252

The Nanostation runs a tiny Linux system ("AirOS"), and many of the
standard UNIX tools are available
via [busybox](http://www.busybox.net).
Fortunately,
[AirOS allows users to add scripts](http://wiki.ubnt.com/Manual_Routes) in
the `/etc/persistent` directory of the Ubiquiti device.

Create a file `/etc/persistent/reservations.conf` containing the
DHCP reservations:

    # raspberrypi
    dhcp-host=b8:27:eb:d2:7a:da,192.168.50.222
    # my-laptop
    dhcp-host=5c:96:9d:76:42:29,192.168.50.223

These statements will ensure that the device with the given Mac
address is given the same IP address every time it joins the network.
In order to prevent that other devices are given these adresses, they
should be chosen outside the range of IP addresses normally handed out
by the DHCP daemon. This range is defined in the Nanostation's web
GUI.

The file `/etc/dnsmasq.conf` is rewritten every time the Nanostation
boots, taking parameters from the GUI interface, so it is useless to
edit it manually. Create a boot-time script
`/etc/persistent/rc.prestart`:

    RES=/etc/persistent/reservations.conf
    if test -r $RES; then
       cat /etc/persistent/reservations.conf >> /etc/dnsmasq.conf
    fi

Do a save:

    M.v5.5.10# save

(The command `save` is an alias: `save='cfgmtd -w -p /etc/'`)

Now, you can kill the `dnsmasq` daemon, it is restarted automatically
by init (it is listed in `/var/etc/inittab`). You can also reboot the
Nanostation:

    M.v5.5.10# reboot

When the Nanostation is back up, reboot your device or simply renew
the DHCP lease. Below is shown how this is done in Mac OSX 10.10.

![Renew DHCP lease, Mac OS X 10.10](/posts/nanostation/2015-05-15-renew-lease.png)
