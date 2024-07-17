# IDEA本地将镜像推送到coding制品仓库



### 创建制品仓库

![093128GCejqP](https://xuemingde.com/pages/image/2024/04/24/093128GCejqP.png)

> 假设仓库名称为docker

![0931443Aeb93](https://xuemingde.com/pages/image/2024/04/24/0931443Aeb93.png)

### 在IDEA 添加Docker 注册表

> IDEA必须先安装docker插件

![093156se64Ci](https://xuemingde.com/pages/image/2024/04/24/093156se64Ci.png)

- 地址 

  ![093536ho52bf](https://xuemingde.com/pages/image/2024/04/24/093536ho52bf.png)

- 用户名和密码就是coding的登录名和密码

- 服务器
  ![093205RVyLTM](https://xuemingde.com/pages/image/2024/04/24/093205RVyLTM.png)

> 最好本地安装docker桌面版，更容易操作

测试连接成功

### 推送镜像到coding的docker制品仓库

![093218es0jcz](https://xuemingde.com/pages/image/2024/04/24/093218es0jcz.png)
选中某个镜像 鼠标右键
![093231dj0f99](https://xuemingde.com/pages/image/2024/04/24/093231dj0f99.png)
![093242xsaaz8](https://xuemingde.com/pages/image/2024/04/24/093242xsaaz8.png)

- 注册表
  选择添加的注册表
- 仓库
  填写与注册表的地址一致
  ![093259zSom6C](https://xuemingde.com/pages/image/2024/04/24/093259zSom6C.png)
- 标签
  就是镜像的tag，可以是镜像的版本号，例如v1.0.0；可以是其他；但不能有 `: `号，可以是`-`

