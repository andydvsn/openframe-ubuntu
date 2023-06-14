Linux for OpenFrame
====================

These are the scripts and overlays which I use to create debootstrapped operating systems for OpenFrame devices, notably the OpenFrame 1 (sold in the UK as the O2 Joggler) and OpenFrame 2.


of-builder.sh
--------------

So you want to automatically build images for OpenFrame devices on your server? This is the script for you.

### Build Environment

The OpenFrame devices are all 32-bit, so you're going to need a i386 environment.

Our solution is to create a 32-bit chroot on a 64-bit system. For Debian Bullseye it can be achieved like this:

```
sudo apt install debootstrap schroot
sudo nano /etc/schroot/chroot.d/i386.conf
```

Copy in:

```
[i386]
description=i386
type=directory
union-type=overlay
directory=/var/lib/schroot/chroots/i386
personality=linux32
groups=root,sudo
root-groups=root,sudo
```

Then install the base system with debootstrap:

```
sudo mkdir -p /var/lib/schroot/chroots/i386
sudo debootstrap --arch=i386 bullseye /var/lib/schroot/chroots/i386 http://deb.debian.org/debian
```

Hop in to the chrooted environment like this:

```
sudo schroot -c source:i386
```

You can then complete the setup of your build environment.

```
rm /var/lib/dpkg/statoverride 
rm /var/lib/dpkg/lock
dpkg --configure -a
apt update && apt upgrade
apt install autoconf bc bison build-essential curl debootstrap dosfstools flex git libncurses5-dev libssl-dev lsb-release parted rsync wget 

```

For normal use, get in to the chroot like this:

```
schroot -c i386
```


of-cnc.sh
----------

This script provides a handy method of copying everything back from a storage device for packaging up into a .tgz. That .tgz can then magically be used by __ofimgcreate.sh__ to generate a reasonably fresh image.


of-imgcreate.sh
----------------

This script does the hard work of creating an image file of a given size, creating  filesystem on it, mounting it, and then going on to fetch and install Ubuntu using debootstrap.

You will also need the overlay files provided in this repo and some working kernel image packages.


of-imgmnt.sh
-------------

Used to mount image files for minor tweaks, meaning we don't need to rebuild things every time. Now with variable partition support!

Overlays
---------

The overlay files provide modifications to the vanilla system to provide some necessary and some nice-to-have features, including automatically configured GRUB, network settings from the `/boot` volume (ideal for preconfiguring), sensible system defaults, minimal firmware files and helpful scripts.

With regard to this last element, all OpenFrame related scripts are named `of-*` and live in `/usr/local/sbin`. There is one script, `of-update`, which polls a support server each day to check for script updates. For this reason, please don't edit `of-*` scripts directly, as anything with this prefix in this location may be overwritten.

If this is not to your liking you may disable the update service as follows:

```
sudo systemctl disable of-update
sudo systemctl stop of-update
```

You can run `sudo of-update` to manually check for updates, or download individual scripts from this repository.
