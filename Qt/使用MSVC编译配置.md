在使用Mingw编译器编译没有问题的代码，如果使用MSVC编译器编译可能会出现字符集的问题。

## Qt Creator

从Mingw编译器改为MSVC，需要在.pro文件中增加：

```
msvc {
    QMAKE_CFLAGS += /utf-8
    QMAKE_CXXFLAGS += /utf-8
}
```

如果不增加，会有报错，报错是由于字符集编码导致。



## CLion

修改 CMakeLists.txt 文件

编译器改为msvc

```
set(CMAKE_PREFIX_PATH "D:/install/Qt/Qt5.12.2/5.12.2/msvc2017_64")
```

增加编码

```
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")
```

> 必须在 add_executable 之前



CLoin添加VS

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240828/image.7egpman90z.jpg)

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240828/image.5fkivyjekv.jpg)



## VS

安装 Qt VS Tools for Visual Studio 插件

安装后配置

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240828/image.b8u6p0lef.jpg)

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240828/image.7p3jfgm14e.jpg)

> 这里安装的是Qt5.12，根据自己安装的版本情况来设定
>
> 也可点击Autodetect来自动设定
>
> 要选择需要的编译器



![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240828/image.lvnzuhjw6.jpg)

![image](https://github.com/wuwenyishi/picx-images-hosting/raw/master/20240828/image.1hs5farwh3.jpg)

