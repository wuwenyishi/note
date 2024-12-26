

#### 下载服务端压缩包

下载地址：链接: https://pan.baidu.com/s/18GR-zSHjCF-Qg98au6zfaA 提取码: n9fk 

#### 启动

![image-20210304163751931](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210304163758.png)

![image-20210304163843821](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210304163845.png)

访问：http://127.0.0.1:8848/nacos  

账户密码默认：nacos/nacos 

#### 登录首页

![image-20210304163959764](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210304164000.png)

#### 客户端设置与代码实现 （Springboot 项目）

1. 新建  命名空间

   ![image-20210305092812721](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305092814.png)

![image-20210305092919689](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305092920.png)

![image-20210305092944790](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305092945.png)

![image-20210305093108311](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305093109.png)

新建配置

![image-20210305093323554](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305093324.png)

![image-20210305093505015](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305093506.png)

代码配置



maven引用

<!--nacos组件-->

```
     <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>
    </dependencies>
```

新建 bootstrap.yml 文件，内容如下：

![image-20210305093729513](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305093731.png)



与服务端配置对比

![image-20210305094238911](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305094240.png)

![image-20210305094353139](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305094354.png)



新建获取配置信息文件，示例：

![image-20210305094645537](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305094646.png)

![image-20210305094635853](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305094636.png)

测试获取配置信息：

![image-20210305094744176](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305094745.png)

测试：

![image-20210305095009444](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20210305095010.png)



