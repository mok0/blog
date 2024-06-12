---
layout: post
title: How to use ssh authentication keys
date: 2006-05-06 14:18
created: 1146917880
modified: 1146920624
categories:
  - linux
author:
  - Morten Kjeldgaard
---

If you have an account on a remote computer that you use very
often, it is convienient if you do not have to type your password
every time you log on to another host. You can achieve this by using
ssh's authorization keys.

First, you need to generate a key-pair. A key-pair consists of two
small files. One file contains your _secret_ key, the other your
_public_ key. Your secret key is, as the name implies, secret. You
want to make sure noone gets hold of it. The public key is not secret
at all, you can post it on your web-page if you wish. To generate the
keypair, use the __ssh-keygen__ program:

    [somebody@monster ~]$ ssh-keygen -t rsa -b 4096
    Generating public/private rsa key pair.
    Enter file in which to save the key (/u/somebody/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /u/somebody/.ssh/id_rsa.
    Your public key has been saved in /u/somebody/.ssh/id_rsa.pub.
    The key fingerprint is:
    5a:0c:9d:c8:2b:aa:4d:1c:db:3e:ca:e2:1e:43:27:3c
    somebody@monster.bioxray.dk

Notice, I did not enter a passphrase. The passphrase is an optional
password that protects your secret key. It is a password you have to
type every time your secret key is used, but your security is only
compromised if someone gets hold of your secret key, so you must
protect that file using mode `600`. Now, the two files have been
created in `$HOME/.ssh`:

    [somebody@monster ~]$ ls .ssh
    authorized_keys id_rsa id_rsa.pub known_hosts

To achieve login on a remote machine without password, you need to
append your public key to the `.ssh/authorized_keys` file on the
remote machine. That is, if you specified a passphrase for your
key-pair, you have to type _that_ when you log on, but not the
password on the remote machine.

On bioxray, all users have NFS mounted home directories, so we can
just do this:

    cp .ssh/id_dsa.pub .ssh/authorized_keys

and we can use `slogin` and `ssh` between all hosts without specifying
a password.

You can also use the ssh authentication key scheme to give access to
your account to a friend. Your friend must also generate a key-pair,
and must send you the public part of it. You append that public key to
your `authorized_keys` file. Your friend can then log on to your
account, as long as the public key is in the file.

If you have an account on a remote computer that you use very often,
it is convienient if you do not have to type your password every time
you log on to another host. You can achieve this by using ssh's
authorization keys.

## Edit 20170123

* Changed key type from rsa to dsa 4096 bits. For further information
look [here](https://help.ubuntu.com/community/SSH/OpenSSH/Keys).
* Add note about passwordless secret key.
