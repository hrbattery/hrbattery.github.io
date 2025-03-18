---
title: learncpp - 04 - 基础数据类型
date: 2025-03-19 01:39:16
tags:
---

初见端倪

## 4.1 基础数据类型简介

- 变量本质上是一段可以存放信息的内存的名称，而数据类型是计算机用以将内存数据解析为有意义的值的规则
- 一般来说，1byte=8bit
- c++的基本类型只能用一大坨来形容，这就是祖师爷级语言给我的自信
  - 整数类型：short、int、long、long long
  - 字符类型：char、wchar_t、char8_t、char16_t、char32_t
  - 布尔类型：bool
  - 浮点类型：float、double、long double
  - std::nullptr_t、void
- 热知识：字符串不是基本类型

## 4.3 - 对象的大小和 sizeof 操作符

跑一跑下面这段代码：

```cpp
#include <iostream>
 
int main()
{
    std::cout << "bool:\t\t" << sizeof(bool) << " bytes\n";
    std::cout << "char:\t\t" << sizeof(char) << " bytes\n";
    std::cout << "char16_t:\t" << sizeof(char16_t) << " bytes\n";
    std::cout << "char32_t:\t" << sizeof(char32_t) << " bytes\n";
    std::cout << "short:\t\t" << sizeof(short) << " bytes\n";
    std::cout << "int:\t\t" << sizeof(int) << " bytes\n";
    std::cout << "long:\t\t" << sizeof(long) << " bytes\n";
    std::cout << "long long:\t" << sizeof(long long) << " bytes\n";
    std::cout << "float:\t\t" << sizeof(float) << " bytes\n";
    std::cout << "double:\t\t" << sizeof(double) << " bytes\n";
    std::cout << "long double:\t" << sizeof(long double) << " bytes\n";
 
    return 0;
}
```
总之在mac上出来的结果是这样的
```
bool:		1 bytes
char:		1 bytes
char16_t:	2 bytes
char32_t:	4 bytes
short:		2 bytes
int:		4 bytes
long:		8 bytes
long long:	8 bytes
float:		4 bytes
double:		8 bytes
long double:	8 bytes
```

比较有趣的是long居然和longlong是一样的（

## 4.4 - 有符号整型

- 有符号整型的溢出会导致未定义行为

## 4.5 - 无符号整型以及为什么要避免使用它

- 无符号整型表示超过范围的数时不会产生未定义行为，它的行为是确定的
- 但总之能不用unsigned的就不用，其中一个理由是两个signed的能溢出的时候很好发现，但unsigned因为不能保存负数所以一猪脑过载就忘了触发溢出了（
- 但还是有一些场合是需要或者可以使用无符号数的
  - **在进行位运算时是推荐使用的**，在一些确实需要反转行为的场合，无符号数也是有用的（例如一些加密算法和随机数生成）。
  - 在有些时候无符号数是无法避免的（很多与数组进行索引有关），这时通常直接转成有符号数也没啥（
  - 在嵌入式系统等内存吃紧时，出于性能考虑使用，能干这个的人是这个（

## 4.6 - 固定宽度整型和 size_t

- 从c继承来的整型大小可以随地大小变，所以c++决定提供固定宽度的整型
  - 比如int16_t之类的
  - 但并不是用越小宽度的类型越牛逼，注意字节对齐的问题
  - 所以还有`std::int_fast#_t`和`std::int_least#_t`这种玩意儿，记得有这种东西就行
- `std::int8_t`和`std:: uint8_t`的行为可能更像字符而非整型，有坑别用

一些最佳实践：
- 如果整型的大小不重要（例如存放的数总是在2字节有符号整型范围内），推荐使用`int`。
- 如果需要存储数量值，且范围必须明确时，推荐使用`std::int#_t`；
- 如果需要进行位运算或想要利用无符号数的翻转特性时，使用`std::uint#_t`。

尽量避免：
- 使用无符号类型保存数量值；
- 使用 8 为固定宽度的整型；
- 使用速度或大小优先的固定宽度类型；
- 任何与特定编译器相关的固定整型类型