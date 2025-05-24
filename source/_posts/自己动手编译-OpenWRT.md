---
title: 自己动手编译 OpenWRT
date: 2024-08-08 16:57:11
tags:
---

文章来源：https://lgiki.net/post/compile-openwrt/

## Intro

学校最近升级了校园网设备，终于从年久失修的龟速 100Mbps 升级到了 1Gbps，尽管还是使用锐捷的 ePortal 网页认证，还能通过简单的 Shell 脚本在路由器上自动认证，但新设备却限制了每个用户最多只能同时有两台设备在线，使用路由器会被检测出使用多设备，并被加入黑名单，被断网。


都 2023 年了，随便手机、电脑再加个平板就仨设备了，更不用说可能还有 Kindle、Switch 等其他设备，只能同时两台设备在线怎么可能满足学生正常使用校园网的需求。更何况，计算机专业的学生可能还会使用虚拟机等软件，但虚拟机类的软件也可能被检测为是多设备同时在线，所以你让计算机专业的学生拿啥学习呢，内存条含嘴里脑补画面吗？


限制校园网的同时在线设备数量无非就两个出发点：（1）如果不限制，一个宿舍只需要开通一个宽带账号，所有人都可以上网了，运营商的收入就减少了；（2）如果学生出现一些非法的上网行为，可以快速准确地定位到具体学生。但我觉得这都不是学校限制学生只能使用两个设备的理由，哪怕提升到 5 个设备我觉得也合理一些。

对于限制校园网设备数量的学校，我只能说 Shame on you！

对于限制校园网设备数量的学校，我只能说 Shame on you！

对于限制校园网设备数量的学校，我只能说 Shame on you！

那就…只能另辟蹊径，想其他办法了。

还好，限制同时在线设备数量这件事早已有其他学校实施过了，也已经有人提出了一些对策。
例如，在 https://blog.sunbk201.site/posts/crack-campus-network 这篇博客里就详细列举了校园网检测在线设备可能采用的手段以及相应的应对措施，我们只需要照着做就行了。

其实主要就是为 OpenWRT 编译 [rkp-ipid](https://github.com/CHN-beta/rkp-ipid) 和 [UA2F](https://github.com/Zxilly/UA2F) 这两个包，而这两个包需要在编译内核的时候开启一些选项，所以需要重新编译 OpenWRT 固件，也因此写下这篇文章记录一下编译 OpenWRT 的过程。

## 安装编译环境

要编译 Linux 当然是使用 Linux 啦！

如果是 Debian 系的发行版（例如 Ubuntu），可以使用以下指令来安装编译所需的依赖：
```sh
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 aria2 asciidoc autoconf automake autopoint binutils bison build-essential \
bzip2 ccache cmake cpio curl device-tree-compiler fastjar flex gawk gettext gcc-multilib g++-multilib \
git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev \
libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libreadline-dev libssl-dev libtool lrzsz \
mkisofs msmtp nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pip libpython3-dev qemu-utils \
rsync scons squashfs-tools subversion swig texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```
对于 Arch 系的发行版，直接安装 AUR 中的 openwrt-devel 即可：
https://aur.archlinux.org/packages/openwrt-devel

## 获取 OpenWRT 源码

如果你的设备是 OpenWRT 官方支持的设备，并且你想编译 OpenWRT 官方源码，则直接：
```sh
git clone https://github.com/openwrt/openwrt.git
```
即可。

但现在有一些国产的路由器 OpenWRT 官方并不支持，可以看看有没有第三方的 OpenWRT 源码支持了你手上的路由器，下载相应的源代码。
可以通过`cat /etc/openwrt_release`命令来查看设备的 Arch、Target 信息：
```sh
root@QWRT:~# cat /etc/openwrt_release
DISTRIB_ID='OpenWrt'
DISTRIB_RELEASE='19.07-SNAPSHOT'
DISTRIB_TARGET='ipq60xx/generic'
DISTRIB_ARCH='aarch64_cortex-a53+crypto'
DISTRIB_TAINTS='no-all busybox override'
DISTRIB_REVISION='R23.1.20'
DISTRIB_DESCRIPTION='QWRT '
```
获取了 OpenWRT 源代码之后需要更新 Feeds：
```sh
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

## 添加想要编译的第三方包 (可选)

因为我需要编译 rkp-ipid 和 UA2F，因此我还需要将这些包的源码下载下来：
```sh
git clone https://github.com/CHN-beta/rkp-ipid package/rkp-ipid
git clone https://github.com/Zxilly/UA2F package/UA2F
```

## 配置编译目标

一切准备就绪，接下来就是设定编译目标了，执行以下命令：
```sh
make menuconfig
```
然后会出现一个命令行 UI，首先需要重点设定好`Target System`,`Subtarget`,`Target Profile`，将这几个项目设定为你的路由器。 然后就可以选择你需要编译的包啦，使用空格可以选择要编译的包，设定为 <*> 的包将会内置到固件里，刷好就能直接用，设定为 <M> 的包将会编译成 ipk 文件，需要上传到路由器，然后使用`opkg install package_name.ipk`进行安装。

设定好了之后，将配置文件保存为`.config`即可，如果有一些包需要对内核做一些自定义，也可以直接使用文本编辑器编辑保存下来的`.config`文件，例如`UA2F`就需要添加`CONFIG_NETFILTER_NETLINK_GLUE_CT=y`。

## 开始编译

一切准备完毕，接下来开始编译！
```sh
make download -j8
make V=s -j8
```
其中`V=s`表示输出`stdout`和`stderr`，方便在编译失败的时候根据输出信息确定是哪里的错误，`-jn`的`n`表示使用`n`个线程同时编译，对于核心数较多的机器可以将该数值设置得大一些，加快编译过程。

编译完成之后，在`./bin/targets`下就可以找到固件以及相应的`ipk`包了。

## UA2F 的注意事项

要使 UA2F 正常工作需要关闭 “网络”——“Turbo ACC 网络加速设置” 下的所有选项，否则 UA2F 无法正常工作，具体表现为：重启路由器防火墙之后，第一次 HTTP 请求的 User-Agent 成功修改，但后续 HTTP 请求的 User-Agent 没有被修改。