+++
title = "The walpal utility for creating color themes"
date = 2025-05-20T01:50:00+02:00
tags = ["python", "hyprland", "desktop"]
categories = ["Linux"]
draft = false
author = "Morten Kjeldgaard"
description = "walpal create colorschemes"
keywords = "python colors desktop hyprland"
+++

This is the writeup of a new utility I have written, inspired by `Pywal`. You can find it [on Codeberg](https://codeberg.org/mok0/walpal).

---

This project is heavily inspired by Dylan Araps' [Pywal](https://github.com/dylanaraps/pywal) application. Dylan archived the github repository in April 2024, it has since been forked by github user `eylles` and expanded for 16 colors in the project [Pywal16](https://github.com/eylles/pywal16).

However I found that `Pywal16` gave me two sets of identical colors, not 16 unique. It seemed color1 and color8 where identical, color2 and color9, and so on. I tried examining the source code to find out why, but honestly I had trouble understanding it. So I decided to write my own walpaper analysing program that does not rely on external tools, but relies solely on Python modules.

`Walpal` is a lot simpler and limited than `Pywal` and it also works in a somewhat different way. Rather than analysing the background image on the fly, `walpal` must first preprocess the image to determine the palette of dominant colors as the first step in a two stage process. It goes on to save the color palette in the user's `~/.cache` directory. In the following, I will use the words "palette" and "theme" interchangeably. It depends on whether we're talking about a set of RGB colors, or if it's something you activate to change the appearance of the screen.

Once the dominant colors of a background image has been generated, `walpal` can activate the palette similar to what `Pywal` does, and `walpal` can do this very quickly.

I found this two stage process to not be a practical problem, as all it takes is to run the preprocessing on the directory containing images, e.g. `/usr/share/backgrounds/`.

Images are read using the [Python Imaging Library](https://pypi.org/project/pillow/) (pillow), that is capable of reading a great number of different image formats. However, `walpal` expects images to containmeaning three channel RGB or four channel RGBA data that also includes a transparency layer.

Images are then analysed by the Kmeans algorithm in the [scikit-learn](https://scikit-learn.org/stable/) module. The palette data is further saved in a [Pandas](https://pandas.pydata.org/) dataframe, which is saved in the user's `~/.cache` area in the portable [Apache Parqet](https://parquet.apache.org/) format, and contains rgb, hls and luminocity data for each color. Additionally the palette colors are saved in a json file for easy access.


## <span class="section-num">1</span> Preprocessing images {#preprocessing-images}

It is possible to preprocess a single image using the `--image` switch:

```shell
$ walpal --image van-gogh-starry-night.jpg
[INFO] Processing van-gogh-starry-night
```


## <span class="section-num">2</span> Preprocessing a directory {#preprocessing-a-directory}

You can preprocess all images in a directory in one go.

```shell
$ walpal --directory /usr/share/backgrounds/linuxmint-wilma
[INFO] Preprocessing /usr/share/backgrounds/linuxmint-wilma
[INFO] Processing mpiwnick_atacama
[INFO] Processing jcorl_eclipse
[INFO] Processing ztasi_moon_phases
[INFO] Processing meiying_body_of_water
[INFO] Processing mfakurian_abstract
[INFO] Processing mnohassi_morocco
[INFO] Processing jcorl_white_sands
[INFO] Processing aksenapati_snow_mountain
[INFO] Processing mpiwnicki_torres_del_paine
[INFO] Processing pblache_colors
[INFO] Processing kwinegeart_road
[WARNING] Not an image file: Credits
[INFO] Processing jcorl_monument_valley
[INFO] Processing mpiwnicki_palm
[INFO] Processing aksenapati_blanket
[INFO] Processing slee_blocks
[INFO] Processing ztassi_england
[INFO] Processing jpanchal_cpu
[INFO] Processing mpiwnicki_sunset
[INFO] Processing mnohassi_ocean
[INFO] Processing slee_earth
[INFO] Processing mpiwnicki_chalet
[INFO] Processing mnohassi_boat
```

From now on, the palettes (or "themes") are identified by their file name without extension, like it's listed above.

All the information `walpal` gathers while processing the images are stored in the file `~/.cache/walpal/walpal.db`. If you delete this file it will automatically be regenerated, but `walpal` will forget all information about what images have been processed and will not be able to activate the themes, so you will have to preprocess the background images again. If you want `walpal` to forget about a theme, use the `--delete` option (see below). The `walpal.db` file is a Python shelve file, but it's actually in `sqlite` format so you can also manipulate it with `sqlite3` if you know how.


## <span class="section-num">3</span> Activating a theme {#activating-a-theme}

To activate a theme, simply call `walpal` with the `--palette` option, and as argument use the palette name:

```shell
$ walpal --palette slee_earth
```

If you forget what themes/palettes  you have available, they can be listed with the `--list` switch:

```shell
$ walpal --list
mpiwnick_atacama
jcorl_eclipse
ztasi_moon_phases
meiying_body_of_water
...

mpiwnicki_chalet
mnohassi_boat
```

NB: If you need to output the list of themes, for example in a shell script, this construct can be useful:

```shell
themes=$(walpal --list)
```

When you activate a theme, `walpal` finds the current theme and optionally sends control characters to all open terminals changing the ANSI colors&nbsp;[^fn:1].  That's all it does by default, if you need it to do more, see the following sections.

The change of colors are not persistent, however, and they can be reset to their defaults using the teminal initialization command `reset` (see  `reset(1)`). If you want the theme colors to be persistent in the terminal, you need to do that via templates, see below.

If you do not want `walpal` to touch the terminal colors, use the `--no-tty` flag.


### Running scripts {#running-scripts}

As we saw above, `waypal` only generates the color palette and temporarily sets terminal colors. However, when `walpal` runs, it will look for scripts to run in the directory `~/.config/walpal/scripts/`. The scripts need to have the extension `.sh` and they need to be executable. They are executed in globbing order, if you want to be systematic about it, you can name them with leading numerals like this: `10-set-wallpaper.sh`, then they will be executed in that order. The aforementioned script might look like this:

```shell
#!/usr/bin/bash
source $HOME/.cache/walpal/current-theme.sh
swww img --transition-type any "$WALLPAPER"
```

It illustrates the use of the file `current-theme.sh` that `walpal` creates whenever it activates a theme and stores in the `.cache/walpal/` directory. The content of this file is very simple:

```shell
# Generated by walpal 2025-05-22 17:42 CEST
THEME="slee_earth"
WALLPAPER="/usr/share/backgrounds/linuxmint-wilma/slee_earth.jpg"
```

As an inspiration, here is another useful script called `50-waybar.sh` that you can use to restart `waybar` so it uses the active theme, more on how to accomplish that in the next section.

```shell
#!/usr/bin/bash
pkill waybar
waybar &
```


### Templates {#templates}

When `walpal` activates a theme, it does several tings. In the directory `~/.cache/walpal/` it:

-   Open the database `walpal.db` to see if it knows the theme.
-   Reads the pregenerated `parquet` file and generates a color dictionary from it.
-   Saves this dictionary in `colors.json`.
-   Saves a CSS file with the current color palette in `colors.css`.
-   Creates the aforementioned shell source file `current-theme.sh`

The next thing `walpal` does when activating a theme is to generate theme files for other applications, and places them in your directory `~/.cache/walpal`.

The templates must be found in your config directory `~/.config/walpal/templates/`. The template files **must** be in [Jinja2 format](https://pypi.org/project/Jinja2/). **OBS** This is different from the template format used in `Pywal`. The important difference if you want to port your `Pywal` templates is that variables are inclosed in double braces, like this `{{ color3 }}`. Here is an example for use with `waybar`:

```css
@define-color foreground {{ foreground }};
@define-color background {{ background }};
@define-color cursor {{ cursor }};
@define-color color0 {{ color0 }};
@define-color color1 {{ color1 }};

... etc

@define-color color14 {{ color14 }};
@define-color color15 {{ color15 }};
```

This template will be converted and placed in `~/.cache/walpal/colors-waybar.css`. In other words, it retains the same name, but will expanded and placed in the `~/.cache/walpal` directory. In the proper `waybar` style sheet, that is placed whereever `waybar` finds its style (`~/.config/waybar/style.css`). You then need to include the palette in that file with something analogous to this:

```css
@import url("/home/mok/.cache/walpal/colors-waybar.css");
```

Be aware that `walpal` orders the colors in the palette according to their luminocity. That means that when it generates a dark theme (option `--dark`) the darkest colors will be color1, color2, etc, and the lightest colors will be color13, color14 and color15. When generating a light theme, the ordering of colors is reversed, so the light colors have the lowest numbers and the dark colors have ... 13, 14, 15. The background and foreground colors are chosen from the darkest and lightest colors, but made a bit darker and a bit lighter, depending whether the theme is dark or light.

If you would like to see and study the palette, you can run `walpal` with the `--display=` flag, which generates an overview image and calls you system image display program to show it. If you want to save the image, you can do it from there. If you want to study the hexcodes from the pallete, you can find the json format files for all themes that have been activated in the directory  `~/.cache/walpal/palettes`.

**Finally**, after all you templates have been processed, `walpal` runs all scripts found in `~/.config/walpal/scripts` like described above. Of course this means that anything done by these scripts is done with the current, last activated theme.


## <span class="section-num">4</span> Installing walpal from source {#installing-walpal-from-source}

If you've come this far, you probably already know what to do, but just in case, to download the source code of `walpal`, clone it using `git`:

```shell
git clone https://codeberg.org/mok0/walpal.git
```

Next, if you haven't already, you need to install the Python build module:

```shell
pip install build
```

then cd into the `walpal` source directory and:

```shell
python -m build --wheel
pip install dist/walpal-0.8-py3-none-any.whl
```

`walpal` requires the following external packages:

-   `pandas`
-   `numpy`
-   `jinja2`
-   `pillow`
-   `scikit-learn`

but `pip` will automatically fetch and install them for you.

Pip will install `walpal` as an executable script, you may have to modify your `PATH` environmental variable for the shell to find it.


## <span class="section-num">5</span> Command line options {#command-line-options}

```shell
walpal --help
usage: __main__.py [-h] [--image IMAGE] [--directory DIRECTORY] [--light]
                   [--dark] [--force] [--quiet] [--no-tty] [--debug] [--list]
                   [--palette PALETTE] [--display] [--delete DELETE]
                   [--version]

walpal -- Generate colorschemes from background images

options:
  -h, --help            show this help message and exit
  --image, -i IMAGE     Preprocess single background image
  --directory, -d DIRECTORY
                        Preprocess image directory
  --light               Generate a light colorscheme.
  --dark                Generate a dark colorscheme (default).
  --force               Force (re)preparation of image.
  --quiet, -q           Be quiet.
  --no-tty, -t          Be quiet.
  --debug               Output debug info.
  --list                List all information.
  --palette, -p PALETTE
                        Set active palette.
  --display             Show the active palette.
  --delete DELETE       Delete palette from database.
  --version             Print walpal version and exit.
```

----

[^fn:1]: This piece of code is adopted from `Pywal` with thanks.
