---
title: Ubuntu搭建STM32开发环境
date: 2023-02-16 22:23:19
tags: STM32
---

## 使用软件

- 代码生成：STM32CubeMX
- 编译代码：arm-none-eabi toolchain 和 make
- 调试下载：openocd（兼容多种调试器）
- 编辑代码：vscode（stm32-for-vsocde插件）

## Stm32CubeMX

[官网下载地址](https://www.st.com/en/development-tools/stm32cubemx.html)点击下载，输入姓名邮箱后，下载链接会发到你的邮箱里面。

下载完成后，打开文件夹直接运行（有很多之前的教程说要下载jdk啥的，现在cubemx文件夹自带有jre无需自行下载安装）

``` bash
sudo ./SetupSTM32CubeMX-xx
```

## arm-none-eabi toolchain

apt直接安装方式只会安装gcc，而没有gdb。无法进行调试，而且版本很低。

从[ARM官方下载链接](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)下载安装包（x86_64 Linux hosted cross toolchains 下面的 AArch32 bare-metal target)。最好不要下最新版本，可能会有bug。

解压后，将解压后文件夹的bin目录放入环境变量中

## make

绝大多数linux系统自带。如若没有，按照如下方式安装

``` bash
sudo ./SetupSTM32CubeMX-xx
```

## openocd

apt直接安装的版本太老，使用cortex-debug调试时可能会有bug。

从[GitHub](https://github.com/xpack-dev-tools/openocd-xpack/releases)下载linux-x64版本（例如：xpack-openocd-0.12.0-1-linux-x64.tar.gz ）
解压后，将解压后文件夹的bin目录放入环境变量中

## vscode

如若没有vscode，请参照[微软官方教程](https://code.visualstudio.com/docs/setup/linux)安装

下载C/c++插件和stm32-for-vscode插件
打开stm32-for-vscode插件，配置相应的路径（如果加入了环境变量，应该会自动配置）
![](https://image-1305582579.cos.ap-chengdu.myqcloud.com/screenshot.png)
接上自己的调试器后，在stm32-for-vscode插件中测试编译、调试功能是否正常（点击调试按钮后，插件就会自动生成c_cpp_properties.json，vscode就不会代码报错了）
![](https://image-1305582579.cos.ap-chengdu.myqcloud.com/screenshot_1.png)