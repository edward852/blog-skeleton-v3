---
title: "C++基础"
date: 2018-03-06T16:19:28+08:00
lastmod: 2020-03-21T16:19:28+08:00
draft: false
tags: ["cpp"]
categories: ["language"]
---

本文主要参考 [Learn X in Y minutes](https://learnxinyminutes.com/docs/c%2B%2B/)上的教程介绍C++相关的基础知识。
<!--more-->

相对于C语言，C++具有以下特点：  
-   支持数据的抽象与封装
-   支持面向对象编程
-   支持泛型编程

# 头文件

C语言的标准库头文件可以在C++中使用，方法是加上\"c\"前缀、去掉.h后缀。\
C++标准库的头文件使用时也不需要指定后缀，比如说iostream。

```cpp
#include <cstdio>
#include <iostream>
```

# 命名空间

C++引入了命名空间(namespace)概念，为变量、函数、类等提供隔离的作用域，防止命名冲突。\
通过namespace关键字可以定义命名空间(允许嵌套)。\

```cpp
namespace MyMod {
    namespace MySubMod {
        void foo()
        {
            printf("This is MyMod::MySubMod::foo\n");
        }
    } // end nested namespace MySubMod
} // end namespace MyMod

void foo()
{
    printf("This is global foo\n");
}

int main()
{
    MyMod::MySubMod::foo();

    ::foo(); // using global namespace
    foo();
}
```

通过using
namespace可以引入指定命名空间内所有symbol，比如说std命名空间。\

```cpp
using namespace std;
```

又或者只引入某个symbol：\

```cpp
using namespace std::cout;
```

# 输入/输出
C++使用"流"来输入输出。  
通过`<<`输入数据到流中。  
通过`>>`从流中输出数据。  
cin、cout、和cerr分别对应stdin(标准输入)、stdout(标准输出)和stderr(标准错误)。  

```cpp
#include <iostream> // 引入输入/输出流的头文件uint64_t在32位使用%llu，在64位使用%lu打印
C99还可以使用PRIu64打印uint64

using namespace std; // 引入std命名空间中的符号，包括cin、cout等

int main()
{
   int myInt;

   // 在标准输出中显示
   cout << "Enter your favorite number:\n";
   // 从标准输入取值
   cin >> myInt;
   // 显示“Your favorite number is <myInt>”
   cout << "Your favorite number is " << myInt << endl;
}
```

如果需要实现类似于printf这样控制宽度、精度、对齐、进制(base)的效果，有两种选择：  
- 引入cstdio头文件使用printf函数  
- 通过 [`ios_base类`](http://www.cplusplus.com/reference/ios/ios_base/) 进行设置  

uint64_t打印在32位平台使用%llu，在64位平台使用%lu。  
支持C99标准的话可以使用PRIu64打印。  
```cpp
uint64_t u64 = 100;
printf("u64: %"PRIu64"\n", u64);
```

# 字符串

## string
C++中的字符串是string类，提供比char *更好的封装与功能。  
可以使用+或者+=拼接字符串(通过运算符重载实现)。  
`to_string` 可以将常见类型转换为string类型。  

```cpp
#include <string>

using namespace std; // std::string

string strHello = "Hello";
string strWorld = " World";

cout << strHello + strWorld; // "Hello World"
cout << strHello + " You";
```

## wstring
对于非英文字符串应该使用wstring存储，比如说中文。\
string虽然也可以存储，但实际上是以char作为字符单元，类似计算长度、反转等操作结果并不正确。

```cpp
int main() {

    wcout.imbue(locale("zh_CN.UTF-8"));

    wstring zhHello = L"你好吗？";

    wcout << zhHello << L" 长度： " << to_wstring(zhHello.size()) << endl;
    // 显示 "你好吗？ 长度： 4"

    reverse(zhHello.begin(), zhHello.end());
    wcout << zhHello << endl;
    // 显示 "？吗好你"

    return 0;
}
```

## stringstream
如果有比较多的拼接操作，那么可以考虑使用stringstream提高效率。  
```cpp
std::stringstream ss;
ss << "Your age is " << 28;
cout << ss.str() << endl;
```
注意清除内容不是`.clear()` 而是 `.str("")` 。  

# 引用

C++提供引用来设置变量别名，本质上是特殊的指针，初始化后不能重新赋值。\
使用引用时的语法与原变量相同：不需要通过\*解引用。\
常量引用不允许改变变量的值。\

```cpp
string foo = "I am foo";
string bar = "I am bar";

string& fooRef = foo; // 建立了一个对foo的引用。
fooRef += ". Hi!"; // 通过引用来修改foo的值
cout << fooRef; // "I am foo. Hi!"

const string& barRef = bar; // 建立指向bar的常量引用。
// 和C语言中一样，（指针和引用）声明为常量时，对应的值不能被修改。
barRef += ". Hi!"; // 这是错误的，不能修改一个常量引用的值。
```

对于临时对象的常量引用会使其生命周期延长到当前作用域。

```cpp
string tempObjectFun() { ... }
string retVal = tempObjectFun();

// 第二行代码实际上做了以下操作：
//   - tempObjectFun返回一个string对象
//   - 以返回的对象为参数调用构造函数生成新的string对象
//   - 销毁返回的对象
// 返回的对象就是临时对象。
// 临时对象在函数返回对象的时候创建，在表达式求值结束时销毁(编译器可能会优化)
foo(bar(tempObjectFun()))

// 假设foo和bar都存在，tempObjectFun返回的对象传递给了bar并在foo调用前销毁
// 在表达式求值结束时销毁临时对象的原则有一个例外的情况：
// 当临时对象绑定到常量引用时，该临时对象的生命周期会延长到当前作用域。

void constReferenceTempObjectFun() {
  // constRef绑定了返回的临时对象，该对象在本函数范围内仍然有效。
  const string& constRef = tempObjectFun();
  ...
}
```

另外还有右值引用，因为超出本文范围，建议自行了解。

# 类与面向对象编程

```cpp
#include <iostream>

// 声明一个类。
// 类通常在头文件（.h或.hpp）中声明。
class Dog {
    // 成员变量和成员函数默认情况下是私有（private）的。
    std::string name;
    int weight;

// 在这个标签之后，所有声明都是公有（public）的，
// 直到重新指定“private:”（私有继承）或“protected:”（保护继承）为止
public:

    // 默认的构造器
    Dog();

    // 这里是成员函数声明的一个例子。
    // 可以注意到，我们在此处使用了std::string，而不是using namespace std
    // 语句using namespace绝不应当出现在头文件当中。
    void setName(const std::string& dogsName);

    void setWeight(int dogsWeight);

    // 如果一个函数不对对象的状态进行修改，
    // 应当在声明中加上const。
    // 这样，你就可以对一个以常量方式引用的对象执行该操作。
    // 同时可以注意到，当父类的成员函数需要被子类重写时，
    // 父类中的函数必须被显式声明为虚函数（virtual）。
    // 考虑到性能方面的因素，函数默认情况下不会被声明为虚函数。
    virtual void print() const;

    // 函数也可以在class body内部定义。
    // 这样定义的函数会自动成为内联函数。
    void bark() const { std::cout << name << " barks!\n" }

    // 除了构造器以外，C++还提供了析构器。
    // 当一个对象被删除或者脱离其定义域时，它的析构函数会被调用。
    // 这使得RAII这样的强大范式（参见下文）成为可能。
    // 为了衍生出子类来，基类的析构函数必须定义为虚函数。
    // 如果没有定义为虚函数，那么通过基类引用或者指针销毁时，衍生类的析构函数并不会被调用。
    virtual ~Dog();

}; // 在类的定义之后，要加一个分号

// 类的成员函数通常在.cpp文件中实现。
void Dog::Dog()
{
    std::cout << "A dog has been constructed\n";
}

// 复杂对象（例如字符串）应当以引用的形式传递，
// 对于不需要修改的对象，最好使用常量引用。
void Dog::setName(const std::string& dogsName)
{
    name = dogsName;
}

void Dog::setWeight(int dogsWeight)
{
    weight = dogsWeight;
}

// 虚函数的virtual关键字只需要在声明时使用，不需要在定义时重复
void Dog::print() const
{
    std::cout << "Dog is " << name << " and weighs " << weight << "kg\n";
}

void Dog::~Dog()
{
    std::cout << "Goodbye " << name << "\n";
}

int main() {
    Dog myDog; // 此时显示“A dog has been constructed”
    myDog.setName("Barkley");
    myDog.setWeight(10);
    myDog.print(); // 显示“Dog is Barkley and weighs 10 kg”
    return 0;
} // 显示“Goodbye Barkley”
```

## 初始化与运算符重载

变量可以直接赋初值，也可以在初始化列表中根据外部参数赋值。\
另外C++允许运算符重载，也就是改变运算符（比如说+，\*）对于对象的行为。

```cpp
#include <iostream>
using namespace std;

class Point {
public:
    // 可以以这样的方式为成员变量设置默认值。
    double x = 0;
    double y = 0;

    // 定义一个默认的构造器。
    // 除了将Point初始化为(0, 0)以外，这个函数什么都不做。
    Point() { };

    // 下面使用的语法称为初始化列表，
    // 这是初始化类中成员变量的正确方式。
    Point (double a, double b) :
        x(a),
        y(b)
    { /* 除了初始化成员变量外，什么都不做 */ }

    // 重载 + 运算符
    Point operator+(const Point& rhs) const;

    // 重载 += 运算符
    Point& operator+=(const Point& rhs);

    // 增加 - 和 -= 运算符也是有意义的，但这里不再赘述。
};

Point Point::operator+(const Point& rhs) const
{
    // 创建一个新的点，
    // 其横纵坐标分别为这个点与另一点在对应方向上的坐标之和。
    return Point(x + rhs.x, y + rhs.y);
}

Point& Point::operator+=(const Point& rhs)
{
    x += rhs.x;
    y += rhs.y;
    return *this;
}

int main () {
    Point up (0,1);
    Point right (1,0);
    // 这里使用了Point类型的运算符“+”
    // 调用up（Point类型）的“+”方法，并以right作为函数的参数
    Point result = up + right;
    // 显示“Result is upright (1,1)”
    cout << "Result is upright (" << result.x << ',' << result.y << ")\n";
    return 0;
}
```

## 继承

继承可以复用已有代码，可以分为单继承、多继承、虚拟继承。\
利用多态可以提高灵活性、增加可维护性。\
除了继承外，合理使用组合也是不错的选择。

```cpp
// 这个类继承了Dog类中的公有（public）和保护（protected）内容，
// 私有（private）内容实际上也继承了，但是只能通过public或者protected方法进行访问
class OwnedDog : public Dog {

    void setOwner(const std::string& dogsOwner)

    // 重写OwnedDogs类的print方法。
    // 如果你不熟悉子类多态的话，可以参考这个页面中的概述：
    // http://zh.wikipedia.org/wiki/%E5%AD%90%E7%B1%BB%E5%9E%8B

    // override关键字是可选的，它确保你所重写的是基类中的方法。
    void print() const override;

private:
    std::string owner;
};

// 与此同时，在对应的.cpp文件里：

void OwnedDog::setOwner(const std::string& dogsOwner)
{
    owner = dogsOwner;
}

void OwnedDog::print() const
{
    Dog::print(); // 调用基类Dog中的print方法
    // "Dog is <name> and weights <weight>"

    std::cout << "Dog is owned by " << owner << "\n";
    // "Dog is owned by <owner>"
}
```

## 静态成员变量
```cpp
// .h
class IpUtils
{
    ...
    static const size_t IPV6_ADDR_SIZE = 16;
    static const size_t IPV4_ADDR_SIZE = 4;
    ...
};

// .cpp
const size_t IpUtils::IPV6_ADDR_SIZE;
const size_t IpUtils::IPV4_ADDR_SIZE;
```

# 模板

C++模板主要用于泛型编程。

```cpp
template<class T>
class Box {
public:
    // In this class, T can be used as any other type.
    void insert(const T&) { ... }
};

// During compilation, the compiler actually generates copies of each template
// with parameters substituted, so the full definition of the class must be
// present at each invocation. This is why you will see template classes defined
// entirely in header files.

// To instantiate a template class on the stack:
Box<int> intBox;

// and you can use it as you would expect:
intBox.insert(123);

// You can, of course, nest templates:
Box<Box<int> > boxOfBox;
boxOfBox.insert(intBox);

// Until C++11, you had to place a space between the two '>'s, otherwise '>>'
// would be parsed as the right shift operator.

// You will sometimes see
//   template<typename T>
// instead. The 'class' keyword and 'typename' keywords are _mostly_
// interchangeable in this case. For the full explanation, see
//   http://en.wikipedia.org/wiki/Typename
// (yes, that keyword has its own Wikipedia page).

// Similarly, a template function:
template<class T>
void barkThreeTimes(const T& input)
{
    input.bark();
    input.bark();
    input.bark();
}

// Notice that nothing is specified about the type parameters here. The compiler
// will generate and then type-check every invocation of the template, so the
// above function works with any type 'T' that has a const 'bark' method!

Dog fluffy;
fluffy.setName("Fluffy")
barkThreeTimes(fluffy); // Prints "Fluffy barks" three times.

// Template parameters don't have to be classes:
template<int Y>
void printMessage() {
  cout << "Learn C++ in " << Y << " minutes!" << endl;
}

// And you can explicitly specialize templates for more efficient code. Of
// course, most real-world uses of specialization are not as trivial as this.
// Note that you still need to declare the function (or class) as a template
// even if you explicitly specified all parameters.
template<>
void printMessage<10>() {
  cout << "Learn C++ faster in only 10 minutes!" << endl;
}

printMessage<20>();  // Prints "Learn C++ in 20 minutes!"
printMessage<10>();  // Prints "Learn C++ faster in only 10 minutes!"
```

# 异常处理
## try catch
```cpp
// 在try代码块中拋出的异常可以被随后的catch捕获。
try {
    // 不要用 new关键字在堆上为异常分配空间。
    throw std::exception("A problem occurred");
}
// 如果拋出的异常是一个对象，可以用常量引用来捕获它
catch (const std::exception& ex)
{
  std::cout << ex.what();
// 捕获尚未被catch处理的所有错误
}
catch (...)
{
    std::cout << "Unknown exception caught";
    throw; // 重新拋出异常
}

```
try catch不能捕获SEGV错误，需要注册相应信号处理器。  

## goto 或 do while(0)
C风格处理出错情况。  
```cpp
do
{
    if (出错)
    {
        设置flag;
        break;
    }
} while (0);

if (flag)
{
    错误处理;
}
```

# RAII

RAII指的是"资源获取就是初始化"（Resource Allocation Is
Initialization）。\
RAII是C++中最强大的编程范式之一。\
简单来说，就是在构造函数中申请资源，在析构函数中释放资源。

```cpp
void doSomethingWithAFile(const char* filename)
{
    // 首先，让我们假设一切都会顺利进行。

    FILE* fh = fopen(filename, "r"); // 以只读模式打开文件

    doSomethingWithTheFile(fh);
    doSomethingElseWithIt(fh);

    fclose(fh); // 关闭文件句柄
}

// 不幸的是，随着错误处理机制的引入，事情会变得复杂。
// 假设fopen函数有可能执行失败，
// 而doSomethingWithTheFile和doSomethingElseWithIt会在失败时返回错误代码。
// （虽然异常是C++中处理错误的推荐方式，
// 但是某些程序员，尤其是有C语言背景的，并不认可异常捕获机制的作用）。
// 现在，我们必须检查每个函数调用是否成功执行，并在问题发生的时候关闭文件句柄。
bool doSomethingWithAFile(const char* filename)
{
    FILE* fh = fopen(filename, "r"); // 以只读模式打开文件
    if (fh == nullptr) // 当执行失败是，返回的指针是nullptr
        return false; // 向调用者汇报错误

    // 假设每个函数会在执行失败时返回false
    if (!doSomethingWithTheFile(fh)) {
        fclose(fh); // 关闭文件句柄，避免造成内存泄漏。
        return false; // 反馈错误
    }
    if (!doSomethingElseWithIt(fh)) {
        fclose(fh); // 关闭文件句柄
        return false; // 反馈错误
    }

    fclose(fh); // 关闭文件句柄
    return true; // 指示函数已成功执行
}

// C语言的程序员通常会借助goto语句简化上面的代码：
bool doSomethingWithAFile(const char* filename)
{
    FILE* fh = fopen(filename, "r");
    if (fh == nullptr)
        return false;

    if (!doSomethingWithTheFile(fh))
        goto failure;

    if (!doSomethingElseWithIt(fh))
        goto failure;

    fclose(fh); // 关闭文件
    return true; // 执行成功

failure:
    fclose(fh);
    return false; // 反馈错误
}

// 如果用异常捕获机制来指示错误的话，
// 代码会变得清晰一些，但是仍然有优化的余地。
void doSomethingWithAFile(const char* filename)
{
    FILE* fh = fopen(filename, "r"); // 以只读模式打开文件
    if (fh == nullptr)
        throw std::exception("Could not open the file.");

    try {
        doSomethingWithTheFile(fh);
        doSomethingElseWithIt(fh);
    }
    catch (...) {
        fclose(fh); // 保证出错的时候文件被正确关闭
        throw; // 之后，重新抛出这个异常
    }

    fclose(fh); // 关闭文件
    // 所有工作顺利完成
}

// 相比之下，使用C++中的文件流类（fstream）时，
// fstream会利用自己的析构器来关闭文件句柄。
// 只要离开了某一对象的定义域，它的析构函数就会被自动调用。
void doSomethingWithAFile(const std::string& filename)
{
    // ifstream是输入文件流（input file stream）的简称
    std::ifstream fh(filename); // 打开一个文件

    // 对文件进行一些操作
    doSomethingWithTheFile(fh);
    doSomethingElseWithIt(fh);

} // 文件已经被析构器自动关闭

```


# 单例模式
```cpp
// works for c++11(or later standard) only!
class SingletonCfg {
public:
    static SingletonCfg& getInst()
    {
        // c++11(or later standard) ensure thread-safe static local initialization
        static SingletonCfg s;

        return s;
    }
    void display();

private:
    SingletonCfg();
    SingletonCfg(const SingletonCfg &);             // do not implement
    SingletonCfg& operator=(const SingletonCfg &);  // do not implement

};
```

# 杂项Misc

## 函数重载

C++支持函数重载，也就是可以定义一组名称相同而参数不同的函数。\
实现上类似于根据函数名+参数类型生成新的函数名。

```cpp
void print(char *myString)
{
    printf("String %s\n", myString);
}

void print(int myInt)
{
    printf("My int is %d", myInt);
}

int main()
{
    print("Hello");
    print(15);
}
```

## 参数默认值设置
可以为函数的参数指定默认值，在调用者没有提供相应参数时按照默认值调用。  
具有默认值的参数必须放在所有的常规参数之后。  
一般在声明的时候指定默认值。  

```cpp
void doSomethingWithInts(int a = 1, int b = 4)
{

}
void invalidDeclaration(int a = 1, int b) // 这是错误的！
{

}

int main()
{
    doSomethingWithInts();      // a = 1,  b = 4
    doSomethingWithInts(20);    // a = 20, b = 4
    doSomethingWithInts(20, 5); // a = 20, b = 5
}
```

## 空指针

在C++中使用nullptr代替C语言中的NULL。

## 严格原型
```cpp
// C++的函数原型与函数定义是严格匹配的
void func(); // 这个函数不能接受任何参数

// 而在C语言中
void func(); // 这个函数能接受任意数量的参数
```
## extern "C"
可以实现C与C++混合编程。  

C++支持函数重载，底层实现函数名带有参数信息，而C不支持重载。  
加上 `extern "C"` 编译器才能把C与C++代码链接在一起。  
```cpp
#ifdef __cplusplus
extern "C" {
#endif

// 正式定义。。。

#ifdef __cplusplus
}
#endif
```

# STL
## 容器
具体参考 [STL容器](/post/stl容器) 。  

## 算法
具体参考 [STL算法](/post/stl算法/) 。  

# 参考链接
- [C++ Reference](http://www.cplusplus.com/reference/)
