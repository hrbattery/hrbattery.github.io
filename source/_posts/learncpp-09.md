---
title: learncpp - 09 复合类型 - 引用和指针
date: 2025-03-25 01:39:16
tags:
---

大的要来了

<!-- more -->

## 9.1 - 复合数据类型

C++支持以下复合数据结构：

- 函数/Function
- 数组/Array
- 指针/Pointer type
  - 指向对象的指针/Pointer to object
  - 指向函数的指针/Pointer to function
  - 指向成员的指针/Pointer to member
    - 指向数据成员的指针/Pointer to data member
    - 指向成员函数的指针/Pointer to member function
- 引用/Reference type
  - 左值引用/L-value references
  - 右值引用/R-value references
- 枚举/Enumerated type
  - 无作用域枚举类型/Unscoped enumeration
  - 限定作用域枚举/Scoped enumeration
- 类/Class type
  - 结构体/Struct
  - 类/Class
  - 联合体/Union

## 9.2 - 值的分类（左值和右值）

C++中所有的表达式都有两个属性：**表达式类型**和**表达式值分类**。

前者等于**表达式求值所得到的值、对象或函数的类型**，编译器会借此判断表达式在特定上下文中是不是合法的表达式。

后者则表明一个表达式会求值得到一个值、一个函数还是一个对象。这个属性就是我们所讨论的左值和右值。

左值类型的表达式，最终会求值得到一个具有身份特征的函数或对象，所谓的「具有身份特征」，指一个对象**具有标识符**或者一个**可被标识的内存地址**。具有标识的对象的生命周期会超过表达式本身。

右值类型的表达式则是除了左值以外的所有表达式。常见的右值包括字面量、函数或操作符的返回值，其作用域只存在于使用它的表达式中。

## 9.3 - 左值引用

之前说了这么多的左值和右值，就是为了引出下面的这碟饺子。

在 C++ 中，**引用其实就是某个已存在对象的引用**。一旦引用被定义，**任何对引用的操作都会直接应用于被引用的对象**。可以说，引用在本质上和被引用的对象是一回事。

现代C++存在两种类型的引用：**左值引用**和**右值引用**。在本章中我们会首先讨论左值引用。

我们使用&符号来声明左值类型：

```cpp
int      // a normal int type
int&     // an lvalue reference to an int object
double&  // an lvalue reference to a double object
```

这可以用来声明一个左值引用变量。

```cpp
int main()
{
    int x { 5 };    // x is a normal integer variable
    int& ref { x }; // ref is an lvalue reference variable that can now be used as an alias for variable x
 
    std::cout << x << '\n';  // print the value of x (5)
    std::cout << ref << '\n'; // print the value of x via ref (5)
 
    return 0;
}
```

**所有的引用都必须被初始化。引用一旦被初始化，便不能被重新绑定到其他对象，**

引用和被引用对象具有独立的生命周期。这意味着可能发生以下两种情况：

- 引用可以先与被引用对象销毁，这并不会对被引用对象造成任何影响。
- 被引用对象可以先于引用销毁。该引用会成为一个**悬垂引用**，对其访问会导致未定义行为。

普通的左值引用只能绑定到一个非常量的左值上，所以我们需要一种可以指向const的左值引用：

```cpp
#include <iostream>
 
int main()
{
    const int x { 5 };    // x is a non-modifiable lvalue
    const int& ref { x }; // okay: ref is a an lvalue reference to a const value
 
    std::cout << ref;     // okay: we can access the const object
    ref = 6;              // error: we can not modify a const object
 
    return 0;
}
```

const左值引用可以也可以绑定可修改的左值，但此时就不能通过引用来修改原对象了。**甚至，const左值引用可以绑定到一个右值上。**

**当const左值引用被绑定到一个临时对象时，临时对象的生存期将被扩展到与引用的生存期相匹配。**

