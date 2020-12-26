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
Makedumpfile 使用此符号获取 pglist_data结构，该符号用于描述内存布局。

User-space tools use this to exclude free pages when dumping memory.\
用户空间工具在转储内存时使用此符号来排除内存空闲页。

### mem_section|(mem_section, NR_SECTION_ROOTS)|(mem_section, section_mem_map)

The address of the mem_section array, its length, structure size, and the section_mem_map offset.\
数组 mem_section 的地址、长度、结构大小和 section_mem_map 偏移。

It exists in the sparse memory mapping model, and it is also somewhat similar to the mem_map variable, both of them are used to translate an address.\
它存在于稀疏内存映射模型中，并且它也与 mem_map 变量有些相似，它们都用于地址转换。

### MAX_PHYSMEM_BITS

Defines the maximum supported physical address space memory.\
定义支持的最大物理地址空间内存。

### page

The size of a page structure. struct page is an important data structure and it is widely used to compute contiguous memory.\
page structure 结构的大小。struct page 是一种重要的数据结构，它广泛用于计算连续内存。

### pglist_data

The size of a pglist_data structure. This value is used to check if the pglist_data structure is valid. It is also used for checking the memory type.\
pglist_data 结构大小。此值用于检查 pglist_data 结构是否有效。它还用于检查内存类型。

### zone

The size of a zone structure. This value is used to check if the zone structure has been found. It is also used for excluding free pages.\
zone 结构的大小。此值用于检查是否找到了 zone 结构。它还用于排除内存空闲页。

### free_area

The size of a free_area structure. It indicates whether the free_area structure is valid or not. Useful when excluding free pages.\
free_area 结构大小。它指示 free_area 结构是否有效。在排除可用内存空闲页时很有用。

### list_head

The size of a list_head structure. Used when iterating lists in a post-mortem analysis session.\
list_head 结构大小。在事后分析会话中遍历列表时使用。

### nodemask_t

The size of a nodemask_t type. Used to compute the number of online nodes.\
nodemask_t 类型大小。用于计算联机节点的数量。

### (page, flags|_refcount|mapping|lru|_mapcount|private|compound_dtor|compound_order|compound_head)

User-space tools compute their values based on the offset of these variables. The variables are used when excluding unnecessary pages.\
用户空间工具根据这些变量的偏移量计算其值。这些变量用于排除不可用的内存页。

### (pglist_data, node_zones|nr_zones|node_mem_map|node_start_pfn|node_spanned_pages|node_id)

On NUMA machines, each NUMA node has a pg_data_t to describe its memory layout. On UMA machines there is a single pglist_data which describes the whole memory.\
在 NUMA 计算机上，每个 NUMA 节点都有一个 pg_data_t 来描述其内存布局信息。在 UMA 计算机上有一个 pglist_data，它描述了整个内存。

These values are used to check the memory type and to compute the virtual address for memory map.\
这些值用于检查内存类型和计算内存映射的虚拟地址。

### (zone, free_area|vm_stat|spanned_pages)

Each node is divided into a number of blocks called zones which represent ranges within memory. A zone is described by a structure zone.\
每个 node 节点被划分为多个称为 zones 的块，它们表示内存中的范围。zone 由 structure zone 结构描述。

User-space tools compute required values based on the offset of these variables.\
用户空间工具根据这些变量的偏移量计算所需的值。

### (free_area, free_list)

Offset of the free_list’s member. This value is used to compute the number of free pages.\
free_list 成员的偏移。此值用于计算内存空闲页数。

Each zone has a free_area structure array called free_area[MAX_ORDER]. The free_list represents a linked list of free page blocks.\
每个 zone 区域都有一个 free_area 结构数组，称为 free_area[MAX_ORDER]。free_list 表示内存空闲页块的链接列表。

### (list_head, next|prev)

Offsets of the list_head’s members. list_head is used to define a circular linked list. User-space tools need these in order to traverse lists.\
list_head 成员的偏移量。list_head 用于定义循环链接列表。用户空间工具需要 list_head 才能遍历列表。

### (vmap_area, va_start|list)

Offsets of the vmap_area’s members. They carry vmalloc-specific information. Makedumpfile gets the start address of the vmalloc region from this.\
vmap_area 成员的偏移量。它们携带 vmalloc-specific 的信息。Makedumpfile 从中获取 vmalloc 区域的开始地址。

### (zone.free_area, MAX_ORDER)

Free areas descriptor. User-space tools use this value to iterate the free_area ranges. MAX_ORDER is used by the zone buddy allocator.\
空闲区域描述符。用户空间工具使用此值来遍历 free_area 范围。MAX_ORDER 由 zone buddy allocator 使用。

### prb

A pointer to the printk ringbuffer (struct printk_ringbuffer). This may be pointing to the static boot ringbuffer or the dynamically allocated ringbuffer, depending on when the the core dump occurred. Used by user-space tools to read the active kernel log buffer.\
一个指针指向打印环形缓冲区（ printk_ringbuffer 结构）。这可能指向静态引导环形缓冲区或动态分配的环形缓冲区，具体取决于核心转储发生的时间。用户空间工具用于读取活动内核的日志缓冲区。

### printk_rb_static

A pointer to the static boot printk ringbuffer. If @prb has a different value, this is useful for viewing the initial boot messages, which may have been overwritten in the dynamically allocated ringbuffer.\
一个指向静态引导打印环形缓冲区的指针。如果 @prb 具有不同的值，这对于查看初始引导消息非常有用，这些消息可能在动态分配的环缓冲区中被覆盖。

### clear_seq

The sequence number of the printk() record after the last clear command. It indicates the first record after the last SYSLOG_ACTION_CLEAR, like issued by ‘dmesg -c’. Used by user-space tools to dump a subset of the dmesg log.\
最后一个 clear 清除命令后 printk() 记录的序列号。它指示上次 SYSLOG_ACTION_CLEAR 记录后的第一个记录，如由"dmesg-c"发布。用户空间工具用于转储 dmesg 日志的子集。

### printk_ringbuffer

The size of a printk_ringbuffer structure. This structure contains all information required for accessing the various components of the kernel log buffer.\
printk_ringbuffer 结构大小。此结构包含访问内核日志缓冲区各组件所需的所有信息。

### (printk_ringbuffer, desc_ring|text_data_ring|dict_data_ring|fail)

Offsets for the various components of the printk ringbuffer. Used by user-space tools to view the kernel log buffer without requiring the declaration of the structure.\
打印环缓冲器各组件的偏移量。用户空间工具用于查看内核日志缓冲区，而无需声明此结构。

### prb_desc_ring

The size of the prb_desc_ring structure. This structure contains information about the set of record descriptors.\
prb_desc_ring 结构大小。此结构包含有关记录描述符集的信息。

### (prb_desc_ring, count_bits|descs|head_id|tail_id)

Offsets for the fields describing the set of record descriptors. Used by user-space tools to be able to traverse the descriptors without requiring the declaration of the structure.\
描述记录描述符集字段的偏移量。用户空间工具用以遍历描述符，而无需声明此结构。

### prb_desc

The size of the prb_desc structure. This structure contains information about a single record descriptor.\
prb_desc 结构大小。此结构包含有关记录描述符的信息。

### (prb_desc, info|state_var|text_blk_lpos|dict_blk_lpos)

Offsets for the fields describing a record descriptors. Used by user-space tools to be able to read descriptors without requiring the declaration of the structure.\
描述记录描述符字段的偏移量。用户空间工具用以读取描述符，而无需声明此结构。

### prb_data_blk_lpos

The size of the prb_data_blk_lpos structure. This structure contains information about where the text or dictionary data (data block) is located within the respective data ring.\
prb_data_blk_lpos 结构大小。此结构包含有关文本或字典数据（数据块）在相应数据环形缓冲区中位置的信息。

### (prb_data_blk_lpos, begin|next)

Offsets for the fields describing the location of a data block. Used by user-space tools to be able to locate data blocks without requiring the declaration of the structure.\
描述数据块位置字段的偏移量。用户空间工具用以定位数据块，而无需声明此结构。

### printk_info

The size of the printk_info structure. This structure contains all the meta-data for a record.\
printk_info 结构大小。此结构包含记录的所有元数据。

### (printk_info, seq|ts_nsec|text_len|dict_len|caller_id)

Offsets for the fields providing the meta-data for a record. Used by user-space tools to be able to read the information without requiring the declaration of the structure.\
为记录提供元数据字段的偏移量。用户空间工具用以读取信息，而无需声明此结构。

### prb_data_ring

The size of the prb_data_ring structure. This structure contains information about a set of data blocks.\
prb_data_ring 结构大小。此结构包含有关一组数据块的信息。

### (prb_data_ring, size_bits|data|head_lpos|tail_lpos)

Offsets for the fields describing a set of data blocks. Used by user-space tools to be able to access the data blocks without requiring the declaration of the structure.\
描述一组数据块字段的偏移量。用户空间工具用以访问数据块，而无需声明此结构。

### atomic_long_t

The size of the atomic_long_t structure. Used by user-space tools to be able to copy the full structure, regardless of its architecture-specific implementation.\
atomic_long_t 结构大小。用户空间工具用以复制整个结构，而不管其特定于体系结构如何实现。

### (atomic_long_t, counter)

Offset for the long value of an atomic_long_t variable. Used by user-space tools to access the long value without requiring the architecture-specific declaration.\
atomic_long_t 变量的长整型值的偏移量。用户空间工具用于访问长整型值，而无需特定于体系结构的声明。

### (free_area.free_list, MIGRATE_TYPES)

The number of migrate types for pages. The free_list is described by the array. Used by tools to compute the number of free pages.\
页面迁移页数。free_list 用此数组描述。工具用以计算可用空闲内存页数。

### NR_FREE_PAGES

On linux-2.6.21 or later, the number of free pages is in vm_stat[NR_FREE_PAGES]. Used to get the number of free pages.\
在 linux-2.6.21 或更晚的，vm_stat[NR_FREE_PAGES] 表示可用页数。用于获取可用空闲内存页数。

### PG_lru|PG_private|PG_swapcache|PG_swapbacked|PG_slab|PG_hwpoision|PG_head_mask

Page attributes. These flags are used to filter various unnecessary for dumping pages.\
页面属性。这些标志用于筛选各种不必要的转储页面。

### PAGE_BUDDY_MAPCOUNT_VALUE(~PG_buddy)|PAGE_OFFLINE_MAPCOUNT_VALUE(~PG_offline)

More page attributes. These flags are used to filter various unnecessary for dumping pages.\
更多页面属性。这些标志用于筛选各种不必要的转储页面。

### HUGETLB_PAGE_DTOR

The HUGETLB_PAGE_DTOR flag denotes hugetlbfs pages. Makedumpfile excludes these pages.\
HUGETLB_PAGE_DTOR 标志表示 hugetlbfs 页面。Makedumpfile 不包括这些页面。

## x86_64
x86_64

### phys_base

Used to convert the virtual address of an exported kernel symbol to its corresponding physical address.\
用于将导出的内核符号的虚拟地址转换为相应的物理地址。

### init_top_pgt

Used to walk through the whole page table and convert virtual addresses to physical addresses. The init_top_pgt is somewhat similar to swapper_pg_dir, but it is only used in x86_64.\
用于浏览整个页面表并将虚拟地址转换为物理地址。init_top_pgt 与 swapper_pg_dir 有些类似，但仅用于 x86_64。

### pgtable_l5_enabled

User-space tools need to know whether the crash kernel was in 5-level paging mode.\
用户空间工具需要知道崩溃内核是否处于 5 级分页模式。

### node_data

This is a struct pglist_data array and stores all NUMA nodes information. Makedumpfile gets the pglist_data structure from it.\
这是一个 pglist_data 结构并存储所有 NUMA 节点信息。Makedumpfile 从中获取 pglist_data 结构。

### (node_data, MAX_NUMNODES)

The maximum number of nodes in system.\
系统中的最大节点数。

### KERNELOFFSET

The kernel randomization offset. Used to compute the page offset. If KASLR is disabled, this value is zero.\
内核随机偏移。用于计算页面偏移量。如果禁用 KASLR，则此值为零。

### KERNEL_IMAGE_SIZE

Currently unused by Makedumpfile. Used to compute the module virtual address by Crash.\
Makedumpfile 当前未使用此值 。Crash 用于计算模块虚拟地址。

### sme_mask

AMD-specific with SME support: it indicates the secure memory encryption mask. Makedumpfile tools need to know whether the crash kernel was encrypted. If SME is enabled in the first kernel, the crash kernel’s page table entries (pgd/pud/pmd/pte) contain the memory encryption mask. This is used to remove the SME mask and obtain the true physical address.\
AMD 特定于支持 SME：它指示安全内存加密掩码。Makedumpfile 工具需要知道崩溃内核是否加密。如果在第一个内核中启用了 SME，则崩溃内核的页面表条目（pgd/pud/pmd/pte）包含内存加密掩码。这用于删除 SME 掩码并获取真正的物理地址。

Currently, sme_mask stores the value of the C-bit position. If needed, additional SME-relevant info can be placed in that variable.\
目前，sme_mask 存储 C-bit 位置的值。如果需要，可以在该变量中放置其他 SME 相关信息。

For example:\
例如：

    [ misc                ][ enc bit  ][ other misc SME info       ]
    0000_0000_0000_0000_1000_0000_0000_0000_0000_0000_..._0000
    63   59   55   51   47   43   39   35   31   27   ... 3

## x86_32
x86_32

### X86_PAE

Denotes whether physical address extensions are enabled. It has the cost of a higher page table lookup overhead, and also consumes more page table space per process. Used to check whether PAE was enabled in the crash kernel when converting virtual addresses to physical addresses.\
表示是否启用物理地址扩展。它具有较高的页面表查找开销的成本，并且每个进程也会占用更多的页面表空间。用于检查在将虚拟地址转换为物理地址时是否在崩溃内核中启用了 PAE。

## ia64

### pgdat_list|(pgdat_list, MAX_NUMNODES)

pg_data_t array storing all NUMA nodes information. MAX_NUMNODES indicates the number of the nodes.\
pg_data_t 数组存储所有 NUMA 节点信息。MAX_NUMNODES 指示节点数。

### node_memblk|(node_memblk, NR_NODE_MEMBLKS)

List of node memory chunks. Filled when parsing the SRAT table to obtain information about memory nodes. NR_NODE_MEMBLKS indicates the number of node memory chunks.\
节点内存块的列表。分析 SRAT 表以获取有关内存节点的信息时填充。NR_NODE_MEMBLKS 指示节点内存块的数量。

These values are used to compute the number of nodes the crashed kernel used.\
这些值用于计算崩溃内核使用的节点数。

### node_memblk_s|(node_memblk_s, start_paddr)|(node_memblk_s, size)

The size of a struct node_memblk_s and the offsets of the node_memblk_s’s members. Used to compute the number of nodes.\
node_memblk_s 结构的大小和 node_memblk_s 结构的偏移。用于计算节点数。

### PGTABLE_3|PGTABLE_4

User-space tools need to know whether the crash kernel was in 3-level or 4-level paging mode. Used to distinguish the page table.\
用户空间工具需要知道崩溃内核是处于 3 级还是 4 级分页模式。用于区分页表。

## ARM64
ARM64

### VA_BITS

The maximum number of bits for virtual addresses. Used to compute the virtual memory ranges.\
虚拟地址的最大位数。用于计算虚拟内存范围。

### kimage_voffset

The offset between the kernel virtual and physical mappings. Used to translate virtual to physical addresses.\
内核虚拟映射和物理映射之间的偏移量。用于将虚拟地址转换为物理地址。

### PHYS_OFFSET

Indicates the physical address of the start of memory. Similar to kimage_voffset, which is used to translate virtual to physical addresses.\
指示内存开始的物理地址。类似于 kimage_voffset，用于将虚拟地址转换为物理地址。

### KERNELOFFSET

The kernel randomization offset. Used to compute the page offset. If KASLR is disabled, this value is zero.\
内核随机偏移。用于计算页面偏移量。如果禁用 KASLR，则此值为零。

### KERNELPACMASK

The mask to extract the Pointer Authentication Code from a kernel virtual address.\
用于从内核虚拟地址提取 Pointer Authentication Code 的掩码。

### TCR_EL1.T1SZ

Indicates the size offset of the memory region addressed by TTBR1_EL1. The region size is 2^(64-T1SZ) bytes.\
指示由 TTBR1_EL1 处理的内存区域定位的偏移大小。区域大小为 2^(64-T1SZ) 字节。

TTBR1_EL1 is the table base address register specified by ARMv8-A architecture which is used to lookup the page-tables for the Virtual addresses in the higher VA range (refer to ARMv8 ARM document for more details).\
TTBR1_EL1 是 ARMv8-A 体系结构指定的表基地址寄存器，用于查找较高 VA 范围内虚拟地址的页面表（有关更多详细信息，请参阅 ARMv8 ARM 文档）。

## arm

### ARM_LPAE

It indicates whether the crash kernel supports large physical address extensions. Used to translate virtual to physical addresses.\
它指示崩溃内核是否支持大型物理地址扩展。用于将虚拟地址转换为物理地址。

## s390

### lowcore_ptr

An array with a pointer to the lowcore of every CPU. Used to print the psw and all registers information.\
具有指向每个 CPU 的 lowcore 的指针的数组。用于打印 psw 和所有寄存器信息。

### high_memory

Used to get the vmalloc_start address from the high_memory symbol.\
用于从 high_memory 符号获取 vmalloc_start 地址。

### (lowcore_ptr, NR_CPUS)

The maximum number of CPUs.\
CPU 的最大数量。

## powerpc

### node_data|(node_data, MAX_NUMNODES)

See above.\
见上文。

### contig_page_data

See above.\
见上文。

### vmemmap_list

The vmemmap_list maintains the entire vmemmap physical mapping. Used to get vmemmap list count and populated vmemmap regions info. If the vmemmap address translation information is stored in the crash kernel, it is used to translate vmemmap kernel virtual addresses.\
vmemmap_list 维护整个 vmemmap 物理映射。用于获取 vmemmap 列表计数和填充的 vmemmap 区域信息。如果 vmemmap 地址转换信息存储在崩溃内核中，则用于转换 vmemmap 内核虚拟地址。

### mmu_vmemmap_psize

The size of a page. Used to translate virtual to physical addresses.\
页面的大小。用于将虚拟地址转换为物理地址。

### mmu_psize_defs

Page size definitions, i.e. 4k, 64k, or 16M.
页面大小定义，即 4k、64k 或 16M。

Used to make vtop translations.\
用于进行 vtop 转换。

### vmemmap_backing|(vmemmap_backing, list)|(vmemmap_backing, phys)|(vmemmap_backing, virt_addr)

The vmemmap virtual address space management does not have a traditional page table to track which virtual struct pages are backed by a physical mapping. The virtual to physical mappings are tracked in a simple linked list format.\
vmemmap 虚拟地址空间管理，没有传统的页面表来跟踪哪些虚拟结构页支持物理映射。虚拟到物理映射以简单的链接列表格式进行跟踪。

User-space tools need to know the offset of list, phys and virt_addr when computing the count of vmemmap regions.\
用户空间工具在计算 vmemmap 区域数时，需要知道链接列表偏移量、phys 和 virt_addr。

### mmu_psize_def|(mmu_psize_def, shift)

The size of a struct mmu_psize_def and the offset of mmu_psize_def’s member.\
mmu_psize_def 结构的大小和 mmu_psize_def 成员的偏移量。

Used in vtop translations.\
用于 vtop 转换。

## sh

### node_data|(node_data, MAX_NUMNODES)

See above.\
见上文。

### X2TLB

Indicates whether the crashed kernel enabled SH extended mode.\
指示崩溃的内核是否启用 SH 扩展模式。

