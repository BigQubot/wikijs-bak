---
title: Getting Started with BPI-M1
description: 
published: true
date: 2023-10-17T16:49:07.169Z
tags: 
editor: markdown
dateCreated: 2023-10-17T16:43:11.880Z
---

# Getting Started with BPI-M1

## Introduction

### BPI-M1

> The Banana Pi M1 is our first product in our goal of creating an open source devlopment board community. With a Banana Pi, we want you to explore and experience the world of DIY projects and portable computing. We welcome all companies, DIYers, and tech loving people within our community! Together, we can make a difference, we can discover our passions, inspire others, and build a practical project

Get more information:[Banana_Pi_BPI-M1](/en/Banana_Pi_BPI-M1)

### Key Features

- Dual-core 1.0GHz CPU
- 1 GB DDR3 memeory
- Mali-400 MP2 with Open GL ES 2.0/1.1

## Development For Armbian

### Basic requirements

x86_64 or aarch64 machine with at least 2GB of memory and ~35GB of disk space for a virtual machine, container or bare metal installation Ubuntu Jammy 22.04.x amd64 or aarch64 for native building or any Docker capable amd64 / aarch64 Linux for containerised

```bash
 apt-get -y install git
 git clone https://github.com/armbian/build
 cd build
 ./compile.sh
```

### Latest images
[https://www.armbian.com/bananapi/](https://www.armbian.com/bananapi/)

## Development For Android

### Load your first image on M1

1. You could download latest image from our forum.
 	Ex:
2. Put your TF card into a TF-USB adapter, and then plug adapter in your Windows PC usb interface.
3. Prepare your image, and download image burning tools PhoenixCard.exe.
4. Use "PhoenixCard.exe" to burn android image to TF card.
![m3_android_burning.png](/m3_android_burning.png)
> Download PhoenixCard: https://pan.baidu.com/s/1-fjvPqtG_zewVzqnXf1AHw?pwd=eid9


{.is-info}
