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
