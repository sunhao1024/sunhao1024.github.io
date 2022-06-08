# ASM
LDR R0, [R1]		读值到R0(从R1地址)	读指令
STR R0, [R1] 		写值从R0(向R1地址)	写指令
MOV R0,  R1 		R0=R1 				赋值
MOV R0, #0x100 		R0=0x100 			立即数
LDR R0, =0X12345678 R0=0X12345678 		伪指令

```asm led.s
	.text					//代码段
	.global_start			//入口标号

	_start
		/* 配置GPIO */
		//0x100 -> 0x5600 0050
		ldr R1, =0x56000050
		ldr R0, =0x100
		str R0, [R1]


		/* SET */
		//0x10 -> 0x5600 0054
		ldr R1, =0x5600 0054
		ldr R0, =0x10
		str R0, [R1]


		/* RESET */
		//0x0 -> 0x5600 0054
		ldr R1, =0x5600 0054
		ldr R0, =0x0
		str R0, [R1]

	/* While(1) */
	halt:
		b halt
```

```sh makefile
all:
	arm-linux-gcc -c -o led.o led.s   					//编译
	arm-linux-ld -Ttext 0 led.o -o led.elf 				//链接
	arm-linux-objcopy -O binary -S led.elf led.bin		//得到bin
clean:
	rm *.bin *.o *.elf
```
# Make
make 可以根据文件后缀自动执行相应的编译命令
比如有 a.c 和 b.sh 两个文件, 可以直接使用 make a, make b(去掉文件后缀)
make -f <makefile> 指定 makefile

## gcc
GCC(GNU Compiler Collection), GNU编译器套装, 是一套由 GNU 开发的编程语言编译器
Clang
MSVC

### binutils
一般指 GNU 与编译器一起提供的二进制工具集, 包括链接器ld, 汇编器as, 工具类(addr2line/nm)
* addr2line	将程序地址转换成其所对应的程序源文件及所对应的代码行(函数名)
* nm		列出 elf 文件中的所有符号以及其类型: B/b(.bss), D/d(.data), T/t(.text), R/r(.rdata), U(未定义的外部符号)
* size		列出 elf 文件中各 section 的大小(text/data/bss)
GNU Bintuils 最终和 GCC、GNU Make 一起完成整个构建过程
* strip		去除 elf 文件中的调试信息, 相当于 objcopy --strip-debug
* objdump	反汇编 elf 文件(objdump -d), 查看 section 信息(objdump -h)和符号信息
* objcopy	常用于格式转换(objcopy -O binary a.elf a.bin)
* readelf	显示 elf 文件的相关信息, 包括显示 elf 文件头(readelf -h)

### 编译
单个编译单元(c/cpp文件)编译过程
cpp test.c -o test.i
gcc test.i -o test.s
ar test.s -o test.o
gcc test.o -o test.elf(省略了动态/静态链接的具体过程)
ld -dynamic-linker /lib64/ld-linux-x86_64.so.2 -lc -o test.elf test.o /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o /usr/lib/x86_64-linux-gnu/crtn.o
ld -static -o test.elf test.o -L`gcc --print-file-name=` /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o  /usr/lib/x86_64-linux-gnu/crtn.o --start-group -lc -lgcc -lgcc_eh --end-group

编译单元(.c) -> 预处理文件(.i)
预处理文件(.i) -> 汇编文件(.s)
汇编文件(.s) -> 目标文件(.o)
目标文件(.o) -> 可执行文件(elf)

1. 预处理
预处理器(cpp)完成#指定的预编译指令(替换宏定义/行号和文件名等), 编译器预处理指令, 设置编译器属性
2. 编译
编译器(gcc, 实际为 cc1)完成词法分析、语法分析、语义分析等操作, 生成硬件平台相关的汇编语言
3. 汇编
汇编器(as)将汇编语言翻译成二进制机器码
4. 链接
C运行时所依赖的环境包括 crt1.o, crti.o, crtn.o, 这几个库是在处理 main 函数调用之前和程序退出之后的事情, 是与操作系统相关的
/lib64/ld-linux-x86_64.so.2 是链接器(ld)本身依赖的动态库

./contrib/download_prerequisites
mkdir build
cd build/
./../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

### ar
ar 将目标文件(.o)打包成一个静态库(.a)
ar rc libxxx.a xxx.o
-r 将目标文件插入到静态库中
-c 创建新的静态库文件

## ELF 文件
ELF(Executable Linkable Format)可执行可链接格式文件, 即 Executable File
可执行文件在 Linux 下格式为 ELF, 在 Windows 下为 PE-COFF(Portable Executable Common File Format)

### 对象文件
对象文件(Object files)有三类: 可重定向的对象文件, 可执行的对象文件, 可被共享的对象文件
1. 可重定向文件(.o)Relocatable object file
由汇编器汇编生成的 .o ,后面链接器(link editor)将一个或多个 Relocatable object files 链接处理后生成一个可执行文件(Executable file)或一个可被共享的对象文件(Shared object file).
我们可以用ar工具将众多的.o归档(archive)成.a(静态库文件).内核可加载模块(.ko)文件也是 Relocatable object file.
2. 可执行的对象文件(Executable object file)
常见的应用程序都是Executable object file.在Linux系统里面存在两种可执行的东西,即此处的Executable object file或可执行的脚本(如shell脚本).
3. 可被共享的对象文件(Shared object file)
即所谓的动态库文件,也即 .so 文件.如果拿前面的静态库来生成可执行程序,那每个生成的可执行程序中都会有一份库代码的拷贝.
如果在磁盘中存储就会占用额外的磁盘空间；如果在Linux系统上一起运行也会浪费物理内存.动态库作用过程:
a 链接编辑器(link editor)拿它和其他Relocatable object file以及其他shared object file作为输入,经链接处理后,生存另外的 shared object file 或者 executable file
b 运行时动态链接器/解释器(dynamic linker/interpreter)拿它和一个Executable file以及另外一些 Shared object file 来一起处理,在Linux系统里面创建一个进程映像.
* so文件命名规则被称为SONAME
libname.so.x.y.z
lib是前缀, 这是一个约定俗成的规则。x为主版本号(Major Version), y为次版本号(Minor Version), z为发布版本号(Release Version)
Major Version 表示重大升级, 不同 Major Version 之间的库是不兼容的
Major Version升级后, 或者依赖旧Major Version的程序需要更新代码, 重新编译, 才可以在新的 Major Version 上运行
或者操作系统保留旧Major Version, 使得老程序依然能运行
Minor Version 表示增量更新, 一般是增加了一些新接口, 原来的接口不变
所以, 在 Major Version 相同的情况下, Minor Version 从高到低是兼容的
Release Version表示库的一些bug修复, 性能改进等, 不添加任何新的接口, 不改变原来的接口
4. 核心转储文件(Core Dump File)
进程意外终止时产生的文件, 存储着该进程的内存空间中的内容等信息

nm xxx.elf
查看 elf 文件中的符号
nm 列出的符号常见的有三种:
T: 在本文件中定义的函数, 最常见的
U: 在本文件中被调用, 但没有定义(表明需要其他库支持)
W: 所谓的"弱态"符号, 它们虽然在本文件中被定义, 但是可能被其他库中的同名符号覆盖

ldd xxx.elf
查看 elf 文件依赖的库文件

### ELF 文件结构
ELF 文件主要包括 ELF header, Program header, Section header 以及两种头表指向的数据组成

| ELF 文件(链接视图)    | ELF 文件(执行视图)    |
|----------------------|----------------------|
| ELF header           | ELF header           |
| Section Header Table | Program Header Table |
| Section 1            | Segment 1            |
| Section 2            | Segment 2            |
| ……                   | ……                   |
| Section n            | Segment n            |

硬盘上的 ELF 文件是链接视图形式的, 由 ELF header 和 Section Header Table 以及对应的数组组成, Program header 位于 ELF header 后面, Section Header 位于 ELF 文件的尾部, 段表偏移就等于 elf 头大小:
e_phoff = sizeof(e_ehsize)
整个 ELF 文件大小:
elf_size = e_shoff + e_shnum * sizeof(e_shentsize) + 1
* 链接视图:
链接视图使用 Section Header Table 和各个 Section, 静态链接器(即ld)链接的过程就是将多个目标文件(*.o)相同的节合并到一起, 节的顺序是不固定的而且各个节区的权限(RWX)不同, 因此执行时需要重新加载

在执行视图中数据分为若干个段(Segment), 每个段包含多个同属性(R/W/X)的节(Section)
* 执行视图:
执行视图使用 Program Header Table 和各个 Segment, 动态链接器(也即加载器, Eg. /lib/ld-linux.so.2)解析 ELF 文件并动态链接其他库文件， 然后加载到内存中去(需要把相同权限的节合并到同一个段中)

* ELF 头(Elf64_Ehdr)
ELF 标头的数据结构声明位于系统头文件 elf.h 中, 以64位为例, 主要包含了
变量        | 含义
------------|---------
e_type      | ELF 文件类型
e_entry     | 入口虚拟地址
e_phoff     | 段表偏移
e_shoff     | 节表偏移
e_ehsize    | elf 头大小
e_phentsize | 段表大小
e_phnum     | 段数量
e_shentsize | 节表大小
e_shnum     | 节数量
e_shstrndx  | 节名字符串表
ELF 头定义了使用32位地址还是64位地址, 对于32位和64位二进制文​​件, ELF 头的长度分别为52或64字节, 除了重定位类型稍有区别, 其它大致相同

* 节
节头(Elf64_shdr)包括了所有节区的节头依次排列， 包含的信息有:
变量         | 含义
-------------|--------------
sh_name      | 节名
sh_type      | 节类型
sh_flags     | 节标志
sh_addr      | 节执行虚拟地址
sh_offset    | 节在 elf 文件中的偏移
sh_size      | 节大小
sh_link      |
sh_info      |
sh_addralign |
sh_entsize   |

	* 自定义节
	__attribute__((section(".mysec")))
	将函数放在自定义节区:
	void my_func() __attribute__((section(".mysec")));
	定义一个在 main 之前执行的函数:
	void init_func() __attribute__((constructor));


* 段
段头(Elf64_phdr)
变量         | 含义
-------------|---
sh_name      |
sh_type      |
sh_flags     |
sh_addr      |
sh_offset    |
sh_size      |
sh_link      |
sh_info      |
sh_addralign |
sh_entsize   |

### 加载 ELF 文件
linux内核启动时将ELF格式注册到内核可支持的文件格式链表中, 也就是通过register_binfmt 函数将定义的elf_format结构体添加到链表中
1. 内核态
当我们执行一个可执行程序的时候, 内核会list_for_each_entry遍历所有注册的linux_binfmt对象, 对其调用load_binrary方法来尝试加载, 直到加载成功为止， ELF 文件加载函数即为 load_elf_binary
2. 用户态
解释器(动态链接器ld)首先检查可执行程序所依赖的共享库, 并在需要的时候对其进行加载
ELF 文件有一个特别的节区： .dynamic, 它存放了和动态链接相关的很多信息, 例如动态链接器通过它找到该文件使用的动态链接库。不过, 该信息并未包含动态链接库的绝对路径, 但解释器通过 LD_LIBRARY_PATH 参数可以找到(它类似 Shell 解释器中用于查找可执行文件的 PATH 环境变量, 也是通过冒号分开指定了各个存放库函数的路径)该变量实际上也可以通过/etc/ld.so.conf 文件来指定, 一行对应一个路径名。为了提高查找和加载动态链接库的效率, 系统启动后会通过 ldconfig 工具创建一个库的缓存 /etc/ld.so.cache 。如果用户通过 /etc/ld.so.conf 加入了新的库搜索路径或者是把新库加到某个原有的库目录下, 最好是执行一下 ldconfig 以便刷新缓存。
找到动态链接库后, 就可以将其加载到内存中。

* note 节
linux内核vmlinux的二进制映像包含名为 .notes 的ELF节
其中包含简短的数据, 二进制和文本, 可以轻松读取, 并且可能包含有关内核版本, Xen兼容性等方面的提示。
目的是使读取引导数据非常容易,可能是Xen loader使用该数据
要查看字段的值, 使用“ -n”和“ -x .notes”和“ -p .notes”
readelf -n vmlinux
readelf -p .notes vmlinux
readelf -x .notes vmlinux

* 加载视图
readelf -l test

Elf 文件类型为 DYN (共享目标文件)
Entry point 0x11c0
There are 13 program headers, starting at offset 64
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000950 0x0000000000000950  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x0000000000000f85 0x0000000000000f85  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000002f0 0x00000000000002f0  R      0x1000
  LOAD           0x0000000000002d30 0x0000000000003d30 0x0000000000003d30
                 0x00000000000002e0 0x00000000000002f0  RW     0x1000
  DYNAMIC        0x0000000000002d40 0x0000000000003d40 0x0000000000003d40
                 0x0000000000000220 0x0000000000000220  RW     0x8

 Section to Segment mapping:
   01     .interp
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt
   03     .init .plt .plt.got .plt.sec .text .fini
   04     .rodata .eh_frame_hdr .eh_frame
   05     .init_array .fini_array .dynamic .got .data .bss
   06     .dynamic

上述段中只有 load 类型的是需要加载到内存中的, 根据权限分三个:
1. RE	可执行, 包括 .text .init .fini 等 section
2. R	只读, 包括 .rodate
3. RW	读写, 包括 .data .bss
加载过程是页(4K)对齐的, 因此段映射也可以减少内存浪费
对应进程虚拟空间中的 vma(虚拟内存区域)
cat /proc/<pid>/maps
55efe9698000-55efe9699000 r--p 00000000 103:06 513034 test
55efe9699000-55efe969b000 r-xp 00001000 103:06 513034 test
55efe969b000-55efe969c000 r--p 00003000 103:06 513034 test
55efe969c000-55efe969d000 r--p 00003000 103:06 513034 test
55efe969d000-55efe969e000 rw-p 00004000 103:06 513034 test
第三列就是 VMA 中的 Segment 在文件中的偏移. (VMA 中的 Segment 和可执行文件中的 Segment 其实不是完全对应的)
最后一列是映像文件的路径, 如果没有就表示匿名映射
简化模型为(高地址到低地址):
| 虚拟地址空间 | 权限 |
|--------------|------|
| Kernel space |      |
| stack        | RW   |
| mmap         | RW   |
| heap         | RW   |
| bss          | RW   |
| data         | RW   |
| rodate       | R    |
| text         | RX   |

### brk 系统调用
brk 是传统的分配/释放内存(堆空间)系统调用, 堆空间位于低地址, 由低向高生长
分配内存时, 将数据段(.data)的最高地址指针_edata往高地址扩展
释放内存时, 把_edata向低地址收缩

### mmap 系统调用
mmap 系统调用是在进程堆和栈中间找一块空闲的虚拟内存(Memory Mapping Segment),
mmap 可以进行匿名映射和文件映射

### glibc 内存分配策略
由于 brk 系统调用分配内存是线性增长的, 容易产生内存利用率低(如先分配100M再分配1K, 则由于仅能移动_edata指针, 一旦1K晚于100M释放则前100M无法及时利用)和内存碎片化(反之先分配1K再分配100M)等问题
实际上 brk 就是匿名映射(一块一块分配内存), 所以也可以通过 mmap 的匿名映射分配内存
而 glibc 库中 malloc 函数的实现就是根据分配内存的 size 来决定使用哪个分配函数, 小于等于128KB时调用 brk 分配, 大于128KB时调用 mmap 分配内存
这两种方式分配的都是虚拟内存, 实际上没有分配物理内存. 当第一次访问分配的虚拟地址空间的时候, 再由缺页中断实际分配物理内存, 然后更新页表建立虚拟内存和物理内存之间的映射关系

### 链接库
1. 静态链接库(libxxx.a)
静态库, 由多个.o 链接得到, 用于静态链接

生成静态库
ar -r libxxx.a xxx.o
链接静态库
gcc main.c libxxx.a -o main

列出静态库成员
ar -t libxxx.a
向已存在的静态库追加
ar -q libxxx.a xxx.o xxx.o

2. 动态链接库(libxxx.so)
共享库, 用于动态链接, 相当于 win 上的 xxx.dll

生成 so 库
gcc -shared -fpic xxx.o -o libxxx.so
链接 so 库
gcc xxx.o -lxxx -L./common -o xxx

* 动态链接库的加载顺序
1. 编译目标代码时指定的动态库搜索路径
2. 环境变量LD_LIBRARY_PATH指定的动态库搜索路径
3. 配置文件/etc/ld.so.conf中指定的动态库搜索路径
4. 默认的动态库搜索路径/lib
5. 默认的动态库搜索路径/usr/lib

ldconfig -p
列出当前系统已经安装的所有动态库信息

### tools
1. readelf
readelf -h

readelf -S	链接视图
readelf -l	加载视图

//Forbidden waring
scripts/gcc-wrapper.py
interpret_warning

## Ｍakefile
Makefile 的基本原理是由一系列的目标(target)和目标依赖(prerequisites)以及目标行为(recipe)组成
target: prerequisites
    recipe

目标特定变量
<target>: <var>=xxx

默认 Makefile 会在执行每行目标行为时先打印该行内容(回显), 在该行内容之前加上 '@' 符号可以取消这一行为, 或者在执行 make 时加上 -s(--silent) 选项


* linux 内核 Makefile
直接编译:obj-y      			+= hello.o
条件编译:obj-$(CONFIG_HELLO) 	+= hello.o
最终 Makefile 根据 .config 中的 CONFIG_HELLO 来决定 y/m/n

多文件模块(模块名ext2,编译生成ext2.o和ext2.ko)
obj-$(CONFIG_EXT2_FS)               += ext2.o
ext2-y                              := file.o dir.o fsync.o ioctl.o inode.o
ext2-$(CONFIG_EXT2_FS_XIP)          += xip.o
ext2-$(CONFIG_EXT2_FS_SECURITY)     += security.o
'Note'若为新目录需要在父目录的Ｍakefile中添加   obj-$(CONFIG_EXT2_FS) += ext2/ 以使make进入子目录

makefile预定义变量
预定义变量	作用
AR			库文件维护程序的名称,默认为ar
AS			汇编程序的名称,默认为as
CC			c编译器的名称,默认为gcc
CXX			c++编译器的名称,默认为g++
ARFLAGS		库文件维护程序选项,无默认值
ASFLAGS		汇编程序选项,无默认值
CFLAGS		c编译器选项,无默认值
CXXFLAGS	c++编译器选项,无默认值
$(MAKE)				当前make解释器的文件名
$(MAKECMDGOALS)		命令行中指定的目标名(make的命令行参数）
$(MAKEFILE_LIST)	make所需要处理的makefile文件列表, 当前makefile的文件名总是位于列表的最后, 文件名之间以空格进行分隔
$(MAKE_VERSION)		当前make解释器的版本
$(CURDIR)			当前make解释器的工作目录
$(.VARIABLES)		所有已经定义的变量名列表(预定义变量和自定义变量）

自动变量		作用
$@			目标文件的完整名称
$?			所有时间戳比目标文件晚的依赖文件
$*			不包含扩展名的目标文件名称
$<			第一个依赖文件名称
$^			所有不重复的依赖文件
$(MAKE)		进入子目录递归 makefile
% 为 Makefile 的规则通配符, 允许对文件名进行类似正则运算的匹配

`$$var` 引用 shell 的变量
`include <makefile>` 引用其他 makefile

赋值
= 是最基本的赋值,
:= 是覆盖之前的值
?= 是如果没有被赋值过就赋予等号后面的值
+= 是添加等号后面的值

makefile环境变量
普通变量导出以后即为环境变量,环境变量类似于工程中所有makefile之间共享的全局变量
一般要求环境变量大写,普通变量小写,使用export变量名进行导出
执行make命令的传参操作也相当于传入了一个环境变量(优先级最高,可以覆盖原来makefile文件中定义的变量值)
makefile隐式规则
自动寻找.o文件对应的同名.c文件
不用生成.o文件的规则,指定.o文件以后,会自动将同名.c文件进行编译
在uboot以及linux kernel中经常出现include ···config.mk
这也相当于包含一个子makefile,虽然文件名不像,但可以把它当作一个makefile文件来看待,二者基本没有区别

引用其他makefile及makefile嵌套
include makefile文件名
相当于子makefile文件直接展开
嵌套:
subsystem:
cd subdir && $(MAKE)
等价于:
subsystem:
$(MAKE) -C subdir

输出信息
$(info "xxx")
$(warning  $(XXX))
$(error "xxx")
info 信息不打印信息所在行号, error 信息停止 make

@+命令  不输出命令, 只输出命令结果
@echo "xxx"(只能作为 target 后面的语句, 以 tab 开头)

make -n 仅显示命令而不执行 --just-print
make -s 禁止显示所有命令   --silent
MAKEFLAGS+= --no-print-directory 全局标志

PHONY += clean
.PHONY
clean
make 时一定执行 clean 目标

$(shell ls) 调用 shell 命令

### makefile 函数
`自定义函数`
makefile中不支持真正意义上的自定义函数, 其本质是多行变量
define <func_name>
	@echo "argv[0] = $(0)"
endef

$(call <func_name>,[arg1,arg2,...])
$(0) 函数名, $(n) 第n各参数

`内部函数`
VALUE = $(<func_name> [arg1,arg2,...])

1. wildcard	通配符查找
$(wildcard <pattern>)

2. patsubst	通配符替换
$(patsubst <pattern>, <replacement>, <text>)

3. addprefix 添加前缀
$(addprefix <prefix>, <text>)

4. notdir		去除路径

5. foreach		循环 <list> 赋给 <i>, 然后执行并返回 <text>
$(foreach <i>,<list>,<text>)
每次循环 <text> 返回的字符串以空格分隔, 最后组成的整个字符串做为 foreach 函数的返回值
obj = $(foreach n,$(file_list),$(n).o)

6. filter		模式保留过滤(filter-out 剔除)
VALUE = $(filter <pattern>,<text>)
VALUE = $(filter-out <pattern>,<text>)

7. word			取出第 n 个字符串(words 统计单词数)
$(word <n>,<text>)
$(words, <text>)统计列表中的单词数

8. sort			排序(升序)
$(sort <list>)

9. strip		去除字符串开头和结尾的空格, 并且将多个连续的空格合并成为一个空格
$(strip <string>)

10. basename	返回文件名称中不含后缀的部分
$(basename <filename>)


```sh
SRC = $(wildcard *.c ./src/*.c)

SOURCES := $(wildcard [0-9]*x[0-9]*.s)
BIN	:= $(patsubst %.S, %.bin, $(SOURCES))
```

obj 文件除使用 patsubst 还可以使用另一种过滤方法
obj = $(src:.c=.o)

* 条件分支
```sh
ifdef DEBUG
	CFLAGS += -g
endif

ifndef bar #这里使用变量名判断是否定义
    var = 123
else
    var = $(bar)
endif

ifeq ($(DEBUG), 1)
    OPTS= -O0 -g
else
    OPTS = -O2
endif

```

### 全局宏
1. makefile 中增加宏定义 -D
ccflags-y := -Iinclude/drm -DCONFIG_SHINANO_LPM=y

2. EXTRA_CFLAGS
```sh
config USE_POWER_MANAGEMENT
	int "USE_POWER_MANAGEMENT"
	depends on VIVANTE
	default 0
```

```sh
ifeq ($(CONFIG_USE_POWER_MANAGEMENT), 1)
EXTRA_CFLAGS += -DgcdPOWER_MANAGEMENT=1
else
EXTRA_CFLAGS += -DgcdPOWER_MANAGEMENT=0
endif
```
### Makefile例子
```bash
COMPILER ?= gcc
OUT = c.elf

obj += demo_c.o
# obj += temp_c.o
# deps =

all:$(obj)
	$(COMPILER) -o $(OUT) $(obj)

# %.o:%.c $(deps)
%.o:%.c
	$(COMPILER) -c $< -o $@

clean:
	rm -rf $(obj) $(OUT)
```

## pkg-config
用于获得某一个库/模块的所有编译相关的信息
CFLAGS=`pkg-config --cflags libdrm`
LIBS=`pkg-config --cflags --libs libdrm`

pkg-config 在 PKG_CONFIG_LIBDIR/PKG_CONFIG_PATH 指定的路径利用已安装库的 .pc 文件获取相关信息
pkg-config --variable pc_path pkg-config
使用指定的外部库(PKG_CONFIG_LIBDIR 优先级更高)
export PKG_CONFIG_LIBDIR=xxx

## linux内核编译
1. 遍历各源码目录下的Makefile
2. 根据Kconfig定制要编译的对象
3. 回到顶层目录的Makeifle执行编译
顶层目录下的.config保存配置

### menuconfig Kconfig
menuconfig 就是通过调用各级目录下 Kconfig 文件来形成图形界面的

* menuconfig 配置文件
1. 合并多个配置文件
scripts/kconfig/merge_config.sh 工具可以合并多个配置文件
  -h    display this help text
  -m    only merge the fragments, do not execute the make command
  -n    use allnoconfig instead of alldefconfig
  -r    list redundant entries when merging fragments
  -y    make builtin have precedence over modules
  -O    dir to put generated output files.  Consider setting $KCONFIG_CONFIG instead.

merge_config.sh -m -r <合入的目标配置文件> <额外的配置文件>
2. 生成新的默认配置文件
make menuconfig 修改内核配置
make ARCH=$ARCH savedefconfig 命令会生成 ./defconfig 文件
不能直接修改 .config 或者将 .config 复制为 xxx_defconfig
在 make xxx_defconfig 时会自生成公共配置项,
cp ./defconfig arch/xxx/configs/xxx_defconfig

为什么不能直接修改 .config 并复制到 arch/xxx/configs/xxx_defconfig:
arch/xxx/configs/xxx_defconfig 并不是完整的 config 文件, 一些通用的内容并不会保存在其中(可以节省空间)

* kconfig 语法
1. 菜单入口
mainmenu "主菜单名字"    最顶层的菜单
2. 多选菜单(带配置项本身不可配置,属性只有依赖和可见)
menu "string"
	... ...
endmenu
3. 可选菜单(带配置项本身可配置,关键字前缀CONFIG_后就构成了“.config”中的配置项名字)
menuconfig 配置关键字
4. 配置项(关键字前缀CONFIG_后就构成了“.config”中的配置项名字,但不是配置界面显示的字符,配置界面显示的是提示字符)
config 配置关键字
5. 配置项显示字符(bool[Y,N]tristate<Y,N,M>后的字符串是该配置项在menuconfig中显示的内容 )
bool "string"
tristate "string"
6. 引入下一级Kconfig
source "...dir/Kconfig"

```sh
config DRM_LSDC
depends on DRM
select I2C_GPIO if (DRM_LSDC_PLATFORM_DRIVER && CPU_LOONGSON2K)
default m
help
```


#### OPTION `local version`
1. Local version - append to kernel release
字符串变量,修改后会追加在 kernel release 后面
2. Automatically append version information to the version string
3. Build ID Salt
是编译器/链接器的--build-id选项, 可将唯一的Build ID号添加到生成的二进制ELF文件(如vmlinux)的 .note 段
盐是随机数据, 用作哈希数据, 密码或密码的单向函数的附加输入,构建ID的计算中使用该值
Build ID对于想要确保内部版本在各个版本之间是唯一的发行版非常有用
取消Build ID, 例如生成Same Kernel构建, 可以编辑主Makefile并更改--build-id=none
```bash
sed -i Makefile  -e 's/--build-id/--build-id=none/g'
```
4. localversion
根目录下的文件

make kernelversion
make kernelrelease
./scripts/setlocalversion
cat include/config/kernel.release
include/generated/utsrelease.h
define UTS_RELEASE "5.9.0-rc3+"
modinfo可查看编译出来的ko文件对应的内核版本号
uname或者 cat/proc/version 可在目标系统上查看内核版本号


kernelversion 后缀添加规则
* 涉及的makefile和配置文件
1. arch/mips/Makefile
KERNELRELEASE
可用来区分用户空间和内核空间
ifneq ($(KERNELRELEASE),)
```Makefile
# Read KERNELRELEASE from include/config/kernel.release (if it exists)
KERNELRELEASE = $(shell cat include/config/kernel.release 2> /dev/null)
KERNELVERSION = $(VERSION)$(if $(PATCHLEVEL),.$(PATCHLEVEL)$(if $(SUBLEVEL),.$(SUBLEVEL)))$(EXTRAVERSION)
export VERSION PATCHLEVEL SUBLEVEL KERNELRELEASE KERNELVERSION

install:
  $(Q)install -D -m 755 vmlinux $(INSTALL_PATH)/vmlinux-$(KERNELRELEASE)
ifdef CONFIG_SYS_SUPPORTS_ZBOOT
  $(Q)install -D -m 755 vmlinuz $(INSTALL_PATH)/vmlinuz-$(KERNELRELEASE)
endif
  $(Q)install -D -m 644 .config $(INSTALL_PATH)/config-$(KERNELRELEASE)
  $(Q)install -D -m 644 System.map $(INSTALL_PATH)/System.map-$(KERNELRELEASE)
```

2. Makefile
```makefile
# SPDX-License-Identifier: GPL-2.0
VERSION = 5
PATCHLEVEL = 9
SUBLEVEL = 0
EXTRAVERSION = -rc3
NAME = Kleptomaniac Octopus

# Read KERNELRELEASE from include/config/kernel.release (if it exists)
KERNELRELEASE = $(shell cat include/config/kernel.release 2> /dev/null)
KERNELVERSION = $(VERSION)$(if $(PATCHLEVEL),.$(PATCHLEVEL)$(if $(SUBLEVEL),.$(SUBLEVEL)))$(EXTRAVERSION)
export VERSION PATCHLEVEL SUBLEVEL KERNELRELEASE KERNELVERSION

#
# INSTALL_PATH specifies where to place the updated kernel and system map
# images. Default is /boot, but you can set it to other values
export  INSTALL_PATH ?= /boot
```

3. include/config/kernel.release
保存了自动生成的 kernel release 字符串
./scripts/setlocalversion
cat include/config/kernel.release
Makefile
```makefile
# Store (new) KERNELRELEASE string in include/config/kernel.release
include/config/kernel.release: FORCE
  $(call filechk,kernel.release)
```

4. scripts/setlocalversion
```bash
# CONFIG_LOCALVERSION and LOCALVERSION (if set)
res="${res}${CONFIG_LOCALVERSION}${LOCALVERSION}"

# scm version string if not at a tagged commit
if test "$CONFIG_LOCALVERSION_AUTO" = "y"; then
  # full scm version string
  res="$res$(scm_version)"
else
  # append a plus sign if the repository is not in a clean
  # annotated or signed tagged state (as git describe only
  # looks at signed or annotated tags - git tag -a/-s) and
  # LOCALVERSION= is not specified
  if test "${LOCALVERSION+set}" != "set"; then
	scm=$(scm_version --short)
	res="$res${scm:++}"
  fi
fi
```
1. 如果定义了CONFIG_LOCALVERSION_AUTO(CONFIG_LOCALVERSION_AUTO=y）
此时会执行第一个if
res="$res$(scm_version)"
如果代码属于git管理：
打了tag, 则会添加tag相关字符；
没有打tag, 则会添加log相加字符, 例如最新的commit是
commit cdebe039ded3e7fcd00c6e5603a878b14d7e564e
则编译之后文件include/config/kernel.release的内容为2.6.35.7-gcdebe03

2. 如果没有定义了CONFIG_LOCALVERSION_AUTO。
此时会执行else
	如果没有定义LOCALVERSION, 版本号后面会添加“+”号：执行else里的if下的脚本scm=$(scm_version --short), 在函数scm_version --short里, 如果传入参数short会添加“+”号,
```bash
   if $short; then
	echo "+"
	return
   fi
```
定义了LOCALVERSION则不会执行else里if所在的脚本, 从而不会在后面添加“+”号。
LOCALVERSION变量可在命令行定义,或者添加为环境变量
make LOCALVERSION=.88 include/config/kernel.release
如果既不想添加字符, 又不想有“+”号：不定义CONFIG_LOCALVERSION_AUTO, 将LOCALVERSION变量定义为空：LOCALVERSION=

3. 往版本号里添加字符的方式
在scripts/setlocalversion文件中还有有这么一段：
```bash
# localversion* files in the build and source directory
res="$(collect_files localversion*)"
if test ! "$srctree" -ef .; then
 res="$res$(collect_files "$srctree"/localversion*)"
fi

# CONFIG_LOCALVERSION and LOCALVERSION (if set)
res="${res}${CONFIG_LOCALVERSION}${LOCALVERSION}"
```
由此可看出, 如果想往版本号里添加字符, 有几种方式：
* 使用LOCALVERSION变量(或者在命令行, 或者添加为环境变量）
* 在linux-2.6.35目录下添加文件localversion, 文件内容会自动添加到版本号里去。
* 定义CONFIG_LOCALVERSION变量
* 如果linux-2.6.35目录下有 localversion 文件, 也使用了LOCALVERSION变量, 也定义了CONFIG_LOCALVERSION=".XYZ"。

echo "loongson" > localversion-next
make LOCALVERSION=-drm
最终 make install 时的版本号后缀包括三部分
* localversion 文件的内容
* CONFIG_LOCALVERSION 的值
* LOCALVERSION 的值

4. 关于scripts/setlocalversion文件
在scripts/setlocalversion文件中, 可用echo "aaa" >&2来输出显示相关信息, 例如：
echo "LOCALVERSION=${LOCALVERSION}" >&2
这个文件里很多地方是跟根据一些git命令来进行判断的, 例如
```bash
# Check for git and a git repo.
if test -z "$(git rev-parse --show-cdup 2>/dev/null)" &&
   head=$(git rev-parse --verify --short HEAD 2>/dev/null); then

  # If we are at a tagged commit (like "v2.6.30-rc6"), we ignore
  # it, because this version is defined in the top level Makefile.
  if [ -z "$(git describe --exact-match 2>/dev/null)" ]; then

	# If only the short version is requested, don't bother
	# running further git commands
	if $short; then
	  echo "+"
	  return
	fi
	# If we are past a tagged commit (like
	# "v2.6.30-rc5-302-g72357d5"), we pretty print it.
	if atag="$(git describe 2>/dev/null)"; then
	  echo "$atag" | awk -F- '{printf("-%05d-%s", $(NF-1),$(NF))}'

	# If we don't have a tag at all we print -g{commitish}.
	else
	  printf '%s%s' -g $head
	fi
  fi

  # Is this git on svn?
  if git config --get svn-remote.svn.url >/dev/null; then
	printf -- '-svn%s' "$(git svn find-rev $head)"
  fi

  # Check for uncommitted changes.
  # First, with git-status, but --no-optional-locks is only
  # supported in git >= 2.14, so fall back to git-diff-index if
  # it fails. Note that git-diff-index does not refresh the
  # index, so it may give misleading results. See
  # git-update-index(1), git-diff-index(1), and git-status(1).
  if {
	git --no-optional-locks status -uno --porcelain 2>/dev/null ||
	git diff-index --name-only HEAD
  } | grep -qvE '^(.. )?scripts/package'; then
	printf '%s' -dirty
  fi

  # All done with git
  return
fi

```

5. 关于内核版本的其他问题
./scripts/setlocalversion
make kernelversion
make kernelrelease
使用modinfo可查看编译出来的ko文件对应的内核版本号
使用uname或者 cat/proc/version 可在目标系统上查看内核版本号。
include/generated/utsrelease.h
#define UTS_RELEASE "5.9.0-rc3-next-20200903-drm"
可查看kernel编译过程生成的文件确定编译出来的内核的版本号。

6. 编译环境信息
include/generated/compile.h

7. 去除内核版本后缀
lib/modules/xxx-yyy
yyy 为 git commit sha1 后缀
echo "" > .scmversion

#### OPTION `local host`
主机名
Default hostname
CONFIG_DEFAULT_HOSTNAME
Defined at init/Kconfig:314

本地修改
hostname xxx 	临时修改, 重启服务器后就不生效了

hostnamectl set-hostname xxx
vim /etc/hostname

#### OPTION `编译环境信息`
include/generated/compile.h

### 编译内核
make -j9 编译生成内核镜像

1. make install
编译出来的压缩内核镜像拷贝到/boot/文件夹下,修改/boot/grub/grub.cfg配置文件,才能出现启动时我们看到的选项。
make install命令在grub.cfg文件中增加了一个submenu段里面注明了新内核名字,启动镜像位置,根目录所在磁盘的uuid等信息
/boot/vmlinux-4.19.73+
/boot/vmlinuz-4.19.73+
/boot/config-4.19.73+
/boot/System.map-4.19.73+

2. make headers_install
3. make modules_install

/lib/modules/liunx-x.xx.x
(/dev/sda1 /boot)

4. 修改 bootloader 引导启动新内核

grub 引导项
vim /etc/grub2.cfg  grub2的配置文件
vim /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg

pmon 引导
/boot/boot.cfg
kernel (wd0,0)/vmlinuz

uname -r 重启后,查看是否生效



make htmldocs
Documentation/Changes中有该版本内核编译所依赖的工具以及最低版本信息列表
scripts/ver_linux可以快速获取当前主机上各个工具以及当前版本的信息
make list-defconfigs 列出所有支持的 defconfig
make menuconfig 图形化的内核配置
make mrproper 删除不必要的文件和目录
make xconfig 基于QT的qconf (qttools5-dev)
make gconfig 基于GTK的gconf(libglade2-dev)
make bzImage — 编译基本的内核(make menuconfig这一步中选*的部分）,并制成压缩镜像



make deb-pkg

交叉编译工具链
export PATH=$PATH:/opt/mips-loongson-gcc7.3-linux-gnu/2019.06-29/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/mips-loongson-gcc7.3-linux-gnu/2019.06-29/lib

make moudles
编译内核模块(make menuconfig这一步选择m的部分)
make modules_install
将上一步编译好的模块拷贝到/lib/modules/liunx-x.xx.x,modprobe会在路径

模块编译
make modules M=<module_path> ARCH=<arch> CROSS_COMPILE=<cc> CONFIG_XXX=m
make modules_install M=<module_path> INSTALL_MOD_PATH=<install_path>

Eg.
make modules M=drivers/gpu/drm/lsdc ARCH=mips CROSS_COMPILE=mips-linux-gnu- CONFIG_DRM_LOONGSON=m
make modules_install M=drivers/gpu/drm/lsdc INSTALL_MOD_PATH=~/Downloads/ ARCH=mips CROSS_COMPILE=mips-linux-gnu- CONFIG_DRM_LOONGSON=m

非内核源码路径需要额外指定内核路径
make modules M=<module_path> -C <kernel_src_path>
编译完成后复制到 /lib/modules/<kernel_version> , 然后 depmod -a

ifneq ($(KERNELRELEASE),)
obj-m := mytest.o
mytest-objs := file1.o file2.o file3.o
else
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)
default:
		$(MAKE) -C $(KDIR) M=$(PWD) modules
endif








清理
make clean < make mrproper < make distclean
1)make clean:清理编译生成的绝大多数文件,但会保留config,及编译外部模块所需要的文件
2)make mrproper:清理编译生成的所有文件,包括配置生成的config文件及某些备份文件
3)make distclean:相当于mrproper,额外清理各种patches以及编辑器备份文件

### 其他内核编译工具
1. Sparse
内核代码静态分析工具, 能够帮助我们找出代码中的隐患
Sparse通过 gcc 的扩展属性 __attribute__ 以及自己定义的 __context__ 来对代码进行静态检查.

`检查简单的源文件`
sparse -a sparse_test.c
`检查内核代码`
make C=1 检查所有重新编译的文件
make C=2 检查需要重新编译的文件
make C=2 <dir> 检查指定目录的文件
W=1

## Android 编译
### 内核空间编译规则
1. makefile
```
KVERS =$(shell uname -r)
#Flags for module debug compilation infomation
#EXTRA_CFLAGS =-g -O0
build: XXX_modules
XXX_modules:
	obj-$(CONFIG_SPI_SPIDEV_TEST)       += spiexpeed.o
```
2. Kconfig
linux内核中添加文件夹后要在这个文件夹下创建一个Kconfig文件,然后在上层目录的Kconfig中source这个文件夹下的Kconfig文件
config SPI_SPIDEV_TEST
	tristate "llll"
3. sdm845_defconfig
CONFIG_SPI_SPIDEV_TEST=y
LINUX/android/kernel/msm-4.9/arch/arm64/configs/sdm845_defconfig
依赖于LINUX/android/device/qcom/aikit/AndroidBoard.mk(确定板相关配置sdm845_defconfig)
CONFIG_SERIAL_MSM_GENI_CONSOLE=y(串口log配置项)

### 用户空间编译(NDK-build)
修改Android.mk或Android.bp

LOCAL_PATH:= $(call my-dir)             #返回当前路径
include $(CLEAR_VARS)                   #清除定义的变量
LOCAL_MODULE := JniDemo                 #生成模块的名字
LOCAL_SRC_FILES := test.call            #需要编译的源文件
LOCAL_MODULE_TAGS := optional           #编译版本选项(user eng tests optional所有)
include $(BUILD_SHARED_LIBRARY)         #编译输出(BUILD_STATIC_LIBRARY静态库,BUILD_SHARED_LIBRARY动态库,BUILD_EXECUTABLE Native C .exe)
LOCAL_LDLIBS := -llog                   #编译所需要的库
LOCAL_SHARED_LIBRARIES := myadd         #编译所需要的静态库

mm编译输出
out/target/product/aikit/system/bin/expeed_spiautotest

编译依赖
1.  envsetup.sh
command make --no-print-directory -f build/core/config.mk dumpvar-abs-$1)
local TOPFILE=build/core/envsetup.mk
2. 1 config.mk
include $(BUILD_SYSTEM)/envsetup.mk
2. 2 envsetup.mk
board_config_mk := \
$(strip $(sort $(wildcard \
	$(SRC_TARGET_DIR)/board/$(TARGET_DEVICE)/BoardConfig.mk \
	$(shell test -d device && find -L device -maxdepth 4 -path '*/$(TARGET_DEVICE)/BoardConfig.mk') \
	$(shell test -d vendor && find -L vendor -maxdepth 4 -path '*/$(TARGET_DEVICE)/BoardConfig.mk') \
)))
//获取到对应的BoardConfig.mk文件,可存在于$(SRC_TARGET_DIR)/board/$(TARGET_DEVICE)或vendor/*/$(TARGET_DEVICE)之一
TARGET_DEVICE_DIR := $(patsubst %/,%,$(dir $(board_config_mk))) //获取对应的路径
include $(board_config_mk)
matthew@matthew:/media/matt2/8937-evb-1/build/target/board$ grep -r "TARGET_DEVICE_DIR" ./
./Android.mk:-include $(TARGET_DEVICE_DIR)/AndroidBoard.mk //加载对应的AndroidBoard.mk
lunch函数会到venor和device目录中寻找vendorsetup.sh
envsetup.sh -> config.mk - envsetup.mk -> $TARGET_DEVICE_DIR/AndroidBoard.mk
vendor/qcom/AIkit/AndroidBoard.mk
ifeq ($(KERNEL_DEFCONFIG),)
	ifeq ($(TARGET_BUILD_VARIANT),user)
	 KERNEL_DEFCONFIG := aikit-perf_defconfig
	else
	 KERNEL_DEFCONFIG := aikit_defconfig
	endif
endif


### Build Android Image
cd LINUX/android/
1. source build/envsetup.sh
2. lunch aikit-userdebug
3. make bootimage -j4

source 	加载系统编译命令envsetup.sh 厂商自定义vendorsetup.sh
lunch 	选择编译版本(Name-LOCAL_MODULE_TAGS,eng工程机,user最终用户机,userdebug调试测试机,tests测试机)
main.mk里有说明,在Android的源码里,每一个目标(也可以看成工程)目录都有一个Android.mk的makefile,每个目标的Android.mk中有一个类型声明:LOCAL_MODULE_TAGS
PS:Android.mk和Linux里的makefile不太一样,它是Android编译系统自己定义的一个makefile来方便编译成:c,c++的动态、静态库或可执行程序,或java库或android的程序,


### 定制产品流程
1. 创建vendor目录
	#mkdir vendor/farsight
2. 创建一个vendorsetup.sh文件,将当前产品编译项添加到lunch里,让lunch能找到用户个性定制编译项
	#echo "add_lunch_combo fs100-eng" > vendor/farsight/vendorsetup.sh
3. 仿着Android示例代码,在公司目录下创建products目录
	#mkdir -p vendor/farsight/products
4. 仿着Android示例代码,在products目录下创建两个mk文件
	#touch vendor/farsight/products/AndroidProduct.mk vendor/farsight/products/fs100.mk
	在AndroidProduct.mk里添加如下内容:
	PRODUCT_MAKEFILES := $(LOCAL_DIR)/fs100.mk
	表示只有一个产品fs100,它对应的配置文件在当前目录下的fs100.mk。
5. 在产品配置文件里添加最基本信息
	PRODUCT_PACKAGES := \
	IM \
	VoiceDialer
	$(call inherit-product, build/target/product/generic.mk)  ##从某一默认配置开始派生余下内容参考派生起点
	#Overrides
	PRODUCT_MANUFACTURER := farsight
	PRODUCT_NAME := fs100
	PRODUCT_DEVICE := fs100

# 构建(Build)

## CMake
cmake -S<src_path> -B<build_path> -G <build_system> -H<home_path>
cmake -S/home/sunhao/work/ubus/libubus -B/home/sunhao/work/ubus/libubus/build -G "Unix Makefiles" --no-warn-unused-cli -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Debug -DCMAKE_C_COMPILER:FILEPATH=/usr/bin/i686-linux-gnu-gcc-10 -DCMAKE_CXX_COMPILER:FILEPATH=/usr/bin/i686-linux-gnu-g++-10

* 交叉编译时用 CMAKE_TOOLCHAIN_FILE 指定交叉编译相关变量
cmake -DCMAKE_TOOLCHAIN_FILE=../arm_linux_toolchain.cmake

arm_linux_toolchain.cmake
```sh
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm64)

set(toolchain /home/sunhao/work/press_temp/RK3566-SDK/buildroot/output/rockchip_rk3566/host)
set(CMAKE_C_COMPILER ${toolchain}/bin/aarch64-linux-gcc)
set(CMAKE_CXX_COMPILER ${toolchain}/bin/aarch64-linux-g++)
```

cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=on	开启生成 compile_commands.json

### set/unset
设置普通变量, 缓存变量, 环境变量(字符串类型值,区分大小写)
```
set(<variable> <value>... [PARENT_SCOPE])
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
set(ENV{<variable>} [<value>])
```

引用变量
```
${<variable>}
$ENV{<variable>}
$CACHE{<variable>}
```
作用域
局部变量function()内部
目录变量非function()内部
缓存变量构建系统
环境变量进程作用域

### list
```
set(srcs a.c b.c c.c) # sets "srcs" to "a.c;b.c;c.c"
set(x a "b;c") # sets "x" to "a;b;c", not "a;b\;c"
list(LENGTH <list><output variable>)
list(GET <list> <elementindex> [<element index> ...]<output variable>)
list(APPEND <list><element> [<element> ...])
list(FIND <list> <value><output variable>)
list(INSERT <list><element_index> <element> [<element> ...])
list(REMOVE_ITEM <list> <value>[<value> ...])
list(REMOVE_AT <list><index> [<index> ...])
list(REMOVE_DUPLICATES <list>)
list(REVERSE <list>)
list(SORT <list>)
```
LENGTH					返回list的长度
GET						返回list中index的element到value中
APPEND					添加新element到list中
FIND					返回list中element的index, 没有找到返回-1
INSERT					将新element插入到list中index的位置
REMOVE_ITEM				从list中删除某个element
REMOVE_AT				从list中删除指定index的element
REMOVE_DUPLICATES		从list中删除重复的element
REVERSE 				将list的内容反转
SORT 					将list按字母顺序排序

### 条件
```
if(<condition>)
  <commands>
elseif(<condition>) # optional block, can be repeated
  <commands>
else()              # optional block
  <commands>
endif()
```
### 循环
foreach(<loop_var> <items>)
  <commands>
endforeach(<loop_var>)
其中<items>是用分号或空格分隔的项目列表
支持break() 和 continue()

### 函数
function(<name> [<arg1> ...])
  <commands>
endfunction()
函数调用不区分大小写function(foo)
foo() Foo() FOO()
cmake_language(CALL foo)

### pkg-config
find_package(PkgConfig)
.pc

`find_package`
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
			 [REQUIRED] [[COMPONENTS] [components...]]
			 [OPTIONAL_COMPONENTS components...]
			 [NO_POLICY_SCOPE])
version指定的是版本, 如果指定就必须检查找到的包的版本是否和version兼容
指定EXACT则表示必须完全匹配的版本而不是兼容版本就可以
QUIET 可选字段, 表示如果查找失败, 不会在屏幕进行输出
如果指定了REQUIRED字段, 找不到的话就立即停掉整个cmake, 输出查找失败提示语
Module模式查找
系统变量
PKG_CONFIG_VERSION_STRING pkg-config的版本
PKG_CONFIG_EXECUTABLE     pkg-config程序的路径名
PKG_CONFIG_FOUND          如果找到pkg-config可执行文件

`pkg_check_modules`
检查所有给定的模块, 在调用范围内设置各种结果变量。
pkg_check_modules(<prefix>
				  [REQUIRED] [QUIET]
				  [NO_CMAKE_PATH]
				  [NO_CMAKE_ENVIRONMENT_PATH]
				  [IMPORTED_TARGET [GLOBAL]]
				  <moduleSpec> [<moduleSpec>...])
pkg_check_modules (GLIB2 glib-2.0)
查找glib2的任何版本。如果找到, 则输出变量 GLIB2_VERSION将保存找到的实际版本。
pkg_check_modules (GLIB2 glib-2.0>=2.10)
查找至少glib2 2.10版本。如果找到, 则输出变量 GLIB2_VERSION将保存找到的实际版本。
pkg_check_modules (FOO glib-2.0>=2.10 gtk+-2.0)
查找glib2-2.0(至少为2.10版)和gtk2 + -2.0的任何版本。只有同时找到两个都将FOO被视为找到。该FOO_glib-2.0_VERSION和FOO_gtk+-2.0_VERSION变量将被设置为各自的FOUND模块版本。
pkg_check_modules (XRENDER REQUIRED xrender)
需要任何版本的xrender。通过成功调用设置的示例输出变量：
XRENDER_LIBRARIES=Xrender;X11
XRENDER_STATIC_LIBRARIES=Xrender;X11;pthread;Xau;Xdmcp

`pkg_search_module`
只检查第一个成功匹配项, 而不检查所有模块
pkg_search_module(<prefix>
				  [REQUIRED] [QUIET]
				  [NO_CMAKE_PATH]
				  [NO_CMAKE_ENVIRONMENT_PATH]
				  [IMPORTED_TARGET [GLOBAL]]
				  <moduleSpec> [<moduleSpec>...])
pkg_search_module (BAR libxml-2.0 libxml2 libxml>=2)

`prefix`返回的结果变量<XXX>
```
<XXX>_FOUND
如果存在模块, 则设置为1
<XXX>_LIBRARIES
仅库(不带“ -l”）
<XXX>_LINK_LIBRARIES
库及其绝对路径
<XXX>_LIBRARY_DIRS
库的路径(不带“ -L”）
<XXX>_LDFLAGS
所有必需的链接器标志
<XXX>_LDFLAGS_OTHER
所有其他链接器标志
<XXX>_INCLUDE_DIRS
“ -I”预处理器标志(不带“ -I”）
<XXX>_CFLAGS
所有必需的标志
<XXX>_CFLAGS_OTHER
其他编译器标志
如果从中返回的关联变量具有多个值, 则所有字符都<XXX>_FOUND可能是;-pkg-config列表。
有一些特殊变量, 其前缀取决于<moduleSpec>给定的数目 。当只有一个时<moduleSpec>,  <YYY>将简单地为<prefix>, 但是如果给出两个或多个<moduleSpec> 项目, <YYY>则将为<prefix>_<moduleName>。
<YYY>_VERSION
模块版本
<YYY>_PREFIX
模块的前缀目录
<YYY>_INCLUDEDIR
包含模块目录
<YYY>_LIBDIR
模块的lib目录
```

### build_system
Unix Makefiles               = Generates standard UNIX makefiles.
Green Hills MULTI            = Generates Green Hills MULTI files
							   (experimental, work-in-progress).
Ninja                        = Generates build.ninja files.
Watcom WMake                 = Generates Watcom WMake makefiles.
CodeBlocks - Ninja           = Generates CodeBlocks project files.
CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
CodeLite - Ninja             = Generates CodeLite project files.
CodeLite - Unix Makefiles    = Generates CodeLite project files.
Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
Sublime Text 2 - Unix Makefiles
							 = Generates Sublime Text 2 project files.
Kate - Ninja                 = Generates Kate project files.
Kate - Unix Makefiles        = Generates Kate project files.
Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.

### CmakeLists.txt
```sh
cmake_minimum_required(VERSION 2.6)

#==============================================================================
# 项目名称, 会隐式定义XXX_BINARY_DIR XXX_SOURCE_DIR
project(PROPERTY)

# SOURCE——DIR始终一致
# CMAKE_SOURCE_DIR PROJECT_SOURCE_DIR XXX_SOURCE_DIR
# CMAKE_BINARY_DIR PROJECT_BINARY_DIR XXX_BINARY_DIR
# 外部编译(out-of-source build)时BINARY_DIR有区别, 为外部路径

#==============================================================================
# 添加子目录
# add_subdirectory()

# 源文件目录,添加到SRC_LIST
aux_source_directory(${PROJECT_SOURCE_DIR} SRC_LIST)
message(SRC_LIST= ${SRC_LIST})

# 头文件目录,等价于gcc -I
# include_directories(/usr/include/libdrm)

#==============================================================================
# 指定链接库
# 链接库目录
# link_directories(/usr/include/libdrm)
# 链接指定库,等价于gcc -l
# link_libraries(drm)

#==============================================================================
# 自动查找链接库
set(LIB_NAME libdrm)
find_package(PkgConfig REQUIRED)
pkg_search_module(LIB_DRM REQUIRED ${LIB_NAME})
# 链接库目录
include_directories(${LIB_DRM_INCLUDE_DIRS})
# 链接指定库,等价于gcc -l
link_libraries(${LIB_DRM_LIBRARIES})

#==============================================================================
# 添加编译器参数
# add_definitions(-I/usr/include/libdrm)
# 添加预处理器参数(-D)
# add_compile_definitions()

# 设置变量
# $ENV{NAME}可以引用环境变量
# set(SRC_DIR .)
# set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR})
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR})
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR})
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/DEBUG)
#==============================================================================
# 生成目标文件

# 生成可执行文件
add_executable(ums.elf ${SRC_LIST})

# 生成链接库(默认为静态链接库.a/.lib, 添加SHARED为动态链接库.so)
# 或者set(BUILD_SHARED_LIBSON)默认生成动态链接库
# add_library(libums SHARED ${LIB_SRC})
# set_target_properties(libums PROPERTIES OUTPUT_NAME "ums.so")

#==============================================================================
# 对目标链接指定的库
# target_link_libraries(ums.elf ${LIB_NAME})


#==============================================================================
# 安装动作
# install(TARGETS <target>... [...])			安装目标的生成结果
# install({FILES | PROGRAMS} <file>... [...])	安装文件
# install(DIRECTORY <dir>... [...])				安装目录
# install(SCRIPT <file> [...])					自定义安装脚本
# install(CODE <code> [...])					自定义安装命令
# install(EXPORT <export-name> [...])
# 安装对象类型
# ARCHIVE	静态库			${CMAKE_INSTALL_LIBDIR} 默认为lib
# LIBRARY	动态库			${CMAKE_INSTALL_LIBDIR}	默认为lib
# RUNTIME	可执行二进制文件	${CMAKE_INSTALL_BINDIR}	默认为bin
# 参数
# DESTINATION		安装到指定路径, 可以是绝对路径或者相对于 CMAKE_INSTALL_PREFIX
# CONFIGURATIONS	可以根据不同配置设置不同的安装规则(如 Debug/Release 两种配置)
# PERMISSIONS		指定安装文件权限, 包括
#					OWNER_READ, OWNER_WRITE, OWNER_EXECUTE, GROUP_READ, GROUP_WRITE, GROUP_EXECUTE, WORLD_READ, WORLD_WRITE, WORLD_EXECUTE, SETUID, 和 SETGID
# install(TARGETS libaiq CONFIGURATIONS Debug RUNTIME DESTINATION Debug/bin)
# install(TARGETS libaiq CONFIGURATIONS Release RUNTIME DESTINATION Release/bin)

#==============================================================================
# 输出 log
string(ASCII 27 Esc)
set(Reset	"${Esc}[m")
set(Red	"${Esc}[31m")
set(Green	"${Esc}[32m")
set(Yellow	"${Esc}[33m")

message("${Red}This is Red${Reset}")
```

## meson
Meson 旨在开发最具可用性和快速的构建系统。提供简单但强大的声明式语言用来描述构建。原生支持最新的工具和框架, 如 Qt5 、代码覆盖率、单元测试和预编译头文件等。利用一组优化技术来快速变异代码, 包括增量编译和完全编译
Meson是一个Python实现的开源项目构建系统, 其思想是, 开发人员花费在构建调试上的每一秒都是浪费, 同样等待构建过程直到真正开始编译都是不值得的
很多构建工具开始使用 meson + ninja
以后包的编译可能会出现下面转换
./autogen.sh && ./configure && make && sudo make install
--->
meson build && ninja -C build && sudo ninja -C build install

pip3 install meson ninja
一般将Meson和Ninja配合使用, Meson负责构建项目依赖关系, Ninja进行编译。


### 安装
sudo apt install python3 python3-pip ninja-build
pip3 install --user meson

### 自带例程
* 初始化一个基于 C 的例程
meson init --name mesontest
* 指定源码路径和编译路径, 源码路径可以省略, 默认为当前目录
meson setup build ./
* 构建, 生成到编译目录
meson build
* 编译
meson compile -C build
ninja -C build
* 测试
meson test -C build
ninja -C build test
* 安装, 默认路径是/usr/local, 可以用选项(--prefix)或者环境变量(DESTDIR)指定
meson install -C build
ninja -C build install
默认情况下，Meson 不会安装任何东西。可以通过
```sh
# 在构建目标中标记为可安装, 默认根据构建目标类型安装到对应目录, 可以使用 --prefix 指定安装路径前缀
shared_library('mylib', 'libfile.c', install : true)

# 使用 install_dir 参数指定安装路径
shared_library('mylib', 'libfile.c', install : true, install_dir:'/usr/local/lib')

install_headers('header.h', subdir : 'projname') # -> include/projname/header.h
install_man('foo.1') # -> share/man/man1/foo.1
install_data('datafile.dat', install_dir : get_option('datadir') / 'progname')
# -> share/progname/datafile.dat

# 重命名
# file.txt -> {datadir}/{projectname}/new-name.txt
install_data('file.txt', rename : 'new-name.txt')

# 递归复制整个目录
# mydir subtree -> include/mydir
install_subdir('mydir', install_dir:'include')

# 自定义安装脚本
meson.add_install_script('install.sh')
```

* 发布打包
meson dist

### 其它选项
* 输出不同等级的日志信息
```sh
message('This is a message')
warning('This is a warning')
error('This is a error')
```
* 指定编译后端
meson 可以使用 --backend 指定其他编译后端
meson build --backend=gcc
* 指定构建目标
Meson 提供了四种构建目标：可执行文件、库（可以在构建配置时再设置为静态或共享或两者都构建）、静态库和共享库。它们分别使用命令 executable 、 library 和 static_library 来创建 shared_library
* 指定依赖库/头文件路径
```sh
libs = [dependency(gtk+-3.0), dependency(drm)]
inc_dir = /usr/include
executable('mosetest', 'main.c', dependencies: libs, include_directories: [inc_dir])
```
* 自定义编译控制选项
使用 get_option 获取命令选项
```sh
install_tests = get_option('install-test-programs')
executable('modetest',files('modeset.c'),install : install_tests)
```

### 本地编译 c 工程
创建 meson 编译规则文件 meson.build
```sh
cc = meson.get_compiler('c')
cxx = meson.get_compiler('cpp')

libm = cc.find_library('m', required : false)

project('tutorial', 'c')
executable('demo', 'main.c')
```
meson build && cd build && ninja
./demo

### 交叉编译
meson 可以指定交叉编译规则文件
meson --cross-file <cross_compile_rule_file.txt>
```sh arm_cross_file.txt
[binaries]
exe_wrapper = 'wine'	# 指定运行命令时的包装器

[properties]			# 指定平台相关属性
sizeof_int = 4
sizeof_void* = 4

alignment_char = 1
alignment_void* = 4
alignment_double = 4

# 指定外部库目录和最终使用的 CFLAGS/LDFLAGS 的前缀
sys_root = '/some/path'
pkg_config_libdir = '/some/path/lib/pkgconfig'

needs_exe_wrapper = true

[host_machine]			# 交叉编译必须指定 host 和 target
system = 'windows'
cpu_family = 'x86'
cpu = 'i686'
endian = 'little'

[target_machine]
system = 'linux'
cpu_family = 'aarch64'
cpu = 'armv8a'
endian = 'little'
```

* 可用于交叉编译的辅助函数
```sh
# 检查是否为交叉编译, 如果是返回1
meson.is_cross_build()
# 检查 host 可执行文件能否运行, 或者是包装器或是原生的
meson.can_run_host_binaries()	

build_compiler = meson.get_compiler('c', native : true)
host_compiler = meson.get_compiler('c', native : false)

build_int_size = build_compiler.sizeof('int')
host_int_size  = host_compiler.sizeof('int')
```

## rpm build
yumdownloader --source pkg_name
rpm -i pkg_name
cd /root/rpmbuild
ls SOURCE SPECS
yum-builddep SPECS/xxx.spec
rpmbuild -ba SPECS/xxx.spec
SOURCE/configure

# 交叉编译
交叉编译工具链包括: 编译器(gcc) binutils(as,ld) c-rt库(glibc)
工具链的命名规则
arch-core-kernel-system-language
arch		体系架构, 如ARM/MIPS等, 表示编译器用于的目标平台
core		CPU Core型号(如Cortex A8)或是工具链的供应商
kernel		运行的OS, 常见的有Linux/uclinux/bare(无OS)
system		库函数和ABI, 如 gnu(glibc+oabi), gnueabi(glibc+eabi)
language	编译语言, 如 gcc/g++

## 制作交叉编译工具链
1. 编译Binutils
configure \
--target=${TARGET} \
--prefix=${RESULT_DIR} \
--disable-nls \
--disable-werror \
--disable-multilib \
--enable-shared
2. 安装内核头文件
make headers_install ARCH=arm CROSS_COMPILE=${TARGET}- INSTALL_HDR_PATH=${TARGET_PREFIX}
3. 编译 gcc(不包含目标平台 glibc, 可以用于编译内核跟 uboot 等不依赖 glibc)
./configure --prefix=${RESULT_DIR} --build=${HOST} --host=${HOST} --target=${TARGET} --without-headers
4. 编译目标平台的 glibc
5. 编译完整的 gcc
　　
## 用户态内核
* 编译内核
```sh
make defconfig ARCH=um
```

* 在本地 rootfs 中构建用户态根文件系统
```sh
sudo apt install debootstrap
fallocate -l 5G rootfs.iso
mkfs.ext4 rootfs.iso
mount rootfs.iso rootfs
cd rootfs
debootstrap sid ./ https://mirrors.tuna.tsinghua.edu.cn/debian
# 改变为根目录并创建 root 密码
chroot ./
passwd
# 退出 chroot 环境
exit
umount rootfs
设置 rootfs.iso 的所有权为普通用户
sudo chown `whoami` rootfs.iso
```

* 启动
```sh
screen ./vmlinux mem=1G root=/dev/root rootfstype=hostfs hostfs=./rootfs.iso  con=null con0=null,fd:2 con1=fd:0,fd:1
```

* 引发一个 kernel panic
```sh
# 开启 SysRq 功能(Magic System Request Key), 只要内核没有挂掉它就会响应你要求的所有操作(需要内核支持 CONFIG_MAGIC_SYSRQ 选项)
echo 1 > /proc/sys/kernel/sysrq
# 用于触发对应事件
# b重启, o关机, m内存信息, pCPU 寄存器信息, t线程信息, c崩溃, s重新挂载所有文件系统,
echo c > /proc/sysrq-trigger
```

## buildroot

## debootstrap
debootstrap是debian/ubuntu下的一个工具，用来构建一套基本的系统(根文件系统)。生成的目录符合Linux文件系统标准(FHS)，即包含了/boot、/etc、/bin、/usr等等目录，但它比发行版本的Linux体积小很多，当然功能也没那么强大，因此只能说是“基本的系统”。
sudo apt-get install debootstrap
sudo debootstrap --arch [平台] [发行版本代号] [目录] [源]

deb源码包下载和编译
在source.list里配好deb-src的源
apt-get source [name] # 下载并解包
apt-get source -d [name] # 下载源码包，不解包
dpkg-source -x [xx.dsc] # 解包
apt-get build-dep [name] # 安装指定包编译所需依赖
dpkg-buildpackage # 编译all
dpkg-buildpackage -b -aamd64  # 只编译指定架构

* dpkg-buildpackage部分参数解析
-D 检查依赖性
-d 不检查依赖和冲突
-nc 清理源码树
dpkg-buildpackage -nc 2>&1 | tee ../mk.log # 收集信息输出log

* amd64架构debian10案例
```sh
# 创建环境进入
mkdir buster&cd buster
sudo debootstrap --arch amd64 buster ./
sudo chroot .

# 修改云源
echo deb http://mirrors.ustc.edu.cn/debian buster main > /etc/apt/sources.list
echo deb-src http://mirrors.ustc.edu.cn/debian buster main >> /etc/apt/sources.list
apt-get update

# 部分版本是没有装编译工具包的
apt-get install dpkg-dev
apt-get install debhelper

# 下载编译源码
cd /tmp
apt-get source redis
apt-get build-dep redis
cd redis.x.x
dpkg-buildpackage
find ../*.deb
```

## buildd

## Ubuntu Builder


## LFS
https://linux.cn/lfs/LFS-BOOK-7.7-systemd/index.html
Linux From Scratch项目简称LFS，它提供具体的步骤、特定的补丁、必须的脚本，从而提供一个简便的创建Linux发行版的途径。LFS并不是一个发行版，但是它可以作为制作初级发行版的良好练习。
