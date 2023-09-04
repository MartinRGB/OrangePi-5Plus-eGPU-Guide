# OrangePi-5Plus-eGPU-Guide

## Bootloader

Related reddit topic [Upstream U-Boot support for OrangePi 5/5B/5+](https://www.reddit.com/r/OrangePI/comments/157l7hv/upstream_uboot_support_for_orangepi_55b5/)

1.in PC Linux System,prepare these package

```
sudo apt install git bc bison flex libssl-dev make 
sudo apt-get install gcc-aarch64-linux-gnu gh
``` 

2.clone orangepi's `u-boot` fork

```
git clone https://github.com/orangepi-xunlong/u-boot-orangepi.git
git switch v2017.09-rk3588
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- orangepi_5_plus_defconfig
make ARCH=arm CROSS_COMPILE=aarch64-linux-gnu-
```

## RKrdeveloptool

```
sudo apt-get install libudev-dev libusb-1.0-0-dev dh-autoreconf
git clone https://github.com/rockchip-linux/rkdeveloptool.git
cd rkdeveloptool
aclocal
autoreconf -i
autoheader
automake --add-missing
./configure
```

modify `main.cpp`,refer to this [issue](https://github.com/rockchip-linux/rkdeveloptool/issues/70)

```
diff --git a/main.cpp b/main.cpp
index 72bd94b..ec5257b 100644
--- a/main.cpp
+++ b/main.cpp
@@ -1489,8 +1489,12 @@ static bool saveEntry(FILE* outFile, char* path, rk_entry_type type,
 
 static inline uint32_t convertChipType(const char* chip) {
        char buffer[5];
+        int ret = 0;
+
        memset(buffer, 0, sizeof(buffer));
-       snprintf(buffer, sizeof(buffer), "%s", chip);
+       if ((ret = snprintf(buffer, sizeof(buffer), "%s", chip))) {
+            perror("snprintf");
+        }
        return buffer[0] << 24 | buffer[1] << 16 | buffer[2] << 8 | buffer[3];
 }
```

make and copy to bin

```
make
sudo cp rkdeveloptool /usr/local/bin/
sudo ldconfig
```

test if installed

```
rkdeveloptool -v
rkdeveloptool ver 1.32
```

## Connect OrangePi 5 Plus to PC

image & guide via [orangepiwiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_5_Plus)

1.connect OrangePi 5 Plus to the Windows computer through the Type-C data cable.
   
![](http://www.orangepi.org/orangepiwiki/images/9/96/Plus5-img52.png)

2.press and hold the MaskROM button

![](http://www.orangepi.org/orangepiwiki/images/7/77/Plus5-img53.png)

3.connect the power supply of the Type-C interface to the development board, and power on, and then release the MaskROM button

![](http://www.orangepi.org/orangepiwiki/images/1/10/Plus5-img54.png)

4.run `rkdeveloptool ld` to find device


## Build Kernel

```
sudo apt-get install u-boot-tools
git clone https://github.com/orangepi-xunlong/linux-orangepi.git
cd inux-orangepi
git switch orange-pi-5.10-rk3588
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu rockchip_linux_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

>`Device Drivers`>`PCI Support`>`PCI Express Port Bus support`
>`Graphics Support`>`ATI Radeon` & `AMD GPU` (Enable SI & CIK Support)

```
make -j16 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

## Reference 

[Building mainline U-boot and Linux kernel for Orange Pi boards](https://uthings.uniud.it/building-mainline-u-boot-and-linux-kernel-for-orange-pi-boards)

[orangepiwiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_5_Plus)
