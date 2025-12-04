# Chrome OS RMA Shim Bootloader

Shimboot is a collection of scripts for patching a Chrome OS RMA shim to serve as a bootloader for a standard Linux distribution. It allows you to boot a full desktop Debian install on a Chromebook, without needing to unenroll it or modify the firmware.


## Table of Contents:
- [Features](#features)
- [About](#about)
  * [Partition Layout](#partition-layout)
- [Status](#status)
  * [Device Compatibility Table](#device-compatibility-table)
  * [TODO](#todo)
- [Usage](#usage)
  * [Prerequisites](#prerequisites)
  * [Video Tutorial](#video-tutorial)
  * [Build Instructions](#build-instructions)
  * [Booting the Image](#booting-the-image)
- [FAQ](#faq)
- [Copyright](#copyright)
  * [Copyright Notice](#copyright-notice)
## FORK FEATURES:


## About:
this forks purpose is to make it easy  building shimboot with diffrent desktops and hopefully diffrent distros in github actions

### Partition Layout:
1. 1MB dummy stateful partition
2. 32MB Chrome OS kernel
3. 20MB bootloader
4. The rootfs partitions fill the rest of the disk

Note that rootfs partitions have to be named `shimboot_rootfs:<partname>` for the bootloader to recognize them.

## Status:
Driver support depends on the device you are using shimboot on. The `patch_rootfs.sh` script attempts to copy all the firmware and drivers from the shim and recovery image into the rootfs, so expect most things to work on other boards. Both x86_64 and ARM64 chromebooks are supported.

### Device Compatibility Table:
| Board Name                                          | X11               | Wifi              | Speakers | Backlight | Touchscreen | 3D Accel          | Bluetooth | Webcam   |
|-----------------------------------------------------|-------------------|-------------------|----------|-----------|-------------|-------------------|-----------|----------|
| [`dedede`](https://cros.download/recovery/dedede)   | yes               | yes               | no       | yes       | yes         | yes               | yes       | yes      |
| [`octopus`](https://cros.download/recovery/octopus) | yes               | yes               | yes      | yes       | yes         | yes               | yes       | yes      |
| [`nissa`](https://cros.download/recovery/nissa)     | yes               | yes               | no       | yes       | yes         | yes               | yes       | yes      |
| [`reks`](https://cros.download/recovery/reks)       | no<sup>[1]</sup>  | yes               | untested | untested  | untested    | no                | untested  | untested |
| [`kefka`](https://cros.download/recovery/kefka)     | no<sup>[1]</sup>  | yes               | yes      | yes       | untested    | no                | untested  | untested |
| [`zork`](https://cros.download/recovery/zork)       | yes               | yes               | no       | yes       | yes         | yes               | yes       | yes      |
| [`grunt`](https://cros.download/recovery/grunt)     | yes<sup>[4]</sup> | yes<sup>[3]</sup> | no       | yes       | yes         | yes               | yes       | yes      |
| [`jacuzzi`](https://cros.download/recovery/jacuzzi) | yes               | yes               | no       | yes       | untested    | no                | no        | yes      |
| [`corsola`](https://cros.download/recovery/corsola) | yes               | yes               | no       | yes       | yes         | yes<sup>[5]</sup> | yes       | yes      |
| [`hatch`](https://cros.download/recovery/hatch)     | yes               | yes<sup>[2]</sup> | no       | yes       | yes         | yes               | yes       | yes      |
| [`snappy`](https://cros.download/recovery/snappy)   | yes               | yes               | yes      | yes       | yes         | yes               | yes       | yes      |
| [`hana`](https://cros.download/recovery/hana)       | yes               | yes               | no       | yes       | untested    | yes               | yes       | no       |

<sup>1. The kernel is too old.</sup><br>
<sup>2. 5ghz wifi networks do not work, but 2.4ghz networks do.</sup><br>
<sup>3. You may need to compile the wifi driver from source. See issue #69.</sup><br>
<sup>4. X11 and LightDM might have some graphical issues.</sup><br>
<sup>5. You need to use Wayland instead of X11.</sup>

This table is incomplete. If you want to contribute a device compatibility report please create a new issue on the Github repository.

On all devices, expect the following features to work:
- Zram (compressed memory)
- Disk compression with squashfs

On all devices, the following features will not work:
- Suspend (disabled by the kernel)
- Swap (disabled by the kernel)

A possible workaround for audio issues is using a USB sound card. Certain "USB to headphone jack" adapters are complete sound cards, which are supported by Linux. See [issue #234](https://github.com/ading2210/shimboot/issues/234).

### TODO:
- Finish Python TUI rewrite (see the `python` branch if you want to help with this)
- Support for more distros (Ubuntu and Arch maybe)
- Eliminate binwalk dependency
- Get audio to work on dedede
- Get kexec working

PRs and contributions are welcome to help implement these features.

## Usage:
### Build Instructions for github actions:
1.fork this repo

2,first choose a desktop enviorment like
gnome, xfce, kde, lxde, gnome-flashback, cinnamon, mate, and lxqt.

once you did navigate to branches 
and choose from the listed



<img src="/website/assets/shimboot_demo_1.jpg" alt="list of branches." width="400"/>
after that go to actions and start the build

## FAQ:

#### I want to use a different Linux distribution. How can I do that?
Using any Linux distro is possible, provided that you apply the [proper patches](https://github.com/ading2210/chromeos-systemd) to systemd and recompile it. Most distros have some sort of bootstrapping tool that allows you to install it to a directory on your host PC. Then, you can just pass that rootfs directory into `patch_rootfs.sh` and `build.sh`.

Here is a list of distros that are supported out of the box:
- Debian 12 (Bookworm) - This is the default.
- Debian 13 (Trixie)
- Debian Unstable (Sid)
- Alpine Linux

PRs to enable support for other distros are welcome. 

Debian Sid (unstable rolling release) and Trixie (upcoming Debian 13 release) is also supported if you just want newer packages, and you can install it by passing an argument to `build_complete.sh`: 
```bash
sudo ./build_complete.sh dedede release=unstable
```
```bash
sudo ./build_complete.sh dedede release=trixie
```

There is also experimental support for Alpine Linux. The Alpine disk image is about half the size compared to Debian, although some applications are missing. Pass the `distro=alpine` to use it:
```bash
sudo ./build_complete.sh dedede distro=alpine
```

#### How can I install a desktop environment other than XFCE?
You can pass the `desktop` argument to the `build_complete.sh` script, like this:
```bash
sudo ./build_complete.sh grunt desktop=lxde
```
The valid values for this argument are: `gnome`, `xfce`, `kde`, `lxde`, `gnome-flashback`, `cinnamon`, `mate`, and `lxqt`.

#### GPU acceleration isn't working, how can I fix this?
If your kernel version is too old, the standard Mesa drivers will fail to load. Instead, you must download and install the `mesa-amber` drivers. Run the following commands:
```bash
sudo apt install libglx-amber0 libegl-amber0
echo "MESA_LOADER_DRIVER_OVERRIDE=i965" | sudo tee -a /etc/environment
```
You may need to change `i965` to `i915` (or `r100`/`r200` for AMD hardware), depending on what GPU you have.

For ARM Chromebooks, you may have to tweak the [Xorg configuration](https://xkcd.com/963/) instead.

You can also try switching between X11 and W
#### Can I unplug the USB drive while using Debian?
By default, this is not possible. However, you can simply copy your Debian rootfs onto your internal storage by first using `fdisk` to repartition it, using `dd` to copy the partition, and `resize2fs` to have it take up the entire drive. In the future, loading the OS to RAM may be supported, but this isn't a priority at the moment. You can also just blindly copy the contents of your Shimboot USB to the internal storage without bothering to repartition:
```bash
#check the output of this to know what disk you're copying to and from
fdisk -l

#run this from within the shimboot bootloader
#this assumes the usb drive is on sda and internal storage is on mmcblk1
dd if=/dev/sda of=/dev/mmcblk1 bs=1M oflag=direct status=progress
```
wayland, but this requires a different desktop environment than XFCE.

#### Can the rootfs be compressed to save space?
Compressing the Debian rootfs with a squashfs is supported, and you can do this by running the regular Debian rootfs through `./build_squashfs.sh`. For example:
```bash
sudo ./build_rootfs.sh data/rootfs bookworm
sudo ./build_squashfs.sh data/rootfs_compressed data/rootfs path_to_shim
sudo ./build.sh image.bin path_to_shim data/rootfs_compressed
```
Any writes to the squashfs will persist, but they will not be compressed when saved. For the compression to be the most effective, consider pre-installing most of the software you use with `custom_packages=` before building the squashfs.

On the regular XFCE4 image, this brings the rootfs size down to 1.2GB from 3.5GB.


#### I want to install another desktop without building an image myself.
You can replace the desktop environment in your existing Shimboot installation easily, using APT. For example:
```bash
sudo apt install task-cinnamon-desktop *xfce*- thunar- --autoremove
```
Replace `task-cinnamon-desktop` with the DE that you want to install (such as `task-kde-desktop`). This installs the other DE and uninstalls XFCE at the same time. Then once the installation has finished, reboot the system.

#### My Chromebook is enrolled and it doesn't recognize the USB drive.
Chromebooks that were manufactured after early 2023 contain a patch in the read-only firmware that prevents Shimboot from booting, even if you switch to dev mode. This only affects enrolled devices, and there is no workaround if your device is affected. 

#### How can I encrypt my Shimboot USB?
You can encrypt the root partition using the `luks` option when building the image. For instance:
```bash
sudo ./build_complete.sh corsola luks=1
```
```
## Copyright:
Shimboot is licensed under the [GNU GPL v3](https://www.gnu.org/licenses/gpl-3.0.txt). 

Unless otherwise indicated, all code has been written by me, [ading2210](https://github.com/ading2210).

Other contributors:
- [@a1g0r1thm9](https://github.com/a1g0r1thm9) - LUKS2 encryption feature ([PR #300](https://github.com/ading2210/shimboot/pull/300))

### Copyright Notice:
```
ading2210/shimboot: Boot desktop Linux from a Chrome OS RMA shim.
Copyright (C) 2025 ading2210

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
```
