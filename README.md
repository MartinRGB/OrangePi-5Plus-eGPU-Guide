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

on OPI

```bash
sudo apt-get update && sudo apt update
sudo apt-get -y upgrade && sudo apt -y upgrade
sudo apt-get install -y armbian-config firmware-amd-graphics pciutils git pkg-config libpng-dev libgl1-mesa-dev
# Boot into CLI
sudo systemctl set-default multi-user.target 
#extraargs=video=1920x1080@60
sudo nano /boot/armbianEnv.txt

# sudo apt-get update && sudo apt update && sudo apt-get -y upgrade && sudo apt -y upgrade && sudo apt-get install -y armbian-config firmware-amd-graphics pciutils git pkg-config libpng-dev libgl1-mesa-dev && sudo systemctl set-default multi-user.target 
```

7.Enable 3D Acceleration (Ubuntu variant only):

```bash
sudo apt-get install python3-launchpadlib
sudo add-apt-repository ppa:liujianfeng1994/panfork-mesa
sudo add-apt-repository ppa:liujianfeng1994/rockchip-multimedia
sudo apt update
sudo apt dist-upgrade
sudo apt install mali-g610-firmware rockchip-multimedia-config
```

8.blacklist amdgpu or radeon

```bash
sudo nano /etc/modprobe.d/blacklist-radeon.conf

# content is: 
blacklist radeon

sudo nano /etc/modprobe.d/blacklist-amdgpu.conf

# content is: 
blacklist amdgpu
```

9.rebuild soft link
 
```
sudo rm -rf /vmlinuz /vmlinuz.old
sudo ln -s /boot/Image /vmlinuz
```


shutdown & take the SD card out,insert SD card into reader,plug in PC

## Build & Copy Kernel

### Build Kernel

on PC

```bash
sudo apt-get install u-boot-tools
git clone https://github.com/orangepi-xunlong/linux-orangepi.git
cd linux-orangepi
git switch orange-pi-5.10-rk3588
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rockchip_linux_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

>`Device Drivers`>`PCI Support`>`PCI Express Port Bus support`

>`Graphics Support`>`ATI Radeon` & `AMD GPU` (Enable SI & CIK Support)

I disabled Realtek Wifi driver & Rockchip Wireless Lan Support in `menuconfig`
> `Device Drivers` > `Network device support` > `Wireless LAN` > `[] Realtek devices` | `[]Rockchip Wireless Lan Support`

```
make -j16 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

### Copy Kernel

go into linux folder

```bash
sudo env PATH=$PATH make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=/media/${USER}/armbi_root modules_install

sudo cp ./arch/arm64/boot/Image /media/${USER}/armbi_boot/Image
sudo cp ./arch/arm64/boot/dts/rockchip/*.dtb /media/${USER}/armbi_boot/dtb/rockchip
sudo cp ./arch/arm64/boot/dts/rockchip/overlay/*.dtb* /media/${USER}/armbi_boot/dtb/rockchip/overlay
sudo cp ./arch/arm64/boot/dts/rockchip/overlay/README.rockchip-overlays /media/${USER}/armbi_boot/dtb/rockchip/overlay

```
insert the SD Card into OPI,and power up

## Reference 

[Building mainline U-boot and Linux kernel for Orange Pi boards](https://uthings.uniud.it/building-mainline-u-boot-and-linux-kernel-for-orange-pi-boards)

[orangepiwiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_5_Plus)
