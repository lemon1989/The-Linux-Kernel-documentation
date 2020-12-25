# [VMCOREINFO](https://www.kernel.org/doc/html/latest/admin-guide/kdump/vmcoreinfo.html)

## What is it?
VMCOREINFO 什么？

VMCOREINFO is a special ELF note section. It contains various information from the kernel like structure size, page size, symbol values, field offsets, etc. These data are packed into an ELF note section and used by user-space tools like crash and makedumpfile to analyze a kernel’s memory layout.\
VMCOREINFO 是一个特殊的 ELF 注释部分。它包含内核的各种信息，如结构大小、页面大小、符号值、字段偏移等。这些数据被打包到 ELF 注释部分中，同时可使用用户空间工具 crash and makedumpfile 来分析内核的内存布局。

## Common variables
通用变量

### init_uts_ns.name.release

The version of the Linux kernel. Used to find the corresponding source code from which the kernel has been built. For example, crash uses it to find the corresponding vmlinux in order to process vmcore.\
Linux 内核的版本。用于查找生成内核的相应源代码。例如，crash 使用它来查找相应的 vmlinux 以此来处理 vmcore。

### PAGE_SIZE

The size of a page. It is the smallest unit of data used by the memory management facilities. It is usually 4096 bytes of size and a page is aligned on 4096 bytes. Used for computing page addresses.\
页面的大小。它是内存管理系统使用的最小数据单位。它的大小通常是 4096 字节，并且页面在 4096 字节上对齐。用于计算页面地址。

### init_uts_ns

The UTS namespace which is used to isolate two specific elements of the system that relate to the uname(2) system call. It is named after the data structure used to store information returned by the uname(2) system call.\
UTS 命名空间用于隔离 uname(2) 系统调用，init_uts_ns 和 uname(2) 属于两个系统特定元素。init_uts_ns 是以用于存储由 uname(2) 系统调用返回的信息的数据结构命名。

User-space tools can get the kernel name, host name, kernel release number, kernel version, architecture name and OS type from it.\
用户空间工具可以从中获取内核名称、主机名、内核版本号、内核版本、体系结构名称和操作系统类型。

### node_online_map

An array node_states[N_ONLINE] which represents the set of online nodes in a system, one bit position per node number. Used to keep track of which nodes are in the system and online.\
数组 node_states[N_ONLINE]，它表示系统中的联机节点集，每个节点编号占用一个位定位。用于跟踪系统和联机的节点。

### swapper_pg_dir

The global page directory pointer of the kernel. Used to translate virtual to physical addresses.\
内核的全局页面目录指针。用于将虚拟地址转换为物理地址。

### _stext

Defines the beginning of the text section. In general, _stext indicates the kernel start address. Used to convert a virtual address from the direct kernel map to a physical address.\
定义代码部分的开头。通常， _stext 指示内核开始地址。用于将虚拟地址从直接内核映射转换为物理地址。

### vmap_area_list

Stores the virtual area list. makedumpfile gets the vmalloc start value from this variable and its value is necessary for vmalloc translation.\
存储虚拟区域列表。makedumpfile 从该变量获取 vmalloc 开始值，其值对于 vmalloc 转换是必要的。

### mem_map

Physical addresses are translated to struct pages by treating them as an index into the mem_map array. Right-shifting a physical address PAGE_SHIFT bits converts it into a page frame number which is an index into that mem_map array.\
通过将物理地址转换为 struct pages，作为索引写入到 mem_map 数组中。右移一个物理地址的 PAGE_SHIFT 位将其转换为页框号，该页框号是 mem_map 数组的索引。

Used to map an address to the corresponding struct page.\
用于将地址映射到相应的 struct page。

### contig_page_data

Makedumpfile gets the pglist_data structure from this symbol, which is used to describe the memory layout.\
Makedumpfile pglist_data符号获取结构，该符号用于描述内存布局。

User-space tools use this to exclude free pages when dumping memory.\
用户空间工具在转储内存时使用此来排除可用页面。

## x86_64
x86_64
