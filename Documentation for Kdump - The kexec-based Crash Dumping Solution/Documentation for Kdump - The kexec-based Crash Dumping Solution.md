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

If the start address is specified, note that the start address of the kernel will be aligned to 64Mb, so if the start address is not then any space below the alignment point will be wasted.\
如果指定了开始地址，请注意内核的开始地址将对齐到 64Mb，因此，如果开始地址不是，则对齐点下面的任何空间都将被浪费。

### Dump-capture kernel config options (Arch Dependent, arm)
转储捕获内核配置选项（与架构有关，arm）

- To use a relocatable kernel, Enable “AUTO_ZRELADDR” support under “Boot” options:\
要使用可重定位内核，启用"Boot"选项下的"AUTO_ZRELADDR"支持：

    AUTO_ZRELADDR=y

### Dump-capture kernel config options (Arch Dependent, arm64)
转储捕获内核配置选项（与架构有关，arm64）

- Please note that kvm of the dump-capture kernel will not be enabled on non-VHE systems even if it is configured. This is because the CPU will not be reset to EL2 on panic.\
请注意，转储捕获内核的 kvm 不会在 non-VHE 系统上起作用，即使它被配置。这是因为 CPU 不会在出错时重置为 EL2。

## Extended crashkernel syntax
扩展的 crashkernel 语法

While the “crashkernel=size[@offset]” syntax is sufficient for most configurations, sometimes it’s handy to have the reserved memory dependent on the value of System RAM – that’s mostly for distributors that pre-setup the kernel command line to avoid a unbootable system after some memory has been removed from the machine.\
虽然"crashkernel=size=@offset"语法对于大多数配置来说已经足够了，但有时保留内存依赖于 System RAM 的值是很方便的，这主要是对于预先设置内核命令行以避免从计算机中删除某些内存后无法启动的系统分发服务器。

The syntax is:\
语法：

    crashkernel=<range1>:<size1>[,<range2>:<size2>,...][@offset]
    range=start-[end]

For example:\
例如：

    crashkernel=512M-2G:64M,2G-:128M

This would mean:\
这意味着：

1.if the RAM is smaller than 512M, then don’t reserve anything (this is the “rescue” case)\
如果 RAM 小于 512M，则不要保留任何内容（这是"rescue"案例）

2.if the RAM size is between 512M and 2G (exclusive), then reserve 64M\
如果 RAM 大小介于 512M 和 2G（独有的）之间，则保留 64M

3.if the RAM size is larger than 2G, then reserve 128M\
如果 RAM 大小大于 2G，则保留 128M

## Boot into System Kernel
引导到系统内核

1.Update the boot loader (such as grub, yaboot, or lilo) configuration files as necessary.\
需要更新引导加载程序（例如 grub、yaboot 或 lilo）配置文件。

2.Boot the system kernel with the boot parameter “crashkernel=Y@X”, where Y specifies how much memory to reserve for the dump-capture kernel and X specifies the beginning of this reserved memory. For example, “crashkernel=64M@16M” tells the system kernel to reserve 64 MB of memory starting at physical address 0x01000000 (16MB) for the dump-capture kernel.\
使用引导参数"crashkernel=Y@X"启动系统内核，其中 Y 指定为转储捕获内核保留多少内存，X 指定此保留内存的开头。例如，"crashkernel=64M@16M"告诉系统内核从转储捕获内核的物理地址 0x01000000 （16MB） 开始保留 64 MB 的内存。

On x86 and x86_64, use “crashkernel=64M@16M”.

On ppc64, use “crashkernel=128M@32M”.

On ia64, 256M@256M is a generous value that typically works. The region may be automatically placed on ia64, see the dump-capture kernel config option notes above. If use sparse memory, the size should be rounded to GRANULE boundaries.\
在 ia64 上，256M@256M是一个有效的通用值。该区域可能自动放置在 ia64 上，请参阅上面的转储捕获内核配置选项注释。如果使用稀疏内存，则大小应四舍五入到 GRANULE 边界。

On s390x, typically use “crashkernel=xxM”. The value of xx is dependent on the memory consumption of the kdump system. In general this is not dependent on the memory size of the production system.\
在 s390x 上，通常使用"crashkernel=xxM"。xx 的值取决于 kdump 系统的内存消耗。通常，这不依赖于当前系统的内存大小。

On arm, the use of “crashkernel=Y@X” is no longer necessary; the kernel will automatically locate the crash kernel image within the first 512MB of RAM if X is not given.\
在 arm 上，不再需要使用"Y@X";如果未提供 X，内核将自动定位前 512MB RAM 中的 crash kernel 映像。

On arm64, use “crashkernel=Y[@X]”. Note that the start address of the kernel, X if explicitly specified, must be aligned to 2MiB (0x200000).\
在 arm64 上，使用"crashkernel=Y[@X]"。请注意，内核的启动地址 X（如果显式指定）必须与 2MiB （0x200000） 对齐。

## Load the Dump-capture Kernel
加载转储捕获内核

After booting to the system kernel, dump-capture kernel needs to be loaded.\
引导到系统内核后，需要加载转储捕获内核。

Based on the architecture and type of image (relocatable or not), one can choose to load the uncompressed vmlinux or compressed bzImage/vmlinuz of dump-capture kernel. Following is the summary.\
根据映像的体系结构和类型（可重定位与否），可以选择加载转储捕获内核的未压缩的 vmlinux 或压缩的 bzImage/vmlinuz。以下是摘要。

For i386 and x86_64:\
对于 i386 和 x86_64：

- Use vmlinux if kernel is not relocatable.\
如果内核不可重定位，请使用 vmlinux。

- Use bzImage/vmlinuz if kernel is relocatable.\
如果内核可重定位，请使用 bzImage/vmlinuz。

For ppc64:\
对于 ppc64:

- Use vmlinux

For ia64:\
对于 ia64:

- Use vmlinux or vmlinuz.gz

For s390x:\
对于 s390x:

- Use image or bzImage

For arm:\
对于 arm:

- Use zImage

For arm64:\
对于 arm64:

- Use vmlinux or Image

If you are using an uncompressed vmlinux image then use following command to load dump-capture kernel:\
如果使用未压缩的 vmlinux 映像，请使用以下命令加载转储捕获内核：

    kexec -p <dump-capture-kernel-vmlinux-image> \
    --initrd=<initrd-for-dump-capture-kernel> --args-linux \
    --append="root=<root-dev> <arch-specific-options>"

If you are using a compressed bzImage/vmlinuz, then use following command to load dump-capture kernel:\
如果使用压缩 bzImage/vmlinuz，请使用以下命令加载转储捕获内核：

    kexec -p <dump-capture-kernel-bzImage> \
    --initrd=<initrd-for-dump-capture-kernel> \
    --append="root=<root-dev> <arch-specific-options>"

If you are using a compressed zImage, then use following command to load dump-capture kernel:\
如果使用压缩 zImage，请使用以下命令加载转储捕获内核：

    kexec --type zImage -p <dump-capture-kernel-bzImage> \
    --initrd=<initrd-for-dump-capture-kernel> \
    --dtb=<dtb-for-dump-capture-kernel> \
    --append="root=<root-dev> <arch-specific-options>"

If you are using an uncompressed Image, then use following command to load dump-capture kernel:\
如果使用未压缩映像，请使用以下命令加载转储捕获内核：

    kexec -p <dump-capture-kernel-Image> \
    --initrd=<initrd-for-dump-capture-kernel> \
    --append="root=<root-dev> <arch-specific-options>"
  
Please note, that –args-linux does not need to be specified for ia64. It is planned to make this a no-op on that architecture, but for now it should be omitted
请注意，不需要为 ia64 指定 args-linux。原本计划在该体系结构上加 no-op，但现在应省略

Following are the arch specific command line options to be used while loading dump-capture kernel.\
以下是在加载转储捕获内核时要使用的架构特定命令行选项。

For i386, x86_64 and ia64:

    “1 irqpoll maxcpus=1 reset_devices”

For ppc64:

    “1 maxcpus=1 noirqdistrib reset_devices”

For s390x:

    “1 maxcpus=1 cgroup_disable=memory”

For arm:

    “1 maxcpus=1 reset_devices”

For arm64:

    “1 maxcpus=1 reset_devices”

Notes on loading the dump-capture kernel:\
加载转储捕获内核的注意事项：

- By default, the ELF headers are stored in ELF64 format to support systems with more than 4GB memory. On i386, kexec automatically checks if the physical RAM size exceeds the 4 GB limit and if not, uses ELF32. So, on non-PAE systems, ELF32 is always used.\
默认情况下，ELF 标头以 ELF64 格式存储，以支持内存超过 4GB 的系统。在 i386 上，kexec 会自动检查物理 RAM 大小是否超过 4 GB 限制，如果没有，则使用 ELF32。因此，在 non-PAE 系统上，始终使用 ELF32。

The –elf32-core-headers option can be used to force the generation of ELF32 headers. This is necessary because GDB currently cannot open vmcore files with ELF64 headers on 32-bit systems.\
配置 elf32-core-headers 选项后可强制生成 ELF32 标头。因为目前 GDB 在32位系统上无法打开带有 ELF64 标头的 vmcore 文件。

- The “irqpoll” boot parameter reduces driver initialization failures due to shared interrupts in the dump-capture kernel.\
"irqpoll"引导参数减少了由于转储捕获内核中的共享中断导致的驱动程序初始化失败。

- You must specify <root-dev> in the format corresponding to the root device name in the output of mount command.\
你必须在 mount 挂载命令中，按照<root-dev>格式指定根设备名称。
    
- Boot parameter “1” boots the dump-capture kernel into single-user mode without networking. If you want networking, use “3”.\
如果未联网时，引导参数"1"将转储捕获内核引导到单用户模式。如果要联网，请使用"3"。

- We generally don’t have to bring up a SMP kernel just to capture the dump. Hence generally it is useful either to build a UP dump-capture kernel or specify maxcpus=1 option while loading dump-capture kernel. Note, though maxcpus always works, you had better replace it with nr_cpus to save memory if supported by the current ARCH, such as x86.\
我们通常不必使用 SMP 内核来捕获输出。因此，一般构建一个可运行转储捕获内核或在加载转储捕获内核时指定 maxcpus=1 选项非常有用。请注意，虽然 maxcpus 始终有效，你最好使用 nr_cpus 替换 maxcpus=1 来保存内存，例如，x86 架构支持 nr_cpus。

- You should enable multi-cpu support in dump-capture kernel if you intend to use multi-thread programs with it, such as parallel dump feature of makedumpfile. Otherwise, the multi-thread program may have a great performance degradation. To enable multi-cpu support, you should bring up an SMP dump-capture kernel and specify maxcpus/nr_cpus, disable_cpu_apicid=[X] options while loading it.\
如果你打算使用多线程程序（如 makedumpfile 的并行转储功能），则应在转储捕获内核中启用 multi-cpu 支持。否则，多线程程序可能会严重性能下降。若要启用 multi-cpu 支持，应在加载时启动 SMP 转储捕获内核并指定 maxcpus/nr_cpus，disable_cpu_apicid[X] 选项。

- For s390x there are two kdump modes: If a ELF header is specified with the elfcorehdr= kernel parameter, it is used by the kdump kernel as it is done on all other architectures. If no elfcorehdr= kernel parameter is specified, the s390x kdump kernel dynamically creates the header. The second mode has the advantage that for CPU and memory hotplug, kdump has not to be reloaded with kexec_load().\
对于 s390x 有两种 kdump 模式：如果使用 elfcorehdr= kernel parameter 指定了 ELF 标头，则 kdump 内核会使用它，就像它在所有其他体系结构上一样。如果未指定 elfcorehdr= kernel parameter，则 s390x kdump 内核将动态创建标头。第二种模式的优点是，对于 CPU 和内存热插拔，kdump 不需要用 kexec_load（） 重新加载。

- For s390x systems with many attached devices the “cio_ignore” kernel parameter should be used for the kdump kernel in order to prevent allocation of kernel memory for devices that are not relevant for kdump. The same applies to systems that use SCSI/FCP devices. In that case the “allow_lun_scan” zfcp module parameter should be set to zero before setting FCP devices online.\
对于具有许多连接设备的 s390x 系统，应对 kdump 内核使用"cio_ignore"内核参数，以防止为与 kdump 不相关的设备分配内核内存。这同样适用于使用 SCSI/FCP 设备的系统。在这种情况下，设置 FCP 设备之前，zfcp 模块 allow_lun_scan 参数应设置为零。

## Kernel Panic
内核报错

After successfully loading the dump-capture kernel as previously described, the system will reboot into the dump-capture kernel if a system crash is triggered. Trigger points are located in panic(), die(), die_nmi() and in the sysrq handler (ALT-SysRq-c).\
成功加载转储捕获内核后，如前所述，如果触发系统崩溃，系统将重新启动到转储捕获内核。触发点位于 panic()、die()、die_nmi() 和系统处理程序(ALT-SysRq-c)。

The following conditions will execute a crash trigger point:\
以下条件将执行崩溃触发器点：

If a hard lockup is detected and “NMI watchdog” is configured, the system will boot into the dump-capture kernel ( die_nmi() ).\
如果检测到硬锁定并配置了"NMI watchdog"，系统将启动到转储捕获内核 ( die_nmi() )。

If die() is called, and it happens to be a thread with pid 0 or 1, or die() is called inside interrupt context or die() is called and panic_on_oops is set, the system will boot into the dump-capture kernel.\
如果调用 die()，并且它恰好是具有 pid 0 或 1 的线程，或者 die() 在中断上下文中调用或 die() 被调用并设置 panic_on_oops，则系统将引导到转储捕获内核。

On powerpc systems when a soft-reset is generated, die() is called by all cpus and the system will boot into the dump-capture kernel.\
在 powerpc 系统生成软重置时，所有 cpu 调用了 die()，系统将引导到转储捕获内核。

For testing purposes, you can trigger a crash by using “ALT-SysRq-c”, “echo c > /proc/sysrq-trigger” or write a module to force the panic.\
出于测试目的，你可以使用"ALT-SysRq-c"，"echo c > /proc/sysrq-trigger"触发崩溃，或编写模块来强制出错。

## Write Out the Dump File
写入转储文件

After the dump-capture kernel is booted, write out the dump file with the following command:\
启动转储捕获内核后，使用以下命令写入转储文件：

    cp /proc/vmcore <dump-file>
    
## Analysis 
分析

Before analyzing the dump image, you should reboot into a stable kernel.\
在分析转储映像之前，应重新启动到稳定的内核中。

You can do limited analysis using GDB on the dump file copied out of /proc/vmcore. Use the debug vmlinux built with -g and run the following command:\
你可以使用 GDB 对从 /proc/vmcore 复制的转储文件进行有限的分析。使用 -g 构建的调试 vmlinux 并运行以下命令：

    gdb vmlinux <dump-file>

Stack trace for the task on processor 0, register display, and memory display work fine.\
处理器 0 上任务的堆栈跟踪、寄存器显示和内存显示工作正常。

Note: GDB cannot analyze core files generated in ELF64 format for x86. On systems with a maximum of 4GB of memory, you can generate ELF32-format headers using the –elf32-core-headers kernel option on the dump kernel.\
注意：GDB 无法分析以 ELF64 格式生成的 x86 核心文件。在内存最多为 4GB 的系统上，你可以使用转储内核上的 elf32-core-headers 配置项生成 ELF32 格式标头。

You can also use the Crash utility to analyze dump files in Kdump format. Crash is available at the following URL:\
还可以使用 Crash 实用程序以 Kdump 格式分析转储文件。Crash 可在以下 URL 中使用：

[https://github.com/crash-utility/crash](https://github.com/crash-utility/crash)

Crash document can be found at:\
Crash 文档在以下链接：

[https://crash-utility.github.io/](https://crash-utility.github.io/)

## Trigger Kdump on WARN()







