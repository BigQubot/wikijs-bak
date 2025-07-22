---
title: Getting Started BPI-R4 Lite
description: Getting Started for BPI-R4 Lite
published: true
date: 2025-07-22T03:04:54.419Z
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
![r4_lite_bootstrap.png](/bpi-r4_lite/r4_lite_bootstrap.png)

1. A/B/D Jumper is "0", BPI-R4_Lite will boot from SD card
![sd.png](/bpi-r4_lite/sd.png)
2. A Jumper is "0" and B/C Jumper is "1", BPI-R4_Lite will boot from SPI NOR Flash
![nor.png](/bpi-r4_lite/nor.png)
3. A Jumper is "1" and B/C Jumper is "0", BPI-R4_Lite will boot from SPI NAND Flash
![nand.png](/bpi-r4_lite/nand.png)
4. A/B/D Jumper is "1",BPI-R4_Lite will boot from EMMC
![emmc.png](/bpi-r4_lite/emmc.png)
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
![r4_lite_flash1.png](/bpi-r4_lite/r4_lite_flash1.png)
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
> When you want to Update Nor device, Firstly Change boot switch to boot from SD device and insert one SD with SD boot Image, then after boot up,you need flash one Nor image into Nor device. Finally you change bootstrap to boot from Nor device.
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
5. Power off BPI-R4_Lite board, unplug u-disk driver, change bootstrap to boot from Nand device.


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
 
5. Power off R4_Lite board, remove u-disk driver, change bootstrap to boot from emmc device.

## Network-Configuration

* Network-Configuration refer to: http://www.fw-web.de/dokuwiki/doku.php?id=en:bpi-r2:network:start
* Network Interface: eth1,lan5 is for WAN; lan0, lan1, lan2,lan3 is for LAN, ra0/ra1 is for 2.4G wireless, rai0 is for 5G wifi6 wireless, rax0 is for 6G wifi7 wireless.
    
![r4-lite_wan.png](/bpi-r4_lite/r4-lite_wan.png)

```bash
root@OpenWrt:~# ifconfig
br-lan    Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          inet addr:10.0.6.1  Bcast:10.0.6.255  Mask:255.255.255.0
          inet6 addr: fd7f:7a27:1d87::1/60 Scope:Global
          inet6 addr: fe80::b8f0:a3ff:fe27:b253/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:99527801 errors:0 dropped:0 overruns:0 frame:0
          TX packets:37709738 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:149207799463 (138.9 GiB)  TX bytes:2498673864 (2.3 GiB)

br-wan    Link encap:Ethernet  HWaddr AA:0E:53:9B:EB:46  
          inet addr:10.168.1.125  Bcast:10.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::a80e:53ff:fe9b:eb46/64 Scope:Link
          inet6 addr: fd3f:1b63:79e:0:a80e:53ff:fe9b:eb46/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:34911 errors:0 dropped:0 overruns:0 frame:0
          TX packets:26689 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:16527400 (15.7 MiB)  TX bytes:10371963 (9.8 MiB)

eth0      Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          inet6 addr: fe80::b8f0:a3ff:fe27:b253/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1504  Metric:1
          RX packets:1866 errors:0 dropped:1 overruns:0 frame:0
          TX packets:6100 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:483284 (471.9 KiB)  TX bytes:4001138 (3.8 MiB)
          Interrupt:73 

eth1      Link encap:Ethernet  HWaddr AA:0E:53:9B:EB:46  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3251292 errors:0 dropped:0 overruns:0 frame:1
          TX packets:19493357 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:219561937 (209.3 MiB)  TX bytes:29475758425 (27.4 GiB)
          Interrupt:73 

lan0      Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:1615 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1583 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:417484 (407.6 KiB)  TX bytes:710675 (694.0 KiB)

lan1      Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lan2      Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lan3      Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lan5      Link encap:Ethernet  HWaddr BA:F0:A3:27:B2:53  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2336 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:273011 (266.6 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:331 errors:0 dropped:0 overruns:0 frame:0
          TX packets:331 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:89520 (87.4 KiB)  TX bytes:89520 (87.4 KiB)

ra0       Link encap:Ethernet  HWaddr 00:0C:43:26:60:88  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:19506966 errors:570 dropped:570 overruns:0 frame:0
          TX packets:3871107 errors:57295 dropped:57295 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:29798518080 (27.7 GiB)  TX bytes:253500928 (241.7 MiB)

ra1       Link encap:Ethernet  HWaddr 02:0C:43:36:60:88  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

rai0      Link encap:Ethernet  HWaddr 00:0C:43:26:60:C0  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:99966624 errors:1084 dropped:1084 overruns:0 frame:0
          TX packets:47357921 errors:278811 dropped:278811 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:152961245728 (142.4 GiB)  TX bytes:3031675968 (2.8 GiB)

rax0      Link encap:Ethernet  HWaddr 00:0C:43:26:60:78  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

root@OpenWrt:~# brctl show br-wan
bridge name     bridge id               STP enabled     interfaces
br-wan          7fff.aa0e539beb46       no              eth1
root@OpenWrt:~# brctl show br-lan
bridge name     bridge id               STP enabled     interfaces
br-lan          7fff.baf0a327b253       no              apclii0
                                                        apclix0
                                                        apcli0
                                                        ra1
                                                        rai0
                                                        rax0
                                                        lan2
                                                        lan0
                                                        lan5
                                                        ra0
                                                        lan3
                                                        lan1
root@OpenWrt:~# 
```

# Accessories
## 4G/5G Module


### M.2 4G/5G Module(USB Interface)
BPI-R4_Lite supports M.2 USB Interface 4G LTE/5G Modules: **Quectel  EM05, RM500U-CN ,RM500Q-GL& RM520N-GL ** 

If you want to use M.2 Cellular Module on BPI-R4:

. Install 4G/5G Cellular Module into CN9 Slot(M.2 KEYB)
. Inset NANOSIM Card into SIMSlot(SIM1) (pay attention to the direction)
. Install antenna on the module
. After powering on, it will automatically dial

>  The availability of 4G/5G depends on the local carrier frequency band.
{.is-info}

![rm520ngl.png](/bpi-r4_lite/rm520ngl.png)
![r4_lite_sim1.png](/bpi-r4_lite/r4_lite_sim1.png)


### miniPCIe 4G/5G Module(USB Interface)
BPI-R4_Lite supports MiniPCIe USB Interface 4G LTE Module :**Quectel EC25**

If you want to use MiniPCIe Cellular Module on BPI-R4_Lite:

1. Install 4G Cellular Module into CN11 Slot
2. Inset NANOSIM Card into SIMSlot(SIM2) with card tray(pay attention to the direction)
3. Install antenna on the module
4. After powering on, it will automatically dial

**CN13 (SIM3) is also available**

>  The availability of 4G depends on the local carrier frequency band.
{.is-info}


>  __Due to the compatibility of the BPI-R4 with Qualcomm/Unisoc modules, the EC25 module cannot directly access the DNS server and connect to the internet. Therefore, manual configuration is required to modify the   **default.script**  file via console port.__

```sh
vim /usr/share/udhcpc/default.script
```
![bpi-r4_ec25e_module_modification_1.png](/bpi-r4/bpi-r4_ec25e_module_modification_1.png)

Add:  echo "nameserver 202.96.128.86" >> /etc/resolv.conf
```sh
echo "nameserver 202.96.128.86" >> /etc/resolv.conf
```
> Modify the IP address according to your local DNS server.
{.is-info}
{.is-info}

![bpi-r4_ec25e_module_modification_2.png](/bpi-r4/bpi-r4_ec25e_module_modification_2.png)}

## Storage
###  PCIe to USB
BPI-R4_Lite Also supports PCIe to USB

## Wi-Fi7 NIC
You can insert the BPI-R4-NIC into CN12 and CN14 at the bottom of BPI-R4-Main, and then fix it with two M2 screws.

The BPI-R4-NIC module requires 12V power supply, so the power supply on the BPI-R4-Main must be turned on before powering on (SW4 is turned to the "ON" position, and the 12V LED will lights up when power on)

IMPORTANT:  The 12V power supply will be supplied to the BPI-R4-NIC through PIN6/28/48 of the miniPCI socket. 
When plugging in other modules, be sure to turn off SW4 if you cannot confirm whether the module can withstand 12V.

### BPI-R4-NIC-BE14

BPI-R4-NIC-BE14 : MT7995AV+MT7976CN+MT7977IAN

### OpenWrt
OpenWRT MTK MP4.0 WiFi Setting:

When all functions are OK, we can detect Three SSIDs and the three blue LEDs on BE14 will also light up.
     
#### How to turn on WiFi hotspot

Open web and configure the corresponding STA hotspot in config.
And click Connect to connect.

## Heat sink
MTK OpwnWRT fan with PWM control reference.


# GPIO Define 
## 26 Pin GPIO define
== Hardware Spec

[options="header",cols="1,5",width="70%"]
|=====
2+| **HardWare Specification of Banana Pi BPI-CM6 RISC-V Core board**
| CPU                               |  SpacemiT K1 Otca-core X60(RV64GCVB),RVA22,RVV1.0

| AI                                |  2.0Tops from RlSC-V Core   
| GPU  | IMG BXE-2-32@819MHz,32KB SLCOpenGL ES1.1/3.2EGL1.50penCL 3.0Vulkan 1.3
| Memory                            | 8/16 GB LPDDR4 (default is 8GB)                                                                                 
| Storage                           | 8/16/32/128 GB eMMC flash.(default is 16GB)
| WiFi/BT  | support Wifi/Bt module onboard
|=====




