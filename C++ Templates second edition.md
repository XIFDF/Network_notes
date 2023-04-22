# C++ Templates 第二版 读书笔记
## 6.1 Perfect Forwarding
Suppose you want to write generic code that forwards the basic property of passed
arguments:
* Modifyable objects should be forwarded so that they still can be modified.
* Constant objects should be forwarded as read-only objects.
* Movable objects (objects we can “steal” from because they are about to expire)
should be forwarded as movable objects.
To achieve this functionality without templates, we have to program all three cases.
For example, to forward a call of f() to a corresponding function g\

假设您想编写通用代码，以转发传递参数的基本属性：
* 可修改对象应转发，以便仍可对其进行修改。
* 常量对象应作为只读对象转发。
* 可移动对象（我们可以从中“窃取”的对象，因为它们即将过期）应转发为可移动对象。
要在没有模板的情况下实现此功能，我们必须编写三种情况的代码。
例如，要将对f()的调用转发到相应的函数g()：

```c++
#include <utility>
#include <iostream>
class X {
…
};
void g (X&) {
    std::cout << "g() for variable\n";
}
void g (X const&) {
    std::cout << "g() for constant\n";
}
void g (X&&) {
    std::cout << "g() for movable object\n";
}

// let f() forward argument val to g():
void f (X& val) {
    g(val); // val is non-const lvalue => calls g(X&)
}
void f (X const& val) {
    g(val); // val is const lvalue => calls g(X const&)
}
void f (X&& val) {
    g(std::move(val)); // val is non-const lvalue => needs
    std::move() to call g(X&&)
}
int main()
{
    X v; // create variable
    X const c; // create constant
    f(v); // f() for nonconstant object calls f(X&) =>calls g(X&)
    f(c); // f() for constant object calls f(X const& => calls g(X const&)
    f(X()); // f() for temporary calls f(X&&) => calls g(X&&)
    f(std::move(v)); // f() for movable variable calls f(X&&) => calls g(X&&)
}
```
这里，我们看到了三种不同的实现，将其参数转发给g():
```c++
void f (X& val) {
    g(val); // val is non-const lvalue => calls g(X&)
}
void f (X const& val) {
    g(val); // val is const lvalue => calls g(X const&)
}
void f (X&& val) {
    g(std::move(val)); // val is non-const lvalue => needs
    std::move() to call g(X&&)
}
```
请注意，对于可移动对象的代码（通过右值引用），与其他代码不同：它需要一个std::move()，因为根据语言规则，移动语义不会传递。
尽管第三个f()中的val声明为右值引用，但当用作表达式时，它的值类别是非常量左值（请参见附录B），并且与第一个f()中的val的行为相同。如果没有std::move()，则将调用g(X&)而不是g(&&)。

如果我们想要在通用代码中组合所有三种情况，则会遇到问题：
```c++
template<typename T>
void f (T val) {
    g(T);
}
```
为了解决这个问题，C++11引入了完美转发参数的特殊规则。实现完美转发的惯用代码模式如下：
```c++
template<typename T>
void f (T&& val) {
    g(std::forward<T>(val)); // perfect forward val to g()
}
```
Note that std::move() has no template parameter and “triggers” move semantics
for the passed argument, while std::forward<>() “forwards” potential move
semantic depending on a passed template argument. 
Don’t assume that T&& for a template parameter T behaves as X&& for a specific
type X. Different rules apply! However, syntactically they look identical:
* X&& for a specific type X declares a parameter to be an rvalue reference. It can only be bound to a movable object (a prvalue, such as a temporary object, and an xvalue, such as an object passed with std::move(); see Appendix B for details). It is always mutable and you can always “steal” its value.
* T&& for a template parameter T declares a forwarding reference (also called universal reference). It can be bound to a mutable, immutable (i.e., const), or movable object. Inside the function definition, the parameter may be mutable, immutable, or refer to a value you can “steal” the internals from.
Note that T must really be the name of a template parameter. Depending on a
template parameter is not sufficient. For a template parameter T, a declaration such
as typename T::iterator&& is just an rvalue reference, not a forwarding
reference.
So, the whole program to perfect forward arguments will look as follows:

请注意，std::move()没有模板参数，并且“触发”传递的参数的移动语义，而std::forward<>()“转发”取决于传递的模板参数的潜在移动语义。
不要假定对于模板参数T，T&&的行为类似于特定类型X的X&&。不同的规则适用！然而，它们在语法上看起来是相同的：
* 对于特定类型X，X&&声明参数为右值引用。它只能绑定到可移动对象（如临时对象的prvalue和xvalue；请参见附录B了解详情）。它始终是可变的，您始终可以“窃取”它的值。
* 对于模板参数T，T&&声明为转发引用（也称为通用引用）。它可以绑定到可变，不可变（即const）或可移动对象。在函数定义内部，该参数可以是可变的、不可变的或引用可以“窃取”内部的值。
请注意，T必须真正是模板参数的名称。仅仅取决于模板参数是不够的。对于模板参数T，例如typename T::iterator&&声明仅仅是一个右值引用，而不是转发引用。
因此，完美转发参数的整个程序如下所示：

```c++
#include <utility>
#include <iostream>
class X {
…
};
void g (X&) {
    std::cout << "g() for variable\n";
}
void g (X const&) {
    std::cout << "g() for constant\n";
}
void g (X&&) {
    std::cout << "g() for movable object\n";
}

// let f() perfect forward argument val to g():
template<typename T>
void f (T&& val) {
    g(std::forward<T>(val)); // call the right g() for any passed argument val
}
int main()
{
    X v; // create variable
    X const c; // create constant
    f(v); // f() for variable calls f(X&) => calls
    g(X&)
    f(c); // f() for constant calls f(X const&) => calls g(X const&)
    f(X()); // f() for temporary calls f(X&&) => calls
    g(X&&)
    f(std::move(v)); // f() for move-enabled variable calls f(X&&) => calls g(X&&)
}
```