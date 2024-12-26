## 文件系统类型

**内核中的模块：** ext4, xfs, vfat

**Linux的虚拟文件系统：** VFS

**用户空间的管理工具：** mkfs.ext4，mkfs.xfs，mkfs.vfat

* XFS（文件系统）

  - CenOS 7系统中默认使用的文件系统，高性能的日志型文件系统，特别擅长于处理大文件，可支持
  - 上百万TB的存储空间
  - 存放文件和目录数据的分区
  - 数据完整性：根据所记录的日志在很短时间内迅速恢复磁盘文件内容
  - 传输带宽 ： XFS 能以接近裸设备I/O的性能存储数据。对单个文件的读写操作，吞吐量可达4GB每秒
  - 传输特性 ：用优化算法，日志记录对整体文件操作影响非常小。查询与分配存储空间非常快

* SWAP（文件交换系统）

  * 必备分区
  * 为Linux系统建立交换分区
  * 一般设置为物理内存的1.5~2倍

* EXT4

  Extended file system 适用于那些分区容量不是太大，更新也不频繁的情况，例如 /boot 分 区是

  ext 文件系统的最新版。提供了很多新的特性，包括纳秒级时间戳、创建和使用巨型文件 (16TB)、

  最大1EB的文件系统以及速度的提升





## 磁盘分区

### 分区类型

MBR（Master Boot Record）和GPT（GUID Partition Table）是在磁盘上存储分区信息的两种不同方式

这样操作系统才知道哪个扇区是属于哪个分区的，以及哪个分区是可以启动的。在磁盘上创建分区时，你必须在MBR和GPT之间做出选择。

MBR是Master Boot Record的简称，也就是主引导记录，是位于磁盘最前边的一段引导（Loader）代码，主要用来引导操作系统的加载与启动。

特点：

- MBR支持最大2TB磁盘，它无法处理大于2TB容量的磁盘
- 只支持最多4个主分区。若想要更多分区，需要创建扩展分区，并在其中创建逻辑分区

GPT磁盘是指使用GUID分区表的磁盘，GUID磁盘分区表（GUID Partition Table，缩写：GPT）其含义为“全局唯一标识磁盘分区表”，是一个实体硬盘的分区表的结构布局的标准。

特点：

- GPT对磁盘大小没有限制
- 最多可以创建128个分区




### 分区工具

fdisk是Linux下常用的磁盘分区工具。受mbr分区表的限制，fdisk工具只能给小于2TB的磁盘划分分区。如果使用fdisk对大于2TB的磁盘进行分区，虽然可以分区，但其仅识别2TB的空间，所以磁盘容量若超过2TB，就要使用`parted分区工具`（后面会讲）进行分区。

#### fdisk

- -l 列出素所有分区表
- -u 与 -l 搭配使用，显示分区数目



进入fdisk交互式界面

```shell
fdisk /dev/sda
```

查看分区信息，输入 p

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110948319.png)



查看当前硬盘的分区情况

```shell
fdisk -l
```

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110949354.png)



 帮助详情

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110949707.png)

a 切换分区启动标记

d 删除分区

g 创建一个新的空GPT分区表

G 创建一个IRIX (SGI)分区表

l 列出已知分区类型

m 打印此菜单

n 添加一个新分区

o 创建一个新的空DOS分区表

p 打印分区表

q 不保存更改而退出

s 创建一个新的空Sun磁盘标签

t 更改分区的系统id

v 验证分区表

w 将表写入磁盘并退出

x 额外功能(仅限专家使用)




### 查看所有磁盘和分区使用情况

```shell
lsblk
```

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110949043.png)

NAME：设备的名称。这是设备的唯一标识符，通常以 /dev/ 开头，例如 /dev/sda、/dev/sdb1。

MAJ:MIN：主设备号和次设备号，用于唯一标识设备。主设备号表示设备类型，次设备号用于区分同一类型的不同设备。例如，主设备号为 11 表示块设备，次设备号为 0 表示第一个 SCSI 设备，因此 11:0 表示 /dev/sda。

RM：可移动设备标志。0 表示非可移动设备，1 表示可移动设备。可移动设备是指可以插拔的设备，例如 USB 闪存驱动器。这个标志表示设备是否可以被安全地移除。

SIZE：设备的大小。通常以字节、千字节、兆字节或其他容量单位表示。例如，100G 表示 100GB。

RO：只读标志。0 表示设备可读写，1 表示设备只读。如果设备是只读的，则不能对其进行写操作，只能读取其中的数据。

TYPE：设备类型。表示设备的类型，常见的类型包括磁盘（disk）、分区（part）、光盘驱动器（rom）、循环设备（loop）等。

MOUNTPOINT：挂载点。如果设备已经挂载，则显示设备被挂载到的目录路径。如果设备没有被挂载，则显示为空。例如，/mnt/mydisk 表示设备被挂载到 /mnt/mydisk 目录下。





### 显示磁盘空间使用情况

```shell
df -h
```

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110949552.png)



### 格式化磁盘

* 格式化为ext4文件系统：mkfs.ext4 /dev/sdx

* 格式化为ext3文件系统：mkfs.ext3 /dev/sdx

* 格式化为NTFS文件系统：mkfs.ntfs /dev/sdx

   其中，/dev/sdx是待格式化的磁盘设备名，如/dev/sda、/dev/sdb等。




mkfs命令

用于创建文件系统，格式化指定的磁盘分区，例如创建Ext2、Ext3、Ext4、XFS等文件系统

**mkfs命令** 用于在设备上（通常为硬盘）创建Linux文件系统。mkfs本身并不执行建立文件系统的工作，而是去调用相关的程序来执行。

命令全称：make file system

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110950016.png)

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110950100.png)






### 查看磁盘负载状况 

> 推荐命令【top】【iostat】

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110950443.png)



### 分区扩容





## 磁盘挂载

### 显示磁盘空间使用情况

```shell
df -h
```

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110950900.png)



### 挂载

手动挂载

> 将设备 /dev/vdb1 挂载到 /data目录

* 创建挂载点

  ```shell
  mkdir /data
  ```

* 挂载

  ```shell
  mount /dev/vdb1 /data
  ```

* 查看挂载情况

  ```shell
  df -h
  ```

自动挂载

> 通过修改/etc/fstab实现自动挂载

修改/etc/fstab文件

文件的末尾添加一行，按照以下格式指定挂载点：

```
/dev/vdb1 /data                                 ext4    defaults        0 0
```

* /dev/vdb1 磁盘
* /data 挂载点

添加完成后，执行mount -a 即可生效

还有一种自动挂载的说法

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110951072.png)



分享一个自动化分区的脚本：【注意：需要根据线上环境微调部分参数】

```bash
#!/bin/bash

df -h|grep '/hadoop' && exit 1

yum install parted kmod-xfs xfsprogs -y
disk_num=`fdisk -l | grep 8001 | awk '{print $2}'| awk -F ':' '{print $1}'`
NUM=0
for i in $disk_num
do
        parted  -s $i mklabel gpt
        parted  -s $i mkpart primary 1 100%
        mkfs.xfs -f ${i}1
 
        if [ $NUM -eq 0 ];then
                TMP=""
        else
                TMP=$NUM
        fi
 
        mkdir /hadoop${TMP}
        mount -o noatime,nodiratime ${i}1 /hadoop${TMP}
        uuid=`blkid ${i}1 |awk '{print $2}' |sed s#\"##g`
        echo "$uuid     /hadoop${TMP}   xfs     noatime,nodiratime 0       0">>/etc/fstab
        ((NUM++))
done

```



## 查看

du 命令也是查看使用空间的，但是与 df 命令不同的是 Linux du 命令是对文件和目录磁盘使用的空间的查看，还是和df命令有一些区别的

![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110951691.png)



* 只列出当前目录下的所有文件夹容量

  > 包括隐藏文件夹

  ![](https://raw.githubusercontent.com/wuwenyishi/pages/gh-pages/image/others/202409110951035.png)

  



