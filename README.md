# OrangePi-5Plus-eGPU-Guide

## Hardware Preparation

- Cables
    - USB to TypeC calbe
    - 5V 4A Power Supply
    - HDMI cable(Monitor)

- 500W+ ATX PSU
    - 24 Pin ATX Connectors(with Jumper)
    - PCI-E power cables x 2
    - Floppy 4 pins to SATA x 1(for riser) 

- Monitor
- Linux PC
- Orange PI 5 Plus
- M.2 To PCIE x16 adapater
- Graphic Card
- SD Card(32GB+) with reader

## Software Preparation

### Flash Armbian on SD Card

1.download armbian img [here](https://www.armbian.com/orangepi-5/)

I choosed [Orangepi5-plus_bookworm_legacy_5.10.160_minimal.img.xz](https://fi.mirror.armbian.de/dl/orangepi5-plus/archive/Armbian_23.8.1_Orangepi5-plus_bookworm_legacy_5.10.160_minimal.img.xz)

2.get balenaEtcher

[download link](https://github.com/balena-io/etcher/releases)

3.flash the image
Insert the SD card into the card reader and plug it into the USB port of your computer.

>`Flash from file`>`Select Target`>`Flash`

4.insert the sd card into OrangePi-5-Plus's SD Card Slot

5.Boot into OS.

<img src="https://raw.githubusercontent.com/MartinRGB/OrangePi-5Plus-eGPU-Guide/main/art/neofetch1.png" width="50%" height="50%">

6.install necessary tools

```
sudo apt-get update && sudo apt update
sudo apt-get upgrade && sudo apt upgrade
sudo apt-get install -y armbian-config
sudo armbian-config
#>Personal>Locales>en.US_UTF8
sudo nano /boot/armbianEnv.txt3
#content is `extraargs=video=1920x1080@60`
```

7.blacklist amdgpu or radeon

```
sudo nano /etc/modprobe.d/blacklist-radeon.conf

# content is: 
blacklist radeon

sudo nano /etc/modprobe.d/blacklist-amdgpu.conf

# content is: 
blacklist amdgpu
```

## Build & Copy Kernel

### Build Kernel

```
sudo apt-get install u-boot-tools
git clone https://github.com/orangepi-xunlong/linux-orangepi.git
cd linux-orangepi
git switch orange-pi-5.10-rk3588
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu rockchip_linux_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

>`Device Drivers`>`PCI Support`>`PCI Express Port Bus support`

>`Graphics Support`>`ATI Radeon` & `AMD GPU` (Enable SI & CIK Support)

```
make -j16 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

### Copy Kernel

**Enabling ssh services in armbian**

go into linux folder

```bash
rm -rf build_kernel
mkdir build_kernel
sudo env PATH=$PATH make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=./build_kernel modules_install
# scp ./arch/arm64/boot/Image [username]@[OrangePi-5-Plus's Ip address]:~
scp ./arch/arm64/boot/Image mart@192.168.0.101:~
tar -czvf ./build_kernel/lib.tar.gz ./build_kernel/lib
# scp ./arch/arm64/boot/Image [username]@[OrangePi-5-Plus's Ip address]:~
scp ./build_kernel/lib.tar.gz mart@192.168.0.101:~
```

ssh login your OPI

```
# ssh [username]@[OrangePi-5-Plus's Ip address]
ssh mart@192.168.0.101
sudo mv ./Image /boot
tar -xvzf lib.tar.gz
sudo cp -r ./build_kernel/lib/modules /lib
```

reboot

## Reference 

[Building mainline U-boot and Linux kernel for Orange Pi boards](https://uthings.uniud.it/building-mainline-u-boot-and-linux-kernel-for-orange-pi-boards)

[orangepiwiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_5_Plus)
