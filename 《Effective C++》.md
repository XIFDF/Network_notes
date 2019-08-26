# 《Effective C++》读书笔记
## 摘自《Effective C++》第三版
## 条款01：视C++为一个语言联邦
### View C++ as a federation of language.
关注C++的四个次语言：
* C：C语言
* Object-Oriented C++：面向对象
* Template C++：泛型编程
* STL
## 条款02：尽量以 const，enum，inline 替换 #define
### Prefer const,enums,and inlines to #defines.
对于单纯常量，最好以const对象或enums替换#define。
```c++
#define ASPECT_RATIO 1.653

/*替换成*/

const double AspectRatio = 1.653;
```
如果为了将常量的作用域(scope)限制在class内，你必须让它成为class的一个成员(member)<br>
而为确保此常量至多只有一份实体，你必须让它成为一个static成员：
```c++
class GamePlayer {
    static const int NumTurns = 5;  //常量声明式
    int scores[NumTurns];           //使用该常量
    ...
};
```
对于形似函数的宏(macros)，最好改用inline函数替换#define。
```c++
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))

/*替换成*/

template<typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b);
}
```
## 条款03：尽可能使用const
### Use const whenever possible
如果关键字const出现在星号左边，表示被指物体是常量；如果出现在星号右边，表示指针自身是常量；如果出现在星号两边，表示被指物和指针两者都是常量。<br>
主要作用：将某些东西声明为const可帮助编译器侦查出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。<br>
* 如果要使用迭代器
```c++
std::vector<int> vec;
...
const std::vector<int>::iterator iter = vec.begin();    //iter 的作用像个 T* const
*iter = 10;       //没问题，改变iter所指物
++iter;           //错误！iter是const

std::vector<int>::const_iterator cIter = vec.begin();   //cIter 的作用像个const T*
*cIter = 10;      //错误！*cIter是const
++cIter;          //没问题，改变cIter
```
* 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可以避免代码重复。
```c++
class TextBlock {
public:
    ...
    const char& operator[](std::size_t position) cosnt  //一如既往
    {
        ...
        ...
        return text[position];
    }
    char& operator[](std:size_t position)
    {
        return 
            const_cast<chat&>(
                static_cast<const TextBlock&>(*this)
                    [position]
            );
    }
};
...
```
