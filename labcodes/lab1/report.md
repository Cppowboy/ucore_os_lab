#lab1实验报告
##实验目的
操作系统是一个软件，也需要通过某种机制加载并运行它。在这里我们将通过另外一个更加简单的软件-bootloader来完成这些工作。为此，我们需要完成一个能够切换到x86的保护模式并显示字符的bootloader，为启动操作系统ucore做准备。lab1提供了一个非常小的bootloader和ucore OS，整个bootloader执行代码小于512个字节，这样才能放到硬盘的主引导扇区中。
##实验内容
lab1中包含一个bootloader和一个OS。这个bootloader可以切换到X86保护模式，能够读磁盘并加载ELF执行文件格式，并显示字符。而这lab1中的OS只是一个可以处理时钟中断和显示字符的幼儿园级别OS。
##练习一
1. 操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)
    执行命令"make V="，显示结果如下
    ```
    * cc kern/init/init.c
    gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
    kern/init/init.c:95:1: warning: ‘lab1_switch_test’ defined but not used [-Wunused-function]
     lab1_switch_test(void) {
     ^
    * cc kern/libs/readline.c
    gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/readline.c -o obj/kern/libs/readline.o
    * cc kern/libs/stdio.c
    gcc -Ikern/libs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/libs/stdio.c -o obj/kern/libs/stdio.o
    * cc kern/debug/kdebug.c
    gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kdebug.c -o obj/kern/debug/kdebug.o
    kern/debug/kdebug.c:251:1: warning: ‘read_eip’ defined but not used [-Wunused-function]
     read_eip(void) {
     ^
    * cc kern/debug/kmonitor.c
    gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/kmonitor.c -o obj/kern/debug/kmonitor.o
    * cc kern/debug/panic.c
    gcc -Ikern/debug/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/debug/panic.c -o obj/kern/debug/panic.o
    * cc kern/driver/clock.c
    gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/clock.c -o obj/kern/driver/clock.o
    * cc kern/driver/console.c
    gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/console.c -o obj/kern/driver/console.o
    * cc kern/driver/intr.c
    gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/intr.c -o obj/kern/driver/intr.o
    * cc kern/driver/picirq.c
    gcc -Ikern/driver/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/driver/picirq.c -o obj/kern/driver/picirq.o
    * cc kern/trap/trap.c
    gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trap.c -o obj/kern/trap/trap.o
    kern/trap/trap.c:14:13: warning: ‘print_ticks’ defined but not used [-Wunused-function]
     static void print_ticks() {
                 ^
    kern/trap/trap.c:30:26: warning: ‘idt_pd’ defined but not used [-Wunused-variable]
     static struct pseudodesc idt_pd = {
                              ^
    * cc kern/trap/trapentry.S
    gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/trapentry.S -o obj/kern/trap/trapentry.o
    * cc kern/trap/vectors.S
    gcc -Ikern/trap/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/trap/vectors.S -o obj/kern/trap/vectors.o
    * cc kern/mm/pmm.c
    gcc -Ikern/mm/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/mm/pmm.c -o obj/kern/mm/pmm.o
    * cc libs/printfmt.c
    gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/printfmt.c -o obj/libs/printfmt.o
    * cc libs/string.c
    gcc -Ilibs/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/  -c libs/string.c -o obj/libs/string.o
    * ld bin/kernel
    ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o obj/kern/debug/kmonitor.o obj/kern/debug/panic.o obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o obj/kern/driver/picirq.o obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o
    * cc boot/bootasm.S
    gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
    * cc boot/bootmain.c
    gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o
    * cc tools/sign.c
    gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
    gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
    * ld bin/bootblock
    ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
    'obj/bootblock.out' size: 472 bytes
    build 512 bytes boot sector: 'bin/bootblock' success!
    dd if=/dev/zero of=bin/ucore.img count=10000
    10000+0 records in
    10000+0 records out
    5120000 bytes (5.1 MB) copied, 0.0159265 s, 321 MB/s
    dd if=bin/bootblock of=bin/ucore.img conv=notrunc
    1+0 records in
    1+0 records out
    512 bytes (512 B) copied, 0.000110722 s, 4.6 MB/s
    dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
    138+1 records in
    138+1 records out
    70783 bytes (71 kB) copied, 0.000315606 s, 224 MB/s
    ```
    即用gcc或cc将.c文件编译为.o文件，然后用ld将它们链接起来，最后用dd生成ucore.img文件。
    Makefile文件中有如下描述：
    ```
    # create ucore.img
    UCOREIMG    := $(call totarget,ucore.img)

    $(UCOREIMG): $(kernel) $(bootblock)
        $(V)dd if=/dev/zero of=$@ count=10000
        $(V)dd if=$(bootblock) of=$@ conv=notrunc
        $(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc

    $(call create_target,ucore.img)
    ```
    可知，先要生成kernel和bootblock，才能生成ucore.img。
    一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?
    在/tools/sign.c中，有如下代码：
    ```
    if (st.st_size > 510) {
        fprintf(stderr, "%lld >> 510!!\n", (long long)st.st_size);
        return -1;
    }
    char buf[512];
    memset(buf, 0, sizeof(buf));
    FILE *ifp = fopen(argv[1], "rb");
    int size = fread(buf, 1, st.st_size, ifp);
    if (size != st.st_size) {
        fprintf(stderr, "read '%s' error, size is %d.\n", argv[1], size);
        return -1;
    }
    fclose(ifp);
    buf[510] = 0x55;
    buf[511] = 0xAA;
    FILE *ofp = fopen(argv[2], "wb+");
    size = fwrite(buf, 1, 512, ofp);
    if (size != 512) {
        fprintf(stderr, "write '%s' error, size is %d.\n", argv[2], size);
        return -1;
    }
    fclose(ofp);
    printf("build 512 bytes boot sector: '%s' success!\n", argv[2]);
    ```
    可以看到，大小是512个字节，且最后两个字节是0x55AA，前面的内容的大小不能超过510个字节。
2. 为了熟悉使用qemu和gdb进行的调试工作，我们进行如下的小练习：
    + 从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。
    执行命令make debug，并在显示的gdb界面中执行命令si，就可以开始单步调试了，x /10i $pc可以用来显示当前的命令。
    + 在初始化位置0x7c00设置实地址断点,测试断点正常。
    在/tools/gdbinit文件中加入
    ```
    b 0x7c00
    c
    x /10i $pc
    ```
    然后执行make debug，在0x7c00处中断，且显示出在0x7c00处的十条指令。
    + 从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
    步骤与上一步相同，显示的指令如下：
    ```
    0x7c00 <start>:      cli
   0x7c01 <start+1>:    cld
   0x7c02 <start+2>:    xor    %eax,%eax
   0x7c04 <start+4>:    mov    %eax,%ds
   0x7c06 <start+6>:    mov    %eax,%es
   0x7c08 <start+8>:    mov    %eax,%ss
   0x7c0a <seta20.1>:   in     $0x64,%al
   0x7c0c <seta20.1+2>: test   $0x2,%al
   0x7c0e <seta20.1+4>: jne    0x7c0a <seta20.1>
   0x7c10 <seta20.1+6>: mov    $0xd1,%al
    ```
    这与/boot/bootasm.S中的内容相同。
3. BIOS将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行bootloader。请分析bootloader是如何完成从实模式进入保护模式的。
    ```
    .code16                                             # Assemble for 16-bit mode
        # 取消中断，寄存器清零
        cli                                             # Disable interrupts
        cld                                             # String operations increment
    
        # Set up the important data segment registers (DS, ES, SS).
        xorw %ax, %ax                                   # Segment number zero
        movw %ax, %ds                                   # -> Data Segment
        movw %ax, %es                                   # -> Extra Segment
        movw %ax, %ss                                   # -> Stack Segment
    
        # 开启A20
        # Enable A20:
        #  For backwards compatibility with the earliest PCs, physical
        #  address line 20 is tied low, so that addresses higher than
        #  1MB wrap around to zero by default. This code undoes this.
    seta20.1:
        inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
        testb $0x2, %al
        jnz seta20.1
    
        movb $0xd1, %al                                 # 0xd1 -> port 0x64
        outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port
    
    seta20.2:
        inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
        testb $0x2, %al
        jnz seta20.2
    
        movb $0xdf, %al                                 # 0xdf -> port 0x60
        outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
    
        #切换到保护模式
        # Switch from real to protected mode, using a bootstrap GDT
        # and segment translation that makes virtual addresses
        # identical to physical addresses, so that the
        # effective memory map does not change during the switch.
        #初始化GDT
        lgdt gdtdesc

        #进入保护模式，把cr0寄存器PE位赋值为1
        movl %cr0, %eax
        orl $CR0_PE_ON, %eax
        movl %eax, %cr0
    
        # Jump to next instruction, but in 32-bit code segment.
        # Switches processor into 32-bit mode.
        #跳转到protcseg函数
        ljmp $PROT_MODE_CSEG, $protcseg
    ```

    ```
        .code32                                             # Assemble for 32-bit mode
        protcseg:
            # Set up the protected-mode data segment registers
            # 设置寄存器
            movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
            movw %ax, %ds                                   # -> DS: Data Segment
            movw %ax, %es                                   # -> ES: Extra Segment
            movw %ax, %fs                                   # -> FS
            movw %ax, %gs                                   # -> GS
            movw %ax, %ss                                   # -> SS: Stack Segment

            # Set up the stack pointer and call into C. The stack region is 
            from 0--start(0x7c00)
            # 建立堆栈
            movl $0x0, %ebp
            movl $start, %esp
            #调用bootmain函数
            call bootmain
        ```
4. 通过阅读bootmain.c，了解bootloader如何加载ELF文件。通过分析源代码和通过qemu来运行并调试bootloader&OS，bootloader如何读取硬盘扇区的？bootloader是如何加载ELF格式的OS？
    static void readsect(void *dst, uint32_t secno)将第secno个扇区读到dst的位置。
    static void readseg(uintptr_t va, uint32_t count, uint32_t offset)可以读多个扇区。
    ```
    /* bootmain - the entry of bootloader */
    void
    bootmain(void) {
        // read the 1st page off disk
        //读取第一个扇区，也就是文件的头部
        readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

        // is this a valid ELF?
        if (ELFHDR->e_magic != ELF_MAGIC) {
            goto bad;
        }

        struct proghdr *ph, *eph;

        // load each program segment (ignores ph flags)
        ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
        eph = ph + ELFHDR->e_phnum;
        //读入内存
        for (; ph < eph; ph ++) {
            readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
        }

        // call the entry point from the ELF header
        // note: does not return
        //找到内核地址
        ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

    bad:
        outw(0x8A00, 0x8A00);
        outw(0x8A00, 0x8E00);

        /* do nothing */
        while (1);
    }
    ```
5. 我们需要在lab1中完成kdebug.c中函数print_stackframe的实现，可以通过函数print_stackframe来跟踪函数调用堆栈中记录的返回地址。
    ebp[0]储存着调用者的ebp，然后循环就可以知道其它的ebp。ebp[1]是调用者调用时的eip，ebp[2]等是函数参数。
    ```
        /* LAB1 YOUR CODE : STEP 1 */
         /* (1) call read_ebp() to get the value of ebp. the type is (uint32_t);
          - (2) call read_eip() to get the value of eip. the type is (uint32_t);
          - (3) from 0 .. STACKFRAME_DEPTH
          -    (3.1) printf value of ebp, eip
          -    (3.2) (uint32_t)calling arguments [0..4] = the contents in address (unit32_t)ebp +2 [0..4]
          -    (3.3) cprintf("\n");
          -    (3.4) call print_debuginfo(eip-1) to print the C calling function name and line number, etc.
          -    (3.5) popup a calling stackframe
          -           NOTICE: the calling funciton's return addr eip  = ss:[ebp+4]
          -                   the calling funciton's ebp = ss:[ebp]
          */
        uint32_t ebp=read_ebp();
        uint32_t eip=read_eip();
        int i,j;
        for(i=0;i<STACKFRAME_DEPTH;i++)
        {
            if(ebp==0)
                break;
            cprintf("ebp:0x%08x eip:0x%08x args:", ebp, eip);
            uint32_t *args = (uint32_t *)ebp + 2;
            for (j = 0; j < 4; j ++) {
                cprintf("0x%08x ", args[j]);
            }
            cprintf("\n");
            print_debuginfo(eip - 1);
            eip = ((uint32_t *)ebp)[1];
            ebp = ((uint32_t *)ebp)[0];
        }
    ```
6. 请完成编码工作和回答如下问题：
    中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？
    8个字节，第0，1,6,7位构成入口的地址。
    请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。
    extern uintptr_t __vectors[];
    int i;
    for (i = 0; i < 256; i ++)
    {
        SETGATE(idt[i],0,GD_KTEXT,__vectors[i],0);
    }
    SETGATE(idt[T_SWITCH_TOK],0,GD_KTEXT,__vectors[T_SWITCH_TOK],3);
    lidt(&idt_pd);
    请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向屏幕上打印一行文字”100 ticks”。
    ticks++;
    if(ticks%TICK_NUM==0)
        print_ticks();
    break;


