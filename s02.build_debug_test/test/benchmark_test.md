linux performance 测试工具
工具名称		测试项目			summary 										项目地址
coremark		cpu			测评cpu的整体性能（列举、矩阵运算、状态机、CRC）
coremark_pro	cpu			coremark的升级版，测评cpu整体性能
super PI		cpu			测评PI的计算										ftp://pi.super-computing.org/Linux/
SPEC			cpu			测评cpu性能										http://www.spec.org/spec/
dhrystone		cpu			测评CPU整形计算
whetstone		cpu			测评CPU浮点运算
nbench			cpu&mem		测评CPU运算性能（整数运算、双精度浮点运算）/mem指数主要体现处理器总线、cache和存储器性能
																			http://www.tux.org/~mayer/linux/bmark.html
utest_mem		mem			测评mem bandwidth
cachebench		mem			测评mem&cache bandwidth					http://icl.cs.utk.edu/projects/llcbench/cachebench.html
copybw			mem			测评mem bandwidth						http://www.tux.org/pub/benchmarks/CPU/copybw.c
ramspeed		mem			测评cache有效带宽
bonnie			IO			测评IO性能								http://www.textuality.com/bonnie/
Fio				IO			测评IO性能								http://freshmeat.net/projects/fio/
lmbench			CPU/mem/IO	测评cpu/mem/IO bandwidth & latency
sysbench		CPU/mem/IO	多线程性能测试
blktrace

# unixbench
UnixBench是原始的BYTE UNIX基准测试套件, 目的是提供类Unix系统性能的基本指标。因此，可以使用多个测试来测试系统性能的各个方面。然后将这些测试结果与基线系统的分数进行比较，以生成一个索引值，该索引值通常比原始分数更易于处理。然后，将整个索引值集组合在一起，以构成系统的整体索引。
包括一些非常简单的图形测试，以测量系统的2D和3D图形性能。
如果是多个CPU系统，则默认行为是运行选定的测试两次-一次运行每个测试程序一个副本，一次运行N个副本，其中N是CPU的数量。目的是让您评估：
运行单个任务时系统的性能
运行多个任务时系统的性能
系统实现并行处理的收益

包含的测试
Dhrystone测试
测试聚焦在字符串处理，没有浮点运算操作。这个测试用于测试链接器编译、代码优化、内存缓存、等待状态、整数数据类型等，硬件和软件设计都会非常大的影响测试结果。
Whetstone 测试
这项测试项目用于测试浮点运算效率和速度。这项测试项目包含若干个科学计算的典型性能模块，包含大量的C语言函数,sin cos sqrt exp和日志以及使用整数和浮点的数学操作。包含数组访问、条件分支和过程调用。
Execl Throughput测试
（execl 吞吐，这里的execl是类unix系统非常重要的函数，非办公软件的excel）
这项测试测试每秒execl函数调用次数。execl是 exec函数家族的一部分，使用新的图形处理代替当前的图形处理。有许多命令和前端的execve()函数命令非常相似。
File Copy测试
这项测试衡量文件数据从一个文件被传输到另外一个，使用大量的缓存。包括文件的读、写、复制测试，测试指标是一定时间内（默认是10秒）被重写、读、复制的字符数量。
Pipe Throughput（管道吞吐）测试
pipe是简单的进程之间的通讯。管道吞吐测试是测试在一秒钟一个进程写512比特到一个管道中并且读回来的次数。管道吞吐测试和实际编程有差距。
Pipe-based Context Switching （基于管道的上下文交互）测试
这项测试衡量两个进程通过管道交换和整数倍的增加吞吐的次数。基于管道的上下文切换和真实程序很类似。测试程序产生一个双向管道通讯的子线程。
Process Creation(进程创建)测试
这项测试衡量一个进程能产生子线程并且立即退出的次数。新进程真的创建进程阻塞和内存占用，所以测试程序直接使用内存带宽。这项测试用于典型的比较大量的操作系统进程创建操作。
Shell Scripts测试
shell脚本测试用于衡量在一分钟内，一个进程可以启动并停止shell脚本的次数，通常会测试1，2， 3， 4， 8 个shell脚本的共同拷贝，shell脚本是一套转化数据文件的脚本。
System Call Overhead （系统调用消耗）测试
这项测试衡量进入和离开系统内核的消耗，例如，系统调用的消耗。程序简单重复的执行getpid调用（返回调用的进程id）。消耗的指标是调用进入和离开内核的执行时间。
Graphical Tests
提供2D和3D图形测试；目前，由ubgears程序组成的3D套件非常有限。这些测试旨在为系统的2D和3D图形性能提供一个非常粗糙的概念。当然，请记住，报告的性能不仅取决于硬件，还取决于系统是否具有合适的驱动程序。

开始测试
从5.1版开始的UnixBench具有系统和图形测试
启用图形测试需要编辑 Makefile 配置选项 GRAPHIC_TESTS = define
make
./Run			运行系统测试
./Run graphics	运行图形测试
./Run gindex	两者都测试
apt-get install libxext-dev libgl1-mesa-dev
yum install -y SDL-devel mesa-libGL-devel

Run [ -q | -v ] [-i <n> ] [-c <n> [-c <n> ...]] [test ...]
-q 			不显示测试过程
-v 			显示测试过程
-i <count>	执行次数，最低3次，默认10
-c <n>		每次测试并行n个copies（并行任务）
Run -c 1 -c 4表示执行两次，第一次单个copies,第二次4个copies的测试任务
Run 依赖于 perl

安装
5.13
snap install unixbench
wget https://github.com/kdlucas/byte-unixbench/archive/v5.1.3.tar.gz

# bogoMIPS
Bogomips是Linux操作系统中衡量计算机处理器运行速度的的一种尺度，在linux和uClinux启动过程中，是通过calibrate_delay（）函数计算出来的。
可用来粗略计算处理器的性能，并不十分精确。
内核中 BogoMIPS 实现在源文件
/usr/src/linux/init/calibrate.c

cat /proc/cpuinfo
也会显示 bogoMIPS



# x11perf
xorg-x11-apps.mips64el.0.7.7-12.fc21.loongson
依赖
libXaw.mips64el.0.1.0.13-4.fc21.loongson
xorg-x11-fonts-misc.noarch.0.7.5-14.fc21.loongson
xorg-x11-xbitmaps.noarch.0.1.1.1-7.fc21.loongson


# CTS 兼容性测试套件
Compatibility Test Suite


strace ls
# 内存工具
https://www.ibm.com/developerworks/cn/linux/l-cn-valgrind/
Valgrind

# perf
内置于Linux内核源码树中的性能剖析(profiling)工具
应用程序可以利用 PMU，tracepoint 和内核中的特殊计数器来进行性能统计
还可以分析程序运行期间发生的硬件事件，比如 cache miss等；也可以分析软件事件，比如 page fault 和进程切换。

1）PMU：性能监控单元(Performance Monitor Unit), CPU提供的一个性能监视单元，用于统计CPU性能数据；
2）Tracepoint：散落在内核源代码中的一些 hook，它们可以在特定的代码被运行到时被触发，这一特性可以被各种 trace/debug 工具所使用。
3）内核运行状态计数，例如:  1) 进程切换   2) Page fault   3) 中断计数

perf list [hw/sw/cache/pmu]
列出所有能够触发perf采样点的事件

perf top
实时的观察下CPU时间的花费情况
-e <event>：指明要分析的性能事件
-p <pid>：Profile events on existing Process ID (comma sperated list). 仅分析目标进程及其创建的线程
-k <path>：Path to vmlinux. Required for annotation functionality. 带符号表的内核映像所在的路径
-K：不显示属于内核或模块的符号
-U：不显示属于用户态程序的符号
-d <n>：界面的刷新周期，默认为2s，因为perf top默认每2s从mmap的内存区域读取一次性能数据
-g：得到函数的调用关系图

perf stat
启动应用程序并分析该程序完整生命周期的性能状况

perf record
收集采样信息，并将其记录在一个文档里(默认是perf.data)，随后可用于perf report对数据文件进行分析
perf record -F 99 -p 13204 -g -- sleep 30
perf record表示记录，-F 99表示每秒99次，-p 13204是进程号，即对哪个进程进行分析，-g表示记录调用栈，sleep 30则是持续30秒。另外perf record常用选项如下：

-e record指定PMU事件
    --filter  event事件过滤器
-a  录取所有CPU的事件
-p  录取指定pid进程的事件
-o  指定录取保存数据的文件名
-g  使能函数调用图功能
-C 录取指定CPU的事件

perf report
读取perf record创建的文件，并给出热点分析结果

## WSL 安装 perf
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git --depth=1
git clone https://gitee.com/mirrors/WSL2-Linux-Kernel.git

make -j`nproc` -C WSL2-Linux-Kernel/tools/perf

# linux 测试
## x11perf
https://www.x.org/archive/individual/app 下载源码
./configure --build=mips-linux 	(build,host,target)

make

make install

## googletest
c++ 测试
