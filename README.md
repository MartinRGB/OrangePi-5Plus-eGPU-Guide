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

### Flash Armbian to SD Card

1.download armbian img [here](https://www.armbian.com/orangepi-5/)

I choosed [Orangepi5-plus_bookworm_legacy_5.10.160_minimal.img.xz](https://fi.mirror.armbian.de/dl/orangepi5-plus/archive/Armbian_23.8.1_Orangepi5-plus_bookworm_legacy_5.10.160_minimal.img.xz)

2.get balenaEtcher

[download link](https://github.com/balena-io/etcher/releases)

3.insert the sd card into PC via reader & flash the image
Insert the SD card into the card reader and plug it into the USB port of your computer.

>`Flash from file`>`Select Target`>`Flash`

4.insert the sd card into OrangePi-5-Plus's SD Card Slot

5.Boot into OS.

<img src="https://raw.githubusercontent.com/MartinRGB/OrangePi-5Plus-eGPU-Guide/main/art/neofetch1.png" width="50%" height="50%">

6.install necessary tools on OPI

```bash
sudo apt-get update && sudo apt update
sudo apt-get -y upgrade && sudo apt -y upgrade
sudo apt-get install -y armbian-config firmware-amd-graphics pciutils git pkg-config libpng-dev libgl1-mesa-dev build-essential iptables
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
sudo rm -rf /vmlinuz
sudo ln -s /boot/Image /vmlinuz
```

10.shutdown & take the SD card out,insert the sd card into PC via reader

## 5.10 Kernel & Modules

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

I disabled Realtek Wifi driver & Rockchip Wireless Lan Support in `menuconfig`( I use Intel's AX210)
> `Device Drivers` > `Network device support` > `Wireless LAN` > `[] Realtek devices` | `[]Rockchip Wireless Lan Support`

> `kernel hacking` > `Compile-time checks and compiler options`> `(4096) Warn for stack frames larger than ...`

```
make -j16 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

### Copy builit Kernel & install modules

go into linux folder

```bash
sudo env PATH=$PATH make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=/media/${USER}/armbi_root modules_install

sudo cp ./arch/arm64/boot/Image /media/${USER}/armbi_boot/Image
sudo cp ./arch/arm64/boot/dts/rockchip/*.dtb /media/${USER}/armbi_boot/dtb/rockchip
sudo cp ./arch/arm64/boot/dts/rockchip/overlay/*.dtb* /media/${USER}/armbi_boot/dtb/rockchip/overlay
sudo cp ./arch/arm64/boot/dts/rockchip/overlay/README.rockchip-overlays /media/${USER}/armbi_boot/dtb/rockchip/overlay

# sudo cp ./arch/arm64/boot/Image /media/${USER}/armbi_boot/Image && sudo cp ./arch/arm64/boot/dts/rockchip/*.dtb /media/${USER}/armbi_boot/dtb/rockchip && sudo cp ./arch/arm64/boot/dts/rockchip/overlay/*.dtb* /media/${USER}/armbi_boot/dtb/rockchip/overlay && sudo cp ./arch/arm64/boot/dts/rockchip/overlay/README.rockchip-overlays /media/${USER}/armbi_boot/dtb/rockchip/overlay

```
insert the SD Card into OPI,and power up

## Tools & Patches

```
sudo apt-get install -y xorg lightdm lxde
```

benchmark

```
sudo apt-get install glmark-x11 mesa-utils
vblank_mode=0 optirun glmark2
```

## Mesa

```
sudo apt install clang lldb lld
sudo apt-get install cmake libzstd-dev valgrind bison libunwind-dev libsensors-dev libarchive-dev libconfig-dev clang libvdpau-dev libxvmc-dev libxv-dev libomxil-bellagio-dev libva-dev libelf-dev
sudo apt-get install meson llvm byacc libwayland-dev wayland-protocols libwayland-egl-backend-dev libxcb-glx0-dev libxcb-xrm-dev libxcb-composite0-dev libxcb-shm0-dev libx11-xcb-dev libxcb-dri2-0-dev libxcb-dri3-dev libxcb-present-dev libxshmfence-dev python3-mako flex libc6-dev libdrm-dev libxext-dev libxfixes-dev libxxf86vm-dev libxrandr-dev 
git clone https://github.com/Coreforge/mesa.git
cd mesa
git checkout pistuff
mkdir build
cd build
meson ..
# Do not forget add 'r600,radeonsi' into 'gallium-drivers' option
meson configure -Dgallium-drivers=v3d,vc4,freedreno,etnaviv,nouveau,tegra,virgl,lima,panfrost,swrast,r600,radeonsi
sudo ninja install

sudo cp /usr/local/lib/aarch64-linux-gnu/libXvMCr600.so.1.0 /usr/lib/aarch64-linux-gnu/libXvMCr600.so.1.0
sudo cp /usr/local/lib/aarch64-linux-gnu/libXvMCr600.so.1.0.0 /usr/lib/aarch64-linux-gnu/libXvMCr600.so.1.0.0
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so.1.0.0 /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so.1.0.0
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so.1 /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so.1
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so.1.0 /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_r600.so.1.0
sudo cp /usr/local/lib/aarch64-linux-gnu/libXvMCr600.so /usr/lib/aarch64-linux-gnu/libXvMCr600.so
sudo cp /usr/local/lib/aarch64-linux-gnu/libXvMCr600.so.1 /usr/lib/aarch64-linux-gnu/libXvMCr600.so.1
sudo cp /usr/local/lib/aarch64-linux-gnu/dri/r600_drv_video.so /usr/lib/aarch64-linux-gnu/dri/r600_drv_video.so
sudo cp /usr/local/lib/aarch64-linux-gnu/dri/r600_dri.so /usr/lib/aarch64-linux-gnu/dri/r600_dri.so

sudo cp /usr/local/lib/aarch64-linux-gnu/dri/radeonsi_drv_video.so /usr/lib/aarch64-linux-gnu/dri/radeonsi_drv_video.so
sudo cp /usr/local/lib/aarch64-linux-gnu/dri/radeonsi_dri.so /usr/lib/aarch64-linux-gnu/dri/radeonsi_dri.so
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so.1.0.0 /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so.1.0.0
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so.1 /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so.1
sudo cp /usr/local/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so.1.0 /usr/lib/aarch64-linux-gnu/vdpau/libvdpau_radeonsi.so.1.0
```

If use LLVM14,for `‘class llvm::AttributeList’ has no member named ‘hasAttribute’` issue,check [this](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/12715) request

>By the way, with LLVM 14 (59c954f76a66c6fc715610e85be71e9c050f2302), !12715 (merged) applied and the additional patch mentioned there by Kai, I still get the following output with Chrome when using --use-gl=desktop:
>![image](https://github.com/MartinRGB/OrangePi-5Plus-eGPU-Guide/assets/7036706/b0ca8e59-710d-4aaf-9577-86e70023f7df)


## Reference 

[Building mainline U-boot and Linux kernel for Orange Pi boards](https://uthings.uniud.it/building-mainline-u-boot-and-linux-kernel-for-orange-pi-boards)

[orangepiwiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_5_Plus)
