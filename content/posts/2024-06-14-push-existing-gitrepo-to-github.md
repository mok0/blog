+++
title = "How to push an existing repo to github"
date =  2024-06-27T22:20:00+02:00
tags = ["git", "github"]
categories = ["git"]
draft = false
author = "Morten Kjeldgaard"
description = "How to push existing repo to github"
keywords = "git github"
+++

An existing git repo on your own machine, for example in a directory called `myproject`. Make sure everything in the repo is committed to a branch.

<!--more-->

```shell
$ git status
On branch main
nothing to commit, working tree clean
```

Create an empty repository on github, through the web interface. Get the address of the github repo using the green button. Then back on your own system:

```shell
$ git remote add origin https://github.com/youruid/myproject.git
```

that will tie your existing git repo to the remote. Now you need to pull from github, but no worries, the directory up there is empty, unless you chose to create a `README.md` file when creating the empty repo.

```shell
$ git pull
```

Now github is ready to recieve an upload of your repo:

```shell
$ git push origin main
```

iff `main` is the branch you wish to push. You can also use `git push --all` to push all branches.
