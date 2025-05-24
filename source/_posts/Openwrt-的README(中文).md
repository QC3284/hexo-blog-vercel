---
title: OpenWrt 说明文档
date: 2024-08-08 12:41:32
tags:
---

![OpenWrt logo](https://jsd.xcqcoo.top/gh/openwrt/openwrt@main/include/logo.png)  

OpenWrt 项目是一个面向嵌入式设备的 Linux 操作系统。它并非提供单一的静态固件，而是通过完全可写的文件系统与包管理机制，赋予用户自由选择应用和配置的能力。这意味着您可以摆脱厂商预置的限制，通过安装软件包定制设备功能，满足任何应用场景需求。对于开发者，OpenWrt 提供了一个无需构建完整固件的开发框架；对于用户，则意味着能够以厂商未曾设想的方式实现设备的全面定制。  

## 下载  

我们为多种架构提供了预编译的固件镜像，默认包含适用于家用无线路由器的软件包。若需快速找到从厂商原厂固件迁移至 OpenWrt 的可用出厂镜像，请使用 **固件选择器**：  

* [OpenWrt 固件选择器](https://firmware-selector.openwrt.org/)  

如果您的设备已被支持，请点击 **信息** 链接查看安装说明，或参考下方列出的支持资源。  

## 高级下载  

高级用户可能需要特定或额外的软件包（如工具链、SDK 等）。除基础固件下载外，其他需求请访问 Wiki 下载页面：  

* [OpenWrt Wiki 下载页](https://openwrt.org/downloads)  

## 开发  

要自行编译固件，您需要 GNU/Linux、BSD 或 macOS 系统（需区分大小写的文件系统）。由于缺乏大小写敏感的文件系统支持，Cygwin 不被支持。  

### 编译要求  

编译 OpenWrt 需以下工具，不同发行版的软件包名称可能不同。完整列表及发行版特定依赖项请参阅 [构建系统配置文档](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)：  

```  
binutils bzip2 diff find flex gawk gcc-6+ getopt grep install libc-dev libz-dev  
make4.1+ perl python3.7+ rsync subversion unzip which  
```  

### 快速入门  

1. 运行 `./scripts/feeds update -a` 以获取 `feeds.conf` 或 `feeds.conf.default` 中定义的所有最新软件包信息。  
2. 运行 `./scripts/feeds install -a` 将所有软件包的符号链接安装到 `package/feeds/` 目录。  
3. 运行 `make menuconfig` 配置工具链、目标系统及固件包的选项。  
4. 运行 `make` 开始编译固件。此过程将下载所有源码、构建交叉编译工具链，并为目标系统交叉编译 Linux 内核及所选应用程序。  

### 相关代码库  

主仓库通过多个子仓库管理不同类别的软件包。所有软件包均通过 OpenWrt 包管理器 `opkg` 安装。如需开发网页界面或移植软件包，请参考以下仓库：  

* [LuCI 网页界面](https://github.com/openwrt/luci)：通过浏览器控制设备的现代化模块化界面。  
* [OpenWrt 软件包](https://github.com/openwrt/packages)：社区维护的移植软件包仓库。  
* [OpenWrt 路由](https://github.com/openwrt/routing)：专注于（网状）路由的软件包。  
* [OpenWrt 视频](https://github.com/openwrt/video)：专注于显示服务与客户端（Xorg 和 Wayland）的软件包。  

## 支持信息  

支持设备列表请查看 [OpenWrt 硬件数据库](https://openwrt.org/supported_devices)。  

### 文档  

* [快速入门指南](https://openwrt.org/docs/guide-quick-start/start)  
* [用户指南](https://openwrt.org/docs/guide-user/start)  
* [开发者文档](https://openwrt.org/docs/guide-developer/start)  
* [技术参考](https://openwrt.org/docs/techref/start)  

### 支持社区  

* [论坛](https://forum.openwrt.org)：讨论使用、项目、硬件建议。  
* [支持聊天室](https://webchat.oftc.net/#openwrt)：**oftc.net** 的 `#openwrt` 频道。  

### 开发者社区  

* [问题报告](https://bugs.openwrt.org)：提交 OpenWrt 的 Bug。  
* [开发邮件列表](https://lists.openwrt.org/mailman/listinfo/openwrt-devel)：提交代码补丁。  
* [开发聊天室](https://webchat.oftc.net/#openwrt-devel)：**oftc.net** 的 `#openwrt-devel` 频道。  

## 许可证  

OpenWrt 遵循 **GPL-2.0** 协议。  
[file content end]  

---

### 翻译说明
1. By Deepseek
2. **术语一致性**：技术术语（如 `package management` → **包管理**）严格按行业标准翻译，专有名词（如 `LuCI`、`opkg`）保留原名。  
3. **格式保留**：Markdown 语法、代码块、链接均未改动，确保文档结构清晰。  
4. **语句优化**：在保持原意的基础上，调整部分长句结构以提高中文可读性。  
5. **功能验证**：所有外部链接均未修改，确保跳转功能正常。