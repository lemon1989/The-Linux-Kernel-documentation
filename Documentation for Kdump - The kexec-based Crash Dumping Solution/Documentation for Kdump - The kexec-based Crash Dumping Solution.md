# [Documentation for Kdump - The kexec-based Crash Dumping Solution](https://www.kernel.org/doc/html/latest/admin-guide/kdump/kdump.html)
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
当前系统内核启动时，它会为转储捕获内核保留一小部分内存。这可确保来自当前系统内核的持续直接内存访问（DMA）不会损坏转储捕获内核。kexec -p 命令会将转储捕获内核加载到此保留的内存中。

On x86 machines, the first [640 KB of physical memory](https://www.xtof.info/blog/?p=985) is needed to boot, regardless of where the kernel loads. Therefore, kexec backs up this region just before rebooting into the dump-capture kernel.\
在 x86 计算机上，加载内核时需要使用前 640 KB 的物理内存。因此，kexec 在重新启动到转储捕获内核之前需要先备份此物理内存区域。

Similarly on PPC64 machines first 32KB of physical memory is needed for booting regardless of where the kernel is loaded and to support 64K page size kexec backs up the first 64KB memory.\
同样地，在 PPC64 计算机上，加载内核时需要使用前 32KB 的物理内存，并且可以支持 64K 页大小的 kexec 来备份第一个 64KB 内存。

For s390x, when kdump is triggered, the crashkernel region is exchanged with the region [0, crashkernel region size] and then the kdump kernel runs in [0, crashkernel region size]. Therefore no relocatable kernel is needed for s390x.\
对于 s390x，当触发 kdump 时，崩溃内核区域与区域 [0，崩溃内核区域大小] 进行交换，然后 kdump 内核在这个区域 {0， 崩溃内核区域大小} 运行。因此，s390x 不需要可重定向的内核。

All of the necessary information about the system kernel’s core image is encoded in the ELF format, and stored in a reserved area of memory before a crash. The physical address of the start of the ELF header is passed to the dump-capture kernel through the elfcorehdr= boot parameter. Optionally the size of the ELF header can also be passed when using the elfcorehdr=[size[KMG]@]offset[KMG] syntax.\
系统内核的核心映像包含的所有必要信息会以 ELF 格式进行编码，并在崩溃前被存储在保留的内存区域。从 ELF 标头开始的物理地址通过 elfcorehdr= boot parameter 的形式传递给转储捕获内核。使用 elfcorehdr=[size[KMG]@]offset[KMG] 语法时，也可以传递 ELF 标头的大小。

With the dump-capture kernel, you can access the memory image through /proc/vmcore. This exports the dump as an ELF-format file that you can write out using file copy commands such as cp or scp. Further, you can use analysis tools such as the GNU Debugger (GDB) and the Crash tool to debug the dump file. This method ensures that the dump pages are correctly ordered.\
使用转储捕获内核，你可以通过 /proc/vmcore 访问内存映像。这会将转储导出为一个 ELF 格式的文件，你可以使用文件复制命令（如 cp 或 scp）将其导出。此外，你可以使用分析工具，例如，GNU 调试器 （GDB） 和 Crash 工具来调试转储文件。此方法可确保转储页有正确的排序。

## Setup and Installation
设置和安装

### Install kexec-tools
安装 kexec-tools

1.Login as the root user.\
以 root 用户登录。

2.Download the kexec-tools user-space package from the following URL:\
从以下 URL 地址下载 kexec-tools 工具的用户空间 package 包：

[http://kernel.org/pub/linux/utils/kernel/kexec/kexec-tools.tar.gz](http://kernel.org/pub/linux/utils/kernel/kexec/kexec-tools.tar.gz)

This is a symlink to the latest version.\
这是最新版本的链接地址。

The latest kexec-tools git tree is available at:\
最新的 kexec-tools 工具的 git tree 的下载地址：

- git://git.kernel.org/pub/scm/utils/kernel/kexec/kexec-tools.git
- [http://www.kernel.org/pub/scm/utils/kernel/kexec/kexec-tools.git](http://www.kernel.org/pub/scm/utils/kernel/kexec/kexec-tools.git)




