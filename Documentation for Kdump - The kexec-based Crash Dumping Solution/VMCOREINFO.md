# [VMCOREINFO](https://www.kernel.org/doc/html/latest/admin-guide/kdump/vmcoreinfo.html)

## What is it?
这是什么？

VMCOREINFO is a special ELF note section. It contains various information from the kernel like structure size, page size, symbol values, field offsets, etc. These data are packed into an ELF note section and used by user-space tools like crash and makedumpfile to analyze a kernel’s memory layout.\
VMCOREINFO 是一个特殊的 ELF 注释部分。它包含内核的各种信息，如结构大小、页面大小、符号值、字段偏移等。这些数据被打包到 ELF 注释部分中，并使用用户空间工具 crash and makedumpfile 来分析内核的内存布局。

## Common variables
通用变量

### init_uts_ns.name.release

The version of the Linux kernel. Used to find the corresponding source code from which the kernel has been built. For example, crash uses it to find the corresponding vmlinux in order to process vmcore.\
Linux 内核的版本。用于查找从中生成内核的相应源代码。例如，崩溃使用它来查找相应的 vmlinux 来处理 vmcore。

### PAGE_SIZE

The size of a page. It is the smallest unit of data used by the memory management facilities. It is usually 4096 bytes of size and a page is aligned on 4096 bytes. Used for computing page addresses.\
页面的大小。它是内存管理设施使用的最小数据单位。它的大小通常是 4096 字节，并且页面在 4096 字节上对齐。用于计算页面地址。

### init_uts_ns

The UTS namespace which is used to isolate two specific elements of the system that relate to the uname(2) system call. It is named after the data structure used to store information returned by the uname(2) system call.\
用于隔离与无名（2） 系统调用相关的系统两个特定元素的 UTS 命名空间。它以用于存储由无名（2） 系统调用返回的信息的数据结构命名。

User-space tools can get the kernel name, host name, kernel release number, kernel version, architecture name and OS type from it.\
用户空间工具可以从中获取内核名称、主机名、内核版本号、内核版本、体系结构名称和操作系统类型。

### node_online_map

An array node_states[N_ONLINE] which represents the set of online nodes in a system, one bit position per node number. Used to keep track of which nodes are in the system and online.\
数组node_states[N_ONLINE]，它表示系统中的联机节点集，每个节点编号一个位位置。用于跟踪系统中和联机的节点。

## x86_64
x86_64
