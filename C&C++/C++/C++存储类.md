## auto

这是默认的存储类说明符，通常可以省略不写。auto 指定的变量具有自动存储期，即它们的生命周期仅限于定义它们的块（block）。auto 变量通常在栈上分配。

C++98标准中auto关键字用于自动变量的声明，但由于使用极少且多余，在C++11中已删除这一用法。

```C++
auto f=3.14;      //double
auto s("hello");  //const char*
auto z = new auto(9); // int*
auto x1 = 5, x2 = 5.0, x3='r';//错误，必须是初始化为同一类型
```

[auto (C++) | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/auto-cpp?view=msvc-170)



## register

用于声明变量，并提示编译器将这些变量存储在寄存器中，以便快速访问。

使用 register 关键字可以提高程序的执行速度，因为它减少了对内存的访问次数。

然而，需要注意的是，register 存储类只是一种提示，编译器可以忽略它，因为现代的编译器通常会自动优化代码，选择合适的存储位置。

用于定义存储在寄存器中而不是 RAM 中的局部变量。这意味着变量的最大尺寸等于寄存器的大小（通常是一个词），且不能对它应用一元的 '&' 运算符（因为它没有内存位置）。

**语法格式：**

```
register data_type variable_name;
```

- `register` 是存储类的关键字，用于提示编译器将变量存储在寄存器中。
- `data_type` 是变量的数据类型，可以是任何合法的 C++ 数据类型。
- `variable_name` 是变量的名称。

```c++
void loop() {
    register int i;
    for (i = 0; i < 1000; ++i) {
        // 循环体
    }
}
```

register 存储类用于提示编译器将变量存储在寄存器中，以便提高访问速度。然而，由于现代编译器的自动优化能力，使用 register 关键字并不是必需的，而且在实践中很少使用。

在 C++11 标准中，register 关键字不再是一个存储类说明符，而是一个废弃的特性。这意味着在 C++11 及以后的版本中，使用 register 关键字将不会对程序产生任何影响。

在 C++ 中，可以使用引用或指针来提高访问速度，尤其是在处理大型数据结构时。

```c++
register int y = 20; // 使用register存储类，但编译器可能会忽略

std::cout << "Value of y: " << y << std::endl;

// 注意：register关键字在现代C++中并不常用，因为编译器会自动优化
```



## static

用于定义具有静态存储期的变量或函数，它们的生命周期贯穿整个程序的运行期。在函数内部，static变量的值在函数调用之间保持不变。在文件内部或全局作用域，static变量具有内部链接，只能在定义它们的文件中访问。



**static**存储类指示编译器在程序而不是创建和每次进入和进入的作用域之时摧毁它的寿命时间内保持一个局部变量存在。因此，将局部变量设为静态可以使它们在函数调用之间保持其值。static饰符也可以应用于全局变量。完成此操作后，它将导致该变量的作用域仅限于声明该变量的文件。

在C++中，当对类数据成员使用static时，它将导致该类的所有对象仅共享该成员的一个副本。



**static** 存储类指示编译器在程序的生命周期内保持局部变量的存在，而不需要在每次它进入和离开作用域时进行创建和销毁。因此，使用 static 修饰局部变量可以在函数调用之间保持局部变量的值。

static 修饰符也可以应用于全局变量。当 static 修饰全局变量时，会使变量的作用域限制在声明它的文件内。

在 C++ 中，当 static 用在类数据成员上时，会导致仅有一个该成员的副本被类的所有对象共享。



**`static`** 关键字可用于在全局范围、命名空间范围和类范围声明变量和函数。 静态变量还可在本地范围声明。

静态持续时间意味着，在程序启动时分配对象或变量，并在程序结束时释放对象或变量。 外部链接意味着，变量的名称在用于声明变量的文件的外部是可见的。 相反，内部链接意味着，名称在用于声明变量的文件的外部是不可见的。 默认情况下，在全局命名空间中定义的对象或变量具有静态持续时间和外部链接。 在以下情况下，可使用 **`static`** 关键字。

1. 在文件范围（全局和/或命名空间范围）内声明变量或函数时，**`static`** 关键字将指定变量或函数具有内部链接。 在声明变量时，变量具有静态持续时间，并且除非您指定另一个值，否则编译器会将变量初始化为 0。
2. 在函数中声明变量时，**`static`** 关键字将指定变量将在对该函数的调用中保持其状态。
3. 在类声明中声明数据成员时，**`static`** 关键字将指定类的所有实例共享该成员的一个副本。 必须在文件范围内定义 `static` 数据成员。 声明为 **`const static`** 的整型数据成员可以有初始化表达式。
4. 在类声明中声明成员函数时，**`static`** 关键字将指定类的所有实例共享该函数。 由于函数没有隐式 **`this`** 指针，因此 `static` 成员函数不能访问实例成员。 若要访问实例成员，请使用作为实例指针或引用的参数来声明函数。
5. 不能将 `union` 的成员声明为 **`static`**。 但是，全局声明的匿名 `union` 必须是显式声明的 **`static`**。

```C++
#include <iostream>
 
// 函数声明 
void func(void);
 
static int count = 10; /* 全局变量 */
 
int main()
{
    while(count--)
    {
       func();
    }
    return 0;
}
// 函数定义
void func( void )
{
    static int i = 5; // 局部静态变量
    i++;
    std::cout << "变量 i 为 " << i ;
    std::cout << " , 变量 count 为 " << count << std::endl;
}
```



此示例说明了函数中声明的变量 **`static`** 如何在对该函数的调用间保持其状态。

```cpp
// static1.cpp
// compile with: /EHsc
#include <iostream>

using namespace std;
void showstat( int curr ) {
   static int nStatic;    // 保留nStatic的值
                          // 在每个函数调用之间
   nStatic += curr;
   cout << "nStatic is " << nStatic << endl;
}

int main() {
   for ( int i = 0; i < 5; i++ )
      showstat( i );
}
nStatic is 0
nStatic is 1
nStatic is 3
nStatic is 6
nStatic is 10
```

此示例说明了 **`static`** 在类中的用法。

```cpp
// static2.cpp
// compile with: /EHsc
#include <iostream>

using namespace std;
class CMyClass {
public:
   static int m_i;
};

int CMyClass::m_i = 0;
CMyClass myObject1;
CMyClass myObject2;

int main() {
   cout << myObject1.m_i << endl;
   cout << myObject2.m_i << endl;

   myObject1.m_i = 1;
   cout << myObject1.m_i << endl;
   cout << myObject2.m_i << endl;

   myObject2.m_i = 2;
   cout << myObject1.m_i << endl;
   cout << myObject2.m_i << endl;

   CMyClass::m_i = 3;
   cout << myObject1.m_i << endl;
   cout << myObject2.m_i << endl;
}
0
0
1
1
2
2
3
3
```

以下示例显示了成员函数中声明的局部变量 **`static`**。 **`static`** 变量对整个程序可用；该类型的所有实例共享 **`static`** 变量的同一副本。

```cpp
// static3.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;
struct C {
   void Test(int value) {
      static int var = 0;
      if (var == value)
         cout << "var == value" << endl;
      else
         cout << "var != value" << endl;

      var = value;
   }
};

int main() {
   C c1;
   C c2;
   c1.Test(100);
   c2.Test(100);
}
var != value
var == value
```

从 C++11 开始，可以保证 **`static`** 局部变量初始化是线程安全的。 此功能有时称为神奇的静态对象。 但是，在多线程应用程序中，必须同步所有后续分配。 可以通过使用 [`/Zc:threadSafeInit-`](https://learn.microsoft.com/zh-cn/cpp/build/reference/zc-threadsafeinit-thread-safe-local-static-initialization?view=msvc-170) 标志避免对 CRT 形成依赖，来禁用线程安全的静态初始化功能。



## extern 

**extern** 存储类用于提供一个全局变量的引用，全局变量对所有的程序文件都是可见的。当您使用 'extern' 时，对于无法初始化的变量，会把变量名指向一个之前定义过的存储位置。

当您有多个文件且定义了一个可以在其他文件中使用的全局变量或函数时，可以在其他文件中使用 *extern* 来得到已定义的变量或函数的引用。可以这么理解，*extern* 是用来在另一个文件中声明一个全局变量或函数



**xtern**存储类是用来给一个全局变量，那就是所有程序文件可见的参考。当您使用“extern”时，变量无法初始化，因为将变量名指向先前已定义的存储位置即可。当您有多个文件并且定义了一个全局变量或函数时，也将在其他文件中使用该变量或函数，然后在另一个文件中使用extern来引用已定义的变量或函数。只是为了理解，extern用于在另一个文件中声明全局变量或函数。



extern 修饰符通常用于当有两个或多个文件共享相同的全局变量或函数的时候，如下所示：

第一个文件：main.cpp

实例

```c++
#include <iostream>
 
int count ;
extern void write_extern();
 
int main()
{
   count = 5;
   write_extern();
}
```

第二个文件：support.cpp

实例

```c++
#include <iostream>
 
extern int count;
 
void write_extern(void)
{
   std::cout << "Count is " << count << std::endl;
}
```

在这里，第二个文件中的 *extern* 关键字用于声明已经在第一个文件 main.cpp 中定义的 count。现在 ，编译这两个文件，如下所示：

```
$ g++ main.cpp support.cpp -o write
```

这会产生 **write** 可执行程序，尝试执行 **write**，它会产生下列结果：

```
$ ./write
Count is 5
```



关键字 **`extern`** 可以应用于全局变量、函数或模板声明。 它指定符号具有 *external 链接*。 有关链接的背景信息以及为何不鼓励使用全局变量，请参阅[翻译单元和链接](https://learn.microsoft.com/zh-cn/cpp/cpp/program-and-linkage-cpp?view=msvc-170)。

关键字 **`extern`** 具有四种含义，具体取决于上下文：

- 在非 **`const`** 全局变量声明中，**`extern`** 指定变量或函数在另一个转换单元中定义。 必须在除定义变量的文件之外的所有文件中应用 **`extern`**。
- 在 **`const`** 变量声明中，它指定变量具有 external 链接。 **`extern`** 必须应用于所有文件中的所有声明。 （默认情况下，全局 **`const`** 变量具有内部链接。）
- **`extern "C"`** 指定函数在别处定义并使用 C 语言调用约定。 `extern "C"` 修饰符也可以应用于块中的多个函数声明。
- 在模板声明中，**`extern`** 指定模板已在其他位置实例化。 **`extern`** 告知编译器它可以重复使用另一个实例化，而不是在当前位置创建新实例。 有关使用 **`extern`** 的详细信息，请参阅[显式实例化](https://learn.microsoft.com/zh-cn/cpp/cpp/explicit-instantiation?view=msvc-170)。



### `extern` 非 `const` 全局链接

链接器在全局变量声明之前看到 **`extern`** 时，它会在另一个转换单元中查找定义。 默认情况下，全局范围内的非 **`const`** 变量声明为 external。 仅将 **`extern`** 用于未提供定义的声明。

```cpp
//fileA.cpp
int i = 42; // declaration and definition

//fileB.cpp
extern int i;  // declaration only. same as i in FileA

//fileC.cpp
extern int i;  // declaration only. same as i in FileA

//fileD.cpp
int i = 43; // LNK2005! 'i' already has a definition.
extern int i = 43; // same error (extern is ignored on definitions)
```



### `extern` 全局 `const` 链接

默认情况下，**`const`** 全局变量具有内部链接。 如果希望变量具有 external 链接，请将 **`extern`** 关键字应用于定义以及其他文件中的所有其他声明：

```cpp
//fileA.cpp
extern const int i = 42; // extern const definition

//fileB.cpp
extern const int i;  // declaration only. same as i in FileA
```



### `extern constexpr` 链接

在 Visual Studio 2017 版本 15.3 或更早版本中，编译器总是提供 **`constexpr`** 变量内部链接，即使在变量标记为 **`extern`** 时也是如此。 在 Visual Studio 2017 版本 15.5 或更高版本中，[`/Zc:externConstexpr`](https://learn.microsoft.com/zh-cn/cpp/build/reference/zc-externconstexpr?view=msvc-170) 编译器开关启用正确且符合标准的行为。 选项最终将成为默认设置。 [`/permissive-`](https://learn.microsoft.com/zh-cn/cpp/build/reference/permissive-standards-conformance?view=msvc-170) 选项不启用 **`/Zc:externConstexpr`**。

```cpp
extern constexpr int x = 10; //error LNK2005: "int const x" already defined
```

如果头文件包含声明 **`extern`** **`constexpr`** 的变量，必须将它标记为 `__declspec(selectany)`，以便正确组合其重复声明：

```cpp
extern constexpr __declspec(selectany) int x = 10;
```



### `extern "C"` 和 `extern "C++"` 函数声明

在 C++ 中，与字符串一起使用时，**`extern`** 指定其他语言的链接约定将用于声明符。 仅在之前被声明为具有 C 链接的情况下，才能访问 C 函数和数据。 但是，必须在单独编译的翻译单元中定义它们。

Microsoft C++ 支持 *string-literal* 字段中的字符串 **`"C"`** 和 **`"C++"`**。 所有标准包含文件都使用 **`extern "C"`** 语法以允许运行时库函数用于 C++ 程序。



### 示例

以下示例演示如何声明具有 C 链接的名称：

```cpp
// Declare printf with C linkage.
extern "C" int printf(const char *fmt, ...);

//  Cause everything in the specified
//  header files to have C linkage.
extern "C" {
    // add your #include statements here
#include <stdio.h>
}

//  Declare the two functions ShowChar
//  and GetChar with C linkage.
extern "C" {
    char ShowChar(char ch);
    char GetChar(void);
}

//  Define the two functions
//  ShowChar and GetChar with C linkage.
extern "C" char ShowChar(char ch) {
    putchar(ch);
    return ch;
}

extern "C" char GetChar(void) {
    char ch;
    ch = getchar();
    return ch;
}

// Declare a global variable, errno, with C linkage.
extern "C" int errno;
```

如果一个函数具有多个链接规范，这些规范必须统一。 声明函数同时具有 C 和 C++ 链接是错误的。 此外，如果一个函数的两个声明出现在一个程序中，并且它们一个有链接规范，另一个没有，则有链接规范的声明必须是第一个。 将为已具有链接规范的函数的所有冗余声明提供第一个声明中指定的链接。 例如：

```cpp
extern "C" int CFunc1();
...
int CFunc1();            // Redeclaration is benign; C linkage is
                         //  retained.

int CFunc2();
...
extern "C" int CFunc2(); // Error: not the first declaration of
                         //  CFunc2;  cannot contain linkage
                         //  specifier.
```

从 Visual Studio 2019 开始，当指定 **`/permissive-`** 时，编译器会检查 `extern "C"` 函数参数的声明是否也匹配。 不能重载声明为 `extern "C"` 的函数。 从 Visual Studio 2019 版本 16.3 开始，可以使用 **`/permissive-`** 选项后面的 [`/Zc:externC-`](https://learn.microsoft.com/zh-cn/cpp/build/reference/zc-externc?view=msvc-170) 编译器选项替代此检查。

[extern (C++) | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/extern-cpp?view=msvc-170)



## thread_local 

使用 thread_local 说明符声明的变量仅可在它在其上创建的线程上访问。 变量在创建线程时创建，并在销毁线程时销毁。 每个线程都有其自己的变量副本。

thread_local 说明符可以与 static 或 extern 合并。

可以将 thread_local 仅应用于数据声明和定义，thread_local 不能用于函数声明或定义。

以下演示了可以被声明为 thread_local 的变量：

```c++
thread_local int x;  // 命名空间下的全局变量
class X
{
    static thread_local std::string s; // 类的static成员变量
};
static thread_local std::string X::s;  // X::s 是需要定义的
 
void foo()
{
    thread_local std::vector<int> v;  // 本地变量
}
```



使用 **`thread_local`** 说明符声明的变量仅可在它在其上创建的线程上访问。 变量在创建线程时创建，并在销毁线程时销毁。 每个线程都有其自己的变量副本。 在 Windows 上，**`thread_local`** 在功能上等效于特定于 Microsoft 的 [`__declspec( thread )`](https://learn.microsoft.com/zh-cn/cpp/cpp/thread?view=msvc-150) 属性。

```c++
thread_local float f = 42.0; // 全局命名空间。不是隐式静态的。

struct S // 不能应用于类型定义
{
    thread_local int i; //非法的。成员必须是静态的。
    thread_local static char buf[10]; // OK
};

void DoSomething()
{
    //将thread_local应用于局部变量。
//隐式地"thread_local static S my_struct"。
    thread_local S my_struct;
}
```

有关 **`thread_local`** 说明符的注意事项：

- DLL 中动态初始化的线程局部变量可能无法在所有调用线程上正确初始化。 有关详细信息，请参阅 [`thread`](https://learn.microsoft.com/zh-cn/cpp/cpp/thread?view=msvc-150)。
- **`thread_local`** 说明符可以与 **`static`** 或 **`extern`** 合并。
- 可以将 **`thread_local`** 仅应用于数据声明和定义；**`thread_local`** 不能用于函数声明或定义。
- 只能在具有静态存储持续时间的数据项上指定 **`thread_local`**，其中包括全局数据对象（**`static`** 和 **`extern`**）、局部静态对象和类的静态数据成员。 如果未提供其他存储类，则声明 **`thread_local`** 的任何局部变量都为隐式静态；换句话说，在块范围内，**`thread_local`** 等效于 **`thread_local static`**。
- 必须为线程本地对象的声明和定义指定 **`thread_local`**，无论声明和定义是在同一文件中发生还是在单独的文件中发生。
- 不建议将 **`thread_local`** 变量与 `std::launch::async` 一起使用。 有关详细信息，请参阅 [`` 函数](https://learn.microsoft.com/zh-cn/cpp/standard-library/future-functions?view=msvc-150)。

在 Windows 上，**`thread_local`** 在功能上等效于 [`__declspec(thread)`](https://learn.microsoft.com/zh-cn/cpp/cpp/thread?view=msvc-150)，但 _*`\*__declspec(thread)`** 可以应用于类型定义并且在 C 代码中有效。 请尽可能使用 **`thread_local`**，因为它是 C++ 标准的一部分，因此更易于移植。

[C++11 thread\_local 用法 | 拾荒志](https://murphypei.github.io/blog/2020/02/thread-local)

[thread\_local的一些用法 - 白伟碧一些小心得 - 博客园](https://www.cnblogs.com/bwbfight/p/18025752)



