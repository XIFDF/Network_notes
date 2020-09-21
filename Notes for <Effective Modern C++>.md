# 《Effective Modern C++》读书笔记
## 摘自《Effective Modern C++》中文版
## 条款01：理解模板型别推导
在编译期，编译期会通过expr推导两个型别：一个是T的类型，另一个是ParamType的型别，这两个型别往往不一样。因为，ParamType常会包含了一些饰词，如const或引用符号等限定词。
```c++
// 伪代码
template<typename T>
void f(ParamType param);

// 模板声明
template<typename T>
void f(const T& param);  // ParaType 是 const T&

// 调用
int x = 0;
f(expr)
```
### T的型别推导结果，不仅仅依赖expr的型别，还依赖ParamType的形式。具体分三种情况：
* ParamType具有指针或者引用型别，但不是个万能引用。
* ParamType是一个万能引用。
* ParamType既非指针也非引用。
#### ParamType是一个指针或引用，但不是个万能引用
1. 若expr具有引用类型，先将引用部分忽略。
2. 尔后，对expr的型别和ParamType的型别执行模式匹配，来决定T的型别。
```c++
// 模板声明
template<typename T>
void f(T& param);   // param是个引用

// 变量声明
int x = 27;         // x的型别是int
const int cx = x;   // cx的型别是const int
const int& rx = x;  // rx是x的型别为const int的引用

// 推导结果
f(x);       // T的型别是int，param的型别是int&
f(cx);      // T的型别是const int， param的型别是 const int&
f(rx);      // T的型别是const int， param的型别是 const int&

// 若模板声明为
template<typename T>
void f(const T& param);

//推导结果
f(x);       // T的型别是const int， param的型别是 const int&
f(cx);      // T的型别是const int， param的型别是 const int&
f(rx);      // T的型别是const int， param的型别是 const int&
```

#### ParamType是一个万能引用
在函数模板中持有型别形参T时，万能引用的声明型别写作T&&
1. 如果expr是个左值，T和ParamType都会被推导为左值引用，这个结果具有双重的奇特之处：首先，这个在模板型别推导中，T被推导为引用型别的唯一情形。其次，尽管在声明时使用的是右值引用语法，它的类型推导结果却是左值引用。
2. 如果expr是个右值，则应用“常规”（即情况1中的）规则。


#### ParamType既非指针也非引用
