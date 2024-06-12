---
layout: post
title:  Install newer version of Ruby on OS X Yosemite
date: 2015-05-02 16:00:00
categories:
  - osx
author:
  - Morten Kjeldgaard
---

Following this video: <https://youtu.be/jx0NrIbQbzI>
```sh
$ brew update
```
First install `rbenv`:
```sh
$ brew install rbenv
$ brew info rbenv
$ emacs .bashrc
```
Execute (and add to `.bashrc`):

    export RBENV_ROOT=/usr/local/var/rbenv
    if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi

Now we have `rbenv` installed. See if it works:

    $ rbenv

We need to install the `ruby-build` plugin. Clone the git directory
directly into the `rbenv` plugin directory in the [homebrew][1]
managed tree:

    $ cd /usr/local/var/rbenv/
    $ mkdir plugins
    $ cd plugins/
    $ git clone --depth=1 git@github.com:sstephenson/ruby-build.git
    $ rbenv

`rbenv` now has some new commands, e.g. install. What versions of Ruby
are available?

    $ rbenv install --list

Install newest one (2.2.2 as of now):

    $ rbenv install 2.2.2
    $ ruby --version
    ruby 2.0.0p481 (2014-05-08 revision 45883) [universal.x86_64-darwin14]

Still running system Ruby version, use `rbenv` to switch:

    $ rbenv global 2.2.2
    $ ruby --version
    ruby 2.2.2p95 (2015-04-13 revision 50295) [x86_64-darwin14]

Now we are running 2.2.2. Install the `bundler` gem:

     $ gem install bundler

[Bundler][2] provides a consistent environment for Ruby projects by
tracking and installing the exact gems and versions that are needed.

[1]: http://brew.sh
[2]: http://bundler.io
