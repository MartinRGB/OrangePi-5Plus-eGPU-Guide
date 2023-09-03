# OrangePi-5Plus-eGPU-Guide

### Bootloader

Related reddit topic [Upstream U-Boot support for OrangePi 5/5B/5+](https://www.reddit.com/r/OrangePI/comments/157l7hv/upstream_uboot_support_for_orangepi_55b5/)

1. in PC Linux System,prepare these package

```
sudo apt install git bc bison flex libssl-dev make 
sudo apt-get install gcc-aarch64-linux-gnu gh
``` 

2.clone orangepi's `u-boot` fork

```
git clone https://github.com/orangepi-xunlong/u-boot-orangepi.git
git switch v2017.09-rk3588
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- orangepi_5_plus_defconfig
```



## Reference 
[Building mainline U-boot and Linux kernel for Orange Pi boards](https://uthings.uniud.it/building-mainline-u-boot-and-linux-kernel-for-orange-pi-boards)
