---
title: Getting Started BPI-R4 Lite
description: Getting Started for BPI-R4 Lite
published: true
date: 2025-07-21T02:09:32.210Z
tags: 
editor: markdown
dateCreated: 2025-07-10T09:06:02.179Z
---

# Introduction
Banana Pi BPI-R4 Lite Router board with MediaTek MT7987A Quad-core Arm Cortex-A53 design ,2GB DDR4,8GB eMMC flash,256MB SPI-NAND Flash,32MB SPI-NOR Flash and Micro SD card slot onboard, also have 1x 2.5G SFP, 1x 2.5G RJ45 WAN,4x 1G RJ45 LAN,with USB3.0 port,1x M.2 KEY-B slot with USB3.0 for 5G Module.1x  miniPCIe slot with PCIe3.0 2lane interface for Wi-Fi 7 NIC (Option: 2x miniPCIe slots with PCIe3.0 1lane),1x miniPCIe slot with USB2.0 interfaceIt and 1x USB TypeC Debug Console. It is a very high performance open source router development board.

>More Infomation: [BananaPi_BPI-R4_Lite](/en/BPI-R4_Lite/BananaPi_BPI-R4_Lite)
{.is-success}

{.is-success}

# Development
## Prepare
* Prepare 8G/above TF card, USB-Serial cable, Ubuntu System
* 12V/2A power adapter (without any peripherals, the power consumption of BPI-R4_Lite Main Board will not exceed 10W in the most extreme cases. but you need to determine whether you need a higher power power adapter according to your own accessory usage)
* Using your USB TypeC Debug Console
* link:                              [Download the image] to be burned
* Default IP address for LAN port: 192.168.1.1
* User name/password: pi/bananapi ,root/bananapi. +
Or the user is root without a password.
* WIFI: AP_MTK_MT7990_2G/AP_MTK_MT7990_5G/AP_MTK_MT7990_6G
![r4-lite_bootstrap.jpg](/bpi-r4_lite/r4-lite_bootstrap.jpg)
                           
## BPI-R4_Lite bootstrap and device select Jumper Setting:
![banana_pi_r4_lite_bootstrap.png](/bpi-r4_lite/banana_pi_r4_lite_bootstrap.png)
1. A/B/D Jumper is "0", BPI-R4_Lite will boot from SD card
2. A Jumper is "0" and B/C Jumper is "1", BPI-R4_Lite will boot from SPI NOR Flash
3. A Jumper is "1" and B/C Jumper is "0", BPI-R4_Lite will boot from SPI NAND Flash
4. A/B/D Jumper is "1",BPI-R4_Lite will boot from EMMC
5. If the console said "system halt!", it means that the bootup storage does not cotain any OS
+
```bash
  F0: 102B 0000
  FA: 5100 0000
  FA: 5100 0000 [0200]
  F9: 1041 0000
  F3: 1001 0000 [0200]
  F3: 1001 0000
  F6: 102C 0000
  F5: 1026 0000
  00: 1005 0000
  FA: 5100 0000
  FA: 5100 0000 [0200]
  F9: 1041 0000
  F3: 1001 0000 [0200]
  F3: 1001 0000
  F6: 102C 0000
  01: 102A 0001
  02: 1005 0000
  BP: 0200 00C0 [0001]
  EC: 0000 0000 [0000]
  MK: 0000 0000 [0000]
  T0: 0000 00D7 [0101]
  System halt!
```

## How to burn image to SD card.

- Please follow the following diagram for Windows PC.

                         image
- Linux PC, you can use the "**mksf**" command for formatting, or use the "**dd**" command to write zeros.
**Burn image to SD card on windows computer**
[Balena Etcher](https://balena.io/etcher) is an opensource GUI flash tool by Balena, Flash OS images to SDcard or USB drive.
- Click on "**Flash from file**" to select image. 
- Click on "**Select target**" to select USB device. 
- Click on "**Flash!**" Start burning.
                  image
                  
 **Burn image to SD card on linux computer**
1. You could download latest image from our forum     
2. Install bpi-tools on your Ubuntu. If you can't access this URL or any other problems, please go to bpi-tools repo and install this tools manually.
+
```sh
apt-get install pv
curl -sL https://github.com/BPI-SINOVOIP/bpi-tools/raw/master/bpi-tools | sudo -E bash
```
 After you download the image, insert your TF card into your Ubuntu. Execute

+
```sh
bpi-copy xxx.img /dev/sdx
```
to install image on your TF card

. After step 3, then you can insert your TF card into R4_Lite, and press power button to setup R4_Lite
 
**Change Boot Jumper to boot from SD, Enable SD Card Device.**

## How to burn image to onboard NOR
> When you want to Update Nand device, Firstly Change boot switch to boot from SD device and insert one SD with SD boot Image, then after boot up,you need flash one Nand image into Nor device. Finally you change bootstrap to boot from Nor device.
{.is-info}

Before burning image into Nand, please prepare a USB disk. Let’s take OpenWrt image (mtk-bpi-r4lite-NOR.img ) for example, the steps are below:

1. Copy Nand boot OpenWrt image(**mtk-bpi-r4lite-NOR.img**) to USB disk. 
2. Change boot switch Jumper, the board boot from SD device, then power up the board.
3. Plug in USB disk to the board, and mount the USB to /mnt or other directory as follows: (you can skip mounting if it is mounted automatically)

+
```SH
mount -t vfat /dev/sda1 /mnt 
cd /mnt
```
4. Execute following command to erase the whole Nor flash and copy image to nor device:

+
```sh
mtd erase /dev/mtd0
dd if=mtk-bpi-r4lite-NOR.img of=/dev/mtdblock0
```
5. Power off BPI-R4_Lite board, unplug u-disk driver, change bootstrap to boot from Nand device.

## How to burn image to onboard Nand
> **NAND has an image burned at the factory. If you want to use it, simply switch to the corresponding boot and then power on to start.**
{.is-info}


>  When you want to Update Nand device, Firstly Change boot switch to boot from SD device and insert one SD with SD boot Image, then after boot up,you need flash one Nand image into Nand device. Finally you change bootstrap to boot from Nand device.
{.is-info}


Before burning image into Nand, please prepare a USB disk. Let's take OpenWrt image (mtk-bpi-r4lite-NAND-2PCIe-1L.img) for example, the steps are below:

1. Copy Nand boot OpenWrt image(**mtk-bpi-r4lite-NAND-2PCIe-1L.img**) to USB disk. 
2. Change boot switch Jumper, the board boot from SD device, then power up the board.
3. Plug in USB disk to the board, and mount the USB to /mnt or other directory as follows: (you can skip mounting if it is mounted automatically)

+
```SH
mount -t vfat /dev/sda1 /mnt 
cd /mnt/sda1
```
4. Execute following command to erase the whole Nand flash and copy image to nand device:

+
```sh
mtd erase /dev/mtd0
mtd write mtk-bpi-r4lite-NAND-2PCIe-1L.img /dev/mtd0
```
. Power off BPI-R4_Lite board, unplug u-disk driver, change bootstrap to boot from Nand device.


## How to burn image to onboard eMMC
> Because SD card and EMMC device share one SOC's controller, it is necessary to switch to NAND startup and then burn the EMMC image into the EMMC. Finally, you will change the boot to boot from EMMC.
{.is-info}


Before burning image to eMMC, please prepare a USB disk. Let's take OpenWrt image (bl2_emmc.img, mtk-bpi-r4lite-EMMC-NAND.img) for example, the steps are below:

1. Copy EMMC boot OpenWrt image(**bl2_emmc-r4.img**,**mtk-bpi-r4-EMMC-20231030.img**) to USB disk, if the image is compressed please uncompress it before copying to USB disk.

2. Change the switch jumper to Nand and start the motherboard from Nand.
 
3. Plug in USB disk to the board, and mount the USB to /mnt or other directory as follows: (you can skip mounting if it is mounted automatically)

+
```sh
mount -t vfat /dev/sda1 /mnt 
cd /mnt/sda1
```

4. Execute :

+
```sh
echo 0 > /sys/block/mmcblk0boot0/force_ro
dd if=bl2_emmc.img of=/dev/mmcblk0boot0
dd if=mtk-bpi-r4lite-EMMC-NAND.img of=/dev/mmcblk0
mmc bootpart enable 1 1 /dev/mmcblk0
sync
sync
```
 
. Power off R4_Lite board, remove u-disk driver, change bootstrap to boot from emmc device.