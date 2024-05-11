---
title: "Linux驱动模型详解"
date: 2024-1-02 4:11:00
updated: 2024-3-21 19:19:01
cover: https://s21.ax1x.com/2024/05/10/pkZRsUS.png
description: 无事可做，诸君！
tag: LINUX
swiper_index: 2
---
# 1.Linux 驱动模型
>本节内容，重在理解。面试时，面试官很大可能会让你挑一个你熟悉的驱动讲讲，如何编写的？能回答出大概的驱动框架就可以。
## 1.1 字符设备驱动模型
驱动初始化中涉及到一个设备描述结构的概念。在任何一种驱动模型中，设备都会用内核中的一种结构来描述，这种结构成为设备描述结构。字符设备在内核中使用`struct cdev`这种结构来描述。
```c
struct cdev 
{
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops; //设备操作集
    struct list_head list;
    dev_t dev; //设备号
    unsigned int count; //设备数
};
```
count表明该类型设备的数目，如有两个串口，则count的值为2。

dev是设备号，包含有主设备号和次设备号的信息。主设备号用于区分设备的类型，次设备号用于标记相同类型的设备的不同个体。如串口1和串口2使用同一驱动程序，则其主设备号相同，但次设备号不同。Linux内核中使用`dev_t`类型来定义设备号，`dev_t`这种类型其实质为32位的`unsigned int`，其中高12位为主设备号，低20位为次设备号。

知道主设备号与次设备号，可通过`dev_t dev = MKDEV(主设备号，次设备号) `获得设备号；
从设备号分解出主设备号：主设备号 = `MAJOR(dev_t dev)`
从设备号分解出次设备号：次设备号 = `MINOR(dev_t dev)`
主设备号是一个重要的资源，可以通过静态申请和动态分配为设备分配一个主设备号：

静态申请：开发者自己选择一个数字作为主设备号，然后通过函数`register_chrdev_region`向内核申请使用。这种方法的缺点是如果申请使用的设备号已经被内核中的其它驱动使用了，则申请失败。
动态分配：使用`alloc_chrdev_region`由内核分配一个可用的主设备号。因为内核知道哪些号已经被使用了，所以不会导致分配到已经被使用的号。既然设备号是一种资源，则设备驱动在退出后都应该释放该资源。使用`unregister_chrdev_region`函数释放这些设备号。
ops是操作函数集。`file_operations`是一个很重要的结构，该结构的成员基本都是函数指针，并且是一些文件操作的函数的指针。
```c
struct cdev 
{
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops; //设备操作集
    struct list_head list;
    dev_t dev; //设备号
    unsigned int count; //设备数
};
```

`struct file_operations`是一个函数指针的集合，定义能在设备上进行的操作。结构中的函数指针指向驱动中的函数，这些函数实现一个针对设备的操作, 对于不支持的操作则设置函数指针为 NULL。例如：
```c
struct file_operations dev_fops = {
    .llseek = NULL,
    .read = dev_read,
    .write = dev_write,
    .ioctl = dev_ioctl,
    .open = dev_open,
    .release = dev_release,
};
```

## 1.2 实现设备操作
由struct file_operations可以看出，要实现的操作并不少，这里只介绍一些重要的操作。
```c
int (*open)(struct inode *, struct file *) //打开设备，响应open系统调用
int (*release)(struct inode *, struct file *);//关闭设备，响应close系统调用
loff_t (*llseek)(struct file *, loff_t, int) //重定位读写指针，响应lseek系统调用
ssize_t (*read)(struct file *, char __user *, size_t, loff_t *) //从设备读取数据，响应read系统调用
ssize_t (*write)(struct file *, const char __user *, size_t, loff_t *) //向设备写入数据，响应write系统调用
```
以上几个函数涉及到了`struct inode`和`struct file`这两种结构体。

在Linux系统中，每一个打开的文件，在内核中都会关联一个`struct file`结构体，它由内核在打开文件时创建，在文件关闭后释放。该结构体的重要成员有：
```c
loff_t f_pos /*文件读写指针*/
struct file_operations *f_op /*该文件所对应的操作*/
```
每一个存在于文件系统里面的文件都会关联一个`inode `结构，该结构主要用来记录文件物理上的信息。因此，它和代表打开文件的file结构是不同的。一个文件没有被打开时不会关联file结构，但是却会关联一个inode结构。该结构体重要的成员有：
```c
dev_t i_rdev /*设备号*/
```
一个设备支持的函数操作又称为设备方法。

# 2.Linux内核

1.uboot经过编译直接生成的elf格式的可执行程序是u-boot，这个程序类似于windows下的exe格式，在操作系统下是可以直接执行的。但是这种格式不能用来烧录下载。我们用来烧录下载的是u-boot.bin，这个东西是由u-boot使用arm-linux-objcopy工具进行加工（主要目的是去掉一些无用的东西）得到的。这个u-boot.bin就叫镜像（image），镜像就是用来烧录到iNand中执行的。
2.linux内核经过编译后也会生成一个elf格式的可执行程序，叫vmlinux或vmlinuz，这个就是**原始的未经任何处理加工的原版内核elf文件**；嵌入式系统部署时烧录的一般不是这个vmlinuz/vmlinux，而是要用objcopy工具去制作成烧录镜像格式（就是u-boot.bin这种，但是内核没有.bin后缀），经过制作加工成烧录镜像的文件就叫Image（制作把78M大的精简成了7.5M，因此这个制作烧录镜像主要目的就是缩减大小，节省磁盘）。
3.原则上Image就可以直接被烧录到Flash上进行启动执行（类似于u-boot.bin），但是实际上并不是这么简单。实际上linux的作者们觉得Image还是太大了所以对Image进行了压缩，并且在image压缩后的文件的前端附加了一部分解压缩代码。构成了一个压缩格式的镜像就叫zImage。（因为当年Image大小刚好比一张软盘（软盘有2种，1.2M的和1.44MB两种）大，为了节省1张软盘的钱于是乎设计了这种压缩Image成zImage的技术）。
4.uboot为了启动linux内核，还发明了一种内核格式叫uImage。uImage是由zImage加工得到的，uboot中有一个工具，可以将zImage加工生成uImage。注意：uImage不关linux内核的事，linux内核只管生成zImage即可，然后uboot中的mkimage工具再去由zImage加工生成uImage来给uboot启动。这个加工过程其实就是在zImage前面加上64字节的uImage的头信息即可。
5.原则上uboot启动时应该给他uImage格式的内核镜像，但是实际上uboot中也可以支持zImage，是否支持就看x210_sd.h中是否定义了LINUX_ZIMAGE_MAGIC这个宏。所以大家可以看出：有些uboot是支持zImage启动的，有些则不支持。但是所有的uboot肯定都支持uImage启动。
6.如果直接在kernel底下去`make uImage`会提供`mkimage command not found`。解决方案是去`uboot/tools`下`cp mkimage /usr/local/bin/`，复制mkimage工具到系统目录下。再去`make uImage`即可。
>通过上面的介绍我们了解了内核镜像的各种格式，如果通过uboot启动内核，Linux必须为uImage格式。

# 3.设备驱动

**主设备号：主设备号标识设备对应的特定的驱动程序**。虽然现代的linux内核允许多个驱动程序共享主设备号，但我们看待的大多数设备仍然按照“一个主设备对应一个驱动程序”的原则组织
**次设备号：次设备号由内核使用，用于确定由主设备号对应驱动程序中的各个设备**。依赖于驱动程序的编写方式，我们可以通过次设备号获得一个指向内核设备的直接指针，也可将此设备号当作设备本地数组的索引。