关键字 **`constexpr`** 是在 C++11 中引入的，并在 C++14 中进行了改进。 它表示 constant（常数）表达式。 与 **`const`** 一样，它可以应用于变量：如果任何代码试图 modify（修改）该值，将引发编译器错误。 与 **`const`** 不同，**`constexpr`** 也可以应用于函数和类 constructor（构造函数）。 **`constexpr`** 指示值或返回值是 constant（常数），如果可能，将在编译时进行计算。

每当需要 `const` 整数时（如在模板自变量和数组声明中），都可以使用 **`constexpr`** 整数值。 如果在编译时（而非运行时）计算某个值，它可以使程序运行速度更快、占用内存更少。

为了限制计算编译时 constant（常数）的复杂性及其对编译时间的潜在影响，C++14 标准要求 constant（常数）表达式中所涉及的类型为[文本类型](https://learn.microsoft.com/zh-cn/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-140#literal_types)。

## 参数

*`params`*
一个或多个参数，每个参数都必须是文本类型，并且本身必须是 constant（常数）表达式。

## 返回值

**`constexpr`** 变量或函数必须返回[文本类型](https://learn.microsoft.com/zh-cn/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-140#literal_types)。

##  `constexpr` 变量

**`const`** 与 **`constexpr`** 变量之间的主要 difference（区别）是，**`const`** 变量的初始化可以推迟到运行时进行。 **`constexpr`** 变量必须在编译时进行初始化。 所有的 **`constexpr`** 变量都是 **`const`**。

- 如果一个变量具有文本类型并且已初始化，则可以使用 **`constexpr`** 声明该变量。 如果初始化是由 constructor（构造函数）performed（执行）的，则必须将 constructor（构造函数）声明为 **`constexpr`**。
- 当满足以下两个条件时，引用可以被声明为 **`constexpr`**：引用的对象是由 constant（常数）常数表达式初始化的，初始化期间调用的任何隐式转换也是 constant（常数）表达式。
- **`constexpr`** 变量或函数的所有声明都必须具有 **`constexpr`** specifier（说明符）。



```c++
constexpr float x = 42.0;
constexpr float y{108};
constexpr float z = exp(5, 3);
constexpr int i; // 错误!没有初始化
int j = 0;
constexpr int k = j + 1; //错误!j 不是一个常数表达式
```



## `constexpr` 函数

**`constexpr`** 函数是在使用需要它的代码时，可在编译时计算其返回值的函数。 使用代码需要编译时的返回值来初始化 **`constexpr`** 变量，或者用于提供非类型模板自变量。 当其自变量为 **`constexpr`** 值时，函数 **`constexpr`** 将生成编译时 constant（常数）。 使用非 **`constexpr`** 自变量调用时，或者编译时不需要其值时，它将与正则函数一样，在运行时生成一个值。 （此双重行为使你无需编写同一函数的 **`constexpr`** 和非 **`constexpr`** 版本。）

**`constexpr`** 函数或 constructor（构造函数）通过隐式方式 **`inline`**。

以下规则适用于 `constexpr` 函数：

- **`constexpr`** 函数必须只接受并返回[文本类型](https://learn.microsoft.com/zh-cn/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-140#literal_types)。
- **`constexpr`** 函数可以是递归的。
- 可以是fore C++20，函数**`constexpr`**不能是[虚拟](https://learn.microsoft.com/zh-cn/cpp/cpp/virtual-cpp?view=msvc-140)的，当封闭类具有任何虚拟基类时，不能定义`const`一个 **`constexpr`**ructor。 在 C++20 及更高版本中，**`constexpr`** 函数可以是虚函数。 指定 [`/std:c++20`](https://learn.microsoft.com/zh-cn/cpp/build/reference/std-specify-language-standard-version?view=msvc-140) 或更高版本的编译器选项时，Visual Studio 2019 版本 16.10 及更高版本支持 **`constexpr`** 虚函数。
- 主体可以定义为 `= default` 或 `= delete`。
- 正文不能包含如何 **`goto`** 语句或 **`try`** 块。
- 可以将非 **`constexpr`** 模板的显式专用化声明为 **`constexpr`**：
- **`constexpr`** 模板的显式专用化不需要同时也是 **`constexpr`**：

以下规则适用于 Visual Studio 2017 及更高版本中的 **`constexpr`** 函数：

- 它可以包含 **`if`** 和 **`switch`** 语句，以及所有循环语句，包括 **`for`**、基于范围的 **`for`**、**`while`**、和 do-while。
- 它可能包含局部变量声明，但必须初始化该变量。 它必须是文本类型，不能是 **`static`** 或线程本地的。 本地声明的变量不需要是 **`const`**，并且可以变化。
- **`constexpr`** 非 **`static`** 成员函数不需要通过隐式方式 **`const`**。



```c++
constexpr float exp(float x, int n)
{
    return n == 0 ? 1 :
        n % 2 == 0 ? exp(x * x, n / 2) :
        exp(x * x, (n - 1) / 2) * x;
}
```



> 在 Visual Studio 调试器中，在调试非优化调试版本时，可以看出 **`constexpr`** 函数是否是通过在其内部放置一个断点在编译时计算的。 如果命中该断点，则在运行时调用该函数。 如果未命中，则在编译时调用该函数。



##  extern constexpr

[/Zc:externConstexpr](https://learn.microsoft.com/zh-cn/cpp/build/reference/zc-externconstexpr?view=msvc-140) 编译器选项使编译器将[外部链接](https://learn.microsoft.com/zh-cn/cpp/c-language/external-linkage?view=msvc-140)应用于使用 extern constexpr 声明的变量。 在早期版本的 Visual Studio 中，默认情况下或者 specified（指定）了 /Zc:externConstexpr- 时，即使使用关键字 **`extern`**，Visual Studio 也会将内部链接应用于 **`constexpr`** 变量。 从 Visual Studio 2017 Update 15.6 开始，可以使用 /Zc:externConstexpr 选项，默认情况下它处于关闭状态。 [/permissive-](https://learn.microsoft.com/zh-cn/cpp/build/reference/permissive-standards-conformance?view=msvc-140) 选项不会启用 /Zc:externConstexpr。



## 示例

下面的示例展示了 **`constexpr`** 变量、函数和用户定义类型。 在 `main()` 的最后一个语句中，**`constexpr`** 成员函数 `GetValue()` 是一个运行时调用，因为不需要在编译时知道该值。

```c++
// constexpr.cpp
// Compile with: cl /EHsc /W4 constexpr.cpp
#include <iostream>

using namespace std;

// Pass by value
constexpr float exp(float x, int n)
{
    return n == 0 ? 1 :
        n % 2 == 0 ? exp(x * x, n / 2) :
        exp(x * x, (n - 1) / 2) * x;
}

// Pass by reference
constexpr float exp2(const float& x, const int& n)
{
    return n == 0 ? 1 :
        n % 2 == 0 ? exp2(x * x, n / 2) :
        exp2(x * x, (n - 1) / 2) * x;
}

// Compile-time computation of array length
template<typename T, int N>
constexpr int length(const T(&)[N])
{
    return N;
}

// Recursive constexpr function
constexpr int fac(int n)
{
    return n == 1 ? 1 : n * fac(n - 1);
}

// User-defined type
class Foo
{
public:
    constexpr explicit Foo(int i) : _i(i) {}
    constexpr int GetValue() const
    {
        return _i;
    }
private:
    int _i;
};

int main()
{
    // foo is const:
    constexpr Foo foo(5);
    // foo = Foo(6); //Error!

    // Compile time:
    constexpr float x = exp(5, 3);
    constexpr float y { exp(2, 5) };
    constexpr int val = foo.GetValue();
    constexpr int f5 = fac(5);
    const int nums[] { 1, 2, 3, 4 };
    const int nums2[length(nums) * 2] { 1, 2, 3, 4, 5, 6, 7, 8 };

    // Run time:
    cout << "The value of foo is " << foo.GetValue() << endl;
}
```



> 以上来自文章：[constexpr (C++) | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/constexpr-cpp?view=msvc-170)

***

## constexpr 的其他文章解析



> 测试环境：windows10 + gcc8.1 1、constexpr产生背景 c++11以后，为了保证写出的代码比以往任何时候的执行效率都要好而进行了许多改善。其中，这种改善之一就是生成常量表达式，允许程序利用编译时的计算能力。常量表达式主要是允许一些计算发生在编译时期，而不是运行时期。这是一个很


测试环境：windows10 + gcc8.1

### 1、constexpr 产生背景



c++11 以后，为了保证写出的代码比以往任何时候的执行效率都要好而进行了许多改善。其中，这种改善之一就是生成常量表达式，允许程序利用编译时的计算能力。常量表达式主要是允许一些计算发生在编译时期，而不是运行时期。这是一个很进步的优化：假如有些事情可以在编译时计算，它将只计算一次，而不是在运行时每一次都进行计算。需要计算一个编译时已知的常量，比如特定的 sin 或者 cos 值，确实可以使用库函数 sin 和 cos，但那样做是必须花费运行时的开销。此时可以使用 constexpr 创建一个编译时的函数，它将在编译时期计算出你需要的数值，而在用户的电脑上将无需做这些工作。

### 2、constexpr 用法

为了使函数获取编译时计算的能力，必须给该函数指定 constexpr 关键字。

```
constexpr int multiply(int x,int y)
{
    return x* y;
}
//将在编译时期计算
const int var = multiply(10,10);
```

除了编译时计算性能的优化，congtexpr 的另外一个优势是：允许函数被应用到以前调用宏的所有场合。例如：想要计算数组 size 的函数，size 是 10 的倍数。如果不用 constexpr，则需要创建一个宏或者模板，因为我们不能用函数的返回值去声明数组的大小。但是我们可以调用一个 constexpr 函数去声明一个数组。

```
constexpr int getDefaultArraySize(int value)
{
    return value*10;
}
int my_array[getDefaultArraySize(3)];
```

### 3、constexpr 使用限制

c++11 中的 constexpr 指定的函数返回值和参数都必须保证是字面值，而且必须有且只有一行代码（return 代码）。所以通常只能通过 return 三目运算符 + 递归来计算返回的字面值。

```
constexpr int factorial (int n)
{
    return n &gt; 0 ? n * factorial( n - 1 ) : 1;
}
```

c++14 中则只要保证返回值和参数是字面值就行，函数体中可以加入更多的语句，实现了更灵活的计算。

```
// C++14
constexpr int factorial2(int n)
{
    int result = 1;
    for (int i = 1; i &lt;= n; ++i)
        result *= i;
    return result;
}
```

c++17 中 lambda 表达式可以被声明为 constexpr。对于一个 lambda 而言，只要被捕获的变量是字面量类型（lieteral type），那么整个 lambda 也将表现为字面量类型。

```
//显示声明为constexpr类型
template &lt;typename T&gt;
constexpr auto addTo(T i){
    return[i](auto j){return i+j;};
}
constexpr auto add5 = addTo(5);
```

当一个闭包再 constexpr 环境下被使用时，当它满足了 constexpr 的条件，则无论它有没有被显示地声明为 constexpr，它仍然是 constexpr 的。

```
 //这里没有显式声明为constexpr，但依然可以表现为constexpr
 auto answer = [](int n)
 {
   return 32 + n;
 };
 //在一个constexpr环境中被使用
 constexpr int response = answer(10);
```

在 c++17 中 contexpr if 让以前理应被写在一起的代码，却在 c++17 前都没法被写在一起的情况得到了改善。传统的 if-else 语句是在执行期进行条件判断与选择的，因而在泛型编程中，无法使用 if-else 语句进行判断。在 c++17 中，我们可以在编译期对传统的条件语句做出相应判断，可以忽略那些完全没有被进入的语句。

注意，在老的标准中，计算使用了 if，另一个分支也仍然会被编译，但在 c++17 中，如果使用 if constexpr 来替代 if，编译器甚至会把编译无效条件这个过程忽略掉。

constexpr 对 STL 库标准做出的改进：

以前在标准库中，有许多类型和函数都缺乏了 constexpr 的特性，这些问题在 C++17 中都相应做了改进。最著名的就是 std::array 以及用于范围获取的 std::begin() 和 std::end()。



***



> constexpr 是 C++11 引入的一个关键字，用于声明可以在编译时进行计算的常量表达式。随后的标准（C++14, C++17, C++20）对 constexpr 功能进行了扩展，增加了更多的

`constexpr` 是 C++11 引入的一个关键字，用于声明可以在编译时进行计算的常量表达式。随后的标准（C++14, C++17, C++20）对 `constexpr` 功能进行了扩展，增加了更多的使用场景和能力。下面是 `constexpr` 的主要用法及相应的示例代码。

### 1\. 常量表达式变量

在 C++11 中，`constexpr` 可以用于声明变量，确保该变量的值在编译时是已知的。

```
constexpr int max_size = 100;
constexpr double pi = 3.14159;

```

### 2\. 函数中的 `constexpr`

C++11 允许在函数中使用 `constexpr`，但要求函数体必须足够简单（通常是单个 return 语句）。

```
constexpr int square(int x) {
    return x * x;
}

```

### 3\. C++14 中的扩展

在 C++14 中，`constexpr` 函数的限制被放宽，允许更复杂的逻辑，如局部变量和循环。

```
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i &lt;= n; ++i) {
        result *= i;
    }
    return result;
}

```

### 4\. `constexpr` 构造函数和类

C++11 允许类构造函数为 `constexpr`，使得可以在编译时创建和初始化类的对象。

```
class Point {
public:
    constexpr Point(double xVal, double yVal) : x(xVal), y(yVal) {}
    constexpr double getX() const { return x; }
    constexpr double getY() const { return y; }

private:
    double x, y;
};

constexpr Point origin(0, 0);

```

### 5\. `constexpr` if 语句（C++17）

C++17 引入了 `constexpr if`，允许在编译时根据条件进行分支。

```
template&lt;typename T&gt;
constexpr auto get_value(T t) {
    if constexpr (std::is_pointer&lt;T&gt;::value) {
        return *t;  
    } else {
        return t;   
    }
}

```

### 6\. `constexpr` lambda 表达式（C++17）

C++17 允许 lambda 表达式为 `constexpr`。

```
constexpr auto square_lambda = [](int n) {
    return n * n;
};

```

### 7\. `constexpr` 的动态分配（C++20）

C++20 允许 `constexpr` 中使用动态内存分配。

```
constexpr auto create_array(int n) {
    return std::to_array({1, 2, 3, 4, 5});  
}

```

以上是 `constexpr` 的主要用法。由于 `constexpr` 随着 C++ 标准的发展而不断扩展，可能还有一些边缘用法未被涵盖。但这些是最常见和最实用的场景。



