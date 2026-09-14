---
title: 为 Banana Pi BPI-M5 Pro 编译 Linux 镜像
description: Banana Pi BPI-M5 Pro Linux SDK 下载、编译、定制和调试指南
published: true
date: 2026-09-14T14:45:07.126Z
tags: rk3576, sdk, bpi-m5 pro, linux
editor: markdown
dateCreated: 2026-09-14T14:39:50.077Z
---

= 为 Banana Pi BPI-M5 Pro 编译 Linux 镜像

= 简介

本文介绍如何下载、编译、定制和调试 Banana Pi BPI-M5 Pro Linux SDK。

完成本文操作后，您将能够：

* 使用 `repo` 下载完整的 Linux SDK。
* 了解 SDK 的主要目录。
* 分别编译 U-Boot、Linux 内核、Debian 根文件系统和 `update.img`。
* 修改内核配置、设备树和驱动程序。
* 查找编译日志，并通过串口调试启动问题。

TIP: 有关开发板接口、供电和系统基本使用方法，请参阅 link:/zh/BPI-M5/GettingStarted_BPI-M5_Pro[BPI-M5 Pro 入门指南]。

= 准备工作

== 编译主机

请使用满足以下配置的 x86-64 计算机：

* Ubuntu 22.04 LTS x86-64
* 16 GB 或更多内存
* 120 GB 或更多可用磁盘空间
* 稳定的网络连接
* 具有 `sudo` 权限的用户账号

下载 SDK 前检查主机环境：

```sh
cat /etc/os-release
uname -m
nproc
free -h
df -h .
```

`uname -m` 应输出 `x86_64`。

== 安装编译依赖

```sh
sudo apt update
sudo apt install -y \
  git git-lfs openssh-client make gcc g++ gcc-multilib g++-multilib \
  libssl-dev liblz4-tool expect expect-dev patchelf chrpath gawk texinfo \
  diffstat binfmt-support qemu-user-static live-build bison flex fakeroot \
  cmake unzip device-tree-compiler ncurses-dev libgucharmap-2-90-dev \
  bzip2 expat gpgv2 cpp-aarch64-linux-gnu libgmp-dev libmpc-dev bc \
  python-is-python3 python3-pip python3-pyelftools u-boot-tools curl dpkg-dev
```

启用 Git LFS：

```sh
git lfs install
git lfs version
```

== 安装 repo

```sh
mkdir -p "$HOME/bin"
curl https://storage.googleapis.com/git-repo-downloads/repo -o "$HOME/bin/repo"
chmod a+x "$HOME/bin/repo"
echo 'export PATH="$HOME/bin:$PATH"' >> "$HOME/.bashrc"
export PATH="$HOME/bin:$PATH"
repo version
```

= 下载 Linux SDK

== 创建 SDK 目录

本文使用 `bpi-m5pro` 作为 SDK 目录名：

```sh
mkdir -p "$HOME/bpi-m5pro"
cd "$HOME/bpi-m5pro"
```

== 初始化并同步源码

```sh
repo init \
  -u https://github.com/ArmSoM/manifests.git \
  -b linux \
  -m armsom_linux_generic.xml

repo sync -c -j4
```

首次同步可能需要较长时间。如果网络不稳定，请减少并行任务数后重试：

```sh
.repo/repo/repo sync -c -j2
```

同步完成后检查源码目录：

```sh
test -x build.sh && echo "SDK download completed"
.repo/repo/repo status
du -sh .
```

= SDK 目录结构

[options="header",cols="2,4"]
|====
|目录 |说明
|`device/rockchip/` |板级配置、分区布局、编译钩子和打包配置
|`u-boot/` |U-Boot 源码
|`kernel-6.1/` |Linux 6.1 内核、设备树和驱动程序
|`rkbin/` |Rockchip DDR、BL31 和 Loader 二进制文件
|`prebuilts/` |交叉编译工具链
|`debian11/`、`debian12/` |Debian 根文件系统编译文件
|`ubuntu22.04/`、`ubuntu24.04/` |Ubuntu 根文件系统编译文件
|`tools/` |镜像打包和烧录工具
|`output/` |当前配置、编译日志和中间产物
|`rockdev/` |待烧录的镜像
|`build.sh` |SDK 主编译命令
|====

Linux 启动流程如下：

```text
BootROM -> Loader -> U-Boot -> Linux Kernel and DTB -> Root filesystem
```

遇到编译或启动问题时，首先确定发生故障的阶段。

= 选择 BPI-M5 Pro 配置

列出可用的 BPI-M5 Pro 配置：

```sh
cd "$HOME/bpi-m5pro"
ls device/rockchip/.chips/rk3576/*sige5*_defconfig
```

[options="header",cols="2,2,5"]
|====
|系统 |类型 |配置
|Debian 12 |Server |`armsom_sige5_rk3576_debian_server_defconfig`
|Debian 12 |XFCE |`armsom_sige5_rk3576_debian_xfce_defconfig`
|Debian 12 |GNOME |`armsom_sige5_rk3576_debian_gnome_defconfig`
|Ubuntu 22.04 |Server |`armsom_sige5_rk3576_ubuntu_server_defconfig`
|Ubuntu 22.04 |GNOME |`armsom_sige5_rk3576_ubuntu_gnome_defconfig`
|====

Debian Server 包含的桌面软件包较少，建议首次编译时选择此配置。

选择 RK3576 和 Debian Server 配置：

```sh
./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig
```

检查当前配置：

```sh
readlink -f output/defconfig
grep -E 'RK_ROOTFS_SYSTEM|RK_ROOTFS_TARGET|RK_KERNEL_PREFERRED|RK_KERNEL_CFG|RK_KERNEL_DTS_NAME' \
  output/.config
```

Debian 12 Server 的预期结果：

```text
/home/<user>/bpi-m5pro/device/rockchip/.chips/rk3576/armsom_sige5_rk3576_debian_server_defconfig
RK_ROOTFS_SYSTEM="debian"
RK_ROOTFS_TARGET_SERVER=y
RK_KERNEL_PREFERRED="6.1"
RK_KERNEL_CFG="armsom_linux_rk3576_defconfig"
RK_KERNEL_DTS_NAME="rk3576-armsom-sige5"
```

image::/bpi-m5pro/bpi-m5pro-sdk-defconfig.png[BPI-M5 Pro SDK defconfig 和当前内核配置]

所选配置使用以下三个源文件：

[options="header",cols="3,6"]
|====
|用途 |需要修改的文件
|Debian 镜像类型、内核选择、分区和打包选项 |`device/rockchip/.chips/rk3576/armsom_sige5_rk3576_debian_server_defconfig`
|Linux 内核选项 |`kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig`
|BPI-M5 Pro 板级硬件描述 |`kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts`
|====

`output/.config` 和 `output/defconfig` 是生成的当前配置。如需永久保留修改，请修改上表中的源文件，必要时重新选择板级配置。

WARNING: 开始编译前应始终先选择板级配置。如果使用其他开发板遗留的配置编译，可能生成无法使用的镜像。

= 编译镜像

== 一键编译

使用以下命令编译完整镜像：

```sh
./build.sh
```

该命令会编译所需组件并打包最终镜像。首次编译或排查问题时，可以使用后续章节中的单项命令，分别确认各组件的输入、输出和编译错误。

== 查看编译命令

```sh
./build.sh help
./build.sh kernel:dry-run
```

`dry-run` 会显示内核编译命令和交叉编译器，但不会真正编译内核。

== 编译 U-Boot

```sh
./build.sh uboot
```

检查输出：

```sh
ls -lh u-boot/uboot.img
ls -lh output/firmware/MiniLoaderAll.bin output/firmware/uboot.img
```

== 编译 Linux 内核

```sh
./build.sh kernel
```

内核编译会生成内核镜像、模块、DTB、boot 镜像和 Debian 软件包。

```sh
ls -lh kernel-6.1/extboot.img output/firmware/boot.img
find . -maxdepth 1 -name 'linux-*.deb' -printf '%f\n' | sort
```

== 编译 Debian 根文件系统

```sh
./build.sh debian
```

此步骤需要下载 Debian 软件包，所需时间可能远长于 U-Boot 和内核编译。

检查结果：

```sh
find output -path '*debian*' -name 'rootfs*' -type f -ls
```

== 打包固件

U-Boot、内核和根文件系统准备完成后，打包分区镜像：

```sh
./build.sh firmware
```

生成完整升级镜像：

```sh
./build.sh updateimg
ls -lh rockdev/
sha256sum rockdev/update.img
```

[options="header",cols="2,4"]
|====
|文件 |说明
|`MiniLoaderAll.bin` |Loader 镜像
|`uboot.img` |U-Boot 分区镜像
|`boot.img` |Linux 内核、DTB 和启动文件
|`rootfs.img` |Debian 或 Ubuntu 根文件系统
|`parameter.txt` |烧录分区布局
|`update.img` |完整升级镜像
|====

= 编译日志

每次编译都会在 `output/sessions/` 下创建一个会话目录。

```sh
ls -lt output/sessions/ | head
readlink -f output/sessions/latest
find output/sessions/latest -maxdepth 1 -name '*.log' -type f -print
```

在最新日志中搜索错误：

```sh
grep -RniE 'error:|fatal:|failed|no such file|no rule to make' \
  output/sessions/latest | tail -n 50
```

编译失败时，按照以下顺序排查：

. 查找日志中的第一条具体错误。
. 确定错误属于 U-Boot、内核、根文件系统还是打包阶段。
. 查看第一条错误前后约 30 行内容。
. 修复问题，并且只重新编译失败的组件。
. 记录命令、错误、原因和解决方法。

不要每次出现错误都运行 `cleanall`。增量编译可以保留有用的中间结果，而且速度更快。

= 内核开发

== 修改内核配置

BPI-M5 Pro 使用 Linux 6.1 时，内核 defconfig 为：

```text
kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig
```

通过当前 SDK 配置确认该文件：

```sh
grep '^RK_KERNEL_CFG=' output/.config
test -f kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig \
  && echo "Kernel defconfig found"
```

预期结果：

```text
RK_KERNEL_CFG="armsom_linux_rk3576_defconfig"
Kernel defconfig found
```

使用 SDK 命令修改内核选项：

```sh
./build.sh kernel-config
```

保存并退出 `menuconfig` 后，SDK 会运行 `savedefconfig`，并将结果写回：

```text
kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig
```

在 `menuconfig` 中：

* 按 `/` 搜索配置符号。
* 按空格键在内建、模块和禁用之间切换。
* `[*]` 表示编入内核。
* `[M]` 表示编译为模块。

保存配置后，检查修改并重新编译：

```sh
git -C kernel-6.1 status --short
git -C kernel-6.1 diff -- arch/arm64/configs/armsom_linux_rk3576_defconfig
./build.sh kernel
./build.sh firmware
```

只有需要完整升级镜像时，才重新生成 `update.img`。

== 修改设备树

首先读取当前选择的 DTB 名称：

```sh
grep '^RK_KERNEL_DTS_NAME=' output/.config
```

预期结果：

```text
RK_KERNEL_DTS_NAME="rk3576-armsom-sige5"
```

该值直接对应以下源文件：

```text
kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts
```

确认文件及其 DTB 编译入口：

```sh
DTS=kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts
realpath "$DTS"
sed -n '7,17p' "$DTS"
rg -n 'rk3576-armsom-sige5' \
  kernel-6.1/arch/arm64/boot/dts/rockchip/Makefile
```

关键预期输出：

```text
#include "rk3576.dtsi"
#include "rk3576-rk806.dtsi"
#include "rk3576-linux.dtsi"
7:dtb-$(CONFIG_ARCH_ROCKCHIP) += rk3576-armsom-sige5.dtb
```

image::/bpi-m5pro/bpi-m5pro-kernel-dts.png[BPI-M5 Pro 当前 DTS、包含文件和 DTB 编译入口]

修改 BPI-M5 Pro 板级硬件时，请编辑 `rk3576-armsom-sige5.dts`。常见项目包括：

* `leds` 节点下的 GPIO LED。
* `&gmac0` 和 `&gmac1` 下的以太网。
* `&i2c0` 和 `&i2c2` 下的 RTC 和 Type-C 控制器。
* `&i2c3` 下的音频编解码器。
* `&pcie0` 下的 NVMe。
* `&pinctrl` 下的引脚复用。

修改前定位这些节点：

```sh
rg -n '^\s*(leds:|&gmac0|&gmac1|&i2c0|&i2c2|&i2c3|&pcie0|&pinctrl)' "$DTS"
```

当前 SDK 源码的输出：

```text
72:    leds: leds {
319:&gmac0 {
343:&gmac1 {
386:&i2c0 {
425:&i2c2 {
501:&i2c3 {
557:&pcie0 {
563:&pinctrl {
```

包含的文件提供公共定义：

* `rk3576.dtsi`：RK3576 SoC 设备、地址、中断和时钟。
* `rk3576-rk806.dtsi`：RK806 PMIC 和稳压器定义。
* `rk3576-linux.dtsi`：RK3576 的 Linux 公共配置。

通常应在 `rk3576-armsom-sige5.dts` 中添加或覆盖板级外设。只有修改确实适用于公共范围时，才修改包含的 DTSI 文件。

修改板级 DTS 后：

```sh
git -C kernel-6.1 diff -- arch/arm64/boot/dts/rockchip/
./build.sh kernel
./build.sh firmware
```

在开发板上验证设备树：

```sh
cat /proc/device-tree/model; echo
dmesg | grep -iE 'gpio|pinctrl|i2c|spi|uart|mmc|pcie'
```

如果驱动未执行 probe，请检查 `status`、`compatible`、时钟、复位、GPIO、稳压器，以及实际加载的 DTB。

== 修改内核驱动

在 probe 和资源初始化代码附近添加简洁的日志：

```c
dev_info(dev, "probe started\n");
dev_err(dev, "failed to enable clock: %d\n", ret);
```

修改驱动后重新编译内核：

```sh
./build.sh kernel
./build.sh firmware
```

在开发板上使用以下命令：

```sh
uname -a
dmesg -w
journalctl -k -b
lsmod
modinfo <module-name>
systemctl --failed
```

= U-Boot 和串口调试

修改 `u-boot/` 中的文件后，重新编译 U-Boot：

```sh
./build.sh uboot
```

常用的 U-Boot 调试方法包括：

* 在发生故障的初始化路径附近添加 `printf()`。
* 启用所需的 `CONFIG_DEBUG_*` 选项。
* 中断自动启动，然后运行 `printenv`、`help` 和 `mmc list`。
* 检查 `bootcmd`、`bootargs`、加载地址和当前启动设备。

连接 3.3 V USB 转 TTL 串口适配器：

[options="header",cols="1,1,1"]
|====
|BPI-M5 Pro |连接 |串口适配器
|GND |<---> |GND
|TX |---> |RX
|RX |<--- |TX
|====

波特率使用 `1500000`。请从开发板上电前开始记录完整日志。

如果没有串口输出，请先检查电源、线缆连接、波特率、启动介质和 Loader，再调试 Linux 驱动。

= 根文件系统开发

根文件系统包含软件包、服务、用户、系统配置和应用程序。修改根文件系统不需要重新编译 U-Boot 或内核。

修改前先搜索根文件系统编译文件：

```sh
rg -n '<package-or-file-name>' \
  debian11 debian12 ubuntu22.04 ubuntu24.04
```

重新编译并打包 Debian 修改：

```sh
./build.sh debian
./build.sh firmware
./build.sh updateimg
```

在开发板上验证服务：

```sh
systemctl --failed
systemctl status <service-name>
journalctl -u <service-name> -b
ip address
mount
df -h
```

= 烧录并验证镜像

WARNING: 烧录会覆盖目标存储设备上的数据。继续操作前请备份重要文件。

TIP: 完整烧录方法请参阅 link:/zh/BPI-M5/BananaPi_Flash_image[香蕉派 Rockchip 开发板镜像烧录指南]。

检查开发板是否处于 Loader 或 Maskrom 模式：

```sh
sudo tools/linux/Linux_Upgrade_Tool/Linux_Upgrade_Tool/upgrade_tool ld
```

烧录完整镜像：

```sh
sudo ./rkflash.sh update
```

开发过程中，如果分区表未改变，可以单独更新某个分区：

```sh
sudo ./rkflash.sh uboot
sudo ./rkflash.sh boot
sudo ./rkflash.sh rootfs
```

启动新镜像后，执行基本验证：

```sh
uname -a
cat /proc/device-tree/model; echo
cat /etc/os-release
ip address
df -h
dmesg -T | grep -iE 'error|fail|timeout'
systemctl --failed
```

= 保存并复现修改

SDK 包含多个 Git 仓库。同步前检查所有修改：

```sh
.repo/repo/repo status
.repo/repo/repo diff
git -C kernel-6.1 status --short
git -C kernel-6.1 diff
```

生成锁定修订版本的 manifest，以便复现编译：

```sh
.repo/repo/repo manifest -r -o bpi-m5pro-build-manifest.xml
```

每次发布镜像时，请记录：

* Manifest 修订版本
* BPI-M5 Pro 配置
* 修改的仓库和文件
* 完整编译命令
* `update.img` 的 SHA-256 校验值
* 串口启动日志
* 开发板验证结果

= 故障排查

== 找不到板级配置

如果出现以下错误：

```text
No available defconfigs for: armsom_sige5_rk3576_debian_server_defconfig
```

同时选择 RK3576 和配置：

```sh
./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig
```

== 内核报告 `No rule to make target 'goodix_ts.o'`

检查内核配置、源文件和 Makefile：

```sh
grep CONFIG_TOUCHSCREEN_GOODIX kernel-6.1/.config
ls kernel-6.1/drivers/input/touchscreen/goodix*
grep -n goodix_ts kernel-6.1/drivers/input/touchscreen/Makefile
```

如果源文件存在，并且组合规则被注释，请将：

```make
# goodix_ts-y := goodix.o goodix_fwupload.o
```

修改为：

```make
goodix_ts-y := goodix.o goodix_fwupload.o
```

然后再次执行增量编译：

```sh
./build.sh kernel
```

== Debian 软件包下载失败

```sh
date
getent hosts deb.debian.org
env | grep -i proxy
```

修复网络或镜像源后，只重新编译根文件系统：

```sh
./build.sh debian
```

== 修改未在开发板上生效

按以下顺序检查：

. 确认 `git diff` 包含预期修改。
. 确认对应组件已完成编译。
. 检查 `rockdev/` 中的文件修改时间和校验值。
. 确认烧录了正确的分区。
. 确认开发板当前运行的内核、DTB 或应用程序版本。

= 常用命令

```sh
cd "$HOME/bpi-m5pro"

.repo/repo/repo sync -c -j4
./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig

./build.sh uboot
./build.sh kernel
./build.sh debian
./build.sh firmware
./build.sh updateimg

./build.sh kernel-config

.repo/repo/repo status
.repo/repo/repo diff
ls -lh rockdev/
sha256sum rockdev/update.img
```
