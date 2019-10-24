# C++ Templates 读书笔记
## 类模板的特化
可以用模板实参来特化类模板。和函数模板的重载类似，通过特化类模板，你可以优化基于某种特定类型的实现，或者克服某种特定类型在实例化类模板时多出现的不足。
* 为了特化一个类模板，你必须在起始处声明一个template<>，接下来声明用来特化类模板的类型。这个类型被用作模板实参，且必须在类名的后面直接指定。
```c++
template<>
class Stack<std::string> {
  ...
}
``` 
* 进行类模板的特化时，每个成员函数都必须重新定义为普通函数，原来模板函数中的每个T也相应地被进行特化的类型取代：
```c++
void Stack<std::string>::push (std::string const& elem)
{
    elems.push_back(elem);
}
```

## 包含模型(inclusion model)
按照cpp的习惯：
* 类和其他类型都被放在一个头文件中。
* 对于全局变量和(非内联)函数，只有声明放在头文件中，定义则位于dot-C文件。
我们习惯于作出以下编写：
```c++
//basic/myfirst.hpp
#ifndef MYFIRST_HPP
#DEFINE MYFIRST_HPP

//模板声明
template <typename T>
void print_typeof (T const&)

#endif //MYFIRST_HPP
```
```C++
//basics/myfirst.cpp
#include <iostream>
#incldue <typeinfo>
#include "myfirst.hpp"

//模板的实现/定义
template <typename T>
void print_typeof (T const& x)
{ 
    std::cout << typeid(x).name() << std::endl;
}
```
当我们在另一个dot-C文件里使用这个模板，并且把模板声明包含进这个文件：
```c++
//basics/myfirstmain.cpp
#include "myfirst.hpp"

//使用模板
int main()
{
    double ice = 3.0;
    print_typef(ice); // 调用参数类型为double的函数模板
}
```
大部分C++编译器都会顺利地接受这个程序，但是链接器可能会报错，提示找不到函数print_typeof()的定义。<br>
解决方法：
* 使用包含模型或者分离模型
* 包含模型
通过把
```c++
#include "myfirst.cpp"
```
添加到myfirst.hpp的末尾，或者在每个使用模板的dot-C文件都包含myfirst.cpp。在或者完全不要myfirst.cpp，然后重写myfirst.hpp，让它同时包含模板声明和模板定义。
* 分离模型(separation model)
又称导出模型(export template)
```c++
//使用export关键字
```
## 模板参数
### 类型参数(使用得最多)
类型参数是通过关键字typename或者class引入的：它们两者几乎是等同的。关键字后面必须是一个简单的标识符，后面用逗号来隔开下一个参数声明，等号(=)代表接下来的是缺省模板参数，一个封闭的尖括号(>)表示参数化字句的结束。
### 非类型参数
非类型参数表示的是：在编译期或者链接期可以确定的常值。这种参数的类型必须是下面的一种:
* 整形或者枚举类型。
* 指针类型（包含普通对象的指针类型、函数指针类型、指向成员的指针类型）。
* 引用类型（指向对象或者指向函数的引用都是允许的）
### 模板的模板参数
模板的模板参数是代表类模板的占位符（placeholder）。它的声明和类模板的声明很类似，但不能使用关键字struct和union。
```c++
template <template<typename T,
                  typename A = MyAllacator> class Container>
class Adaptation {
    Container<int> storage; //隐式等同于Container<int, MyAllocator>
    ...
};
```

### List参数
有时候，我们希望可以把具有几个类型的列表看成一个单一的模板实参，并用这个单一的实参进行传递。通常情况下，使用这种列表会有两个目的：声明一个参数个数不固定的函数，或者定义一种具有成员个数不固定的类型结构。
例如，我们希望定义一个能够计算任意多个值中最大者的模板。一种可能的声明语法是：使用省略号标记，从而说明最后一个模板参数的含义是允许匹配任意个数的实参。
```c++
#include <iostream>

template <typename T, ... list>
T const& max (T const, T const&, list const&);
int main()
{
    std::cout << max(1, 2, 3, 4) << std::endl;
}

template <typename T> inline
T const& max (T const& a, T const& b)
{
    return a < b ? b : a; // 我们用于求普通的二元最大值的操作
}

//模板函数的实现1,使用重载函数模板
template <typename T, ... list> inline
T const& max (T const& a, T const& b, list const& x)
//模板函数的实现2,使用模板元编程
```



