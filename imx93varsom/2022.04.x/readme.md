u-boot flash binary for imx93 var som (symphony)

use with rescue SD card or make your own boot card

https://forum.digikey.com/t/debian-getting-started-with-the-mcimx8m-evk/13020

sudo dd if=/dev/zero of=/dev/sdX bs=1M count=100
sudo dd if=imx-boot-sd.bin of=/dev/sdX bs=1K seek=32 conv=fsync
