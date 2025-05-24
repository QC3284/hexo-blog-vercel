---
title: OpenWrt：从源码到固件，编译自己的OpenWrt系统
date: 2024-08-08 12:26:08
tags:
---


本文说明如何一步一步地从源码编译出自己的OpenWrt镜像
文章来源：知乎@小耸

# 准备工作
准备一个GNU/Linux, BSD 或 MacOSX 操作系统。并且，在环境中准备好以下官方教程中要求的工具：
https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem

我的环境是Ubuntu22.04，执行：
```sh
sudo apt update 
sudo apt install build-essential clang flex bison g++ gawk gcc-multilib g++-multilib gettext git libncurses-dev libssl-dev python3-distutils rsync unzip zlib1g-dev file wget
```

# 编译系统
## 下载源码
下载代码：
```sh
git clone https://github.com/openwrt/openwrt.git
cd openwrt
git pull
```
切换分支，一般选择最新的稳定版本：
```sh
git branch -a
git tag
git checkout v23.05.4
```
## 下载软件包
运行`./scripts/feeds update -a`命令，下载或更新在`feeds.conf/feeds.conf.default`中定义的所有最新软件包。

！！注意！！
受限于国内的访问国际互联网的环境中，feeds update这一步特别容易失败。可以通过以下几条方式来提高成功率：
`git config --global http.postBuffer 524288000` `git config --global http.lowSpeedLimit 1000` `git config --global http.lowSpeedTime 600 `以上配置的含义为：配置git缓冲区为500M，配置git访问超时的条件为：速率小于1KB/s，且持续600秒
（有条件的可以开代理）

运行`./scripts/feeds install -a`命令使安装上述软件包在后续的`make menuconfig`中生效。

# 配置编译选项
## 使用已有固件的编译配置
网络上已编译出的固件通常都会把编译配置一并提供（config.buildinfo或config.seed），可以直接使用。
我的目标机是一台小米WR30U，使用mtk的filogic芯片方案，从OpenWrt官网找到对应的编译配置并下载，置于OpenWrt工程根目录下的`.config`文件中：
```sh
wget https://downloads.openwrt.org/releases/23.05.4/targets/mediatek/filogic/config.buildinfo -O .config
```
但这份配置中包含了filogic芯片方案的所有设备的配置，还需进行裁剪和修改。

## menuconfig
在以上配置的基础上，运行`make menuconfig`命令来完成进一步的自定义配置。完成后配置会更新至`.config`文件中。
![v2-2_r](https://pic1.zhimg.com/v2-2be9869a274b1a4728fce7dbe1ca1770_r.jpg)

在Target Profile中，仅保留Xiaomi WR30U的设备支持（我的设备是Xiaomi Mi Router WR30U (112M UBI with NMBM-Enabled layout)），把其它设备的支持删除。

## 编译
使用`make download`预先下载编译过程中需要的代码和依赖项等资源。

. . . . . . . 经过漫长的等待下载完成。到此为止，我们已做好了所有的编译准备。
正式开始编译吧，运行`make`命令来构建固件。该命令将下载所有源代码和依赖项（即使之前已经`make download`，也还有其它包需要下载），构建交叉编译工具链，然后为目标系统交叉编译出OpenWrt内核和应用程序。
```sh
make -j4 V=s
```
## 编译选项说明
`-jN`: make命令可以加上-j参数用于指定使用多少cpu核编译，可以加速编译过程。例如：`make download -j4，make -j5`

`V=s`：make命令可以加上V=s可以输出更多的编译错误信息。

如果顺利的话，这里make完就可以编译出镜像了。但是实际不太可能完全顺利。

## 编译输出
. . . . . . . 经过漫长的编译过程（我的环境中编了5个小时），编译结果存放于`openwrt/bin/targets/`目录下。几类编译产出的镜像说明如下：
1.factory：用于替换厂商的原厂固件，兼容原厂的安装包格式。通常使用原厂的web GUI进行升级或在Uboot中刷入。
2.sysupgrade：用于升级替换已有的OpenWrt版本，这是最常用的镜像。
3.initramfs-kernel：用于开发或特殊情况下的一次性引导，作为安装常规sysupgrade版本的过渡步骤。由于initramfs版本完全运行在RAM中，不会在闪存中存储任何设置，因此不适合用于操作性使用。
