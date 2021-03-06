#lab2

###实验目的
- 理解基于段页式内存地址的转换机制
- 理解页表的建立和使用方法
- 理解物理内存的管理方法

###实验内容
本次实验包含三个部分。首先了解如何发现系统中的物理内存；然后了解如何建立对物理内存的初步管理，即了解连续物理内存管理；最后了解页表相关的操作，即如何建立页表来实现虚拟内存到物理内存之间的映射，对段页式内存管理机制有一个比较全面的了解。本实验里面实现的内存管理还是非常基本的，并没有涉及到对实际机器的优化，比如针对 cache 的优化等。如果大家有余力，尝试完成扩展练习。

##练习题
###练习1:实现first-fit连续物理内存分配算法
在实现first fit 内存分配算法的回收函数时，要考虑地址连续的空闲块之间的合并操作。提示:在建立空闲页块链表时，需要按照空闲页块起始地址来排序，形成一个有序的链表。
Page的数据结构
```
struct Page {
    int ref;                        // page frame's reference counter，被访问的次数
    uint32_t flags;                 // array of flags that describe the status of the page frame，标记位
    unsigned int property;          // the num of free block, used in first fit pm manager，空闲标记块的数量
    list_entry_t page_link;         // free list link
};
```

在/kern/mm/default_pmm.c文件中，函数default_init_memmap，default_alloc_pages，default_free_pages均要被修改。
```
static void
default_init_memmap(struct Page *base, size_t n) {
    assert(n > 0);                                      //n必须大于0
    struct Page *p = base;                              //
    for (; p != base + n; p ++) {                       //
        assert(PageReserved(p));                        //
        p->flags = p->property = 0;                     //标记位设为0,空闲块数量也是0
        SetPageProperty(p);                             //
        set_page_ref(p, 0);
        list_add_before(&free_list,&(p->page_link));    //添加到一个列表中
    }
    base->property = n;                                 //只有第一个空闲块的property表示的是空闲块的数量，其它property都是0
    //SetPageProperty(base);
    nr_free += n;                                       //空闲块数量+n
    //list_add(&free_list, &(base->page_link));
}

static struct Page *
default_alloc_pages(size_t n) {
    assert(n > 0);
    if (n > nr_free) {
        return NULL;
    }
    list_entry_t *le,*nxt;
    le = &free_list;
    while ((le = list_next(le)) != &free_list) {        //遍历
        struct Page *p = le2page(le, page_link);        //获取对应的page结构
        if (p->property >= n) {                         //空闲块大小大于n
            int i;
            for(i=0;i<n;i++)                            //删除这n个空闲块
            {
                nxt=list_next(le);
                struct Page* pp=le2page(le,page_link);
                SetPageReserved(pp);                    //设为被占用
                ClearPageProperty(pp);                  //清除标记
                list_del(le);                           //从列表中删除
                le=nxt;
            }

            if(p->property>n)                           //如果还有剩余，新建一个表项
            {
                struct Page* newpage=le2page(le,page_link);
                newpage->property=p->property-n;        //新表项的大小是property-n
            }
            ClearPageProperty(p);                       //清除标记
            SetPageReserved(p);                         //被占用
            nr_free-=n;                                 //空余数减1
            return p;
            break;
        }
    }
    return NULL;
}

static void
default_free_pages(struct Page *base, size_t n) {
    assert(n > 0);
    assert(PageReserved(base));

    list_entry_t *le=&free_list;
    struct Page * p;
    while((le=list_next(le))!=&free_list)               //遍历查找base对应的那一个表项
    {
      p=le2page(le,page_link);
      if(p>base)
      {
        break;
      }
    }
    //list_add_before(le, base->page_link);
    for(p=base;p<base+n;p++){                       //新添加n个表项
      list_add_before(le,&(p->page_link));
    }
    base->flags=0;                                  //清除标记
    set_page_ref(base,0);                           //访问次数是0
    ClearPageProperty(base);
    SetPageProperty(base);
    base->property=n;                               //空闲块数量n
    
    p=le2page(le,page_link);
    if(base+n==p)                                   //合并可以被合并的空闲块
    {
      base->property+=p->property;
      p->property=0;
    }
    le=list_prev(&(base->page_link));
    p=le2page(le,page_link);
    if(le!=&free_list&&p==base-1)
    {
      while(le!=&free_list)
      {
        if(p->property)
        {
          p->property+=base->property;              //空闲块的大小增加
          base->property=0;                         
          break;
        }
        le=list_prev(le);
        p=le2page(le,page_link);
      }
    }
    nr_free+=n;                                 //空闲块数量加n
    return ;
}

```

first-fit算法的改进空间，first-fit算法每次在列表中查找第一个大小合适的块，但是这样查找的效率比较低，可以将各个空闲的内存块从小到大排列，进行二分查找，这样查找的效率比较高，但是会占用更多的空间。

###练习2：实现寻找虚拟地址对应的页表项
通过设置页表和对应的页表项，可建立虚拟内存地址和物理内存地址的对应关系。其中的get_pte函数是设置页表项环节中的一个重要步骤。此函数找到一个虚地址对应的二级页表项的内核虚地址，如果此二级页表项不存在，则分配一个包含此项的二级页表。
需要修改/kern/mm/mm.c下的get_pteb函数。
```
#if 0
    pde_t *pdep = NULL;   // (1) find page directory entry
    if (0) {              // (2) check if entry is not present
                          // (3) check if creating is needed, then alloc page for page table
                          // CAUTION: this page is used for page table, not for common data page
                          // (4) set page reference
        uintptr_t pa = 0; // (5) get linear address of page
                          // (6) clear page content using memset
                          // (7) set page directory entry's permission
    }
    return NULL;          // (8) return page table entry
#endif
    pde_t* pdep=pgdir+PDX(la);
    if(!(*pdep&PTE_P))
    {
        struct Page* p;
        if(create)
        {
            p=alloc_page();
            if(p==NULL)
                return NULL;
        }else return NULL;
        set_page_ref(p,1);
        uintptr_t pa=page2pa(p);
        memset(KADDR(pa),0,PGSIZE);
        *pdep=pa|PTE_U|PTE_W|PTE_P;
    }
    uintptr_t pa=PDE_ADDR(*pdep);
    pte_t* ka=(pte_t*)KADDR(pa);
    return ka+PTX(la);
//    return &((pte_t*)KADDR(PDE_ADDR(*pdep)))[PTX(la)];
```

请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。

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

如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

当出现页面访问异常的时候，会进入中断处理程序，从把页表从硬盘读到内存中，继续后续的运行，在此过程中要维护现场，保存一些寄存器的值。

###练习3：释放某虚地址所在的页并取消对应二级页表项的映射
当释放一个包含某虚地址的物理内存页时，需要让对应此物理内存页的管理数据结构Page做相关的清除处理，使得此物理内存页成为空闲；另外还需把表示虚地址与物理地址对应关系的二级页表项清除。请仔细查看和理解page_remove_pte函数中的注释。
需要修改/kern/mm/mm.c下的page_remove_pte函数。
```
#if 0
    if (0) {                      //(1) check if this page table entry is present
        struct Page *page = NULL; //(2) find corresponding page to pte
                                  //(3) decrease page reference
                                  //(4) and free this page when page reference reachs 0
                                  //(5) clear second page table entry
                                  //(6) flush tlb
    }
#endif
    if(*ptep&&PTE_P)
    {
        struct Page* p=pte2page(*ptep);
        if(page_ref_dec(p)==0)
            free_page(p);
        *ptep=0;
        tlb_invalidate(pgdir,la);
    }
```

数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？
有对应关系，页表项的基地址存在页目录项中。
如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？
 