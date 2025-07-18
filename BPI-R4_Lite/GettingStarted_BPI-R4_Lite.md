---
title: Getting Started BPI-R4 Lite
description: Getting Started for BPI-R4 Lite
published: true
date: 2025-07-18T09:01:57.229Z
tags: 
editor: markdown
dateCreated: 2025-07-10T09:06:02.179Z
---

# Introduction
Banana Pi BPI-R4 Lite Router board with MediaTek MT7987A Quad-core Arm Cortex-A53 design ,2GB DDR4,8GB eMMC flash,256MB SPI-NAND Flash,32MB SPI-NOR Flash and Micro SD card slot onboard, also have 1x 2.5G SFP, 1x 2.5G RJ45 WAN,4x 1G RJ45 LAN,with USB3.0 port,M.2 support 4G/5G/NVME SSD.1x  miniPCIe slot with PCIe3.0 2lane interface for Wi-Fi 7 NIC (Option: 2x miniPCIe slots with PCIe3.0 1lane),1x miniPCIe slot with USB2.0 interfaceIt and 1x USB TypeC Debug Console. It is a very high performance open source router development board.

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
[etcher](/en/https://balena.io/etcher)is an opensource GUI flash tool by Balena, Flash OS images to SDcard or USB drive.
