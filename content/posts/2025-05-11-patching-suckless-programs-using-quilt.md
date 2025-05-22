+++
title = "Patching suckless programs using Quilt"
date = 2025-05-11T20:39:00+02:00
tags = ["python", "quilt", "suckless", "dwm"]
categories = ["Linux"]
draft = false
author = "Morten Kjeldgaard"
description = "System to patch suckless programs using quilt"
keywords = "suckless quilt python"
+++

I created a system to patch suckless programs using `GNU quilt`. When building suckless programs from source I found that dealing with the patches was confusing and error prone. I therefore made a tool to make looking at patches systematic and reproducible by making use of `quilt`. This tool is used by Debian for patching source packages which is why I know about it.

The advantage of using `quilt` is that you can always reverse the patches and get back to a blank slate, which means you can do `git pull` from the upstream repo and then reapply your patches.

I have written a small Python script that reads a YAML file with the definitions of what patches you want and where to find them, and what priority they have. It's a very simple system that creates a patches directory in the suckless build directory and copies the patches there following a naming scheme that decides in what order patches will be applied.

The git repo is [here on codeberg](https://codeberg.org/mok0/suckless-patches) and there's a tutorial that mainly shows how to use `quilt`. You will notice that the writeup there is pretty much exactly the same as this blog post 😬


## <span class="section-num">1</span> Introduction {#introduction}

The Python script named `suckless-patches.py` can help you download suckless patches and prepare them to be applied using [GNU quilt](https://savannah.nongnu.org/projects/quilt/quilt/) system, that _"allows you to easily manage large numbers of patches by keeping track of the changes each patch makes. Patches can be applied, un-applied, refreshed, and more."_. The `quilt` manpage can be seen [here](https://linux.die.net/man/1/quilt).

In this tutorial, we shall apply patches to the simple terminal `st`, but the system works as well for `dwm` and more. For each program you wish to patch, create a file in YAML format that specifies the patches you want. Here is our example file `st-patches.yaml`.

```shell
---
builddir: ~/Build/st
url: https://st.suckless.org/patches/
patches:
  scrollback:
    patch: st-scrollback-0.9.2.diff
    priority: 40
  fullscreen:
    patch: st-fullscreen-0.8.5.diff
    priority: 10
```

The field `builddir` defines where you have cloned the git repo (or unpacked the `tar.gz` file). The field `url` specifies the base network location of the patches. These are defined in the `patches` dictionary. The name of the patch (e.g. "scrollback") is the component of the url that you find on <https://st.suckless.org/patches> and the `patch` field is the filename in the url. The number in `priority` is rather important, it will translate to the order in which the patches are applied. When the patch is saved to a file, this zero padded, three digit number will be prepended to the patch name.

Note: If a patch has a different URL than the base URL at the top of the file, you can define a "local" one for a single patch, using the key `url`.


## <span class="section-num">2</span> Downloading the patches {#downloading-the-patches}

We have two patches for `st` defined in the YAML file, one for fullscreen and one for scrollback.

Now let `suckless-patches.py` do it's thing. It will create a directory named `patches` in your build directory, and copy the downloaded patches to there, following the naming scheme mentioned above. The script is idempotent, meaning if the patch is already there it won't do anything, except create a new `series` file, but more about that later.

```shell
$ python3 suckless-patches.py st-patches.yaml
2025-05-11 17:58 [INFO] Created directory /home/mok/Build/st/patches
2025-05-11 17:58 [INFO] Downloaded patch: st-scrollback-0.9.2.diff
2025-05-11 17:58 [INFO] Downloaded patch: st-fullscreen-0.8.5.diff
2025-05-11 17:58 [INFO] Adding patch 010-st-fullscreen-0.8.5.diff to series
2025-05-11 17:58 [INFO] Adding patch 040-st-scrollback-0.9.2.diff to series
```

When `suckless-patches.py` has downloaded the files defined in the yaml config file, it will find all `*.diff` files in the `patches` directory, sort them according to their priority number, and write them to the file `series`, which simply lists what patch files `quilt` should apply and in what order. The `series` file will be overwritten every time you run `suckless-patches.py`, but it will include any patches you drop in the patch directory so it shouldn't be a problem.

```shell
$ quilt init
The quilt meta-data is now initialized.
```

This command has created a bookkeeping directory for `quilt` called `.pc`. Specifically check if the file `.pc/.quilt-patches` contains the name of the patches directory. What is written in there depends on the environmental variable `QUILT_PATCHES`, which may or may not be set in `/etc/quilt.quiltrc` or `~/.quiltrc`.


## <span class="section-num">3</span> Pushing the first patch {#pushing-the-first-patch}

Now we push the first patch:

```shell
$ quilt push
Applying patch 010-st-fullscreen-0.8.5.diff
patching file config.def.h
patching file st.h
patching file x.c
Hunk #2 succeeded at 761 (offset 16 lines).
Hunk #3 succeeded at 1246 (offset 19 lines).

Now at patch 010-st-fullscreen-0.8.5.diff
```

That went well, the patch applied fine. However, there was a bit of line offset, lets fix that by refreshing the patch:

```shell
$ quilt refresh
Warning: trailing whitespace in line 770 of x.c
Refreshed patch 010-st-fullscreen-0.8.5.diff
```

There was a bit of trailing whitespace in `x.c`, let's fix that by editing the file:

```shell
$ quilt edit x.c
File x.c is already in patch 010-st-fullscreen-0.8.5.diff
Waiting for Emacs...
```

`Quilt` invokes whatever editer is defined in `EDITOR` environmental variable.  Now refresh the patch again:

```shell
$ quilt refresh
Refreshed patch 010-st-fullscreen-0.8.5.diff
```

At this point the patch is **different from the patch we downloaded** because we have made modifications to what it's doing.


## <span class="section-num">4</span> Pushing the second patch {#pushing-the-second-patch}

Now we continue to the next patch, the one called `040-st-scrollback-0.9.2.diff`. Every time we use `quilt push` it advances to the next patch in the stack and applies it, but only if successful.

```shell
$ quilt push
Applying patch 040-st-scrollback-0.9.2.diff
patching file config.def.h
Hunk #1 FAILED at 201.
1 out of 1 hunk FAILED -- rejects in file config.def.h
patching file st.c
Hunk #15 succeeded at 1350 (offset 3 lines).
Hunk #16 succeeded at 1795 (offset 3 lines).
Hunk #17 succeeded at 2375 (offset 7 lines).
Hunk #18 succeeded at 2388 (offset 7 lines).
Hunk #19 succeeded at 2611 (offset 7 lines).
Hunk #20 succeeded at 2648 (offset 7 lines).
Hunk #21 succeeded at 2714 (offset 7 lines).
Hunk #22 succeeded at 2735 (offset 7 lines).
patching file st.h
Hunk #1 FAILED at 81.
1 out of 1 hunk FAILED -- rejects in file st.h
Patch 040-st-scrollback-0.9.2.diff does not apply (enforce with -f)
```

So we see there were rejections, and because of this, `quilt` has not applied it:

```shell
$ quilt top
010-st-fullscreen-0.8.5.diff
```

The fullscreen patch is still at the top of the stack. So let's force the scrollback patch, so we get some reject files:

```shell
$ quilt push -f
Applying patch 040-st-scrollback-0.9.2.diff
patching file config.def.h
Hunk #1 FAILED at 201.
1 out of 1 hunk FAILED -- saving rejects to file config.def.h.rej
patching file st.c
Hunk #15 succeeded at 1350 (offset 3 lines).
Hunk #16 succeeded at 1795 (offset 3 lines).
Hunk #17 succeeded at 2375 (offset 7 lines).
Hunk #18 succeeded at 2388 (offset 7 lines).
Hunk #19 succeeded at 2611 (offset 7 lines).
Hunk #20 succeeded at 2648 (offset 7 lines).
Hunk #21 succeeded at 2714 (offset 7 lines).
Hunk #22 succeeded at 2735 (offset 7 lines).
patching file st.h
Hunk #1 FAILED at 81.
1 out of 1 hunk FAILED -- saving rejects to file st.h.rej
Applied patch 040-st-scrollback-0.9.2.diff (forced; needs refresh)
```

Now we have two reject files, `config.def.h.rej` and `st.h.rej`. These patches have been rejected because they want to modify code segments that were already modified by our first patch, `010-st-fullscreen-0.8.5.diff`.

Here is the first reject, `config.def.h.rej`:

```diff
--- config.def.h
+++ config.def.h
@@ -201,6 +201,8 @@ static Shortcut shortcuts[] = {
        { TERMMOD,              XK_Y,           selpaste,       {.i =  0} },
        { ShiftMask,            XK_Insert,      selpaste,       {.i =  0} },
        { TERMMOD,              XK_Num_Lock,    numlock,        {.i =  0} },
+       { ShiftMask,            XK_Page_Up,     kscrollup,      {.i = -1} },
+    { ShiftMask,            XK_Page_Down,   kscrolldown,    {.i = -1} },
 };

 /*
```

There is a couple of lines that `quilt` couldn't place, so we add those two lines manually by editing the file. The two lines with `ShiftMask` couldn't be inserted because those lines were already modified by the fullscreen patch. The next reject file, `st.h.rej` looks like this:

```diff
--- st.h
+++ st.h
@@ -81,6 +81,8 @@ void die(const char *, ...);
 void redraw(void);
 void draw(void);

+void kscrolldown(const Arg *);
+void kscrollup(const Arg *);
 void printscreen(const Arg *);
 void printsel(const Arg *);
 void sendbreak(const Arg *);
```

This is two function declaration that we need to put in `st.h`, right around line 81. It's the same problem again, these lines were changed by the fullscreen patch.

So with all problems solved, let's refresh the patch so it will work from now on:

```shell
$ quilt refresh
Refreshed patch 040-st-scrollback-0.9.2.diff

$ quilt top
040-st-scrollback-0.9.2.diff
```

The patch was refreshed (meaning it reflects the changes we made) and it's now on top of the stack. So let's pop off all patches using the `quilt pop -a` command, to go back to where no patches were applied:

```shell
$ quilt pop -a
Removing patch 040-st-scrollback-0.9.2.diff
Restoring st.h
Restoring st.c
Restoring config.def.h

Removing patch 010-st-fullscreen-0.8.5.diff
Restoring x.c
Restoring st.h
Restoring config.def.h

No patches applied
```

At this point **no patches are applied**, if you cloned the git repo you can now check that the files are pristine (except for a bunch of untracked files). Now applying all patches should go without problems:

```shell
$ quilt push -a
Applying patch 010-st-fullscreen-0.8.5.diff
patching file config.def.h
patching file st.h
patching file x.c

Applying patch 040-st-scrollback-0.9.2.diff
patching file config.def.h
patching file st.c
patching file st.h

Now at patch 040-st-scrollback-0.9.2.diff
```

Both patches now applied cleanly, time to see if `st` compiles:

```shell
$ rm config.h
$ make clean st
rm -f st st.o x.o st-0.9.2.tar.gz
cp config.def.h config.h
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9.2\" -D_XOPEN_SOURCE=600  -O1 -c st.c
c99 -I/usr/X11R6/include  `pkg-config --cflags fontconfig`  `pkg-config --cflags freetype2` -DVERSION=\"0.9.2\" -D_XOPEN_SOURCE=600  -O1 -c x.c
c99 -o st st.o x.o -L/usr/X11R6/lib -lm -lrt -lX11 -lutil -lXft  `pkg-config --libs fontconfig`  `pkg-config --libs freetype2`
```

The build completed! Cool!


## <span class="section-num">5</span> Adding a couple of homemade patches {#adding-a-couple-of-homemade-patches}

Now let's change the font, it's too small for me, and it's not my favorite font. I will add a new home made patch to do this:

```shell
$ quilt new 000-font.diff
Patch 000-font.diff is now on top
```

OBS: The file names **must have the suffix** `.diff`.

We also have to **add** a file to this patch. This is important! While `quilt` is a very useful tool, it does not do a lot of handholding. The reason you have to add files to the patch is so `quilt` can stash away pristine copies of it for its bookkeeping, and simply to be able to create the diffs.

```shell
$ quilt add config.def.h
File config.def.h added to patch 000-font.diff
```

The font definition is found in the file `config.def.h`, that's why I added that one. Now I will edit it and introduce my favorite font:

```shell
$ quilt edit config.def.h
```

So, edit, edit, edit, and refresh the patch:

```shell
$ quilt refresh
Refreshed patch 000-font.diff
```

And this is what the diff looks like:

```diff
--- a/config.def.h
+++ b/config.def.h
@@ -5,7 +5,7 @@
  *
  * font: see http://freedesktop.org/software/fontconfig/fontconfig-user.html
  */
-static char *font = "Liberation Mono:pixelsize=12:antialias=true:autohint=true
";
+static char *font = "Monaspace Neon:pixelsize=14:antialias=true:autohint=true"
 static int borderpx = 2;

 /*
```


### Changing the Makefile {#changing-the-makefile}

I find that with suckless programs I always have to delete `config.h` after applying new patches, so I want to modify the `Makefile` to do that automatically.

Let's create a new patch, and add `Makefile` to it:

```shell
$ quilt new 000-makefile.diff
Patch 000-makefile.diff is now on top
$ quilt add Makefile
File Makefile added to patch 000-makefile.diff
```

Now, edit, edit, edit, and refresh the patch, and cat it to review:

```shell
$ quilt refresh
Refreshed patch 000-makefile.diff
```

and this is the new patch:

```diff
$ cat patches/000-makefile.diff
--- a/Makefile
+++ b/Makefile
@@ -24,7 +24,7 @@
$(CC) -o $@ $(OBJ) $(STLDFLAGS)

clean:
-       rm -f st $(OBJ) st-$(VERSION).tar.gz
+       rm -f st $(OBJ) st-$(VERSION).tar.gz config.h

dist: clean
mkdir -p st-$(VERSION)
```

That looks good, I simply added `config.h` to the `clean` target. This file is generated from `config.def.h` if it's not there, so it should never be a problem.

Let's look at out patch stack:

```shell
$ quilt  series
010-st-fullscreen-0.8.5.diff
040-st-scrollback-0.9.2.diff
000-font.diff
000-makefile.diff
```

Everything looks good! I can now pop off all patches (`quilt pop -a`) and push them all again (`quilt push -a`). This is what's in the `patches` directory. You can drop your own patches in there and add them to the `series` file. You don't need to use `suckless-patches.py` you can download directly from the suckless site, but having the YAML file gives you a compact, reproducible recipe, so that is the advantage. But in any case, every time you drop in a new patch, pop off all patches first just to be safe.

```shell
$ tree patches
patches
├── 000-font.diff
├── 000-makefile.diff
├── 010-st-fullscreen-0.8.5.diff
├── 040-st-scrollback-0.9.2.diff
└── series
```

You will notice that the homemade patches `000-*` are listed last in the series file. This is how `quilt` does it, however, if you add another patch to the YAML file and use `suckless-patches.py` again, it will create a new `series` file with all patches in numerical order. The _might_ create problems for you if you are making local edits in the same file that the official patches want to modify, so beware of this.

If a patch causes problems, pop all patches off and move it to the first position of in `patches/series` or simply edit the `series` file to only contain the one problematic patch. But be careful always to make sure what patches in the stack have been applied. The best in my experience is to always start from a "clean slate", meaning pop off all patches before you start to mess with one.


## <span class="section-num">6</span> Conclusion {#conclusion}

`Quilt` provides a systematic and reproducible way to deal with patches. This is one reason that it is used by Debian to manage patches in source packages.

One advantage you will notice is if you need to pull updates to the software from git. You can simply pop off all patches, do a `git pull origin master`, and reapply the patches one by one, solving any problems as you go along.

----
