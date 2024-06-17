---
title:  Convert Launchpad repos to git
date: 2021-11-27
categories: ['Linux']
tags: ['git', 'launchpad']
author: Morten Kjeldgaard
---

This is a short guide on how to convert your Bazaar repos to git on
Launchpad, which is Ubuntu's development server. Bazaar was chosen by
Ubuntu as their revision control system, and the developers did a lot
of work on the Bazaar code, which is written in Python. However,
development stopped several years ago, and Bazaar was no longer
maintained. A spinoff project named [Breezy][breezy] took over, and is
maintaining the code, now invoked by the commend `brz` (but with an
alias to former 'bzr'). Anyway, it's time to convert my archives to
git.
<!--more-->

Select "git" and push "Update at the bottom.

You have to make sure you have pulled the latest version of the
Bazaar repo.

    bzr pull

Stash away the `repo` directory in a tar file:

    tar --xz -cvf repo-save.tar.xz repo-dir

Now initialize the git repo

    cd repo-dir
    git init .

Export the bzr repo logs and everything, and import directly into git, using
the fast-import protocol:

    bzr fast-export --plain . | git fast-import
    git reset --hard

Now save a copy of the `.bzr` directory, and perhaps delete it from the repo:

    tar -cvJf dot-bzr.tar.xz .bzr
    rm -rf .bzr/   # probably not

You can add a remote for your Git repository with the command:

    git remote add origin git+ssh://mok0@git.launchpad.net/mmdb

... and finally push the Git branch to Launchpad with:

    git push origin master


[breezy]: https://github.com/breezy-team/breezy
[mmdb-configure-code]: https://code.launchpad.net/mmdb/+configure-code
