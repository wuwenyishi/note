> 文章来源自 effective c++（第三版） 这本书



## 用consts, enums和inlines取代#defines

“用compiler（编译器）取代 preprocessor（预处理器）”，因为 #define 根本就没有被看作是语言本身的一部分。

`#define ASPECT_RATIO 1.653`

compiler（编译器）也许根本就没有看见这个符号名 ASPECT_RATIO，在 compiler（编译器）得到源代码之前，这个名字就已经被 preprocessor（预处理器）消除了。结果，名字 ASPECT_RATIO 可能就没有被加入 symbol table（符号表）。如果在编译的时候，发现一个 constant（常量）使用的错误，你可能会陷入混乱之中，因为错误信息中很可能用 1.653 取代了 ASPECT_RATIO。如果，ASPECT_RATIO 不是你写的，而是在头文件中定义的，你可能会对 1.653 的出处毫无头绪，你还会为了跟踪它而浪费时间。在 symbolic debugger（符号调试器）中也会遇到同样的问题，同样是因为这个名字可能并没有被加入 symbol table（符号表）。

解决方案是用 constant（常量）来取代 macro（宏）：

`const double AspectRatio = 1.653;`  

> 大写名称通常用于宏，因此更改了名称

作为一个语言常量，AspectRatio 肯定会被编译器看到，当然就会进入记号表内。此外对浮点常量(floating point constant,就像本例)而言，使用常量可能比使用#define导致较小量的码，因为预处理器 ”盲目地将宏名称ASPECT RATIO替换为1.653” 可能导致目标码(object code)出现多份1.653，若改用常量AspectRatio绝不会出现相同情况。