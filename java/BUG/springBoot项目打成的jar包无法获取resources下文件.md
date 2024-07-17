
错误：

> cannot be resolved to absolute file path because it does not reside in the file system: jar:file:/U  


spring boot项目打成的 jar包无法获取src/main/resources下文件  

![bdDMAq](https://xuemingde.com/pages/image/others/bdDMAq.png)  
打成Jar后  
![yC8St7](https://xuemingde.com/pages/image/others/yC8St7.png)  


获取resources下的文件   
![et123k](https://xuemingde.com/pages/image/others/et123k.png)  
```java
 ClassPathResource classPathResource = new ClassPathResource("img/tofflon-logo.png");
 URL url = classPathResource.getURL();
 Image image = Image.getInstance(url);
```
