+++
title = "Install Arch Linux from scratch on Mac mini"
date = 2025-01-07T23:42:00+01:00
tags = ["emacs", "org-mode"]
categories = ["Linux"]
draft = false
author = "Morten Kjeldgaard"
description = "Install Linux on Mac mini"
keywords = "linux arch macmini"
+++

This is a description of how I  installed Arch on my Mac mini (ultimo 2012) that I purchased very cheaply refurbished. This machine has 16 Gb of RAM and 128 Gb SSD and it was expensive when it was first purchased. It is not possible to upgrade this machine past macOS Catalina, but with Linux the support will continue forever, and this little computer can continue to run a modern, fully secure and updated operating system.

Before I had fun doing this, I actually installed Linux Mint 21 on the same hardware, it took aournd 10 minutes and everything worked including wifi after using Mint's driver manager to install the driver, which it found automatically. However, I wanted to try to install Linux on btrfs subvolumes and that is why I started this project. I will need this machine for teaching installation of Linux, so I will wipe the Arch installation again soon, but I had fun doing this, and perhaps I'll repeat this installation when the teaching is done.. I installed Arch several years ago on a different machine, but I can comfort you by saying that it's still a lot of work 😬.

A few things in this document is specific to Macmini6,1, notably the things related to the wireless interface, otherwise you can probably use most of the following for any computer.


## <span class="section-num">1</span> Flashing a USB stick {#flashing-a-usb-stick}

Always use the most recent ISO image from the [Arch Linux download site](https://archlinux.org/download/), otherwise you are very likely to get into problems later.

I first put the Arch ISO on my Ventoy disk, but when I got to the stage of creating the grub installation I ran into a problem because apparently `efivars` was not active on `archiso`. I assume this is because the image is booted by Ventoy and not by UEFI on the Mac. Anyway, when later booted directly from a USB stick, the problem went away.


## <span class="section-num">2</span> Set terminal font size and keyboard layout {#set-terminal-font-size-and-keyboard-layout}

I am using my TV as display, so I need to increase the font size. I also need to change the keyboard layout. The layout codes can be listed using  `localectl list-keymaps`.

```shell
setfont -d
loadkeys dk
```


## <span class="section-num">3</span> Connecting to WiFi {#connecting-to-wifi}

The wireless interface on the Mac mini ("Macmini6,1")  is a BCM 4331 chip which is not by default supported by `archiso`.  I could find the interface using:

```shell
lspci -nn -d 14e4:
```

but when looking at the net interfaces using `ip link`, only the loop and wired interfaces were show. When booting, `archiso` tells you to go to [this link](https://wireless.wiki.kernel.org/en/users/Drivers/b43#devicefirmware), but [this is the correct one](https://wireless.docs.kernel.org/en/latest/en/users/drivers/b43.html#list-of-hardware). After a lot of web searching, trial and error and many reboots I found that I need to blacklist the b43 driver and load wl which is on `archiso` from the start.

```shell
modprobe -r b43 wl
modprobe wl
systemctl restart iwd
```

`iwctl` now shows the device `wlan0`, and I am back to the normal tutorials. However, `ip link` shows the device in the state UNKNOWN.

The next step is to run `wpa_supplicant`.  First I generated a `wpa_supplicant.conf` file:

```shell
wpa_passphrase Kanhavehus2 "Elvin&Enzo" > wpa_supplicant.conf
```

then I ran this command from the wiki:

```shell
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

After this I need to restart `dhcp`:

```shell
dhcpcd wlan0
```

Now the system received an IP address from my DHCP server and `ip link` shows wlan0 as UP, and I can ping 192.168.2.1.


## <span class="section-num">4</span> Create partitions {#create-partitions}

Now the network is up, so I can continue by creating partition. Not going into details with this, but these are the partitions I created:

-   `/dev/sda1` 300 Mb FAT32
-   `/dev/sda2` 8 Gb swap
-   `/dev/sda3` 111 Gb btrfs

The final partition `/dev/sda3` is simply the remaining free space on the 128 Gb SSD that came with the Mac mini.

Many guides tell you to use `zramswap` instead of having a swap partition, but I am old fashioned and I want a swap area outside `btrfs`, and disk space is much cheaper than RAM space.

You _could_ create an additional 4 Gb ext4 partition for `/boot` but here I am creating a `@boot` subvolume for that. You want `/boot` to be isolated from `/` in order to create snapshots that don't create problems when restoring. In fact, I've often thought about unmounting `/boot` once the system is running since you don't need it (except when updating `grub`).


## <span class="section-num">5</span> Create the btrfs main file system {#create-the-btrfs-main-file-system}

I want to use `btrfs` subvolumes so the setup becomes a bit involved and confusing. Basically the `btrfs` main file system can be thought of as an `LVM` volume group that you put logical volumes on. Having a granular set of subvolumes enables you to use snapshots in a rational way, but contrary to `LVM`, the subvolumes all share the total device space with the advantages and disadvantages that entails: You can make use of all available disk space, but if one of the subvolumes fills up, like for example your `$HOME`, the whole system locks up.

The first thing to do is to create all the subvolumes I want. `/dev/sda3` is already formatted as a btrfs volume using `mkfs.btrfs`, so just mount it:

```shell
mount /dev/sda3 /mnt
```

Now create subvolumes:

```shell
btrfs subvolume create @         # for /
btrfs subvolume create @boot     # for /boot
btrfs subvolume create @varlog   # for /var/log
btrfs subvolume create @cache    # for /var/cache
btrfs subvolume create @home     # for /home
```

The reason to keep `/var/log` and `/var/cache` in separate subvolumes is again when restoring `/` from a snapshot, the system retains logs etc. and in addtion, the files in these areas are constantly updated with information and would take up disk space in the snapshot.

Now, unmount `/dev/sda3` and remount the root subvolume (@):

```shell
umount /dev/sda3
 mount -o rw,noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ /dev/sda2 /mnt
```

Create all the mount points needed:

```shell
mkdir /mnt/home
mkdir -p /boot/efi
mkdir -p /var/log
mkdir -p /var/cache
```

and mount all the subvolumes. I found that you actually don't need to include all the options above, it will work fine with this:

```shell
mount /dev/sda3 -o subvol=@ /mnt
mount /dev/sda3 -o subvol=@boot /mnt/boot
mount /dev/sda3 -o subvol=@home /mnt/home
mount /dev/sda3 -o subvol=@varlog /mnt/var/log
mount /dev/sda3 -o subvol=@cache /mnt/var/cache
```

Finally mount the efi partition:

```shell
mount /sda1 /mnt/boot/efi
```

Now we have the naked directory structure needed to install a Linux system mounted on `/mnt`.


### TIP {#tip}

I had to boot from `archiso` many times, and every time it has forgotten everything, except the filesystems we have generated on `/dev/sda3`. I created a few small shell scripts that I store on a second USB stick, and it can be mounted while in `archiso` environment, and I could then copy the scripts from the USB stick to the local `/root.` Fortunately the Mac mini has 4 USB ports to it's possible to connect 2 USB sticks, a keyboard and a mouse.


## <span class="section-num">6</span> Preparing for the Arch installation environment {#preparing-for-the-arch-installation-environment}

Before doing `pacstrap` it's a good idea to refresh the signing keys, _especially_ if your install image is not brand new, the signing keys of the Arch devs seem to expire all the time.  This operation might take quite a while.

```shell
pacman-key --refresh-keys
```


## <span class="section-num">7</span> Bootstrapping a basic Linux system {#bootstrapping-a-basic-linux-system}

Now that we have our partitions mounted, let’s install the set of base packages for Arch. I suppose you could use `debootstrap` if creating a Debian installation instead of Arch.

```shell
pacstrap -i /mnt base
```


## <span class="section-num">8</span> Enter the chroot environment {#enter-the-chroot-environment}

There is nothing else we need to do in the `archiso` environment right now, so we switch root to the newly bootstrapped Linux system:

```shell
cd /root
arch-chroot /mnt
```


## <span class="section-num">9</span> Mount the swap partition {#mount-the-swap-partition}

Next step is creating and mounting the swap partion in our new Linux system. As seen above I created an 8 Gb partition for swap, now activate it:

```shell
mkswap /dev/sda2
swapon /dev/sda2
```


## <span class="section-num">10</span> Generate an fstab file {#generate-an-fstab-file}

Now we have all subvolumes plus swap mounted, so generate an `/etc/fstab` file to save all this information for the next boot:

Generate an fstab and place it in `/etc`:

```shell
genfstab -U > /etc/fstab
```

The `-U` switch prompts `genfstab` to output UUIDs instead of device names that might change. Inspect the `/etc/fstab` file to check that all the information is correct, and that all our partitions are listed in the file.


## <span class="section-num">11</span> Set zoneinfo and hostname {#set-zoneinfo-and-hostname}

Just set the timezone, it's also possible to wait and do it via the Cosmic settings, but we might as well do it now we're here:

```shell
ln -s /usr/share/zoneinfo/Europe/Copenhagen /etc/localtime
```

Set the hostname, I name it after the monster Grendel:

```shell
cat "grendel" > /etc/hostname
```

We also need to define the hostname in `/etc/hosts`, from here it will be recognized by the local network, thanks to mDNS (`avahi_daemon`) that we installed above.

```shell
cat << EOF > /etc/hosts
127.0.0.1 localhost
::1       localhost
127.0.1.1 grendel
EOF
```


## <span class="section-num">12</span> User accounts {#user-accounts}

To protect the root account, set a password for it:

```shell
passwd
```

and I also create a user for myself:

```shell
useradd  --groups wheel,users --home-dir /home/mok --uid 1026 mok
```

Then, I set a password:

```shell
passwd mok
```


## <span class="section-num">13</span> Install basic packages {#install-basic-packages}

Next step is to install basic packages:

```shell
pacman -S base-devel btrfs-progs grub efibootmgr mtools networkmanager openssh sudo acpid vim
```


## <span class="section-num">14</span> Install Linux kernel {#install-linux-kernel}

Install at least one kernel:

```shell
pacman -S linux linux-headers
```

and install firmware files:

```shell
pacman -S linux-firmware
```

The Mac mini has an Intel GPU, so I install mesa:

```shell
pacman -S mesa
```


## <span class="section-num">15</span> More packages to install {#more-packages-to-install}

More packages to install, in no particular order:

-   broadcom-wl  (necessary for Mac mini)
-   git
-   dnsutils
-   inetutils
-   dnsutils
-   fastfetch
-   htop
-   btop
-   man-db
-   man-pages
-   pipewire
-   wireplumber
-   pipewire-alsa
-   pipewire-pulseaudio
-   fzf
-   zoxide
-   eza
-   bat
-   emacs
-   tmux
-   mosh
-   otf-monaspace-nerd
-   ttf-firacode-nerd
-   firefox

Copy/paste this list from here:

```text
bat bind bluez broadcom-wl btop btrfs-progs chromium cmake
cosmic dnsutils emacs eza fastfetch firefox fzf git grub-btrfs htop
inetutils inotify-tools ipython linux linux-firmware linux-headers
man-db man-pages mosh openssh otf-monaspace-nerd pipewire pipewire-alsa
pipewire-pulse pipewire-pulseaudio reflector timeshift tmux
ttf-firacode-nerd wireplumber xdg-desktop-portal-cosmic zoxide
zsh
```

Many guides on the Internet talk about `os-prober`, but I don't plan to add more operating systems so I am omitting it.


## <span class="section-num">16</span> Install Cosmic desktop {#install-cosmic-desktop}

Install the Cosmic desktop and enable it  when booting

```shell
pacman -S cosmic
```

Note, `cosmic` is a meta package that will install around 20 other packages such as `cosmic-greeter`, `cosmic-settings` and `cosmic-terminal`. You can select to install all of them (recommended).

Next enable `cosmic-greeter` to present the login screen when booting next time.

```nil
systemctl enable cosmic-greeter
```


## <span class="section-num">17</span> Enable ssh {#enable-ssh}

I installed `openssh` so enable it to start up at boot time.

```shell
systemctl enable sshd --now
```

The `--now` switch tells `systemd` to start the service after enabling it for boot. At this point you should in fact be able to ssh into the new machine from another computer.


## <span class="section-num">18</span> Generate kernel ramdisks {#generate-kernel-ramdisks}

Generate the intial Linux image to boot:

```shell
mkinitcpio -p linux
```


## <span class="section-num">19</span> Set up GRUB {#set-up-grub}

Install GRUB:

```shell
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

and generate a config file for GRUB:

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```


## <span class="section-num">20</span> Other services to start {#other-services-to-start}

Enable NetworkManager so networking will function when you reboot:

```shell
systemctl enable NetworkManager
```

acpid for power management:

```shell
systemctl enable acpid
```

next, `systemd-resolved` for DNS resolution and `avahi-daemon` for name resolution on your local network (mDNS):

```shell
systemctl enable systemd-resolved
systemctl enable avahi-daemon
```

Enable bluetooth

```shell
systemctl enable bluetooth
```

Enable boot from timeshift snapshots. You will have to configure this system later. Check out manuals and YouTube videos for guides. I find that Timeshift does not open a window on Wayland, btw.

```shell
systemctl enable grub-btrfs
```

For the sound system to work, enable everything pipewire:

```shell
systemctl enable pipewire-pulse
systemctl enable pipewire
systemctl enable wireplumber.service
```

That's it for now. Once the newly installed system boots, there will be opportunity to enable other services if needed.


## <span class="section-num">21</span> Update the pacman mirrorlist {#update-the-pacman-mirrorlist}

Let's just update the list of pacman mirrors to be the fastest I can access:

```shell
reflector --country DK --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```


## <span class="section-num">22</span> Blacklist the b43 driver {#blacklist-the-b43-driver}

**Important!**

The Macmini6,1 wireless interface needs the `wl` kernel module that we installed above in the `broadcom-wl` package. But we need to prevent the `b43` module from starting, otherwise wifi won't work. Create a file in `/etc/modprobe.d/` called `blacklist.conf`:

```shell
echo "blacklist b43" > /etc/modprobe.d/blacklist.conf
```


## <span class="section-num">23</span> Wrapping Up {#wrapping-up}

Exit our chroot environment:

```shell
exit
```

Now we are back on `archiso`. Unmount all partitions and reboot.

```shell
umount -R /mnt
reboot
```

If you are lucky, the system will boot up in the Cosmic greeter. Otherwise, back to the drawing board.

---
