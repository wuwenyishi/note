## 文件系统类型

**内核中的模块：**ext4, xfs, vfat

**Linux的虚拟文件系统：**VFS

**用户空间的管理工具：**mkfs.ext4，mkfs.xfs，mkfs.vfat





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



### 查看所有磁盘和分区使用情况

```shell
lsblk
```

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0g0NWxO3vbxUTohy5fpzpqDs8WJccCL0nQ2RpQxiasykca7P49mTlG0vA/0?wx_fmt=png&from=appmsg)

sda: 磁盘

sda1、sda1、sda1 为三个分区



### 查看当前硬盘的分区情况

```shell
fdisk -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0gvT6bJ0OjWHmPhQEYMqSwRxYnA5lUmWOIfdU4GkPLFHuicJV6WegdTBw/0?wx_fmt=png&from=appmsg)



### 显示磁盘空间使用情况

```shell
df -h
```

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLsZSibL8icxYc3v9nWL4Rw24pzPZuxgp4xAGiaNLc4d56iat1PFGxVxK2NqtgQoic7o74jCGAyhWXZTwIw/0?wx_fmt=png&from=appmsg)



### 进入fdisk交互式界面

```shell
fdisk /dev/sda
```

查看分区信息，输入 p

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0gDFkWUPBCfqGvSljRdRGia4icUmAkAn36Jr0UDWv9HhpeX09yeR2fr1Tw/0?wx_fmt=png&from=appmsg)



m 帮助详情

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0gsWY4WvVe0dMib46xRBS7CcpC5hqgshUrF4R4cV8mHKeH4cyAgBDyUhQ/0?wx_fmt=png&from=appmsg)

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



