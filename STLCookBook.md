

# 前言

1、想删除 vector 的匹配的元素，可以使用 erase-remove 方法:

```c++
auto it = std::remove(vec1.begin(), vec1.end(), value);
vec1.erase(it, vec1.end()); 
```

从 C++20 起，可以直接使用新的 std::erase 函数，并在一个函数调用中完成这些工作: 

```C++
std::erase(vec1, value)
```

2、大括号初始化

大括号初始化使用列表初始化操作符 {}(C++11 引入)，避免赋值操作符的缺点，赋值操作符也是一个复制构造函数，所以数据类型可能会进行隐式窄向转换。

```C++
std::string name{ "Jimi Hendrix" }; // braced initialization
std::string name = "Jimi Hendrix"; // copy initialization
```

```C++
int x; // uninitialized bad :(
int x = 0; // zero (copy constructed) good :)
int x{}; // zero (zero-initialized) best :D
//空大括号 0 初始化为初始化新变量，提供了一种更简单快捷的方式。
```

3、隐藏std::命名空间

代码中导入整个 std:: 命名空间是一种糟糕的做法。应该避免这样使用:

```C++
using namespace std; // bad. don't do that.
cout << "Hello, Jimi!\n";
```

使用 cout 时，可以假设已经包含了一个 using 声明，就像这样: 1

```C++
using std::cout; // cout is now sans prefix 2 
cout << "Hello, Jimi!\n";
```

命名空间包含数千个标识符，没有理由让命名空间充满标识符。冲突的可能性还是有的，并且 对于调试来说风险更高。当使用一个没有 std:: 前缀的名称时，首选的方法是一次导入一个命名空 间。 为了进一步避免命名空间冲突，我经常为重用的类使用单独的命名空间。我倾向于使用 bw(自定义的) 作 为我自己的命名空间，你也可以选择适合你的命名空间名称。

4、使用 using 进行类型别名

本书对类型别名使用的是 using，而不是 typedef。 

STL 类和类型有时会很冗长。例如，模板化迭代器类可能长这样:

```C++
std::vector<std::pair<int,std::string>>::iterator
```

长类型名不仅很难输入，而且容易出错。

一种常见的方式是用 typedef 缩写长类型名:

```C++
typedef std::vector<std::pair<int,std::string>>::iterator vecit_t
```

这为迭代器类型声明了一个别名。typedef 继承自 C 语言，其语法反映了这一点。 

从 C++11 开始，using 关键字可以用来创建类型别名:

```C++
using vecit_t = std::vector<std::pair<int,std::string>>::iterator;
```

通常，using 别名等同于 typedef。最重要的区别是 using 别名可以模板化:

```C++
template<typename T>
using v = std::vector<T>;
v<int> x{};
```

本书更倾向于使用 using 来处理类型别名。

5、简化函数模板

从 C++20 开始，可以在没有模板文件的情况下指定简化函数模板。例如:

```C++
void printc(const auto& c) {
    for (auto i : c) {
		std::cout << i << '\n';
		}
}

```

参数列表中的 auto 类型类似于匿名模板 typename:

```C++
template<typename C>
void printc(const C& c) {
	for (auto i : c) {
		std::cout << i << '\n';
	}
}
```

虽然才在 C++20 标准中出现，但主流编译器早已支持简化函数模板了。本书将在许多示例中使用简化函数模板。

6、C++20 的 format() 

C++20 前，我们可以选择使用传统的 printf() 或 STL cout 来格式化文本。虽然两者都有缺陷，但确实好用。从 C++20 开始，format() 函数的加入提供了文本格式化，灵感来自 Python3。

# 第 1 章  C++20 的新特性

## 1.2. 格式化文本

`.ixx`后缀的文件一般专用于提供内敛定义。`.ixx` 为后缀的文件通常是 **C++模块接口文件**（Module Interface File），主要用于现代C++（C++20及以上版本）中的模块化编程。

### format (C++20)

新格式库位于 <format>头文件中。可以使用其参考实现进行理解 fmt.dev(`j.bw.org/fmt`)。

https://cppreference.com/w/cpp/header/format.html

**格式化字符串使用大括号 {} 作为类型安全的占位符，可以将任何兼容类型的值转换为合理的字符串表示形式。**

format() 函数本身返回一个字符串对象。 若想打印字符串，需要使用 iostream 或 cstdio。

```C++
//函数签名
template< class... Args >
std::string format( std::format_string<Args...> fmt, Args&&... args );
```

```C++
cout << format("Hello, {}", who) << "\n";  //iostream
puts(format("Hello, {}", who).c_str());  //cstdio
//这两种方法都不理想，但是编写一个简单的 print() 函数并不难
```

```C++
#include <format>
#include <iostream>
#include <set>
#include <string>
#include <string_view>

template<typename... Args>   // 可变参数模板声明
std::string dyna_print(std::string_view rt_fmt_str, Args&&... args)     // Args&&... args  这是一个 C++ 的可变参数模板和完美转发的语法。
{
	return std::vformat(rt_fmt_str, std::make_format_args(args...));
}

int main()
{
#ifdef __cpp_lib_format_ranges
	const std::set<std::string_view> containers
	{
		"Africa",   "America",      "Antarctica",
		"Asia",     "Australia",    "Europe"
	};
	std::cout << std::format("Hello {}!\n", containers);
#else
	std::cout << std::format("Hello {}!\n", "container");
#endif

	std::string fmt;
	for (int i{}; i != 3; ++i)
	{
		fmt += "{} ";
		std::cout << fmt << " : ";
		std::cout << dyna_print(fmt, "alpha", 'Z', 3.14, "unused");
		std::cout << "\n";
	}
}
```

完整的工作原理：

```C++

// 调用：dyna_print("Value: {}", some_variable)

// 1. 模板参数推导：Args 被推导为 some_variable 的类型
// 2. 参数包 args 包含 some_variable
// 3. args... 展开为函数参数
// 4. std::make_format_args(args...) 创建格式化参数
// 5. std::vformat 执行实际格式化
```

```
Hello {"Africa", "America", "Antarctica", "Asia", "Australia", "Europe"}!
{}  : alpha
{} {}  : alpha Z
{} {} {}  : alpha Z 3.14
```

`typename... Args`

- `...` 表示**参数包**（parameter pack）
- `Args` 是一个模板参数包，可以接受 0 个或多个类型

`Args&&... args`

- `args` 是函数参数包，包含所有传递给函数的参数
- `&&` 与模板参数结合时成为**万能引用**（universal reference）

<font color=red>总结</font>

`Args&&... args` 的含义：

- **`Args...`**：可变类型参数包
- **`args...`**：可变函数参数包
- **`&&`**：万能引用，保持值类别（左值/右值）
- **完美转发**：保留参数的原始类型和值类别信息

这种设计使得 `dyna_print` 函数能够：

- 接受任意数量和类型的参数
- 高效地处理各种值类别
- 与 `std::format` 系统无缝集成

```C++
// 模板实例化
template<>
std::string dyna_print<int, const char*>(std::string_view rt_fmt_str, int&& arg1, const char*&& arg2)
{
    return std::vformat(rt_fmt_str, std::make_format_args(arg1, arg2));
    //                                            ↑↑↑↑↑↑↑↑
    //                               参数包展开：args... → arg1, arg2
}
```

```C++
std::string s = "hello";
int x = 42;

// 保留参数原始的值类别
dyna_print("{} {}", s, 123);
// 重点: s 作为左值传递，123 作为右值传递
```

### 参数包的各种操作（不是很懂）

#### 获取参数包大小：

```C++
template<typename... Args>
void print_count(Args&&... args) {
    std::cout << "Number of arguments: " << sizeof...(Args) << std::endl;
    std::cout << "Number of args: " << sizeof...(args) << std::endl;
}
```

#### 递归展开参数包：

```C++
// 基准情况
void print_all() {}

// 递归情况
template<typename T, typename... Rest>
void print_all(T&& first, Rest&&... rest) {
    std::cout << first << " ";
    print_all(std::forward<Rest>(rest)...);
}
```

#### 折叠表达式（C++17）：

```C++
template<typename... Args>
auto sum(Args&&... args) {
    return (args + ...);  // 折叠表达式
    // 等价于：arg1 + arg2 + ... + argN
}
```

#### 与 `std::make_format_args` 的关系

```C++
template<typename... Args>
std::string dyna_print(std::string_view rt_fmt_str, Args&&... args)
{
    // std::make_format_args(args...) 将参数包转换为 format_args
    // 它会正确处理各种值类别和类型
    return std::vformat(rt_fmt_str, std::make_format_args(args...));
}
```

### string_view(C++17)

`<string_view>` 是 C++17 引入的头文件，提供了非拥有（non-owning）的字符串视图，用于高效地处理字符串而不进行内存分配。

主要特点：

- 非拥有语义：不管理字符串内存，只是观察（view）现有字符串
- 零拷贝：不会复制字符串数据
- 轻量级：通常只包含指针和长度信息
- 只读：不能通过 `string_view` 修改底层字符串

### print(C++23)

```C++
#include <format>
#include <string_view>
#include <cstdio>

template<typename... Args>
	void print(const string_view fmt_str, Args&&... args) {
	auto fmt_args{ make_format_args(args...) };
	string outstr{ vformat(fmt_str, fmt_args) };
	fputs(outstr.c_str(), stdout);
}
```

`make_format_args() 函数的作用`: 接受参数包并返回一个对象，该对象包含适合格式化的已擦除 类型的值。然后，将该对象传递给 vformat()，vformat() 再返回适合打印的字符串。我们使用 fputs() 将值输出到控制台上 (这比 cout 高效得多)。

另一种描述: 接受一个参数包并返回 format_args 对象。vformat() 函数的 作用是: 接受格式字符串和 format_args 对象，并返回一个 std::string。然后，使用 c_str() 方法来获取 用于 fputs() 的 C 字符串。

```C++
print("Hello, {}!\n", who);
print("π: {}\n", pi);
print("Hello {1} {0}\n", ival, who);
print("{:.^10}\n", ival);
print("{:.5}\n", pi);
```

<font color = red>`__cpp_lib_format` 宏</font>

这是 C++ 标准库的特性测试宏（Feature Test Macro）：

- 当编译器支持 C++20 的 `<format>` 头文件时，会定义这个宏
- 开发者可以通过检查这个宏来判断是否可以使用标准库的格式化功能

```C++
#ifdef __cpp_lib_format
    #include <format>
    #define formatter std::formatter
#else
    #include <fmt/format.h>
    #define formatter fmt::formatter
#endif
```

***There’s more…***

当格式系统遇到要转换的对象时，其会寻找具有相应类型的格式化程序对象的特化。标准的特化对于常见的对象，如字符串和数字等。Frac 类型的特化非常简单:

```C++
#include <bwprint.h>  //bwprint-module.ixx
#include <string_view>
#include <string>

#ifdef __cpp_lib_format
#define formatter std::formatter
#else
#define formatter fmt::formatter
#endif // __cpp_lib_format

using std::string_view;
using std::string;
using std::print;

struct Frac {
    long n;
    long d;
};

// 为 Frac 类型特化 formatter 模板
template<>
struct formatter<Frac> {      //为 Frac 类型特化 formatter 模板类。相当于告诉格式化库："我知道如何格式化 Frac 类型"。
    // 解析格式说明符（如 {:10.2f} 中的 "10.2f"）
    template<typename ParseContext>
    constexpr auto parse(ParseContext& ctx) {    //parse 方法: 解析格式字符串中的格式说明符。
        return ctx.begin();    //return ctx.begin(); 表示"我不处理任何格式说明符，直接返回"
    }

    // 执行实际的格式化操作
    template<typename FormatContext>
    auto format(const Frac& f, FormatContext& ctx) {   //const Frac& f：要格式化的分数对象,FormatContext& ctx：格式化上下文，包含输出位置等信息
        return format_to(ctx.out(), "{0:d}/{1:d}", f.n, f.d);  //ctx.out()：输出迭代器，指向格式化结果应该写入的位置。format_to()：将格式化结果写入指定输出迭代器的函数。"{0:d}/{1:d}"：格式化字符串，将分数显示为 分子/分母
    }
};

int main() {
    Frac f{ 5, 3 };
    print("Frac: {}\n", f);
}
```

**执行步骤**：

1. `print` 遇到 `{}`，发现需要格式化 `Frac` 类型
2. 查找 `formatter<Frac>` 特化版本
3. 调用 `parse()` 方法解析格式说明符（这里是空）
4. 调用 `format()` 方法生成格式化结果
5. 将 `"5/3"` 插入到输出中

**格式化特化，是具有两个简短模板函数的类: **

• parse() 函数解析格式字符串，从冒号之后 (若没有冒号，则在开大括号之后) 直到但不包括结 束大括号 (就是指定对象类型的部分)。其接受一个 ParseContext 对象，并返回一个迭代器。这 里，可以只返回 begin() 迭代器。因为我们的类型不需要新语法，所以无需准备任何东西。

 • format() 函数接受一个 Frac 对象和一个 FormatContext 对象，返回结束迭代器。format_to() 函 数可使这变得很容易，其可以接受一个迭代器、一个格式字符串和一个参数包。本例中，参 数包是 Frac 类的两个属性，分子和分母。 需要做的就是提供一个简单的格式字符串“{0}/{1}”以及分子和分母的值 (0 和 1 表示参数的 位置)。

## 1.3. constexpr——使用编译时 vector 和字符串

C++20 允许在新的上下文中使用 constexpr，这些语句可以在编译时计算，从而提高了效率。

C++20 开始，标准 string 和 vector 类具有限定的构造函数和析构函数，这是可在编译时使用的 前提。所以，分配给 string 或 vector 对象的内存，也必须在编译时释放。

但是若试图在运行时环境中使用结果，会得到一个在常量求值期间分配内存的错误:

```C++
int main() {
	constexpr auto vec = use_vector();
	return vec[0];
}
```

正确的代码：

```C++
#include<iostream>
#include<string>
#include<vector>
#include <numeric>
#include<print>
constexpr auto use_string()
{
	std::string str{ "string" };
	return str.size();
}

constexpr auto use_vector()
{
	std::vector<int> vec{ 1,2,3,4,5 };
	//return accumulate(begin(vec), end(vec), 0);
	return vec;
}

int main()
{
	//constexpr auto value = use_vector();
	constexpr auto vec = use_vector().size();  //因为 size() 方法是 constexpr 限定的，所以表达式可以在编译时求值。

	//std::print("{}", value);
	std::print("{}", vec);
	return 0;
}
```

## 1.4. 安全地比较不同类型的整数

### utility ( C++ library headers ) 

C++20 标准在<utitlity>头文件中包含了一组整数安全的比较函数。

#### 常用函数：swap   exchange   move  declval   make_pair

https://cppreference.com/w/cpp/utility/intcmp.html

```C++
#include <utility>
int main() {
	int x{ -3 };
	unsigned y{ 7 };
	if(cmp_less(x, y)) puts("true");
	else puts("false");
}

```

函数实现

```C++
template< class T, class U >
constexpr bool cmp_less( T t, U u ) noexcept
{
	using UT = make_unsigned_t<T>;      // template<class T>
    									//using make_unsigned_t = typename make_unsigned<T>::type;
	using UU = make_unsigned_t<U>;
	if constexpr (is_signed_v<T> == is_signed_v<U>)
	return t < u;
	else if constexpr (is_signed_v<T>)
	return t < 0 ? true : UT(t) < u;
	else
	return u < 0 ? false : t < UU(u);
}
```

UT 和 UU 别名声明为 make_unsigned_t，这是 C++17 引入的一种辅助类型。

其允许有符号类型 到无符号类型的安全转换。 

函数首先测试两个参数是有符号的，还是无符号的。然后，返回一个简单的比较。 然后，测试两边是否有符号。若该带符号值小于零，则可以返回 true 或 false 而不执行比较。否 则，将有符号值转换为无符号值进行比较。 

相同的逻辑也适用于其他比较函数。

### type_traits ( C++11 )  : This header is part of the [metaprogramming](https://cppreference.com/w/cpp/meta.html) library.

<font color="red">头文件中大多都是 is_xxx()  函数</font>

`make_unsigned_t` 是 C++ 标准库中的**类型特性（type trait）**，定义在 `<type_traits>` 头文件中。

`<type_traits>` 头文件包含丰富的类型转换工具：

```C++
namespace std {
    template<typename T>
    struct make_unsigned;          // 主模板
    
    template<typename T>
    using make_unsigned_t = typename make_unsigned<T>::type;  // C++14 起
}
```

```C++
#include <type_traits>
#include <iostream>

int main() {
    // 获取无符号版本的类型
    using UnsignedInt = std::make_unsigned_t<int>;        // unsigned int
    using UnsignedLong = std::make_unsigned_t<long>;      // unsigned long
    using UnsignedChar = std::make_unsigned_t<char>;      // unsigned char
    
    // 验证类型
    std::cout << std::is_same_v<UnsignedInt, unsigned int> << std::endl;      // 1 (true)
    std::cout << std::is_same_v<UnsignedLong, unsigned long> << std::endl;    // 1 (true)
    
    return 0;
}
```

## 1.5. 三向比较运算符 <=>  ---- 进行三种比较

### compare (C++ 20)

`<compare>` 头文件是 C++20 引入的，主要用于**三路比较操作**和**定义比较类别**。它提供了强大的比较功能，简化了自定义类型的比较操作。

这三种方式的比较方式有所不同，会返回三种状态之一。若操作数相等，三向操作符将返回一 个等于 0 的值，若左操作数小于右操作数则返回负数，若左操作数大于右操作数则返回正数。

1. 比较类别类型 (Comparison Categories)

```C++
#include <compare>

// 主要的比较类别
std::strong_ordering    // 强序（完全可比较）
std::weak_ordering      // 弱序（等价但不完全相等）
std::partial_ordering   // 偏序（可能存在不可比较的值）
```

```C++
#include <compare>
#include <iostream>

struct Point {
    int x, y;
    
    // C++20: 自动生成所有比较操作符
    auto operator<=>(const Point&) const = default;
};

int main() {
    Point p1{1, 2}, p2{1, 3}, p3{1, 2};
    
    std::cout << (p1 == p3) << std::endl;  // true
    std::cout << (p1 < p2) << std::endl;   // true
    std::cout << (p1 <= p3) << std::endl;  // true
    
    // 使用三路比较
    auto result = p1 <=> p2;
    if (result < 0) {
        std::cout << "p1 < p2" << std::endl;
    }
}
```

```C++
#include <iostream>
#include <compare>

struct Frac {
	int n;
	int d;
	constexpr Frac(int a, int b) : n{ a }, d{ b } {}
	constexpr double dbl() const {
		return static_cast<double>(n) /
			static_cast<double>(d);
	}
	constexpr auto operator<=>(const Frac& rhs) const {
		return dbl() <=> rhs.dbl();

	};
	constexpr auto operator==(const Frac& rhs) const {
		return dbl() <=> rhs.dbl() == 0;

	};
};


constexpr Frac a(10, 15); // compares equal with 2/3
constexpr Frac b(2, 3);
constexpr Frac c(5, 3);

int main() {
	static_assert(a < c);
	static_assert(c > a);
	static_assert(a == b);
	static_assert(a <= b);
	static_assert(a <= c);
	static_assert(c >= a);
	static_assert(a != c);

	return 0;
}
```

本例中，需要定义运算符 <=> 的重载，因为数据成员不是独立的标量。重载很简单，而且效果很好。 

注意，还需要一个运算符 == 重载。因为表达式重写规则不会使用自定义操作符 <=> 重载重写 == 和!=，所以需要定义操作符 ==，从而编译器会根据需要重写!= 表达式。

## 1.6.  version头文件-----查找特性宏

### version(C++ 20)

只要添加了新特性，C++ 就会提供了某种形式的特性测试宏。C++20 起这个过程标准化了，所有库特性的测试宏，都会放到头文件中。这将使测试代码中的新特性变得更加容易。

<font color='red'>所有特性测试宏都以前缀__cpp_开头。库特性以__cpp_lib_开头。</font>语言特性测试宏通常由编译器定义，库特性测试宏定义在新的头文件中。可以像使用其他预处理器宏一样使用:

https://cppreference.com/w/cpp/utility/feature_test.html

```C++
#include <version>
#ifdef __cpp_lib_three_way_comparison
# include <compare>
#else
# error Spaceship has not yet landed
#endif
```

某些情况下，可以使用__has_include 预处理器操作符 (C++17) 来测试是否包含了某个文件。

```C++
#if __has_include(<compare>)
# include <compare>
#else
# error Spaceship has not yet landed
#endif
```

因为它是一个预处理器指令，所以可以使用__has_include 来测试头文件是否存在。

```C++
#include <iostream>
using namespace std;

int main() {
//把下面的代码放到主函数外面会报错
#include <version>
#ifdef __cpp_lib_three_way_comparison
	cout << "value is " << __cpp_lib_three_way_comparison << "\n";
#endif
	return 0;
}
```

## 1.7. 概念 (concept) 和约束 (constraint)——创建更安全的模板

1. 基本概念（Concepts）:  概念是一种命名约束，用于在编译时检查模板参数是否满足特定要求。

```C++
#include <concepts>
#include <iostream>
#include <vector>
#include <list>

template<typename T>
requires std::integral<T>   // 要求 T 是整型
T add(T a, T b)
{
	return a + b;
}
// 使用简写语法
auto multiply(std::integral auto a, std::integral auto b)
{
	return a * b;
}

int main()
{
	std::cout << add(5, 3) << std::endl;
	//std::cout << add(5.5, 3.3) << std::endl;   //错误
	std::cout << multiply(4, 6) << std::endl;   // 正确
	return 0;
}
```

2. 自定义概念 : 

```C++
#include <concepts>
#include <iostream>

// 定义自定义概念
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;  // 要求 a+b 返回 T 类型
    { a += b } -> std::same_as<T&>; // 要求 a+=b 返回 T& 类型
};

template<typename T>
concept Printable = requires(T t) {
    std::cout << t;  // 要求可以用 std::cout 输出
};

// 使用自定义概念
template<Addable T>
T sum(T a, T b) {
    return a + b;
}

void print(Printable auto const& value) {
    std::cout << value << std::endl;
}

// 测试类
struct Point {
    int x, y;
    
    Point operator+(const Point& other) const {
        return Point{x + other.x, y + other.y};
    }
    
    Point& operator+=(const Point& other) {
        x += other.x;
        y += other.y;
        return *this;
    }
    
    // 不太懂这些ostream 输入输出的原理，以及friend关键字的作用
    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        return os << "Point(" << p.x << ", " << p.y << ")";
    }
};

int main() {
    std::cout << sum(10, 20) << std::endl;      // 正确：int 满足 Addable
    std::cout << sum(Point{1,2}, Point{3,4});   // 正确：Point 满足 Addable
    
    print(42);              // 正确：int 满足 Printable
    print(Point{5, 6});     // 正确：Point 满足 Printable
    // print(std::vector<int>{}); // 错误：vector 不满足 Printable
    
    return 0;
}
```

3. 复合约束

```C++
// 这个案例是模仿(x^y)
#include <concepts>
#include <iostream>
#include <vector>

template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<typename T>
concept Container = requires(T container) {
    // 含义：检查类型 T 是否有一个名为 value_type 的嵌套类型。
	// 相当于：T 必须定义 using value_type = ...; 或 typedef ... value_type;
    typename T::value_type;          // 必须有 value_type
    
    // 含义：检查类型 T 是否有一个名为 iterator 的嵌套类型。
    // 相当于：T 必须定义 using iterator = ...;
    typename T::iterator;            // 必须有 iterator
    { container.begin() } -> std::same_as<typename T::iterator>;
    { container.end() } -> std::same_as<typename T::iterator>;
    { container.size() } -> std::integral;
};

template<typename T>
concept NumericContainer = Container<T> && Numeric<typename T::value_type>;

// 使用复合约束
template<NumericContainer C>
auto sum_container(const C& container) {
    typename C::value_type total{};   //typename C::value_type - 类型声明   total - 变量名    {} - 初始化方式
    for (const auto& element : container) {
        total += element;
    }
    return total;
}

template<Container C>
void print_container(const C& container) {
    for (const auto& element : container) {
        std::cout << element << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::vector<int> numbers{1, 2, 3, 4, 5};
    std::vector<double> doubles{1.1, 2.2, 3.3};
    
    std::cout << "Sum: " << sum_container(numbers) << std::endl;
    std::cout << "Sum: " << sum_container(doubles) << std::endl;
    
    print_container(numbers);
    
    return 0;
}
```

4. 约束的多种写法

```C++
#include <concepts>
#include <iostream>

template<typename T>
concept Arithmetic = std::integral<T> || std::floating_point<T>;

// 写法1：requires 子句
template<typename T>
requires Arithmetic<T>
T power1(T base, int exponent) {
    T result = 1;
    for (int i = 0; i < exponent; ++i) {
        result *= base;
    }
    return result;
}

// 写法2：尾置 requires
template<typename T>
T power2(T base, int exponent) requires Arithmetic<T> {
    T result = 1;
    for (int i = 0; i < exponent; ++i) {
        result *= base;
    }
    return result;
}

// 写法3：模板参数中使用概念
template<Arithmetic T>
T power3(T base, int exponent) {
    T result = 1;
    for (int i = 0; i < exponent; ++i) {
        result *= base;
    }
    return result;
}

// 写法4：简写函数模板
auto power4(Arithmetic auto base, int exponent) {
    decltype(base) result = 1;
    for (int i = 0; i < exponent; ++i) {
        result *= base;
    }
    return result;
}

int main() {
    std::cout << power1(2, 3) << std::endl;    // 8
    std::cout << power2(2.5, 2) << std::endl;  // 6.25
    std::cout << power3(3, 3) << std::endl;    // 27
    std::cout << power4(1.5, 3) << std::endl;  // 3.375
    
    return 0;
}
```

5. 高级用法：SFINAE 的现代替代

```C++
#include <concepts>
#include <iostream>
#include <type_traits>
#include <vector>
// 传统 SFINAE 方式（C++17 及之前）
template<typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
T old_way(T value) {
    return value * 2;
}

// C++20 概念方式
template<typename T>
requires std::integral<T>
T new_way(T value) {
    return value * 2;
}

// 更复杂的约束示例
template<typename T>
concept HasSizeAndEmpty = requires(T t) {
    { t.size() } -> std::integral;
    { t.empty() } -> std::convertible_to<bool>;
};

template<HasSizeAndEmpty Container>
void process_container(const Container& container) {
    std::cout << "Size: " << container.size() 
              << ", Empty: " << (container.empty() ? "true" : "false") 
              << std::endl;
}

// 约束的否定
template<typename T>
concept NotPointer = !std::is_pointer_v<T>;

template<NotPointer T>
void safe_process(T value) {
    std::cout << "Processing: " << value << std::endl;
}

int main() {
    std::vector<int> vec{1, 2, 3};
    std::vector<int> empty_vec;
    
    process_container(vec);        // Size: 3, Empty: false
    process_container(empty_vec);  // Size: 0, Empty: true
    
    safe_process(42);              // 正确
    // safe_process(nullptr);      // 错误：指针不满足 NotPointer
    
    return 0;
}
```

6. 概念与重载

```C++
#include <concepts>
#include <iostream>
#include <ranges>
#include <vector>
// 为不同类型提供不同的实现
// 这个函数跟最后一个函数冲突了
//template<typename T>
//void process(T value) {
//    std::cout << "Generic: " << value << std::endl;
//}

template<std::integral T>
void process(T value) {
    std::cout << "Integral: " << value << " (square: " << value * value << ")" << std::endl;
}

template<std::floating_point T>
void process(T value) {
    std::cout << "Floating point: " << value << " (reciprocal: " << 1.0 / value << ")" << std::endl;
}

template<std::ranges::range T>   // 范围 可以作为约束吗？
void process(const T& container) {
    std::cout << "Container with " << std::ranges::size(container) << " elements: ";
    for (const auto& elem : container) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;
}

int main() {
    process(42);                    // 调用 integral 版本
    process(3.14);                  // 调用 floating_point 版本
    process("hello");               // 调用 generic 版本（字符串字面量）
    
    std::vector<int> vec{1, 2, 3, 4, 5};
    process(vec);                   // 调用 range 版本
    
    return 0;
}
```

## 1.8. 模块——避免重新编译模板库（看视频）



## 1.9. 范围容器中创建视图

### ranges (C++ 20)

范围库是 C++20 中添加的重要特性，可为过滤和处理容器提供了一种新的范例。

•“范围”是一个可以迭代的对象的集合，支持 begin() 和 end() 迭代器的结构都是范围。这包括 大多数 STL 容器。

 •“视图”是转换另一个基础范围的范围。视图是惰性的，只在范围迭代时操作。视图从底层范 围返回数据，不拥有任何数据。视图的运行时间复杂度是 O(1)。

 • 视图适配器是一个对象，可接受一个范围，并返回一个视图对象。视图适配器可以使用 | 操作 符连接到其他视图适配器。

<font color = red>视图是一个对象，操作一个范围并返回一个修改后的范围。视图为惰性操作的，不包含自己的 数据。不保留底层数据的副本，只是根据需要返回底层元素的迭代器。</font>

```C++
#include <iostream>
#include <vector>
#include <ranges>

namespace ranges = std::ranges;
namespace views = std::ranges::views;
using std::cout;

auto even = [](auto i) { return 0 == i % 2; };
auto times2 = [](auto i) { return i * 2; };

auto nl = []() { cout << "\n"; };
auto printr = [](auto r) { for (auto e : r) cout << e << " "; nl(); };

int main() {
    std::vector<int> v{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    printr(v | views::take(5));
    printr(v | views::reverse);
    printr(views::iota(100) | views::take(12) | views::reverse);
    printr(v | views::filter(even));
    printr(v | views::filter(even) | views::transform(times2));
}
```

### algorithm (C++ 20)

从 C++20 开始， 头文件中的大多数算法都会基于范围。这些版本在头文件中，但是在 std::ranges 命名空间中，这将它们与传统算法区别开来。

`export import std.core;` 是 C++23 中引入的标准库模块语法，但目前在实际使用中需要注意一些限制。

- **`import std.core`**：导入标准库的核心模块
- **`export`**：将导入的模块重新导出，使得导入当前模块的用户也能访问 `std.core`

# 第 2 章   STL的泛型特性

## 2.2. span 类——使 C 语言数组更安全

### span (C++ 20)

对于 C++20，std::span 类是一个包装器，可在连续的对象序列上创建视图。span 没有属于自己 的数据，其引用底层结构中的数据。可以把它看作 C 数组的 string_view，底层结构可以是 C 数组、 vector 或 STL array。

span 的目的是封装原始数据，并以最小的开销保证安全性和实用性。

也就是说，span 类本身不拥有数据，数据属于底层数据结构。span 本质上是底层数据的视图，并提供了一 些有用的成员函数。

`span`

```C++
#include <format>
#include <iostream>
#include <span>

using std::format;
using std::cout;
using std::span;

template<typename T>
void pspan(span<T> s) {
    cout << format("number of elements: {}\n", s.size());
    cout << format("size of span: {}\n", s.size_bytes());
    for(auto e : s) cout << format("{} ", e);
    cout << "\n";
}

int main() {
    int ca1[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    pspan<int>(ca1);
    
    return 0;
}
```

`std::dynamic_extent`

```C++
#include <array>
#include <cassert>
#include <cstddef>
#include <iostream>
#include <span>
#include <string_view>
#include <vector>

int main()
{
	auto print = [](std::string_view const name, std::size_t ex)
		{
			std::cout << name << ", ";
			if (std::dynamic_extent == ex)
				std::cout << "dynamic extent\n";
			else
				std::cout << "static extent = " << ex << '\n';
		};
	int a[]{ 1,2,3,4,5 };

	std::span span1{ a };
	print("span1", span1.extent);
	
    // span2: 显式指定为动态范围
	std::span<int, std::dynamic_extent> span2{ a };
	print("span2", span2.extent);

	std::array ar{ 1,2,3,4,5 };
	std::span span3{ ar };
	print("span3", span3.extent);

	std::vector v{ 1,2,3,4,5 };
	std::span span4{ v };
	print("span4", span4.extent);
}
```

这段代码展示了如何使用`std::span`来创建不同容器（数组、std::array、std::vector）的视图，并检查它们的**extent**（范围大小）。

`关键概念解释`

`extent`（范围）

- **extent** 表示`std::span`知道的元素数量
- 可以是**静态的**（编译时已知）或**动态的**（运行时确定）

`dynamic_extent`（动态范围）

- **作用**：表示`std::span`的大小在编译时未知，需要在运行时确定
- **值**：通常定义为`std::size_t`的最大值
- **使用场景**：当span需要引用大小在编译时不确定的数据时

`实际意义`

- **静态extent**：编译时优化机会，更安全（边界检查）
- **动态extent**：更灵活，可以处理运行时确定大小的数据

`std::span`的主要作用是提供对连续内存序列的统一视图，而不拥有数据。

## 2.3. 结构化绑定

结构化绑定使用 pair、tuple、array 和 struct。C++20 起，还会包括位域。

因为结构化绑定使用自动类型推断，所以类型必须是 auto。

```C++
things_pair<int,int> { 47, 9 };   // things_pair的出处, 但<utility>和<regex>头文件中有类似的表达式
auto [this, that] = things_pair;

int nums[] { 1, 2, 3, 4, 5 };
auto [ a, b, c, d, e ] = nums;

array<int,5> nums { 1, 2, 3, 4, 5 };
auto [ a, b, c, d, e ] = nums;

tuple<int, double, string> nums{ 1, 2.7, "three" };
auto [ a, b, c ] = nums;
    
struct Things { int i{}; double d{}; string s{}; }
Things nums{ 1, 2.7, "three" };
auto [ a, b, c ] = nums;

// 可以使用带有结构化绑定的引用，可以修改绑定容器中的值，同时避免数据复制:
array<int,5> nums { 1, 2, 3, 4, 5 };
auto& [ a, b, c, d, e ] = nums;
    
// 可以声明数组 const 来避免值的改变
const array<int,5> nums { 1, 2, 3, 4, 5 };
auto& [ a, b, c, d, e ] = nums;
    
//或者可以声明为 const 绑定来达到同样的效果，同时允许在其他地方更改数组，从而避免复制数据
const array<int,5> nums { 1, 2, 3, 4, 5 };
const auto& [ a, b, c, d, e ] = nums;
```

## 2.4. if 和 switch 语句中初始化变量

```C++
if(auto var{ init_value }; condition) {
	// var is visible
} else {
	// var is visible
}
// var is NOT visible
```

```C++
switch(auto var{ init_value }; var) {
	case 1: ...
	case 2: ...
	case 3: ...
	...
	Default: ...
}
// var is NOT visible
```

一个有趣的用例是限制锁定互斥锁的 lock_guard 的作用域。使用初始化表达式，会让代码变得 更简单:

```C++
if (lock_guard<mutex> lg{ my_mutex }; condition) {
	// interesting things happen here
}
```

lock_guard 在构造函数中锁定互斥量，在析构函数中解锁互斥量。过去，必须删除它或将整个 if 语句括在一个额外的大括号块中。现在，当 lock_guard 超出 if 语句的作用域时，将自动销毁。

## 2.5. 模板参数推导（不是很懂，需要学习万能引用）

当模板函数或类模板构造函数 (C++17 起) 的实参类型足够清楚，无需使用模板实参，编译器就 能理解时，就会进行模板实参推导。这个功能有一定的规则，但主要规则是很直观的。

```C++
//  template-deduction.cpp
//  as of 2021-11-01 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <typeinfo>
#include <tuple>
#include <regex>

using std::cout, std::format, std::string;
using namespace std::literals;  // for "literal string"s syntax

// simple template function 
template<typename T1, typename T2>
string f(const T1 a, const T2 b) {
    return format("{} {}", typeid(T1).name(), typeid(T2).name());
}

// simple template class
template<typename T1, typename T2, typename T3>
class Things {
    T1 v1{};
    T2 v2{};
    T3 v3{};
public:
    explicit Things(T1 p1, T2 p2, T3 p3)
        : v1{ p1 }, v2{ p2 }, v3{ p3 } {
    }
    string print() {
        return format("{}, {}, {}\n",
            typeid(v1).name(),
            typeid(v2).name(),
            typeid(v3).name()
        );
    }
};

// requires template deduction guide 
template <typename T>
class Sum {
    T v{};
public:
    template <typename... Ts>
    Sum(Ts&& ... values) : v{ (values + ...) } {}
    const T& value() const { return v; }
};

// template deduction guide 
template <typename... Ts>
Sum(Ts&& ... ts) -> Sum<std::common_type_t<Ts...>>;

int main() {
    std::pair p{47,47.0};
    std::tuple t(9, 17, 2.5);
    // simple template function
    cout << format("T1 T2: {}\n", f(47, 47L));
    cout << format("T1 T2: {}\n", f(47L, 47.0));
    cout << format("T1 T2: {}\n", f(47.0, "47"));
    cout << format("T1 T2: {}\n", f(p,t));

    // test the simple template class
    Things thing1{ 1, 47.0, "three" };

    // test the template deduction guide 
    Sum s1{ 1u, 2.0, 3, 4.0f };
    Sum s2{ "abc"s, "def" };
    auto v1 = s1.value();
    auto v2 = s2.value();

    cout << format("s1 is {} {}, s2 is {} {}",
        typeid(v1).name(), v1, typeid(v2).name(), v2);

}
```

## 2.6. if constexpr——简化编译时决策

constexpr if 语句的工作方式与普通 if 语句类似，只不过其是在编译时求值的。运行时代码将不 包含来自 constexpr if 语句的任何分支。

```C++
#include <format>
#include <iostream>
#include <type_traits>

using std::format;
using std::cout;

template<typename T>
auto value_of(const T v) {
    if constexpr (std::is_pointer_v<T>) {
        return *v;
    } else {
        return v;
    }
}

int main() {
    int x{47};
    int* y{&x};
    
    cout << format("value is {}\n", value_of(x));  // value
    cout << format("value is {}\n", value_of(y));  // pointer
    return 0;
}

```



# 第 3 章 STL 容器

### 顺序容器：

`array`   最简单和访问速度最快的连续存储容器。

`vector`     

`list`     双链表结构  (非随机访问容器)

`forward_list`     单链表  （非随机访问容器）

`deque`     双端队列，是连续的容器

### 关联容器

关联容器将一个键与每个元素关联起来。元素是通过键来引用的，而不是其在容器中的位置。：

• `set` 是一个关联容器，每个元素也是自己的键，元素通常按某种二叉树方式排序。set 中的元素 不可变，不能修改，但是可以插入和移除。set 中的元素是唯一的，不允许重复。set 可以根据 排序操作符按顺序进行迭代。 （非随机访问容器）

• `multiset` 就像一个具有非唯一键的集合，允许重复。 （非随机访问容器）

• `unordered_set` 就像一个不按顺序迭代的集合。元素不按特定顺序排序，而是根据哈希值进行 组织，以便快速访问。 

• `unordered_multiset` 类似于 unordered_set，允许重复。 

• `map` 是键-值对的关联容器，其中每个键都映射到特定的值 (或有效负载)。键和值的类型可能 不同；键是唯一的，但值不是。map 根据其排序操作符，按键的顺序进行迭代。 （非随机访问容器）

• `multimap` 类似于具有非唯一键的映射，允许重复键。 （非随机访问容器）

• `unordered_map` 就像一个没有按顺序迭代的 map。 

• `unordered_multimap` 类似于 unordered_map，允许重复。

### 容器适配器

容器适配器是封装底层容器的类，容器类提供了一组特定的成员函数来访问底层容器元素。

• `stack` 提供了后进先出 (LIFO) 接口，该接口中只能从容器的一端添加和提取元素。底层容器可 以是 vector、deque 或 list 中的一种。若没有指定底层容器，默认为 deque。 

• `queue` 提供了先进先出 (FIFO) 接口，其中元素可以在容器的一端添加，并从另一端提取。底 层容器可以是 deque 或 list 容器之一。若没有指定底层容器，默认为 deque。 

• `priority_queue` 按照严格的弱顺序将最大的值元素保持在顶部，以对数时间插入和提取为代价， 提供了对最大值元素的常数时间查找。底层容器可以是 vector 或 deque 中的一个。若没有指 定底层容器，默认为 vector。

### 字符串

- `std::string` - 字符串
- `std::string_view` - 字符串视图

### 适配器

- `std::span` (C++20) - **连续序列视图**

## 3.3. 使用擦除函数从容器中删除项

C++20 前，erase-remove 通常用于从 STL 容器中删除元素。这操作有点麻烦，通常使用这样的 函数来完成:

<font color='blue'>注：课本 P57 记录了有关 erase-remove 的执行流程 （How it works…）</font>

```C++
template<typename Tc, typename Tv>
	void remove_value(Tc & c, const Tv v) {  
	auto remove_it = std::remove(c.begin(), c.end(), v);     // algorithm中有remove的定义
	c.erase(remove_it, c.end());
}

```

C++ 20 中建议使用擦除函数：

erase(container, value);

容器可以是顺序容器 (vector, list, forward_list, deque)，数组除外，数组不能改变大小。

erase_if(container, predicate);

这种形式适用于使用 erase() 的容器，也适用于关联容器、set、map 及其多键和无序的版本。

<font color='red'>erase() 和 erase_if() 函数只是一次性执行 erase-remove 的包装器</font>

<font color='red'>函数 erase() 和 erase_if() 作为非成员函数定义在相应容器的头文件中。</font>

```C++
#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <list>

using std::format;
using std::cout;
using std::string, std::vector, std::map, std::list;
using std::erase, std::erase_if;

void printc(auto& r) {
    cout << format("size: {}: ", r.size());
    for( auto& e : r ) cout << format("{} ", e);
    cout << "\n";
}

void print_assoc(auto& r) {
    cout << format("size: {}: ", r.size());
    for( auto& [k, v] : r ) cout << format("{}:{} ", k, v);
    cout << "\n";
}

int main() {
    {
        puts("initialize vector:");
        vector v{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        printc(v);
        puts("erase 5:");
        erase(v, 5);
        printc(v);
    }

    {
        puts("initialize vector:");
        vector v{ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        printc(v);
        puts("erase_if even:");
        erase_if(v, [](auto x) { return x % 2 == 0; });
        printc(v);
    }

    puts("initialize map:");
    map<int, string> m{ {1, "uno"}, {2, "dos"}, {3, "tres"}, {4, "quatro"}, {5, "cinco"} };
    print_assoc(m);
    puts("erase_if even:");
    erase_if(m, [](auto& p) { auto& [k, v] = p; return k % 2 == 0; });
    print_assoc(m);
}
```

## 3.4. 常数时间内从未排序的向量中删除项（被删除元素移到末尾）

使用擦除函数 (或 erase-remove) 从 vector 中间删除项需要 O(n)(线性) 时间。因为元素必须从向 量的末尾移动，以填补删除项之间的空白。<font color='red'>若 vector 中项目的顺序不重要，就可以优化这个过程， 使其花费 O(1)(常数) 时间。</font>

<font color='blue'>注：课本 P60 记录了有关 erase-remove 的执行流程 （How it works…）</font>

```C++
#include <format>
#include <iostream>
#include <vector> 
#include <algorithm>

using std::format;
using std::cout;
using std::vector;
using std::move;
using std::ranges::find;

void printc(auto& r) {
    cout << format("size({}) ", r.size());
    for (auto& e : r) cout << format("{} ", e);
    cout << "\n";
}

template<typename T>
void quick_delete(T& v, size_t idx)
{
    if (idx < v.size())
    {
        v[idx] = move(v.back());
        v.pop_back();
    }
}

template<typename T>
void quick_delete(T& v, typename T::iterator it)
{
    if (it < v.end())
    {
        *it = move(v.back());
        v.pop_back();
    }
}

int main()
{
    vector v{ 12, 196, 47, 38, 19 };
    printc(v);
    auto it = find(v, 47);
    quick_delete(v, it);
    printc(v);
    quick_delete(v, 1);
    printc(v);
}
```

## 3.5. 安全地访问 vector 元素（at）

at() 函数执行边界检查，而 [] 操作符不检查，[] 操作符为了 保持与原始 C 数组的兼容性。

vector 类通常用作直接访问容器，而 array 和 deque 容器也同时支持 [] 操作符和 at() 成员函数。

```C++
#include <format>
#include <iostream>
#include <vector> 

using std::format;
using std::cout;
using std::vector;

int main() {
    vector v{ 19, 71, 47, 192, 4004 };
    try {
        v.at(5) = 2001;
    } catch (const std::out_of_range & e) {
        std::cout << format("Ouch!\n{}\n", e.what());
    }
    cout << format("end element is {}\n", v.back());
}
```

## 3.6. 保持 vector 元素的顺序

vector 是一个顺序容器，会按照插入元素的顺序保存元素。并不对元素进行排序，也不以任何 方式改变其顺序。其他容器，如 set 和 map，会将元素排序，但这些容器不可随机访问。不过，只 需要稍加处理，vector 也可以保持有序。

```C++
#include <format>
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

using std::format;
using std::cout;
using std::vector;
using std::string;

namespace ranges = std::ranges;

using Vstr = std::vector<std::string>;
using Vint = std::vector<int>;

// print a vector
void printv(const auto& v) {
    for (const auto& e : v) {
        cout << format("{} ", e);
    }
    cout << "\n";
}

// is it sorted? 
void psorted(const auto& v) {
    if (ranges::is_sorted(v)) cout << "sorted: ";   // psorted() 函数使用 is_sorted() 算法的范围版本，确定 vector是否已排序。
    else cout << "unsorted: ";
    printv(v);
}

// insert sorted elements
template<typename C, typename E>
void insert_sorted(C& c, const E& e)
{
    // insert_sorted() 函数用于将元素插入到已排序的 vector 中，同时保持其顺序:
    // insert_sorted() 函数使用 lower_bound() 算法的范围版本来获取 insert() 函数的迭代器，该迭代器保持 vector 的排序。
    // lower_bound() 算法查找不小于实参的第一个元素。然后，使用 lower_bound() 返回的迭代器在正确的位置插入一个元素。
    const auto pos{ ranges::lower_bound(c, e) };
    c.insert(pos, e);
}

int main() {
    Vstr v{
        "Miles",
        "Hendrix",
        "Beatles",
        "Zappa",
        "Shostakovich"
    };
    psorted(v);

    // sort it
    ranges::sort(v);  // 使用 sort() 算法的范围版本对 vector 排序。
    psorted(v);

    // insert music here
    insert_sorted(v, "Ella");
    insert_sorted(v, "Stones");
    psorted(v);

    // once more with ints! 
    Vint vi{ 192, 47, 71, 1914, 2001 };
    psorted(vi);
    ranges::sort(vi);
    psorted(vi);
    insert_sorted(vi, 300);
    insert_sorted(vi, 1999);
    psorted(vi);
}

```

<font color='red'>std::sort() 算法 (及其衍生算法) 需要支持随机访问的容器，并不是所有的 STL 容器都满足这个 要求。值得注意的是，std::list 不支持随机访问。</font>

## 3.7. 高效地将元素插入到 map 中

```C++
map<string, string> m;
m["Miles"] = "Trumpet"
m.insert(pair<string,string>("Hendrix", "Guitar"));
m.emplace("Krupa", "Drums");
```

我倾向于使用 emplace() 函数，可以**完全转发** 来为容器放置(在适当的位置创建) 新元素。参数 直接转发给元素构造函数，快速、高效且易于阅读。

尽管 emplace() 肯定是对其他选项的改进，但其问题在于，即使在不需要对象时，也会构造对象。这包括调用构造函数、分配内存、移动数据，然后丢弃临时对象。

为了解决这个问题，C++17 提供了新的 try_emplace() 函数，该函数只在需要时构造值对象，这 对于大型对象尤为重要。

map 的每个元素都是一个键值对，其中元素命名为 first 和 second，但在 map 中是键和值。我 倾向于将值对象视为有效负载。要**搜索一个现有的键，try_emplace() 函数必须构造键对象，但不需要构造有效负载对象，除非需要插入到 map 中。**

新的 try_emplace() 函数避免了构造有效负载对象的开销，这在键值碰撞的情况下效率很高，特别是在大有效载荷的情况下。

```C++
#include <format>
#include <iostream>
#include <map>
#include <unordered_map>
#include <string>
#include <algorithm>

using std::format;
using std::cout;
using std::map;
using std::unordered_map;
using std::string;

struct BigThing {
    string v_;
    BigThing(const char * v) : v_(v) {
        cout << format("BigThing constructed {}\n", v_);
    }
};

using Mymap = map<string, BigThing>;   // convenience alias

void printm(Mymap& m) {
    for(auto& [k, v] : m) {
        cout << format("[{}:{}] ", k, v.v_);
    }
    cout << "\n";
}

int main() {
    Mymap m;

    m.try_emplace("Miles", "Trumpet");
    m.try_emplace("Hendrix", "Guitar");
    m.try_emplace("Krupa", "Drums");
    m.try_emplace("Zappa", "Guitar");
    m.try_emplace("Liszt", "Piano");
    printm(m);

    cout << "\n";
    cout << "emplace(Hendrix)\n";
    m.emplace("Hendrix", "Singer");
    cout << "try_emplace(Zappa)\n";
    m.try_emplace("Zappa", "Composer");
    printm(m);

    const char * key{"Zappa"};
    const char * payload{"Composer"};
    if(auto [it, success] = m.try_emplace(key, payload); !success) {
        cout << "update\n";
        it->second = payload;
    }
    printm(m);
}
```

函数签名：

```C++
// try_emplace
pair<iterator, bool> try_emplace( const Key& k,Args&&... args );
// 区别在于 try_emplace() 为键参数使用了一个单独的形参，这允许在构造时隔离。
// emplace
pair<iterator,bool> emplace( Args&&... args );
```

## 3.8. 高效地修改 map 项的键值

map 是存储键值对的关联容器，容器是按键排序的。键必须是唯一的，并且是 const 限定的，所 以不能更改。

**若需要重新排序 map 容器，可以通过使用 extract() 方法交换键来实现。C++17 中，extract() 是 map 类，及其派生类中的成员函数。 它允许从序列中提取 map 元素，而不涉及有效负载。当提取出来时，键就不再是 const 限定的， 并且可以修改。**

```C++
#include <format>
#include <iostream>
#include <map>
#include <string>

using std::format;
using std::cout;
using std::map;
using std::string;
using std::swap;

using Racermap = map<unsigned int, string>;

void printm(const Racermap &m)
{
    cout << "Rank:\n";
    for (const auto& [rank, racer] : m) {
        cout << format("{}:{}\n", rank, racer);
    }
}

template<typename M, typename K>
bool node_swap(M & m, K k1, K k2) {
    auto node1{ m.extract(k1) };
    auto node2{ m.extract(k2) };
    if(node1.empty() || node2.empty()) {
        return false;
    }
    swap(node1.key(), node2.key());
    m.insert(move(node1));
    m.insert(move(node2));
    return true;
}

int main() {
    Racermap racers {
        {1, "Mario"}, {2, "Luigi"}, {3, "Bowser"},
        {4, "Peach"}, {5, "Donkey Kong Jr."}
    };
    printm(racers);

    node_swap(racers, 3, 5);
    printm(racers);
}
```

node_swap() 这个函数使用 map.extract() 方法从映射中提取指定的元素。这些提取出来的元素称为节点。 节点是一个从 C++17 出现的新概念，可以在不涉及元素本身的情况下从 map 类型结构中提取 元素。解除节点链接，返回节点句柄。提取后，节点句柄通过节点的 key() 函数提供对键的可写访问。然后，可以交换键并插入到 map 中，无需复制或操作有效负载。

原理：

该技术使用 extract() 函数，该函数返回一个 node_handle 对象，node_handle 是节点的句柄，由 关联元素及其相关结构组成。extract 函数在将节点保留在原处的同时将其解除关联，并返回一个 node_handle 对象。这样做的效果是从关联容器中删除节点，而不涉及数据本身。node_handle 允许 访问已解除关联的节点。 node_handle 有一个成员函数 key()，返回一个对节点键的可写引用。这就可以在键与容器解关联时，对键值进行修改。

**There' s more**

1.  若没有找到键，extract() 函数返回一个空节点句柄。可以用 empty() 函数测试节点句柄是否为空:

```C++
auto node{ mapthing.extract(key) };
if(node.empty()) {
	// node handle is empty
}
```

​	2.键插入映射时必须保持唯一。 例如，若将一个键更改为映射中已经存在的值:

```C++
auto node_x{ racers.extract(racers.begin()) };
node_x.key() = 5; // 5 is Donkey Kong Jr
auto status = racers.insert(move(node_x));
if(!status.inserted) {
	cout << format("insert failed, dup key: {}",
	status.position->second);
	exit(1);
}
```

本例中，将 begin() 迭代器传递给 extract()。然后，为键值分配了一个已经在使用的值 (5, Donkey Kong Jr)。插入失败和结果 status.inserted 为 false。status.position 是指向已找到键的迭代器。在 if() 中，我使用 format() 来打印找到的键值。

## 3.9. 自定义键值的 unordered_map

对于有序 map，键的类型必须是可排序的，必须至少支持小于比较操作符。

```C++
#include <format>
#include <iostream>
#include <unordered_map>
#include <string>

using std::format;
using std::cout;
using std::unordered_map;

struct Coord {
    int x{};
    int y{};
};

using Coordmap = unordered_map<Coord, int>;

bool operator==(const Coord& lhs, const Coord& rhs) {
    return lhs.x == rhs.x && lhs.y == rhs.y;
}

namespace std {
    template<>
    struct hash<Coord> {
        size_t operator()(const Coord& c) const {
            return static_cast<size_t>(c.x)
                + static_cast<size_t>(c.y);
        }
    };
}

void print_Coordmap(const Coordmap& m) {
    for (const auto& [key, value] : m) {
        cout << format("{{ ({}, {}): {} }} ",
            key.x, key.y, value);
    }
    cout << '\n';
}

int main() {
    Coordmap m{
        { {0, 0}, 1 },
        { {0, 1}, 2 },
        { {2, 1}, 3 }
    };
    print_Coordmap(m);

    Coord k{ 0, 1 };
    cout << format("{{ ({}, {}): {} }}\n",
        k.x, k.y, m.at(k));
}
```

**这段代码中定义相等比较运算符和特化的 `std::hash` 类是必须的，因为 `std::unordered_map` 需要它们来正确处理自定义的 `Coord` 结构体作为键类型。**

`std::unordered_map` 使用哈希表实现，当发生**哈希冲突**（不同键产生相同哈希值）时，需要比较键是否真正相等来确定是插入新元素还是更新现有元素。

`std::unordered_map` 需要计算每个键的哈希值来确定在哈希表中的存储位置。

**工作流程**：

```
插入 Coord{0, 1} 作为键时：
1. 计算哈希值：hash(Coord{0, 1}) = 0 + 1 = 1
2. 在哈希表位置1查找
3. 如果该位置已有元素，使用 operator== 比较是否相同
4. 根据比较结果决定覆盖还是插入
```

## 3.10. 使用 set 对输入进行排序和筛选（VS 执行不了这段代码）

源码看不懂

```C++
//  set-words.cpp
//  as of 2021-11-17 bw [bw.org]

#include <format>
#include <iostream>
#include <set>
#include <string>
#include <iterator>
#include <algorithm>

using std::format;
using std::cout;
using std::set;
using std::string;
using std::istream_iterator;
using std::cin;
using std::copy;
using std::inserter;

using input_it = istream_iterator<string>;

int main() {
    set<string> words;

    input_it it{ cin };
    input_it end{};

    copy(it, end, inserter(words, words.end()));
    
    for(const string & w : words) {
        cout << format("{} ", w);
    }
    cout << '\n';
}
```

## 3.11. 简单的 RPN 计算器与 deque（未做）

## 3.12. 使用 map 的词频计数器（未做）

## 3.13. 找出含有相应长句的 vector（未做）

## 3.14. 使用 multimap 制作待办事项列表（未做）

# 第 4 章 兼容迭代器

```C++
const int a[]{ 1, 2, 3, 4, 5 };
size_t count{ sizeof(a) / sizeof(int) };
for(const int* p = a; count > 0; ++p, --count) {
	cout << *p << '\n';
}
```

```C++
{
    auto begin_it{ std::begin(container) };
    auto end_it{ std::end(container) };
    for ( ; begin_it != end_it; ++begin_it) {
        auto e{ *begin_it };
        cout << e << '\n';
    }
}

```

<img src="C:\Users\86178\AppData\Roaming\Typora\typora-user-images\image-20251005153556452.png" alt="image-20251005153556452" style="zoom:80%;" />

<img src="C:\Users\86178\AppData\Roaming\Typora\typora-user-images\image-20251005163816813.png" alt="image-20251005163816813" style="zoom:80%;" />

<img src="C:\Users\86178\AppData\Roaming\Typora\typora-user-images\image-20251005163846508.png" alt="image-20251005163846508" style="zoom:80%;" />

## 4.3. 创建可迭代范围

制作一个**`序列生成器`**，与基于范围的 for 循环一起工作。

支持基于范围的 for 循环的最低要求是解引用操作符、前自增操作符和不等比较操作符。

```C++
#include <format>
#include <iostream>

using std::format;
using std::cout;

template<typename T>
class Seq {
    T start_{};
    T end_{};

public:
    Seq(T start, T end) : start_{ start }, end_{ end } {}

    class iterator {
        T value_{};
    public:
        // 迭代器构造函数是限定显式的，以避免隐式转换。
        explicit iterator(T position = 0) : value_{position} {}

        T operator*() const { return value_; }

        // value_变量由迭代器维护，可以解引用指针返回一个值。
        iterator& operator++() {
            ++value_;
            return *this;
        }

        bool operator!=(const iterator& other) const {
            return value_ != other.value_;
        }
    };

    iterator begin() const { return iterator{start_}; }
    iterator end() const { return iterator{end_}; }
};

int main() {
    Seq<int> r{ 100, 110 };

    for (auto v : r) {
        cout << format("{} ", v);
    }
    cout << '\n';
    
    //Seq<int>::iterator it = r.begin();
    
    // 对序列进行左闭右开的范围打印数值
    {
	auto begin_it{ std::begin(r) };
	auto end_it{ std::end(r) };
	for (; begin_it != end_it; ++begin_it) {
			auto e{ *begin_it };
			cout << e << '\n';
		}
}
}

```

## 4.4. 使迭代器与 STL 迭代器特性兼容

typename    typedef    using    namespaces

### iterator (C++ library headers)

在C++中，`<iterator>` 库是STL（标准模板库）的重要组成部分，它提供了一系列与迭代器相关的工具和功能。迭代器本身是**一种智能指针类**，用于**遍历和访问容器中的元素**，同时隐藏了底层数据结构的实现细节。

```C++
#include <format>
#include <iostream>
#include <algorithm>
#include <ranges>
#include <iterator>

using std::format;
using std::cout;

namespace ranges = std::ranges;

template<typename T>
class Seq {
	T start_{};
	T end_{};

public:
	Seq(T start, T end) : start_(start),end_(end){}
	
	class iterator {
		T value_{};
	public:
        // iterator_traits 类在迭代器类中查找一组类型定义 (使用别名实现)
		using iterator_concept = std::forward_iterator_tag;   // iterator_concept/category - 定义迭代器能力级别
		using iterator_category = std::forward_iterator_tag; 
		using value_type = std::remove_cv_t<T>;  // value_type - 元素类型（移除const/volatile）
		using difference_type = std::ptrdiff_t;   // difference_type - 迭代器距离类型
		using pointer = const T*;      // pointer/reference - 指针和引用类型
		using reference = const T&;

		explicit iterator(T position=0):value_(position){}
		
        // 满足前向迭代器的操作符要求
		T operator * () const { return value_; }

		iterator& operator++() {
			++value_;
			return *this;
		}

		iterator operator++(T) {
			iterator temp = *this;
			++*this;
			return temp;
		}

		bool operator != (const iterator& other) const noexcept{
			return value_ != other.value_;
		}
        // 上下这两个函数可以互换
        //bool operator != (const iterator& other) const noexcept {
		//	return !(value_ == other.value_);
		//}

		bool operator==(const iterator& other) const noexcept {
			return value_ == other.value_;
		}
	};

	iterator begin() const { return iterator{ start_ }; }
	iterator end() const { return iterator{ end_ }; }
};

// 定义这些特征允许在迭代器中使用概念受限的模板
// 这个输出序列的函数受到 forward_iterator 概念的限制。若类不符合条件，就不会编译。
template<typename T>
requires std::forward_iterator<typename T::iterator>
void printc(const T& c) {
	for (auto v : c) {
		cout << format("{} ", v);
	}
	cout << '\n';
}

int main()
{
	Seq<int> r{ 100,150 };

	auto [min_it, max_it] = ranges::minmax_element(r);
	cout << format("{} - {}\n", *min_it, *max_it);

	printc(r);

	static_assert(ranges::forward_range<Seq<int>>);
}
```

using 语句是用于定义迭代器可以执行哪些功能的特性，来看一下:

- 前两个是 category 和 concept，都设置为 forward_iterator_tag。该值表示迭代器符合前向迭代器 规范。
- value_type 是 std::remove_cv_t 的别名，这是值的类型，可以删除 const 限定符。 
- difference_type 是 std::ptrdiff_t 的别名，作为指针地址差异的特殊类型。 
- 指针和引用别名分别设置为，指针和引用的 const 限定版本。 
- **定义这些类型别名是大多数迭代器的基本要求**。

## 4.5. 使用迭代器适配器填充 STL 容器

迭代器本质上是一种抽象，有一个特定的接口，并以特定的方式使用。

STL 附带了各种迭代器适配器，通常与算法库一起使用。STL 迭代器适配器通常分为三类: 

• 插入迭代器或插入器用于在容器中插入元素。 

• 流迭代器读取和写入流。 

• 反向迭代器反转迭代器的方向。

```C++
#include <format>
#include <iostream>
#include <string_view>
#include <string>
#include <vector>
#include <deque>
#include <algorithm>

using std::format;
using std::cout;
using std::cin;
using std::vector;
using std::deque;
using std::string_view;
using std::string;
using std::ostream_iterator;
using std::istream_iterator;

void printc(const auto & v, const string_view s = "") {
    if(s.size()) cout << format("{}: ", s);
    for(auto e : v) cout << format("{} ", e);
    cout << '\n';
}

int main() {
    deque<int> d1{ 1, 2, 3, 4, 5 };
    deque<int> d2(d1.size());
        
	// copy() 算法不会分配空间，因此 d2 必须为元素留有空间。
    copy(d1.begin(), d1.end(), d2.begin());
    printc(d1);
    printc(d2, "d2 after copy");

    // back_inserter() 是一个插入迭代器适配器，为分配给它的每个项调用 push_back()，可以在需要输出迭代器的地方使用。
    copy(d1.begin(), d1.end(), back_inserter(d2));
    printc(d2, "d2 after back_inserter");

    // front_inserter() 适配器使用容器的 push_front() 方法在前端插入元素。因为每个元素都插入到前一个元素之前，所以目标容器中的元素是反向的。
    deque<int> d3{ 47, 73, 114, 138, 54 };
    copy(d3.begin(), d3.end(), front_inserter(d2));
    printc(d2, "d2 after front_inserter");

    auto it2{ d2.begin() + 2};
    copy(d1.begin(), d1.end(), inserter(d2, it2));
    printc(d2, "d2 after middle insert");

    // 流迭代器可以方便地读写 iostream 对象，这是 ostream_iterator():
    cout << "ostream_iterator: ";
    copy(d1.begin(), d1.end(), ostream_iterator<int>(cout));
    cout << '\n';

    // 反向适配器
    cout << "Reverse iterator: ";
    for(auto it = d1.rbegin(); it != d1.rend(); ++it) {
        cout << format("{} ", *it);
    }
    cout << '\n';
    
    vector<string> vs{};
    copy(istream_iterator<string>(cin), istream_iterator<string>(), back_inserter(vs));
    printc(vs, "vs2 from istream");
}

```

<font color='red'>课本 P101 讲解迭代器适配器 的工作原理</font>

istream_adapter() 也需要一个哨兵，哨兵表示长度不确定的迭代器的结束。当从一个流中读取 时，并不知道流中有多少对象。当流到达结束时，哨兵将与迭代器进行相等比较，标志流的结束。 istream_adapter() 在不带参数的情况下调用时会创建一个哨兵:

```C++
// 看不懂
auto it = istream_adapter<string>(cin); // 起始迭代器标志
auto it_end = istream_adapter<string>();  // 结束迭代器标志

for(auto it = istream_iterator<string>(cin);
	it != istream_iterator<string>();
	++it) {
	cout << format("{} ", *it);
}

```

## 4.6. 创建一个迭代器生成器

生成器是生成自己的值序列的迭代器，不使用容器。它动态地创建值，根据需要一次返回一个 值。

stop_变量将在后面用作哨兵，设置为要生成的值的数量。count_用于跟踪生成了多少个值。 a_和 b_是前两个序列值，用于计算下一个值。

```C++
#include<format>
#include<iostream>
#include<string_view>
#include<algorithm>
#include<ranges>

using std::format;
using std::cout;
using std::string_view;

namespace ranges = std::ranges;

void printc(const auto& c, string_view s = "")
{
	if (s.size()) cout << format("{}", s);
	for (auto e : c) cout << format("{}", e); // 把auto 写成 auto&, 导致运行出错
	cout << '\n';
}

class fib_generator
{
	using fib_t = unsigned long;

	fib_t stop_{};
	fib_t count_{ 0 };
	fib_t a_{ 0 };
	fib_t b_{ 1 };

	constexpr void do_fib()
	{
		const fib_t old_b = b_;
		b_ += a_;
		a_ = old_b;
	}

public:
	using iterator_concept = std::forward_iterator_tag;
	using iterator_category = std::forward_iterator_tag;
	using value_type = std::remove_cv_t<fib_t>;
	using difference_type = std::ptrdiff_t;
	using pointer = const fib_t*;
	using reference = const fib_t&;

	explicit fib_generator(fib_t stop=0):stop_(stop){}

	fib_t operator *() const { return b_; }
 
	constexpr fib_generator& operator++()
	{
		do_fib();
		++count_;
		return *this;
	}

	fib_generator operator++(int)
	{
		auto temp{ *this };
		++*this;
		return temp;
	}

	bool operator!=(const fib_generator& o) const {
		return count_ != o.count_;
	}

	bool operator==(const fib_generator& o) const {
		return count_ == o.count_;
	}

	const fib_generator& begin() const { return *this; }

    // end() 函数创建了一个对象，其中 count_变量等于 stop_变量，从而创建了一个哨兵:
	const fib_generator end() const
	{
		auto sentinel = fib_generator();
		sentinel.count_ = stop_;
		return sentinel;
	}

    // 还有一个简单的 size() 函数，若需要为复制操作初始化一个目标容器，这个函数会很有用。(不明白)
	fib_t size() { return stop_; }
};

int main()
{
	printc(fib_generator(10));  // 创建一个匿名的 fib_generator 对象传递给 printc() 函数

    // 若想让生成器与算法库一起工作，需要提供特性别名。这些放在 public 的顶部:(using别名)
	fib_generator fib(10);
	auto x = ranges::views::transform(fib, [](unsigned long x) { return x * x; });
	printc(x, "squared");
}
```

## 4.7. 反向迭代器适配器的反向迭代

**就是  rbegin(),  rend()  这两个函数的用法**

反向迭代器适配器拦截迭代器接口，并将其反转。使 begin() 迭代器指向最后一个元素，end() 迭代器指向第一个元素之前。++ 和–操作符也是颠倒的

反向迭代器中，++ 操作符递减，--操作符递增。 大多数双向 STL 容器已经包含了一个反向迭代器适配器，可以通过成员函数 rbegin() 和 rend() 访问

```C++
#include <format>
#include <iostream>
#include <string_view>
#include <vector>
#include <iterator>
#include <list>

using std::format;
using std::cout;
using std::string_view;
using std::vector;


void printc(const auto & c, const string_view s = "") {
    if(s.size()) cout << format("{}: ", s);
    for(auto e : c) cout << format("{} ", e);
    cout << '\n';
}

void printr(const auto & c, const string_view s = "") {
    if(s.size()) cout << format("{}: ", s);
    auto it = std::rbegin(c);
    auto end_it = std::rend(c);
    while(it != end_it) {
        cout << format("{} ", *it++);
    }
    cout << '\n';
}

int main() {
    int array[]{ 1, 2, 3, 4, 5 };
    printc(array, "c-array");
    printr(array, "rev c-array");

    vector<int> v{ 1, 2, 3, 4, 5 };
    printc(v, "vector");
    printr(v, "rev vector");
}
```

## 4.8. 用哨兵迭代未知长度的对象

```C++
#include <format>
#include <iostream>

using std::format;
using std::cout;
using std::string_view;
using std::vector;

using sentinel_t = const char;
constexpr sentinel_t nullchar = '\0';

class cstr_it
{
	const char* s{};

public:
	explicit cstr_it(const char *str):s{str}{}

	char operator *() const { return *s; }

	cstr_it& operator++() {
		++s;
		return *this;
	}

	bool operator != (sentinel_t) const {
		return s != nullptr && *s != nullchar;
	}

	cstr_it begin() { return *this; }
	sentinel_t end() { return nullchar; }
};

void print_cstr(const char* s)
{
	cout << format("{}: ", s);
	for (char c : cstr_it(s))  // cstr_it 这里是重点
	{
		cout << format("{:02x} ", c);
	}
	std::cout << '\n';
}

int main()
{
	const char carray[]{ "array" };
	print_cstr(carray);

	const char* cstr{ "c-string" };
	print_cstr(cstr);
}
```

## 4.9. 构建 zip 迭代器适配器

```C++
#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <algorithm>
#include <utility>

using std::format;
using std::cout;
using std::string;
using std::vector;
using std::map;
using std::pair;

template<typename T>
class zip_iterator {
	using val_t = typename T::value_type;
	using ret_t = std::pair<val_t, val_t>;
	using it_t = typename T::iterator;

	// 迭代器中不存储数据，只存储目标容器的 begin() 和 end() 迭代器的副本
	it_t ita_{};
	it_t itb_{};
	// for begin() and end() objects
	it_t ita_begin_{};
	it_t itb_begin_{};
	it_t ita_end_{};
	it_t itb_end_{};

	zip_iterator(it_t ita, it_t itb) :ita_{ ita }, itb_{ itb } {}

public:
    // 定义了一个迭代器的类型特征（type traits），用于让这个类与C++标准库的迭代器系统兼容
	using iterator_concept = std::forward_iterator_tag;
	using iterator_category = std::forward_iterator_tag;
	using value_type = std::pair<val_t, val_t>;
	using difference_t = long int;
	using pointer = const val_t*;
	using reference = const val_t&;

	zip_iterator(T& a, T& b):
		ita_{a.begin()},
		itb_{b.begin()},
		ita_begin_{ita_},
		itb_begin_{itb_},
		ita_end_{a.end()},
		itb_end_{b.end()}
	{ }

	zip_iterator& operator++() {
		++ita_;
		++itb_;
		return *this;
	}

	bool operator == (const zip_iterator& o) const {
		return ita_ == o.ita_ || itb_ == o.itb_;
	}

	bool operator != (const zip_iterator& o) const {
		return !operator==(o);
	}

	ret_t operator * () const {
		return { *ita_,*itb_ };
	}

	zip_iterator begin() const {
		return zip_iterator(ita_begin_, itb_begin_);
	}
	zip_iterator end() const {
		return zip_iterator(ita_end_, itb_end_);
	}
};

int main()
{
	vector<std::string> vec_a{ "Bob", "John", "Joni" };
	vector<std::string> vec_b{ "Dylan", "Williams", "Mitchell" };

	cout << "vec_a: ";
	for (auto e : vec_a) cout << format("{} ", e);
	cout << '\n';

	cout << "vec_b: ";
	for (auto e : vec_b) cout << format("{} ", e);
	cout << '\n';

	cout << "zipped: ";
	for (auto [a, b] : zip_iterator(vec_a, vec_b)) {
		cout << format("[{}, {}]", a, b);
	}
	cout << '\n';

	map<string, string> name_map{};

	for (auto [a, b] : zip_iterator(vec_a, vec_b)) {
		name_map.try_emplace(a, b);
	}

	cout << "name_map: ";
	for (auto &[a, b] : name_map) {   // 这里用引用，避免多余的复制
		cout << format("[{}, {}] ", a, b);
	}
	cout << '\n';
}
```

**定义类型特征的作用总结**：

1. **标准库兼容性**：让自定义迭代器能与STL算法一起工作
2. **类型安全**：明确指定迭代器的特性和行为
3. **概念检查**：帮助编译器验证迭代器是否满足特定要求
4. **算法优化**：标准库算法可以根据迭代器类别选择最优实现

这样定义后，你的迭代器就可以用在`std::for_each`、`std::find`等标准算法中了。

## 4.10. 创建随机访问迭代器（未做）

# 第 5 章 Lambda 表达式

lambda  匿名函数

```C++
auto la = []{ return "Hello World"; };
auto lb = [](auto a) { return a(); };
cout << lb(la());

// 闭包   捕获外部变量
const char * greeting{ "Hello\n" };
const auto la = [greeting]{ return greeting; };
cout << la();
```

## 5.3. 用于作用域可重用代码

```C++
//  lambdas.cpp
//  as of 2021-12-18 bw [bw.org]

#include <format>
#include <iostream>
#include <string>

using std::format;
using std::cout;
using std::string;

int main() {
    auto one = []() -> const char * { return "one"; };
    auto two = []{ return "two"; };

    cout << "one and two:\n";
    cout << one() << '\n';
    cout << format("{}\n", two());
	
    // auto p = []<template T>(T v) { cout << v() << '\n'; };
    
    auto p = [](auto v){ cout << v() << '\n'; };
    p([]{ return "lambda call lambda"; });

    cout << "anonymous lambda: ";
    cout << [](auto l, auto r){ return l + r; }(47, 73) << '\n';
    
    cout << "capture by reference: ";
    int num{0};
    auto inc = [&num]{ num++; };
    for (size_t i{0}; i < 5; ++i) {
        inc();
    }
    cout << num << '\n';

    cout << "counter: ";
    // 可变说明符允许 lambda 修改它的捕获，lambda 默认限定为 const。
    auto counter = [n = 0]() mutable { return ++n; };   // 定义一个本地捕获变量来维护其状态  
    for (size_t i{0}; i < 5; ++i) {
        cout << format("{}, ", counter());
    }
    cout << '\n';

    int a{ 47 };
    int b{ 73 };
    // 一种默认捕获类型用等号表示,将捕获 lambda 作用域中的所有符号，等号通过复制执行捕获。将捕获对象的副本，就像使用赋值操作符复制对象一样。
    // 另一个默认捕获使用 & 号进行引用捕获
    auto l1 = [=]{ return a + b; };
    cout << format("default capture: {}\n", l1());
}

```

```
// Syntax of the lambda expression
[capture-list] (parameters)
mutable (optional)
constexpr (optional)
exception attr (optional)
-> return type (optional)
{ body }

```

**在类内使用 lambda 时，不能直接捕获对象成员，可以捕获 this 或 *this 来解除对类成员的引 用。**

可以使用 constexpr 显式指定您希望将 lambda 视为常量表达式，其可以在编译时计算。弱 lambda 满足要求，即使没有说明符，也可以将其视为 constexpr。

异常属性 (可选)  :  可以使用 noexcept 说明符表明 lambda 表达式不抛出任何异常。

## 5.4. 算法库中作为谓词

算法库中的某些函数需要使用谓词函数。谓词是测试条件并返回布尔 true/false 响应的函数 (或 函子或 lambda)。

```C++
// 函数
bool is_div4(int i) {
	return i % 4 == 0;
}
int count = count_if(v.begin(), v.end(), is_div4);

// 函子
// 函子的优点是可以携带上下文并访问类和实例变量。
struct is_div4 {
	bool operator()(int i) {
		return i % 4 == 0;
	}
};
int count = count_if(v.begin(), v.end(), is_div4());

auto is_div4 = [](int i){ return i % 4 == 0; };
int count = count_if(v.begin(), v.end(), is_div4);
```

```C++
//  algorithm.cpp
//  as of 2021-12-17 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>

using std::format;
using std::cout;
using std::string;
using std::vector;
using std::count_if;

auto is_div_by(int divisor) {
    return [divisor](int i) { return i % divisor == 0; };
}

int main() {
    const vector<int> v{ 1, 7, 4, 9, 4, 8, 12, 10, 20 };

    for (int i : { 3, 4, 5 }) {
        auto pred = is_div_by(i);
        int count = count_if(v.begin(), v.end(), pred);
        cout << format("numbers divisible by {}: {}\n", i, count);
    }
}
```

<font color='red'>理解函数指针和指针函数的区别</font>

```
函数指针（指向函数的指针）：返回类型 (*指针变量名)(参数列表);
指针函数：
返回类型* 函数名(参数列表) {
    // 函数体
    return 指针;
}
```

## 5.5. 与 std::function 一起作为多态包装器

类模板 std::function 是函数的精简多态包装器，可以存储、复制和调用函数、lambda 表达式或 其他函数对象，在想要存储对函数或 lambda 的引用的地方很有用。使用 std::function 允许在同一个 容器中存储具有不同签名的函数和 lambda，并且其可以维护 lambda 捕获的上下文。

### functional (C++ library headers)

提供了标准库中的其他算术函数对象，例如：

```C++
#include <functional>

// std::plus的类似实现
struct PlusInt {
    int operator()(int a, int b) const {
        return a + b;
    }
};

PlusInt{};  // 创建临时对象

std::plus<int>{}        // 加法：a + b
std::minus<int>{}       // 减法：a - b  
std::multiplies<int>{}  // 乘法：a * b
std::divides<int>{}     // 除法：a / b
std::modulus<int>{}     // 取模：a % b
std::negate<int>{}      // 取负：-a
```

使用 std::function 类来存储 vector 中 lambda 的不同特化

```C++
//  function.cpp
//  as of 2021-12-08 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <deque>
#include <list>
#include <vector>
#include <functional>

using std::format;
using std::cout;
using std::string;
using std::deque;
using std::list;
using std::vector;
using std::function;

struct hello {
    void greeting() const { cout << "Hello Bob\n"; }
    void operator()() { greeting(); }
};

int main() {
    deque<int> d;
    list<int> l;
    vector<int> v;

    auto print_c = [](auto& c) {
        for(auto i : c) cout << format("{} ", i);
        cout << '\n';
    };

    // push_c 接受对容器的引用，该容器由匿名 lambda 捕获。匿名 lambda 调用捕获容器上的push_back() 成员。push_c 的返回值是匿名 lambda。
    auto push_c = [](auto& container) {
        return [&container](auto value) {
            container.push_back(value);
        };
    };

    // 现在声明一个 std::function 元素的 vector，并用 push_c() 的三个实例对其进行填充
    // 初始化器列表中的每个元素都是对 push_c 的函数调用。push_c 返回匿名 lambda 的实例，该实例通过函数包装器存储在 vector 中。push_c 是用 d、l 和 v 这三个容器调用的，容器用匿名lambda 作为捕获进行传递。
    // push_c 返回一个匿名 lambda, 这个匿名表达式存储在 vector 中
    // 这是 consumers 容器的定义，由三个元素初始化，其中每个元素都通过调用 push_c 进行初始化，该调用返回一个匿名 lambda。存储在 vector 中的是匿名表达式，而不是 push_c。
    const vector<std::function<void(int)>> 
        consumers { push_c(d), push_c(l), push_c(v) };
    
    for(auto &c : consumers) {
        for (int i{0}; i < 10; ++i) {
            c(i);
        }
    }

    print_c(d);
    print_c(l);
    print_c(v);

    hello bob{};
    const function<void(const hello&)> h1 = &hello::greeting;
    const function<void(void)> h2 = std::bind(&hello::greeting, &bob);
    const function<void(void)> h3 = hello();    // functor
    h1(bob);
    h2();
    h3();
}
```

function 类的性质使它在很多方面都很有用，可以将其视为一个多态函数容器。可以存存储为一 个独立的函数，可以存储一个成员函数，使用 std::bind 来绑定函数形参，可以存储可执行对象

## 5.6. 用递归连接 lambda（看不太懂）

```C++
//  concatenation.cpp
//  as of 2021-12-13 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <functional>

using std::format;
using std::cout;
using std::string;

template <typename T, typename ...Ts>
auto concat(T t, Ts ...ts) {
    if constexpr (sizeof...(ts) > 0) {
        return [&](auto ...parameters) { return t(concat(ts...)(parameters...)); };
    } else  {
        return t;
    }
}

int main() {
    auto twice = [](auto i) { return i * 2; };
    auto thrice = [](auto i) { return i * 3; };

    auto combined = concat(thrice, twice, std::plus<int>{});

    std::cout << format("{}\n", combined(2, 3));
}

```

**展开过程：**

1. **第一次递归调用**：`concat(thrice, twice, std::plus<int>{})`
   - `sizeof...(ts) = 2` (twice, std::plus<int>{}) > 0
   - 返回：`[&](auto ...parameters) { return thrice(concat(twice, std::plus<int>{})(parameters...)); }`
2. **第二次递归调用**：`concat(twice, std::plus<int>{})`
   - `sizeof...(ts) = 1` (std::plus<int>{}) > 0
   - 返回：`[&](auto ...parameters) { return twice(concat(std::plus<int>{})(parameters...)); }`
3. **第三次递归调用**：`concat(std::plus<int>{})`
   - `sizeof...(ts) = 0`
   - 返回：`std::plus<int>{}` (基础情况)

```C++
// combined 的实际结构：
auto combined = [&](auto p1, auto p2) {
    return thrice(
        [&](auto p1, auto p2) {
            return twice(
                std::plus<int>{}(p1, p2)
            );
        }(p1, p2)
    );
};

// concat(thrice, twice, std::plus<int>{}) 展开后：
auto combined = [](int a, int b) {
    return thrice(
        [](int a, int b) {
            return twice(
                [](int a, int b) {
                    return std::plus<int>{}(a, b);  // a + b
                }(a, b)
            );
        }(a, b)
    );
};

// 执行 combined(2, 3):
// 1. std::plus<int>{}(2, 3) = 5
// 2. twice(5) = 10  
// 3. thrice(10) = 30
```

## 5.7. 将谓词与逻辑连接词结合起来

```C++
//  conjunction.cpp
//  as of 2021-12-20 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <functional>
#include <algorithm>

using std::format;
using std::cout;
using std::string;
using std::cin;
using std::istream_iterator;
using std::ostream_iterator;


template<typename F,typename A,typename B>
auto combine(F binary_func, A a, B b) {
	return [=](auto param) {
		return binary_func(a(param), b(param));
		};
}

int main() {
	auto begins_with = [](const string& s) {
		return s.find("a") == 0;
		};
	auto ends_with = [](const string& s) {
		return s.rfind("b") == s.length() - 1;
		};
	auto bool_and = [](const auto& l, const auto& r) {
		return l && r;
		};

	std::copy_if(
		istream_iterator<string>{cin}, {},
		ostream_iterator<string>{cout, " \n"},  // 这里可以多加一个换行符，下一次输入就可以在另一行
		combine(bool_and, begins_with, ends_with)
	);

	std::cout << '\n';
}
```

```C++
// first - 输入范围的起始迭代器
// last - 输入范围的结束迭代器
// d_first - 输出目标的起始迭代器
template< class InputIt, class OutputIt, class UnaryPredicate >
OutputIt copy_if( InputIt first, InputIt last,
                  OutputIt d_first, UnaryPredicate pred );
```

begin_with 和 ends_with 是简单的过滤器谓词，用于分别查找以’a’ 开头和以’b’ 结尾的字符串。 bool_and 可对两个参数进行与操作。	

std::copy_if() 算法需要一个带有一个形参的谓词函数，但连接需要两个形参，每个形参都需要 一个参数。我们用一个函数（combine）来解决这个问题，该函数返回一个专门用于此上下文的 lambda:

combine() 函数从三个形参创建一个 lambda，每个形参都是一个函数。返回的 lambda 接受谓词 函数所需的一个参数。现在可以用 combine() 函数调用 copy_if():

## 5.8. 用相同的输入调用多个 lambda

```C++
//  braces.cpp
//  as of 2021-12-16 bw [bw.org]

#include <format>
#include <iostream>
#include <string>

using std::format;
using std::cout;

auto braces (const char a, const char b) {
    return [a, b](const auto v) {
        cout << format("{}{}{} ", a, v, b);
    };
}

int main() {
    auto a = braces('(', ')');
    auto b = braces('[', ']');
    auto c = braces('{', '}');
    auto d = braces('|', '|');

    for( int i : { 1, 2, 3, 4, 5 } ) {
        for( auto x : { a, b, c, d } ) x(i);
        cout << '\n';
    }
}

```

braces() 函数包装了一个 lambda，该 lambda 返回一个三值字符串，其中第一个和最后一个值 是作为捕获传递给 lambda 的字符，中间的值作为参数传递。

通过将括号 () 函数参数传递给 lambda，可以返回具有该上下文的 lambda。

## 5.9. 对跳转表使用映射 lambda

### cctype ( C++ headers for C library facilities )

`cctype` 是C++标准库中的头文件，用于处理字符分类和转换。它是C语言 `ctype.h` 的C++版本。

```C++
//  jump.cpp
//  as of 2021-12-17 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <map>
#include <cctype>

using std::format;
using std::cout;
using std::cin;
using std::getline;
using std::string;
using std::map;

const char prompt(const char * p) {
    std::string r;
    cout << format("{} > ", p);
    getline(cin, r, '\n');

    if(r.size() < 1) return '\0';
    if(r.size() > 1) {
        cout << "Response too long\n";
        return '\0';
    }
    return toupper(r[0]);
}

int main() {
    using jumpfunc = void(*)();
    // alternately, std::function works too:
    // using jumpfunc = std::function<void()>;

    map<const char, jumpfunc> jumpmap {
        { 'A', []{ cout << "func A\n"; } },
        { 'B', []{ cout << "func B\n"; } },
        { 'C', []{ cout << "func C\n"; } },
        { 'D', []{ cout << "func D\n"; } },
        { 'X', []{ cout << "Bye!\n"; } }
    };
    
    char select{};
    while(select != 'X') {
        if((select = prompt("select A/B/C/D/X"))) {
            auto it = jumpmap.find(select);
            if(it != jumpmap.end()) it->second();
            else cout << "Invalid response\n";
        }
    }
}
```

# 第 6 章 STL 算法

### numeric ( **C++ library headers** ) （未加案例）

数值算法

### memory ( C++ library headers )  （未加案例）

与内存相关的算法

## 6.2. 基于迭代器的复制

```C++
//  copy.cpp
//  as of 2021-12-20 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <string_view>
#include <vector>
#include <iterator>
#include <algorithm>

using std::format;
using std::cout;
using std::string;
using std::string_view;
using std::vector;
using std::ostream_iterator;

namespace ranges = std::ranges;

void printc(auto& c, string_view s = "") {
    if(s.size()) cout << format("{}: ", s);
    for(auto e : c) cout << format("[{}] ", e);
    cout << '\n';
}

int main() {
    vector<string> v1 { "alpha", "beta", "gamma", "delta", "epsilon" };
    printc(v1, "v1");

    vector<string> v2{};
    ranges::copy(v1, back_inserter(v2));
    printc(v2, "v2");

    vector<string> v3{};
    std::copy_n(v1.begin(), 3, back_inserter(v3));
    printc(v3, "v3");

    vector<string> v4{};
    ranges::copy_if(v1, back_inserter(v4), [](string& s){ return s.size() > 4; });
    printc(v4, "v4");

    ostream_iterator<string> out_it(cout, " ");
    ranges::copy(v1, out_it);
    cout << '\n';

    std::move(v1.begin(), v1.end(), v2.begin());
    printc(v1, "after move1: v1");
    printc(v2, "after move1: v2");

    ranges::move(v2, v1.begin());
    printc(v1, "after move2: v1");
    printc(v2, "after move2: v2");
}

```

## 6.3. 将容器元素连接到一个字符串中（自定义添加空格符）

### sstream ( **C++ library headers** )

`<sstream>` 是C++标准库中的头文件，提供了基于字符串的流处理功能，允许像操作输入输出流一样操作字符串。

### numbers ( C++ 20 )

该头文件保留C++中的科学计数值

```C++
#include <format>
#include <iostream>
#include <string>
#include <string_view>
#include <vector>
#include <list>
#include <sstream>
#include <ranges>
#include <numbers>

using std::format;
using std::cout;
using std::ostream;
using std::string;
using std::string_view;
using std::ostringstream;
using std::vector;
using std::list;

namespace ranges = std::ranges;
namespace views = std::ranges::views;

namespace bw {

	//使用 ostream 对象用分隔符连接元素:
	template<typename I>
	ostream& join(I it, I end_it, ostream& o, string_view sep = "") {
		if (it != end_it) o << *it++;
		while (it != end_it) o << sep << *it++;
		return o;    // 方便起见，返回 ostream 对象。这允许用户可以向流中添加换行符或其他对象:
	}

	template<typename I>
	string join(I it, I end_it, string_view sep = "") {
		ostringstream ostr;
		join(it, end_it, ostr, sep);
		return ostr.str();
	}

	string join(const auto& c, string_view sep = "") {
		return join(begin(c), end(c), sep);
	}
}

int main() {
	vector<string> greek{ "alpha", "beta", "gamma", "delta", "epsilon" };

	cout << "views::join: ";
	auto greek_view = views::join(greek);
	for (const char c : greek_view) cout << c;//按字符打印
	cout << '\n';

	cout << "bw::join (cout): ";
	bw::join(greek.begin(), greek.end(), cout, ", ") << '\n';

	cout << "bw::join (string): ";
	string s = bw::join(greek, ", ");
	cout << s << '\n';

	namespace num = std::numbers;
	cout << "bw::join (constants): ";
	list<double> constants{ num::pi, num::e, num::sqrt2 };
	cout << bw::join(constants, ", ") << '\n';

	cout << "bw::join (greek_view): ";
	cout << bw::join(greek_view, ":") << '\n';
}
```

## 6.4. std::sort——排序容器元素

```C++
//  sort.cpp
//  as of 2021-12-24 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <iterator>
#include <random>

using std::format;
using std::cout;
using std::string;
using std::vector;

// std::random_device 类使用系统的硬件熵源。大多数现代系统都有一个，否则库将对其进行模拟。
// std::default_random_engine() 函数从熵源生成随机数，std::shuffle() 用其来随机化容器。
void randomize(auto& c) {
	static std::random_device rd;
	static std::default_random_engine rng(rd());
	std::shuffle(c.begin(), c.end(), rng);
}

void check_sorted(auto& c) {
	if (!is_sorted(c.begin(), c.end())) cout << "un";
	cout << "sorted: ";
}

void printc(const auto& c) {
	check_sorted(c);
	for (auto& e : c) cout << e << ' ';
	cout << '\n';
}

void print_things(const auto& c) {
	for (auto& v : c) cout << v.str() << ' ';
	cout << '\n';
}

struct things{
	string s_;
	int i_;

	string str() const {
		return format("{}, {}", s_, i_);
	}
};

int main() {
	vector<int> v{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
	printc(v);

	vector<things> vthings{ {"button", 40}, {"hamburger", 20},
		{"blog", 1000}, {"page", 100}, {"science", 60} };

	for (int i{ 3 }; i; --i) {
		randomize(v);
		printc(v);
	}

	std::sort(v.begin(), v.end());
	printc(v);
	cout << "partial_sort:\n";
	randomize(v);
	auto middle{ v.begin() + (v.size() / 2) };
	std::partial_sort(v.begin(), middle, v.end());
	printc(v);

	cout << "partition:\n";
	randomize(v);
	printc(v);
	partition(v.begin(), v.end(), [](int i) {
		return i > 5;
		});
	printc(v);

	cout << "things:\n";
	std::sort(vthings.begin(), vthings.end(), [](const things& lhs, const things& rhs) {
		return lhs.i_ < rhs.i_;
		});
	print_things(vthings);

	std::sort(vthings.begin(), vthings.end(),
		[](const things& lhs, const things& rhs) {
			return lhs.s_ < rhs.s_;
		});
	print_things(vthings);
}
```

## 6.5. std::transform——修改容器内容

std::transform() 函数非常强大和灵活，是库中最常用的算法之一。**它将函数或 lambda 应用于容 器中的每个元素，将结果存储在另一个容器中，同时保留原始的元素。**

std::transform() 函数的工作原理与 std::copy() 相似，只不过增加了用户提供的函数。输入范围内 的每个元素都传递给函数，函数的返回值会复制赋值给目标迭代器。**值得注意的是，transform() 并不保证元素将按顺序处理。**

```C++
//  transform.cpp
//  as of 2021-12-30 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <string_view>
#include <vector>
#include <cctype>
#include <algorithm>
#include <iterator>
#include <ranges>

using std::format;
using std::cout;
using std::string;
using std::string_view;
using std::vector;
using std::transform;
using std::back_inserter;

namespace ranges = std::ranges;
namespace views = ranges::views;

void printc(auto& c, string_view s = "") {
    if(s.size()) cout << format("{}: ", s);
    for(auto e : c) cout << format("{} ", e);
    cout << '\n';
}

string str_lower(const string& s) {
    string outstr{};
    for(const char& c : s) {
        outstr += static_cast<char>(std::tolower(c));
    }
    return outstr;
}

int main() {
    vector<int> v1{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    vector<int> v2;
    printc(v1, "v1");

    cout << "squares:\n";
    transform(v1.begin(), v1.end(), back_inserter(v2), [](int x){ return x * x; });
    printc(v2, "v2");

    vector<string> vstr1{ "Mercury", "Venus", "Earth", "Mars",
        "Jupiter", "Saturn", "Uranus", "Neptune", "Pluto" };
    vector<string> vstr2;

    printc(vstr1, "vstr1");
    cout << "str_lower:\n";
    transform(vstr1.begin(), vstr1.end(), back_inserter(vstr2), [](string& x){ return str_lower(x); });
    printc(vstr2, "vstr2");

    cout << "ranges squares:\n";
    auto view1 = views::transform(v1, [](int x){ return x * x; });
    printc(view1, "view1");

    v2.clear();
    for(auto e : v1) v2.push_back(e * e);
    printc(v2, "v2");
}

```

## 6.6. 查找特定项（find/find_if）

find() 算法适用于满足前向或输入迭代器条件的容器

find() 算法接受三个参数:begin() 和 end() 迭代器，以及要搜索的值。算法会返回其找到的第一 个元素的迭代器，若搜索未能找到匹配，则返回 end() 迭代器。

也可以寻找比标量更复杂的对象，相应的对象需要支持**相等比较运算符**。下面是一个简单的 结构体，带有 operator==() 的重载

find() 和 find_if() 函数只返回一个迭代器。范围库提供了 ranges::views::filter()，这是一个视图 适配器，可为我们提供所有匹配的迭代器位置，而不会干扰到 vector:

```C++
//  find.cpp
//  as of 2021-12-27 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <iterator>
#include <ranges>

using std::format;
using std::cout;
using std::string;
using std::string_view;
using std::vector;
using std::back_inserter;
using std::find;
using std::find_if;

namespace ranges = std::ranges;

struct City {
    string name{};
    unsigned pop{};
    bool operator==(const City& o) const {
        return name == o.name;
    }
    string str() const {
        return format("[{}, {}]", name, pop);
    }
};

int main() {
    const vector<int> v{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

    cout << "find 7:\n";
    auto it1 = find(v.begin(), v.end(), 7);
    if(it1 != v.end()) cout << format("found: {}\n", *it1);
    else cout << "not found\n";

    const vector<City> c{
        { "London", 9425622 },
        { "Berlin", 3566791 },
        { "Tokyo",  37435191 },
        { "Cairo",  20485965 }
    };

    cout << "find Berlin:\n";
    auto it2 = find(c.begin(), c.end(), City("Berlin"));
    if(it2 != c.end()) cout << format("found: {}\n", it2->str());
    else cout << "not found\n";

    cout << "find pop > 20000000:\n";
    auto it3 = find_if(begin(c), end(c),
        [](const City& item) { return item.pop > 20000000; });
    if(it3 != c.end()) cout << format("found: {}\n", it3->str());
    else cout << "not found\n";

    cout << "views::filter pop > 20000000:\n";
    auto vw1 = ranges::views::filter(c, [](const City& c){ return c.pop > 20000000; });
    for(const City& e : vw1) cout << format("{}\n", e.str());
}
```

## 6.7. 将容器的元素值限制在 std::clamp 的范围内

C++17 中添加了 std::clamp() 函数，可将标量数值的范围限制在最小值和最大值之间。该函数 经过优化，可以使用移动语义，以获得最大的速度和效率。

```C++
//  clamp.cpp
//  as of 2021-12-27 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <list>
#include <algorithm>
#include <iterator>

using std::format;
using std::cout;
using std::string;
using std::string_view;
using std::vector;
using std::list;
using std::transform;
using std::clamp;

void printc(auto& c, string_view s = "") {
    if(s.size()) cout << format("{}: ", s);
    for(auto e : c) cout << format("{:>5} ", e);
    cout << '\n';
}

int main() {
    const auto il = { 0, -12, 2001, 4, 5, -14, 100, 200, 30000 };
    constexpr int ilow{0};
    constexpr int ihigh{500};

    vector<int> voi{ il };
    cout << "vector voi before:\n";
    printc(voi);

    cout << "vector voi after:\n";
    for(auto& e : voi) e = clamp(e, ilow, ihigh);
    printc(voi);

    list<int> loi{ il };
    cout << "list loi before:\n";
    printc(loi);

    transform(loi.begin(), loi.end(), loi.begin(), [=](auto e){ return clamp(e, ilow, ihigh); });
    cout << "list loi after:\n";
    printc(loi);
}

```

## 6.8. std::sample——采集样本数据集（未做）

## 6.9. 生成有序数据序列（未做）

对有序数据序列进行排列组合   next_permutation()

## 6.10. 合并已排序容器（未做）

# 第 7 章 字符串、流和格式化

```C++
const std::basic_string<char> s{"hello"};
const char * sdata = s.data();
for(size_t i{0}; i < s.size(); ++i) {
	cout << sdata[i] << ' ';
}
cout << '\n';
```

```C++
// std::string 是 std::basic_string<char> 类型的别名:
using std::string = std::basic_string<char>;
```



## 7.3. 轻量级字符串对象——string_view

string_view 构造函数创建底层数据的视图，但不保留自己的副本。这会更高效，但也会有副 作用。  因为 string_view 不复制底层数据，源数据必须在 string_view 对象存在期间保持在作用域内。

还有一个 stringview 文字操作符 sv，定义在 std::literals 命名空间中:

literal 在很多头文件都有定义，包括 <string>,<string_view>,<complex>等头文件

```C++
std::literals::string_literals::operator""s
std::literals::string_view_literals::operator""sv
```

```
using namespace std::literals;
cout << "hello"sv.substr(1, 4) << '\n';
```

这构造了一个 constexpr 的 string_view 对象，并调用 substr() 来获得从索引 1 开始的 4 个值。 输出为: ello

<font color='red'>How it works</font>

string_view 类实际上是一个连续字符序列上的迭代器适配器。实现通常有两个成员: 一个 const CharT * 和一个 size_t。工作原理是在源数据周围包装一个 contiguous_iterator。

可以像 std::string 那样使用，但有一些区别: 

• 复制构造函数不复制数据，当复制 string_view 时，复制操作都会对相同的底层数据进行:

```C++
//  string_view.cpp
//  as of 2022-01-15 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <string_view>

using std::format;
using std::cout;
using std::string;
using std::string_view;

int main() {
    char text[]{ "hello" };
    string_view sv1{ text };
    string_view sv2{ sv1 };
    string_view sv3{ sv2 };
    string_view sv4{ sv3 };

    string str1{ text };
    string str2{ str1 };
    string str3{ str2 };
    string str4{ str3 };

    text[0] = 'J';

    cout << format("sv1: {} {}\n", (void*)sv1.data(), sv1);
    cout << format("sv2: {} {}\n", (void*)sv2.data(), sv2);
    cout << format("sv3: {} {}\n", (void*)sv3.data(), sv3);
    cout << format("sv4: {} {}\n", (void*)sv4.data(), sv4);

    cout << format("str1: {} {}\n", (void*)str1.data(), str1);
    cout << format("str2: {} {}\n", (void*)str2.data(), str2);
    cout << format("str3: {} {}\n", (void*)str3.data(), str3);
    cout << format("str4: {} {}\n", (void*)str4.data(), str4);
}
```

• 将 string_view 传递给函数时，会使用复制构造函数:

```C++
#include <iostream>
#include <string_view>
using namespace std;

// string_view对象可以通过指针可以直接修改底层数据
void f(string_view sv) {
    if (sv.size()) {
        char* x = (char*)sv.data(); // dangerous
        x[0] = 'J'; // modifies the source
    }
    cout << format("f(sv): {} {}\n", (void*)sv.data(), sv);
}
int main() {
    char text[]{ "hello" };
    string_view sv1{ text };
    cout << format("sv1: {} {}\n", (void*)sv1.data(), sv1);
    f(sv1);
    cout << format("sv1: {} {}\n", (void*)sv1.data(), sv1);
}
```

因为复制构造函数不会对底层数据进行复制，底层数据的地址 (由 data() 成员函数返回) 对于 string_view 的所有实例都相同。尽管 string_view 成员指针是 const 限定的，但可以取消 const 限定符。但不建议这样做，因为可能会导致意想不到的副作用。但需要注意的是，数据永远 不会进行复制。

• string_view 类缺少直接操作底层字符串的方法。在 string 中支持的 append()、operator+()、 push_back()、pop_back()、replace() 和 resize() 等方法在 string_view 中不支持。 若需要用加法操作符连接字符串，需要使用 std::string。例如，这对 string_view 不起作用:

## 7.4. 连接字符串

string 对象有两种不同的方法用于连接字符串: 加法操作符和 append() 成员函数。 

append() 成员函数的作用是: 将数据添加到字符串对象的数据末尾，必须分配和管理内存来完 成这个任务。 加法操作符使用 operator+() 重载构造一个包含新旧数据的新字符串对象，并返回新对象。 

ostringstream 对象的工作方式类似于 ostream，但将其输出存储为字符串使用。 

```C++
ostringstream x{};  // sstream头文件中
x << a << ", " << b << "\n";
cout << x.str();
```

C++20 的 format() 函数使用带有可变参数的格式字符串，并返回一个新构造的字符串对象。

基准测试

```C++
//  concatenate.cpp
//  as of 2022-01-16 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <sstream>
#include <algorithm>
#include <chrono>

using std::format;
using std::cout;
using std::string;
using std::ostringstream;
using std::chrono::high_resolution_clock;
using std::chrono::duration;

void timer(string(*f)()) {
    auto t1 = high_resolution_clock::now();
    string s{ f() };
    auto t2 = high_resolution_clock::now();
    duration<double, std::milli> ms = t2 - t1;
    cout << s;
    cout << format("duration: {} ms\n", ms.count());
}

string append_string() {
    cout << "append_string\n";
    string a{ "a" };
    string b{ "b" };
    long n{0};
    while(++n) {
        string x{};
        x.append(a);
        x.append(", ");
        x.append(b);
        x.append("\n");
        if(n >= 10000000) return x;
    }
    return "error\n";
}

string concat_string() {
    cout << "concat_string\n";
    string a{ "a" };
    string b{ "b" };
    long n{0};
    while(++n) {
        string x{};
        x += a + ", " + b + "\n";
        if(n >= 10000000) return x;
    }
    return "error\n";
}

string concat_ostringstream() {
    cout << "ostringstream\n";
    string a { "a" };
    string b { "b" };
    long n{0};
    while(++n) {
        ostringstream x{};
        x << a << ", " << b << "\n";
        if(n >= 10000000) return x.str();
    }
    return "error\n";
}

string concat_format() {
    cout << "append_format\n";
    string a{ "a" };
    string b{ "b" };
    long n{0};
    while(++n) {
        string x{};
        x = format("{}, {}\n", a, b);
        if(n >= 10000000) return x;
    }
    return "error\n";
}

int main() {
    timer(append_string);
    timer(concat_string);
    timer(concat_ostringstream);
    timer(concat_format);
}

```

<img src="C:\Users\86178\AppData\Roaming\Typora\typora-user-images\image-20251020094906154.png" alt="image-20251020094906154" style="zoom: 80%;" />

## 7.5. 转换字符串

string 类是一个连续容器，很像 vector 或数组，并且支持 contiguous_iterator 和所有算法。 

string 类是具有 char 类型的 basic_string 的特化，所以容器的元素是 char 类型的。其他特化也可 用，但字符串是最常见的。 

其本质上是一个连续的 char 元素容器，所以 string 可以与 transform() 算法或其他使用 contiguous_iterator 的技术一起使用。

### cctype ( C++ headers for C library facilities )

```C++
//  transform.cpp
//  as of 2022-01-17 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <algorithm>
#include <cctype>

using std::format;
using std::cout;
using std::string;
using std::string_view;

char char_upper(const char& c) {
    return static_cast<char>(std::toupper(c));
}

char char_lower(const char& c) {
    return static_cast<char>(std::tolower(c));
}

char rot13(const char& x) {
    auto rot13a = [](char x, char a)->char {
        return a + (x - a + 13) % 26;
        };
    if (x >= 'A' && x <= 'Z') return rot13a(x, 'A');
    if (x >= 'a' && x <= 'z') return rot13a(x, 'a');
    return x;
}

// 每个单词的首字母大写
string& title_case(string& s) {
    auto begin = s.begin();
    auto end = s.end();
    *begin++ = char_upper(*begin);  // first element, 从左往右计算, 先把第一个单词的第一个字母变成大写,并自增+1
    bool space_flag{ false };
    for (auto it{ begin }; it != end; ++it) { // 循环从第一个单词的第二个字母开始遍历执行
        if (*it == ' ') {
            space_flag = true;
        }
        else {
            if (space_flag) *it = char_upper(*it);
            space_flag = false;
        }
    }
    return s;
}

int main() {
    string s{ "hello jimi\n" };
    cout << s;

    std::transform(s.begin(), s.end(), s.begin(), char_upper);
    cout << s;

    for (auto& c : s) c = rot13(c);
    cout << s;

    for (auto& c : s) c = rot13(char_lower(c));  // 这里改为char_lower(rot13(c))执行结果也是一样的
    cout << s;

    cout << title_case(s);
}
```

## 7.6. 使用格式库格式化文本

```C++
//  format.cpp
//  as of 2022-04-27 bw [bw.org]
```

## 7.7. 删除字符串中的空白（头尾端的空格）

### string ( **C++ library headers** )

用户的输入通常在字符串的一端或两端包含无关的空格。字符串类方法移除空格

find_first_not_of()

find_last_not_of()

```C++
//  trim.cpp
//  as of 2022-01-20 bw [bw.org]

#include <format>
#include <iostream>
#include <string>

using std::format;
using std::cout;
using std::string;

string trimstr(const string& s) {
    constexpr const char * whitespace{ " \t\r\n\v\f" };

    if(s.empty()) return s;     // empty string
    const auto first{ s.find_first_not_of(whitespace) };  // string 类的各种 find…() 成员函数返回一个位置作为 size_t 值:
    if(first == string::npos) return {};  // all whitespace
    const auto last{ s.find_last_not_of(whitespace) };
    return s.substr(first, (last - first + 1));   // s.substr() 方法，通过确认第一个和最后一个字符的位置，来返回不带空格的字符串。
}

int main() {
    string s{" \t  ten-thumbed input   \t   \n \t "};
    cout << format("[{}]\n", s);

    cout << format("[{}]\n", trimstr(s));
}

```

**substr() 参数计算**：

- **`first`**：第一个非空白字符的位置
- **`last`**：最后一个非空白字符的位置
- **`last - first + 1`**：要提取的子字符串长度

## 7.8. 从用户输入中读取字符串

### cstdio ( **C++ headers for C library facilities** )



STL 使用 std::cin 对象从标准输入流提供基于字符的输入。**cin 对象是一个全局单例对象**，可从控制台读取输入作为 istream 输入流。

```C++
//  input.cpp
//  as of 2022-01-22 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <string_view>
#include <sstream>
#include <cstdio>

using std::format;
using std::cout;
using std::cin;   // cin 对象是一个全局单例对象
using std::string;
using std::string_view;
using std::stringstream;

constexpr size_t MAXLINE{ 1024 * 10 };

string trimstr(const string& s) {
	constexpr const char* whitespace{ " \t\r\n\v\f" };

	if (s.empty()) return s;
	const auto first{ s.find_first_not_of(whitespace) };
	if (first == string::npos) return {};
	const auto last{ s.find_last_not_of(whitespace) };
	return s.substr(first, (last - first + 1));
}

// 这段代码的作用是清理输入流中的错误状态和残留内容。
void clearistream() {
	string s{};

	//clear()函数作用：清除输入流的错误状态标志
	//解决的问题：当输入类型不匹配时（如期望数字但输入字母），流会进入错误状态，后续所有输入操作都会失败
	//恢复状态：将 cin 的状态重置为 goodbit，使其恢复正常工作
	cin.clear();
	getline(cin, s);
}

bool prompt(const string_view s, const string_view s2 = "") {
	if (s2.size()) cout << format("{} ({}): ", s, s2);
	else cout << format("{}: ", s);
	cout.flush();  // 强制刷新输出缓冲区
	return true;   // cout.flush() 确保输出立即显示，适用于需要实时反馈的场景。
}

int main() {
	double a{};
	double b{};
	char s[MAXLINE]{};
	string word{};
	string line{};

	const char* p1{ "Words here" };
	const char* p1a{ "More words here" };
	const char* p2{ "Please enter two numbers" };
	const char* p3{ "Comma-separated words" };

	/*prompt(p1);
	cin.getline(s, MAXLINE, '\n');
	cout << s << '\n';*/

	//prompt(p1a, "p1a");
	// 第一个参数是输出流，第二个参数是对字符串对象的引用，第三个参数是行结束分隔符。
	//getline(cin, line, '\n');
	//cout << line << '\n';

	/*for (prompt(p2); !(cin >> a >> b); prompt(p2)) {
		cout << "not numeric\n";
		clearistream();
	}
	cout << format("You entered {} and {}\n", a, b);*/

	line.clear();
	prompt(p3);
	while (line.empty()) getline(cin, line);
	stringstream ss(line);
	while (getline(ss, word, ',')) {
		if (word.empty()) continue;
		cout << format("word: [{}]\n", trimstr(word)); 
	}
}
```

## 7.9. 统计文件中的单词数

默认情况下，basic_istream 类每次读取一个单词。可以利用这个属性使用 istream_iterator 来计 算单词数。

```C++
//  count-words.cpp
//  as of 2022-01-27 bw [bw.org]

#include <format>
#include <iostream>
#include <fstream>
#include <string>

using std::format;
using std::cout;
using std::string;
using std::string_view;

size_t wordcount(auto& is) {
    using it_t = std::istream_iterator<string>;
    return std::distance(it_t{is}, it_t{});
}

int main() {
    const char * fn{ "the-raven.txt" };
    std::ifstream infile{fn, std::ios_base::in};
    size_t wc{ wordcount(infile) };
    cout << format("There are {} words in the file.\n", wc);
}

```

`std::istream_iterator<string>` 的作用

这是一个输入流迭代器，能够：

- 从输入流中读取 `string` 类型的数据
- 每次读取一个单词（以空白字符分隔）
- 到达流末尾时变为结束迭代器

在你的代码中，它用于统计文件中的单词数量。

`std::distance()` 是C++标准库中的一个算法函数，用于**计算两个迭代器之间的距离**（即元素个数）。

```C++
// 函数原型
template<class InputIt>
typename std::iterator_traits<InputIt>::difference_type
distance(InputIt first, InputIt last);

// 等价实现
size_t wordcount(auto& is) {
    using it_t = std::istream_iterator<string>;
    it_t begin{is};
    it_t end{};
    
    size_t count = 0;
    while (begin != end) {
        ++count;
        ++begin;
    }
    return count;
}
```

`std::ifstream` 是C++中用于**文件输入**的流类，继承自 `std::istream`。

**常用成员函数**：

| 函数        | 作用                 |
| :---------- | :------------------- |
| `is_open()` | 检查文件是否成功打开 |
| `good()`    | 检查流状态是否良好   |
| `eof()`     | 是否到达文件末尾     |
| `fail()`    | 是否发生错误         |
| `close()`   | 关闭文件             |
| `seekg()`   | 移动读取位置         |
| `tellg()`   | 获取当前读取位置     |

## 7.10. 使用文件输入初始化复杂结构体（看不懂）

输入流的优点是能够解析文本文件中不同类型的数据，并将它们转换为相应的基本类型。下面 是一个使用输入流将数据导入结构容器的简单技术。

### iomanip ( **C++ library headers** )

`<iomanip>` 是C++标准库中的头文件，提供了**输入/输出格式化**的功能，用于控制数据的显示格式。

控制数据在流中的显示方式，包括：

- 数字的进制和精度
- 字段宽度和对齐
- 填充字符
- 布尔值显示格式等

### fstream ( C++ library headers )

seekg() 和 seekp() 这两个函数的用法：https://c.biancheng.net/view/1541.html

```C++
#include <format>
#include <iostream>
#include <string>
#include <iomanip>
#include <vector>
#include <fstream>

using std::format;
using std::cout;
using std::cin;
using std::ifstream;
using std::string;
using std::vector;

constexpr const char* fn{ "cities.txt" };

struct City {
	string name;
	unsigned long population;
	double latitude;
	double longitude;
};

// skip BOM for UTF-8 on Windows
void skip_bom(auto& fs) {
	const unsigned char boms[]{ 0xef, 0xbb, 0xbf };
	bool have_bom{ true };
	for (const auto& c : boms) {
		if ((unsigned char)fs.get() != c) have_bom = false;
	}
	if (!have_bom) fs.seekg(0);   // get
	return;
}

string make_commas(const unsigned long num) {
	string s{ std::to_string(num) };
	for (int l = (int)s.length() - 3; l > 0; l -= 3) {
		s.insert(l, ",");
	}
	return s;
}

std::istream& operator >> (std::istream& in, City& c) {
	in >> std::ws;   // std::ws 是一个输入操纵器（manipulator）,功能是提取并丢弃流中的前导空白字符
	std::getline(in, c.name);  // 获取一整行输入
	in >> c.population >> c.latitude >> c.longitude;
	return in;
}

int main() {
	vector<City> cities;
	ifstream infile(fn, std::ios_base::in);   // ios/iostream头文件中,用于指定以只读方式打开文件
	if (!infile.is_open()) {
		cout << format("failed to open file {}\n", fn);
		return 1;
	}

	skip_bom(infile);
	for (City c{}; infile >> c;) cities.emplace_back(c);

	cout << format("{} cities imported\n", cities.size());
	for (const auto& [name, pop, lat, lon] : cities) {
		cout << format("{:.<15} pop {:<10} coords {}, {}\n", name, make_commas(pop), lat, lon);
	}
}
```

**其他相关的打开模式：**

- `std::ios_base::out` - 输出模式（写入）
- `std::ios_base::app` - 追加模式
- `std::ios_base::binary` - 二进制模式
- `std::ios_base::trunc` - 清空文件模式

## 7.11. 使用 char_traits 定义一个字符串类

```C++
//  ctraits.cpp
//  as of 2022-01-23 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <compare>

using std::format;
using std::cout;
using std::string;

constexpr char char_lower(const char& c) {
    if(c >= 'A' && c <= 'Z') return c + ('a' - 'A');
    else return c;
}

class ci_traits : public std::char_traits<char> {
public:
    static constexpr bool lt( char_type a, char_type b ) noexcept {
        return char_lower(a) < char_lower(b);
    }

    static constexpr bool eq( char_type a, char_type b ) noexcept {
        return char_lower(a) == char_lower(b);
    }

    static constexpr int compare(const char_type* s1, const char_type* s2, size_t count) {
        for(size_t i{0}; i < count; ++i) {
            auto diff{ char_lower(s1[i]) <=> char_lower(s2[i]) };
            if(diff > 0) return 1;
            if(diff < 0) return -1;
            }
        return 0;
    }

    static constexpr const char_type* find(const char_type* p, size_t count, const char_type& ch) {
        const char_type find_c{ char_lower(ch) };
        for(size_t i{0}; i < count; ++i) {
            if(find_c == char_lower(p[i])) return p + i;
        }
        return nullptr;
    }
};

class lc_traits : public std::char_traits<char> {
public:
    static constexpr void assign( char_type& r, const char_type& a ) noexcept {
        r = char_lower(a);
    }

    static constexpr char_type* assign( char_type* p, std::size_t count, char_type a ) {
        for(size_t i{}; i < count; ++i) p[i] = char_lower(a);
        return p;
    }

    static constexpr char_type* copy(char_type* dest, const char_type* src, size_t count) {
        for(size_t i{0}; i < count; ++i) dest[i] = char_lower(src[i]);
        return dest;
    }

};

using ci_string = std::basic_string<char, ci_traits>;
using lc_string = std::basic_string<char, lc_traits>;

std::ostream& operator<<(std::ostream& os, const ci_string& str) {
    return os << str.c_str();
}

std::ostream& operator<<(std::ostream& os, const lc_string& str) {
    return os << str.c_str();
}

int main() {
    string s{"Foo Bar Baz"};
    ci_string ci_s{"Foo Bar Baz"};

    cout << "string: " << s << '\n';
    cout << "ci_string: " << ci_s << '\n';

    ci_string compare1{"CoMpArE StRiNg"};
    ci_string compare2{"compare string"};

    if (compare1 == compare2) {
        cout << format("Match! {} == {}\n", compare1, compare2);
    } else {
        cout << format("no match {} != {}\n", compare1, compare2);
    }

    size_t found = ci_s.find('b');
    cout << format("found: pos {} char {}\n", found, ci_s[found]);

    lc_string lc_s{"Foo Bar Baz"};
    cout << "lc_string: " << lc_s << '\n';

    return 0;
}

```

## 7.12. 用正则表达式解析字符串（未做）

### regex ( **Headers added in C++11** )

《Mastering Regular Expressions》

# 第 8 章 实用工具类

本章在以下主题中介绍了一些通用的工具，包括时间测量、泛型类型、智能指针等:

## 8.2. std::optional 管理可选值

用于表示**可能存在也可能不存在的值**。

F:\Cprojects\STL\chapter8\optional_example.cpp

**优点总结**

1. **类型安全**：避免空指针解引用
2. **语义明确**：清楚表达"可能不存在"的意图
3. **无额外开销**：通常与手写实现性能相当
4. **标准统一**：提供一致的 API 来处理可选值
5. **易于使用**：简洁的语法和丰富的工具函数

`std::optional` 是现代 C++ 中处理可选值的推荐方式，让代码更安全、更易读。

## 8.3. std::any 保证类型安全

### typeinfo ( **C++ library headers** )

`<typeinfo>` 是 C++ 标准库头文件，主要用于**运行时类型信息（RTTI）** 的操作，提供了在程序运行时获取和比较类型信息的能力。

**总结**

`<typeinfo>` 头文件的主要作用：

1. **运行时类型识别**：获取对象的实际类型信息
2. **类型比较**：判断两个对象是否是相同类型
3. **调试支持**：在日志和调试输出中显示类型信息
4. **基于类型的决策**：根据类型执行不同的逻辑

**注意事项**：

- 需要开启 RTTI 支持（通常默认开启）
- 对多态类型返回实际类型，对非多态类型返回静态类型
- `type_info` 对象不可拷贝，只能通过引用传递
- 类型名称的格式是编译器相关的

虽然 RTTI 在某些性能敏感场景中可能被禁用，但在需要灵活类型处理的场合，`<typeinfo>` 提供了强大的运行时类型操作能力。

### any ( **C++17** )

C++17 中引入了 std::any 类，可为任何类型的单个对象提供了类型安全的容器。

```C++
//  any.cpp
//  as of 2022-01-30 bw [bw.org]

#include <format>
#include <iostream>
#include <string>
#include <vector>
#include <list>
#include <any>
#include <typeinfo>

using std::format;
using std::cout;
using std::string;
using std::list;
using std::vector;
using std::any;
using std::any_cast;

using namespace std::literals;

void p_any(const any& a) {
	if (!a.has_value()) {
		cout << "None.\n";
	}
	else if (a.type()==typeid(int)) {
		cout << format("int: {}\n", any_cast<int>(a));
	}
	else if (a.type()==typeid(string)) {
		cout << format("string: \"{}\"\n", any_cast<const string&>(a));
	}
	else if (a.type() == typeid(list<int>)) {
		cout << "list<int>: ";
		for (auto& i : any_cast<const list<int>&> (a))
			cout << format("{} ", i);
		cout << '\n';
	}
	else {
		cout << format("something else: {}\n", a.type().name());
	}
}

int main() {
	any x{};
	if (x.has_value()) cout << "have value\n";
	else cout << "no value\n";

	// type() 方法返回一个 type_info 对象，type_info::name() 方法返回 C-string 类型的实现定义的名称
    // any_cast<type>() 非成员函数强制转换值进行使用
	x = 42;
	if (x.has_value()) {
		cout << format("x has type: {}\n", x.type().name());
		cout << format("x has value: {}\n", any_cast<int>(x));
	}
	else {
		cout << "no value\n";
	}

	x = "ab"s;
	cout << format("x is type {} with value {}\n", x.type().name(), any_cast<string>(x));

	x = list{ 1, 2, 3 };
	cout << format("x is type {} with value ", x.type().name());
	for (const int& i : any_cast<list<int>&>(x)) cout << i << ' ';
	cout << '\n';

	try {
		cout << any_cast<int>(x) << '\n';
	}
	catch (std::bad_any_cast& e) {
		cout << format("any: {}\n", e.what());
	}

	p_any({});
	p_any(47);
	p_any("abc"s);
	p_any(any(list{ 1, 2, 3 }));
	p_any(any(vector{ 1, 2, 3 }));
}
```

**How it works… **

any 复制构造函数和赋值操作符，可以直接初始化来生成目标对象的非 const 副本作为包含对象，包含的对象类型可以作为 typeid 单独存储。 初始化后，any 对象有以下方法: 

• emplace() 替换包含的对象，在适当的位置构造新对象。 

• reset() 替换包含的对象，在适当的位置构造新对象。 

• has_value() 若有包含对象，则返回 true。 

• type() 返回 typeid 对象，表示所包含对象的类型。 

• operator=() 用复制或移动操作替换包含的对象。 any 类还支持以下非成员函数: 

• any_cast()，模板函数，提供对包含对象的类型安全访问。 any_cast() 函数返回包含对象的副本，可以使用 any_cast() 来返回引用。 

• std::swap() 专门处理 std::swap 算法。

## 8.4. std::variant 存储不同的类型

C++17 中引入的 std::variant 类可以保存不同的值，每次一个，其中每个值必须适合相同的分配内存空间。对于保存可供在单个上下文中使用的替代类型非常有用。

`<variant>` 是 C++17 引入的标准库头文件，它定义了 `std::variant` 类模板，用于表示**类型安全的联合类型**。

### variant ( **C++17** )

**总结**

`std::variant` 相比传统 `union` 的主要优势：

1. **类型安全** - 编译时和运行时检查防止误用
2. **支持现代类型** - 可以存储 `std::string`、`std::vector` 等
3. **自动生命周期管理** - 自动调用构造函数和析构函数
4. **丰富的API** - `visit()`, `get()`, `get_if()` 等安全访问方式
5. **异常安全** - 提供强异常保证

**使用场景**：

- 需要类型安全的联合类型
- 状态机实现
- 错误处理
- 配置系统
- 解析器返回值（成功值或错误信息）

`std::variant` 是现代 C++ 中处理"多选一"数据的推荐方式，提供了更好的安全性和易用性。

**visit() 函数工作原理：**

- `std::visit` 会检查 `variant` 当前实际存储的是哪种类型的值
- 然后根据这个实际类型，调用对应的 `visitor` 重载
- 这个过程是在**编译期**确定的，类型安全且高效

```C++
//  variant.cpp
//  as of 2022-02-01 bw [bw.org]

#include <format>
#include <iostream>
#include <variant>
#include <list>
#include <string_view>

using std::format;
using std::cout;
using std::list;
using std::string_view;

class Animal {
    string_view _name{};
    string_view _sound{};
    Animal();
public:
    Animal(string_view n, string_view s) : _name{ n }, _sound{ s } {}
    void speak() const {
        cout << format("{} says {}\n", _name, _sound);
    }
    void sound(string_view s) {
        _sound = s;
    }
};

class Cat :public Animal {
public:
    Cat(string_view n) : Animal(n, "meow") {}
};

class Dog : public Animal {
public:
    Dog(string_view n) : Animal(n, "arf!") {}
};

class Wookie : public Animal {
public:
    Wookie(string_view n) : Animal(n, "grrraarrgghh!") {}
};

using v_animal = std::variant<Cat, Dog, Wookie>;

// visit() 调用带有当前包含在变体中的对象的函子。首先，定义一个接受宠物的函子:
struct animal_speaks {
    void operator()(const Dog& d) const { d.speak(); }
    void operator()(const Cat& c) const { c.speak(); }
    void operator()(const Wookie& w) const { w.speak(); }
};

int main() {
    list<v_animal> pets{ Cat{"Hobbes"}, Dog{"Fido"}, Cat{"Max"}, Wookie{"Chewie"} };

    // 这是一个简单的函子类，每个 Animal 子类都有重载。使用 visit() 调用它，并使用列表的每个元素:
    // visit() 非成员函数使用一个可调用对象，将当前变量对象作为其唯一形参:
    // visit() 函数是在不测试对象类型的情况下，检索对象的唯一方法。结合一个可以处理每种类型的函子，就非常灵活了:
    cout << "visit:\n";
    for (const v_animal& a : pets) {
        visit(animal_speaks{}, a);
    }

    cout << "index:\n";
    for (const v_animal& a : pets) {
        auto idx{ a.index() };
        if (idx == 0) get<Cat>(a).speak();
        if (idx == 1) get<Dog>(a).speak();
        if (idx == 2) get<Wookie>(a).speak();
    }
    cout << '\n';

	// get_if<T>() 函数的作用是: 若元素类型与 T 匹配，则返回一个指针; 否则，返回 nullptr。
    cout << "get_if:\n";
    for (const v_animal& a : pets) {
        if (const auto c{ get_if<Cat>(&a) }; c)
            c->speak();
        else if (const auto d{ get_if<Dog>(&a) }; d)
            d->speak();
        else if (const auto w{ get_if<Wookie>(&a) }; w)
            w->speak();
    }
    cout << '\n';

    size_t n_cats{}, n_dogs{}, n_wookies{};
    // hold_alternative <T>() 函数返回 true 或 false。可以使用 this 来测试一个类型是否对应一个元素，而不返回该元素
    for (const v_animal& a : pets) {
        if (holds_alternative<Cat>(a)) ++n_cats;
        if (holds_alternative<Dog>(a)) ++n_dogs;
        if (holds_alternative<Wookie>(a)) ++n_wookies;
    }
    cout << format("there are {} cat(s), "
        "{} dog(s), "
        "and {} wookie(s)\n",
        n_cats, n_dogs, n_wookies);
}
```

## 8.5. std::chrono 的时间事件（未做）

### chrono ( C++ 11 )

**来源**：C++11 标准引入，包含 `<chrono>`

**核心功能**：

- 提供类型安全的时间处理
- 支持高精度计时和时间点计算
- 强大的时间间隔和单位转换能力

### ctime ( **C++ headers for C library facilities** )

**来源**：来自C标准库，在C++中包含 `<ctime>`

**核心功能**：

- 提供基本的时间获取和格式化
- 使用 `time_t` 和 `struct tm` 等基本类型
- 主要用于简单的日期时间操作

### ratio ( **C++11** )

`<ratio>` 是 C++11 引入的编译时分数库，用于在**编译时**表示和操作有理数（分数）。

## 8.6. 对可变元组使用折叠表达式（看不懂）

在折叠表达式 之前，展开参数包需要一个递归函数:

```C++
#include <iostream>
using namespace std;

template<typename T>
void f(T final) {
    cout << final << '\n';
}

template<typename T, typename... Args>
void f(T first, Args... args) {
    cout << first;
    f(args...);
}

int main() {
    f("hello", ' ', 47, ' ', "world");
}
```

- `typename... Args` 表示参数包（parameter pack）
- `Args... args` 表示函数参数包
- `args...` 表示展开参数包

C++17 以后使用折叠表达式  （先学习泛型编程）

```C++
#include <format>
#include <iostream>
#include <tuple>

using std::cout;
using std::format;
using std::tuple;
using std::get;

template<typename... T>
constexpr void print_t(const tuple<T...>& tup) {  // const tuple<T...>& tup：接受任意类型的元组引用
	// 这是 C++20 的新特性：泛型 Lambda 模板
	auto lpt = [&tup] <size_t... I> (std::index_sequence<I...> ) constexpr {
		(..., (cout <<
			format((I ? ", {}" : "{}"), get<I>(tup)))
			);
		cout << '\n';
	};
	lpt(std::make_index_sequence<sizeof...(T)>());
}

template<typename... T>
constexpr int sum_t(const tuple<T...>& tup) {
	int accum{};
	auto lpt = [&tup, &accum] <size_t... I> (std::index_sequence<I...>) constexpr {
		(..., (
			accum += get<I>(tup)
			));
	};
	lpt(std::make_index_sequence<sizeof...(T)>());
	return accum;
}

int main() {
    tuple lables{ "ID", "Name", "Scale" };
    tuple employee{ 123456, "John Doe", 3.7 };
    tuple nums{ 1, 7, "forty-two", 47, 73L, -111.11 };

    print_t(lables);
    print_t(employee);
    print_t(nums);

    tuple ti1{ 1, 2, 3, 4, 5 };
    tuple ti2{ 9, 10, 11, 12, 13, 14, 15 };
    tuple ti3{ 47, 73, 42 };
    auto sum1{ sum_t(ti1) };
    auto sum2{ sum_t(ti2) };
    auto sum3{ sum_t(ti3) };
    cout << format("sum of ti1: {}\n", sum1);
    cout << format("sum of ti2: {}\n", sum2);
    cout << format("sum of ti3: {}\n", sum3);
}
```



## 8.7. std::unique_ptr 管理已分配的内存

智能指针是管理已分配堆内存的优秀工具。

C 中的malloc() 和 free() 是最底层的内存分配操作函数

C++14 中引入的智能指针遵循资源获取即初始化 (Resource Acquisition Is Initialization, RAII) 习 惯用法，当为一个对象分配内存时，将调用该对象的构造函数。当调用对象的析构函数时，内存会 自动返回到堆中。

unique_ptr 是一个智能指针，只允许一个指针实例。它可以移动，但不能复制。

```C++
//  unique_ptr.cpp
//  as of 2022-02-12 bw [bw.org]

#include <format>
#include <iostream>
#include <string_view>
#include <memory>

using std::cout;
using std::format;
using std::string_view;
using std::make_unique;
using std::unique_ptr;

struct Thing {
	string_view thname{ "unk" };

	Thing() {
		cout << format("default ctor: {}\n", thname);
	}
	Thing(const string_view& n):thname(n) {
		cout << format("param ctor: {}\n", thname);
	}

	~Thing() {
		cout << format("dtor: {}\n", thname);
	}
};

void process_thing(const unique_ptr<Thing>& p) {
	if (p) cout << format("processing: {}\n", p->thname);
	else cout << "invalid pointer\n";
}

int main() {

	// 当只构造一个 unique_ptr 时，不会分配内存或构造一个托管对象
	// unique_ptr<Thing> p1;

	// make_unique() 工厂函数负责内存分配并返回 unique_ptr 对象，这是构造 unique_ptr 的推荐方法。
	auto p1 = make_unique<Thing>("Thing 1");
	process_thing(p1);
	process_thing(make_unique<Thing>("Thing 2"));

	// 指针对象不能复制和赋值，只能通过移动或者引用
	auto p2 = std::move(p1);
	process_thing(p1);
	process_thing(p2);

	// 当使用 new 操作符时，会分配内存并构造一个 Thing 对象:
	p2.reset(new Thing("Thing 3"));
	process_thing(p2);

	cout << "end of main()\n";
}
```

## 8.8. std::shared_ptr 的共享对象

shared_ptr 类是一个智能指针，拥有自己的托管对象，并维护一个使用计数器来跟踪副本。此示例探索了 shared_ptr 的使用方式，以在共享指针副本的同时管理内存。

```C++
//  unique_ptr.cpp
//  as of 2022-02-12 bw [bw.org]

#include <format>
#include <iostream>
#include <string_view>
#include <memory>

using std::cout;
using std::format;
using std::string_view;
using std::make_shared;
using std::shared_ptr;

struct Thing {
	string_view thname{ "unk" };

	Thing() {
		cout << format("default ctor: {}\n", thname);
	}
	Thing(const string_view& n):thname(n) {
		cout << format("param ctor: {}\n", thname);
	}

	~Thing() {
		cout << format("dtor: {}\n", thname);
	}
};

void check_thing_ptr(const shared_ptr<Thing>& p) {
	if (p) cout << format("{} use count: {}\n", p->thname, p.use_count());
	else cout << "invalid pointer\n";
}

int main() {
	
	shared_ptr<Thing> p1{ new Thing("Thing 1") };
	auto p2 = make_shared<Thing>("Thing 2");

	// 销毁副本将减少使用计数，但不会销毁管理对象。当最终副本超出作用域且使用计数为零时，对象将销毁:
	{
		cout << "make 4 copies of p1:\n";
		auto pa = p1;
		auto pb = p1;
		auto pc = p1;
		auto pd = p1;
		check_thing_ptr(p1);
		pb.reset();
		p1.reset();
		check_thing_ptr(pd);  // 在这段作用域尾部将销毁p1对象
	}
	
	check_thing_ptr(p1);
	check_thing_ptr(p2);
	cout << "end of main()\n";
}
```

## 8.9. 对共享对象使用弱指针

严格地说，std::weak_ptr 不是一个智能指针。相反，它是一个与 shared_ptr 合作运行的观察者。 weak_ptr 本身不保存指针。

某些情况下，shared_ptr 对象可能会创建悬空指针或数据竞争，这可能导致内存泄漏或其他问 题。**解决方案是使用具有 shared_ptr 的 weak_ptr 对象。**

weak_ptr 对象是一个观察者，其对 shared_ptr 对象属于非所有引用。weak_ptr 会观察 shared_ptr， 这样就知道托管对象什么时候可用，什么时候不可用。这允许在您不需要知道托管对象是否处于活 动状态的情况下，使用 shared_ptr。

```C++
//  unique_ptr.cpp
//  as of 2022-02-12 bw [bw.org]

#include <format>
#include <iostream>
#include <string_view>
#include <memory>

using std::cout;
using std::format;
using std::string_view;
using std::make_shared;
using std::shared_ptr;
using std::weak_ptr;

struct circB;
struct circA {
	shared_ptr<circB> p;
	~circA() {
		cout << "dtor A\n";
	}
};

struct circB {
	weak_ptr<circA> p;  // 弱指针
	~circB() {
		cout << "dtor B\n";
	}
};

struct Thing {
	string_view thname{ "unk" };

	Thing() {
		cout << format("default ctor: {}\n", thname);
	}
	Thing(const string_view& n):thname(n) {
		cout << format("param ctor: {}\n", thname);
	}

	~Thing() {
		cout << format("dtor: {}\n", thname);
	}
};

void get_weak_thing(const weak_ptr<Thing>& p) {
	// weak_ptr 本身不作为指针操作，其需要使用 shared_ptr.lock() 函数返回一个 shared_ptr 对象，然后可以使用该对象访问托管对象。
	if (auto sp = p.lock()) cout << format("{}: count {}\n", sp->thname, sp.use_count());
	else cout << "no shared object\n";
}
// get_weak_thing() 函数现在可以获取锁并访问托管对象。lock() 方法返回一个 shared_ptr，use_count() 反映了现在有第二个 shared_ptr 管理 Thing 对象的情况。

int main() {
	
	auto thing1 = make_shared<Thing>("Thing 1");
	weak_ptr<Thing> wp1;
	cout << format("expired: {}\n", wp1.expired());
	get_weak_thing(wp1);

	cout << "\nassign wp1 = things\n";
	wp1 = thing1;
	get_weak_thing(wp1);

	cout << "\nconstruct weak_ptr with shared_ptr\n";
	weak_ptr<Thing> wp2(thing1);
	get_weak_thing(wp2);

	cout << "\nreset thing1\n";
	thing1.reset();  // reset() 后，使用计数为零，管理对象销毁，内存释放。
	get_weak_thing(wp1);
	get_weak_thing(wp2);

	cout << "\nresolve circular reference\n";
	auto a = make_shared<circA>();
	auto b = make_shared<circB>();

	// 如果 struct circB 里面保存的是shared_ptr指针，最后析构函数从未调用
	// 因为对象维护着相互引用的共享指针，所以使用计数永远不会达到零，并且托管对象永远不会销毁。导致内存泄漏
	a->p = b;
	b->p = a;   // weak_ptr 不会增加引用计数
	
	cout << "end of main()\n";
}
```

您实际上已经解决了这个问题，因为：

```C++
struct circB {
    weak_ptr<circA> p;  // 使用 weak_ptr 打破循环引用
};
```

`weak_ptr` 不会增加引用计数，所以：

- `a` 的引用计数：1（main 中的 a） + 0（b->p 是 weak_ptr） = 1
- `b` 的引用计数：1（main 中的 b） + 1（a->p 是 shared_ptr） = 2

当离开作用域时：

- `a` 的引用计数从 1 减到 0 → 释放 circA
- circA 析构时，其 `shared_ptr<circB> p` 释放 → `b` 的引用计数从 2 减到 1
- `b` 的引用计数从 1 减到 0 → 释放 circB

## 8.10. 共享管理对象的成员

```C++
//  unique_ptr.cpp
//  as of 2022-02-12 bw [bw.org]

#include <format>
#include <iostream>
#include <memory>
#include <string>

using std::cout;
using std::format;
using std::make_shared;
using std::shared_ptr;
using std::string;
using std::tuple;

struct animal {
	string name{};
	string sound{};

	animal(const string& n, const string& s) :name(n), sound(s) {
		cout << format("ctor: {}\n", name);
	}

	~animal() {
		cout << format("dtor: {}\n", name);
	}
};

auto make_animal(const string& n, const string& s) {
	auto ap = make_shared<animal>(n, s);        // ap 引用计数 = 1
	auto np = shared_ptr<string>(ap, &ap->name);  // np 引用计数 = 2
	auto sp = shared_ptr<string>(ap, &ap->sound); // sp 引用计数 = 2
	return tuple(np, sp);
}

int main() {
	
	auto [name, sound] = make_animal("Velociraptor", "Grrrr!");
	cout << format("The {} says {}\n", *name, *sound);
	cout << format("Use count: name {}, sound {}\n", name.use_count(), sound.use_count());
	
	cout << "end of main()\n";
}
```

**引用计数解释**

1. **`ap`**：直接拥有 `animal` 对象，引用计数 = 1
2. **`np`**：通过别名构造函数创建，它**共享 `ap` 的所有权**但指向 `ap->name`
   - 增加 `ap` 的引用计数到 2
3. **`sp`**：同样通过别名构造函数创建，共享 `ap` 的所有权但指向 `ap->sound`
   - 增加 `ap` 的引用计数到 3

当函数返回时：

- `ap` 被销毁，引用计数从 3 减到 2
- 剩下的 `np` 和 `sp` 各持有一个引用

**内存释放时机**

当 `main` 函数结束时：

1. `sound` 析构 → 引用计数从 2 减到 1
2. `name` 析构 → 引用计数从 1 减到 0
3. 引用计数为 0，`animal` 对象被销毁

## 8.11. 比较随机数引擎（未做）

## 8.12. 比较随机数分布发生器（未做）

# 第 9 章 并发和并行（未做）

# 第 10 章 文件系统（未做）

标准库使用  头文件，std::filesystem 命名空间通常别名为 fs:

### filesystem ( **C++17** ) ( 不会用 )

`<filesystem>` 头文件的主要作用是**为 C++ 程序提供一套统一的接口，用于执行常见的文件系统操作**

1. `std::filesystem::path` - 路径管理类

```C++
#include <iostream>
#include <filesystem>
namespace fs = std::filesystem; // 常用命名空间别名

int main() {
    // 创建一个路径对象
    fs::path p = "/usr/local/bin/my_app";

    // 分解路径
    std::cout << "文件名: " << p.filename() << std::endl;   // "my_app"
    std::cout << "扩展名: " << p.extension() << std::endl;  // ""
    std::cout << "父路径: " << p.parent_path() << std::endl; // "/usr/local/bin"

    // 组合路径
    fs::path config_path = p.parent_path() / "config" / "app.conf";
    std::cout << "配置路径: " << config_path << std::endl; 
    // 在Unix上输出: "/usr/local/bin/config/app.conf"
    // 在Windows上输出: "\\usr\\local\\bin\\config\\app.conf" (如果p是Windows路径)
}
```

2. 目录遍历

这是 `<filesystem>` 库最强大的功能之一。

- `std::filesystem::directory_iterator`：用于遍历目录中的条目（不递归）。
- `std::filesystem::recursive_directory_iterator`：用于递归遍历目录及其所有子目录。

```C++
#include <iostream>
#include <filesystem>
namespace fs = std::filesystem;

int main() {
    std::string folder_path = "./some_folder";

    try {
        // 遍历目录
        for (const auto& entry : fs::directory_iterator(folder_path)) {
            const auto& path = entry.path();
            std::cout << "找到: " << path.filename() << " ["
                      << (fs::is_directory(entry.status()) ? "目录" : "文件") 
                      << "]" << std::endl;
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "文件系统错误: " << e.what() << std::endl;
    }
}
```

3. 文件和目录操作与查询

提供了一系列函数来检查状态和进行操作。

- **查询操作**：
  - `exists(path)`：路径是否存在。
  - `is_directory(path)`, `is_regular_file(path)`：判断路径类型。
  - `file_size(path)`：获取文件大小。
  - `last_write_time(path)`：获取最后修改时间。
- **操作命令**：
  - `create_directories(path)`：递归创建目录（类似于 `mkdir -p`）。
  - `copy(source, target)`：复制文件或目录。
  - `rename(old_path, new_path)`：重命名或移动文件/目录。
  - `remove(path)`, `remove_all(path)`：删除文件或递归删除目录。

```C++
#include <iostream>
#include <filesystem>
#include <chrono>
namespace fs = std::filesystem;

int main() {
    fs::path my_file = "test.txt";

    // 检查文件是否存在及其属性
    if (fs::exists(my_file)) {
        std::cout << "文件大小: " << fs::file_size(my_file) << " 字节" << std::endl;
        auto ftime = fs::last_write_time(my_file);
        // 可以将ftime转换为time_t并进一步格式化输出
    } else {
        std::cout << "文件不存在。" << std::endl;
    }
    
    // 创建新目录
    fs::create_directories("./backup/data");

    // 复制文件
    fs::copy(my_file, "./backup/data/test_backup.txt");
}
```

