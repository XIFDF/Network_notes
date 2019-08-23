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
