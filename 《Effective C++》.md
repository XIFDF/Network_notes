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
**如果关键字const出现在星号左边，表示被指物体是常量；如果出现在星号右边，表示指针自身是常量；如果出现在星号两边，表示被指物和指针两者都是常量。**<br>
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
## 条款04：确定对象被使用前已先被初始化
### Make sure that object are initialized before they're used.
* 区分赋值和初始化
```c++
class PhoneNumber {...};
class ABEntry {
public:
    ABEntry(const std::string& name, const std::string& address,
            cosnt std::list<PhoneNumber>& phones);
private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
};
ABEntry::ABEntry(const std::string& name, const std::string& address,
            cosnt std::list<PhoneNumber>& phones)
{
    theName = name;            //这些都是赋值
    theAddress = address;      //而非初始化
    thePhones = phones;
    numTimesConsulted = 0;
}
/*更佳写法，是用成员初值列替换赋值动作*/
ABEntry::ABEntry(const std::string& name, const std::string& address,
            cosnt std::list<PhoneNumber>& phones)
            :theName(name),
             theAddress(address),
             thePhones(phones),
             numTimesConsulted(0)
{ }
```
* 当某编译单元内的某个non-local static对象的初始化动作使用了另一编译单元内的某个non-local static 对象，它所用到的这个对象可能尚未被初始化，因为C++对“定义于不同编译单元内的non-local static对象”的初始化次序并无明确定义。
注：编译单元是指产出单一目标文件的那些源码。基本上它是单一源码文件加上其所含入的头文件。
```c++
class FIleSystem {      //来自你的程序库
public:
    ...
    std::size_t numDisks() const;
    ...
};
extern FileSystem tfs;  //预备给客户使用的对象
```
```c++
class Directory {       //由程序库和客户建立
public；
    Directory( parames );
    ...
};
Directory::Directory( params )
{
    ...
    std:size_t disks = tfs.numDisks(); //使用tfs对象
}

/*此时无法确定tfs会在tempDir之前先被初始化*/
Directory tempDir( params );
```
这类问题可以用设计模式中单例模式的一种实现手法来解决：
```c++
class FileSystem { ... }
FileSystem& tfs()
{
    static FileSystem fs;
    return fs;
}
class Directory { ... }
Directory::Directroy( params )
{
    ...
    std::size_t disks = tfs().numDisks();
    ...
}
Directory& tempDir()
{
    static Directory td;
    return td;
}
```
总结：
* 为内置对象进行手工初始化，因为C++不保证初始化它们。
* 构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作。初值列列出的成员变量，其排列次序应该和它们在class中的声明次序相同。
* 为免除“跨编译单元之初始化次序”问题，请以local static 对象替换non-local static对象。

## 条款05：了解C++默默编写并调用哪些函数
### Know what functions C++ silently writes and calls.
* 编译器可以暗自为class创建default构造函数、copy构造函数、copy assignment操作符，以及析构函数。
如果你声明了一个构造函数，编译器于是不再为它创建default构造函数。

## 条款06：若不想使用编译器自动生成的函数，就该明确拒绝
### Explicitly disallow the use of compiler-generated functions you do not want.
* 为驳回编译器自动(暗自)提供的机能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种做法。
```c++
class HomeForSale { //销售的房子不应可被复制
public:
    ...
private:
    ...
    HomeForSale(const HomeForSale&);    //只有声明，无需实现，故省略了参数名称
    HomeForSale& operator=(const HomeForSale&);
}
```
另外一种做法：
```c++
class Uncopyable {
protected:
    Uncopyable() {}         //允许derived(派生)对象构造和析构
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);      //但阻止copying
    Uncopyable& operator=(const Uncopyable&);
}；
```
```c++
class HomeForSale: private Uncopyable {
    ...
};
```

## 条款07：为多态基类声明virtual析构函数
### Declare destructors virtual in polymorphic base classes.
* polymorphic(带多态性质的)base classes应该声明一个virtual析构函数。如果class带有任何virtual函数，它就应该拥有一个virtual析构函数。
* Classes的设计目的如果不是作为base classes使用，或不是为了具备多态性，就不该声明virtual函数。
* 只有当class内含至少一个virtual函数，才为它声明virtual析构函数。

## 条款08：别让异常逃离析构函数
### Prevent exceptions from leaving destructors.
* 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们(不传播)或结束程序。
```c++
/*如果close抛出异常就结束程序。通常通过调用abort完成*/
DBConn::~DBConn()
{
    try { db.close(); }
    catch (...) {
        制作运转记录，记下对close的调用失败；
        std::abort();
    }
}
/////////////////////////////////////////
/*吞下因调用close而发生的异常*/
DBConn::~DBConn()
{
    try { db.close(); }
    catch (...) {
        制作运转记录，记下对close的调用失败；
    }
}
```
* 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数(而非在析构函数中)执行该操作。

## 条款09：绝不在构造和析构过程中调用virtual函数
### Never call virtual function during construction or destruction.
* 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class。

## 条款10：令operator=返回一个reference to *this
### Have assignment operators return a reference to *this.

## 条款11：在operator=中处理“自我赋值”
### Handle assignment to self in operator=.
对象可能会赋值给自己。
```c++
/* 潜在的自我赋值 */
a[i] = a[j];
*px = *py;
```
```c++
class Bitmap { ... }
class Widget {
    ...
private:
    Bitmap *pb;
};

/* 不安全版本 */
Widget& Widget::operator=(const Widget& rhs)
{
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

## 条款12：复制对象时勿忘其每一个成分
### Copy all parts of an Object.
* Copying 函数应该确保复制“对象内的所有成员变量”及“所有base class成分”。
* 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并有两个copying函数共同调用。


## 条款13：以对象管理资源
### Use object to manage resources.
* 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。
* 两个常被使用的RAII classes分别是tr1::shared_ptr和auto_ptr。前者通常是较佳选择，因为其copy行为比较直观。若选择auto_ptr，复制动作会使它(被复制物)指向null。

## 条款14：在资源管理类中小心copying行为
### Think carefully about copying behavior in resource-managing classes.
* 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
* 普遍而常见的RAII class copying行为是：抑制copying、施行引用计数法（reference）。不过其他行为也都可能被实现。


