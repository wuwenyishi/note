# IDEA本地将镜像推送到coding制品仓库



### 创建制品仓库

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.7zqcflzogk.jpg)

> 假设仓库名称为docker

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.77dgxvjzna.jpg)

### 在IDEA 添加Docker 注册表

> IDEA必须先安装docker插件

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.45hkwnjegx.jpg)

- 地址 

  ![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.1hs4maqy9q.jpg)

- 用户名和密码就是coding的登录名和密码   

- 服务器    

  ![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.7i0ar113nt.jpg)

> 最好本地安装docker桌面版，更容易操作

测试连接成功

### 推送镜像到coding的docker制品仓库

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.8hge47497y.jpg)



选中某个镜像 鼠标右键  
![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.4xugee1yl3.jpg)

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.77dgxvnigp.jpg)

- 注册表  
  选择添加的注册表  
- 仓库  
  填写与注册表的地址一致  
  ![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240808/image.6ik7duzk2y.jpg)
- 标签  
  就是镜像的tag，可以是镜像的版本号，例如v1.0.0；可以是其他；但不能有 `: `号，可以是`-`


