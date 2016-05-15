###lab3

####实验目的
- 了解虚拟内存的Page Fault异常处理实现
- 了解页替换算法在操作系统中的实现
####实验内容
本次实验是在实验二的基础上，借助于页表机制和实验一中涉及的中断异常处理机制，完成Page Fault异常处理和FIFO页替换算法的实现，结合磁盘提供的缓存空间，从而能够支持虚存管理，提供一个比实际物理内存空间“更大”的虚拟内存空间给系统使用。这个实验与实际操作系统中的实现比较起来要简单，不过需要了解实验一和实验二的具体实现。实际操作系统系统中的虚拟内存管理设计与实现是相当复杂的，涉及到与进程管理系统、文件系统等的交叉访问。

####练习1：给未被映射的地址映射上物理页（需要编程）
完成do_pgfault（mm/vmm.c）函数，给未被映射的地址映射上物理页。设置访问权限 的时候需要参考页面所在 VMA 的权限，同时需要注意映射物理页时需要操作内存控制 结构所指定的页表，而不是内核的页表。注意：在LAB2 EXERCISE 1处填写代码。执行
make　qemu
后，如果通过check_pgfault函数的测试后，会有“check_pgfault() succeeded!”的输出，表示练习1基本正确。

请在实验报告中简要说明你的设计实现过程。请回答如下问题：

do_pgfault用来处理页缺失中断，处理三种情形：写一个有效地址，写一个无效但是可写的地址，读一个无效但是可读的地址。发生缺页时，首先从硬盘的swap中读取一个页，因此要完成线性地址到swap的对应扇区的转换。 发生页缺失，用get_pte获取页表项，然后对相应的page处理。 页表项全部为空指针时，没有映射对应的物理空间，用pgdir_alloc_page分配空间，同时调用page_insert建立地址映射，加入FIFO链表。

请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。
如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

```
// page directory index
#define PDX(la) ((((uintptr_t)(la)) >> PDXSHIFT) & 0x3FF)
//二级页表在页目录项当中的index，用来作为索引查询二级页表
// page table index
#define PTX(la) ((((uintptr_t)(la)) >> PTXSHIFT) & 0x3FF)

// page number field of address
#define PPN(la) (((uintptr_t)(la)) >> PTXSHIFT)

// offset in page
#define PGOFF(la) (((uintptr_t)(la)) & 0xFFF)

// construct linear address from indexes and offset
#define PGADDR(d, t, o) ((uintptr_t)((d) << PDXSHIFT | (t) << PTXSHIFT | (o)))

// address in page table or page directory entry
页表或者页目录项的地址
#define PTE_ADDR(pte)   ((uintptr_t)(pte) & ~0xFFF)
#define PDE_ADDR(pde)   PTE_ADDR(pde)

/* page table/directory entry flags */
#define PTE_P           0x001                   // Present，是否存在
#define PTE_W           0x002                   // Writeable，是否可写
#define PTE_U           0x004                   // User，用户的权限
#define PTE_PWT         0x008                   // Write-Through，可否拖后写
#define PTE_PCD         0x010                   // Cache-Disable，可否支持cache
#define PTE_A           0x020                   // Accessed，是否可访问
#define PTE_D           0x040                   // Dirty，是否是脏页
#define PTE_PS          0x080                   // Page Size，页的大小
#define PTE_MBZ         0x180                   // Bits must be zero
#define PTE_AVAIL       0xE00                   // Available for software use
                                                // The PTE_AVAIL bits aren't used by the kernel or interpreted by the，是否可用
                                                // hardware, so user processes are allowed to set them arbitrarily.

#define PTE_USER        (PTE_U | PTE_W | PTE_P)

```

当出现页面访问异常的时候，会进入中断处理程序，从把页表从硬盘读到内存中，继续后续的运行，在此过程中要维护现场，保存一些寄存器的值。

练习2：补充完成基于FIFO的页面替换算法（需要编程）

完成vmm.c中的do_pgfault函数，并且在实现FIFO算法的swap_fifo.c中完成map_swappable和swap_out_vistim函数。通过对swap的测试。注意：在LAB2 EXERCISE 2处填写代码。执行

make　qemu
后，如果通过check_swap函数的测试后，会有“check_swap() succeeded!”的输出，表示练习2基本正确。

请在实验报告中简要说明你的设计实现过程。

_fifo_map_swappable：添加相应的表项即可list_add(head,entry)。
_fifo_swap_out_victim：先把pra_list中最早的页删除，然后获取这个被删除的页。

请在实验报告中回答如下问题：

如果要在ucore上实现"extended clock页替换算法"请给你的设计方案，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。如果不是，请给出你的新的扩展和基此扩展的设计方案。并需要回答如下问题
需要被换出的页的特征是什么？
access和dirty都是0。
在ucore中如何判断具有这样特征的页？
判断PTE_A和PTE_D是否为0。
何时进行换入和换出操作？