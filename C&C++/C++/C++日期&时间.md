结构类型 **tm** 把日期和时间以 C 结构的形式保存，tm 结构的定义如下：

```c
struct tm {
  int tm_sec;   // 秒，正常范围从 0 到 59，但允许至 61
  int tm_min;   // 分，范围从 0 到 59
  int tm_hour;  // 小时，范围从 0 到 23
  int tm_mday;  // 一月中的第几天，范围从 1 到 31
  int tm_mon;   // 月，范围从 0 到 11
  int tm_year;  // 自 1900 年起的年数
  int tm_wday;  // 一周中的第几天，范围从 0 到 6，从星期日算起
  int tm_yday;  // 一年中的第几天，范围从 0 到 365，从 1 月 1 日算起
  int tm_isdst; // 夏令时
};
```

下面是 C/C++ 中关于日期和时间的重要函数。所有这些函数都是 C/C++ 标准库的组成部分，您可以在 C++ 标准库中查看一下各个函数的细节。

| 序号 | 函数 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | [**time_t time(time_t \*time);**](https://www.runoob.com/cplusplus/c-function-time.html) 该函数返回系统的当前日历时间，自 1970 年 1 月 1 日以来经过的秒数。如果系统没有时间，则返回 -1。 |
| 2    | [**char \*ctime(const time_t \*time);**](https://www.runoob.com/cplusplus/c-function-ctime.html) 该返回一个表示当地时间的字符串指针，字符串形式 *day month year hours:minutes:seconds year\n\0*。 |
| 3    | [**struct tm \*localtime(const time_t \*time);**](https://www.runoob.com/cplusplus/c-function-localtime.html) 该函数返回一个指向表示本地时间的 **tm** 结构的指针。 |
| 4    | [**clock_t clock(void);**](https://www.runoob.com/cplusplus/c-function-clock.html) 该函数返回程序执行起（一般为程序的开头），处理器时钟所使用的时间。如果时间不可用，则返回 -1。 |
| 5    | [**char \* asctime ( const struct tm \* time );**](https://www.runoob.com/cplusplus/c-function-asctime.html) 该函数返回一个指向字符串的指针，字符串包含了 time 所指向结构中存储的信息，返回形式为：day month date hours:minutes:seconds year\n\0。 |
| 6    | [**struct tm \*gmtime(const time_t \*time);**](https://www.runoob.com/cplusplus/c-function-gmtime.html) 该函数返回一个指向 time 的指针，time 为 tm 结构，用协调世界时（UTC）也被称为格林尼治标准时间（GMT）表示。 |
| 7    | [**time_t  mktime(struct tm \*time);**](https://www.runoob.com/cplusplus/c-function-mktime.html) 该函数返回日历时间，相当于 time 所指向结构中存储的时间。 |
| 8    | [**double  difftime ( time_t time2, time_t time1 );**](https://www.runoob.com/cplusplus/c-function-difftime.html) 该函数返回 time1 和 time2 之间相差的秒数。 |
| 9    | [**size_t strftime();**](https://www.runoob.com/cplusplus/c-function-strftime.html) 该函数可用于格式化日期和时间为指定的格式。 |



##  获取当前时间 

```c++
time_t now = time(0);
// 把 now 转换为字符串形式
char* dt = ctime(&now);
cout << "本地日期和时间：" << dt << endl;

本地日期和时间：Fri Aug 23 09:49:28 2024
```



```c++
time_t now = time(0);
// 将time_t转换为tm结构体
struct tm *local = localtime(&now);

// 打印年、月、日、时、分、秒
std::cout << 1900 + local->tm_year << '-'
          << 1 + local->tm_mon << '-'
          << local->tm_mday << ' '
          << local->tm_hour << ':'
          << local->tm_min << ':'
          << local->tm_sec << std::endl;

2024-8-23 9:50:26
```
```c++
time_t now = time(0);
tm *gmtm = gmtime(&now);
dt = asctime(gmtm);
cout << "UTC 日期和时间："<< dt << endl;

UTC 日期和时间: Fri Aug 23 01:58:14 2024
```



`time_t now = time(0);` 语句获取当前的时间，并将其存储在 `now` 变量中。这个时间值是一个 `time_t` 类型，通常表示自 Unix 纪元（1970 年 1 月 1 日 00:00:00 UTC）以来的秒数。

接下来，`ctime(&now)`、`localtime(&now)` 和 `gmtime(&now)` 都是将这个 `time_t` 类型的时间值转换为不同形式的时间表示。让我们看看它们之间的区别。

### `ctime(&now)`
- **功能**: `ctime()` 将 `time_t` 类型的时间值转换为人类可读的字符串格式，表示本地时间。
- **返回值**: 返回一个指向格式化字符串的 `char*`。例如 `"Wed Aug 23 14:55:02 2023\n"`。
- **特点**:
  - 输出的时间是基于系统的本地时区。
  - 返回的字符串包含换行符 `\n`，并以空字符 `\0` 结尾。
  - 由于字符串是静态分配的，后续对 `ctime()` 的调用会覆盖之前的值。

### `localtime(&now)`
- **功能**: `localtime()` 将 `time_t` 类型的时间值转换为本地时间，并以 `struct tm` 结构的形式返回。
- **返回值**: 返回一个指向 `struct tm` 结构的指针，该结构包含详细的本地时间信息，如年、月、日、时、分、秒等。
- **特点**:
  - 结构中的时间值是基于系统的本地时区。
  - 返回的 `struct tm` 结构的指针指向的是静态内存，后续对 `localtime()` 的调用会覆盖之前的值。

### gmtime(&now)`
- **功能**: `gmtime()` 将 `time_t` 类型的时间值转换为 UTC（协调世界时，即 GMT）时间，并以 `struct tm` 结构的形式返回。
- **返回值**: 返回一个指向 `struct tm` 结构的指针，该结构包含详细的 UTC 时间信息，如年、月、日、时、分、秒等。
- **特点**:
  - 结构中的时间值是基于 UTC 而非本地时区。
  - 返回的 `struct tm` 结构的指针指向的是静态内存，后续对 `gmtime()` 的调用会覆盖之前的值。

### 总结对比
- **时区**:
  - `ctime(&now)`：返回基于本地时区的可读字符串。
  - `localtime(&now)`：返回基于本地时区的 `struct tm` 结构。
  - `gmtime(&now)`：返回基于 UTC（GMT）的 `struct tm` 结构。

- **返回类型**:
  - `ctime(&now)`：返回 `char*` 类型的字符串。
  - `localtime(&now)` 和 `gmtime(&now)`：返回 `struct tm*` 类型的指针，指向包含详细时间信息的结构。

- **使用场景**:
  - 如果你只需要一个简单的、可读的本地时间字符串，使用 `ctime()`。
  - 如果你需要对本地时间的各个部分进行操作（例如计算日期差、处理时区），使用 `localtime()`。
  - 如果你需要操作 UTC 时间（例如在跨时区系统中），使用 `gmtime()`。



## 时间戳

秒级

```cassandra
time_t timestamp;
std::cout << time(&timestamp) << std::endl;//秒级时间戳

 time_t now = time(0);
 std::cout << now << std::endl;//秒级时间戳
```



毫秒级

```c++
#include <chrono>

// 获取当前时间戳
auto now = chrono::system_clock::now();
// 将时间戳转换为毫秒数
auto now_ms = chrono::time_point_cast<chrono::milliseconds>(now);
auto value = now_ms.time_since_epoch().count();
```



## 对比

相差

```C++
// 获取当前时间
time_t start_time = time(0);

// 模拟一个延迟，等待 10 秒
std::cout << "等待 10 秒...\n";
sleep(10);

// 获取当前时间
time_t end_time = time(0);

// 计算时间差
double seconds_diff = difftime(end_time, start_time);

std::cout << "两个时间之间的差异为: " << seconds_diff << " 秒" << std::endl;
```



大小

```C++
// 获取当前时间
time_t time1 = time(0);

// 模拟一个延迟，等待 5 秒
std::cout << "等待 5 秒...\n";
sleep(5);

// 获取当前时间
time_t time2 = time(0);

// 比较两个时间
if (time1 < time2) {
std::cout << "time1 早于 time2" << std::endl;
} else if (time1 > time2) {
std::cout << "time1 晚于 time2" << std::endl;
} else {
std::cout << "time1 和 time2 相等" << std::endl;
}

```



## 偏移

当前时间加（减）一天

```C++
// 获取当前时间
time_t now = time(0);
std::cout << "当前时间: " << ctime(&now);

// 将 time_t 转换为 tm 结构
struct tm* time_info = localtime(&now);

// 增加一天
time_info->tm_mday += 1;
//减一天
// time_info->tm_mday -= 1;

// 增加一年
//time_info->tm_year += 1;
// 减一年
//time_info->tm_year -= 1;

// 将修改后的 tm 结构转换回 time_t
time_t future_time = mktime(time_info);

std::cout << "增加1天后的时间: " << ctime(&future_time);
```



## 转化

### 字符串转化为时间戳

在 C++ 中，将时间字符串转换为时间戳（`time_t` 类型）通常涉及以下步骤：

1. **解析时间字符串**：使用适当的库函数将时间字符串转换为 `struct tm`。
2. **将 `struct tm` 转换为时间戳**：使用 `mktime` 函数将 `struct tm` 转换为 `time_t` 类型。

以下是几个常见的方法来实现这一转换：

1. 使用 `std::tm` 和 `mktime`

```cpp
#include <iostream>
#include <sstream>
#include <iomanip>
#include <ctime>

int main() {
    // 时间字符串
    std::string timeStr = "2024-08-23 14:30:00";

    // 解析时间字符串到 std::tm 结构
    std::tm tm = {};
    std::istringstream ss(timeStr);
    ss >> std::get_time(&tm, "%Y-%m-%d %H:%M:%S");

    if (ss.fail()) {
        std::cerr << "解析时间字符串失败" << std::endl;
        return 1;
    }

    // 将 std::tm 转换为 time_t
    std::time_t timeStamp = std::mktime(&tm);

    if (timeStamp == -1) {
        std::cerr << "转换时间戳失败" << std::endl;
        return 1;
    }

    std::cout << "时间戳: " << timeStamp << std::endl;
    return 0;
}
```

2. 使用 `<chrono>` 和 `std::chrono::parse`

C++20 引入了新的时间解析功能，你可以使用 `std::chrono::parse` 函数（在某些实现中）来解析时间字符串。这需要使用 `<chrono>` 头文件，并且支持 C++20 的编译器和标准库。

```cpp
#include <iostream>
#include <chrono>
#include <format>

int main() {
    // 时间字符串
    std::string timeStr = "2024-08-23 14:30:00";

    // 解析时间字符串
    std::chrono::system_clock::time_point tp;
    std::istringstream iss(timeStr);
    iss >> std::chrono::parse("%F %T", tp);

    if (iss.fail()) {
        std::cerr << "解析时间字符串失败" << std::endl;
        return 1;
    }

    // 转换为时间戳
    auto timeStamp = std::chrono::system_clock::to_time_t(tp);
    std::cout << "时间戳: " << timeStamp << std::endl;

    return 0;
}
```

3. 使用 Boost.Date_Time 库

Boost.Date_Time 提供了更强大的时间处理功能，可以用于解析时间字符串和转换为时间戳。

```cpp
#include <iostream>
#include <boost/date_time/posix_time/posix_time.hpp>

int main() {
    // 时间字符串
    std::string timeStr = "2024-08-23 14:30:00";

    // 解析时间字符串
    boost::posix_time::ptime pt(boost::date_time::time_from_string(timeStr));

    // 转换为 time_t
    std::time_t timeStamp = (pt - boost::posix_time::from_time_t(0)).total_seconds();
    std::cout << "时间戳: " << timeStamp << std::endl;

    return 0;
}
```

总结

- **使用 `std::tm` 和 `mktime`**：适用于 C++98 和 C++11，标准方法。
- **使用 `<chrono>` 和 C++20**：提供现代化的时间处理功能，需支持 C++20。
- **使用 Boost.Date_Time**：适用于 Boost 用户，功能丰富，支持复杂的日期和时间操作。

选择合适的方法取决于你的编译器支持的 C++ 标准、是否使用 Boost 库以及具体的时间格式需求。



### 时间戳转化为字符串

在 C++ 中，将时间戳（`time_t` 类型）转换为日期时间字符串可以通过以下几种方法实现。这些方法可以将 `time_t` 转换为 `std::tm` 结构，然后使用格式化函数将其转换为字符串。

1. 使用 `std::localtime` 和 `std::strftime`

这是 C++98 和 C++11 中最常用的方法，适用于大多数标准 C++ 编译器。

```cpp
#include <iostream>
#include <iomanip>
#include <ctime>

int main() {
    // 时间戳
    std::time_t timeStamp = std::time(nullptr); // 当前时间戳

    // 转换为 std::tm 结构
    std::tm* tm = std::localtime(&timeStamp);

    // 格式化为字符串
    char buffer[100];
    std::strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", tm);

    std::cout << "日期时间字符串: " << buffer << std::endl;
    return 0;
}
```

2. 使用 `<chrono>` 和 C++20

C++20 引入了对时间格式化的支持，你可以使用 `std::chrono` 库来进行格式化。如果你的编译器支持 C++20，下面的示例可以直接使用。

```cpp
#include <iostream>
#include <chrono>
#include <format>

int main() {
    // 时间戳
    std::time_t timeStamp = std::time(nullptr); // 当前时间戳

    // 转换为 std::chrono::system_clock::time_point
    auto tp = std::chrono::system_clock::from_time_t(timeStamp);

    // 格式化为字符串
    std::string timeStr = std::format("{:%Y-%m-%d %H:%M:%S}", std::chrono::floor<std::chrono::seconds>(tp));

    std::cout << "日期时间字符串: " << timeStr << std::endl;
    return 0;
}
```

3. 使用 Boost.Date_Time 库

如果你在使用 Boost 库，`Boost.Date_Time` 提供了灵活的日期时间格式化功能。

```cpp
#include <iostream>
#include <boost/date_time/posix_time/posix_time.hpp>

int main() {
    // 时间戳
    std::time_t timeStamp = std::time(nullptr); // 当前时间戳

    // 转换为 ptime
    boost::posix_time::ptime pt = boost::posix_time::from_time_t(timeStamp);

    // 格式化为字符串
    std::string timeStr = boost::posix_time::to_simple_string(pt);

    std::cout << "日期时间字符串: " << timeStr << std::endl;
    return 0;
}
```

4. 使用 `std::put_time` 和 `std::localtime`

在 C++11 及之后的版本中，你可以使用 `std::put_time` 和 `std::localtime` 来实现格式化。

```cpp
#include <iostream>
#include <iomanip>
#include <ctime>

int main() {
    // 时间戳
    std::time_t timeStamp = std::time(nullptr); // 当前时间戳

    // 转换为 std::tm 结构
    std::tm* tm = std::localtime(&timeStamp);

    // 格式化为字符串
    std::cout << "日期时间字符串: " << std::put_time(tm, "%Y-%m-%d %H:%M:%S") << std::endl;
    return 0;
}
```

总结

- **`std::localtime` 和 `std::strftime`**：适用于 C++98 和 C++11，简单直接。
- **C++20 `<chrono>` 和 `std::format`**：现代化的时间格式化，需支持 C++20。
- **Boost.Date_Time**：功能丰富，适用于 Boost 用户。
- **`std::put_time`**：适用于 C++11 及之后版本，提供便捷的时间格式化功能。

根据你的编译器支持和项目需求选择适合的方法。





##  chrono库

C++ 的 `<chrono>` 库是一个强大且灵活的时间库，提供了高精度的计时器、时钟、时间点、持续时间和时间单位。以下是 `<chrono>` 库的主要组件和常用用法的详细介绍：

### **时钟（Clocks）**

时钟用于获取当前的时间点。`<chrono>` 提供了几种不同的时钟：

- **`std::chrono::system_clock`**：表示系统的墙钟时间（系统时间）。可以转换为 `time_t` 类型。
- **`std::chrono::steady_clock`**：表示单调时间（不受系统时间更改影响）。适用于测量时间间隔。
- **`std::chrono::high_resolution_clock`**：表示具有最高分辨率的时钟，通常基于 `steady_clock`。

#### 示例：
```cpp
#include <iostream>
#include <chrono>

int main() {
    // 获取系统时间点
    auto now = std::chrono::system_clock::now();
    std::cout << "系统时间点: " << now.time_since_epoch().count() << " 以纪元为基准的纳秒数" << std::endl;

    // 获取单调时间点
    auto steady_now = std::chrono::steady_clock::now();
    std::cout << "单调时间点: " << steady_now.time_since_epoch().count() << " 以纪元为基准的纳秒数" << std::endl;

    return 0;
}
```

### **时间点（Time Points）**

时间点表示特定的时间。可以使用不同的时钟来表示时间点。

#### 示例：
```cpp
#include <iostream>
#include <chrono>
#include <ctime>

int main() {
    // 获取系统时间点
    auto now = std::chrono::system_clock::now();

    // 转换为 time_t 类型（自纪元以来的秒数）
    std::time_t now_c = std::chrono::system_clock::to_time_t(now);
    std::cout << "当前时间: " << std::ctime(&now_c);

    return 0;
}
```

### **持续时间（Durations）**

持续时间表示时间的长度。可以用不同的单位表示，例如秒、毫秒、微秒等。

#### 示例：
```cpp
#include <iostream>
#include <chrono>

int main() {
    // 定义持续时间
    std::chrono::seconds sec(60);
    std::chrono::minutes min(1);
    std::chrono::milliseconds ms(1000);

    // 转换为秒
    auto sec_from_min = std::chrono::duration_cast<std::chrono::seconds>(min);

    // 输出结果
    std::cout << "1 分钟等于 " << sec_from_min.count() << " 秒" << std::endl;
    std::cout << "1000 毫秒等于 " << ms.count() << " 毫秒" << std::endl;

    return 0;
}
```

### **计算时间差（Duration Between Two Time Points）**

计算两个时间点之间的持续时间。

#### 示例：
```cpp
#include <iostream>
#include <chrono>
#include <thread>

int main() {
    // 获取起始时间点
    auto start = std::chrono::high_resolution_clock::now();

    // 模拟操作（等待 2 秒）
    std::this_thread::sleep_for(std::chrono::seconds(2));

    // 获取结束时间点
    auto end = std::chrono::high_resolution_clock::now();

    // 计算时间差
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "操作耗时: " << duration.count() << " 毫秒" << std::endl;

    return 0;
}
```

### **时间单位（Time Units）**

`<chrono>` 提供了多种时间单位，你可以自定义时间单位或使用标准单位，如秒、毫秒、微秒、纳秒等。

#### 示例：
```cpp
#include <iostream>
#include <chrono>

int main() {
    // 定义时间单位
    std::chrono::milliseconds ms(1000);
    std::chrono::seconds sec = std::chrono::duration_cast<std::chrono::seconds>(ms);

    std::cout << "1000 毫秒等于 " << sec.count() << " 秒" << std::endl;

    return 0;
}
```

### **暂停和延迟（Sleep and Delay）**

使用 `std::this_thread::sleep_for` 和 `std::this_thread::sleep_until` 来实现线程的暂停或延迟。

#### 示例：
```cpp
#include <iostream>
#include <chrono>
#include <thread>

int main() {
    std::cout << "等待 3 秒..." << std::endl;

    // 暂停 3 秒
    std::this_thread::sleep_for(std::chrono::seconds(3));

    std::cout << "3 秒已过。" << std::endl;

    // 使用 sleep_until
    auto now = std::chrono::steady_clock::now();
    std::this_thread::sleep_until(now + std::chrono::seconds(2));

    std::cout << "额外的 2 秒已过。" << std::endl;

    return 0;
}
```

### **自定义时间单位**

你可以自定义时间单位来满足特定需求。

#### 示例：
```cpp
#include <iostream>
#include <chrono>

int main() {
    // 自定义时间单位
    using my_hours = std::chrono::duration<int, std::ratio<3600>>;

    // 定义 1 小时
    my_hours one_hour(1);

    // 转换为秒
    auto seconds_in_one_hour = std::chrono::duration_cast<std::chrono::seconds>(one_hour);

    std::cout << "1 小时等于 " << seconds_in_one_hour.count() << " 秒" << std::endl;

    return 0;
}
```
