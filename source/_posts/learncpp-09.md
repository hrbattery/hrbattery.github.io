---
title: learncpp - 09 - 复合类型 - 引用和指针
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

## 9.5 - 传递左值引用

此前我们所知道的函数传参的方式只有一种：按值传递，是通过值复制的方式传递的。现在我们学另一种：**按引用传递**。

不适用按值传递的原因有二：
- 有些对象的拷贝开销很大，是非必要的拷贝，毕竟很多时候函数作用域内的变量都是用完即弃的；
- 某些时候，我们希望在函数内修改传入的实参。

```cpp
#include <iostream>
#include <string>
 
void printValue(std::string& y) // type changed to std::string&
{
    std::cout << y << '\n';
} // y is destroyed here
 
int main()
{
    std::string x { "Hello, world!" };
 
    printValue(x); // x is now passed by reference into reference parameter y (inexpensive)
 
    return 0;
}
```

如果我们定义按引用传递的参数为非const值的引用，那么其就不能接收const变量或者字面量，这极大地影响了按引用传递的实用性；同时，很多时候我们也不希望函数内可以修改传入的实参。因此，我们可以将参数定义为const的引用：

```cpp
#include <iostream>
#include <string>
 
void printValue(const int& y) // y is now a const reference
{
    std::cout << y << '\n';
}
 
int main()
{
    int x { 5 };
    printValue(x); // ok: x is a modifiable lvalue
 
    const int z { 5 };
    printValue(z); // ok: z is a non-modifiable lvalue
 
    printValue(5); // ok: 5 is a literal rvalue
 
    return 0;
}
```

这样就可以绑定任何类型的实参了。**按引用传递最好定义为const类型的引用，除非你有很好的理由（例如函数必须修改实参的值）。**

**对基本数据类型使用按值传递，而对于类或者结构体使用const的按值传递。**

## 9.6 指针

> 大的要来了

在默认情况下，变量的地址并不会返回给用户，但实际上是可以获取该地址的：使用**取地址操作符&** 就可以返回其操作变量的地址。

```cpp
#include <iostream>
 
int main()
{
    int x{ 5 };
    std::cout << x << '\n';  // print the value of variable x
    std::cout << &x << '\n'; // print the memory address of variable x
 
    return 0;
}
```

> &符号在不同的语境下有不同的含义，比较容易混淆：
> 
> - 当后面接着类型名时，&表示一个左值引用:  `int& ref`；
> - 当用于一元表达式时，&表示取地址运算符:  `std::cout << &x`.
> - 当用于二元表达式时，&是按位与操作:  `std::cout << x & y`.

当然，只是取得变量的地址没什么用，我们需要取得其存放的值：使用**解引用运算符\***。

### 指针

指针可以看作是一个持有内存地址的对象，和使用&定义引用类型类似，我们使用*定义指针类型：

```cpp
int;  // a normal int
int&; // an lvalue reference to an int value
 
int*; // a pointer to an int value (holds the address of an integer value)
```

和引用类型不同，**指针默认是不初始化的**，但这么干的话就会多出来一个**野指针**，这不好

> **将军说过，指针必须初始化**

一旦我们有了一个持有对象地址的指针，我们就可以用解引用操作符*来访问该地址。

```cpp
#include <iostream>
 
int main()
{
    int x{ 5 };
    std::cout << x << '\n'; // 打印变量 x 的值
 
    int* ptr{ &x }; // ptr 保存着 x 的地址
    std::cout << *ptr << '\n'; // 使用解引用操作符打印该地址的内容
 
    return 0;
}
```

### 指针和赋值

在谈论指针的赋值时，可能有两种含义：

1. 给指针赋一个新的值，使其指向其他地址
2. 给指针解引用的结果赋新值，改变指针所指内容的值

需要注意，**取地址操作符&**返回的并不是地址的字面量，它返回的是一个指向操作数地址的指针，该指针的类型取决于参数的类型。

## 9.8 - 指针和const

和引用类似，由于const变量的不可变性，**普通指针不能指向const变量**。讨论指针和const的关系，我们会得出三种排列组合：

- 指向常量的指针：const int*
- 常量指针：int* const
- 指向常量的常量指针：const int* const

与引用类似，指向常量的指针也可以指向非常量的变量，但该指针指向的值会被视为常量，无论最初的定义如何。

## 9.9 - 按地址传递

既然引用可以引申出按引用传递的传参方式，那么基于指针也可以引申出一种新的传参方式：**按地址传递**。

```cpp
#include <iostream>
#include <string>
 
void printByValue(std::string val) // The function parameter is a copy of str
{
    std::cout << val << '\n'; // print the value via the copy
}
 
void printByReference(const std::string& ref) // The function parameter is a reference that binds to str
{
    std::cout << ref << '\n'; // print the value via the reference
}
 
void printByAddress(const std::string* ptr) // The function parameter is a pointer that holds the address of str
{
    std::cout << *ptr << '\n'; // print the value via the dereferenced pointer
}
 
int main()
{
    std::string str{ "Hello, world!" };
 
    printByValue(str); // pass str by value, makes a copy of str
    printByReference(str); // pass str by reference, does not make a copy of str
    printByAddress(&str); // pass str by address, does not make a copy of str
 
    return 0;
}
```

在传入函数时，与按引用传递不同，我们传入的是对象的指针&str。而在函数内部，我们使用*ptr解引用来获取指针对应的str的值。

**除非有特殊原因要使用通过地址传递，否则最好通过引用传递而不是通过地址传递。**

不过实际上其实这三种传递值的方式都是按值传递。引用通常由编译器由指针实现，而传递进去的地址可以被解引用、以此修改实参罢了，而普通形参做不到这一点。

## 9.11 - 按引用返回和按地址返回

参数可以花式传入函数，函数自然也可以花式传出返回值。类似的，有按引用返回和按地址返回的返回方式。

```cpp
std::string&       returnByReference(); // returns a reference to an existing std::string (cheap)
const std::string& returnByReferenceToConst(); // returns a const reference to an existing std::string (cheap)
const std::string* returnByPointerToConst(); // returns a const pointer to an existing std::string
```

但需要注意以下要点：
- **永远不要通过引用返回局部变量**。这会导致悬垂引用。
- **不要按引用返回非const的局部静态变量**，这会容易导致预期外的变量被修改。
- 如果参数本身是按引用传递进函数的，那么按引用返回就是安全的。

按地址返回的返回值是指向对象的指针，主要的优点在于**可以用nullptr来代表没有要返回的有效对象**。但代价是，可能会发生空指针解引用导致未定义行为，所以优先选择按引用返回。

## 9.12 - 指针、引用和const的类型推断

`auto`关键字可以让编译器根据初始化值对变量类型进行推断。默认情况下，类型推断也会删除引用修饰符。如果有需要，我们可以手动添加回来。

对于`const`的处理，要分类讨论：

- 如果这个`const`修饰符是修饰对象本身的，我们称其为顶层`const`，这会被类型推断所丢弃。
- 如果这个`const`修饰符是修饰指向对象的引用或者指针，我们称其为底层`const`，这不会被类型推断所丢弃。

加上类型推断时舍弃和加上修饰符的顺序问题，这可能会导致在类型推断时，有的类型会和预期有所差别。只要在使用`auto`修饰符时显式写出需要的修饰符，一般就不会有问题。

```cpp
#include <string>
 
const std::string& getRef(); // some function that returns a const reference
 
int main()
{
    auto ref1{ getRef() };        // std::string (top-level const and reference dropped)
    const auto ref2{ getRef() };  // const std::string (const reapplied, reference dropped)
 
    auto& ref3{ getRef() };       // const std::string& (reference reapplied, low-level const not dropped)
    const auto& ref4{ getRef() }; // const std::string& (reference and const reapplied)
 
    return 0;
}
```

但与引用不同，指针是不会被丢弃的。基本只有这一点区别，总之无论是什么修饰符，尽量都明确标识出来、让自己的意图更加明确，避免导致语义不清导致的错误。
