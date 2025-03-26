---
title: learncpp - 10 - 复合类型 - 枚举和结构体
date: 2025-03-26 13:37:59
tags:
---

别急

<!-- more -->

## 10.2 - 无作用域枚举类型

稍微了解下枚举的写法就好了

```cpp
enum CardSuits
{
    Clubs = 1,
    Diamonds,
    Hearts = 4,
    Spades
};
```

之所以叫**无作用域枚举类型**，是因为它会将定义的枚举值暴露在其所在的作用中、而不是像namespace那样创造了一个新的作用域，所以像下面这样的写法过不了编译：

```cpp
enum Color
{
    red,
    green,
    blue, // blue 在全局作用域
};
 
enum Feeling
{
    happy,
    tired,
    blue, // 错误：命名冲突
};
```

但命名作用是有的：

```cpp
enum Color
{
    Red,
    Blue,
    Green
};

int main()
{
    Color apple {Red};
    Color strawberry {Color::Red};

    return 0;
}
```

可以在前面套上一层namespace，但还有高手

## 10.4 - 限定作用域枚举（枚举类）

有作用域枚举和无作用域枚举用法类似，不过它们有两个主要的区别：
1. 有作用域枚举是强类型的(它**不会被隐式地转换为整型**) 
2. 它具有强作用域约束 (它的**枚举值只会被放置在枚举类的作用域中**)。

使用`enum class`来创建有作用域枚举。

不过其不会隐式转化为整型的特性，导致要经常使用static_cast来转换，

## 10.5 - 结构体、成员和成员选择

```cpp
struct Employee
{
    int id {};
    int age {};
    double wage {};
};

// 使用`.`来访问成员变量

int main()
{
    Employee joe;
 
    joe.age = 32;  // use member selection operator (.) to select the age member of variable joe
 
    std::cout << joe.age << '\n'; // print joe's age
 
    return 0;
}
```

> **记得写分号**

## 10.6 - 结构体的聚合初始化

```cpp
struct Employee
{
    int id {};
    int age {};
    double wage {};
};
 
int main()
{
    Employee frank = { 1, 32, 60000.0 }; // 拷贝列表初始化，使用大括号
    Employee robert ( 3, 45, 62500.0 );  // 使用小括号的直接初始化(C++20)
    Employee joe { 2, 28, 45000.0 };     // 使用大括号列表的列表初始化（推荐）
    return 0;
}
```

如果聚合数据类型被初始化，但初始化值列表中的值个数少于成员个数，则剩余的成员会被值初始化。

我们也可以设置一个默认初始化值：

```cpp
struct Something
{
    int x;       // no initialization value (bad)
    int y {};    // value-initialized by default
    int z { 2 }; // explicit default value
};
 
int main()
{
    Something s1; // s1.x is uninitialized, s1.y is 0, and s1.z is 2
 
    return 0;
}
```

最好始终提供结构体的默认初始化值。

## 10.8 - 结构体传递及其他

通常通过const引用来传递结构体（避免拷贝和修改），而在返回时按值返回（避免产生悬垂引用）。

## 10.9 - 基于指针和引用的成员选择

对于结构体的引用，我们可以像对结构体本身一样使用`.`来访问结构体成员：

```cpp
struct Employee
{
    int id{};
    int age{};
    double wage{};
};

void printEmployee(const Employee& e)
{
    std::cout << "Id: " << e.id << '\n';
    std::cout << "  Age: " << e.age << '\n';
    std::cout << "  Wage: " << e.wage << '\n';
}

int main()
{
    Employee joe{ 1, 34, 65000.0 };
 
    ++joe.age;
    joe.wage = 68000.0;
 
    printEmployee(joe);
 
    return 0;
}
```

但对结构体的指针是不能直接用成员选择运算符的，我们必须得先解引用，类似`(*ptr).id`，这样很麻烦，所以C++定制了一个箭头选择符来基于对象指针来访问其成员：

```cpp
#include <iostream>
 
struct Employee
{
    int id{};
    int age{};
    double wage{};
};
 
int main()
{
    Employee joe{ 1, 34, 65000.0 };
 
    ++joe.age;
    joe.wage = 68000.0;
 
    Employee* ptr{ &joe };
    std::cout << ptr->id << '\n'; // Better: use -> to select member from pointer to object
 
    return 0;
}
```

这其中就包括了隐式的解指针操作。

## 10.10 - 类模板

既然有函数模板，自然可以有类模板和结构体模板

```cpp
#include <iostream>
 
template <typename T>
struct Pair
{
    T first{};
    T second{};
};
```

从C++ 17开始，当从类模板实例化一个对象时，编译器可以从对象的初始化式的类型推断出模板类型：

```cpp
#include <utility> // for std::pair
 
int main()
{
    std::pair<int, int> p1{ 1, 2 }; // explicitly specify class template std::pair<int, int> (C++11 onward)
    std::pair p2{ 1, 2 };           // CTAD used to deduce std::pair<int, int> from the initializers (C++17)

    // CTAD 只有在类模板列表中没有提供任何参数时才会进行，因此下面代码中的两种方式都是错误的
    std::pair<> p3 { 1, 2 };    // 错误: 模板参数太少, 两个参数都不会推断
    std::pair<int> p4 { 3, 4 }; // 错误: 模板参数太少,第二个参数不会被推断
 
    return 0;
}
```