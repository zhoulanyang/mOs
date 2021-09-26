#### CONFIG_BLK_DEV_INITRD

#### initramfs概述

​		initramfs与initrd类似，都是初始化好了且存在于ram中，可以压缩也可以不压缩。但是目前initramfs只支持cpio包格式，他会被populate_rootfs->unpack_to_rootfs(&__initramfs_start, &__initramfs_end - &__initramfs_start, 0)函数（解压缩）解析、安装。

#### initramfs与initrd的区别

1. Linux内核只认识cpio格式的initramfs文件包（因为unpack_to_rootfs只能解析cpio格式文件），非cpio格式的initramfs的文件包将被系统抛弃，而initrd可以是cpio包也可以是传统的镜像(image)文件，实际使用中initrd都是传统镜像文件。
2. initramfs在编译内核的同时被编译并与内核链接成一个文件，他被链接到地址__initramfs_start处，与内核同时被bootloader加载到ram中，而initrd是另外单独编译生成的，是一个独立的文件，他由bootloader单独加载到ram中内核空间外的地址，比如加载的地址为addr(是物理地址而非虚拟地址)，大小为8MB，那么只要在命令行加入“initrd=addr,8M”命令，系统就可以找到initrd。
3. initramfs被解析处理后原始的cpio包(压缩或非压缩)所占的空间(&\_\_initramfs_start - &\_\_initramfs_end)是作为系统的一部分直接保留在系统中，不会被释放掉，而对于initrd镜像文件，如果没有在命令行中设置"keepinitd"命令，那么initrd镜像文件被处理后其原始文件所占的空间(initrd_end - initrd_start)将被注释掉。
4. initramfs可以独立ram disk单独存在，而要支持initrd就必须要先支持ram disk，即要配置CONFIG_BLK_DEV_INITRD选项-支持initrd，必须先要配置CONFIG_BLK_DEV_RAM - 支持ramdisk，因为initrd image实际就是初始化好了的ramdisk镜像文件，最后都要解析、写入到ram disk设备/dev/ram或/dev/ram 0中。

Note：使用initramfs，命令行参数将不需要"initrd="和"root="命令。

#### 使用initramfs时的内核配置（使用initramfs做根文件系统）

General setup —> 
[*] Initial RAM filesystem and RAM disk (initramfs/initrd) support 
(/rootfs_dir) Initramfs source file(s) //输入根文件系统的所在目录

使用initramfs的内核启动参数 
不需要”initrd=”和”root=”参数,但是必须在initramfs中创建/init文件或者修改内核启动最后代码(init文件是软连接，指向什么? init -> bin/busybox，否则内核启动将会失败)

链接入内核的initramfs文件在linux-x.xx.xx/usr/initramfs_data.cpio.gz



#### 使用initrd的内核配置（使用网口将根文件系统下载到RAM - tftp addr ramdisk.gz）

1. 配置initrd 
   General setup —> 
   [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support 
   () Initramfs source file(s) //清空根文件系统的目录配置
2. 配置ramdisk 
   Device Drivers —> 
   Block devices —> 
   <*> RAM disk support 
   (16) Default number of RAM disks // 内核在/dev/目录下生成16个ram设备节点 
   (4096) Default RAM disk size (kbytes) 
   (1024) Default RAM disk block size (bytes)

使用 initrd的内核启动参数: 
     initrd=addr,0x400000 root=/dev/ram rw 
注: 
(1) addr是根文件系统的下载地址； 
(2) 0x400000是根文件系统的大小，该大小需要和内核配置的ramdisk size 4096 kbytes相一致； 
(3) /dev/ram是ramdisk的设备节点，rw表示根文件系统可读、可写；

#### 根文件系统存放在FLASH分区

1. 内核启动参数不需要”initrd=”(也可以写成”noinitrd”)； 
   root=/dev/mtdblock2 (/dev/mtdblock2 – 根文件系统所烧写的FLASH分区)
2. 内核配置不需要ram disk；也不需要配置initramfs或者initrd 
   [ ] Initial RAM filesystem and RAM disk (initramfs/initrd) support

Note: boot的FLASH分区要和kernel的FLASH分区匹配(而非一致)，需要进一步解释。



#### kernel启动init的两种方案

一、**ramdisk**，就是把一块内存(ram)当作磁盘(disk)去挂载，然后找到ram里的init进行执行。

二、**ramfs**，直接在ram上挂载文件系统，执行文件系统中的init。

initrd（init ramdisk）就是ramdisk的实现，initramfs就是ramfs的实现。

#### 各种存储的区别（RAM、ROM、FLASH）

1. ROM：Read Only Memory：一旦存储资料就无法再将之改变或删除，通常用在不需要经常变更资料的电子或电脑系统中，资料并且不会因为电源关闭而消失。
2. RAM：Random Access Memory，存储单元的内容可按需随意取出或存入，且存取的速度与存储单元的位置无关的存储器。这种存储器在断电时将丢失存储内容，故主要用于存储短时间使用的程序。典型的RAM就是计算机的内存。

##### RAM的分类

1. SRAM（Static RAM）：SRAM的速度非常快，是目前读写最快的存储设备了，但是他的价格也非常昂贵，所以只在要求很苛刻的地方使用，譬如CPU的一级、二级缓冲。

2. DRAM（Dynamic RAM）：DRAM保留数据的时间很短，速度也比SRAM慢，不过它还是比任何的ROM都要快，但从价格来说DRAM的价格比SRAM要便宜，计算机内存就是DRAM的。

   对于DRAM，此处以DDRAM（Double Date-Rate RAM）为例进行介绍，这种改进型的RAM和SDRAM是基本一致的，不同之处在于它可以在一个时钟读写两次数据，这样就使得数据传输速度加倍了

##### FLASH

​		FLASH存储器又称闪存，它结合了ROM和RAM的优点，不仅具备电子可擦除 可编程（EEPROM）的性能，还不会断电丢失数据同时可以快速读取数据（NVRAM的优势），U盘和MP3里用的就是这种存储器。

​		目前Flash主要由两种NOR Flash和NAND Flash；NOR Flash的读取和我们常见的SDRAM的读取是一样，用户可以直接运行装载在NOR Flash里面运行代码，这样可以减少SRAM的容量从而节约成本。



##### 相关名词解释

1. 什么是DRAM、SRAM、SDRAM？

   DRAM：动态随机存储器，需要不断的刷新，才能保存数据，而且是行列地址复用的，许多都有页模式。

   SRAM：静态随机存储器，加电情况下，不需要刷新，数据不会丢失，而且一般不是行地址复用的。

   SDRAM：同步的DRAM，即数据的读写需要时钟来同步。















