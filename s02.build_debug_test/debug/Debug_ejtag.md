# GDB

## GDB 调试
首先在编译要把调试信息加到可执行文件,使用cc/gcc/g++的 -g 参数
如果没有-g,你将看不见程序的函数名、变量名,所代替的全是运行时的内存地址
默认编译参数调试过程是看不见宏的值, 需要增加编译选项 -g3
查看宏的值与查看变量相同用 p 打印, 还可以查看宏的展开
macro expand <macro_name>
info macro <macro_name>
启动GDB的方法有以下几种:
1. gdb <program>
		 program也就是你的执行文件,一般在当然目录下。
2. gdb <program> core
		 用gdb同时调试一个运行程序和core文件,core是程序非法执行后core dump后产生的文件。
3. gdb <program> <PID>
如果你的程序是一个服务程序,那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去,并调试他。program应该在PATH环境变量中搜索得到。

## GDB 命令行
shell <command string>
make <make-args>
r [args]	开始运行程序
s 			单步运行
n 			单块运行
finish		执行完当前函数
c 			继续运行到下一断点

l 			从第一行开始列出源代码
i			状态
set			设置变量
p i			打印变量i的值
disp		跟踪变量
bt			查看函数堆栈
dir <path>	添加源文件搜索路径
set substitute-path <old_path> <new_path> 替换原有的搜索路径
set substitute-path /build1 /local (/build1/a/1.cpp -> /local/a/1.cpp)
show substitute-path

break
watch

* GDB 启动选项
-symbols <file>
-s <file>
从指定文件中读取符号表。
-se file
从指定文件中读取符号表信息,并把他用在可执行文件中。
-core <file>
-c <file>
调试时core dump的core文件。
-directory <directory>
-d <directory>
加入一个源文件的搜索路径。

* 断点命令
b 16 		设置断点在源代码第16行
b func		设置断点,在函数func()
info b		查看断点信息
list
info breakpoints
break
delete
disable 和 enable
enable once 和 enable delete

* 源文件
可以在运行时额外指定源文件目录, 默认包括环境变量中的 PATH
show dir	显示当前的源文件搜索目录
dir P1:P2	把 P1, P2 加入源文件搜索目录
i source	查看当前源文件信息

* 段错误(coredump)
ulimit -a
第一条core file就是coredump—— 核心转储文件,size为0表示不允许吐核,更改一下文件大小就可以顺利产出吐核文件了。
通过指令更改为大小为无限制
ulimit -c unlimited
gdb elf coredump
signal 11是段错误(Segmentation fault)的典型图腾
通过bt指令查看函数调用栈
段错误
硬件设备MMU发现访问了一个非法的虚拟地址,通知操作系统内核给进程发送11号信号,进程收到了一个11号信号,导致进程异常终止。

在32位机器下,不能直接支持超过2G的文件。
解决方案一
g++   -D _FILE_OFFSET_BITS=64   file_test.cpp  -o   file_test
解决方案二
对与open,可以使用O_LARGEFILE参数,即:
fd   =   open("./bill_test",O_LARGEFILE|O_APPEND|O_RDWR,0666);
但是fopen没有这个参数,只能按照方法一来解决_

coredumo 文件路径
/proc/sys/kernel/core_pattern

## GDBTUI
tui enable
tui reg

1. 快捷键
Ctrl + x + 1	显示 source 窗口
Ctrl + x + 2	显示 assembly 窗口
2. 单键模式
Ctrl + x + s

## vscode
在调试控制台中可以使用 -exec <cmd> 来运行 gdb 中的命令, 如 -exec x/20xc 0x73fdf0 查看内存
x /nfu <addr>
* x 是 examine 的缩写
* n表示要显示的内存单元的个数
* f表示显示方式, 可取如下值:
	x 按十六进制格式显示变量
	d 按十进制格式显示变量
	u 按十进制格式显示无符号整型
	o 按八进制格式显示变量
	t 按二进制格式显示变量
	a 按十六进制格式显示变量
	i 指令地址格式
	c 按字符格式显示变量
	f 按浮点数格式显示变量
* u表示一个地址单元的长度:
	b表示单字节
	h表示双字节
	w表示四字节
	g表示八字节

# kgdb
源码级的Linux内核调试器, 可以对内核进行单步调试，设置断点，观察变量、寄存器的值等
## 配置过程
1. 重新编译带 debug 的内核
make menuconfig
2. 修改内核参数
uboot 修改 bootargs
kgdbwait kgdboc=ttySTM0,115200 ekgdboc
加入 kgdbwait 可以调试内核的启动过程
3. 启动内核, 连接 gdb
arm-none-linux-gnueabi-gdb ./vmlinux

cat /sys/module/kgdboc/parameters/kgdboc
echo g > /proc/sysrq-trigger

报错
kgdb: Unregistered I/O driver, debugger disabled.
串口驱动添加 uart_ops.poll_get_char/poll_put_char

# strace
strace有两种运行模式
一种是通过它启动要跟踪的进程。用法很简单,在原本的命令前加上strace即可。比如我们要跟踪 "ls -lh /var/log/messages" 这个命令的执行,可以这样:
strace ls -lh /var/log/messages
另外一种运行模式,是跟踪已经在运行的进程,在不中断进程执行的情况下,理解它在干嘛。 这种情况,给strace传递个-p pid 选项即可。
比如,有个在运行的some_server服务,第一步,查看pid:
pidof some_server
17553
得到其pid 17553然后就可以用strace跟踪其执行:
strace -p 17553


strace常用选项:
strace -tt -T -v -f -e trace=file -o /data/log/strace.log -s 1024 -p 23489
	-tt 在每行输出的前面,显示毫秒级别的时间
	-T 显示每次系统调用所花费的时间
	-v 对于某些相关调用,把完整的环境变量,文件stat结构等打出来。
	-f 跟踪目标进程,以及目标进程创建的所有子进程
	-e 控制要跟踪的事件和跟踪行为,比如指定要跟踪的系统调用名称
	-o 把strace的输出单独写到指定的文件
	-s 当系统调用的某个参数是字符串时,最多输出指定长度的内容,默认是32个字节
	-p 指定要跟踪的进程pid, 要同时跟踪多个pid, 重复多次-p选项即可。

# systemtap
SystemTap是一个Linux非常有用的调试（跟踪/探测）工具，常用于Linux内核或者应用程序的信息采集
获取一个函数里面运行时的变量、调用堆栈，甚至可以直接修改变量的值，对诊断性能或功能问题非常有帮助
可以快速、安全地提取、过滤和汇总数据，以便诊断复杂的性能或功能问题
SystemTap脚本设计的基本思想是 命名事件，并为它们提供 处理程序 hook
处理程序是一系列脚本语言语句，指定事件发生时要完成的工作。这项工作通常包括从事件上下文中提取数据，将它们存储到内部变量中，或者打印结果
SystemTap通过将脚本转换为C来工作，运行系统的 C 编译器来创建内核模块。当加载模块时，它通过钩住内核来激活所有探测到的事件。然后，当任何处理器上发生事件时，编译的处理程序都会运行。最后，会话停止，挂钩断开，模块移除。整个过程由一个命令行程序stap驱动
sudo stap -ve 'probe begin { log("hello world") exit() }'

何时使用
定位（内核）函数位置
查看函数被调用时的调用堆栈、局部变量、参数
查看函数指针变量实际指的是哪个函数
查看代码的执行轨迹（哪些行被执行了）
查看内核或者进程的执行流程
调试内存泄露或者内存重复释放
统计函数调用次数
......

SystemTap的处理流程有5个步骤：解析script文件(parse)、细化（elaborate）、script文件翻译成C语言代码（translate）、编译C语言代码（生成内核模块）（build）、加载内核模块（run）
https://www.cnblogs.com/xuelisherry/p/7412860.html

# systemtap
stap -v test.stp -p3 > out.c

比较常用和有用的参数：
-e SCRIPT               执行指定脚本
-l PROBE                List matching probes.
-L PROBE                List matching probes and local variables.
-g                      guru mode
-D NM=VAL               emit macro definition into generated C code
-o FILE                 结果输出到文件而不是 STDOUT
-x PID                  设置SystemTap处理函数 target() 指向特定进程号
-c cmd					设置SystemTap处理函数 target() 为指定的命令

SystemTap脚本
SystemTap脚本包含两个重要元素：event和handler。
SystemTap脚本默认使用扩展名.stp。
1. 探测点
探测点(probe)的语法格式如下：

probe event {statements}
一个探测点(probe)可以有多个事件，多个事件之间用逗号(,)隔开。
每个探测点都有一个对应的语句块(statement block)。语句块使用{}括起来。

2. 函数
函数定义语法如下：

function function_name(arguments) {statements}
probe event {function_name(arguments)}

3. 事件
事件可以分为两类：同步事件和异步事件。
同步事件
同步事件指执行到特定内核代码中时发生的事件。
同步事件包含下列几种：

syscall.system_call

监控系统调用，system_call可以改为要监控的系统调用，比如syscall.close。后面加一个.return用来监控系统调用返回，比如syscall.close.return。

vfs.file_operation

监控VFS(Virtual File System)的操作。也可以加.return后缀。

kernel.function(“function”)

监控内核函数。比如：kernel.function("sys_open")。同样，加.return就是监控函数返回。
还可以使用通配符 * 来监控某个文件上的多个函数，比如：

probe kernel.function("*@net/socket.c") {}
probe kernel.function("*@net/socket.c").return { }
kernel.trace(“tracepoint”)

tracepoint的静态探测。2.6.30版本之后的内核支持。需要在内核中静态的标记为tracepoint。例如kernel.trace("kfree_skb")指示每次释放一个网络缓冲区。

module(“module”).function(“function”)

模块中的函数：
probe module("ext3").function("*") { }
probe module("ext3").function("*").return { }


异步事件
异步事件包括：

begin
SystemTap会话开始，即脚本开始执行的时候触发的事件。

end
会话结束的事件

timer events

周期性触发的事件。例如：

probe timer.s(4)
{
	printf("hello world\n");
}
例子中每4秒钟打一次hello world。SystemTap支持下面几种定时器：

timer.ms(milliseconds)
timer.us(microseconds)
timer.ns(nanoseconds)
timer.hz(hertz)
timer.jiffies(jiffies)

SystemTap脚本语法

stap的语法与C语法很像，不过是事件触发语言。
probe begin
{
	printf("hello world\n")
	exit()
}

printf使用

SystemTap中的printf与C中的printf用法一样：
printf("format string\n", arguments)

这里的format string支持%d, %s等格式, arguments是不定参数, 参数之间用逗号隔开
probe syscall.open
{
	printf("%s(%d) open\n", execname(), pid())
}
这个脚本监控系统调用open。printf格式化打印一个字符串%s，即运行程序的名字(execname())，和一个整数，即进程的进程号(pid())。


# loongson-Ejtag
http://www.loongson.cn/uploadfile/embed/ls1b/ejtag/
ln -s /home/sunhao/work/loongson/software_pkg/ejtag-debug/ejtag_debug_usb /usr/local/bin/ejtag

# 配置文件
配置文件是 ejtag.cfg 程序会自动打开 ejtag.cfg 并执行里面的内容, 在运行之前需要根据你要调试的处理器类型对 ejtag.cfg 进行配置
* ssetconfig core.cpucount 1 设置处理器核的个数，如龙芯3A有4个处理器核，应该设置成4，龙芯3B
设置成8，龙芯2H、龙芯1A、1B、1C、1D设置成1。
* setconfig core.cpuwidth 32 设置处理器寄存器宽度，如龙芯3A是64位处理器，应该设置成64，龙
芯3B、龙芯2H设置成64，龙芯1A、1B、1C、1D设置成32。
* usblooptest设置usb ejtag的分频和位相选择,当其他都配置正确ejtag还不正常工作时，可以尝试对
ejtag时钟降频
```sh
	setenv LC_ALL C
	setenv LANG C
	setconfig usb_ejtag.get_speed 0x400
	setconfig usb_ejtag.put_speed 0x100
	setconfig put.pack_size 0x100000
	setconfig usb.maxtimeout 100000
	setconfig icache.size 0x4000
	setconfig icache.ways 4
	setconfig icache.waybit 12
	setconfig dcache.size 0x4000
	setconfig dcache.ways 4
	setconfig dcache.waybit 12
	setenv ENV_busclock 100000000
	setenv ENV_cpuclock 133000000
	setenv ENV_memsize 32
	setenv ENV_highmemsize 0
	setconfig karg.bootparam_addr 0xffffffffa1f00000
	setconfig helpaddr 0xffffffff8000f000
	#setconfig helpaddr 0xffffffffa0100000
	#setconfig putelf.uncached 2
	setconfig putelf.uncached 1
	#newcmd gdb "call gdb" "gdb file --args args"
	#newcmd ddd "shell scripts/ddd.sh" "ddd file --args args"
	newcmd cpu "setconfig core.cpuno" "switch between cpu"
	#newcmd cache_config "call cache_config" "autoconfig cache"
	#newcmd cache_init "call cache_init" "init cache"
	newcmd clkon "cp0s 0 m4 23 0x42110020"
	newcmd clkoff "cp0s 0 m4 23 0x40110020"
	#setconfig core.cpucount 4
	#setconfig core.cpuwidth 64
	#setconfig core.abisize 64
	#usblooptest 0x81000070 0x10010
	#usblooptest 0x81000070 0x20001
	#selectcore 0
	setconfig pp.pins 0x421730
	setenv TERM linux
	setenv TERMINFO ./terminfo
	setenv JTAG_IP 127.0.0.1
	setconfig usb.mode_sync 0
	newcmd mml m4 ""
	newcmd dml d4 ""
	newcmd REM "#" ""
	newcmd dr  "msleep 1000" ""
	newfunc gdbaccess 'gdbmap $$1 $$2 $$3 $$2 $$4'
	#dwarfdump
	#setconfig log.level 100
	#setconfig log.disas 1
	#verbose log.txt
	#cache_config
	#setconfig gdbserver.cpubitmap 1
	#setconfig gdbserver.forcehb 1
	#setconfig gdbserver.runonexit 0
	#setconfig core.nocache 1
	if $(test -e /usr/bin/gdb-multiarch) setenv GDB gdb-multiarch

	#setconfig jtag.fastdata32 0
	source scripts/gdb.cmd
	#setconfig callbin.ejtag 0
	setenv PATH "../eclipse:$EJTAGDIR/../eclipse:$EJTAGDIR:${PATH}"
	#log ${TIME+log/ejtag-%Y%m%d-%H%M%S.log}
```

```sh
	setenv LC_ALL C
	setenv LANG C
	setconfig core.cpucount 4
	setconfig core.cpuwidth 64
	setconfig usb_ejtag.get_speed 0x400
	setconfig usb_ejtag.put_speed 0x100
	setconfig put.pack_size 0x100000
	setconfig usb.maxtimeout 100000
	#timer 2000
```

# 运行ejtag
正确设置core.cpucount, core.cpuwidth
./ejtag_debug_usb
|命令|含义|
|--|--|
jtagled|1命令灯亮，jtagled 0命令灯灭，说明usb驱动安装正常|
usbver|返回版本日期信息|
jtagregs d8 1 1|读处理器的ejtag id寄存器，如果是0x20010819或者是0x5a5a5a5a都说明连接正确|
set|打印处理器的所有通用寄存器|

如果读不出来，按ctrl-c退出。可能是处理器在无程序的情况下运行到地址空洞，设备没响应，总线卡住了。可以运行resetcpu命令来复位cpu，然后按set就能读出通用寄存器内容了

如果ejtag已经插在座上，处理器下电了，处理器再上电后需要在ejtag软件里面允许
jtagled trst:0 trst:1


• cont
cont命令让程序退出ejtag debug状态，继续执行
• resetcpu [arg0] ...
无参数的时候，通过写0x11000到ejtag控制寄存器来复位处理器并进入debug状态,带参数的时候将参
数直接写到control 寄存器里面，如resetcpu 0x10000 0可以使处理器复位后继续执行。
• cpus [count] [file] 扫描每个处理器的asid和pc，格式是低32位是pc，高8位asid。当存文件时，
只保存pc数值。这个命令不影响处理器的执行，但需要处理器支持pc sample
• sample [count] [file] 通过触发ejtag异常来获得pc。这个命令会不断中断处理器执行，处理器执
行会变慢。

# 寄存器读写

## 通用寄存器
读通用寄存器
set [寄存器名]

写通用寄存器
set [寄存器名] [数值]

save [file] : 保存通用寄存器内容到文件/临时内存中
restore [file]: 恢复通用寄存器内容来自于文件/临时内存中

为了方面脚本软件调用，也可以用下面的方法访问
cpuregs
d4 1 2
m4 1 0x100或者

cpuregs d4 1 2
cpuregs m4 1 0x100
cpusregs表示设置d1、d2、d4、d8为1、2、4、8字节寄存器读功能，m1、m2、m4、m8为1、2、4、8字节寄
存器写功能。
d4 1 2表示读寄存器1开始的两个寄存器也就是at和v0寄存器。 m4 1 0x100描述写寄存器1为0x100，也就
是设置at寄存器为0x100

## 协处理器读写
选择协处理器组为sel，默认为0
cp0s [sel]
读协处理器
d4 regno [count]
或者 cp0s [sel] d4 [regno] [count]
cp0s 0 d4 16 1		读cp0_config0
cp0s 1 d4 16 1 		读cp0_config1

写协处理器
m4 regno value
或者 cp0s sel m4 regno val
cp0s 0 m4 16 2 		写cp0config0

## Ejtag寄存器读写
ejtag寄存器读写功能: jtagregs
• ejtag寄存器读: d4 regno [count] 或者 jtagregs d4 regno [count]
• ejtag寄存器写: m4 regno value 或者 jtagregs d4 regno val
读ejtag id寄存器: jtagregs d4 1 1
写ejtag control寄存器为0x1000: jtagregs m4 10 0x10

# 内存读写



龙芯 Ejtag 工具
一、安装：
tar zxvf ejtag-debug.tar.gz

二、运行
sudo su
./ejtag_debug_usb
程序会打开 ejtag.cfg 执行默认的配置
运行参数如下图，一般不需要使用

三、NOR Flash烧写PMON
cpu0- shell perl scripts/hisene.pl //初始化ddr寄存器
cpu0- putelf /home/.../gzrom //将gzrom下载到内存中
cpu0- put /home/.../gzrom.bin 0x8a000000 //将要烧写的gzrom.bin放到0x8a000000
cpu0- cont //启动pmon
在minicom中便可启动pmon
PMON> eraseboot
PMON> prgboot 0xbfc00000 0xe0000

四、调试start.S
cpu0- shell perl scripts/hisene.pl //初始化ddr寄存器
cpu0- putelf /home/.../gzrom
cpu0- gdb /home/.../pmon.gdb
(gdb) hb *0x81001480
(gdb) c
(gdb) display /i $pc //显示当前pc指针指向的指令
(gdb) si

五、启动并调试PMON
cpu0- shell perl scripts/hisene.pl //初始化ddr寄存器
cpu0- putelf /home/.../gzrom
cpu0- gdb /home/.../pmon.gdb
(gdb) hb initmips
(gdb) c

六、启动并调试内核
cpu0- shell perl scripts/hisene.pl //初始化ddr寄存器
cpu0- putelf /home/.../gzrom
cpu0- cont
cpu0- putelf /home/.../vmlinux_hs3000_ramdisk
cpu0- karg console=ttyS0,115200 root=/dev/ram1 rw cca=2
cpu0- gdb /home/.../vmlinux
(gdb) hb start_kernel
(gdb) c
(gdb) b sys_read
(gdb) c
内核shell下输入ls便可以停下。

七、调试app
pmon和kernel都是运行在内核空间的，调试器处理起来比较方便，而app是运行在用户空间的，调试的时候需要进行一步虚拟地址的转换，所以龙芯Ejtag调试器没有很好支持应用程序调试，只能做一些简单的汇编单步。
cpu0- shell perl scripts/hisene.pl //初始化ddr寄存器
cpu0- putelf /home/.../gzrom
cpu0- cont //启动pmon
cpu0- putelf /home/.../vmlinux_hs3000_ramdisk
cpu0- karg console=ttyS0,115200 root=/dev/ram1 rw cca=2
cpu0-cont //启动kernel
cpu0- hb 0x4005f0
cpu0-setconfig jtag.showins 0 //关指令回显，这样会块一些
cpu0-si 100 //单步运行100次
cpu0-unsi //关si
cpu0-cont //继续运行

八、下载内存数据
sget file addr size
strings file > vi file 将下载的二进制文件转换为文本
系统日志在内存中的地址需要返汇编内核二进制查看，返汇编后查找<__log_buf>,
找到后即可得到内存地址。

注意的问题：
1.龙芯的Ejtag原理是FPGA上跑一个gdbserver，然后主机通过gdb与gdbserver通信来进行调试，所以本质上是通过gdb来调试的，所用的命令也跟gdb的命令完全相同。
2.调试的过程退出gdb时，需要先detach一下，然后q退出gdb便可以回到cpu0-模式。
3.需要把源码放到对应的目录，否则无法用list命令查看源码。
4.cpu0-ctrl+r可以筛选历史命令。
5.cpu0-ctrl+x+a可以打开一个简单的调试界面，更友好的调试界面可以安装一个ddd来代替gdb。
6.cpu0-set用来进行寄存器操作，set显示所有寄存器，set pc 0xbfc00000设置寄存器值。


CDT + GDB 图形化远程调试
1.在Eclipse中选择一个项目，点击菜单 Run > Debug Configurations，在左侧的Dubug类型中找到“C/C++ Remote Application”，右击点“New”，创建一个远程调试配置。

Eclipse的C/C++插件CDT已经很好的支持gdb在远程调试了。调试一个应用程序时，CDT有三种运行方式：
Automatic Remote Launcher ：远程自动运行，这是最方便好用的一种方式
Manual Remote Launcher : 远程手动运行。用户自己在目标板上运行gdbserver，然后在开发主机上指定远程连接的方式（如IP地址和端口），连接到gdbserver
Remote Attach Launcher ：远程依附运行。类似于上一种，但它不是重新运行程序开启一个debug会话，而是直接Attach到一个已经运行的程序，然后调试
在Debug Configurations 对话框中，创建一个远程调试配置，这个配置在创建时会根据项目情况提供一个默认的配置，默认将使用第一种Automatic Remote Launcher方式，
这在Main标签中下方“GDB (DSF) Automatic Remote Debugging Launcher”可以看出，点击右边的“Select other…”可以切换其它方式。

2.使用ejtag仿真器中集成的gdbserver,需要使用远程手动，即第二种方式。接下来配置CDT的Debug选项，步骤如下：
选中项目→菜单栏 ”Run“→Debug Configurations…
双击 C/C++ Remote Application 新建一个配置，Eclipse会根据当前选择的项目初始化大部分配置，这里只需修改Debugger配置页
在右下方点击“Select other”，选择“GDB(DSF) Manual Remote Debugging Launcher”，确认
选择进入Debugger配置页，在Main标签中，GDB debugger 填写 arm-linux-gdb，如果未加入PATH环境变量则应该填入绝对路径
在Debugger配置页的Shared Libraries标签中，可以添加库路径，比如调试过程中要步入外部函数，就必须在这里给出带调试信息的库文件路径，否则会找不到该函数的定义
在Debugger配置页的Connection标签中，选择localhost:50010
所有配置完成后，点“Apply”保存配置，并关掉配置窗口
接下来在目标板上运行 gdbserver.

3.目标板的 GDB 服务开启后，我们就可以在开发主机中，点击Eclipse的调试按钮（指定调试配置为刚才配置好的），开始应用程序的远程调试。我们在连接目标板的终端中，
可以看到程序的标准输出；也可以在这里标准输入。这一就可以完成远程的图形化调试。

4.需要提前准备的有：
eclipse-for linux c/c++(CDT)
 mipsel-linux-gcc
 mipsel-gdb
 ejtag-debug(启动ejtag上集成的gdbserver,目前只有x86版本)

用Eclipse和GDB构建ARM交叉编译和在线调试环境http://blog.csdn.net/bugouyonggan/article/details/9276103
Eclipse在线调试ARM11——Tiny6410+OpenJTAGhttp://blog.csdn.net/girlkoo/article/details/8056334