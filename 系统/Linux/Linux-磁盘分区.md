### 列出块设备的信息，包括硬盘、分区和挂载点等

```shell
lsblk
```

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0g0NWxO3vbxUTohy5fpzpqDs8WJccCL0nQ2RpQxiasykca7P49mTlG0vA/0?wx_fmt=png&from=appmsg)



### 查看当前硬盘的分区情况

```shell
fdisk -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0gvT6bJ0OjWHmPhQEYMqSwRxYnA5lUmWOIfdU4GkPLFHuicJV6WegdTBw/0?wx_fmt=png&from=appmsg)



### 显示磁盘空间使用情况

```shell
df -h
```

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0gf9lGpWj9uvkLhOicYHUiav6nykNeOBwK2LvericxMibZVQQ17QwoLTXGIw/0?wx_fmt=png&from=appmsg)



### 进入fdisk交互式界面

```shell
fdisk /dev/sda
```

查看分区信息，输入 p

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0gDFkWUPBCfqGvSljRdRGia4icUmAkAn36Jr0UDWv9HhpeX09yeR2fr1Tw/0?wx_fmt=png&from=appmsg)

