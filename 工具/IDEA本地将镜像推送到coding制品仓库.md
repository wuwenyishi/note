# IDEA本地将镜像推送到coding制品仓库



### 创建制品仓库

![093128GCejqP](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyL79XeF6t6VQm7BYhZ7dmt0tTq3sLDShEN3F8VfXVI5LJo439Md0eicA/640?wx_fmt=png&amp;from=appmsg)

> 假设仓库名称为docker

![0931443Aeb93](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyxia3biaZLH6stoibJ1VTroW0YtkTgAUD7yuSNLwsoPicsiaywjBdodIMFPg/0?wx_fmt=png&from=appmsg)

### 在IDEA 添加Docker 注册表

> IDEA必须先安装docker插件

![093156se64Ci](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwytK31P1MVeSzAm4Gm9w03zezJ08rz1pcQce2oicVSfu0pic3LKROmQ48w/0?wx_fmt=png&from=appmsg)

- 地址 

  ![093536ho52bf](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyPQTWPjJYMby6GXLLzvFbKm9PelXkhgzwGv3PSicj8My6uxuRjp5rxxg/0?wx_fmt=png&from=appmsg)

- 用户名和密码就是coding的登录名和密码   

- 服务器    

  ![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwy7VmOjJIGqf0heRibZIE3lvuRh2LBfOAfkDicWTSx5oxb9bIWpf28pUsw/0?wx_fmt=png&from=appmsg)

> 最好本地安装docker桌面版，更容易操作

测试连接成功

### 推送镜像到coding的docker制品仓库

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyCPuj6z3zPyly7O8wNQ5icp1TFrjibvIVzBB2O5IdjtgbNkGbB2ErnyYg/0?wx_fmt=png&from=appmsg)



选中某个镜像 鼠标右键  
![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyVib4QL1VHjA4OWzmVND0VSEAjJ29WwAqDLzKP9sqHWqKicNPHtwdHMuw/0?wx_fmt=png&from=appmsg)

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwy4pTN4WIcJhqV0sTPW66xuDSOaUKVckZibm15btg28zPFIqfXMqOfOmA/0?wx_fmt=png&from=appmsg)

- 注册表  
  选择添加的注册表  
- 仓库  
  填写与注册表的地址一致  
  ![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyPQTWPjJYMby6GXLLzvFbKm9PelXkhgzwGv3PSicj8My6uxuRjp5rxxg/0?wx_fmt=png&from=appmsg)
- 标签  
  就是镜像的tag，可以是镜像的版本号，例如v1.0.0；可以是其他；但不能有 `: `号，可以是`-`


