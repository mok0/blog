+++
title = "Revisiting the zstd compression algorithm"
date = 2025-04-14T19:42:00+02:00
tags = ["linux", "macos", "compression"]
categories = ["Linux"]
draft = false
author = "Morten Kjeldgaard"
description = "Testing compression programs 2"
keywords = "linux macos compression zstd gzip bzip2 xz"
+++

A few days ago, [I wrote about various compression algorithms](/posts/2025-04-07-testing-compression-algorithms/) and how they perform. Today, I am looking further into the 19 different compression levels available in `zstd`. I ran the experiment in exactly the same way, using the same cpio archive and the same computer.

`zstd` has 19 different compression levels, and in the experiment detailed below they are named "zstd-1", "zstd-2", etc. up to "zstd-19". For purposes of comparison, I have included results from `gzip`, `bzip2` and `xz` from last time.

The data is detailed is in the [table below](#table--data), but let's look at it in graphical form. In the graph below, I have plotted the compression time in seconds along the X-axis, and the size of the compressed file in percentage of the initial size along the Y-axis.

The left panel shows all the runs. At the top left the uncompressed file is indicated by a yellow dot at (0, 100). Then in green, all the `zstd` runs are shown. The result from `gzip` is shown with a blue dot, `bzip2` is magenta dot and `xz` is a red dot. The compression ratio drops very quickly and then flattens out almost in an L-shape.

In order to show some of the `zstd-*` results more clearly, the right panel shows a detail of the left panel, as indicated by the grey rectangle.

When advancing from level 1 to 4, `zstd` delivers increasingly better compression without much cost in terms of run-time. Levels 5-8 do not seem compress any better than level 4, but the execution time increases. The next performance jump comes at level 9 which seems to compress equally efficient to levels 10-16, that all require significantly longer run time. Levels 17-19 delivers a couple of percent better compression, again at the cost of run time.

It seems the default `zstd` level which is set at 3 is a decent compromise between compression ratio and run time. One could consider using level 4, or perhaps level 9 that seems to stand out.

<img alt="Screenshot of terminal with fastfetch output" src="/img/results_2025-04-14-14-33.png"/>


## Here's the data {#here-s-the-data}

<a id="table--data"></a>

| file             | algorithm | size (Mb) | % size | time (s) | decomp (s) | speed (Mb/s) |
|------------------|-----------|-----------|--------|----------|------------|--------------|
| archive.cpio     |           | 1171.3    | 100.0  | 0.0      | 0.0        | inf          |
| archive.cpio.gz  | gzip      | 718.6     | 61.3   | 56.0     | 9.9  (19%) | 12.8         |
| archive.cpio.zst | zstd-1    | 705.3     | 60.2   | 6.8      | 2.9  (43%) | 103.7        |
| archive.cpio.zst | zstd-2    | 688.6     | 58.8   | 7.1      | 3.0  (42%) | 97.0         |
| archive.cpio.zst | zstd-3    | 672.7     | 57.4   | 9.5      | 2.9  (31%) | 70.8         |
| archive.cpio.zst | zstd-4    | 668.5     | 57.1   | 14.2     | 3.1  (22%) | 47.1         |
| archive.cpio.zst | zstd-5    | 670.7     | 57.3   | 24.9     | 3.0  (12%) | 26.9         |
| archive.cpio.zst | zstd-6    | 668.0     | 57.0   | 26.6     | 2.9  (11%) | 25.1         |
| archive.cpio.zst | zstd-7    | 658.5     | 56.2   | 34.5     | 3.0  (9%)  | 19.1         |
| archive.cpio.zst | zstd-8    | 656.9     | 56.1   | 38.6     | 3.3  (10%) | 17.0         |
| archive.cpio.zst | zstd-9    | 635.8     | 54.3   | 50.1     | 3.1  (6%)  | 12.7         |
| archive.cpio.zst | zstd-10   | 634.6     | 54.2   | 53.8     | 3.3  (6%)  | 11.8         |
| archive.cpio.zst | zstd-11   | 634.2     | 54.1   | 73.2     | 3.2  (4%)  | 8.7          |
| archive.cpio.zst | zstd-12   | 634.1     | 54.1   | 75.2     | 3.2  (4%)  | 8.4          |
| archive.cpio.zst | zstd-13   | 634.2     | 54.1   | 152.3    | 3.3  (2%)  | 4.1          |
| archive.cpio.zst | zstd-14   | 633.9     | 54.1   | 137.6    | 3.3  (2%)  | 4.6          |
| archive.cpio.zst | zstd-15   | 633.4     | 54.1   | 159.2    | 3.3  (2%)  | 4.0          |
| archive.cpio.zst | zstd-16   | 627.9     | 53.6   | 207.9    | 2.9  (1%)  | 3.0          |
| archive.cpio.zst | zstd-17   | 609.5     | 52.0   | 300.6    | 3.3  (1%)  | 2.0          |
| archive.cpio.zst | zstd-18   | 610.7     | 52.1   | 379.9    | 3.9  (1%)  | 1.6          |
| archive.cpio.zst | zstd-19   | 606.1     | 51.7   | 476.6    | 4.8  (1%)  | 1.3          |

----
