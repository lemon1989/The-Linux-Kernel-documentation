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
对于 s390x，当触发 kdump 时，崩溃内核区域与区域 [0，崩溃内核区域大小] 进行交换，然后 kdump 内核在这个区域 {0， 崩溃内核区域大小} 运行。因此，s390x 不需要可重定位的内核。

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

There is also a gitweb interface available at [http://www.kernel.org/git/?p=utils/kernel/kexec/kexec-tools.git](http://www.kernel.org/git/?p=utils/kernel/kexec/kexec-tools.git)\
还有一个 gitweb 界面的访问地址 http://www.kernel.org/git/?p=utils/kernel/kexec/kexec-tools.git。

More information about kexec-tools can be found at [http://horms.net/projects/kexec/](http://horms.net/projects/kexec/)\
有关 kexec-tools 工具的更多信息，请查看 http://horms.net/projects/kexec/。

3.Unpack the tarball with the tar command, as follows:\
使用以下命令解压 kexec-tools.tar.gz 包：

tar xvpzf kexec-tools.tar.gz

4.Change to the kexec-tools directory, as follows:\
使用以下命令切换到 kexec-tools 目录：

cd kexec-tools-VERSION

5.Configure the package, as follows:\
配置 kexec-tools-VERSION 工具包：

./configure

6.Compile the package, as follows:\
编译 kexec-tools-VERSION 工具包：

make

7.Install the package, as follows:\
安装 kexec-tools-VERSION 工具包：

make install

### Build the system and dump-capture kernels
构建系统和转储捕获内核

There are two possible methods of using Kdump.\
使用 Kdump 有两种方法。

1.Build a separate custom dump-capture kernel for capturing the kernel core dump.\
构建单独的自定义转储捕获内核，用于捕获内核核心转储。

2.Or use the system kernel binary itself as dump-capture kernel and there is no need to build a separate dump-capture kernel. This is possible only with the architectures which support a relocatable kernel. As of today, i386, x86_64, ppc64, ia64, arm and arm64 architectures support relocatable kernel.\
或者使用系统内核二进制文件本身作为转储捕获内核，无需构建单独的转储捕获内核。这只有在支持可重定位内核的体系结构下才可能实现。截至今天，i386、x86_64、ppc64、ia64、arm 和 arm64 架构支持可重定位内核。

Building a relocatable kernel is advantageous from the point of view that one does not have to build a second kernel for capturing the dump. But at the same time one might want to build a custom dump capture kernel suitable to his needs.\
构建可重定位内核是有利的，因为不必构建第二个内核来捕获转储。但与此同时，人们可能想要构建一个适合其需要的自定义转储捕获内核。

Following are the configuration setting required for system and dump-capture kernels for enabling kdump support.\
以下是系统和转储捕获内核启用 kdump 支持所需的配置设置。

### System kernel config options
系统内核配置选项

1.Enable “kexec system call” in “Processor type and features.”:\
在"Processor type and features."中启用"kexec 系统调用"：

CONFIG_KEXEC=y

2.Enable “sysfs file system support” in “Filesystem” -> “Pseudo filesystems.” This is usually enabled by default:\
在"Filesystem"->"Pseudo filesystems"中启用"sysfs file system support"。默认情况下，这通常启用：

CONFIG_SYSFS=y

Note that “sysfs file system support” might not appear in the “Pseudo filesystems” menu if “Configure standard kernel features (for small systems)” is not enabled in “General Setup.” In this case, check the .config file itself to ensure that sysfs is turned on, as follows:\
请注意，如果在"General Setup."中未启用"Configure standard kernel features (for small systems)"，则"Pseudo filesystems"菜单中可能不会显示"sysfs file system support"。在这种情况下，请检查 .config 文件本身以确保 sysfs 已打开，如下所示：

grep 'CONFIG_SYSFS' .config

3.Enable “Compile the kernel with debug info” in “Kernel hacking.”:\
在"Kernel hacking."中启用"Compile the kernel with debug info"：

CONFIG_DEBUG_INFO=Y

This causes the kernel to be built with debug symbols. The dump analysis tools require a vmlinux with debug symbols in order to read and analyze a dump file.\
这会使内核生成调试符号。转储分析工具需要具有调试符号的 vmlinux 才能读取和分析转储文件。

### Dump-capture kernel config options (Arch Independent)
转储捕获内核配置选项（与架构无关）

1.Enable “kernel crash dumps” support under “Processor type and features”:\
在"Processor type and features"下启用"kernel crash dumps"支持：

CONFIG_CRASH_DUMP=y

2.Enable “/proc/vmcore support” under “Filesystems” -> “Pseudo filesystems”:\
在"Filesystems" -> "Pseudo filesystems"下启用"/proc/vmcore 支持"，：

CONFIG_PROC_VMCORE=y

(CONFIG_PROC_VMCORE is set by default when CONFIG_CRASH_DUMP is selected.)\
CONFIG_PROC_VMCORE 启用时，将默认启用 CONFIG_CRASH_DUMP。

### Dump-capture kernel config options (Arch Dependent, i386 and x86_64)
转储捕获内核配置选项（与架构有关，i386 和x86_64）

1.On i386, enable high memory support under “Processor type and features”:\
在 i386 上，在"Processor type and features"下启用高内存支持：

CONFIG_HIGHMEM64G=y

or:

CONFIG_HIGHMEM4G

2.On i386 and x86_64, disable symmetric multi-processing support under “Processor type and features”:\
在 i386 和 x86_64，禁用"Processor type and features"下的对称多处理支持：

CONFIG_SMP=n

(If CONFIG_SMP=y, then specify maxcpus=1 on the kernel command line when loading the dump-capture kernel, see section “Load the Dump-capture Kernel”.)\
如果 CONFIG_SMP=y，则在加载转储捕获内核时在内核命令行上指定 maxcpus=1，请参阅"加载转储捕获内核"一节。

3.If one wants to build and use a relocatable kernel, Enable “Build a relocatable kernel” support under “Processor type and features”:\
如果要构建和使用可重定位内核，请启用"Processor type and features"下的"Build a relocatable kernel"支持：

CONFIG_RELOCATABLE=y

4.Use a suitable value for “Physical address where the kernel is loaded” (under “Processor type and features”). This only appears when “kernel crash dumps” is enabled. A suitable value depends upon whether kernel is relocatable or not.\
对"Physical address where the kernel is loaded"（在"Processor type and features"下）使用合适的值。这仅在启用"kernel crash dumps"时使用。合适的值取决于内核是否可重定位。

If you are using a relocatable kernel use CONFIG_PHYSICAL_START=0x100000 This will compile the kernel for physical address 1MB, but given the fact kernel is relocatable, it can be run from any physical address hence kexec boot loader will load it in memory region reserved for dump-capture kernel.\
如果使用可重定位捕获内核，启用 CONFIG_PHYSICAL_START=0x100000 这将编译物理地址 1MB 的内核，但事实上需考虑内核是否可重定位捕获，它可以从任何物理地址运行，因此 kexec 通过引导加载程序将加载它(可重定位捕获内核)到为转储捕获内核保留的内存区域。

Otherwise it should be the start of memory region reserved for second kernel using boot parameter “crashkernel=Y@X”. Here X is start of memory region reserved for dump-capture kernel. Generally X is 16MB (0x1000000). So you can set CONFIG_PHYSICAL_START=0x1000000\
否则，它应该是使用引导参数"crashkernel=Y@X"为第二个内核保留的内存区域的开始。此处 X 是为转储捕获内核保留的内存区域的开始。通常 X 为 16MB（0x1000000）。因此，你可以设置CONFIG_PHYSICAL_START=0x1000000

5.Make and install the kernel and its modules. DO NOT add this kernel to the boot loader configuration files.\
制作并安装内核及其模块。不要将此内核添加到引导加载程序配置文件中。

### Dump-capture kernel config options (Arch Dependent, ppc64)
转储捕获内核配置选项（与架构有关，ppc64）

1.Enable “Build a kdump crash kernel” support under “Kernel” options:\
在"Kernel"选项下启用"Build a kdump crash kernel"支持：

CONFIG_CRASH_DUMP=y

2.Enable “Build a relocatable kernel” support:\
启用"Build a relocatable kernel"支持：

CONFIG_RELOCATABLE=y

Make and install the kernel and its modules.\
制作并安装内核及其模块。

### Dump-capture kernel config options (Arch Dependent, ia64)
转储捕获内核配置选项（与架构有关，ia64）

- No specific options are required to create a dump-capture kernel for ia64, other than those specified in the arch independent section above. This means that it is possible to use the system kernel as a dump-capture kernel if desired.\
除了上面的与架构有关的部分中指定的内核外，无需为 ia64 创建转储捕获内核所需的特定选项。这意味着，可以使用系统内核作为转储捕获内核。

The crashkernel region can be automatically placed by the system kernel at run time. This is done by specifying the base address as 0, or omitting it all together:\
崩溃内核区域可以在运行时由系统内核自动放置。这是通过将基地址指定为 0 或同时省略地址来完成的：

crashkernel=256M@0

or:

crashkernel=256M

If the start address is specified, note that the start address of the kernel will be aligned to 64Mb, so if the start address is not then any space below the alignment point will be wasted.
如果指定了开始地址，请注意内核的开始地址将对齐到 64Mb，因此，如果开始地址不是，则对齐点下面的任何空间都将被浪费。

### Dump-capture kernel config options (Arch Dependent, arm)
转储捕获内核配置选项（与架构有关，arm）

- To use a relocatable kernel, Enable “AUTO_ZRELADDR” support under “Boot” options:\
要使用可重定位内核，启用"Boot"选项下的"AUTO_ZRELADDR"支持：

AUTO_ZRELADDR=y

### Dump-capture kernel config options (Arch Dependent, arm64)
转储捕获内核配置选项（与架构有关，arm64）

- Please note that kvm of the dump-capture kernel will not be enabled on non-VHE systems even if it is configured. This is because the CPU will not be reset to EL2 on panic.\
请注意，转储捕获内核的 kvm 不会在 non-VHE 系统上启用，即使它已配置。这是因为 CPU 不会在出错时重置为 EL2。

## Extended crashkernel syntax





