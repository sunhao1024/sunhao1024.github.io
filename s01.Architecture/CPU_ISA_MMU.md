# CPU 设计哲学
* 现代 CPU 设计主要组成部分
1. 流水线
2. 分支预测
3. 超标量
4. 乱序执行
5. 存储器层次
6. 矢量操作
7. 多核处理

* 程序运行效率
CPI(单指令时钟数) * 主频
cycles/instruction * seconds/cycle

* CPU core
ALU GPRn(Lo/Hi)
CP0	MMU
CP1	FPU
CP2	MMI

* ABI
ABI 规定 GPRn 的用途(o32, n32, n64)
0, RA, SP, GP, FP
ABI 规定内存位域 MSB/LSB, 字节序(大小端)
`字节序`
在编译时检测字节序, 一些编译器或系统头文件提供宏来指定字节序, GNU-C 通常提供一个 <endian.h> 包含以下宏
```c
BYTE_ORDER
#define LITTLE_ENDIAN 1234
#define BIG_ENDIAN
define htobe16(x) __bswap_16 (x)
```

平台设备, PCIe 设备, HT 桥
硬件 C-C, 侦听协议, 目录协议

# ISA
* 体系结构(处理器架构)
1. 指令集(架构) 汇编指令 -> 机器指令
2. 寄存器

## x86
X86的指令长度不是固定的, 有向后兼容的沉重历史包袱

Port I/O  IN/OUT 指令访问硬件设备空间(64K 0 - FFFF)
MMIO      PCI规范的一部分, 硬件设备空间映射到内存空间, BIOS 配置

## MIPS
MIPS 简单的指令和格式易于译码和流水线操作, 但是代码密度不高, 导致二进制文件大

MIPS 有 32 个通用寄存器 REG，为什么不是更多呢?
因为更多的寄存器需要更多的指令空间对寄存器编码，也会增加上下文切换的负担。

### MIPS 指令集
mips 指令集属于精简指令集(RISC), 所有指令定长(32 bit, 需要两条指令加载 i32), 三操作数指令(RS,RT,RD)

mips 指令 前6位编码操作种类(最多有 2^6 64条指令), 条件分支指令 < 64K
后26位划分为R指令, I指令和J指令, 寄存器以5bit来编码, 没有 i8/i16 运算
* R指令 纯寄存器指令
所有的操作数(除移位量)均在寄存器中, op字段为0,使用 funct 字段区分指令

* I指令 带立即数的指令
最多使用两个寄存器,包括L/S指令,使用Op字段区分指令

* J指令 长跳转指令
仅有一个立即数操作数,使用Op字段区分指令
跳转范围
编译器开发,优化远距离跳转(中值/偏移),加载大立即数($at重组)

* 关于跳转/寻址范围
J指令:操作码6位,地址字段26位,允许跳转的地址空间为256M(2^28)
指令4字节对齐,最低两个有效位不需要存储。PC是32位的,MIPS跳转指令只替换PC的低28位,高4位保留原值。因此,加载和链接程序必须避免跨越256MB
B指令:条件分支跳转,地址字段16位,可寻址PC±128K(2^18=256K)
寄存器寻址:段外分支跳转,可以跳转到任意地址


整数乘法结果存放在两个寄存器中 mfhi|mflo(结果互锁, 慢于下一条指令)

jal/j R31 PC range 延迟槽
lui/ori
addiu/daddiu
lw/ld
jr/jar
b/bal

访存(L/S) LW,LD LUI(加载无符号立即数的高半字 16bit << 16), LHU(加载半字, 无符号拓展)
计算 ADDI/DADDI IDDIU/DADDIU(没有溢出异常)
逻辑 AND/OR/XOR ORI(立即数加载)
移位 SLL/SLR(逻辑左/右移位, 0扩展) SRA(算数右移, 符号扩展) ROTR(循环右移)
跳转

* Eg. 三操作数加法指令
`add $t0,$s0,$s1`
表示 `$t0 = $s0 + $s1`
	| 操作码 | 操作数1/寄存器号 | 操作数2/寄存器号 | 目的寄存器号 |
	| ------ | ---------------- | ---------------- | ------------ |
	| add    | $t0              | $s0              | $s1          |

  * R-指令各字段的十进制表示为
| OP  | RS  | RT  | RD  | shamt | funct |
| --- | --- | --- | --- | ----- | ----- |
| 0   | 16  | 17  | 8   | 0     | 32    |



### 协处理器
可选部件, 负责指令集在某方面的拓展

CPx
* CP0	处理器控制
CPU 配置(大/小端), cache 控制, IT 控制, 存储管理单元控制
CP0 寄存器:
PRid
SR
异常/中断源
原因寄存器(5 bit ExcCode)
EPC
内部计数器
地址异常
TLB 相关
Config 参数, 冒险屏障
异常入寇基地址
中断向量/优先级
影子
ejtage

```ass
mfc0 d, $n		# 32 bit 读操作
dmfc0 d, $n		# 64 bit 读操作
mtc0 s,<n>		# 把寄存器 s 的值写入 CP0 的 寄存器 n
```


* CP1	FPU
* CP2	Soc 特定指令实现
* CP3	FPU/Media 拓展

### MIPS 虚拟地址空间
kuseg、kseg0、kseg1、kseg2(从低地址到高地址)

| 别名  | 大小 | 属性        | 用途                 |
| ----- | ---- | ----------- | -------------------- |
| kseg2 | 1G   | map/cache   | 高端内存, 内核模块   |
| kseg1 | 512M | ~map/~cache | ROM 存储在           |
| kseg0 | 512M | ~map/cache  | 内核代码和数据存储在 |
| kuseg | 2G   | map/cache   | 用户程序             |
高端内存 > 512M 需要 MMU 映射访问, 低于 512 M 的内存可直接映射到 kseg0， kseg1 访问

### MMU
内存管理单元, 负责 TLB 和 Cache 的管理, 属于一种体系结构中除 ISA 和 寄存器设计外最具区分度的特征

* MMU 设计哲学
1. 安全性
2. 为程序分配连续的内存空间
3. 扩展地址空间
4. 按需分页
5. 重定位

* CPU 访存过程:
CPU 任何时候发出的都是虚拟地址, 需要发给 MMU 查询 TLB/页表得到物理地址, 在该地址经过 cache/主存得到数据

虚拟地址 -> 访问TLB -> TLB命中 -> 物理地址 -> 访问Cache -> Cache命中 -> CPU取得数据

* 访存特点
寄存器和L1缓存可以操作指定字长的数据，而更后面的内存子系统就不会再使用这么小的单位了，都是直接移动数据块，比如以缓存线为单位访问数据

#### 页表
vfn <---> pfn
查页表得到物理页号(pfn), 与原页内偏移组合得到物理地址

* 一级页表
|vfn|offset_in_page|
32  11			   0

一级页表一个 page 为 4K (2^12), vfn 20 bit, 4GB 的内存空间需要 1M 个页表项, 需要占用 4MB 的内存空间(页表需要完全覆盖内存地址空间)

* linux 页表结构
linux 使用三级页表结构, PGD/PMD/PTE

通过在页表项中加入一些标识位可减少访存次数，如修改位，若没有修改位每次替换页面都需要两次访存(一次换入，一次写出), 而加了修改位后可以仅在内存被修改时才写出(内存是外存的副本, 可直接丢弃)

#### TLB
内存映射关系表(快表), 本质上也为 cache, 只是 cache line 的内容为 pfn
cache line 中还包含一些 TLB 特定的 flags

* mips TLB 设计


#### Cache
cache 的设计利用了空间局域性, CPU 直接访存是以字节为单位, cache miss 后则是将若干个字节(一个内存页)加载到一个 cache line 中

cache 信息
/sys/devices/system/cpu/cpu0/cache

CISC 体系结构的 cache 常视为内存的一部分, 而 RISC 体系结构的 cache 更接近于 CPU 的一部分, 因为其与流水线紧密绑定, CPU直接从 cache 中获取数据

MIPS L1-cache(I-cache/D-cache) 独立可并发

* cache 实现方案
  1. 直接映射
|   pfn	  |	page_offset		|
|tag|index|cache_line_offset|
内存中的每个页直接依次线性映射到 cache line, 映射满之后从头再开始下一组映射, 即多对一的映射, 此时对 cache 来说内存分页大小为(cache line 数 * cache line 大小)
Eg. cache line 有 m 行, 内存大小为 n * m, 则访问内存时取 x mod m 得到对应的 cache line
直接映射的 tag 和 index 均来自 pfn, 根据 index 确定 cache line, tag 确定是否命中, 最后根据 offset 访问具体的数据
该种方案映射关系简单, 硬件容易实现, 转换速度快, 不涉及替换算法, 适合于大容量Cache
但是缓存映射关系是多对一, 访存不均匀容易产生竞争和冲突, cache 空间利用不均衡, 命中率低
  2. 全相连
|	pfn		|	page_offset		|
|	tag		|cache_line_offset	|
内存中的每个页可以映射到任一 cache line, 即多对多
全相连仅有 tag, 直接对比 tag 确认是否命中
该种方案映射灵活, 块冲突概率低, cache 利用率高, 适合于小容量Cache
但是硬件设计复杂, 且需要替换算法

  3. 组相连
|	pfn	|		page_offset		|
|	tag	|index|cache_line_offset|
将 cache line 分为若干组, 然后与内存页帧直接映射, 所有组映射完之后从头开始重复映射, 这样一个 cache 组就对应多个内存页帧, 这些页帧在这个组内再全相联映射
pfn 直接作为 tag , index 确定 cache group, tag 确定 cache way
index 来自虚拟页面偏移, 不需要 MMU 转换, tag 需要先经过 MMU 转换, 因此 L1-cache 属于 VIPT, 可以并行查找(cache index 索引和 MMU 转换)
前提是 L1-cache 的一个页大小(即一路的大小 group数 * cache line 大小)  大小不超过一个内存页面的大小
index 的位数由 cache 的组数决定, lg(groups)
tag 的位数由 cache 的页大小决定, lg(内存容量/ cache 页大小), cache 页大小和内存页大小相等时, tag = pfn
cache_line_offset 的位数由  cache line 的大小决定
而 L2-cache 的页大小普遍大于内存页大小, 即 index 的位域会扩展到 pfn 的范围是 PIPT的, 但并不影响访存过程, 因为在 L1-cache 时已经拿到了物理地址

* 一个 cache line 的内容
在一行 cache line 中除了上述内容还有该行 cache 的状态 flag
L1-Icache 有 invalid(无效位), shared(共享位)
L1-Dcache 和 L2-cache 有 MESI 4个状态之一:
modified(修改的), exclusive(独占的), shared(共享的), invalid(无效的)

* cache 冲突:
假设 cache 有512条 cache line , cache line 大小 64B, 总容量32KB
    1. 采用直接映射时, 则所有 32K 倍增的地址都会映射到同一条线上, 极端情况下程序的内存组织不当, 交替访问 32K，64K，96K等地址时 cache 相当于仅有 line-0 一直忙于冲突状态, 而其他 cache line 一直空闲
    2. 采用全相连映射时, 没有线性关系限制, 因此会逐渐占满所有 cache line, 之后再会出现冲突
    3. 采用组相连映射时, 以4路组相连为例, 则每路 128 条 cache line, 一路大小8K, 则首先根据直接映射分组, 所有 8K 倍增地址都映射到同一组, 然后组内全相连可以同时有 4 路数组, 只有访问第5个这类地址时才会发生冲突, 因此极端情况下数组元素大小为 8K, 则访问超过5个不同元素时就会发生组冲突, 都在竞争第0组 cache line, 相当于 cache 仅有组0的4个 cache line 有效

* 关于页面大小和 cache 页大小
page 是 os 的概念, 而 cache 是 cpu 的概念
虚拟地址和物理地址以 page 为单位进行操作的, 由 vfn 和 offset 组成
而 CPU 访存时 cache 角度内存是以 cache 页大小划分的

* cache RAM 一致性维护
读操作先访问 cache, 如果未命中则加载内存数据同时同步到 cache
	1. 写透
写操作先写 cache 再写 RAM
	2. 写回/写缓冲
* cache 替换算法
LRU(最近最少使用替换)

### 流水线
流水线是MIPS设计的初衷之一,即RISC(局域性原理)
MIPS指令分成五个阶段流水线，每阶段通常只占一个或者半个时钟周期,MIPS的五段流水线一共占四个时钟周期
IF->RD->ALU->MEM->WB

流水线效应
MIPS采用了高度的流水线，其中一个最重要的效应就是分支延迟效应。
分支延迟槽：在分支跳转语句后面的那条语句
实际程序执行到分支语句时，当它刚把要跳转到的地址填充好(填充到代码计数器里)，
还没有完成本条指令时，分支语句后面的那个指令就已经执行了，其原因就是流水线效
应 ---- 几条指令同时执行，只是处于不同的阶段。
流水线效应：
mov $a0, $s2
jalr strrchr
move $a0, $s0
在执行第2行跳转分支时，第3行的move指令已经执行完了。因此，在上面指令序列中，
strrchr函数的参数来自第3行的$s0,而不是第1行的$s2。
从流水线效应中可以看出，是否正确理解MIPS指令的这些特点会直接影响我们对MIPS程
序逆向分析的结果，因此，我们需要熟悉把握这些特点。

### mips-loongson ip核
GS132 单发射32bit
GS464E 四发射64bit增强型
GS464V 四发射64bit向量扩展

HT(Hyper transport) 处理器多路级联 NUMA(非对称内存访问),跨HT桥访问RAM
对称多处理器(SMP)

init 10 DOS 模式下直接写屏
微码(microcode)
CPU 级补丁,可借助 SMM(System Managment Mode) 中断修正指令集错误,在 BIOS 软件给出正确的执行结果, 因此每次BIOS开机的时候都会更新CPU Microcode

系统调用/上下文切换
向量化(SIMD)


# 内存管理
Linux 启动过程内存初始化信息
[    0.000000] Memory: 3789320k/4915200k available (6243k kernel code, 795332k absent, 330548k reserved, 4180k data, 1604k init)

* Zone(内存页面管理区, 内存区域)
根据页面物理地址划分, 内核在初始化内存管理区时, 首先建立管理区表zone_table
ZONE_DMA		0 ~ 16M		兼容 24 bit 旧设备执行 DMA 操作
ZONE_DMA32		16M ~ 4G	兼容 32 bit PCI 设备地址
ZONE_NOMAL		线性地址, 能正常映射的页
ZONE_HIGHMEM	高端内存, 物理地址超过线性地址(MIPS32 线性地址为低512M,KSEG0,KSEG1), 线性地址以外的页不能永久映射到内核空间


```c
zone_t *zone_table[MAX_NR_ZONES*MAX_NR_NODES];
EXPORT_SYMBOL(zone_table);
```

* 内存页 page
通常页大小为4k(4096Byte), 每个物理的页由一个 struct page 的数据结构对象来描述
页的数据结构对象都保存在mem_map全局数组中，该数组通常被存放在ZONE_NORMAL的首部

## 进程虚拟内存地址空间
* mm_struct
每个用户态进程(task_struct)都拥有一个内存描述数据结构(mm_struct), 用来管理该进程的虚拟内存, 如用户态栈区间，堆区间的地址和大小等, 该进程的所有的线程都是共享这一个 mm_struct
elf 文件是按段加载的, 每个段就对应 mm_struct 中的一个 vm_area_struct, 他们通过链表(mmap)按地址大小顺序插入一颗红黑树(mm_rb)中进行管理
注意内核态线程是没有用户态虚拟地址空间的, 所以其 mm_struct 字段为 NULL
* vm_area_struct
vma 中的 [vm_start, vm_end)表示一块虚拟内存空间, vm_page_prot 表示内存访问权限
在一个 mm_struct 中，所有的 vma 通过两种方式管理, 一个是双向链表, 一个是红黑树
当遍历这个虚拟地址空间时访问双向链表, 当在虚拟地址空间查找 vma 时访问红黑树查找

* 缺页异常(Page Fault)
第一次访问虚拟地址时会发生缺页异常, 然后在异常处理函数中调用伙伴系统分配物理内存页, 然后更新页表简历映射关系

## 伙伴分配器

## slab分配器
slab slub slob

### slab
cat /proc/slabinfo
1. kmem_cache
struct kmem_cache;
cache_chain 上挂着系统中所有的 kmem_cache, 他们的大小不同
内核初始化时根据 kmalloc_sizes.h 文件中 PAGE_SIZE 的大小初始化好固定大小的 obj 和对应的 kmeme_cache
这些 kmem_cache 称之为通用缓存，提供给kmalloc来使用
kmalloc根据传入size大小来选择合适的kmem_cache，然后从他的array_cache中取出obj
2. object
slab 从 buddy 获取一定大小的内存池, 然后以 object 为单位进行分配, 并缓存在 kmem_cache 实例的 cpu_cache 中, 用户申请内存时, 其实获取的就是一个个object
一旦object缓存耗尽, 就会重新从buddy system申请slab, 并将其拆分成object, 放入内存池
3. cache
slab内存分配器有两种cache, 一个是slab的cache, 一个是object的cache, cache跟硬件cache无关, 是一个纯软件的概念
slab内存分配器从buddy system获取页面后, 会将其加入kmem cache的node节点, 这个就是slab的cache；将slab拆分成多个object, 并将object加入kmem cache的cpu_cache内存池, 这个就是object的cache；可以看到这两种cache实际是对共同的物理页面的两种缓存形式。


## 内存分析工具
cat /proc/meminfo
Linux 系统内存使用状况的主要接口, 我们最常用的”free”、”vmstat”等命令就是通过它获取数据的
procrank
按照内存空间占用的多少进行排序
librank
从共享库角度分析
cat /proc/[pid]/[s]maps(pmap, procmem)
对具体进程进行分析

### asan
libasan(Address Sanitizer)是一个快速的内存错误检测工具。它非常快，只拖慢程序两倍左右(比起Valgrind快多了)。它包括一个编译器instrumentation模块和一个提供malloc()/free()替代项的运行时库。
ASAN（Address-Sanitizier）早先是LLVM中的特性，后被加入GCC 4.8，在GCC 4.9后加入对ARM平台的支持。因此GCC 4.8以上版本使用ASAN时不需要安装第三方库，通过在编译时指定编译CFLAGS即可打开开关。

使用步骤
用-fsanitize=address选项编译和链接你的程序。
用-fno-omit-frame-pointer编译，以得到更容易理解stack trace。
可选择-O1或者更高的优化级别编译
-fno-stack-protector：去使能栈溢出保护
-fno-omit-frame-pointer：去使能栈溢出保护
-fno-var-tracking：默认选项为-fvar-tracking，会导致运行非常慢
-g1：表示最小调试信息，通常debug版本用-g即-g2
CFLAGS += -fno-stack-protector -fno-omit-frame-pointer -fno-var-tracking -g1
gcc -fsanitize=address -fno-omit-frame-pointer -O1 -g use-after-free.c -o use-after-free
运行use-after-fee。如果发现了错误，就会打印出类似下面的信息：
```
==9901==ERROR: AddressSanitizer: heap-use-after-free on address 0x60700000dfb5
at pc 0x45917b bp 0x7fff4490c700 sp 0x7fff4490c6f8
READ of size 1 at 0x60700000dfb5 thread T0
#0 0x45917a in main use-after-free.c:5
#1 0x7fce9f25e76c in __libc_start_main /build/buildd/eglibc-2.15/csu/libc-start.c:226
#2 0x459074 in _start (a.out+0x459074)
```

## MMAP
mmap 本质上相当于 copy_to_user, 即通过将内核地址映射到文件空间来访问
copy_to_user 每次拷贝都要检查指针合法性, 访问非连续物理内存
mmap 在第一次使用时建立进程页表, 后续合法性检查由页表异常完成
mmap
1. mmap 系统调用
asmlinkage long sys_mmap(unsigned long addr, unsigned long len,
						 unsigned long prot, unsigned long flags,
						 unsigned long fd, off_t off)
2. do_mmap_pgoff
unsigned long do_mmap_pgoff(struct file *file, unsigned long addr,
						unsigned long len, unsigned long prot,
						unsigned long flags, unsigned long pgoff,
						unsigned long *populate)
3. 创建映射mmap_region
unsigned long mmap_region(struct file *file, unsigned long addr,
				unsigned long len, vm_flags_t vm_flags, unsigned long pgoff)
4. 将mmap传进来的参数封装成一个vma结构体, 通过
vma->vm_file = get_file(file);
error = file->f_op->mmap(file, vma);
进入具体的驱动实现
int drm_gem_mmap(struct file *filp, struct vm_area_struct *vma)
5. 驱动中需要实现的功能接口
int drm_gem_mmap_obj(struct drm_gem_object *obj, unsigned long obj_size, struct vm_area_struct *vma)
vma->vm_ops = dev->driver->gem_vm_ops;
struct vm_operations_struct {
		void (*open)(struct vm_area_struct * area);
		void (*close)(struct vm_area_struct * area);
		int (*fault)(struct vm_area_struct *vma, struct vm_fault *vmf);
};
6. 获取和 addr 对应的 pte 页表项地址
根据物理页帧号和页表属性信息计算 pte 需要设置的值
设置 pte 页表项数值, 这个时候就在也表中建立好了用户空间到 GEM 对象的映射关系
将新设置的页表数据同步到 tlb 中, 使其立即生效
```c
int vm_insert_pfn(struct vm_area_struct *vma, unsigned long addr, unsigned long pfn)
static int insert_pfn(struct vm_area_struct *vma, unsigned long addr, unsigned long pfn, pgprot_t prot)
{
		struct mm_struct *mm = vma->vm_mm;
		int retval;
		pte_t *pte, entry;
		spinlock_t *ptl;

		retval = -ENOMEM;
		pte = get_locked_pte(mm, addr, &ptl);                                  @1
		if (!pte)
				goto out;
		retval = -EBUSY;
		if (!pte_none(*pte))
				goto out_unlock;

		/* Ok, finally just insert the thing.. */
		entry = pte_mkspecial(pfn_pte(pfn, prot));                              @2
		set_pte_at(mm, addr, pte, entry);                                       @3
		update_mmu_cache(vma, addr, pte); /* XXX: why not for insert_page? */   @4

		retval = 0;
out_unlock:
		pte_unmap_unlock(pte, ptl);
out:
		return retval;
}
```

## ION
 ION是Google的下一代内存管理器，用来支持不同的内存分配机制，如CARVOUT(PMEM)，物理连续内存(kmalloc), 虚拟地址连续但物理不连续内存(vmalloc)， IOMMU等, 用于 Android 平台

用户空间和内核空间都可以使用ION，用户空间是通过/dev/ion来创建client的。

说到client, 顺便看下ION相关比较重要的几个概念。
* Heap
用来表示内存分配的相关信息，包括id, type, name等。用struct ion_heap表示。
* Client
Ion的使用者，用户空间和内核控件要使用ION的buffer,必须先创建一个client,一个client可以有多个buffer，用struct ion_buffer表示。
* Handle
将buffer该抽象出来，可以认为ION用handle来管理buffer，一般用户直接拿到的是handle,而不是buffer。 用struct ion_handle表示。

heap类型：
由于ION可以使用多种memory分配机制，例如物理连续和不连续的，所以ION使用enum ion_heap_type表示。


## CMA
/sys/kernel/debug/cma

### 用途
camera, video_codec 等多媒体设备需要连续的大块内存(MB), 而 kmalloc/alloc_page 不能保证一定会申请到
camera(FIMC), video_codec(MFC) 共享内存, 代替 bootmem 内存分配
三星(CAMIF/FIMC)

### cma-core
cma 首先声明 cma 区域, 然后将内存还给伙伴系统, 设备需要 cma 内存时在 swap 出来
core_initcall(cma_init_reserved_areas);

cma_init_reserved_areas()
	for (cma_area_count)
		cma_create_area()
	//默认的8MB的cma区域保存在dma_contiguous_def_area里边
	dma_contiguous_def_area = cma_get_area(dma_contiguous_def_base);
	bus_register_notifier(&platform_bus_type, &cma_dev_init_nb);

在设备树中 cma_assign_device_from_dt() 函数找到相应的cma区域


start_kernel()
	setup_arch(&command_line);
		arch_mem_init(cmdline_p);
			dma_contiguous_reserve(PFN_PHYS(max_low_pfn));
				dma_contiguous_reserve_area()
					cma_declare_contiguous(base, size, limit, 0, 0, fixed, "reserved", res_cma);

# ELF

## 进程加载 elf 过程
1. 创建进程
创建进程不必多说了，此时会创建如进程标识符、进程优先级之类的信息。注意此时还不涉及到可执行文件。
2. 创建虚拟地址空间
这一步其实应该算在创建进程里面，实际就是创建页表（多级页表），用来与物理内存建立连接，此时这个页表是空的。此时仍然不涉及到可执行文件。
3. 读取可执行文件头，建立虚拟地址空间与可执行文件的映射关系
这个就是关键的一步了。进程开始读取可执行文件头，即ELF文件的头部，此时进程也仅仅读取ELF文件头部，不涉及到其他段。ELF文件头中含有可执行文件各段的起始地址和长度等信息，以及可执行文件入口地址，注意这里"地址"即虚拟内存地址。
4. 设置CPU指令寄存器为可执行文件入口地址
5. 执行，触发缺页中断

整个装载过程也仅仅是读取了ELF头部，仅此而已。因为ELF头部记录了整个可执行文件的节奏，所以根据ELF头部即可建立整个可执行文件的框架。
因此，这一步其实是在建立与磁盘的连接。
举个简单的例子，当发生缺页中断时，操作系统该去哪把缺的页加载到物理内存？这就是这一步的关键之处了。将虚拟内存地址与磁盘地址建立联系，当缺页时即可寻找到对应的磁盘地址，从而加载到物理内存。
CPU在从入口地址取指令时会发生什么。假设入口地址指向.text段首，如图所示为0x8049000；CPU以此虚拟地址查找页表发现该页尚未装载，触发缺页中断。此时操作系统接管，从之前建立的虚拟地址空间与磁盘的映射关系中找到此页在磁盘中的地址；再从此地址读取页，加载到物理内存，缺页中断完毕。CPU重新从入口地址取指令，此时由页表得到物理地址，从内存中得到对应的指令，交给CPU。
随着进程的执行，缺页中断不断出现，磁盘中的可执行文件也逐渐加载到内存中, 可执行文件的数据是按需加载到物理内存的, 缺页中断驱动着进程的执行
