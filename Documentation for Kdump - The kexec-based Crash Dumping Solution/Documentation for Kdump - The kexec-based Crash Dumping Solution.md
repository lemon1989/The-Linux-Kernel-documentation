# Documentation for Kdump - The kexec-based Crash Dumping Solution
Kdump 的文档 - 一种基于 kexec 的崩溃转储解决方案

This document includes overview, setup and installation, and analysis information.\
本文档包括概述、设置和安装以及分析信息共四部分。

## Overview
概述

Kdump uses kexec to quickly boot to a dump-capture kernel whenever a dump of the system kernel’s memory needs to be taken (for example, when the system panics). The system kernel’s memory image is preserved across the reboot and is accessible to the dump-capture kernel.\
每当需要转储当前系统内核的内存时(例如，当系统出现错误时)，Kdump 可以使用 kexec 快速启动到转储捕获内核。当前系统内核的内存映像在重新启动期间被保留，并且可由转储捕获内核访问。

You can use common commands, such as cp and scp, to copy the memory image to a dump file on the local disk, or across the network to a remote system.\
你可以使用常见命令，例如，cp 和 scp 将系统出错的内存映像保存为转储文件并拷贝到本地磁盘，或通过网络拷贝到远程系统。

Kdump and kexec are currently supported on the x86, x86_64, ppc64, ia64, s390x, arm and arm64 architectures.\
Kdump 和 kexec 目前支持 x86、x86_64、ppc64、ia64、s390x、arm 和 arm64 架构。

When the system kernel boots, it reserves a small section of memory for the dump-capture kernel. This ensures that ongoing Direct Memory Access (DMA) from the system kernel does not corrupt the dump-capture kernel. The kexec -p command loads the dump-capture kernel into this reserved memory.\
当前系统内核启动时，它会为转储捕获内核保留一小部分内存。这可确保来自当前系统内核的持续直接内存访问（DMA）不会损坏转储捕获内核。kexec -p 命令将转储捕获内核加载到此保留内存中。

