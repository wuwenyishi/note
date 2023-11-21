* [所有模块 Qt 5.15 中文文档编制,帮助,手册,教程](http://qt5.digitser.top/5.15/zh-CN/qtmodules.html)  
* [NekoSilverFox/OpenGL: 📽 基于 Qt & OpenGL 的计算机图形学实例](https://github.com/NekoSilverFox/OpenGL)  

------

**QComboBox 的选中时触发事件**

在 QComboBox 中，当用户选择下拉列表框中的项时，会触发 `activated` 和 `currentIndexChanged` 两个事件。这两个事件在用户选择不同的项时触发的时机略有不同。

1. `activated(int index)` 事件：当用户从下拉列表框中选择了一个项并完成选择（例如，点击下拉列表框中的项或按下回车键）时触发。参数 `index` 表示用户选择的项在列表中的索引。

```cpp
connect(comboBox, QOverload<int>::of(&QComboBox::activated), [=](int index) {
    // 处理用户选择的项
});
```

2. `currentIndexChanged(int index)` 事件：当用户从下拉列表框中选择了一个项时，即使下拉列表框仍然处于下拉状态，也会触发此事件。参数 `index` 表示用户选择的项在列表中的索引。

```cpp
connect(comboBox, QOverload<int>::of(&QComboBox::currentIndexChanged), [=](int index) {
    // 处理用户选择的项
});
```

根据您的需求，您可以选择连接这两个事件中的一个或两个，以便在用户选择下拉列表框中的项时触发相应的操作或逻辑。



**C++ 哪些参数需要释放内存**

在C++中，需要手动释放内存的参数主要包括以下几种情况：

1. 动态分配的内存：当你使用`new`关键字分配内存时，需要使用`delete`关键字手动释放内存。例如：

```cpp
int* p = new int;
// 使用指针p指向的内存
delete p; // 释放内存
```

2. 动态分配的数组：当你使用`new[]`关键字分配数组内存时，需要使用`delete[]`关键字手动释放内存。例如：

```cpp
int* arr = new int[10];
// 使用数组arr中的元素
delete[] arr; // 释放内存
```

3. 文件指针：当你使用`fopen`函数打开文件并返回一个`FILE*`指针时，需要使用`fclose`函数关闭文件并释放相关资源。例如：

```cpp
FILE* file = fopen("example.txt", "r");
// 使用文件指针file进行文件操作
fclose(file); // 关闭文件并释放资源
```

4. 动态分配的对象：当你使用`new`关键字动态分配一个对象时，需要使用`delete`关键字手动释放对象的内存。例如：

```cpp
class MyClass {
    // 类定义
};

MyClass* obj = new MyClass;
// 使用对象obj
delete obj; // 释放对象内存
```

需要注意的是，在C++中，只有手动分配的内存需要手动释放。对于自动分配的内存，例如局部变量或静态变量，不需要手动释放，因为它们的内存管理由系统自动处理。

同时，还有一些情况下需要注意内存的释放，例如在使用第三方库或操作系统提供的资源时，可能需要按照相应的规则进行内存的释放。具体的释放方式取决于所使用的库或资源的要求和文档。

总之，对于通过`new`、`new[]`或特定函数分配的内存，需要在使用完毕后手动释放对应的资源，以避免内存泄漏和资源浪费。
