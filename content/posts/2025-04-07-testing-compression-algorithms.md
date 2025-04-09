+++
title = "Testing Compression Algorithms"
date = 2025-03-24T10:49:00+01:00
tags = ["linux", "macos"]
categories = ["Linux"]
draft = false
author = "Morten Kjeldgaard"
keywords = "linux macos compression gzip xz bz2 zstd tar cpio"
+++

**Today I tested out various compression programs that are available on macOS/Linux.**


## The compression programs used {#the-compression-programs-used}

TL;DR: Never mind, take me to [Results](#results)!


### GNU gzip {#gnu-gzip}

[GNU Gzip](https://gzip.org/) is the most used and loved compression program in the Linux/macOS world. Originally written by Jean-loup Gailly and Mark Adler for the GNU project. It has been the standard compression program used for many years in the open source community, and is a well established standard, not least for `tar.gz` archives. It is covered by the [GPL3](https://www.gnu.org/licenses/gpl-3.0.en.html) license. `gzip` typically operates on files with names ending in `.gz`, but can also decompress files ending in `.Z` created with the older  compression utility `compress`. To compress a file, simply give the command:

```shell
gzip file.txt
```

The compressed file will be `file.txt.gz` and it will replace the original file. It is possible to give several filenames on the command line, or simply use a wildcard:

```shell
gzip *.txt
```

This command will compress all files with the `.txt` extension an replace them with their compressed `.txt.gz` counterparts.

To **uncompress** a file, or files, use the `-d` flag, or the command `gunzip`:

```shell
gzip -d file.txt.gz
```

or

```shell
gunzip file.txt.gz
```

The `gzip` suite of programs also contains `zcat` (for piping a file to stdout), `zgrep` (for searching inside a compressed file) and `zmore` (for viewing a compressed file) as well as several other tools for dealing with compressed files.

Read more about gzip on [Wikipedia](https://en.wikipedia.org/wiki/Gzip), or visit the project's [homepage](https://gzip.org/).


### Bzip2 {#bzip2}

[Bzip2](https://sourceware.org/bzip2/) is a widely use data compressor, that was the first challenger to `gzip`'s throne, since its compressed somewhat better. It uses a special block compression algorithm that makes it uniquely useful in big data applications with cluster computing frameworks, because a compressed block can be decompressed in isolation without having to process earlier blocks. The data is compressed in blocks between 100 and 900 kB (default); however the process can be quite memory intensive and may problematic on weaker hardware.

Another unique feature of `bzip2` is its ability to do (limited) error correction on old, corrupt archive, for example from old USB sticks, tapes or floppy disks.

`bzip2` is covered by the BSD-style [bzip2 and libbzip2 License](https://spdx.org/licenses/bzip2-1.0.6.html)

To compress a file or files, the command given are analogous to `gzip`:

```shell
bzip2 file.txt
bzip2 *.txt     # Compress all .txt files
```

To **decompress**  use the `-d` or `--decompress` flags, or use the `bunzip2` program todecompress all files, the following two statements are identical:

```shell
bzip2 -d *.txt.bz2
bunzip2 *.txt.bz2
```

The `bzip2` suite comes with several utilities, that are similar in functionality to those that come with `gzip`, namely `bzcat`, `bzdiff`, `bzgrep`, `bzmore`, and `bzless`.

A couple of programs from the `bz2` suite are also worth mentioning, namely `bzip2recover`  that is capable of recovering data from damaged bzip2 files, and  `bzexe`, which is a utility that allows the user to compressed binaries on the fly and have them automatically uncompressed and executed.

Read more about `bzip2` on [Wikipedia](https://en.wikipedia.org/wiki/Bzip2) or visit the project's [homepage](https://sourceware.org/bzip2/).


### XZ Utils {#xz-utils}

[XZ Utils](https://tukaani.org/xz/), is a complete implementation of the `.xz` file format written in the C language. The suite is covered by the Open Sourcee [BSD Zero Clause License](https://landley.net/toybox/license.html). XZ Utils were originally written for POSIX systems but have been ported to a few non-POSIX systems as well. The core of the XZ Utils compression code is based on [LZMA](https://7-zip.org/sdk.html) but it has been modified significantly to be suitable for XZ Utils.

---

The XZ Utils package provides the command line tools for working with XZ compression,  including `xz`, `unxz`, `xzcat`, `xzgrep`, and so on. The `xz` file format is similar to the older LZMA format but includes some improvements.

The interface of these CLI tools is identical to `gzip` and `bzip2`. To compress a file or files invoke `xz` without flags, the default is to compress:

```shell
xz file.txt
xz *.txt     # Compress all .txt files
```

To **decompress**  `.xz` files use the `-d` or `--decompress` flags, or use the command `unxz`, the following two commands are identical:

```shell
xz -d *.txt.xz
unxz *.txt.xz
```

As the case is with `gzip` and `bzip2` , the XZ Utils suite comes with several utilities, namely `xzcat`, `xzdiff`, `xzless` and `xzmore`, that operate on `.xz` files.

Read more about XZ Utils on [Wikipedia](https://en.wikipedia.org/wiki/XZ_Utils), or visit the project's [homepage](https://tukaani.org/xz/).


### Zstandard {#zstandard}

[Zstandard](https://github.com/facebook/zstdo), normally `zstd` for short, is a fast, lossless compression algorithm written at FaceBook. The suite is covered by the Open Source [BSD 3-Clause License](https://opensource.org/license/BSD-3-Clause), and the compression algorith itself is described in the [RFC8878](https://www.rfc-editor.org/rfc/rfc8878.html) standard. It is a very fast compression algorithm that provides high compression ratios.

The `zstd` compression offers over 20 compression levels, so there are many ways to optimize various use cases. The default compression level is 3, which offers a good compromise between compression and speed.

Compression speed can vary by a factor of 20 or more between the fastest and slowest levels, while decompression is uniformly fast, varying by less than 20% between the fastest and slowest levels.

To compress with `zstd`, the command is identical to the other three programs considered here:

```shell
zstd file.txt
zstd *.txt     # Compress all .txt files
```

the default file extention for compressed files is `.zst`.

To **decompress**  `.zst` files use the `-d` or `--decompress` flags; the following three commands are identical:

```shell
zstd -d *.txt.zst
zstd --decompress *.txt.zst
unzstd *.txt.zst
```

The Zstandard suite also includes severay auxilliary programs for peeking at compressed files, namely `zstdcat`, `zstdgrep`, `zstdless` and others, that all operate on `.zst` files.

Read more about zstd on [Wikipedia](https://en.wikipedia.org/wiki/Zstd), or visit the project's [homepage](https://github.com/facebook/zstdo).


## Compression and `tar` {#compression-and-tar}

Perhaps it's good to mention here that the compression algorithms discussed above are all recognized by the archiving tool `tar`, which is able to read archives compressed with any of the four compression algorithms. Here are the compression options of `tar` that are relevant to this post:

-   `-a`, `--auto-compress`
    Use archive suffix to determine the compression program.
-   `-j`, `--bzip2`
    Filter the archive through [bzip2(1)](https://manpages.org/bzip2).
-   `-J`, `--xz`
    Filter the archive through [xz(1)](https://manpages.org/xz).
-   `-z`, `--gzip`, `--gunzip`, `--ungzip`
    Filter the archive through [gzip(1)](https://manpages.org/gzip).
-   `--zstd` Filter the archive through [zstd(1)](https://manpages.org/zstd).

So to create a backup of your entire Documents folder you could use:

```shell
tar -czf documents.tar.gz Documents
```

or

```shell
tar --zstd -cf documents.tar.zst Documents
```

in these, `-c` means "create" a new archive. To extract these archives again you would use the `-x` flag::

```shell
tar -xf documents.tar.zst
```

`Tar` is able to deduce the type of compression used from the file extension, so it's not necessary to specify it.


## Compression and `cpio` {#compression-and-cpio}

In this experiment I have used `cpio`, another Unix archiving utility which is similar to tar, but different.  Instead of specifying a directory, as you do with tar, you feed `cpio` with a list of files via standard input. This means you can use other utilities to generate a list of files you want to back up. In this example, I am looking for all files with the `.tex` extension in `Documents/`, I might not want everything in `Documents/`, only the LaTeX documents. `cpio` by default outputs its archive to `stdout`, so it can be piped into a compression program that further pipes it into a file. `zstd` is easy because you don't need to specify any flags, it's the default behaviour when reading and writing to a pipe. With `gzip`, for example, we would have to specify `gzip -dc` there.

```shell
find ~/Documents -name "*.tex" | cpio -o | zstd > ~/backups/tex-documents-$(date --iso).cpio.zstd
```

The `-o` switch (or `--create`) tells `cpio` that it will be receiving a list of files on `stdin` and it should create an archive. To unpack, one would use the `-i` (or `--extract`) switch, telling `cpio` that it's receiving an archive on `stdin` and it should extract those files.


## Results {#results}

As the basis of this experiment, I created a `cpio` backup of a mixed set of pure text and binary data such as PDF's, of size 1.1 Gb. All runs where made on the same file using the default mode of these programs (no options to compress, `--decompress` to decompress), I did not attempt to optimize the performance (for example `gzip` has both `--fast` and `--best` options). All tests were run on my good old Thinkpad T520 from 2011. As a matter of fact, all my computers are old and slow, or are Raspberry Pi's, which was the reason I started to look at different compression algorithms in the first place.

| file             | algorithm | size (Mb) | % size | time (s) | decomp (s) | speed (Mb/s) |
|------------------|-----------|-----------|--------|----------|------------|--------------|
| archive.cpio     |           | 1171.3    | 100.0  |          |            |              |
| archive.cpio.gz  | gzip      | 718.7     | 61.3   | 55.1     | 9.7 (18%)  | 13.0         |
| archive.cpio.bz2 | bzip2     | 700.2     | 59.8   | 144.0    | 72.2 (50%) | 4.9          |
| archive.cpio.xz  | xz        | 606.6     | 51.8   | 433.7    | 46.2 (11%) | 1.4          |
| archive.cpio.zst | zstd      | 672.9     | 57.4   | 9.6      | 2.9 (31%)  | 70.3         |

The column **size** is the size of the file after compression in Mb, and the column **% size** is the size of the compressed file compared to the uncompressed file. The column **time** is the wall clock execution time in seconds, the the column **decomp** is the time to decompress the resulting compressed file again. The percentage in that column is how fast decompression is compared to compression. FInally, the column labelled **speed** is how effective the compression process was, or in other words, how many megabytes compressed data the algorithm can deliver per second. The number is calculated by dividing numbers in the **size** column by the **time** column.

The tigthtest compression is done by `xz`, the resulting file is 51.8% of the original. Next comes `zst` with a compressed file that is ~66 Mb larger. Number 3 is `bzip2`, and the least compressed file is by `gzip`, it is 112 Mb larger than the winner of the compression category, and results in a file that is 66.3% of the original, uncompressed  file.

Looking at execution time, `zst` is the huge out-of-category winner. It took only 9.6 seconds to compress the archive of 1.1 Gb on this 14 year old computer, it is genuinely impressive! On second place, at 55.1 seconds comes... none other than `gzip`. Third place is `bzip2` at 144 seconds and finally we have `xz` at a whopping 433.7 seconds.

All the programs tested here are faster at decompressing than compressing the file. The winner of this category is `zstd` again at 2.9 seconds, corresponding to 31% of the time it took to compress. At second place comes again `gzip`, with a decompression time of 9.7 seconds, 18% of its compression time. On third place, at 46.2 seconds,  `xz` takes revenge on losing the compression round. However, as we can see, at 11% `xz` is _much_ faster at decompressing files than it is at compressing them. Last place at 72.2 seconds comes `bz2`.

FInally, lets look at the compression efficiency. Not surprisingly, the overall, out-of-category winner is `zstd` at 70.3 Mb/s. Next comes `gzip` at 13 Mb per second then `bzip2` and last is `xz`.


## Conclusion {#conclusion}

The results show that `zstd` is by far the most efficient compression algorithm. While `xz` does slightly better compression, it's also 50 times slower, and `zstd` is second best in compression. So overall, at the default level of compression, `zstd` is the winner. If you really need fast and effective compression/decompression, where portabiliy is not the primary focus, there isn't really a choice.

As a number 2 comes `gzip`. You probably didn't expect that, did you? There's a reason that `gzip` is still the most popular and still has widespread use. This is a very good reason to continue to use `gzip`, you can reasonably expect the compressed file to be decompressed anywhere, and in addition, a huge number of other applications can deal with the `.gz` format. It is also the default format of the Debian project, used to compress the software in the `.deb` packages.

Number 3 is `bzip2`. Relative to the other programs it's not particularly good at compressing files and it's not particularly fast. However, `bzip2`'s ability to uncompress archives asynchronously as well as its ability to restore corrupt archives gives it a niche in practical use.

The program that is able to do the absolute best compression is `xz`, but it does so at a big price in execution time. At a compression speed at only 1.4 Mb/s It comes last in compression efficiency. For the 1.1 Gb archive in this experiment, `xz` saves 66 Mb compared to the runner up, `zstd`. I disk space is very important to you, and you have plenty of CPU power, then `xz` is a good choice. The Arch Linux project uses `xz` as the compression used in their compiled packages, which are in `.tar.xz` format. It means their software archives are smaller in size, and since `xz` decompresses very fast, it works well for the the end-user of those packages.

My initial problem was that I chose `xz` to compress backups from my Raspberry Pi's with the `xz` program, and it simply took forever. Then I started to look at these other programs and found that for my use case, `zstd` is outside any competition the best choice.

----
