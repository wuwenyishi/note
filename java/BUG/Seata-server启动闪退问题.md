
第一步：查看错误日志

打开cmd

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/51f427944a211ae872822076007567e3.png)





运行seata-server.bat查看错误信息

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/d6aa39237babca365050ee5a2cdaaf1b.png)

Error: missing server' JVl at C:\ Program Files (x86)\ Javaljre1. 8. 0_221\ bin\ server \ jvn. d11
Please instal1 or use the TRE or TDK that contains these missing components

找到出错目录

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/dd08f00ad0d66c1d03d8a4c13a5cb16b.png)

搜索jvm.dll

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/cdf348d3c0180b22eb39ccf59338cbcd.png)

在bin文件夹新建server，复制jvm.dll黏贴

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/5efb1191067265b42c97f139f3a7f6a5.png)



返回cmd，重新运行seata-server.bat

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/7ef391ac9de4b7b27c4ca1fb5b78ba0d.png)

说明我们没有足够的内存去使用，所以我们要降级我们内存的配置

用记事本方式打开seata-server.bat，找到参数设置

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/57d754d19a5aa3a373861fadaa01d4d4.png)

将其设置为1024,1024，512即可

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/bb870c732e5a25d30a14c8aafce72d18.png)

然后再次在cmd窗口中运行seata-server.bat

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/9540a6fbf3b5bf045543e90fe103cfd8.png)

如此解决了seata-server闪退的问题



问题二

启动报下面的错误

![image-20210312162616878](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210312162616878.png)

打开nacos，找到配置

![image-20210312163932230](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210312163932230.png)

在链接后面加上&characterEncoding=UTF-8&serverTimezone=UTC，然后发布，重新执行seata-server.bat

![image-20210312163953737](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210312163953737.png)



启动成功：

![image-20210312164119600](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210312164119600.png)



> 参考文章：https://www.pianshen.com/article/97751361089/