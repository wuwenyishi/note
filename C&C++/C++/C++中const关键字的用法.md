## `const` 值

**`const`** 关键字指定变量的值是常量并通知编译器防止程序员对其进行修改。

```c++
// constant_values1.cpp
int main() {
   const int i = 5;
   i = 10;   // C3892
   i++;   // C2105
}
```

在 C++ 中，可以使用 **`const`** 关键字而不是 [`#define`](https://learn.microsoft.com/zh-cn/cpp/preprocessor/hash-define-directive-c-cpp?view=msvc-140) 预处理器指令来定义常量值。 使用 **`const`** 定义的值需要接受类型检查，并可以替代常量表达式。 在 C++ 中，可以使用 **`const`** 变量指定数组的大小，如下所示：

```c++
// constant_values2.cpp
// compile with: /c
const int maxarray = 255;
char store_char[maxarray];  // allowed in C++; not allowed in C
```

在 C 中，常量值默认为外部链接，因此它们只能出现在源文件中。 在 C++ 中，常量值默认为内部链接，这使它们可以出现在标头文件中。

**`const`** 关键字还可在指针声明中使用。

```c++
// constant_values3.cpp
int main() {
   char this_char{'a'}, that_char{'b'};
   char *mybuf = &this_char, *yourbuf = &that_char;
   char *const aptr = mybuf;
   *aptr = 'c';   // OK
   aptr = yourbuf;   // C3892
}
```

指向声明为 **`const`** 的变量的指针只能赋给也声明为 **`const`** 的指针。

```c++
// constant_values4.cpp
#include <stdio.h>
int main() {
   const char *mybuf = "test";
   char *yourbuf = "test2";
   printf_s("%s\n", mybuf);

   const char *bptr = mybuf;   // Pointer to constant data
   printf_s("%s\n", bptr);

   // *bptr = 'a';   // Error
}
```

可以将指向常量数据的指针用作函数参数，以防止函数修改通过指针传递的参数。

对于声明为 **`const`** 的对象，只能调用常量成员函数。 编译器确保从不修改常量对象。

```c++
birthday.getMonth();    // Okay
birthday.setMonth( 4 ); // Error
```

可以为非常量对象调用常量或非常量成员函数。 还可以使用 **`const`** 关键字重载成员函数；这使得可以对常量和非常量对象调用不同版本的函数。

不能使用 **`const`** 关键词声明构造函数或析构函数。



## `const` 成员函数

声明带 **`const`** 关键字的成员函数将指定该函数是一个“只读”函数，它不会修改为其调用该函数的对象。 常量成员函数不能修改任何非静态数据成员或调用不是常量的任何成员函数。 若要声明常量成员函数，请在参数列表的右括号后放置 **`const`** 关键字。 声明和定义中都需要 **`const`** 关键字。

```c++
// constant_member_function.cpp
class Date
{
public:
   Date( int mn, int dy, int yr );
   int getMonth() const;     // 只读功能
   void setMonth( int mn );   // 写函数;不能是const
private:
   int month;
};

int Date::getMonth() const
{
   return month;        // 不会改变任何东西
}
void Date::setMonth( int mn )
{
   month = mn;          // 修改数据成员
}
int main()
{
   Date MyDate( 7, 4, 1998 );
   const Date BirthDate( 1, 18, 1953 );
   MyDate.setMonth( 4 );    // Okay
   BirthDate.getMonth();    // Okay
   BirthDate.setMonth( 4 ); // C2662 Error
}
```

