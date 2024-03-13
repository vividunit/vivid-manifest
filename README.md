Vivid Repo Manifest README
==========================

This repo is used to download manifests for Vivid SDK releases.

Specific instructions will reside in READMEs in each branch.

Set up an Development Environment
---------------------------------

It is recommended to use Ubuntu 20.04 for compilation. Other Linux versions may need to adjust the software package accordingly. In addition to the system requirements, there are other hardware and software requirements.

Hardware requirements: 64-bit system, hard disk space should be greater than 40G. If you do multiple builds, you will need more hard drive space.

Software requirements: Ubuntu 20.04 system:
Please install software packages with below commands to setup SDK compiling environment:

```
$: sudo apt-get update
$: sudo apt-get install git ssh make gcc libssl-dev liblz4-tool expect \
g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib \
g++-multilib unzip device-tree-compiler ncurses-dev libgucharmap-2-90-dev \
bzip2 expat gpgv2 cpp-aarch64-linux-gnu time mtd-utils python3-pip zstd curl \
python-is-python3
$: sudo pip3 install asciidoc
```

It is recommended to use Ubuntu 20.04 system or higher version for development. If you encounter an error during compilation, you can check the error message and install the corresponding software packages accordingly.

Set up git
----------

Set up your email and name if you didn't set it before for git.
Replace the following "you@example.com" and "Your Name" to yours.

```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```


Install the `repo` utility:
---------------------------

To use this manifest repo, the `repo` tool must be installed first.

```
$: mkdir ~/bin
$: curl http://commondatastorage.googleapis.com/git-repo-downloads/repo  > ~/bin/repo
$: chmod a+x ~/bin/repo
$: PATH=${PATH}:~/bin
```

Download the Debian Project SDK
-------------------------------

```
$: mkdir vivid-bsp-release
$: cd vivid-bsp-release
$: repo init -u https://github.com/vividunit/vivid-manifest -b <branch name> [ -m <release manifest>]
$: repo sync
```

Each branch will have detailed READMEs describing exact syntax.

Examples
--------

To download the 5.10.110-1.0.0 release

```
$: cd vivid-bsp-release
$: repo init -u https://github.com/vividunit/vivid-manifest -b vivid-linux-bullseye -m vivid-rk3399-5.10.110-1.0.0.xml
$: repo sync
```

Download the Debian prebuilt rootfs image
-----------------------------------------

The rootfs image will be released separatly. You can issue the following commands to add it to the SDK.

```
$: cd vivid-bsp-release
$: mkdir debian 
$: cd debian
$: wget -c https://www.vividunit.com/download/rootfs/rootfs-vivid-latest.img.xz
$: xz -d rootfs-vivid-latest.img.xz
$: ln -sf rootfs-vivid-latest.img linaro-rootfs.img
```

Build an image:
---------------

```
$: cd vivid-bsp-release
$: ./build.sh BoardConfig-rk3399-vivid-unit-v13.mk
$: ./build.sh all
```

After done, you can get `rk3399-vivid-unit-v13-debian11-yyymmdd.img` from vivid-bsp-release/rockdev/ directory.


Recompile custom kernel:
------------------------

For example, we will add the following kernel config

```
CONFIG_OVERLAY_FS=y
CONFIG_OVERLAY_FS_INDEX=y
CONFIG_OVERLAY_FS_XINO_AUTO=y
```

Lets to do it

```
$: cd vivid-bsp-release/kernel
$: make ARCH=arm64 menuconfig
```

Then enable the following kernel configuration

```
File system->
  <*> Overlay filesystem support
  [*] Overlayfs: follow redirects even if redirects are turned off
  [*] Overlayfs: turn on inodes index feature by default
  [*] Overlayfs: auto enable inode number mapping
```

Then save config

```
$: make ARCH=arm64 savedefconfig
$: cp defconfig arch/arm64/configs/vivid_unit_defconfig
```

Recompile the kernel

```
$: cd ../
$: ./build.sh kernel && ./build.sh firmware
```

Now we get the new boot.img from the rockdev directory, and the WiFi/Bt driver module, \
as the kernel configration is changed, we also need to update bcmdhd.ko and dhd_static_buf.ko \
in the rootfs. Use the following command to upload the file to target system.

```
$: scp rockdev/boot.img vivid@xxx.xxx.xxx.xxx:
$: scp kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/bcmdhd.ko vivid@xxx.xxx.xxx.xxx:/system/lib/modules/
$: scp kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/dhd_static_buf.ko vivid@xxx.xxx.xxx.xxx:/system/lib/modules/
```

use ssh to login the target vivid board and use dd command to update the boot.img

```
$: ssh vivid@xxx.xxx.xxx.xxx
$: sudo dd if=boot.img of=/dev/mmcblk0p4 conv=fsync
$: sudo reboot
```

Now we are run in new compiled kernel, run the following commands to check the kernel build time

```
$: uname -ra
```

If you want to generate one new system image, do it like the following commands(option)

```
$: cd vivid-bsp-release/
// update bcmdhd.ko and dhd_static_buf.ko in rootfs
$: sudo mount vivid-bsp-release/ /mnt/
$: sudo cp kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/bcmdhd.ko /mnt/system/lib/modules/
$: sudo cp kernel/drivers/net/wireless/rockchip_wlan/rkwifi/bcmdhd/dhd_static_buf.ko /mnt/system/lib/modules/
$: sync
$: sudo umount /mnt/
$: ./build.sh updateimg
```

Now we will get new updated system image `rk3399-vivid-unit-v13-debian11-yyymmdd.img` from vivid-bsp-release/rockdev/ directory.
