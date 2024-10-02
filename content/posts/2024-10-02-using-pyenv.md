+++
title = "Using pyenv to manage your Python versions"
date = 2024-10-02T21:03:00+02:00
tags = ["emacs", "org-mode"]
categories = ["Python"]
draft = false
author = "Morten Kjeldgaard"
description = "Manage Python version with pyenv"
keywords = "python pyenv environment tutorial"
+++

In this blog post I will explain how I use `pyenv` to manage my Python installation. The following is only of relevance if you develop Python programs.

Most -- if not all -- Linux distributions come with a system version of Python, and the most important reason to include it as a system package is to run Python code from other packages.

If you develop Python programs you find you'll need to install other Python modules. Many are available as system packages, but far from all. If you search the web, you'll find that people tell you to install those packages using `pip`, but if you try to do that, you'll find that you aren't allowed to install your own modules to the system version of Python... for good reason. Another option is to add the `--user` switch to `pip`, and that will install the packages somewhere in your `~/.local` directory hierachy, but that quickly becomes unmanageable as the system version of Python with time is updated, while your user-local version of packages aren't. Or perhaps you'll find you suddenly need multiple versions of Python modules installed.


## <span class="section-num">1</span> Pyenv to the rescue {#pyenv-to-the-rescue}

`Pyenv` is a system that lets you install your own Python versions, that live in your home directory tree, more explicitly in `~/.pyenv`. You can install any version of Python, there are actually hundreds of various Python versions you can install using this system. What actually happens when you install something using `pyenv` is that is fetches the source code, compiles a special version for your machine, and puts it somewhere in `~/.pyenv`. When you activate Python from now on, it will not (necessarily) invoke the system version of Python, but the one you have installed using `pyenv`.

But wait... isn't that an enormous waste of valuable disk space?

Well, it depends on what your needs are. A full installation of Python including a handsome bunch of extra modules might take up 1 Gigabyte of space, but that is no more than many flatpak programs you install take. So if you need it, you need it. Another advantage is that you might want a newer --or older-- version of Python than the one your distro ships with.

Here, I will not go into how to install `pyenv`, just go [here](https://github.com/pyenv/pyenv) and follow the instructions. Basically, you install the software (which lives in `~/.pyenv`)  with `git` and put some shell commands in your `.bashrc` or `.zshrc` file to initalize `pyenv`.


## <span class="section-num">2</span> Install your first version of Python {#install-your-first-version-of-python}

To see what versions of Python are available, you can now list them all:

```shell
$ pyenv install --list
```

As of writing, this generates a list of 842 different versions available to you. But to install Python version 3.12.6, which is the newest stable version, simply go:

```shell
$ pyenv install 3.12.6
```

`Pyenv` goes ahead and downloads the file and compiles it. You'll have to first install the tools to compile a and build C program, on the Debian family of distros, this package is called `build-essential`.  After the build has completed, you can take a look at what Python versions are avaible in `pyenv`:

```shell
$ pyenv versions
  * system
  3.12.6
```

The one with an asterisk is the currently active version. This is my system version of Python at the moment:

```shell
$ python3 --version
Python 3.11.2
```

To activate the version we just installed, try this:

```shell
$ pyenv shell 3.12.6
$ python3 --version
Python 3.12.6
```

Now the shell you are running is using Python version 3.12.6. If you open another terminal window, you will still be running the system version, so the `shell` keyword means that it's only set for that session. There are ways to make version 3.12.6 permanent, using either the `local` (local to a particular directory) or `global` (everywhere) keywords. Check out the `pyenv` documentation for more on this.


## <span class="section-num">3</span> Next step: Install plugins {#next-step-install-plugins}

Before going further, I recommend that you install a few `pyenv` plugins. Here are the two most important plugins you'll need to begin with:

-   Pyenv [virtualenv](https://github.com/pyenv/pyenv-virtualenv.git), enables and manages virtual environments in `pyenv`
-   Pyenv [update](https://github.com/pyenv/pyenv-update.git), updates `pyenv` and all installed plugins

    To install, go to the directory `~/.pyenv/plugins` and clone the git repos of the plugins you want:

<!--listend-->

```shell
$ cd ~/.pyenv/plugins
$ git clone https://github.com/pyenv/pyenv-virtualenv.git
$ git clone https://github.com/pyenv/pyenv-update.git
```

When you have installed these plugins, a few extra commands will be available, `pyenv help` shows all of them, but the one you'll need now is `virtualenv`, that creates a Python virtual environment. What is a virtual environment? Very simply put, it is not much more than a separate copy of the `site-packages` directory, where `pip` will install all third-party Python packages.


## <span class="section-num">4</span> Always use a virtual environment {#always-use-a-virtual-environment}

I recommend _never_ install third party packages in your newly created Python directory. I like to keep this version minimal and pristine. Instead, I prefer to create virtual environments based on that version. It's possible to create as many as you want. Many people use a particular virtual environment for a particular project. Or, for example, if you want to play with a Python package that you don't know if you really want, you could create a virtual environment called `junk` and install the package there. Some Python packages pull in a great number of dependencies, and if you regret, there is no simple way to get rid of those 50 dependencies. But if you isolate your work to a virtual environment, you can use `pyenv  virtualenv-delete` to get rid of `junk` again.

So before going any further, I suggest to create a virtual environment based on 3.12.6.

```shell
$ pyenv virtualenv 3.12.6 env-3.12.6
```

Now, change to the new environment:

```shell
$ pyenv shell env-3.12.6
```

Look at what we now have:

```shell
$ pyenv versions
  system
  3.12.6
  3.12.6/envs/env-3.12.6
.* env-3.12.6 --> /home/mok/.pyenv/versions/3.12.6/envs/env-3.12.6 (set by PYENV_VERSION environment variable)
```

`Pyenv` lists the new virtual environment twice for some reason, but there only is one. The files are in this directory:

```shell
$ ls -F ~/.pyenv/versions
3.12.6/  env-3.12.6@
```

Now you can go ahead and install all the packages you need with `pip`, and they will go into the virtual environment called `env-3.12.6` (or whatever name you give it):

```shell
$ pip install numpy
```


## <span class="section-num">5</span> Updating your installation {#updating-your-installation}

After a while, new Python version appear, but they will not be shown by your `pyenv` unless you update it (in reality, pull from the `pyenv` git repo). Here, the `update` plugin we installed above comes in handy. All you need to do at regular intervals is to:

```shell
$ pyenv update
pyenv update
Updating /home/mok/.pyenv...
remote: Enumerating objects: 128, done.
remote: Counting objects: 100% (107/107), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 25 (delta 15), reused 21 (delta 11), pack-reused 0 (from 0)
Unpacking objects: 100% (25/25), 4.85 KiB | 236.00 KiB/s, done.
From https://github.com/pyenv/pyenv
 * branch              master     -> FETCH_HEAD
 * [new tag]           v2.4.14    -> v2.4.14
   3f2ef9e0..468dc811  master     -> origin/master
Updating 3f2ef9e0..468dc811
Fast-forward
 .github/workflows/modified_scripts_build.yml                     | 3 +--
 CHANGELOG.md                                                     | 4 ++++
 libexec/pyenv---version                                          | 2 +-
 plugins/python-build/share/python-build/3.12.7                   | 9 +++++++++
 plugins/python-build/share/python-build/3.13.0rc2t               | 2 --
 plugins/python-build/share/python-build/{3.13.0rc2 => 3.13.0rc3} | 4 ++--
 plugins/python-build/share/python-build/3.13.0rc3t               | 2 ++
 7 files changed, 19 insertions(+), 7 deletions(-)
 create mode 100644 plugins/python-build/share/python-build/3.12.7
 delete mode 100644 plugins/python-build/share/python-build/3.13.0rc2t
 rename plugins/python-build/share/python-build/{3.13.0rc2 => 3.13.0rc3} (59%)
 create mode 100644 plugins/python-build/share/python-build/3.13.0rc3t
Updating /home/mok/.pyenv/plugins/pyenv-doctor...
From https://github.com/pyenv/pyenv-doctor
 * branch            master     -> FETCH_HEAD
Already up to date.
Updating /home/mok/.pyenv/plugins/pyenv-pip-migrate...
From https://github.com/pyenv/pyenv-pip-migrate
 * branch            master     -> FETCH_HEAD
Already up to date.
Updating /home/mok/.pyenv/plugins/pyenv-update...
From https://github.com/pyenv/pyenv-update
 * branch            master     -> FETCH_HEAD
Already up to date.
Updating /home/mok/.pyenv/plugins/pyenv-virtualenv...
From https://github.com/pyenv/pyenv-virtualenv
 * branch            master     -> FETCH_HEAD
Already up to date.
```

Notice that there are already updates to `pyenv` even though I just did an update a few hours ago in preparation for this posting.


## <span class="section-num">6</span> Migrate your set of packages {#migrate-your-set-of-packages}

You can keep your virtual environments around; I prefer to use the same virtual environment everywhere until i decide to upgrade to a new Python version. So for a while, I wil be using `env-3.12.6`. But eventually, I will want to use a newer version of Python, and that is where the `pyenv-pip-migrate` plugin is really handy. It lets you copy all installed packages from one virtual environment to another. You can find the `migrate` plugin [here](https://github.com/pyenv/pyenv-pip-migrate.git), just clone it using `git` like we did above.

So say I wanted to try out the new Python version 3.13.0rc3. I would compile and install the Python binary like before, create a virtual environment, and then migrate all my `pip` installed packages like this:

```shell
$ pyenv migrate env-3.12.6 env-3.13.0rc3
```

When in the new environment, I might update the most important packages with a script that calls `pip` like this:

```shell
#! /bin/bash

PACKAGES="ipython pandas numpy scipy seaborn matplotlib pillow
SQLAlchemy pyarrow pytz pytest openpyxl loguru jupyter jupyterlab
beautifulsoup4 colored yamllib"

for p in $PACKAGES; do
    echo Upgrading: $p
    pip --quiet install --upgrade $p
done
```

----
