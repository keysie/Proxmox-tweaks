# Fixing Realtek USB 3.0 to 2.5G Ethernet Dongle r8156

- Verify hardware: run `lsusb` and look for `ID 0bda:8156 Realtek Semiconductor Corp. USB 10/100/1G/2.5G LAN`
- Check current kernel driver version: `modinfo r8152 | grep ^version:`
  If output is smth like `1.12.13` you might want to upgrade. latest is smth like `2.21.4`
- Install pre-requisites:
  `sudo apt-get install dh-dkms dkms build-essential pve-headers`
- Download the latest release form [here](https://github.com/awesometic/realtek-r8152-dkms)
- Install it using `dpkg -i <filename.deb>`
- ideally, `reboot`

## Pre-compile
Whenever possible use `dkms install realtek-r8152/2.21.4 -k 7.0.0-3-pve` (or similar) after running ```apt upgrade``` against any newly installed kernel. This will help you prevent issues when rebooting.

## After upgrade of kernel version (or sometimes after reboot)

- Use `dmesg | grep r8152` to see if/what is wrong
- Use `dkms status` to see versions and full names of the packages in question
- Usually, it is an issue with the dkms module not being rebuilt correctly. Fix like this:
  - Make sure kernel headers are installed and up to date wiht `sudo apt install pve-headers`
  - Remove current module using `sudo dkms remove realtek-r8152/2.21.4` (or similar)
  - Rebuild/reinstall with `sudo dkms install realtek-r8152/2.21.4` (or similar)
  - Load module using `sudo modprobe r8152`
  - Reload pve interfaces with `ifreload -a`
  - (Maybe adjust naming of interfaces in your bridges etc.)

## Kernel version 7.0

- See [the issue](https://github.com/awesometic/realtek-r8152-dkms/issues/37) and [source 1](https://github.com/torvalds/linux/commit/24c7763) and [source 2](https://aur.archlinux.org/packages/r8152-dkms?O=0&PP=10#comment-1067256)
- TLDR: Modify ```/var/lib/dkms/realtek-r8152/2.21.4/source/src/r8152.c``` to include the line ```#include <linux/hex.h>```
