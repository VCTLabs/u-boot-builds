#u-boot flash binary for imx93 var som (symphony)

Use the u-boot blob with varascite rescue SD card or make your own boot card.

Debian rootfs process outlined below, adapted from both of the following:

https://variwiki.com/index.php?title=Yocto_Build_Linux&release=mx93-yocto-kirkstone-5.15.71_2.2.0-v1.2
https://forum.digikey.com/t/debian-getting-started-with-the-mcimx8m-evk/13020

Make a bootable SDCard and populate with u-boot, kernel, and recent debian
bullseye (or buster or ubuntu) using yocto-built kernel/u-boot.  Note this
will use a single rootfs partition to match up with the varascite `.wic` file
in `meta-variscite-bsp/wic`.

##Basic Requirements

either:

* follow varascite wiki pages to build u-boot and kernel from source

or:

* yocto build tree *after* building meta-toolchain and virtual/kernel targets
* clone this repository

plus:

* suitable sdcard at least 4GB in size

The following (specific) steps assume the yocto build tree (option 2 above).

Start from the repo root directory (after the clone step above).

Download a rootfs from here: https://rcn-ee.com/rootfs/eewiki/minfs/

```
wget -c https://rcn-ee.com/rootfs/eewiki/minfs/debian-11.6-minimal-arm64-2022-12-20.tar.xz
```

Extract:

```
tar xf debian-11.6-minimal-arm64-2022-12-20.tar.xz

```

Run lsblk to help figure out what linux device has been reserved for your External Drive.

```
#Example: for DISK=/dev/sdX
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465.8G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
└─sda2   8:2    0 465.3G  0 part /                <- Development Machine Root Partition
sdb      8:16   1   962M  0 disk                  <- microSD/USB Storage Device
└─sdb1   8:17   1   961M  0 part                  <- microSD/USB Storage Partition
```

Erase partition table/labels on microSD card; *be sure to use your sdcard device!!*

```
sudo dd if=/dev/zero of=/dev/sdX bs=1M count=100
```

Install Bootloader; *be sure to use your sdcard device!!*

```
sudo dd if=imx93varsom/2022.04.x/imx-boot-sd.bin of=/dev/sdX bs=1K seek=32 conv=fsync
```

The next command requires at least sfdisk from util-linux 2.26:

```
#Check the version of sfdisk installed on your pc is atleast 2.26.x or newer.
sudo sfdisk --version
```

If good, setup partitions; *be sure to use your sdcard device!!*

```
#sfdisk >= 2.26.x
sudo sfdisk /dev/sdX <<-__EOF__
16M,,L,*
__EOF__
```

Format Partition; *be sure to use your sdcard device!!*

``` 
sudo mkfs.ext4 -L rootfs /dev/sdX1
```

Mount Partition; on most systems these partitions may be auto-mounted

```
sudo mount /dev/sdX1 /media/rootfs/
```

Copy Root File System

```
#user@localhost:~$
sudo tar xfvp ./debian-*-*-arm64-*/arm64-rootfs-*.tar -C /media/rootfs/
sync
```

Copy Kernel Image

Next change to your fsl-yocto build directory, by default `build_xwayland`.

```
#user@localhost:~$
cd path/to/yocto/build_xwayland
export kver=$(zcat tmp/deploy/images/imx93-var-som/Image.gz | strings | grep -m 1 -i "Linux version" | awk '{print $3}')
sudo cp -vL tmp/deploy/images/imx93-var-som/Image.gz /media/rootfs/boot/Image.gz-${kver}
sudo ln -fs /boot/Image.gz-${kver} /media/rootfs/boot/Image.gz
```

Copy Kernel Device Tree Binaries

```
#user@localhost:~$
sudo cp -vL tmp/deploy/images/imx93-var-som/imx93-var-som-symphony.dtb /media/rootfs/boot/imx93-var-som-symphony.dtb
```

Copy Kernel Modules

```
#user@localhost:~$
sudo tar xfv tmp/deploy/images/imx93-var-som/modules-imx93-var-som.tgz -C /media/rootfs/
```

File Systems Table (/etc/fstab)

```
#user@localhost:~/$
sudo sh -c "echo '/dev/mmcblk1p1  /  auto  errors=remount-ro  0  1' >> /media/rootfs/etc/fstab"
```

Remove microSD/SD card

```
sync
sudo umount /media/rootfs
```

User login

Initial login credentials are shown in the console motd.


