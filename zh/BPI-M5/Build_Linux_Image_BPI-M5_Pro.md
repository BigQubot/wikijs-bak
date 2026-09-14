---
title: Banana Pi BPI-M5 Pro Linux 镜像编译与调试教程
description: Banana Pi BPI-M5 Pro Linux SDK 下载、编译、定制和调试实操教程
published: true
date: 2026-09-14T14:39:50.077Z
tags: rk3576, sdk, bpi-m5 pro, linux, 编译教程
editor: markdown
dateCreated: 2026-09-14T14:39:50.077Z
---

# Banana Pi BPI-M5 Pro Linux 镜像编译与调试教程

本文是一份可跟着操作的新人培训。目标是完成一次真实构建，并知道修改 U-Boot、内核、设备树或根文件系统后应该重新编译什么、去哪里看日志、怎样判断产物是否正确。

## 1. 培训目标

完成本文后，应能够：

1. 在 Ubuntu 主机上搭建 BPI-M5Pro SDK 环境；
2. 使用 `repo` 下载和更新 SDK；
3. 说清楚 SDK 主要目录的用途；
4. 编译 Debian 或 Ubuntu 完整固件；
5. 单独编译 U-Boot、Linux 内核和 rootfs；
6. 修改内核配置或设备树并完成增量验证；
7. 从构建日志、串口日志和系统日志定位问题；
8. 保存修改，避免更新 SDK 时丢失代码。

目标板为 Banana Pi BPI-M5Pro，主控为 Rockchip RK3576，当前 SDK 默认使用 Linux 6.1。

## 2. 开始前先理解 SDK

### 2.1 系统由什么组成

SDK 是构建一套可启动系统所需源码和工具的集合：

- Loader 和 U-Boot：初始化硬件并加载内核；
- Linux Kernel：管理 CPU、内存和硬件驱动；
- Device Tree：描述板卡的硬件连接；
- Rootfs：Debian 或 Ubuntu 用户空间；
- 打包工具：生成分区镜像和 `update.img`。

启动关系可以简化为：

```text
BootROM → Loader → U-Boot → Linux Kernel + DTB → Rootfs → 用户程序
```

出现问题时先判断故障发生在哪一层，比反复全量编译更重要。

### 2.2 repo 是什么

SDK 由多个 Git 仓库组成。`repo` 根据 manifest 统一管理这些仓库：

```bash
repo sync -c -j4    # 同步源码
repo status         # 查看各仓库状态
repo diff           # 查看各仓库未提交的差异
```

## 3. 实操一：准备编译主机

### 3.1 推荐配置

- Ubuntu 22.04 x86_64；
- 16 GB 或更多内存；
- 120 GB 以上可用磁盘；
- 稳定网络；
- 普通用户具有 `sudo` 权限。

不要使用 root 用户完成整个编译，只在安装软件包等必要操作时使用 `sudo`。

### 3.2 检查环境

```bash
cat /etc/os-release
uname -m
nproc
free -h
df -h .
```

确认 `uname -m` 输出 `x86_64`，内存最好不低于 16 GB，磁盘剩余空间最好大于 120 GB。

### 3.3 安装依赖

```bash
sudo apt update
sudo apt install -y \
  git git-lfs openssh-client make gcc g++ gcc-multilib g++-multilib \
  libssl-dev liblz4-tool expect expect-dev patchelf chrpath gawk texinfo \
  diffstat binfmt-support qemu-user-static live-build bison flex fakeroot \
  cmake unzip device-tree-compiler ncurses-dev libgucharmap-2-90-dev \
  bzip2 expat gpgv2 cpp-aarch64-linux-gnu libgmp-dev libmpc-dev bc \
  python-is-python3 python3-pip python3-pyelftools u-boot-tools curl dpkg-dev
```

```bash
git lfs install
git --version
git lfs version
python -V
gcc --version
```

Python 应为 Python 3。Ubuntu 22.04 通常不再提供 `python2`，本教程也不需要额外安装 Python 2。

### 3.4 安装 repo

```bash
mkdir -p "$HOME/bin"
curl https://storage.googleapis.com/git-repo-downloads/repo -o "$HOME/bin/repo"
chmod a+x "$HOME/bin/repo"
echo 'export PATH="$HOME/bin:$PATH"' >> "$HOME/.bashrc"
export PATH="$HOME/bin:$PATH"
repo version
```

如果上述下载地址不可访问，可尝试：

```bash
sudo apt install -y repo
```

### 3.5 配置 Git 身份

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

只做本地编译时，不要求与 GitHub 账号一致。

## 4. 实操二：下载 SDK

### 4.1 创建工作目录

本文统一使用 `bpi-m5pro`：

```bash
mkdir -p "$HOME/bpi-m5pro"
cd "$HOME/bpi-m5pro"
pwd
```

后续命令如无特别说明，都在 `bpi-m5pro` 顶层执行。

如果已经进入准备存放 SDK 的空目录，也可以直接在当前目录初始化：

```bash
pwd
test -z "$(find . -mindepth 1 -maxdepth 1 -print -quit)" \
  && echo "当前目录为空，可以初始化"
```

本次实操使用的就是当前工作区 `/home/qubot/bpi/BPI-M5Pro`。已有代码的非空目录不要直接初始化，避免文件相互覆盖。

### 4.2 初始化 manifest

```bash
repo init \
  -u https://github.com/ArmSoM/manifests.git \
  -b linux \
  -m armsom_linux_generic.xml
```

成功时会看到：

```text
repo has been initialized in .../bpi-m5pro
```

此时只创建了 `.repo/`，并未下载全部 SDK。

### 4.3 同步源码

```bash
repo sync -c -j4
```

- `-c`：只同步 manifest 指定分支；
- `-j4`：同时执行 4 个下载任务；
- 网络不稳定时可降为 `-j2`；
- 中断后重复执行同一命令即可继续。

同步完成后检查：

```bash
repo status
test -x build.sh && echo "SDK 下载完成"
```

SDK 初始化完成后，也可以不依赖全局命令，直接使用：

```bash
.repo/repo/repo sync -c -j4
```

如果要使用 Ubuntu 24.04 仓库中的 Git LFS 文件：

```bash
cd "$HOME/bpi-m5pro/ubuntu24.04"
git lfs pull
cd "$HOME/bpi-m5pro"
```

## 5. 实操三：认识 SDK 目录

同步完成后先观察目录：

```bash
cd "$HOME/bpi-m5pro"
find . -maxdepth 1 -mindepth 1 -printf '%f\n' | sort
```

| 路径 | 用途 | 常见修改场景 |
| --- | --- | --- |
| `.repo/` | repo 元数据和 manifest | 一般不手工修改 |
| `device/rockchip/` | 板级配置、构建脚本、分区配置 | 新增板型或改打包配置 |
| `u-boot/` | U-Boot 源码 | 修改启动流程、介质和启动日志 |
| `kernel-6.1/` | Linux 6.1 源码 | 修改驱动、DTS 和内核配置 |
| `kernel-5.10/` | Linux 5.10 源码 | BPI-M5Pro 默认不用 |
| `rkbin/` | DDR、BL31 等 Rockchip 二进制 | 通常不修改 |
| `prebuilts/` | 交叉编译工具链 | 通常不修改 |
| `debian11/`、`debian12/` | Debian rootfs 构建 | 定制 Debian |
| `ubuntu22.04/`、`ubuntu24.04/` | Ubuntu rootfs 构建 | 定制 Ubuntu |
| `tools/` | 打包和烧录工具 | 通常不修改 |
| `build.sh` | SDK 编译入口 | 日常使用，不建议直接改 |
| `output/` | 配置、日志和中间产物 | 编译后生成 |
| `rockdev/` | 待烧录的最终镜像 | 编译后生成 |

关键关系：

```text
device/rockchip/.chips/rk3576/     BPI-M5Pro SDK 配置
kernel-6.1/arch/arm64/...          驱动、DTS、内核 defconfig
output/                            构建过程和日志
rockdev/                           最终镜像
```

`kernel`、`output/defconfig`、`output/firmware/*.img` 中有一部分是符号链接。排查“文件明明存在却指向错误版本”时使用：

```bash
readlink -f kernel
readlink -f output/defconfig
find output/firmware -maxdepth 1 -type l -printf '%f -> %l\n'
```

### 5.1 找出板级配置

```bash
ls device/rockchip/.chips/rk3576/*sige5*_defconfig
```

| 系统 | 类型 | 配置名 |
| --- | --- | --- |
| Debian | Server | `armsom_sige5_rk3576_debian_server_defconfig` |
| Debian | XFCE | `armsom_sige5_rk3576_debian_xfce_defconfig` |
| Debian | GNOME | `armsom_sige5_rk3576_debian_gnome_defconfig` |
| Ubuntu 22.04 | Server | `armsom_sige5_rk3576_ubuntu_server_defconfig` |
| Ubuntu 22.04 | GNOME | `armsom_sige5_rk3576_ubuntu_gnome_defconfig` |

第一次培训推荐 Debian Server：没有桌面，下载和构建通常更快，日志也更容易观察。

### 5.2 阅读配置

```bash
sed -n '1,120p' \
  device/rockchip/.chips/rk3576/armsom_sige5_rk3576_debian_server_defconfig
```

重点认识：

```text
RK_ROOTFS_TARGET_SERVER=y
RK_KERNEL_PREFERRED="6.1"
RK_KERNEL_CFG="armsom_linux_rk3576_defconfig"
RK_KERNEL_DTS_NAME="rk3576-armsom-sige5"
RK_KERNEL_EXTBOOT=y
```

它们分别确定 rootfs 类型、内核版本、内核配置、设备树和 boot 分区形式。

## 6. 实操四：完成第一次编译

### 6.1 选择配置

```bash
cd "$HOME/bpi-m5pro"
./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig
```

首次必须同时指定 `rk3576`。新同步的 SDK 可能仍指向其他芯片目录，如果直接执行 defconfig，会出现：

```text
No available defconfigs for: armsom_sige5_rk3576_debian_server_defconfig
```

完成一次 RK3576 选择后，后续切换同芯片的系统配置才可以只写 defconfig。也可以先运行 `./build.sh chip`，依次选择 RK3576 和 BPI-M5Pro 对应配置。

检查实际选择：

```bash
readlink -f output/defconfig
grep -E 'RK_ROOTFS_SYSTEM|RK_ROOTFS_TARGET|RK_KERNEL_PREFERRED|RK_KERNEL_CFG|RK_KERNEL_DTS_NAME' \
  output/.config
```

Debian Server 配置的预期输出为：

```text
/home/<用户名>/bpi-m5pro/device/rockchip/.chips/rk3576/armsom_sige5_rk3576_debian_server_defconfig
RK_ROOTFS_SYSTEM="debian"
RK_ROOTFS_TARGET_SERVER=y
RK_KERNEL_PREFERRED="6.1"
RK_KERNEL_CFG="armsom_linux_rk3576_defconfig"
RK_KERNEL_DTS_NAME="rk3576-armsom-sige5"
```

![BPI-M5Pro 板级 defconfig 与当前内核配置](/bpi-m5pro/bpi-m5pro-sdk-defconfig.png)

本配置明确对应三个需要长期保存修改的文件：

| 用途 | 修改文件 |
| --- | --- |
| Debian 类型、内核选择、分区和打包 | `device/rockchip/.chips/rk3576/armsom_sige5_rk3576_debian_server_defconfig` |
| Linux 内核选项 | `kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig` |
| BPI-M5Pro 板级硬件描述 | `kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts` |

`output/.config` 和 `output/defconfig` 是构建系统生成的当前选择，不应把长期修改只保存在这里。

也可以运行 `./build.sh chip` 交互选择，但团队构建记录应写完整配置名，便于复现。

### 6.2 一键编译

直接构建完整镜像：

```bash
./build.sh
```

该命令会编译所需组件并打包最终镜像。新人第一次操作或排查故障时，建议按照下面的分步命令执行，以便看清每个模块的输入、输出和故障边界。

### 6.3 先看帮助和 dry-run

```bash
./build.sh help
./build.sh kernel:dry-run
```

`dry-run` 显示将调用的内核构建命令，方便学习交叉编译参数，不会代替真正编译。

### 6.4 编译 U-Boot

```bash
./build.sh uboot
```

```bash
ls -lh u-boot/uboot.img
ls -lh output/firmware/MiniLoaderAll.bin output/firmware/uboot.img
```

如果这里失败，重点检查工具链、`rkbin`、U-Boot 配置和 U-Boot 源码，不要先修改 rootfs。

### 6.5 编译 Linux 内核

```bash
./build.sh kernel
```

```bash
ls -lh kernel-6.1/extboot.img output/firmware/boot.img
find . -maxdepth 1 -name 'linux-*.deb' -printf '%f\n' | sort
```

内核阶段会生成内核、模块、DTB、boot 镜像及内核 DEB 包。

### 6.6 编译 Debian rootfs

```bash
./build.sh debian
```

此阶段需要联网下载软件包。失败时先检查日志中的第一个错误，再确认 DNS、代理、软件源和系统时间。

```bash
find output -path '*debian*' -name 'rootfs*' -type f -ls
```

### 6.7 打包固件

```bash
./build.sh firmware
./build.sh updateimg
ls -lh rockdev/
```

重点产物：

| 文件 | 作用 |
| --- | --- |
| `MiniLoaderAll.bin` | Loader |
| `uboot.img` | U-Boot 分区 |
| `boot.img` | 内核、启动文件和 DTB |
| `rootfs.img` | Debian/Ubuntu 根文件系统 |
| `parameter.txt` | 分区表 |
| `update.img` | 整包烧录镜像 |

## 7. 学会看编译日志

```bash
find output -maxdepth 4 -type f -name '*.log' -printf '%p\n' | sort
ls -l output/log output/sessions/latest 2>/dev/null
```

搜索最新错误：

```bash
find output/log -type f -name '*.log' -print0 2>/dev/null \
  | xargs -0 grep -nEi 'error:|fatal:|failed|no such file' \
  | tail -n 50
```

排错顺序：

1. 找终端中的第一个错误，不只看最后一行；
2. 判断是 U-Boot、kernel、rootfs 还是 firmware；
3. 查看对应日志中错误前后约 30 行；
4. 只清理并重编失败模块；
5. 记录命令、错误、原因和解决方法。

不要一失败就运行 `cleanall`，否则会丢失有用的中间状态并浪费时间。

## 8. 实操五：修改内核配置

BPI-M5Pro 的 Linux 6.1 内核配置文件是：

```text
kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig
```

先确认当前选择：

```bash
grep '^RK_KERNEL_CFG=' output/.config
test -f kernel-6.1/arch/arm64/configs/armsom_linux_rk3576_defconfig \
  && echo "Kernel defconfig found"
```

预期输出：

```text
RK_KERNEL_CFG="armsom_linux_rk3576_defconfig"
Kernel defconfig found
```

```bash
cd "$HOME/bpi-m5pro"
./build.sh kernel-config
```

保存并退出 `menuconfig` 后，脚本会运行 `savedefconfig`，并将结果写回上述内核 defconfig。

在 `menuconfig` 中：

- `/`：搜索配置项；
- 空格：切换内建、模块或关闭；
- `[*]`：编入内核；
- `[M]`：编译为模块。

保存退出后查看变化：

```bash
git -C kernel-6.1 status --short
git -C kernel-6.1 diff -- arch/arm64/configs/armsom_linux_rk3576_defconfig
```

增量构建：

```bash
./build.sh kernel
./build.sh firmware
./build.sh updateimg
```

只改内核时不需要重新构建 rootfs。上板后验证：

```bash
uname -a
zcat /proc/config.gz | grep CONFIG_目标配置名
```

若没有 `/proc/config.gz`，检查 `/boot/config-*`。不能只看编译成功，还要确认开发板实际启动了新内核。

## 9. 实操六：修改设备树

### 9.1 找到主设备树

```bash
cd "$HOME/bpi-m5pro"
grep '^RK_KERNEL_DTS_NAME=' output/.config
```

预期输出：

```text
RK_KERNEL_DTS_NAME="rk3576-armsom-sige5"
```

对应的主设备树源码是：

```text
kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts
```

继续确认文件、包含关系和 DTB 构建入口：

```bash
DTS=kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts
realpath "$DTS"
sed -n '7,17p' "$DTS"
rg -n 'rk3576-armsom-sige5' \
  kernel-6.1/arch/arm64/boot/dts/rockchip/Makefile
```

关键输出如下：

```text
#include "rk3576.dtsi"
#include "rk3576-rk806.dtsi"
#include "rk3576-linux.dtsi"
7:dtb-$(CONFIG_ARCH_ROCKCHIP) += rk3576-armsom-sige5.dtb
```

![BPI-M5Pro 主 DTS、包含文件与 DTB 构建入口](/bpi-m5pro/bpi-m5pro-kernel-dts.png)

普通板级外设修改应修改这个 DTS。LED、双网口、I2C、音频、PCIe 和 pinctrl 等节点都在该文件中定义或覆盖。先定位常用节点：

```bash
DTS=kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-armsom-sige5.dts
rg -n '^\s*(leds:|&gmac0|&gmac1|&i2c0|&i2c2|&i2c3|&pcie0|&pinctrl)' "$DTS"
```

当前源码中的运行结果为：

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

该 DTS 包含 `rk3576.dtsi`、`rk3576-rk806.dtsi` 和 `rk3576-linux.dtsi`。它们分别提供 SoC、PMIC 和 Linux 公共定义；只有修改确实属于公共范围时才改 DTSI。

### 9.2 查看差异并编译

新人第一次可先增加一行注释，确认 `git diff` 能看到变化后再恢复。不要用供电、电压或存储控制器节点做第一次试验。

```bash
git -C kernel-6.1 status --short
git -C kernel-6.1 diff -- arch/arm64/boot/dts/rockchip/
./build.sh kernel
./build.sh firmware
```

只更新 boot 分区时不必重编 rootfs；准备交付整包时再执行 `./build.sh updateimg`。

### 9.3 上板验证

```bash
cat /proc/device-tree/model; echo
dmesg | grep -iE 'gpio|pinctrl|i2c|spi|uart|mmc|pcie'
ls -l /boot/rk-kernel.dtb /boot/dtb/ 2>/dev/null
```

驱动没有 probe 时依次检查：

1. 节点 `status` 是否为 `okay`；
2. `compatible` 是否与驱动匹配；
3. 时钟、复位、GPIO 和 regulator 是否正确；
4. 新 DTB 是否真的被打包和加载；
5. `dmesg` 是否有 deferred probe 或资源冲突。

## 10. U-Boot 调试

修改目录为 `u-boot/`，修改后执行：

```bash
./build.sh uboot
```

常见方法：

- 在关键路径增加 `printf()`；
- 打开相应 `CONFIG_DEBUG_*`；
- 串口打断自动启动后使用 `printenv`、`help`、`mmc list`；
- 检查 `bootcmd`、`bootargs`、加载地址和实际启动介质。

串口注意事项：

- 使用 3.3 V TTL，不能接 5 V；
- 板端 TX 接转换器 RX，板端 RX 接转换器 TX，并连接 GND；
- RK3576 BSP 常用 1500000 波特率，无可读输出时应核对原理图和当前配置；
- 保存从上电开始的完整日志。

完全没有串口输出时，先查供电、接线、波特率、启动介质和 Loader，不要先改 Linux 驱动。

## 11. Linux 驱动调试

驱动日志应包含上下文和返回值：

```c
dev_info(dev, "probe started\n");
dev_err(dev, "failed to enable clock: %d\n", ret);
```

修改驱动后：

```bash
./build.sh kernel
./build.sh firmware
```

准备完整固件时再运行 `./build.sh updateimg`。也可把 SDK 顶层生成的非 debug `linux-image-*.deb` 和 `linux-headers-*.deb` 复制到板上安装：

```bash
sudo dpkg -i linux-image-*.deb linux-headers-*.deb
sudo reboot
```

安装前备份 `/boot`，并确保串口或恢复方式可用。

上板常用命令：

```bash
dmesg -w
journalctl -k -b
lsmod
modinfo 模块名
dmesg -T | grep -iE 'error|fail|timeout|defer'
systemctl --failed
```

- 没有 probe 日志：查 DTS、compatible 和内核配置；
- probe 返回错误：查资源、时钟、电源、复位和依赖驱动；
- 模块不存在：查配置和模块是否进入 boot/rootfs；
- 主机有新产物、板上没变化：查烧录分区和实际内核版本。

## 12. Rootfs 定制和调试

rootfs 负责软件包、服务、用户配置和应用。只改 rootfs 时不需要重编 U-Boot 和内核。

```bash
./build.sh debian
# 或者，在选择 Ubuntu 配置后：
./build.sh ubuntu
```

先搜索目标内容在哪里生成：

```bash
rg -n '目标软件包名|目标配置文件名' \
  debian11 debian12 ubuntu22.04 ubuntu24.04
```

重新生成 rootfs 后：

```bash
./build.sh firmware
./build.sh updateimg
```

板上检查：

```bash
systemctl --failed
systemctl status 服务名
journalctl -u 服务名 -b
ip address
mount
df -h
```

## 13. 烧录和上板验收

烧录会覆盖目标存储。先确认产物并记录校验值：

```bash
ls -lh rockdev/
sha256sum rockdev/update.img
```

先让 BPI-M5Pro 进入 Loader 或 Maskrom 模式，再检查 USB 设备：

```bash
sudo tools/linux/Linux_Upgrade_Tool/Linux_Upgrade_Tool/upgrade_tool ld
```

确认只连接了目标开发板且重要数据已备份后，整包烧录：

```bash
sudo ./rkflash.sh update
```

调试时可以只更新变化分区：

```bash
sudo ./rkflash.sh uboot
sudo ./rkflash.sh boot
sudo ./rkflash.sh rootfs
```

前提是分区表没有变化。修改 `parameter.txt` 后不能机械地只刷单个分区。

上板最小验收：

```bash
uname -a
cat /proc/device-tree/model; echo
cat /etc/os-release
ip address
df -h
dmesg -T | grep -iE 'error|fail|timeout'
systemctl --failed
```

## 14. 保存修改和团队复现

```bash
cd "$HOME/bpi-m5pro"
repo status
repo diff
git -C kernel-6.1 status
git -C kernel-6.1 diff
```

每个需求至少记录：

- manifest 版本；
- BPI-M5Pro defconfig；
- 修改的仓库和文件；
- 完整编译命令；
- 镜像 SHA-256；
- 串口启动日志；
- 上板验收结果。

生成精确 manifest：

```bash
.repo/repo/repo manifest -r -o bpi-m5pro-build-manifest.xml
```

## 15. 更新和清理

更新前先运行 `repo status` 和 `repo diff` 保存修改，然后：

```bash
cd "$HOME/bpi-m5pro/.repo/manifests"
git checkout linux
git pull --ff-only

cd "$HOME/bpi-m5pro"
.repo/repo/repo sync -c -j4
```

release manifest 固定了各仓库提交。除专项开发外，不要随意只对某个子仓库 `git pull`，以免组件版本不匹配。

清理命令：

```bash
./build.sh help
./build.sh clean-kernel
./build.sh clean-loader
./build.sh cleanall
```

优先清理单个失败模块。只有确实需要从头构建时才使用 `cleanall`。

## 16. 常见故障

### 16.1 找不到 repo

```bash
export PATH="$HOME/bin:$PATH"
repo version
```

或在已初始化的 SDK 中运行 `.repo/repo/repo version`。

### 16.2 找不到板级配置

```bash
ls device/rockchip/.chips/rk3576/*sige5*_defconfig
```

无输出通常表示源码未完整同步或使用了错误 manifest。

### 16.3 配置选错

```bash
./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig
grep -E 'RK_ROOTFS_SYSTEM|RK_ROOTFS_TARGET|RK_KERNEL_DTS_NAME' output/.config
```

### 16.4 repo sync 失败

```bash
.repo/repo/repo sync -c -j2
```

先降低并发重试，再检查 DNS、代理、磁盘空间和具体失败仓库。

### 16.5 rootfs 下载失败

```bash
date
getent hosts deb.debian.org
env | grep -i proxy
```

修复网络后只重试 `./build.sh debian` 或 `./build.sh ubuntu`。

### 16.6 镜像超过分区大小

`firmware` 会按 `parameter.txt` 检查镜像大小。应精简 rootfs，或理解 Rockchip 分区布局后再调整分区，不能跳过检查。

### 16.7 修改没有在板上生效

依次确认：

1. `git diff` 确实有修改；
2. 对应模块已重新编译成功；
3. `rockdev/` 文件时间和校验值已变化；
4. 烧录的是正确分区；
5. 板上内核、DTB 或程序版本属于新构建。

### 16.8 内核提示 `No rule to make target 'goodix_ts.o'`

当前 SDK 实编时遇到过：

```text
No rule to make target 'drivers/input/touchscreen/goodix_ts.o'
```

先验证配置、源文件和构建规则：

```bash
grep CONFIG_TOUCHSCREEN_GOODIX kernel-6.1/.config
ls kernel-6.1/drivers/input/touchscreen/goodix*
grep -n goodix_ts kernel-6.1/drivers/input/touchscreen/Makefile
```

实测结果是 `CONFIG_TOUCHSCREEN_GOODIX=y`，`goodix.c` 和
`goodix_fwupload.c` 均存在，但组合目标的规则被注释。将 Makefile 中：

```make
# goodix_ts-y := goodix.o goodix_fwupload.o
```

恢复为：

```make
goodix_ts-y := goodix.o goodix_fwupload.o
```

然后重试：

```bash
./build.sh kernel
```

增量构建会复用已完成的目标，不必执行 `cleanall`。本例也说明应优先找日志中的第一条具体错误；末尾的 `run_build_hooks failed` 只是上层脚本汇总，不是根因。

## 17. 新人培训任务单

### 任务一：环境和源码

- [ ] 主机为 Ubuntu x86_64；
- [ ] 剩余空间大于 120 GB；
- [ ] `repo version` 正常；
- [ ] `repo sync -c -j4` 完成；
- [ ] 能解释 `.repo/` 与 Git 子仓库的关系。

### 任务二：第一次构建

- [ ] 选择 Debian Server 配置；
- [ ] 编译 U-Boot；
- [ ] 编译 kernel；
- [ ] 编译 Debian rootfs；
- [ ] 生成 `rockdev/update.img`；
- [ ] 记录镜像大小和 SHA-256。

### 任务三：第一次内核修改

- [ ] 使用 `kernel-config` 修改一个无风险配置；
- [ ] 用 `git diff` 说明修改；
- [ ] 增量编译 kernel；
- [ ] 更新 boot 分区；
- [ ] 在板上验证生效。

### 任务四：第一次设备树调试

- [ ] 找到 BPI-M5Pro 主 DTS；
- [ ] 解释一个节点的 `compatible`、`status` 和 pinctrl；
- [ ] 完成 DTB 构建；
- [ ] 从串口和 `dmesg` 判断驱动是否 probe；
- [ ] 保存修改前后的日志。

## 18. 命令速查

```bash
cd "$HOME/bpi-m5pro"

.repo/repo/repo sync -c -j4
./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig

# 完整编译
./build.sh

# 分步编译
./build.sh uboot
./build.sh kernel
./build.sh debian
./build.sh firmware
./build.sh updateimg

# 调整内核配置
./build.sh kernel-config

# 检查修改和产物
repo status
repo diff
ls -lh rockdev/
sha256sum rockdev/update.img
```

## 19. 本次实操记录

教程整理时已在当前工作区开始实际验证：

```text
主机：Ubuntu 22.04 x86_64
CPU：16 线程
内存：31 GiB
可用磁盘：892 GiB
SDK 初始化：成功
SDK 同步：成功，12 个仓库，约 10 分 15 秒
同步后体积：约 11 GiB
配置选择：RK3576 + Debian 12 Server，成功
首次直接选择 defconfig：失败，原因是 SDK 尚未切换到 RK3576
正确命令：./build.sh rk3576:armsom_sige5_rk3576_debian_server_defconfig
U-Boot 编译：成功，约 27 秒
U-Boot 产物：u-boot/uboot.img，4.0 MiB
U-Boot SHA-256：b6a94d06c863151d6f63e1a108761daff23e6807685e84cfa872ce41f30d3672
首次内核编译：失败，goodix_ts.o 缺少构建规则
内核修复：恢复 goodix_ts-y := goodix.o goodix_fwupload.o
内核增量重编：成功，Linux 6.1.118-rk3576，约 7 分钟
boot 产物：kernel-6.1/extboot.img，128 MiB
boot SHA-256：8ecb23eb3cf5b386d8eb9e18c8cadc5432204014366579b0c4567acc271d5bea
内核 DEB：image 约 18 MiB、headers 约 8.3 MiB、debug 约 59 MiB
Debian rootfs：已验证 debootstrap、QEMU 第二阶段和软件包安装流程
rootfs 下载量：本轮需下载约 287 MiB、安装 483 个软件包
rootfs 本轮结果：镜像源较慢，运行约 22 分钟后人工中止；脚本已卸载临时文件系统
```

U-Boot 和内核已完整实编通过。rootfs 中止不是源码错误；重新执行 `./build.sh debian` 即可从头构建，完成后再执行 `./build.sh firmware` 和 `./build.sh updateimg`。本轮没有伪造 rootfs 或整包成功结果。
