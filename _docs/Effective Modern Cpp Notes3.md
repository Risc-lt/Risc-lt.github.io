---
title: "Effective Modern C++ Notes (3)"
excerpt: "C++11使用统一初始化（*uniform initialization*）来整合这些混乱且不适于所有情景的初始化语法，所谓统一初始化是指在任何涉及初始化的地方都使用单一的初始化语法。 "
collection: docs
---

## CHAPTER 3 Moving to Modern C++

### Item 7: Distinguish between `()` and `{}` when creating objects

#### 统一初始化（*uniform initialization*）

C++11使用统一初始化（*uniform initialization*）来整合这些混乱且不适于所有情景的初始化语法，所谓统一初始化是指在任何涉及初始化的地方都使用单一的初始化语法。 它基于花括号，出于这个原因我更喜欢称之为括号初始化。统一初始化是一个概念上的东西，而括号初始化是一个具体语法结构。

括号初始化让你可以表达以前表达不出的东西。使用花括号，创建并指定一个容器的初始元素变得很容易：

```cpp
std::vector<int> v{ 1, 3, 5 };  //v初始内容为1,3,5
```

括号初始化也能被用于为非静态数据成员指定默认初始值。C++11允许"="初始化不加花括号也拥有这种能力：

```cpp
class Widget{
    …

private:
    int x{ 0 };                 //没问题，x初始值为0
    int y = 0;                  //也可以
    int z(0);                   //错误！
}
```

另一方面，不可拷贝的对象（例如`std::atomic`——见[Item40](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item40.html)）可以使用花括号初始化或者圆括号初始化，但是不能使用"="初始化：

```cpp
std::atomic<int> ai1{ 0 };      //没问题
std::atomic<int> ai2(0);        //没问题
std::atomic<int> ai3 = 0;       //错误！
```

因此我们很容易理解为什么括号初始化又叫统一初始化，在C++中这三种方式都被看做是初始化表达式，但是只有花括号任何地方都能被使用。

#### 不允许隐式的变窄转换（*narrowing conversion*）

括号表达式还有一个少见的特性，即它不允许内置类型间隐式的变窄转换（*narrowing conversion*）。如果一个使用了括号初始化的表达式的值，不能保证由被初始化的对象的类型来表示，代码就不会通过编译：

```cpp
double x, y, z;

int sum1{ x + y + z };          //错误！double的和可能不能表示为int
```

使用圆括号和"="的初始化不检查是否转换为变窄转换，因为由于历史遗留问题它们必须要兼容老旧代码：

```cpp
int sum2(x + y +z);             //可以（表达式的值被截为int）

int sum3 = x + y + z;           //同上
```

#### 免疫C++的解析问题most vexing parse

另一个值得注意的特性是括号表达式对于C++最令人头疼的解析问题有天生的免疫性。（译注：所谓最令人头疼的解析即*most vexing parse*，更多信息请参见https://en.wikipedia.org/wiki/Most_vexing_parse。）C++规定任何*可以被解析*为一个声明的东西*必须被解析*为声明。这个规则的副作用是让很多程序员备受折磨：他们可能想创建一个使用默认构造函数构造的对象，却不小心变成了函数声明。问题的根源是如果你调用带参构造函数，你可以这样做：

```cpp
Widget w1(10);                  //使用实参10调用Widget的一个构造函数
```

但是如果你尝试使用相似的语法调用`Widget`无参构造函数，它就会变成函数声明：

```cpp
Widget w2();                    //最令人头疼的解析！声明一个函数w2，返回Widget
```

由于函数声明中形参列表不能带花括号，所以使用花括号初始化表明你想调用默认构造函数构造对象就没有问题：

```cpp
Widget w3{};                    //调用没有参数的构造函数构造对象
```

#### 括号初始化的一些缺点

括号初始化的缺点是有时它有一些令人惊讶的行为。这些行为使得括号初始化、`std::initializer_list`和构造函数参与重载决议时本来就不清不楚的暧昧关系进一步混乱。把它们放到一起会让看起来应该左转的代码右转。举个例子，[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)解释了当`auto`声明的变量使用花括号初始化，变量类型会被推导为`std::initializer_list`，但是使用相同内容的其他初始化方式会产生更符合直觉的结果。**所以，你越喜欢用`auto`，你就越不能用括号初始化。**

在构造函数调用中，只要不包含`std::initializer_list`形参，那么花括号初始化和圆括号初始化都会产生一样的结果：

```cpp
class Widget { 
public:  
    Widget(int i, bool b);      //构造函数未声明
    Widget(int i, double d);    //std::initializer_list这个形参 
    …
};
Widget w1(10, true);            //调用第一个构造函数
Widget w2{10, true};            //也调用第一个构造函数
Widget w3(10, 5.0);             //调用第二个构造函数
Widget w4{10, 5.0};             //也调用第二个构造函数
```

然而，如果有一个或者多个构造函数的声明包含一个`std::initializer_list`形参，那么使用括号初始化语法的调用更倾向于选择带`std::initializer_list`的那个构造函数。如果编译器遇到一个括号初始化并且有一个带std::initializer_list的构造函数，那么它一定会选择该构造函数。如果上面的`Widget`类有一个`std::initializer_list<long double>`作为参数的构造函数，就像这样：

```cpp
class Widget { 
public:  
    Widget(int i, bool b);      //同上
    Widget(int i, double d);    //同上
    Widget(std::initializer_list<long double> il);      //新添加的
    …
}; 
```

`w2`和`w4`将会使用新添加的构造函数，即使另一个非`std::initializer_list`构造函数和实参更匹配：

```cpp
Widget w1(10, true);    //使用圆括号初始化，同之前一样
                        //调用第一个构造函数

Widget w2{10, true};    //使用花括号初始化，但是现在
                        //调用带std::initializer_list的构造函数
                        //(10 和 true 转化为long double)

Widget w3(10, 5.0);     //使用圆括号初始化，同之前一样
                        //调用第二个构造函数 

Widget w4{10, 5.0};     //使用花括号初始化，但是现在
                        //调用带std::initializer_list的构造函数
                        //(10 和 5.0 转化为long double)
```

甚至普通构造函数和移动构造函数都会被带`std::initializer_list`的构造函数劫持：

```cpp
class Widget { 
public:  
    Widget(int i, bool b);                              //同之前一样
    Widget(int i, double d);                            //同之前一样
    Widget(std::initializer_list<long double> il);      //同之前一样
    operator float() const;                             //转换为float
    …
};

Widget w5(w4);                  //使用圆括号，调用拷贝构造函数

Widget w6{w4};                  //使用花括号，调用std::initializer_list构造
                                //函数（w4转换为float，float转换为double）

Widget w7(std::move(w4));       //使用圆括号，调用移动构造函数

Widget w8{std::move(w4)};       //使用花括号，调用std::initializer_list构造
                                //函数（与w6相同原因）
```

编译器一遇到括号初始化就选择带`std::initializer_list`的构造函数的决心是如此强烈，以至于就算带`std::initializer_list`的构造函数不能被调用，它也会硬选。

```cpp
class Widget { 
public: 
    Widget(int i, bool b);                      //同之前一样
    Widget(int i, double d);                    //同之前一样
    Widget(std::initializer_list<bool> il);     //现在元素类型为bool
    …                                           //没有隐式转换函数
};

Widget w{10, 5.0};              //错误！要求变窄转换
```

这里，编译器会直接忽略前面两个构造函数（其中第二个构造函数是所有实参类型的最佳匹配），然后尝试调用`std::initializer_list<bool>`构造函数。调用这个函数将会把`int(10)`和`double(5.0)`转换为`bool`，由于会产生变窄转换（`bool`不能准确表示其中任何一个值），括号初始化拒绝变窄转换，所以这个调用无效，代码无法通过编译。

如果你**想**用空`std::initializer`来调用`std::initializer_list`构造函数，你就得创建一个空花括号作为函数实参——把空花括号放在圆括号或者另一个花括号内来界定你想传递的东西。

```cpp
Widget w4({});                  //使用空花括号列表调用std::initializer_list构造函数
Widget w5{{}};                  //同上
```

**请记住：**

- 花括号初始化是最广泛使用的初始化语法，它防止变窄转换，并且对于C++最令人头疼的解析有天生的免疫性
- 在构造函数重载决议中，编译器会尽最大努力将括号初始化与`std::initializer_list`参数匹配，即便其他构造函数看起来是更好的选择
- 对于数值类型的`std::vector`来说使用花括号初始化和圆括号初始化会造成巨大的不同
- 在模板类选择使用圆括号初始化或使用花括号初始化创建对象是一个挑战。

### Item 8: Prefer `nullptr` to `0` and `NULL`

#### 0和NULL在重载函数中的问题

如果C++发现在当前上下文只能使用指针，它会很不情愿的把`0`解释为指针，但是那是最后的退路。一般来说C++的解析策略是把`0`看做`int`而不是指针。

实际上，`NULL`也是这样的。但在`NULL`的实现细节有些不确定因素，因为实现被允许给`NULL`一个除了`int`之外的整型类型（比如`long`）。这不常见，但也算不上问题所在。这里的问题不是`NULL`没有一个确定的类型，而是`0`和`NULL`都不是指针类型。

在C++98中，对指针类型和整型进行重载意味着可能导致奇怪的事情。如果给下面的重载函数传递`0`或`NULL`，它们绝不会调用指针版本的重载函数：

```cpp
void f(int);        //三个f的重载函数
void f(bool);
void f(void*);

f(0);               //调用f(int)而不是f(void*)

f(NULL);            //可能不会被编译，一般来说调用f(int)，
                    //绝对不会调用f(void*)
```

而`f(NULL)`的不确定行为是由`NULL`的实现不同造成的。如果`NULL`被定义为`0L`（指的是`0`为`long`类型），这个调用就具有二义性，因为从`long`到`int`的转换或从`long`到`bool`的转换或`0L`到`void*`的转换都同样好。有趣的是源代码**表现出**的意思（“我使用空指针`NULL`调用`f`”）和**实际表达出**的意思（“我是用整型数据而不是空指针调用`f`”）是相矛盾的。

#### nullptr的唯一性

`nullptr`的优点是它不是整型。老实说它也不是一个指针类型，但是你可以把它认为是**所有**类型的指针。`nullptr`的真正类型是`std::nullptr_t`，在一个完美的循环定义以后，`std::nullptr_t`又被定义为`nullptr`。`std::nullptr_t`可以隐式转换为指向任何内置类型的指针，这也是为什么`nullptr`表现得像所有类型的指针。

使用`nullptr`调用`f`将会调用`void*`版本的重载函数，因为`nullptr`不能被视作任何整型：

```cpp
f(nullptr);         //调用重载函数f的f(void*)版本
```

使用`nullptr`代替`0`和`NULL`可以避开了那些令人奇怪的函数重载决议，这不是它的唯一优势。它也可以使代码表意明确，尤其是当涉及到与`auto`声明的变量一起使用时。举个例子，假如你在一个代码库中遇到了这样的代码：

```cpp
auto result = findRecord( /* arguments */ );
if (result == 0) {
    …
} 
```

如果你不知道`findRecord`返回了什么（或者不能轻易的找出），那么你就不太清楚到底`result`是一个指针类型还是一个整型。毕竟，`0`（用来测试`result`的值的那个）也可以像我们之前讨论的那样被解析。但是换一种假设如果你看到这样的代码：

```cpp
auto result = findRecord( /* arguments */ );

if (result == nullptr) {  
    …
}
```

这就没有任何歧义：`result`的结果一定是指针类型。

**记住**

- 优先考虑`nullptr`而非`0`和`NULL`
- 避免重载指针和整型

### Item 9: Prefer alias declarations to `typedef`s

这里，在说它们之前我想提醒一下很多人都发现当声明一个函数指针时别名声明更容易理解：

```cpp
//FP是一个指向函数的指针的同义词，它指向的函数带有
//int和const std::string&形参，不返回任何东西
typedef void (*FP)(int, const std::string&);    //typedef

//含义同上
using FP = void (*)(int, const std::string&);   //别名声明
```

当然，两个结构都不是非常让人满意，没有人喜欢花大量的时间处理函数指针类型的别名（译注：指`FP`），所以至少在这里，没有一个吸引人的理由让你觉得别名声明比`typedef`好。

#### 别名声明可以被模板化（alias templates）

不过有一个地方使用别名声明吸引人的理由是存在的：模板。特别地，别名声明可以被模板化但是`typedef`不能。这使得C++11程序员可以很直接的表达一些C++98中只能把`typedef`嵌套进模板化的`struct`才能表达的东西。考虑一个链表的别名，链表使用自定义的内存分配器，`MyAlloc`。使用别名模板，这真是太容易了：

```cpp
template<typename T>                            //MyAllocList<T>是
using MyAllocList = std::list<T, MyAlloc<T>>;   //std::list<T, MyAlloc<T>>
                                                //的同义词

MyAllocList<Widget> lw;                         //用户代码
```

使用`typedef`，你就只能从头开始：

```cpp
template<typename T>                            //MyAllocList<T>是
struct MyAllocList {                            //std::list<T, MyAlloc<T>>
    typedef std::list<T, MyAlloc<T>> type;      //的同义词  
};

MyAllocList<Widget>::type lw;                   //用户代码
```

更糟糕的是，如果你想使用在一个模板内使用`typedef`声明一个链表对象，而这个对象又使用了模板形参，你就不得不在`typedef`前面加上`typename`：

```cpp
template<typename T>
class Widget {                              //Widget<T>含有一个
private:                                    //MyAllocLIst<T>对象
    typename MyAllocList<T>::type list;     //作为数据成员
    …
}; 
```

这里`MyAllocList<T>::type`使用了一个类型，这个类型依赖于模板参数`T`。因此`MyAllocList<T>::type`是一个依赖类型（*dependent type*），在C++很多讨人喜欢的规则中的一个提到必须要在依赖类型名前加上`typename`。

如果使用别名声明定义一个`MyAllocList`，就不需要使用`typename`（同时省略麻烦的“`::type`”后缀）：

```cpp
template<typename T> 
using MyAllocList = std::list<T, MyAlloc<T>>;   //同之前一样

template<typename T> 
class Widget {
private:
    MyAllocList<T> list;                        //没有“typename”
    …                                           //没有“::type”
};
```

对你来说，`MyAllocList<T>`（使用了模板别名声明的版本）可能看起来和`MyAllocList<T>::type`（使用`typedef`的版本）一样都应该依赖模板参数`T`，但是你不是编译器。当编译器处理`Widget`模板时遇到`MyAllocList<T>`（使用模板别名声明的版本），它们知道`MyAllocList<T>`是一个类型名，因为`MyAllocList`是一个别名模板：它**一定**是一个类型名。因此`MyAllocList<T>`就是一个**非依赖类型**（*non-dependent type*），就不需要也不允许使用`typename`修饰符。

当编译器在`Widget`的模板中看到`MyAllocList<T>::type`（使用`typedef`的版本），它不能确定那是一个类型的名称。因为可能存在一个`MyAllocList`的它们没见到的特化版本，那个版本的`MyAllocList<T>::type`指代了一种不是类型的东西。那听起来很不可思议，但不要责备编译器穷尽考虑所有可能。因为人确实能写出这样的代码。

#### *type traits*转换的别名声明版本

C++11在*type traits*（类型特性）中给了你一系列工具去实现类型转换，如果要使用这些模板请包含头文件`<type_traits>`。里面有许许多多*type traits*，也不全是类型转换的工具，也包含一些可预测接口的工具。给一个你想施加转换的类型`T`，结果类型就是`std::`transformation`<T>::type`，比如：

```cpp
std::remove_const<T>::type          //从const T中产出T
std::remove_reference<T>::type      //从T&和T&&中产出T
std::add_lvalue_reference<T>::type  //从T中产出T&
```

注释仅仅简单的总结了类型转换做了什么，所以不要太随便的使用。在你的项目使用它们之前，你最好看看它们的详细说明书。

尽管写了一些，但我这里不是想给你一个关于*type traits*使用的教程。注意类型转换尾部的`::type`。如果你在一个模板内部将他们施加到类型形参上（实际代码中你也总是这么用），你也需要在它们前面加上`typename`。至于为什么要这么做是因为这些C++11的*type traits*是通过在`struct`内嵌套`typedef`来实现的。是的，它们使用类型同义词（译注：根据上下文指的是使用`typedef`的做法）技术实现，而正如我之前所说这比别名声明要差。

关于为什么这么实现是有历史原因的，但是我们跳过它（我认为太无聊了），因为标准委员会没有及时认识到别名声明是更好的选择，所以直到C++14它们才提供了使用别名声明的版本。这些别名声明有一个通用形式：对于C++11的类型转换`std::`transformation`<T>::type`在C++14中变成了`std::`transformation`_t`。举个例子或许更容易理解：

```cpp
std::remove_const<T>::type          //C++11: const T → T 
std::remove_const_t<T>              //C++14 等价形式

std::remove_reference<T>::type      //C++11: T&/T&& → T 
std::remove_reference_t<T>          //C++14 等价形式

std::add_lvalue_reference<T>::type  //C++11: T → T& 
std::add_lvalue_reference_t<T>      //C++14 等价形式
```

C++11的的形式在C++14中也有效，但是我不能理解为什么你要去用它们。就算你没办法使用C++14，使用别名模板也是小儿科。只需要C++11的语言特性，甚至每个小孩都能仿写，对吧？如果你有一份C++14标准，就更简单了，只需要复制粘贴：

```cpp
template <class T> 
using remove_const_t = typename remove_const<T>::type;

template <class T> 
using remove_reference_t = typename remove_reference<T>::type;

template <class T> 
using add_lvalue_reference_t =
  typename add_lvalue_reference<T>::type; 
```

看见了吧？不能再简单了。

**请记住：**

- `typedef`不支持模板化，但是别名声明支持。
- 别名模板避免了使用“`::type`”后缀，而且在模板中使用`typedef`还需要在前面加上`typename`
- C++14提供了C++11所有*type traits*转换的别名声明版本

### Item 10: Prefer scoped `enum`s to unscoped `enum`s

#### 枚举类（enum class）减少命名空间污染

通常来说，在花括号中声明一个名字会限制它的作用域在花括号之内。但这对于C++98风格的`enum`中声明的枚举名（译注：*enumerator*，连同下文“枚举名”都指*enumerator*）是不成立的。这些枚举名的名字（译注：*enumerator* names，连同下文“名字”都指names）属于包含这个`enum`的作用域，这意味着作用域内不能含有相同名字的其他东西：

```cpp
enum Color { black, white, red };   //black, white, red在
                                    //Color所在的作用域
auto white = false;                 //错误! white早已在这个作用
                                    //域中声明
```

这些枚举名的名字泄漏进它们所被定义的`enum`在的那个作用域，这个事实有一个官方的术语：未限域枚举(*unscoped `enum`*)。在C++11中它们有一个相似物，限域枚举(*scoped `enum`*)，它不会导致枚举名泄漏：

```cpp
enum class Color { black, white, red }; //black, white, red
                                        //限制在Color域内
auto white = false;                     //没问题，域内没有其他“white”

Color c = white;                        //错误，域中没有枚举名叫white

Color c = Color::white;                 //没问题
auto c = Color::white;                  //也没问题（也符合Item5的建议）
```

因为限域`enum`是通过“`enum class`”声明，所以它们有时候也被称为枚举类(*`enum` classes*)。

#### 枚举类的元素名是不能隐式转换的强类型

其实限域`enum`还有第二个吸引人的优点：在它的作用域中，枚举名是强类型。未限域`enum`中的枚举名会隐式转换为整型（现在，也可以转换为浮点类型）。因此下面这种歪曲语义的做法也是完全有效的：

```cpp
enum Color { black, white, red };       //未限域enum

std::vector<std::size_t>                //func返回x的质因子
  primeFactors(std::size_t x);

Color c = red;
…

if (c < 14.5) {                         // Color与double比较 (!)
    auto factors =                      // 计算一个Color的质因子(!)
      primeFactors(c);
    …
}
```

在`enum`后面写一个`class`就可以将非限域`enum`转换为限域`enum`，接下来就是完全不同的故事展开了。现在不存在任何隐式转换可以将限域`enum`中的枚举名转化为任何其他类型：

```cpp
enum class Color { black, white, red }; //Color现在是限域enum

Color c = Color::red;                   //和之前一样，只是
...                                     //多了一个域修饰符

if (c < 14.5) {                         //错误！不能比较
                                        //Color和double
    auto factors =                      //错误！不能向参数为std::size_t
      primeFactors(c);                  //的函数传递Color参数
    …
}
```

如果你真的很想执行`Color`到其他类型的转换，和平常一样，使用正确的类型转换运算符扭曲类型系统：

```cpp
if (static_cast<double>(c) < 14.5) {    //奇怪的代码，
                                        //但是有效
    auto factors =                                  //有问题，但是
      primeFactors(static_cast<std::size_t>(c));    //能通过编译
    …
}
```

#### 限域`enum`可以被前置声明

不能前置声明`enum`也是有缺点的。最大的缺点莫过于它可能增加编译依赖。再次考虑`Status` `enum`：

```cpp
enum Status { good = 0,
              failed = 1,
              incomplete = 100,
              corrupt = 200,
              indeterminate = 0xFFFFFFFF
            };
```

这种`enum`很有可能用于整个系统，因此系统中每个包含这个头文件的组件都会依赖它。如果引入一个新状态值`audited`，

```cpp
enum Status { good = 0,
              failed = 1,
              incomplete = 100,
              corrupt = 200,
              audited = 500,
              indeterminate = 0xFFFFFFFF
            };
```

那么可能整个系统都得重新编译，即使只有一个子系统——或者只有一个函数——使用了新添加的枚举名。这是大家都**不希望**看到的。C++11中的前置声明`enum`s可以解决这个问题。比如这里有一个完全有效的限域`enum`声明和一个以该限域`enum`作为形参的函数声明：

```cpp
enum class Status;                  //前置声明
void continueProcessing(Status s);  //使用前置声明enum
```

即使`Status`的定义发生改变，包含这些声明的头文件也不需要重新编译。而且如果`Status`有改动（比如添加一个`audited`枚举名），`continueProcessing`的行为不受影响（比如因为`continueProcessing`没有使用这个新添加的`audited`），`continueProcessing`也不需要重新编译。

但是如果编译器在使用它之前需要知晓该`enum`的大小，该怎么声明才能让C++11做到C++98不能做到的事情呢？答案很简单：限域`enum`的底层类型总是已知的，而对于非限域`enum`，你可以指定它。

默认情况下，限域枚举的底层类型是`int`：

```cpp
enum class Status;                  //底层类型是int
```

如果默认的`int`不适用，你可以重写它：

```cpp
enum class Status: std::uint32_t;   //Status的底层类型
                                    //是std::uint32_t
                                    //（需要包含 <cstdint>）
```

不管怎样，编译器都知道限域`enum`中的枚举名占用多少字节。

**记住**

- C++98的`enum`即非限域`enum`。
- 限域`enum`的枚举名仅在`enum`内可见。要转换为其它类型只能使用*cast*。
- 非限域/限域`enum`都支持底层类型说明语法，限域`enum`底层类型默认是`int`。非限域`enum`没有默认底层类型。
- 限域`enum`总是可以前置声明。非限域`enum`仅当指定它们的底层类型时才能前置。

### Item 11: Prefer deleted functions to private undefined ones.

#### 声明自动生成的函数为私有（`private`）成员函数并且不定义（或者“`= delete`”）

如果你写的代码要被其他人使用，你不想让他们调用某个特殊的函数，你通常不会声明这个函数。无声明，不函数。简简单单！但有时C++会给你自动声明一些函数，如果你想防止客户调用这些函数，事情就不那么简单了。

上述场景见于特殊的成员函数，即当有必要时C++自动生成的那些函数。[Item17](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item17.html)详细讨论了这些函数，但是现在，我们只关心拷贝构造函数和拷贝赋值运算符重载。本节主要致力于讨论C++98中那些被C++11所取代的最佳实践，而且在C++98中，你想要禁止使用的成员函数，几乎总是拷贝构造函数或者赋值运算符，或者两者都是。

在C++98中防止调用这些函数的方法是将它们声明为私有（`private`）成员函数并且不定义。举个例子，在C++ 标准库*iostream*继承链的顶部是模板类`basic_ios`。所有*istream*和*ostream*类都继承此类（直接或者间接）。拷贝*istream*和*ostream*是不合适的，因为这些操作应该怎么做是模棱两可的。比如一个`istream`对象，代表一个输入值的流，流中有一些已经被读取，有一些可能马上要被读取。如果一个*istream*被拷贝，需要拷贝将要被读取的值和已经被读取的值吗？解决这个问题最好的方法是不定义这个操作。直接禁止拷贝流。

要使这些*istream*和*ostream*类不可拷贝，`basic_ios`在C++98中是这样声明的（包括注释）：

```cpp
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
    …

private:
    basic_ios(const basic_ios& );           // not defined
    basic_ios& operator=(const basic_ios&); // not defined
};
```

将它们声明为私有成员可以防止客户端调用这些函数。故意不定义它们意味着假如还是有代码用它们（比如成员函数或者类的友元`friend`），就会在链接时引发缺少函数定义（*missing function definitions*）错误。

在C++11中有一种更好的方式达到相同目的：用“`= delete`”将拷贝构造函数和拷贝赋值运算符标记为***deleted*函数**（译注：一些文献翻译为“删除的函数”）。上面相同的代码在C++11中是这样声明的：

```cpp
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
    …

    basic_ios(const basic_ios& ) = delete;
    basic_ios& operator=(const basic_ios&) = delete;
    …
};
```

删除这些函数（译注：添加"`= delete`"）和声明为私有成员可能看起来只是方式不同，别无其他区别。其实还有一些实质性意义。*deleted*函数不能以任何方式被调用，即使你在成员函数或者友元函数里面调用*deleted*函数也不能通过编译。这是较之C++98行为的一个改进，C++98中不正确的使用这些函数在链接时才被诊断出来。

#### **任何**函数都可以标记为*deleted*，而只有成员函数可被标记为`private`

*deleted*函数还有一个重要的优势是**任何**函数都可以标记为*deleted*，而只有成员函数可被标记为`private`。（译注：从下文可知“任何”是包含普通函数和成员函数等所有可声明函数的地方，而`private`方法只适用于成员函数）假如我们有一个非成员函数，它接受一个整型参数，检查它是否为幸运数：

```cpp
bool isLucky(int number);
```

C++有沉重的C包袱，使得含糊的、能被视作数值的任何类型都能隐式转换为`int`，但是有一些调用可能是没有意义的：

```cpp
if (isLucky('a')) …         //字符'a'是幸运数？
if (isLucky(true)) …        //"true"是?
if (isLucky(3.5)) …         //难道判断它的幸运之前还要先截尾成3？
```

如果幸运数必须真的是整型，我们该禁止这些调用通过编译。

其中一种方法就是创建*deleted*重载函数，其参数就是我们想要过滤的类型：

```cpp
bool isLucky(int number);       //原始版本
bool isLucky(char) = delete;    //拒绝char
bool isLucky(bool) = delete;    //拒绝bool
bool isLucky(double) = delete;  //拒绝float和double
```

#### *deleted*函数用于禁止一些模板的实例化

另一个*deleted*函数用武之地（`private`成员函数做不到的地方）是禁止一些模板的实例化。假如你要求一个模板仅支持原生指针（尽管[第四章](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item18.html)建议使用智能指针代替原生指针）：

```cpp
template<typename T>
void processPointer(T* ptr);
```

在指针的世界里有两种特殊情况。一是`void*`指针，因为没办法对它们进行解引用，或者加加减减等。另一种指针是`char*`，因为它们通常代表C风格的字符串，而不是正常意义下指向单个字符的指针。这两种情况要特殊处理，在`processPointer`模板里面，我们假设正确的函数应该拒绝这些类型。也即是说，`processPointer`不能被`void*`和`char*`调用。

要想确保这个很容易，使用`delete`标注模板实例：

```cpp
template<>
void processPointer<void>(void*) = delete;

template<>
void processPointer<char>(char*) = delete;
```

现在如果使用`void*`和`char*`调用`processPointer`就是无效的

有趣的是，如果类里面有一个函数模板，你可能想用`private`（经典的C++98惯例）来禁止这些函数模板实例化，但是不能这样做，因为不能给特化的成员模板函数指定一个不同于主函数模板的访问级别。如果`processPointer`是类`Widget`里面的模板函数， 你想禁止它接受`void*`参数，那么通过下面这样C++98的方法就不能通过编译：

```cpp
class Widget {
public:
    …
    template<typename T>
    void processPointer(T* ptr)
    { … }

private:
    template<>                          //错误！
    void processPointer<void>(void*);
    
};
```

问题是模板特例化必须位于一个命名空间作用域，而不是类作用域。*deleted*函数不会出现这个问题，因为它不需要一个不同的访问级别，且他们可以在类外被删除（因此位于命名空间作用域）：

```cpp
class Widget {
public:
    …
    template<typename T>
    void processPointer(T* ptr)
    { … }
    …

};

template<>                                          //还是public，
void Widget::processPointer<void>(void*) = delete;  //但是已经被删除了
```

事实上C++98的最佳实践即声明函数为`private`但不定义是在做C++11 *deleted*函数要做的事情

**请记住：**

- 比起声明函数为`private`但不定义，使用*deleted*函数更好
- 任何函数都能被删除（be deleted），包括非成员函数和模板实例（译注：实例化的函数）

### Item 12: Declare overriding functions `override`

#### 引用限定符（*reference qualifiers*）

要想重写（override）一个函数，必须满足下列要求：

- 基类函数必须是`virtual`
- 基类和派生类函数名必须完全一样（除非是析构函数)
- 基类和派生类函数形参类型必须完全一样
- 基类和派生类函数常量性`const`ness必须完全一样
- 基类和派生类函数的返回值和异常说明（*exception specifications*）必须兼容

除了这些C++98就存在的约束外，C++11又添加了一个：

- 函数的引用限定符（*reference qualifiers*）必须完全一样。成员函数的引用限定符是C++11很少抛头露脸的特性，所以如果你从没听过它无需惊讶。它可以限定成员函数只能用于左值或者右值。成员函数不需要`virtual`也能使用它们：

```cpp
class Widget {
public:
    …
    void doWork() &;    //只有*this为左值的时候才能被调用
    void doWork() &&;   //只有*this为右值的时候才能被调用
}; 
…
Widget makeWidget();    //工厂函数（返回右值）
Widget w;               //普通对象（左值）
…
w.doWork();             //调用被左值引用限定修饰的Widget::doWork版本
                        //（即Widget::doWork &）
makeWidget().doWork();  //调用被右值引用限定修饰的Widget::doWork版本
                        //（即Widget::doWork &&）
```

#### 显式声明为`override`以指定个派生类函数是基类版本的重写

于正确声明派生类的重写函数很重要，但很容易出错，C++11提供一个方法让你可以显式地指定一个派生类函数是基类版本的重写：将它声明为`override`。还是上面那个例子，我们可以这样做：

```cpp
class Derived: public Base {
public:
    virtual void mf1() override;
    virtual void mf2(unsigned int x) override;
    virtual void mf3() && override;
    virtual void mf4() const override;
};
```

代码不能编译，当然了，因为这样写的时候，编译器会抱怨所有与重写有关的问题。这也是你想要的，以及为什么要在所有重写函数后面加上`override`。

使用`override`的代码编译时看起来就像这样（假设我们的目的是`Derived`派生类中的所有函数重写`Base`基类的相应虚函数）:

```cpp
class Base {
public:
    virtual void mf1() const;
    virtual void mf2(int x);
    virtual void mf3() &;
    virtual void mf4() const;
};

class Derived: public Base {
public:
    virtual void mf1() const override;
    virtual void mf2(int x) override;
    virtual void mf3() & override;
    void mf4() const override;          //可以添加virtual，但不是必要
}; 
```

注意在这个例子中`mf4`有别于之前，它在`Base`中的声明有`virtual`修饰，所以能正常工作。大多数和重写有关的错误都是在派生类引发的，但也可能是基类的不正确导致。

比起让编译器（译注：通过warnings）告诉你想重写的而实际没有重写，不如给你的派生类重写函数全都加上`override`。如果你考虑修改修改基类虚函数的函数签名，`override`还可以帮你评估后果。如果派生类全都用上`override`，你可以只改变基类函数签名，重编译系统，再看看你造成了多大的问题（即，多少派生类不能通过编译），然后决定是否值得如此麻烦更改函数签名。没有`override`，你只能寄希望于完善的单元测试，因为，正如我们所见，派生类虚函数本想重写基类，但是没有，编译器也没有探测并发出诊断信息。

C++既有很多关键字，C++11引入了两个上下文关键字（***contextual keywords***），`override`和`final`（向虚函数添加`final`可以防止派生类重写。`final`也能用于类，这时这个类不能用作基类）。这两个关键字的特点是它们是保留的，它们只是**位于特定上下文才被视为关键字**。对于`override`，它只在成员函数声明结尾处才被视为关键字。

**请记住：**

- 为重写函数加上`override`
- 成员函数引用限定让我们可以区别对待左值对象和右值对象（即`*this`)

### Item 13: Prefer `const_iterators` to `iterators`

#### 只需要`const`的情况下建议先使用`const_iterators`

STL `const_iterator`等价于指向常量的指针（pointer-to-`const`）。它们都指向不能被修改的值。标准实践是能加上`const`就加上，这也指示我们需要一个迭代器时只要没必要修改迭代器指向的值，就应当使用`const_iterator`。

所有的这些都在C++11中改变了，现在`const_iterator`既容易获取又容易使用。容器的成员函数`cbegin`和`cend`产出`const_iterator`，甚至对于non-`const`容器也可用，那些之前使用*iterator*指示位置（如`insert`和`erase`）的STL成员函数也可以使用`const_iterator`了。使用C++11 `const_iterator`重写C++98使用`iterator`的代码也稀松平常：

```cpp
std::vector<int> values;                                //和之前一样
…
auto it =                                               //使用cbegin
    std::find(values.cbegin(), values.cend(), 1983);//和cend
values.insert(it, 1998);
```

现在使用`const_iterator`的代码就很实用了！

唯一一个C++11对于`const_iterator`支持不足（译注：C++14支持但是C++11的时候还没）的情况是：当你想写最大程度通用的库，并且这些库代码为一些容器和类似容器的数据结构提供`begin`、`end`（以及`cbegin`，`cend`，`rbegin`，`rend`等）作为**非成员函数**而不是成员函数时。其中一种情况就是原生数组，还有一种情况是一些只由自由函数组成接口的第三方库。（译注：自由函数*free function*，指的是非成员函数，即一个函数，只要不是成员函数就可被称作*free function*）最大程度通用的库会考虑使用非成员函数而不是假设成员函数版本存在。

**请记住：**

- 优先考虑`const_iterator`而非`iterator`
- 在最大程度通用的代码中，优先考虑非成员函数版本的`begin`，`end`，`rbegin`等，而非同名成员函数

### Item 14: Declare functions `noexcept` if they won’t emit exceptions

在C++11标准化过程中，大家一致认为异常说明真正有用的信息是一个函数是否会抛出异常。非黑即白，一个函数可能抛异常，或者不会。这种"可能-绝不"的二元论构成了C++11异常说的基础，从根本上改变了C++98的异常说明。（C++98风格的异常说明也有效，但是已经标记为deprecated（废弃））。在C++11中，无条件的`noexcept`保证函数不会抛出任何异常。

#### 声明函数为`noexcept`来提高代码的异常安全性（*exception safety*）和效率

考虑一个函数`f`，它保证调用者永远不会收到一个异常。两种表达方式如下：

```cpp
int f(int x) throw();   //C++98风格，没有来自f的异常
int f(int x) noexcept;  //C++11风格，没有来自f的异常
```

如果在运行时，`f`出现一个异常，那么就和`f`的异常说明冲突了。在C++98的异常说明中，调用栈（the *call stack*）会展开至`f`的调用者，在一些与这地方不相关的动作后，程序被终止。C++11异常说明的运行时行为有些不同：调用栈只是**可能**在程序终止前展开。

展开调用栈和**可能**展开调用栈两者对于代码生成（code generation）有非常大的影响。在一个`noexcept`函数中，当异常可能传播到函数外时，优化器不需要保证运行时栈（the runtime stack）处于可展开状态；也不需要保证当异常离开`noexcept`函数时，`noexcept`函数中的对象按照构造的反序析构。而标注“`throw()`”异常声明的函数缺少这样的优化灵活性，没加异常声明的函数也一样。可以总结一下：

```cpp
RetType function(params) noexcept;  //极尽所能优化
RetType function(params) throw();   //较少优化
RetType function(params);           //较少优化
```

这是一个充分的理由使得你当知道它不抛异常时加上`noexcept`。

***一个非常明显的经典问题：***

当新元素添加到`std::vector`，`std::vector`可能没地方放它，换句话说，`std::vector`的大小（size）等于它的容量（capacity）。这时候，`std::vector`会分配一个新的更大块的内存用于存放其中元素，然后将元素从老内存区移动到新内存区，然后析构老内存区里的对象。在C++98中，移动是通过复制老内存区的每一个元素到新内存区完成的，然后老内存区的每个元素发生析构。这种方法使得`push_back`可以提供很强的异常安全保证：如果在复制元素期间抛出异常，`std::vector`状态保持不变，因为老内存元素析构必须建立在它们已经成功复制到新内存的前提下。

在C++11中，一个很自然的优化就是将上述复制操作替换为移动操作。但是很不幸运，这会破坏`push_back`的异常安全保证。如果**n**个元素已经从老内存移动到了新内存区，但异常在移动第**n+1**个元素时抛出，那么`push_back`操作就不能完成。但是原始的`std::vector`已经被修改：有**n**个元素已经移动走了。恢复`std::vector`至原始状态也不太可能，因为从新内存移动到老内存本身又可能引发异常。

这是个很严重的问题，因为老代码可能依赖于`push_back`提供的强烈的异常安全保证。因此，C++11版本的实现不能简单的将`push_back`里面的复制操作替换为移动操作，除非知晓移动操作绝不抛异常，这时复制替换为移动就是安全的，唯一的副作用就是性能得到提升。

#### 异常中立（*exception-neutral*）的函数

这些函数自己不抛异常，但是它们内部的调用可能抛出。此时，异常中立函数允许那些抛出异常的函数在调用链上更进一步直到遇到异常处理程序，而不是就地终止。异常中立函数决不应该声明为`noexcept`，因为它们可能抛出那种“让它们过吧”的异常（译注：也就是说在当前这个函数内不处理异常，但是又不立即终止程序，而是让调用这个函数的函数处理异常。）因此大多数函数缺少`noexcept`设计。

然而，一些函数很自然的不应该抛异常，更进一步——尤其是移动操作和`swap`——使其`noexcept`有重大意义，只要可能就应该将它们实现为`noexcept`。对于一些函数，使其成为`noexcept`是很重要的，它们应当默认如是。在C++98，允许内存释放（memory deallocation）函数（即`operator delete`和`operator delete[]`）和析构函数抛出异常是糟糕的代码设计，C++11将这种作风升级为语言规则。默认情况下，内存释放函数和析构函数——不管是用户定义的还是编译器生成的——都是隐式`noexcept`。因此它们不需要声明`noexcept`。

值得注意的是一些库接口设计者会区分有宽泛契约（**wild contracts**）和严格契约（**narrow contracts**）的函数。有宽泛契约的函数没有前置条件。这种函数不管程序状态如何都能调用，它对调用者传来的实参不设约束。宽泛契约的函数决不表现出未定义行为。反之，没有宽泛契约的函数就有严格契约。对于这些函数，如果违反前置条件，结果将会是未定义的。

现在假如`f`有一个前置条件：类型为`std::string`的参数的长度不能超过32个字符。如果现在调用`f`并传给它一个大于32字符的`std::string`，函数行为将是未定义的，因为**根据定义**违反了前置条件，导致了未定义行为。`f`没有义务去检查前置条件，它假设这些前置条件都是满足的。（调用者有责任确保参数字符不超过32字符等这些假设有效。）即使有前置条件，将`f`声明为`noexcept`似乎也是合适的：

```cpp
void f(const std::string& s) noexcept;  //前置条件：
                                        //s.length() <= 32 
```

假定`f`的实现者在函数里面检查前置条件冲突。虽然检查是没有必要的，但是也没禁止这么做，检查前置条件可能也是有用的，比如在系统测试时。debug一个抛出的异常一般都比跟踪未定义行为起因更容易。那么怎么报告前置条件冲突使得测试工具或客户端错误处理程序能检测到它呢？简单直接的做法是抛出“precondition was violated”异常，但是如果`f`声明了`noexcept`，这就行不通了；抛出一个异常会导致程序终止。因为这个原因，区分严格/宽泛契约库设计者一般会将`noexcept`留给宽泛契约函数。

**请记住：**

- `noexcept`是函数接口的一部分，这意味着调用者可能会依赖它
- `noexcept`函数较之于non-`noexcept`函数更容易优化
- `noexcept`对于移动语义，`swap`，内存释放函数和析构函数非常有用
- 大多数函数是异常中立的（译注：可能抛也可能不抛异常）而不是`noexcept`

### Item 15: Use `constexpr` whenever possible

#### `constexpr`表明一个值是编译期可知的常量

从概念上来说，`constexpr`表明一个值不仅仅是常量，还是编译期可知的。

编译期可知的值“享有特权”，它们可能被存放到只读存储空间中。对于那些嵌入式系统的开发者，这个特性是相当重要的。更广泛的应用是“其值编译期可知”的常量整数会出现在需要“整型常量表达式（**integral constant expression**）的上下文中，这类上下文包括数组大小，整数模板参数（包括`std::array`对象的长度），枚举名的值，对齐修饰符（译注：[`alignas(val)`](https://en.cppreference.com/w/cpp/language/alignas)），等等。如果你想在这些上下文中使用变量，你一定会希望将它们声明为`constexpr`，因为编译器会确保它们是编译期可知的：

```cpp
int sz;                             //non-constexpr变量
…
constexpr auto arraySize1 = sz;     //错误！sz的值在
                                    //编译期不可知
std::array<int, sz> data1;          //错误！一样的问题
constexpr auto arraySize2 = 10;     //没问题，10是
                                    //编译期可知常量
std::array<int, arraySize2> data2;  //没问题, arraySize2是constexpr
```

注意`const`不提供`constexpr`所能保证之事，因为`const`对象不需要在编译期初始化它的值。

#### 当传递编译期可知的值时`constexpr`函数可以产出编译期可知的结果

涉及到`constexpr`函数时，`constexpr`对象的使用情况就更有趣了。如果实参是编译期常量，这些函数将产出编译期常量；如果实参是运行时才能知道的值，它们就将产出运行时值。这听起来就像你不知道它们要做什么一样，那么想是错误的，请这么看：

- `constexpr`函数可以用于需求编译期常量的上下文。如果你传给`constexpr`函数的实参在编译期可知，那么结果将在编译期计算。如果实参的值在编译期不知道，你的代码就会被拒绝。
- 当一个`constexpr`函数被一个或者多个编译期不可知值调用时，它就像普通函数一样，运行时计算它的结果。这意味着你不需要两个函数，一个用于编译期计算，一个用于运行时计算。`constexpr`全做了。

#### C++11和C++14中对于`constexpr`的不同限制

C++11中，`constexpr`函数的代码不超过一行语句：一个`return`。听起来很受限，但实际上有两个技巧可以扩展`constexpr`函数的表达能力。第一，使用三元运算符“`?:`”来代替`if`-`else`语句，第二，使用递归代替循环。因此`pow`可以像这样实现：

```cpp
constexpr int pow(int base, int exp) noexcept
{
    return (exp == 0 ? 1 : base * pow(base, exp - 1));
}
```

这样没问题，但是很难想象除了使用函数式语言的程序员外会觉得这样硬核的编程方式更好。在C++14中，`constexpr`函数的限制变得非常宽松了，所以下面的函数实现成为了可能：

```cpp
constexpr int pow(int base, int exp) noexcept   //C++14
{
    auto result = 1;
    for (int i = 0; i < exp; ++i) result *= base;
    
    return result;
}
```

`constexpr`函数限制为只能获取和返回**字面值类型**，这基本上意味着那些有了值的类型能在编译期决定。在C++11中，除了`void`外的所有内置类型，以及一些用户定义类型都可以是字面值类型，因为构造函数和其他成员函数可能是`constexpr`

在C++11中，有两个限制使得`Point`的成员函数`setX`和`setY`不能声明为`constexpr`。第一，它们修改它们操作的对象的状态， 并且在C++11中，`constexpr`成员函数是隐式的`const`。第二，它们有`void`返回类型，`void`类型不是C++11中的字面值类型。这两个限制在C++14中放开了，所以C++14中`Point`的*setter*（赋值器）也能声明为`constexpr`：

**请记住：**

- `constexpr`对象是`const`，它被在编译期可知的值初始化
- 当传递编译期可知的值时，`constexpr`函数可以产出编译期可知的结果
- `constexpr`对象和函数可以使用的范围比non-`constexpr`对象和函数要大
- `constexpr`是对象和函数接口的一部分

### Item 16: Make `const` member functions thread safe

#### 使用`mutablr`声明可以在`const`函数中改变的成员变量

如果我们在数学领域中工作，我们就会发现用一个类表示多项式是很方便的。在这个类中，使用一个函数来计算多项式的根是很有用的，也就是多项式的值为零的时候（译者注：通常也被叫做零点，即使得多项式值为零的那些取值）。这样的一个函数它不会更改多项式。所以，它自然被声明为`const`函数。

```c++
class Polynomial {
public:
    using RootsType =           //数据结构保存多项式为零的值
          std::vector<double>;  //（“using” 的信息查看条款9）
    …
    RootsType roots() const;
    …
};
```

计算多项式的根是很复杂的，因此如果不需要的话，我们就不做。如果必须做，我们肯定不想再做第二次。所以，如果必须计算它们，就缓存多项式的根，然后实现`roots`来返回缓存的值。下面是最基本的实现：

```c++
class Polynomial {
public:
    using RootsType = std::vector<double>;
    
    RootsType roots() const
    {
        if (!rootsAreValid) {               //如果缓存不可用
            …                               //计算根
                                            //用rootVals存储它们
            rootsAreValid = true;
        }
        
        return rootVals;
    }
    
private:
    mutable bool rootsAreValid{ false };    //初始化器（initializer）的
    mutable RootsType rootVals{};           //更多信息请查看条款7
};
```

从概念上讲，`roots`并不改变它所操作的`Polynomial`对象。但是作为缓存的一部分，它也许会改变`rootVals`和`rootsAreValid`的值。这就是`mutable`的经典使用样例，这也是为什么它是数据成员声明的一部分。

#### 使用`std::atomic`变量可能比互斥量`std::mutex`提供更好的性能

`roots`被声明为`const`，但不是线程安全的。`const`声明在C++11中与在C++98中一样正确（检索多项式的根并不会更改多项式的值），因此需要纠正的是线程安全的缺乏。

解决这个问题最普遍简单的方法就是——使用`mutex`（互斥量）：

```c++
class Polynomial {
public:
    using RootsType = std::vector<double>;
    
    RootsType roots() const
    {
        std::lock_guard<std::mutex> g(m);       //锁定互斥量
        
        if (!rootsAreValid) {                   //如果缓存无效
            …                                   //计算/存储根值
            rootsAreValid = true;
        }
        
        return rootsVals;
    }                                           //解锁互斥量
    
private:
    mutable std::mutex m;
    mutable bool rootsAreValid { false };
    mutable RootsType rootsVals {};
};
```

`std::mutex m`被声明为`mutable`，因为锁定和解锁它的都是non-`const`成员函数。在`roots`（`const`成员函数）中，`m`却被视为`const`对象。

在某些情况下，互斥量的副作用显会得过大。例如，如果你所做的只是计算成员函数被调用了多少次，使用`std::atomic` 修饰的计数器（保证其他线程视它的操作为不可分割的整体，参见[item40](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item40.html)）通常会是一个开销更小的方法。（然而它是否轻量取决于你使用的硬件和标准库中互斥量的实现。）以下是如何使用`std::atomic`来统计调用次数。

```c++
class Point {                                   //2D点
public:
    …
    double distanceFromOrigin() const noexcept  //noexcept的使用
    {                                           //参考条款14
        ++callCount;                            //atomic的递增
        
        return std::sqrt((x * x) + (y * y));
    }

private:
    mutable std::atomic<unsigned> callCount{ 0 };
    double x, y;
};
```

因为对`std::atomic`变量的操作通常比互斥量的获取和释放的消耗更小，所以你可能会过度倾向与依赖`std::atomic`。例如，在一个类中，缓存一个开销昂贵的`int`，你就会尝试使用一对`std::atomic`变量而不是互斥量。

**这里有一个坑。对于需要同步的是单个的变量或者内存位置，使用`std::atomic`就足够了。不过，一旦你需要对两个以上的变量或内存位置作为一个单元来操作的话，就应该使用互斥量。**

**请记住：**

- 确保`const`成员函数线程安全，除非你**确定**它们永远不会在并发上下文（*concurrent context*）中使用。
- 使用`std::atomic`变量可能比互斥量提供更好的性能，但是它只适合操作单个变量或内存位置。

### Item 17: Understand special member function generation

#### **特殊成员函数**是指C++类自己默认生成的函数

在C++术语中，**特殊成员函数**是指C++自己生成的函数。C++98有四个：默认构造函数，析构函数，拷贝构造函数，拷贝赋值运算符。当然在这里有些细则要注意。这些函数仅在需要的时候才生成，比如某个代码使用它们但是它们没有在类中明确声明。默认构造函数仅在类完全没有构造函数的时候才生成。（防止编译器为某个类生成构造函数，但是你希望那个构造函数有参数）生成的特殊成员函数是隐式public且`inline`，它们是非虚的，除非相关函数是在派生类中的析构函数，派生类继承了有虚析构函数的基类。在这种情况下，编译器为派生类生成的析构函数是虚的。

但是你早就知道这些了。好吧好吧，都说古老的历史：美索不达米亚，商朝，FORTRAN，C++98。但是时代改变了，C++生成特殊成员的规则也改变了。要留意这些新规则，知道什么时候编译器会悄悄地向你的类中添加成员函数，因为没有什么比这件事对C++高效编程更重要。

C++11特殊成员函数俱乐部迎来了两位新会员：移动构造函数和移动赋值运算符。它们的签名是：

```cpp
class Widget {
public:
    …
    Widget(Widget&& rhs);               //移动构造函数
    Widget& operator=(Widget&& rhs);    //移动赋值运算符
    …
};
```

掌控它们生成和行为的规则类似于拷贝系列。移动操作仅在需要的时候生成，如果生成了，就会对类的non-static数据成员执行逐成员的移动。那意味着移动构造函数根据`rhs`参数里面对应的成员移动构造出新non-static部分，移动赋值运算符根据参数里面对应的non-static成员移动赋值。移动构造函数也移动构造基类部分（如果有的话），移动赋值运算符也是移动赋值基类部分。

#### 析构函数只有在类没有显式声明移动操作和拷贝操作时才自动生成移动操作

像拷贝操作情况一样，如果你自己声明了移动操作，编译器就不会生成。然而它们生成的精确条件与拷贝操作的条件有点不同。

两个拷贝操作是独立的：声明一个不会限制编译器生成另一个。所以如果你声明一个拷贝构造函数，但是没有声明拷贝赋值运算符，如果写的代码用到了拷贝赋值，编译器会帮助你生成拷贝赋值运算符。同样的，如果你声明拷贝赋值运算符但是没有拷贝构造函数，代码用到拷贝构造函数时编译器就会生成它。上述规则在C++98和C++11中都成立。

两个移动操作不是相互独立的。如果你声明了其中一个，编译器就不再生成另一个。如果你给类声明了，比如，一个移动构造函数，就表明对于移动操作应怎样实现，与编译器应生成的默认逐成员移动有些区别。如果逐成员移动构造有些问题，那么逐成员移动赋值同样也可能有问题。所以声明移动构造函数阻止移动赋值运算符的生成，声明移动赋值运算符同样阻止编译器生成移动构造函数。

再进一步，如果一个类显式声明了拷贝操作，编译器就不会生成移动操作。这种限制的解释是如果声明拷贝操作（构造或者赋值）就暗示着平常拷贝对象的方法（逐成员拷贝）不适用于该类，编译器会明白如果逐成员拷贝对拷贝操作来说不合适，逐成员移动也可能对移动操作来说不合适。

#### C++11对于特殊成员函数处理的规则

C++11对于特殊成员函数处理的规则如下：

- **默认构造函数**：和C++98规则相同。仅当类不存在用户声明的构造函数时才自动生成。
- **析构函数**：基本上和C++98相同；稍微不同的是现在析构默认`noexcept`（参见[Item14](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item14.html)）。和C++98一样，仅当基类析构为虚函数时该类析构才为虚函数。
- **拷贝构造函数**：和C++98运行时行为一样：逐成员拷贝non-static数据。仅当类没有用户定义的拷贝构造时才生成。如果类声明了移动操作它就是*delete*的。当用户声明了拷贝赋值或者析构，该函数自动生成已被废弃。
- **拷贝赋值运算符**：和C++98运行时行为一样：逐成员拷贝赋值non-static数据。仅当类没有用户定义的拷贝赋值时才生成。如果类声明了移动操作它就是*delete*的。当用户声明了拷贝构造或者析构，该函数自动生成已被废弃。
- **移动构造函数**和**移动赋值运算符**：都对非static数据执行逐成员移动。仅当类没有用户定义的拷贝操作，移动操作或析构时才自动生成。

注意没有“成员函数**模版**阻止编译器生成特殊成员函数”的规则。

**请记住：**

- 特殊成员函数是编译器可能自动生成的函数：默认构造函数，析构函数，拷贝操作，移动操作。
- 移动操作仅当类没有显式声明移动操作，拷贝操作，析构函数时才自动生成。
- 拷贝构造函数仅当类没有显式声明拷贝构造函数时才自动生成，并且如果用户声明了移动操作，拷贝构造就是*delete*。拷贝赋值运算符仅当类没有显式声明拷贝赋值运算符时才自动生成，并且如果用户声明了移动操作，拷贝赋值运算符就是*delete*。当用户声明了析构函数，拷贝操作的自动生成已被废弃。
- 成员函数模板不抑制特殊成员函数的生成。