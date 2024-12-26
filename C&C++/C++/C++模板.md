> 下面来自ChatGPT

C++ 模板是一种强大的编程工具，它允许你编写泛型代码，可以与不同的数据类型一起工作，而无需重复编写相似的代码。模板主要用于创建泛型类和函数，从而提高代码的可重用性和灵活性。

#### **函数模板**

函数模板用于编写可以处理不同数据类型的通用函数。例如，你可以编写一个排序算法、交换函数、或查找函数，无论传入的参数是什么类型，都可以正确处理。

**函数模板的定义：**

```cpp
template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}
```

在这个例子中，`T` 是一个模板参数，它表示一种类型。当你调用 `max` 函数时，编译器会根据你传入的参数类型推断 `T` 的实际类型。

**使用函数模板：**

```cpp
int a = 10, b = 20;
double x = 10.5, y = 20.5;

std::cout << "Max of a and b: " << max(a, b) << std::endl;
std::cout << "Max of x and y: " << max(x, y) << std::endl;
```

####  **类模板**

类模板允许你创建适用于任意数据类型的类。例如，你可以创建一个通用的堆栈（Stack）类，可以存储任何类型的元素。

**类模板的定义：**
```cpp
template <typename T>
class Stack {
private:
    std::vector<T> elems;  // 使用 vector 来存储元素

public:
    void push(T const&);   // 入栈操作
    void pop();            // 出栈操作
    T top() const;         // 获取栈顶元素
    bool empty() const {   // 判断栈是否为空
        return elems.empty();
    }
};

template <typename T>
void Stack<T>::push(T const& elem) {
    elems.push_back(elem);
}

template <typename T>
void Stack<T>::pop() {
    if (elems.empty()) {
        throw std::out_of_range("Stack<>::pop(): empty stack");
    }
    elems.pop_back();
}

template <typename T>
T Stack<T>::top() const {
    if (elems.empty()) {
        throw std::out_of_range("Stack<>::top(): empty stack");
    }
    return elems.back();
}
```

**使用类模板：**
```cpp
Stack<int> intStack;    // 创建存储 int 类型的栈
intStack.push(7);
std::cout << intStack.top() << std::endl;

Stack<std::string> stringStack;  // 创建存储 string 类型的栈
stringStack.push("hello");
std::cout << stringStack.top() << std::endl;
```

####  **模板特化**

有时，你需要为特定的数据类型定义一个特殊版本的模板。C++ 支持模板的“特化”（Specialization），包括完全特化和部分特化。

**完全特化：**

```cpp
template <>
class Stack<bool> {
    // 为 bool 类型的 Stack 实现特殊化版本
};
```

**部分特化：**

```cpp
template <typename T>
class Stack<std::shared_ptr<T>> {
    // 为 shared_ptr 类型的 Stack 实现部分特化
};
```

#### 模板的高级用法

- **模板递归**：常用于实现编译时计算，例如 Fibonacci 数列或阶乘。
- **变参模板（Variadic Templates）**：用于处理任意数量的模板参数。
- **SFINAE（Substitution Failure Is Not An Error）**：编译器在替换模板参数时，失败不会导致错误，而是继续尝试其他模板匹配。

#### **模板的编译和实例化**

模板在使用时不会立即编译，而是在实例化时才会进行编译。实例化是指编译器为特定的类型替换模板参数并生成代码的过程。这种特性使得模板可以与任何兼容的类型一起使用，但也意味着错误可能在实例化时才会被发现。

#### 总结

C++ 模板是一种非常强大的工具，用于编写通用代码，减少代码重复，提高灵活性。理解和掌握模板，特别是函数模板、类模板，以及特化和高级模板技术，对于编写高效且可扩展的 C++ 程序至关重要。



***

> 下面来自: [模板 (C++) | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/templates-cpp?view=msvc-170)



模板是 C++ 中的泛型编程的基础。 作为强类型语言，C++ 要求所有变量都具有特定类型，由程序员显式声明或编译器推导。 但是，许多数据结构和算法无论在哪种类型上操作，看起来都是相同的。 使用模板可以定义类或函数的操作，并让用户指定这些操作应处理的具体类型。

### 定义和使用模板

模板是基于用户为模板参数提供的参数在编译时生成普通类型或函数的构造。 例如，可以定义如下所示的函数模板：

```c++
template <typename T>
T minimum(const T& lhs, const T& rhs)
{
    return lhs < rhs ? lhs : rhs;
}
```

上面的代码描述了一个具有单个类型参数 T 的泛型函数的模板，其返回值和调用参数（lhs 和 rhs）都具有此类型。 可以随意命名类型参数，但按照约定，最常使用单个大写字母。 T 是模板参数；关键字 **`typename`** 表示此参数是类型的占位符。 调用函数时，编译器会将每个 `T` 实例替换为由用户指定或编译器推导的具体类型参数。 编译器从模板生成类或函数的过程称为“模板实例化”；`minimum<int>` 是模板 `minimum<T>` 的实例化。

在其他地方，用户可以声明专用于 int 的模板实例。假设 get_a() 和 get_b() 是返回 int 的函数：

```c++
int a = get_a();
int b = get_b();
int i = minimum<int>(a, b);
```

但是，由于这是一个函数模板，编译器可以从参数 a 和 b 中推导出类型，因此可以像普通函数一样调用它：`T`

```c++
int i = minimum(a, b);
```

当编译器遇到最后一个语句时，它会生成一个新函数，在该函数中，T 在模板中的每个匹配项都替换为 **`int`**：

```c++
int minimum(const int& lhs, const int& rhs)
{
    return lhs < rhs ? lhs : rhs;
}
```

编译器如何在函数模板中执行类型推导的规则基于普通函数的规则。



### 类型参数

在上面的 `minimum` 模板中，请注意，在将类型参数 T 用于函数调用参数（在这些参数中会添加 const 和引用限定符）之前，不会以任何方式对其进行限定。

类型参数的数量没有实际限制。 以逗号分隔多个参数：

```c++
template <typename T, typename U, typename V> class Foo{};
```

在此上下文中，关键字 **`class`** 等效于 **`typename`**。 可以将前面的示例表示为：

```c++
template <class T, class U, class V> class Foo{};
```

可以使用省略号运算符 (...) 定义采用任意数量的零个或多个类型参数的模板：

```c++
template<typename... Arguments> class vtclass;

vtclass< > vtinstance1;
vtclass<int> vtinstance2;
vtclass<float, bool> vtinstance3;
```

任何内置类型或用户定义的类型都可以用作类型参数。 例如，可以使用标准库中的 [std::vector](https://learn.microsoft.com/zh-cn/cpp/standard-library/vector-class?view=msvc-170) 来存储 **`int`**、**`double`**、[std::string](https://learn.microsoft.com/zh-cn/cpp/standard-library/basic-string-class?view=msvc-170)、`MyClass`、**`const`** `MyClass`*、`MyClass&` 等类型的变量。 使用模板时的主要限制是类型参数必须支持应用于类型参数的任何操作。 例如，如果我们使用 `MyClass` 调用 `minimum`，如以下示例所示：

```c++
class MyClass
{
public:
    int num;
    std::wstring description;
};

int main()
{
    MyClass mc1 {1, L"hello"};
    MyClass mc2 {2, L"goodbye"};
    auto result = minimum(mc1, mc2); // Error! C2678
}
```

将生成编译器错误，因为 `MyClass` 不会为 **<** 运算符提供重载。

没有任何固有要求规定任何特定模板的类型参数都属于同一个对象层次结构，尽管可以定义强制实施此类限制的模板。 可以将面向对象的技巧与模板相结合；例如，可以将 Derived* 存储在向量 <Base*> 中。 请注意，自变量必须是指针

```c++
vector<MyClass*> vec;
MyDerived d(3, L"back again", time(0));
vec.push_back(&d);

// or more realistically:
vector<shared_ptr<MyClass>> vec2;
vec2.push_back(make_shared<MyDerived>());
```

`std::vector` 和其他标准库容器对 `T` 的元素施加的基本要求是 `T` 可复制且可复制构造。



### 非类型参数

与其他语言（如 C# 和 Java）中的泛型类型不同，C++ 模板支持非类型参数，也称为值参数。 例如，可以提供常量整型值来指定数组的长度，例如在以下示例中，它类似于标准库中的 [std::array](https://learn.microsoft.com/zh-cn/cpp/standard-library/array-class-stl?view=msvc-170) 类：

```c++
template<typename T, size_t L>
class MyArray
{
    T arr[L];
public:
    MyArray() { ... }
};
```

记下模板声明中的语法。 `size_t` 值在编译时作为模板参数传入，必须是 **`const`** 或 **`constexpr`** 表达式。 可以如下所示使用它：

```c++
MyArray<MyClass*, 10> arr;
```

其他类型的值（包括指针和引用）可以作为非类型参数传入。 例如，可以传入指向函数或函数对象的指针，以自定义模板代码中的某些操作。



### 非类型模板参数的类型推导

在 Visual Studio 2017 及更高版本中，在 **`/std:c++17`** 模式或更高版本中，编译器会推导使用 **`auto`** 声明的非类型模板参数的类型：

```c++
template <auto x> constexpr auto constant = x;

auto v1 = constant<5>;      // v1 == 5, decltype(v1) is int
auto v2 = constant<true>;   // v2 == true, decltype(v2) is bool
auto v3 = constant<'a'>;    // v3 == 'a', decltype(v3) is char
```



### 模板作为模板参数

模板可以是模板参数。 在此示例中，MyClass2 有两个模板参数：类型名称参数 T 和模板参数 Arr：

```c++
template<typename T, template<typename U, int I> class Arr>
class MyClass2
{
    T t; //OK
    Arr<T, 10> a;
    U u; //Error. U not in scope
};
```

由于 Arr 参数本身没有正文，因此不需要其参数名称。 事实上，从 `MyClass2` 的正文中引用 Arr 的类型名称或类参数名称是错误的。 因此，可以省略 Arr 的类型参数名称，如以下示例所示：

```c++
template<typename T, template<typename, int> class Arr>
class MyClass2
{
    T t; //OK
    Arr<T, 10> a;
};
```



### 默认模板自变量

类和函数模板可以具有默认自变量。 如果模板具有默认自变量，可以在使用时不指定该自变量。 例如，std::vector 模板有一个用于分配器的默认自变量：

```c++
template <class T, class Allocator = allocator<T>> class vector;
```

在大多数情况下，默认的 std::allocator 类是可接受的，因此可以使用向量，如下所示：

```c++
vector<int> myInts;
```

但如有必要，可以指定自定义分配器，如下所示：

```c++
vector<int, MyAllocator> ints;
```

对于多个模板参数，第一个默认参数后的所有参数必须具有默认参数。

使用参数均为默认值的模板时，请使用空尖括号：

```c++
template<typename A = int, typename B = double>
class Bar
{
    //...
};
...
int main()
{
    Bar<> bar; // 使用所有默认类型参数
}
```



### 模板特殊化

在某些情况下，模板不可能或不需要为任何类型都定义完全相同的代码。 例如，你可能希望定义在类型参数为指针、std::wstring 或派生自特定基类的类型时才执行的代码路径。 在这种情况下，可以为该特定类型定义模板的专用化。 当用户使用该类型对模板进行实例化时，编译器使用该专用化来生成类，而对于所有其他类型，编译器会选择更常规的模板。 如果专用化中的所有参数都是专用的，则称为“完整专用化”。 如果只有一些参数是专用的，则称为“部分专用化”。

```c++
template <typename K, typename V>
class MyMap{/*...*/};

//字符串键的部分专门化
template<typename V>
class MyMap<string, V> {/*...*/};
...
MyMap<int, MyClass> classes; // 使用原始模板
MyMap<string, MyClass> classes2; // 使用部分专门化
```

只要每个专用类型参数是唯一的，模板就可以具有任意数量的专用化。 只有类模板可能是部分专用。 模板的所有完整专用化和部分专用化都必须在与原始模板相同的命名空间中声明。



***

> 以下内容来自：[typename | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/typename?view=msvc-170)

### `typename`

在模板定义中，**`typename`** 向编译器提供未知标识符是类型的提示。 在模板参数列表中，它用于指定类型参数。

语法： **`typename`** *`identifier`*

如果模板定义中的名称是依赖于模板自变量的限定名称，则必须使用 **`typename`** 关键字；如果限定名称是独立的，则此名称是可选的

**`typename`** 可由任何类型在模板声明或定义中的任何位置使用。 不允许在基类列表中使用该关键字，除非将它用作模板基类的模板自变量。

```c++
template <class T>
class C1 : typename T::InnerType // Error - typename not allowed.
{};
template <class T>
class C2 : A<typename T::InnerType>  // typename OK.
{};
```

**`typename`** 关键字也可以替代模板参数列表中的 **`class`** 使用。 例如，下面的语句在语义上是等效的：

```c++
template<class T1, class T2>...
template<typename T1, typename T2>...
```

```c++
// typename.cpp
template<class T> class X
{
   typename T::Y m_y;   // treat Y as a type
};

int main()
{
}
```

```c++
#include <iostream>

// 定义一个类 A，包含一个嵌套类型 Y
class A {
public:
    using Y = int;  // Y 是 A 的嵌套类型，定义为 int
};

// 定义模板类 X，接受任意类型 T
template<class T> 
class X {
public:
    typename T::Y m_y;  // m_y 的类型是 T 的嵌套类型 Y

    X(typename T::Y val) : m_y(val) {}

    void print() {
        std::cout << "Value of m_y: " << m_y << std::endl;
    }
};

int main() {
    X<A> x(42);  // 实例化 X，使用 A 作为模板参数，Y 被替换为 int
    x.print();   // 输出 "Value of m_y: 42"
    return 0;
}

```



***

### 类模板

> 以下内容来自：[类模板 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/class-templates?view=msvc-170)

#### 类模板的成员函数

可以在类模板的内部或外部定义成员函数。 如果在类模板的外部定义成员函数，则会像定义函数模板一样定义它们。

```c++
// member_function_templates1.cpp
template<class T, int i>
class MyStack
{
    T*  pStack;
    T StackBuffer[i];
    static const int cItems = i * sizeof(T);
public:
    MyStack( void );
    void push( const T item );
    T& pop( void );
};

template< class T, int i > 
MyStack< T, i >::MyStack( void )
{
};

template< class T, int i > 
void MyStack< T, i >::push( const T item )
{
};

template< class T, int i >
T& MyStack< T, i >::pop( void )
{
};

int main()
{
}
```

就像任何模板类成员函数一样，类的构造函数成员函数的定义包含模板自变量列表两次。

成员函数可以是函数模板，并指定额外参数，如下面的示例所示。

```c++
// member_templates.cpp
template<typename T>
class X
{
public:
   template<typename U>
   void mf(const U &u);
};

template<typename T> template <typename U>
void X<T>::mf(const U &u)
{
}

int main()
{
}
```

#### 嵌套类模板

模板可以在类或类模板中定义，在这种情况下，它们被称为成员模板。 作为类的成员模板称为嵌套类模板。 [成员函数模板](https://learn.microsoft.com/zh-cn/cpp/cpp/member-function-templates?view=msvc-170)中讨论了作为函数的成员模板。

嵌套类模板被声明为外部类范围内的类模板。 可以在封闭类的内部或外部定义它们。

下面的代码演示普通类中的嵌套类模板。

```c++
// nested_class_template1.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

class X
{
   template <class T>
   struct Y
   {
      T m_t;
      Y(T t): m_t(t) { }
   };

   Y<int> yInt;
   Y<char> yChar;

public:
   X(int i, char c) : yInt(i), yChar(c) { }
   void print()
   {
      cout << yInt.m_t << " " << yChar.m_t << endl;
   }
};

int main()
{
   X x(1, 'a');
   x.print();
}
```

以下代码使用嵌套模板类型参数来创建嵌套类模板：

```c++
// nested_class_template2.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

template <class T>
class X
{
   template <class U> class Y
   {
      U* u;
   public:
      Y();
      U& Value();
      void print();
      ~Y();
   };

   Y<int> y;
public:
   X(T t) { y.Value() = t; }
   void print() { y.print(); }
};

template <class T>
template <class U>
X<T>::Y<U>::Y()
{
   cout << "X<T>::Y<U>::Y()" << endl;
   u = new U();
}

template <class T>
template <class U>
U& X<T>::Y<U>::Value()
{
   return *u;
}

template <class T>
template <class U>
void X<T>::Y<U>::print()
{
   cout << this->Value() << endl;
}

template <class T>
template <class U>
X<T>::Y<U>::~Y()
{
   cout << "X<T>::Y<U>::~Y()" << endl;
   delete u;
}

int main()
{
   X<int>* xi = new X<int>(10);
   X<char>* xc = new X<char>('c');
   xi->print();
   xc->print();
   delete xi;
   delete xc;
}

/* Output:
X<T>::Y<U>::Y()
X<T>::Y<U>::Y()
10
99
X<T>::Y<U>::~Y()
X<T>::Y<U>::~Y()
*/
```

局部类不允许具有成员模板。



#### 模板友元

类模板可以具有[友元](https://learn.microsoft.com/zh-cn/cpp/cpp/friend-cpp?view=msvc-170)。 类或类模板、函数或函数模板可以是模板类的友元。 友元也可以是类模板或函数模板的专用化，但不是部分专用化。

在以下示例中，友元函数将定义为类模板中的函数模板。 此代码为模板的每个实例化生成一个友元函数版本。 如果您的友元函数与类依赖于相同的模板参数，则此构造很有用。

```c++
// template_friend1.cpp
// compile with: /EHsc

#include <iostream>
using namespace std;

template <class T> class Array {
   T* array;
   int size;

public:
   Array(int sz): size(sz) {
      array = new T[size];
      memset(array, 0, size * sizeof(T));
   }

   Array(const Array& a) {
      size = a.size;
      array = new T[size];
      memcpy_s(array, a.array, sizeof(T));
   }

   T& operator[](int i) {
      return *(array + i);
   }

   int Length() { return size; }

   void print() {
      for (int i = 0; i < size; i++)
         cout << *(array + i) << " ";

      cout << endl;
   }

   template<class T>
   friend Array<T>* combine(Array<T>& a1, Array<T>& a2);
};

template<class T>
Array<T>* combine(Array<T>& a1, Array<T>& a2) {
   Array<T>* a = new Array<T>(a1.size + a2.size);
   for (int i = 0; i < a1.size; i++)
      (*a)[i] = *(a1.array + i);

   for (int i = 0; i < a2.size; i++)
      (*a)[i + a1.size] = *(a2.array + i);

   return a;
}

int main() {
   Array<char> alpha1(26);
   for (int i = 0 ; i < alpha1.Length() ; i++)
      alpha1[i] = 'A' + i;

   alpha1.print();

   Array<char> alpha2(26);
   for (int i = 0 ; i < alpha2.Length() ; i++)
      alpha2[i] = 'a' + i;

   alpha2.print();
   Array<char>*alpha3 = combine(alpha1, alpha2);
   alpha3->print();
   delete alpha3;
}
/* Output:
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m n o p q r s t u v w x y z
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z
*/
```

下一个示例涉及具有模板专用化的友元。 如果原始函数模板是友元，则函数模板专用化将自动为友元。

也可以只将模板的专用版本声明为友元，如以下代码中的友元声明前面的注释所示。 如果将专用化声明为友元，则必须将友元模板专用化的定义放在模板类之外。

```c++
// template_friend2.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

template <class T>
class Array;

template <class T>
void f(Array<T>& a);

template <class T> class Array
{
    T* array;
    int size;

public:
    Array(int sz): size(sz)
    {
        array = new T[size];
        memset(array, 0, size * sizeof(T));
    }
    Array(const Array& a)
    {
        size = a.size;
        array = new T[size];
        memcpy_s(array, a.array, sizeof(T));
    }
    T& operator[](int i)
    {
        return *(array + i);
    }
    int Length()
    {
        return size;
    }
    void print()
    {
        for (int i = 0; i < size; i++)
        {
            cout << *(array + i) << " ";
        }
        cout << endl;
    }
    // If you replace the friend declaration with the int-specific
    // version, only the int specialization will be a friend.
    // The code in the generic f will fail
    // with C2248: 'Array<T>::size' :
    // cannot access private member declared in class 'Array<T>'.
    //friend void f<int>(Array<int>& a);

    friend void f<>(Array<T>& a);
};

// f function template, friend of Array<T>
template <class T>
void f(Array<T>& a)
{
    cout << a.size << " generic" << endl;
}

// Specialization of f for int arrays
// will be a friend because the template f is a friend.
template<> void f(Array<int>& a)
{
    cout << a.size << " int" << endl;
}

int main()
{
    Array<char> ac(10);
    f(ac);

    Array<int> a(10);
    f(a);
}
/* Output:
10 generic
10 int
*/
```

下一个示例显示在类模板中声明的友元类模板。 该类模板随后用作友元类的模板参数。 友元类模板必须在声明它们的类模板外面定义。 友元模板的所有专用化或部分专用化也是原始类模板的友元。

```c++
// template_friend3.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

template <class T>
class X
{
private:
   T* data;
   void InitData(int seed) { data = new T(seed); }
public:
   void print() { cout << *data << endl; }
   template <class U> friend class Factory;
};

template <class U>
class Factory
{
public:
   U* GetNewObject(int seed)
   {
      U* pu = new U;
      pu->InitData(seed);
      return pu;
   }
};

int main()
{
   Factory< X<int> > XintFactory;
   X<int>* x1 = XintFactory.GetNewObject(65);
   X<int>* x2 = XintFactory.GetNewObject(97);

   Factory< X<char> > XcharFactory;
   X<char>* x3 = XcharFactory.GetNewObject(65);
   X<char>* x4 = XcharFactory.GetNewObject(97);
   x1->print();
   x2->print();
   x3->print();
   x4->print();
}
/* Output:
65
97
A
a
*/
```

#### 重复使用模板参数

模板参数可以在模板参数列表中重复使用。 例如，以下代码是允许的：

```c++
// template_specifications2.cpp

class Y
{
};
template<class T, T* pT> class X1
{
};
template<class T1, class T2 = T1> class X2
{
};

Y aY;

X1<Y, &aY> x1;
X2<int> x2;

int main()
{
}
```



### 函数模板

> 以下来自：[函数模板 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/function-templates?view=msvc-170)

 类模板可定义一系列相关类，这些类基于在实例化时传递到类的类型参数。 函数模板类似于类模板，但定义的是一系列函数。 利用函数模板，你可以指定基于相同代码但作用于不同类型或类的函数集。 以下函数模板交换两个项：

```c++
// function_templates1.cpp
template< class T > void MySwap( T& a, T& b ) {
   T c(a);
   a = b;
   b = c;
}
int main() {
}
```

此代码定义交换自变量的值的一系列函数。 从此模板中，可以生成将交换 **`int`** 和 **`long`** 类型和用户定义类型的函数。 如果正确定义了类的复制构造函数和赋值运算符，`MySwap` 甚至会交换类。

此外，函数模板将阻止你交换不同类型的对象，因为在编译时编译器知道 a 和 b 参数的类型。

尽管非模板化函数可以使用 void 指针运行此函数，但模板版本是 typesafe。 请考虑以下调用：

```c++
int j = 10;
int k = 18;
CString Hello = "Hello, Windows!";
MySwap( j, k );          //OK
MySwap( j, Hello );      //error
```

第二个 `MySwap` 调用触发了编译时错误，因为编译器无法生成具有不同类型的参数的 `MySwap` 函数。 如果使用了 void 指针，两个函数调用都将正确编译，但函数在运行时无法正常工作。

允许显式指定函数模板的模板参数。 例如：

```c++
// function_templates2.cpp
template<class T> void f(T) {}
int main(int j) {
   f<char>(j);   // Generate the specialization f(char).
   // If not explicitly specified, f(int) would be deduced.
}
```

当显式指定模板参数时，将对函数自变量执行常规隐式转换以将其转换为对应的函数模板自变量的类型。 在上面的示例中，编译器将 `j` 转换为类型 **`char`**。



#### 函数模板实例化

> 来自：[函数模板实例化 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/function-template-instantiation?view=msvc-170)

当首次为每个类型调用函数模板时，编译器会创建一个实例化。 每个实例化是专用于该类型的模板化函数版本。 每次将该函数用于该类型时，此实例化都将调用。 如果有几个相同的实例化，即使在不同的模块中，也只有该实例化的一个副本将在可执行文件中结束。

允许在函数模板中为其中的形参不依赖于模板实参的任何实参和形参对进行函数形参转换。

通过将特定类型作为自变量来声明模板，可以将函数模板显示实例化。 例如，以下代码是允许的：

```c++
// function_template_instantiation.cpp
template<class T> void f(T) { }

// Instantiate f with the explicitly specified template.
// argument 'int'
//
template void f<int> (int);

// Instantiate f with the deduced template argument 'char'.
template void f(char);
int main()
{
}
```



#### 显式实例化

您可以使用显式实例化来创建模板化类或函数的实例化，而不用将其实际用于您的代码。 由于这在创建使用模板进行分发的库 (*`.lib`*) 文件时非常有用，因此未实例化的模板定义不会放入对象 (*`.obj`*) 文件中。

##### 示例

此代码针对 **`int`** 变量和六个项显式实例化 `MyStack`：

```c++
template class MyStack<int, 6>;
```

该语句创建 `MyStack` 的实例化，而不保留对象的任何存储。 为所有成员生成代码。

下一行仅显式实例化构造函数成员函数：

```c++
template MyStack<int, 6>::MyStack( void );
```

你可以使用特定类型自变量重新声明函数模板来显式实例化这些模板，如[函数模板实例化](https://learn.microsoft.com/zh-cn/cpp/cpp/function-template-instantiation?view=msvc-170)中的示例所示。

可以使用 **`extern`** 关键字来防止自动实例化成员。 例如：

```c++
extern template class MyStack<int, 6>;
```

同样，您可以将特定成员标记为外部且未实例化：

```c++
extern template MyStack<int, 6>::MyStack( void );
```

你可以使用 **`extern`** 关键字阻止编译器在多个对象模块中生成相同的实例化代码。 如果调用该函数，则必须在至少一个链接模块中使用指定的显式模板参数来实例化该函数模板。 否则，生成程序时会出现链接器错误。

> 专用化中的 **`extern`** 关键字仅适用于在类主体外定义的成员函数。 类声明中定义的函数被视为内联函数，并且始终实例化。



#### 函数模板的显式专用化

利用函数模板，您可以通过为特定类型提供函数模板的显式专用化（重写）来定义该类型的特殊行为。 例如：

```c++
template<> void MySwap(double a, double b);
```

此声明使您能够为 **`double`** 变量定义不同函数。 与非模板函数相似，也会应用标准类型转换（如将 **`float`** 类型的变量提升到 **`double`**）。

##### 示例

```c++
// explicit_specialization.cpp
template<class T> void f(T t)
{
};

// f与'char'的显式特化
// 显式指定的模板参数:
//
template<> void f<char>(char c)
{
}

// f与'double'的显式特化
// 模板参数推导:
//
template<> void f(double d)
{
}
int main()
{
}
```



#### 函数模板的部分排序

有多个与函数调用的自变量列表匹配的函数模板可用。 C++ 定义了函数模板的部分排序以指定应调用的函数。 由于有一些模板可能会视为专用化程度相同，因此排序只是部分的。

编译器从可能的匹配项中选择可用的专用化程度最高的函数模板。 例如，如果一个函数模板采用 `T` 类型，而另一个采用 `T*` 的函数模板可用，则称 `T*` 版本的专用化程度更高。 只要参数是指针类型，它就优于泛型 `T` 版本，即使两者都是允许的匹配项。

通过以下过程可确定一个函数模板候选项是否更加专用化：

1. 考虑两个函数模板：`T1` 和 `T2`。
2. 将 `T1` 中的参数替换为假想的唯一类型 `X`。
3. 当 `T1` 中存在参数列表时，查看 `T2` 是否为该参数列表的有效模板。 忽略任何隐式转换。
4. 对 `T1` 和 `T2` 执行相反的过程。
5. 如果一个模板是另一个模板的有效模板参数列表，但反之不成立，则认为该模板的专用化程度不如另一个模板。 如果通过使用上一步，两个模板为彼此形成有效的参数，那么它们被视为同等专用化，并且尝试使用它们时会导致调用不明确。
6. 使用以下规则：
   1. 针对特定类型的模板的专用化程度高于采用泛型类型参数的模板。
   2. 仅采用 `T*` 的模板的专用化程度比仅采用 `T` 的模板更高，因为假设类型 `X*` 是 `T` 模板参数的有效参数，但 `X` 不是 `T*` 模板参数的有效参数。
   3. `const T` 比 `T` 专用化程度更高，因为 `const X` 是 `T` 模板参数的有效参数，但 `X` 不是 `const T` 模板参数的有效参数。
   4. `const T*` 比 `T*` 专用化程度更高，因为 `const X*` 是 `T*` 模板参数的有效参数，但 `X*` 不是 `const T*` 模板参数的有效参数。



##### 示例

```c++
// partial_ordering_of_function_templates.cpp
// compile with: /EHsc
#include <iostream>

template <class T> void f(T) {
   printf_s("Less specialized function called\n");
}

template <class T> void f(T*) {
   printf_s("More specialized function called\n");
}

template <class T> void f(const T*) {
   printf_s("Even more specialized function for const T*\n");
}

int main() {
   int i =0;
   const int j = 0;
   int *pi = &i;
   const int *cpi = &j;

   f(i);   // Calls less specialized function.
   f(pi);  // Calls more specialized function.
   f(cpi); // Calls even more specialized function.
   // Without partial ordering, these calls would be ambiguous.
}
```



#### 成员函数模板

术语成员模板引用了成员函数模板和嵌套类模板。 成员函数模板是类或类模板的成员的函数模板。

成员函数可以是多个环境中的函数模板。 类模板的所有函数都是泛型的，但却不称为成员模板或成员函数模板。 如果这些成员函数采用其自己的模板自变量，则将它们视为成员函数模板。

##### 示例：声明成员函数模板

非模板类或类模板的成员函数模板将声明为带有其模板参数的函数模板。

```c++
// member_function_templates.cpp
struct X
{
   template <class T> void mf(T* t) {}
};

int main()
{
   int i;
   X* x = new X();
   x->mf(&i);
}
```

##### 示例：类模板的成员函数模板

以下示例显示类模板的成员函数模板。

```c++
// member_function_templates2.cpp
template<typename T>
class X
{
public:
   template<typename U>
   void mf(const U &u)
   {
   }
};

int main()
{
}
```



##### 示例：定义类外部的成员模板

```c++
// defining_member_templates_outside_class.cpp
template<typename T>
class X
{
public:
   template<typename U>
   void mf(const U &u);
};

template<typename T> template <typename U>
void X<T>::mf(const U &u)
{
}

int main()
{
}
```



##### 示例：模板化用户定义的转换

局部类不允许具有成员模板。

成员函数模板不能是虚函数。 当使用与基类虚函数相同的名称进行声明时，成员模板函数不能从基类重写虚函数。

以下示例展示了模板化用户定义的转换：

```c++
// templated_user_defined_conversions.cpp
template <class T>
struct S
{
   template <class U> operator S<U>()
   {
      return S<U>();
   }
};

int main()
{
   S<int> s1;
   S<long> s2 = s1;  // Convert s1 using UDC and copy constructs S<long>.
}
```



### 模板特殊化

类模板可以部分专用化，生成的类仍是模板。 在类似于下面的情况下，部分专用化允许为特定类型部分自定义模板代码：

- 模板有多个类型，且只有一部分需要专用化。 结果是基于其余类型参数化的模板。
- 模板只有一个类型，但指针、引用、指向成员的指针或函数指针类型需要专用化。 专用化本身仍是指向或引用的类型上的模板。



#### 示例：类模板的部分专用化

```c++
// partial_specialization_of_class_templates.cpp
#include <stdio.h>

template <class T> struct PTS {
   enum {
      IsPointer = 0,
      IsPointerToDataMember = 0
   };
};

template <class T> struct PTS<T*> {
   enum {
      IsPointer = 1,
      IsPointerToDataMember = 0
   };
};

template <class T, class U> struct PTS<T U::*> {
   enum {
      IsPointer = 0,
      IsPointerToDataMember = 1
   };
};

struct S{};

int main() {
   printf_s("PTS<S>::IsPointer == %d \nPTS<S>::IsPointerToDataMember == %d\n",
           PTS<S>::IsPointer, PTS<S>:: IsPointerToDataMember);
   printf_s("PTS<S*>::IsPointer == %d \nPTS<S*>::IsPointerToDataMember == %d\n"
           , PTS<S*>::IsPointer, PTS<S*>:: IsPointerToDataMember);
   printf_s("PTS<int S::*>::IsPointer == %d \nPTS"
           "<int S::*>::IsPointerToDataMember == %d\n",
           PTS<int S::*>::IsPointer, PTS<int S::*>::
           IsPointerToDataMember);
}
```



#### 示例：指针类型的部分专用化

如果有一个采用任何 `T` 类型的模板集合类，则可以创建采用任何指针类型 `T*` 的部分专用化。 以下代码演示了一个集合类模板 `Bag` 以及指针类型的部分专用化，在此专用化中，该集合在将指针类型复制到数组前取消引用它们。 该集合随后存储指向的值。 对于原始模板，只有指针本身将存储在集合中，从而使数据易受删除或修改。 在此特殊指针版本的集合中，添加了在 `add` 方法中检查 null 指针的代码。

```c++
// partial_specialization_of_class_templates2.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

// Original template collection class.
template <class T> class Bag {
   T* elem;
   int size;
   int max_size;

public:
   Bag() : elem(0), size(0), max_size(1) {}
   void add(T t) {
      T* tmp;
      if (size + 1 >= max_size) {
         max_size *= 2;
         tmp = new T [max_size];
         for (int i = 0; i < size; i++)
            tmp[i] = elem[i];
         tmp[size++] = t;
         delete[] elem;
         elem = tmp;
      }
      else
         elem[size++] = t;
   }

   void print() {
      for (int i = 0; i < size; i++)
         cout << elem[i] << " ";
      cout << endl;
   }
};

// Template partial specialization for pointer types.
// The collection has been modified to check for NULL
// and store types pointed to.
template <class T> class Bag<T*> {
   T* elem;
   int size;
   int max_size;

public:
   Bag() : elem(0), size(0), max_size(1) {}
   void add(T* t) {
      T* tmp;
      if (t == NULL) {   // Check for NULL
         cout << "Null pointer!" << endl;
         return;
      }

      if (size + 1 >= max_size) {
         max_size *= 2;
         tmp = new T [max_size];
         for (int i = 0; i < size; i++)
            tmp[i] = elem[i];
         tmp[size++] = *t;  // Dereference
         delete[] elem;
         elem = tmp;
      }
      else
         elem[size++] = *t; // Dereference
   }

   void print() {
      for (int i = 0; i < size; i++)
         cout << elem[i] << " ";
      cout << endl;
   }
};

int main() {
   Bag<int> xi;
   Bag<char> xc;
   Bag<int*> xp; // Uses partial specialization for pointer types.

   xi.add(10);
   xi.add(9);
   xi.add(8);
   xi.print();

   xc.add('a');
   xc.add('b');
   xc.add('c');
   xc.print();

   int i = 3, j = 87, *p = new int[2];
   *p = 8;
   *(p + 1) = 100;
   xp.add(&i);
   xp.add(&j);
   xp.add(p);
   xp.add(p + 1);
   delete[] p;
   p = NULL;
   xp.add(p);
   xp.print();
}
```



### 模板和名称解析

#### 模板和名称解析

在模板定义中，有三种类型的名称。

- 本部声明的名称，包括模板本身的名称以及在模板定义中声明的任何名称。
- 模板定义之外的封闭范围中的名称。
- 在某种程度上依赖于模板自变量的名称（称为依赖名称）。

尽管前两个名称也属于类和函数范围，但模板定义中需要针对名称解析的特殊规则来处理依赖名称的提高的复杂性。 原因在于，在对模板进行实例化前，编译器几乎不知道这些名称，因为它们可能是依赖于所使用的模板自变量的完全不同的类型。 将根据常用规则，在模板的定义点上查找非依赖名称。 将为所有模板专用化查找这些名称（独立于模板参数）一次。 在将模板实例化并为每个专用化单独查找模板之前，将不会查找依赖名称。

如果某个类型依赖于模板参数，则该类型属于依赖类型。 具体而言，如果类型是以下项，则属于依赖类型：

* 模板自变量本身：

  ```
  T
  ```

* 具有包括依赖类型的限定的限定名：

  ```
  T::myType
  ```

* 当非限定部分标识依赖类型时的限定名：

  ```
  N::T
  ```

* 其基类型属于依赖类型的不变或可变类型：

  const T

* 基于依赖类型的指针、引用、数组或函数指针类型：

  ```
  T *, T &, T [10], T (*)()
  ```

* 大小基于模板参数的数组：

  ```
  template <int arg> class X {
  int x[arg] ; // dependent type
  }
  ```

* 从模板参数构造的模板类型：

  ```
  T<int>, MyTemplate<T>
  ```



##### 类型依赖和值依赖

模板参数上的名称和表达式依赖项分为类型依赖项或值依赖项，具体取决于模板参数是类型参数还是值参数。 此外，在模板自变量上有类型依赖项的模板中声明的任何标识符都被视为依赖值，使用值依赖表达式初始化的整数类型或枚举类型也是如此。

类型依赖和值依赖表达式是涉及类型依赖或值依赖的变量的表达式。 这些表达式可能有不同的语义，具体取决于用于模板的参数。



#### 依赖类型的名称解析

使用 **`typename`** 作为模板定义中的限定名称，以告知编译器给定的限定名称标识一个类型。 有关详细信息，请参阅 [typename](https://learn.microsoft.com/zh-cn/cpp/cpp/typename?view=msvc-170)。

```c++
// template_name_resolution1.cpp
#include <stdio.h>
template <class T> class X
{
public:
   void f(typename T::myType* mt) {}
};

class Yarg
{
public:
   struct myType { };
};

int main()
{
   X<Yarg> x;
   x.f(new Yarg::myType());
   printf("Name resolved by using typename keyword.");
}
```

```
Name resolved by using typename keyword.
```

针对依赖名称的名称查找将检查模板定义上下文（在下面的示例中，此上下文将查找 `myFunction(char)`）和模板实例化上下文中的名称。在下面的示例中，将在 main 中实例化模板；因此，`MyNamespace::myFunction` 从实例化的角度来看是可见的，因而将选取它作为更好的匹配。 如果重命名 `MyNamespace::myFunction`，则将调用 `myFunction(char)`。

所有名称都会得到解析，就如同它们是依赖名称一样。 尽管如此，如果存在任何可能的冲突，建议您使用完全限定名。

```c++
// template_name_resolution2.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

void myFunction(char)
{
   cout << "Char myFunction" << endl;
}

template <class T> class Class1
{
public:
   Class1(T i)
   {
      // If replaced with myFunction(1), myFunction(char)
      // will be called
      myFunction(i);
}
};

namespace MyNamespace
{
   void myFunction(int)
   {
      cout << "Int MyNamespace::myFunction" << endl;
   }
};

using namespace MyNamespace;

int main()
{
   Class1<int>* c1 = new Class1<int>(100);
}
```

```
Int MyNamespace::myFunction
```

##### 模板消除歧义

Visual Studio 2012 强制实施 C++98/03/11 标准规则以使用“template”关键字消除歧义。 在下面的示例中，Visual Studio 2010 将接受不一致性行和一致性行。 Visual Studio 2012 仅接受一致性行。

```c++
#include <iostream>
#include <ostream>
#include <typeinfo>
using namespace std;

template <typename T> struct Allocator {
    template <typename U> struct Rebind {
        typedef Allocator<U> Other;
    };
};

template <typename X, typename AY> struct Container {
    #if defined(NONCONFORMANT)
        typedef typename AY::Rebind<X>::Other AX; // nonconformant
    #elif defined(CONFORMANT)
        typedef typename AY::template Rebind<X>::Other AX; // conformant
    #else
        #error Define NONCONFORMANT or CONFORMANT.
    #endif
};

int main() {
    cout << typeid(Container<int, Allocator<float>>::AX).name() << endl;
}
```

需要符合消除歧义规则，因为默认情况下，C++ 假定 `AY::Rebind` 不是模板，因此编译器会将后面的“`<`”解释为小于。 它必须知道 `Rebind` 是模板，这样才能正确地将“`<`”分析为尖括号。

#### 本地声明名称的名称解析

通过和不通过模板自变量都可引用模板名称本身。 在类模板的范围内，名称本身引用模板。 在模板专用化或部分专用化的范围中，名称单独引用专用化或部分专用化。 也可以通过适当的模板参数引用模板的其他专用化或部分专用化。

##### 示例：专用化与部分专用化

以下代码演示在专用化或部分专用化的范围内以不同的方式解释类模板的名称 `A`。

```c++
// template_name_resolution3.cpp
// compile with: /c
template <class T> class A {
   A* a1;   // A refers to A<T>
   A<int>* a2;  // A<int> refers to a specialization of A.
   A<T*>* a3;   // A<T*> refers to the partial specialization A<T*>.
};

template <class T> class A<T*> {
   A* a4; // A refers to A<T*>.
};

template<> class A<int> {
   A* a5; // A refers to A<int>.
};
```



##### 示例：模板参数和对象之间的名称冲突.

在模板参数与另一个对象之间出现名称冲突的情况下，模板参数可隐藏，也可不隐藏。 下列规则将帮助确定优先顺序。

模板参数位于从其首先出现的位置开始一直到类或函数末尾的范围中。 如果名称再次出现在模板参数列表或基类列表中，则它将引用同一类型。 在标准 C++ 中，无法在同一范围中声明与模板参数相同的其他名称。 Microsoft 扩展允许在模板范围内重新定义模板参数。 以下示例演示在类模板的基规范中使用模板参数。

```c++
// template_name_resolution4.cpp
// compile with: /EHsc
template <class T>
class Base1 {};

template <class T>
class Derived1 : Base1<T> {};

int main() {
   // Derived1<int> d;
}
```



##### 示例：在类模板外定义成员函数

当在类模板外定义成员函数时，可以使用不同的模板参数名称。 如果类模板成员函数定义与声明对模板参数使用了不同的名称，并且定义中使用的名称与声明的其他成员冲突，则模板声明中的成员优先。

```c++
// template_name_resolution5.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

template <class T> class C {
public:
   struct Z {
      Z() { cout << "Z::Z()" << endl; }
   };
   void f();
};

template <class Z>
void C<Z>::f() {
   // Z refers to the struct Z, not to the template arg;
   // Therefore, the constructor for struct Z will be called.
   Z z;
}

int main() {
   C<int> c;
   c.f();
}
```

```
Z::Z()
```



##### 示例：在命名空间外定义模板或成员函数

在声明模板的命名空间外定义函数模板或成员函数时，模板自变量将优先于命名空间中其他成员的名称。

```c++
// template_name_resolution6.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

namespace NS {
   void g() { cout << "NS::g" << endl; }

   template <class T> struct C {
      void f();
      void g() { cout << "C<T>::g" << endl; }
   };
};

template <class T>
void NS::C<T>::f() {
   g(); // C<T>::g, not NS::g
};

int main() {
   NS::C<int> c;
   c.f();
}
```

```
C<T>::g
```



##### 示例：基类或成员名称隐藏模板参数

在位于模板类声明之外的定义中，如果模板类具有不依赖于模板自变量的基类，并且该基类或其成员之一与模板自变量的名称相同，则该基类或成员名称将隐藏模板自变量。

```c++
// template_name_resolution7.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

struct B {
   int i;
   void print() { cout << "Base" << endl; }
};

template <class T, int i> struct C : public B {
   void f();
};

template <class B, int i>
void C<B, i>::f() {
   B b;   // Base class b, not template argument.
   b.print();
   i = 1; // Set base class's i to 1.
}

int main() {
   C<int, 1> c;
   c.f();
   cout << c.i << endl;
}
```

```
Base
1
```



#### 函数模板调用的重载解析

函数模板可以重载具有相同名称的非模板函数。 在此方案中，编译器首先尝试使用模板自变量推导来解析函数调用，以实例化具有唯一特化的函数模板。 如果模板自变量推导失败，则编译器会考虑实例化函数模板重载和非模板函数重载来解析调用。 这些其他重载称为*候选集*。 如果模板自变量推导成功，则按照重载决策的规则，将生成的函数与候选集中的其他函数进行比较，以确定最佳匹配。 有关详细信息，请参阅[函数重载](https://learn.microsoft.com/zh-cn/cpp/cpp/function-overloading?view=msvc-170)。



##### 示例：选择非模板函数

如果非模板函数是函数模板的很好的匹配，则选择非模板函数（除非已显式指定模板自变量），如以下示例中的 `f(1, 1)` 调用。

```c++
// template_name_resolution9.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

void f(int, int) { cout << "f(int, int)" << endl; }
void f(char, char) { cout << "f(char, char)" << endl; }

template <class T1, class T2>
void f(T1, T2)
{
   cout << "void f(T1, T2)" << endl;
};

int main()
{
   f(1, 1);   // Equally good match; choose the non-template function.
   f('a', 1); // Chooses the function template.
   f<int, int>(2, 2);  // Template arguments explicitly specified.
}
```

```
f(int, int)
void f(T1, T2)
void f(T1, T2)
```



##### 示例：首选精确匹配函数模板

下一个示例阐释了一点，即如果非模板函数需要转换，则完全匹配的函数模板是首选的。

```c++
// template_name_resolution10.cpp
// compile with: /EHsc
#include <iostream>
using namespace std;

void f(int, int) { cout << "f(int, int)" << endl; }

template <class T1, class T2>
void f(T1, T2)
{
   cout << "void f(T1, T2)" << endl;
};

int main()
{
   long l = 0;
   int i = 0;
   // Call the function template f(long, int) because f(int, int)
   // would require a conversion from long to int.
   f(l, i);
}
```

```
void f(T1, T2)
```



#### 源代码组织

在定义类模板时，必须以这样一种方式组织源代码：在编译器需要成员定义时，可以看到成员定义。 可以选择使用包含模型或显式实例化模型。 在包含模型中，在使用模板的每个文件中都加入成员定义。 这是最简单的方法，它在可与模板一起使用的具体类型方面提供了最大的灵活性。 缺点是会增加编译时间。 如果项目或包含的文件本身很大，那么时间可能很长。 使用显式实例化方法，模板本身会实例化特定类型的具体类或类成员。 此方法可以加快编译时间，但它仅限于模板实现者提前启用的那些类的使用情况。 通常，我们建议使用包含模型，除非由于编译时间问题而存在局限性。



##### 背景

模板与普通类不同，因为编译器不会为模板或其任何成员生成对象代码。 在使用具体类型实例化模板之前，不会生成任何内容。 当编译器遇到模板实例化（例如 `MyClass<int> mc;`）并且尚未存在具有该签名的类时，它将生成一个新类。 它还会尝试为使用的任何成员函数生成代码。 如果这些定义位于一个并未直接或间接 #included 在正在编译的 .cpp 文件内的文件中，编译器就无法看到它们。 从编译器的角度来看，这不一定是错误。 这些函数可以在另一个翻译单元中定义，链接器会在那里找到它们。 如果链接器找不到该代码，则会引发无法解析的外部错误。



##### 包含模型

使模板定义在整个翻译单元中可见的最简单、最常见的方法是将定义放入头文件本身。 使用模板的任何 *`.cpp`* 文件只需 `#include` 标头。 这是标准库中使用的方法。

```c++
#ifndef MYARRAY
#define MYARRAY
#include <iostream>

template<typename T, size_t N>
class MyArray
{
    T arr[N];
public:
    // Full definitions:
    MyArray(){}
    void Print()
    {
        for (const auto v : arr)
        {
            std::cout << v << " , ";
        }
    }

    T& operator[](int i)
   {
       return arr[i];
   }
};
#endif
```

使用此方法，编译器有权访问完整的模板定义，并可以按需为任何类型实例化模板。 这很简单，而且相对容易维护。 但是，包含模型确实存在编译时间成本。 在大型程序中，此类成本可能很高，尤其是在模板标头本身 #include 其他标头的情况下。 每个 #include 标头的 *`.cpp`* 文件都将获得自己的函数模板和所有定义的副本。 链接器通常能够解决问题，这样最终便不会得到函数的多个定义，但是完成这项工作需要时间。 在较小的程序中，可能不需要太长的额外编译时间。



##### 显式实例化模型

如果包含模型无法用于你的项目，并且你明确知道将用于实例化模板的类型集，则可以将模板代码分离到 *`.h`* 和 *`.cpp`* 文件中，并在 *`.cpp`* 文件中显式实例化模板。 此方法将生成编译器在遇到用户实例化时将看到的对象代码。

通过使用关键字 **`template`**，然后使用要实例化的实体的签名，创建显式实例化。 此实体可以是类型或成员。 如果显式实例化类型，则会实例化所有成员。

头文件 `MyArray.h` 声明模板类 `MyArray`：

```c++
//MyArray.h
#ifndef MYARRAY
#define MYARRAY

template<typename T, size_t N>
class MyArray
{
    T arr[N];
public:
    MyArray();
    void Print();
    T& operator[](int i);
};
#endif
```

源文件 `MyArray.cpp` 显式实例化 `template MyArray<double, 5>` 和 `template MyArray<string, 5>`：

```c++
//MyArray.cpp
#include <iostream>
#include "MyArray.h"

using namespace std;

template<typename T, size_t N>
MyArray<T,N>::MyArray(){}

template<typename T, size_t N>
void MyArray<T,N>::Print()
{
    for (const auto v : arr)
    {
        cout << v << "'";
    }
    cout << endl;
}

template MyArray<double, 5>;
template MyArray<string, 5>;
```

在前面的示例中，显式实例化位于 *`.cpp`* 文件的底部。 `MyArray` 只能用于 **`double`** 或 `String` 类型。

> 在 C++11 中，模板定义的上下文中已弃用 **`export`** 关键字。 实际上，这几乎没有影响，因为大多数编译器从不支持它。





