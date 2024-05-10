---
title: "Linux驱动基础知识"
date: 2024-5-09 14:56:04
updated: 2024-5-10 16:44:01
cover: https://img2.imgtp.com/2024/05/10/ooBwAdBx.jpg
description: LINUX
tag: LINUX
swiper_index: 1 
---
# 1.指令
## 1.1 常用的Linux指令
### 1.怎么查看当前进程？怎么执行退出？怎么查看当前路径？
查看当前进程： `ps`
执行退出： `exit`
查看当前路径： `pwd`

### 2.ls 命令执行什么功能？ 可以带哪些参数？
功能
列出指定目录中的目录，以及文件
参数
`-a` 显示所有文件及目录 (. 开头的隐藏文件也会列出)
`-l` 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出
`-r` 将文件以相反次序显示(原定依英文字母次序)
`-t` 将文件依建立时间之先后次序列出
`-A` 同 -a ，但不列出 “.” (目前目录) 及 “…” (父目录)
`-F` 在列出的文件名称后加一符号；例如可执行档则加 “*”, 目录则加 “/”
`-R` 若目录下有文件，则以下之文件亦皆依序列出

### 3.创建目录用什么命令？
```shell
mkdir runoob        #在工作目录下，建立一个名为 runoob 的子目录 
mkdir -p runoob2/test   #在工作目录下的 runoob2 目录中，建立一个名为 test 的子目录。
若 runoob2 目录原本不存在，则建立一个。（注：本例若不加 -p 参数，且原本 runoob2 目录不存在，则产生错误。）
```

### 4.创建文件用什么命令？
**vi或vim**
```shell
vi file1.txt #直接创建并打开一个文件file1.txt
```
**touch**
```shell
touch file2.txt #创建新的空文件file2.txt
```
**echo**
```shell
echo "this is a new file" > file3.txt   #创建文件file3.txt并将this is a new file写入（说明：使用>指令覆盖文件原内容并重新输入内容，若文件不存在则创建文件。）
echo "add contents" >>file3.txt    #在已存在的文件补充写入新内容add contents（说明：使用>>指令向文件追加内容，原内容将保存。）
```
**less 、more 、cat**

三者都是将文件内容输出到标准输出，其中less和more可以分页显示，cat是显示全部。
三者可以根据已经存在的文件创建新的文件。假设已经存在文件1.txt。
```shell
cat 1.txt > 2.txt
less 1.txt > 3.txt
more 1.txt > 4.txt
```
## 1.2 常用的GCC指令
### 1.预处理 (-E .i)
```shell
gcc -E test.c -o test.i #把预处理的结果导出到test.i文件
```
### 2.编译为汇编代码(-S .s)
```shell
gcc -S test.i -o test.s #编译器将test.i翻译成汇编语言，并将结果存储在test.s文件中。
```
### 3.汇编(-c .o)
```shell
gcc -c test.s -o test.o #将汇编代码编译为目标文件（.o）但不链接
```
### 4.链接(无后缀)
```shell
gcc test.o -o test #将生成的目标文件test.o生成最终的可执行文件test
```
### 5.一步到位编译(.c 无后缀)
```shell
gcc test.c -o test #将源文件test.c编译链接为可执行文件test
```

## 1.3 常用的GDB调试指令
GDB（GNU symbolic debugger）是 GNU Project 调试器
GNU操作系统是一种由自由软件构成的类 Unix 操作系统
```shell
gcc -g test.c -o test  #编译时生成debug有关的程序信息
gdb test        #启动调试
help #查看命令帮助，具体命令查询在gdb中输入help + 命令,简写h
run #重新开始运行文件（run-text：加载文本文件，run-bin：加载二进制文件）,简写r
start #单步执行，运行程序，停在第一执行语句
list #查看原代码（list-n,从第n行开始查看代码。list+ 函数名：查看具体函数）,简写l
set #设置变量的值
next #单步调试（逐过程，函数直接执行）,简写n
step #单步调试（逐语句：跳入自定义函数内部执行）,简写s
backtrace #查看函数的调用的栈帧和层级关系,简写bt
frame #切换函数的栈帧,简写f
info #查看函数内部局部变量的数值,简写i
finish #结束当前函数，返回到函数调用点
continue #继续运行,简写c
print #打印值及地址,简写p
quit #退出gdb,简写q
break+num #在第num行设置断点,简写b
info breakpoints #查看当前设置的所有断点
delete breakpoints num #删除第num个断点,简写d
display #追踪查看具体变量值
undisplay #取消追踪观察变量
watch #被设置观察点的变量发生修改时，打印显示
i watch #显示观察点
enable breakpoints #启用断点
disable breakpoints #禁用断点
x #查看内存x/20xw 显示20个单元，16进制，4字节每单元
run argv[1] argv[2] #调试时命令行传参
set follow-fork-mode child #Makefile项目管理：选择跟踪父子进程(fork())
```

## 1.4 常用的驱动开发指令
### 1.加载/卸载驱动
```shell
insmod/modprobe #加载驱动
rmmod   #卸载驱动Linux驱动如何查看驱动模块中打印信息？
```
### 2.Linux驱动如何查看驱动模块中打印信息？
```shell
dmesg
```
### 3.如何查看内核中已有的字符设备的信息？
`lsmod `和`modprobe：lsmod`可以查看模块的依赖关系，`modprobe`在加载模块时会加载其他依赖的模块。

# 2.UBOOT

## 2.1 什么是bootloader？
Linux系统要启动就必须需要一个 bootloader程序，也就说芯片上电以后先运行一段bootloader程序。这段 bootloader程序会先初始化时钟，**看门狗，中断，SDRAM，等外设，然后将 Linux内核从 flash（NAND, NOR FLASH,SD,MMC等）拷贝到SDRAM中，最后启动Linux内核**。当然了， bootloader的实际工作要复杂的多，但是它最主要的工作就是启动 Linux内核。
bootloader和 Linux内核的关系就跟PC上的BIOS和 Windows的关系一样，**bootloader就相当于BIOS。总得来说，Bootloader就是一小段程序，它在系统上电时开始执行，初始化硬件设各、准备好软件环境，最后调用操作系统内核。**
## 2.2 uboot启动过程中做了那些事？
第一阶段
初始化时钟，关闭看门狗，关中断，启动ICACHE，关闭DCACHE和TLB，关闭MMU，初始化SDRAM，初始化NAND FLASH，重定位。

>Linux的看门狗（Watchdog）是一种硬件或软件机制，用于监视系统的运行状态，以防止系统出现故障或死机的情况。它能够检测到系统的异常情况并自动采取一些预设的措施，例如重启系统或发送警报信息。

第二阶段
初始化一个串口，检测系统内存映射，将**内核映象和根文件系统映象从Flash上读到SDRAM空间**中，为内核设置启动参数，调用内核。

## 2.3 uboot和内核如何完成参数传递？
uboot启动后已经完成了基本的硬件初始化（如：内存、串口等），接下来，它的主要任务就是加载Linux内核到开发板的内存，然后跳转到Linux内核所在的地址运行。
**PS:只要问到uboot，面试官必问uboot和内核的参数传递，所以一定要知道！**
具体是如何跳转呢？做法很简单，直接修改PC寄存器的值为Linux内核所在的地址，这样CPU就会从Linux内核所在的地址去取指令，从而执行内核代码。
在前面我们已经知道，在跳转到内核以前，uboot需要做好以下三件事情：
### 1.CPU寄存器的设置
R0=0
R1=机器类型ID；对于ARM结构的CPU，其机器类型ID可以参见 linux/arch/arm tools/ mach-types
R2=启动参数标记列表在RAM中起始基地址
### 2.CPU工作模式
必须禁止中断（IRQs和FIQs）
CPU必须为SVC模式
### 3.Cache和MMU的设置
MMU必须关闭
指令 Cache可以打开也可以关闭
数据 Cache必须关闭
其中上面第一步CPU寄存器的设置中，就是通过R0，R1，R2三个参数给内核传递参数的。
## 2.4 为什么要给内核传递参数呢？
在此之前，uboot已经完成了硬件的初始化，可以说已经”适应了“这块开发板。然而，内核并不是对于所有的开发板都能完美适配的（如果适配了，可想而知这个内核有多庞大，又或者有新技术发明了，可以完美的适配各种开发板），此时，对于开发板的环境一无所知。所以，要想启动Linux内核，uboot必须要给内核传递一些必要的信息来告诉内核当前所处的环境。
## 2.5 如何给内核传递参数？
uboot把**机器ID通过R1传递给内核**，Linux内核运行的时候，首先就从R1中读取机器ID来判断是否支持当前机器。这个机器ID实际上就是开发板CPU的ID，每个厂家生产出一款CPU的时候都会给它指定一个唯一的ID，大家可以到uboot源码的`arch\arm\include\asm\mach-type.h`文件中去查看。
**R2存放的是块内存的基地址**，这块内存中存放的是uboot给Linux内核的其他参数。这些参数有内存的起始地址、内存大小、Linux内核启动后挂载文件系统的方式等信息。很明显，参数有多个，不同的参数有不同的内容，为了让Linux内核能精确的解析出这些参数，双方在传递参数的时候要求参数在存放的时猴需要按照双方规定的格式存放。

除了约定好参数存放的地址外，还要规定参数的结构。Linux2.4.x以后的内核都期望以标记列表（tagged_list）的形式来传递启动参数。标记，就是一种数据结构；标记列表，就是挨着存放的多个标记。标记列表以标记ATAG_CORE开始，以标记ATAG_NONE结束。

标记的数据结构为tag，它由一个tag_header结构和一个联合（union）组成。tag_header结构表示标记的类型及长度，比如是表示内存还是表示命令行参数等。对于不同类型的标记使用不同的联合（union），比如表示内存时使用tag_ mem32，表示命令行时使用 tag_cmdline。具体代码见`arch\arm\include\asm\setup.h。`

从上面可以看出，struct_tag结构体由structtag_header+联合体union构成，结构体struct tag_header用来描述每个tag的头部信息，如tag的类型，tag大小。联合体union用来描述每个传递给Linux内核的参数信息。
## 2.6 为什么uboot要关掉caches？
caches是cpu内部的一个2级缓存，它的作用是将常用的数据和指令放在cpu内部。caches是通过CP15管理的，刚上电的时候，cpu还不能管理caches。上电的时候指令cache可关闭，也可不关闭，但数据cache一定要关闭，否则可能导致刚开始的代码里面，去取数据的时候，从cache里面取，而这时候**RAM中数据还没有caches过来，导致数据预取异常。**

# 3.文件系统
## 3.1 什么是根文件系统？
根文件系统首先是一种文件系统，该文件系统不仅具有普通文件系统的存储数据文件的功能，但是相对于普通的文件系统，它的特殊之处在于，它是**内核启动时所挂载（mount）的第一个文件系统**，内核代码的映像文件保存在根文件系统中，系统引导启动程序会在根文件系统挂载之后从中把一些初始化脚本（如rcS、inittab）和服务加载到内存中去运行，里面包含了Linux系统能够运行所必需的应用程序、库等，比如可以给用户提供操作 Linux的控制界面的shell程序、动态连接的程序运行时需要的glibc库等。

文件系统和内核是完全独立的两个部分。在嵌入式中移植的内核下载到开发板上，是没有办法真正的启动Linux操作系统的，会出现无法加载文件系统的错误。

## 3.2 根文件系统为什么这么重要？
根文件系统之所以在前面加一个“根”，说明它是加载其它文件系统的“根”，那么如果没有这个根，其它的文件系统也就没有办法进行加载的。根文件系统包含系统启动时所必须的目录和关键性的文件，以及使其他文件系统得以挂载（mount）所必要的文件。例如：
init进程的应用程序必须运行在根文件系统上。
根文件系统提供了根目录“/”。
linux挂载分区时所依赖的信息存放于根文件系统/etc/fstab这个文件中。
shell命令程序必须运行在根文件系统上，譬如ls、cd等命令。
总结：一套linux体系，只有内核本身是不能工作的，必须要rootfs（上的etc目录下的配置文件、/bin/sbin等目录下的shell命令，还有/lib目录下的库文件等）相配合才能工作。

## 3.3 可执行映像文件通常由几部分构成，它们有什么特点？
可执行映像文件通常由以下几部分构成：
+ 一个或多个代码段，代码段的属性为只读。
+ 零个或多个包含初始化数据的数据段，数据段的属性为可读写。
* 零个或多个不包含初始化数据的数据段，数据段的属性为可读写。