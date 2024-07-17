使用mapstruct 请求报错

Handler dispatch failed; nested exception is java.lang.NoClassDefFoundError: 

 ![](https://xuemingde.com/pages/image/2022/03/08/el10mD.png)



解决：

 ![](https://xuemingde.com/pages/image/2022/03/08/mwqlOu.png)

Mapper 引用的包错误。

错误的：import org.apache.ibatis.annotations.Mapper;

正确的：import org.mapstruct.Mapper;