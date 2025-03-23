---
title: learncpp - 08 - 类型转换和函数重载
date: 2025-03-23 16:14:29
tags:
---

暴雨前的平静

<!-- more -->

C++ 标准定义了不同的基本类型的转换方式：

- 数值提升
- 数制转换
- 算数转换
- 其他转换

本章的前半解会先讲前三种。

## 8.2 - 浮点数和整型提升

数值提升将更窄的数据类型（例如char）转换为更宽的数据类型（例如 int 或 double），使其能被更高效地处理和避免溢出。
所有数值提升都是安全的，所以编译器会根据自己需求自由地使用数值提升、且不会有任何警告。

浮点类型的数值提升比较简单，而整型会稍微复杂点：

- 无符号`char`或者有符号`char`可以被转换为`int`；
- 无符号`char`，`char8_t`以及无符号`short`可以被转换为`int`或无符号`int`；
- 如果`char`默认有符号的， 则会遵循上述有符号`char`的转换规则。如果默认是无符号的，则遵循上述无符号`char`的转换规则；
- `bool`可以被转换为`int`，`false`转换为`0`，`true`转换为`1`。

在大多数情况下，这允许我们编写一个接受形参类型为int的函数，然后与其他各种整型类型配合使用。

## 8.3 - 数值转换

五种基本的数值转换：

- 把一种整型类型转换为另外一种整型类型（整型提升除外）
- 把一种浮点类型转换为另外一种浮点类型（浮点提升除外）
- 将浮点数类型转换为任何整型类型
- 将整型类型转换为任何浮点类型
- 将整型或浮点型转换为bool类型

数值转换可能会导致数据或者精度的丢失，但也可能不会，前者被称为**缩窄转换**。

**尽可能避免缩窄转换。如果一定要进行转换，使用 `static_cast` 显式地进行缩窄转换。**

**括号初始化不允许隐式缩窄转换。**

## 8.4 - 算术转换

C++ 中的一些二元运算符要求两个操作数具有相同的优先级，如果不满足则需要进行类型转换，这叫做**算术转换**。

编译器具有一个按照优先级排序的类型列表：

- long double(最高优先级)
- double
- float
- unsigned long long
- long long
- unsigned long
- long
- unsigned int
- int(最低优先级)

以及两条规则：

- 如果至少有一个操作数在优先级列表中，则具有较低优先级的操作数会被转换为具有较高优先级的操作数；
- 否则(两个操作数的类型均不在表中)，则两个操作数会进行数值提升。

当混合有符号和无符号值时，这种优先级层次结构可能会导致一些问题，比如int会被转换成unsigned int导致溢出，导致预期以外的结果。所以，**避免无符号整数**。

## 8.9 - 函数重载

使用相同函数名但形参不同的函数，称为函数重载。重载函数时，编译器会根据形参个数和形参类型进行区分，这被统称为函数的类型签名，

对于成员函数，const、volatile和引用限定符会被考虑。

将函数调用匹配到特定重载函数的过程称为重载解析：

- 完全匹配
- 数值提升匹配
- 数值转换匹配
- 自定义类型转换匹配
- 省略号匹配
- 放弃并报错

## 8.13 - 函数模板

泛型换个马甲以为就认不出来吗（

```cpp
template <typename T> // this is the template parameter declaration
T max(T x, T y) // this is the function template definition for max<T>
{
    return (x > y) ? x : y;
}
```

模板定义并不受限于ODR，而且通常实例化的函数是隐式内联函数，也不受限于ODR。

我们可以让函数模型支持多个不同的类型：

```cpp
#include <iostream>
 
template <typename T, typename U> // We're using two template type parameters named T and U
T max(T x, U y) // x can resolve to type T, and y can resolve to type U
{
    return (x > y) ? x : y; // uh oh, we have a narrowing conversion problem here
}
 
int main()
{
    std::cout << max(2, 3.5) << '\n';
 
    return 0;
}
```

在c++20里头我们也可以这么写：

```cpp
auto max(auto x, auto y)
{
    return (x > y) ? x : y;
}
```