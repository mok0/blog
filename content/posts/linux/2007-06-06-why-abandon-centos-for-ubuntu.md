---
layout: post
title: Why abandon CentOS for Ubuntu?
date: 2007-06-06 13:49:49 GMT
# Wed, 06 Jun 2007 13:49:49 GMT
created: 1181137789
modified: 1181137789
published: false
categories:
  - linux
author:
  - Morten Kjeldgaard
---

## My Linux experience

My first Linux experience was around 1996 with the slackware
distribution installed from floppies. I quickly went on using RedHat
3.03, and went all the way. When RedHat stopped their free version we
went to Fedora Core 1. However, I quickly got fed up with Fedora's
perpetual upgrade-circus. It was just too much work upgrading all the
time, and with no apparent gains (other that eye-candy stuff, but who
cares). So when RedHat EL4 appeared some years ago we jumped to CentOS
4.

So, for a couple of years, we've had a constant platform (EL4) on
which to base our (1) own software development, and (2) adapting other
peoples software (mainly for macromolecular crystallography and
bioinformatics). But along the way, I have become aware of the
inherent weakness of the RedHat/CentOS platform for our needs.

## Why jump to Ubuntu/Debian?

CentOS is based on RedHat, so the base of the distribution is RedHat's
packages, which also means that the packaging policy is defined by
RedHat. However, they only supply ~2500 packages, which is not
sufficient for our use. We very often need software that is not
available from RedHat. This is where the 3rd party repositories
started playing a role.

A few years ago -- in my naiveté -- I had a whole bunch of repo's
defined in my sources.list.d directory. I really believed in the "let
a thousand repos blossom" idea. However, gradually, I had to remove
repos from the list, because software in our systems started breaking.

Finally, I was down to just two repos when one day, I came in and
discovered that a lot of our local software -- that relied on fftw2 --
did not work anymore because one repo suddenly decided to provide
fftw-3, and that package replaced fftw-2 and crashed our local
software that _needs_ fftw2.

It is
_necessary_ to solve these problems if people are to rely on updating
a system for long periods of time. But RedHat's and Fedora's basic
strategy is to supply a system that you have to upgrade regularly,
replacing all the base software in one big swoop, and then compile all
add-on software again. The constant update-cycle of Fedora disguised
this fact for a while, but it has become visible during the two-year
CentOS period.

This is when I realized that
CentOS is never going to work for us. We can't continue using third
party repos, but this problem is not going away anytime soon and
people don't seem to care about it.

With over 10 years of experience packaging RPMs it has not been easy
to come to the conclusion that Debian is the way to go. It is not the
RPM format _per se_. Although Debians packaging mechanism is a bit
more advanced that for RPMs, I believe that the RPM packaging system
is sufficiently capable, and it _is_ easier to use, and has a gentler
learning curve. The problem is RedHat's packaging policy. Basically,
RedHat has ~2500 packages that they care about. The rest is added by
third party repos who basically make up their own policies. If at all.

## My first Ubuntu experience

In the meantime, I have had an Kubuntu system at home for six months
now, and it has been an amazing experience. There are over 20K
packages available from the repos, and I have not _once_ run into
problems. I can have different packages, that rely on different
versions of the same libraries, installed side-by-side. It _just
works_. I did a dist-upgrade from etchy to feisty without a single
problem. In addition, I have access to all the multimedia packages
that I care to install, the latest KDE/Gnome environments, games, and
so on (which is what our users ask for on the workstations ;-)).

So I decided that rather than porting my repo to EL5, I would port it
to Debian and be in with Flynn :-)

## What packaging means to the lab

We currently have around 25 workstations, some are in a big room in
the basement, some are on people's personal desks. We also have a
small HPC cluster with 5*2 CPUs. All users have an NFS mounted home
directory which is sitting on one of two large diskarrays. The other
diskarray is on a server in a remote building and is used for backups.
We use rsnapshot to save snapshots from the last 40 days. The backup
array is mounted on a virtual partition so every user can visit their
own backup directory.

All workstation have the exact
same configuration of software, this is achieved by using cfengine,
apt, and a few cron scripts.

To keep this setup running without a large support team we absolutely
need to have all software packaged. Until now, I have packaged
everything in RPM packages. Proprietary software -- which we
unfortunately in some cases are forced to use -- is also packaged, but
stored on an apt partition which can not be reached from outside our
network.

At the time of writing (6. june 2007) I have begun porting my RPM
packages to .deb format, and when that process is completed, we will
start a gradual move from CentOS to Kubuntu on the workstations. The
servers will probably be running Debian stable.
