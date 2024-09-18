---
title: 
title: "Effective Modern C++ Notes (1-2)"
excerpt: "如果你不介意浏览少许伪代码，我们可以考虑像这样一个函数模板"
collection: docs
---

## Chapter 1 类型推导

### Item 1: Understand template type deduction

如果你不介意浏览少许伪代码，我们可以考虑像这样一个函数模板：

```cpp
template<typename T>
void f(ParamType param);
```

它的调用看起来像这样

```cpp
f(expr);                        //使用表达式调用f
```

在编译期间，编译器使用`expr`进行两个类型推导：一个是针对`T`的，另一个是针对`ParamType`的。这两个类型通常是不同的，因为`ParamType`包含一些修饰，比如`const`和引用修饰符。

我们可能很自然的期望`T`和传递进函数的实参是相同的类型，也就是，`T`为`expr`的类型。在上面的例子中，事实就是那样：`x`是`int`，`T`被推导为`int`。但有时情况并非总是如此，`T`的类型推导不仅取决于`expr`的类型，也取决于`ParamType`的类型。这里有三种情况：

- `ParamType`是一个指针或引用，但不是通用引用（关于通用引用请参见[Item24](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item24.html)。在这里你只需要知道它存在，而且不同于左值引用和右值引用）
- `ParamType`是一个通用引用
- `ParamType`既不是指针也不是引用

我们下面将分成三个情景来讨论这三种情况，每个情景的都基于我们之前给出的模板

#### 情景一：`ParamType`是一个指针或引用，但不是通用引用

最简单的情况是`ParamType`是一个指针或者引用，但非通用引用。在这种情况下，类型推导会这样进行：

1. 如果`expr`的类型是一个引用，忽略引用部分
2. 然后`expr`的类型与`ParamType`进行模式匹配来决定`T`

#### 情景二：`ParamType`是一个通用引用

模板使用通用引用形参的话，那事情就不那么明显了。这样的形参被声明为像右值引用一样（也就是，在函数模板中假设有一个类型形参`T`，那么通用引用声明形式就是`T&&`)，它们的行为在传入左值实参时大不相同。完整的叙述请参见[Item24](https://cntransgroup.github.io/EffectiveModernCppChinese/5.RRefMovSemPerfForw/item24.html)，在这有些最必要的你还是需要知道：

- 如果`expr`是左值，`T`和`ParamType`都会被推导为左值引用。这非常不寻常，第一，这是模板类型推导中唯一一种`T`被推导为引用的情况。第二，虽然`ParamType`被声明为右值引用类型，但是最后推导的结果是左值引用。
- 如果`expr`是右值，就使用正常的（也就是**情景一**）推导规则

#### 情景三：`ParamType`既不是指针也不是引用

当`ParamType`既不是指针也不是引用时，我们通过传值（pass-by-value）的方式处理：

```cpp
template<typename T>
void f(T param);                //以传值的方式处理param
```

这意味着无论传递什么`param`都会成为它的一份拷贝——一个完整的新对象。事实上`param`成为一个新对象这一行为会影响`T`如何从`expr`中推导出结果。

1. 和之前一样，如果`expr`的类型是一个引用，忽略这个引用部分
2. 如果忽略`expr`的引用性（reference-ness）之后，`expr`是一个`const`，那就再忽略`const`。如果它是`volatile`，也忽略`volatile`（`volatile`对象不常见，它通常用于驱动程序的开发中。关于`volatile`的细节请参见[Item40](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item40.html)）

**请记住：**

- 在模板类型推导时，有引用的实参会被视为无引用，他们的引用会被忽略
- 对于通用引用的推导，左值实参会被特殊对待
- 对于传值类型推导，`const`和/或`volatile`实参会被认为是non-`const`的和non-`volatile`的
- 在模板类型推导时，数组名或者函数名实参会退化为指针，除非它们被用于初始化引用

### Item 2: Understand `auto` type deduction

#### `auto`类型推导和模板类型推导几乎一样的工作

当一个变量使用`auto`进行声明时，`auto`扮演了模板中`T`的角色，变量的类型说明符扮演了`ParamType`的角色。废话少说，这里便是更直观的代码描述，考虑这个例子：

```cpp
auto x = 27;
```

这里`x`的类型说明符是`auto`自己，另一方面，在这个声明中：

```cpp
const auto cx = x;
```

类型说明符是`const auto`。另一个：

```cpp
const auto& rx = x;
```

类型说明符是`const auto&`。在这里例子中要推导`x`，`cx`和`rx`的类型，编译器的行为看起来就像是认为这里每个声明都有一个模板，然后使用合适的初始化表达式进行调用

#### C++11添加了用于支持统一初始化（**uniform initialization**）的语法

```cpp
auto x1 = 27;                   //类型是int，值是27
auto x2(27);                    //同上
auto x3 = { 27 };               //类型是std::initializer_list<int>，
                                //值是{ 27 }
auto x4{ 27 };                  //同上
```

这些声明都能通过编译，但是他们不像替换之前那样有相同的意义。前面两个语句确实声明了一个类型为`int`值为27的变量，但是后面两个声明了一个存储一个元素27的 `std::initializer_list<int>`类型的变量。**如果这样的一个类型不能被成功推导（比如花括号里面包含的是不同类型的变量），编译器会拒绝这样的代码**

因此`auto`类型推导和模板类型推导的真正区别在于，`auto`类型推导假定花括号表示`std::initializer_list`而模板类型推导不会这样（确切的说是不知道怎么办）

对于C++11故事已经说完了。但是对于C++14故事还在继续，C++14允许`auto`用于函数返回值并会被推导（参见[Item3](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item3.html)），而且C++14的*lambda*函数也允许在形参声明中使用`auto`。但是在这些情况下`auto`实际上使用**模板类型推导**的那一套规则在工作，而不是`auto`类型推导，所以说下面这样的代码不会通过编译：

```cpp
auto createInitList()
{
    return { 1, 2, 3 };         //错误！不能推导{ 1, 2, 3 }的类型
}
```

同样在C++14的lambda函数中这样使用auto也不能通过编译：

```cpp
std::vector<int> v;
…
auto resetV = 
    [&v](const auto& newValue){ v = newValue; };        //C++14
…
resetV({ 1, 2, 3 });            //错误！不能推导{ 1, 2, 3 }的类型
```

**请记住：**

- `auto`类型推导通常和模板类型推导相同，但是`auto`类型推导假定花括号初始化代表`std::initializer_list`，而模板类型推导不这样做
- 在C++14中`auto`允许出现在函数返回值或者*lambda*函数形参中，但是它的工作机制是模板类型推导那一套方案，而不是`auto`类型推导

### Item 3: Understand decltype

在C++11中，`decltype`最主要的用途就是用于声明函数模板，而这个函数返回类型依赖于形参类型。

#### **尾置返回类型**语法

对一个`T`类型的容器使用`operator[]` 通常会返回一个`T&`对象，比如`std::deque`就是这样。但是`std::vector`有一个例外，对于`std::vector<bool>`，`operator[]`不会返回`bool&`，它会返回一个全新的对象（译注：MSVC的STL实现中返回的是`std::_Vb_reference<std::_Wrap_alloc<std::allocator<unsigned int>>>`对象）。关于这个问题的详细讨论请参见[Item6](https://cntransgroup.github.io/EffectiveModernCppChinese/2.Auto/item6.html)，这里重要的是我们可以看到对一个容器进行`operator[]`操作返回的类型取决于容器本身。

使用`decltype`使得我们很容易去实现它，这是我们写的第一个版本，使用`decltype`计算返回类型，这个模板需要改良，我们把这个推迟到后面：

```cpp
template<typename Container, typename Index>    //可以工作，
auto authAndAccess(Container& c, Index i)       //但是需要改良
    ->decltype(c[i])
{
    authenticateUser();
    return c[i];
}
```

函数名称前面的`auto`不会做任何的类型推导工作。相反的，他只是暗示使用了C++11的**尾置返回类型**语法，即在函数形参列表后面使用一个”`->`“符号指出函数的返回类型，尾置返回类型的好处是我们可以在函数返回类型中使用函数形参相关的信息。在`authAndAccess`函数中，我们使用`c`和`i`指定返回类型。如果我们按照传统语法把函数返回类型放在函数名称之前，`c`和`i`就未被声明所以不能使用。

在这种声明中，`authAndAccess`函数返回`operator[]`应用到容器中返回的对象的类型，这也正是我们期望的结果。

C++14扩展到允许自动推导所有的*lambda*表达式和函数，甚至它们内含多条语句。对于`authAndAccess`来说这意味着在**C++14标准下我们可以忽略尾置返回类型**，只留下一个`auto`。使用这种声明形式，auto标示这里会发生类型推导。更准确的说，编译器将会从函数实现中推导出函数的返回类型。

#### decltype(auto)

```cpp
template<typename Container, typename Index>    //C++14版本，
auto authAndAccess(Container& c, Index i)       //不那么正确
{
    authenticateUser();
    return c[i];                                //从c[i]中推导返回类型
}
```

[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)解释了函数返回类型中使用`auto`，编译器实际上是使用的模板类型推导的那套规则。如果那样的话这里就会有一些问题。正如我们之前讨论的，`operator[]`对于大多数`T`类型的容器会返回一个`T&`，但是[Item1](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item1.html)解释了在模板类型推导期间，**表达式的引用性（reference-ness）会被忽略**

要想让`authAndAccess`像我们期待的那样工作，我们需要使用`decltype`类型推导来推导它的返回值，即指定`authAndAccess`应该返回一个和`c[i]`表达式类型一样的类型。C++期望在某些情况下当类型被暗示时需要使用`decltype`类型推导的规则，C++14通过使用`decltype(auto)`说明符使得这成为可能。我们第一次看见`decltype(auto)`可能觉得非常的矛盾（到底是`decltype`还是`auto`？），实际上我们可以这样解释它的意义：`auto`说明符表示这个类型将会被推导，`decltype`说明`decltype`的规则将会被用到这个推导过程中。因此我们可以这样写`authAndAccess`：

```cpp
template<typename Container, typename Index>    //C++14版本，
decltype(auto)                                  //可以工作，
authAndAccess(Container& c, Index i)            //但是还需要
{                                               //改良
    authenticateUser();
    return c[i];
}
```

现在`authAndAccess`将会真正的返回`c[i]`的类型。现在事情解决了，一般情况下`c[i]`返回`T&`，`authAndAccess`也会返回`T&`，特殊情况下`c[i]`返回一个对象，`authAndAccess`也会返回一个对象。

将`decltype`应用于变量名会产生该变量名的声明类型。虽然变量名都是左值表达式，但这不会影响`decltype`的行为。（译者注：这里是说对于单纯的变量名，`decltype`只会返回变量的声明类型）然而，对于比单纯的变量名更复杂的左值表达式，`decltype`可以确保报告的类型始终是左值引用。也就是说，如果一个不是单纯变量名的左值表达式的类型是`T`，那么`decltype`会把这个表达式的类型报告为`T&`。这几乎没有什么太大影响，因为大多数左值表达式的类型天生具备一个左值引用修饰符。例如，返回左值的函数总是返回左值引用。

这个行为暗含的意义值得我们注意，在：

```cpp
int x = 0;
```

中，`x`是一个变量的名字，所以`decltype(x)`是`int`。但是如果用一个小括号包覆这个名字，比如这样`(x)` ，就会产生一个比名字更复杂的表达式。对于名字来说，`x`是一个左值，C++11定义了表达式`(x)`也是一个左值。因此`decltype((x))`是`int&`。用小括号覆盖一个名字可以改变`decltype`对于名字产生的结果。

**请记住：**

- `decltype`总是不加修改的产生变量或者表达式的类型。
- 对于`T`类型的不是单纯的变量名的左值表达式，`decltype`总是产出`T`的引用即`T&`。
- C++14支持`decltype(auto)`，就像`auto`一样，推导出类型，但是它使用`decltype`的规则进行推导。

### Item 4: Know how to view deduced types

(……)

## Chapter 2 auto

### item 5: Prefer `auto` to explicit type declarations

#### `auto`变量从初始化表达式中推导出类型，所以我们必须初始化

`std::function`是一个C++11标准模板库中的一个模板，它泛化了函数指针的概念。与函数指针只能指向函数不同，`std::function`可以指向任何可调用对象，也就是那些像函数一样能进行调用的东西。当你声明函数指针时你必须指定函数类型（即函数签名），同样当你创建`std::function`对象时你也需要提供函数签名，由于它是一个模板所以你需要在它的模板参数里面提供。举个例子，假设你想声明一个`std::function`对象`func`使它指向一个可调用对象，比如一个具有这样函数签名的函数，

```cpp
bool(const std::unique_ptr<Widget> &,           //C++11
     const std::unique_ptr<Widget> &)           //std::unique_ptr<Widget>
                                                //比较函数的签名
```

你就得这么写：

```cpp
std::function<bool(const std::unique_ptr<Widget> &,
                   const std::unique_ptr<Widget> &)> func;
```

语法冗长不说，还需要重复写很多形参类型，使用`std::function`还不如使用`auto`。用`auto`声明的变量保存一个和闭包一样类型的（新）闭包，因此使用了与闭包相同大小存储空间。实例化`std::function`并声明一个对象这个对象将会有固定的大小。这个大小可能不足以存储一个闭包，这个时候`std::function`的构造函数将会在堆上面分配内存来存储，这就造成了使用`std::function`比`auto`声明变量会消耗更多的内存。并且通过具体实现我们得知通过`std::function`调用一个闭包几乎无疑比`auto`声明的对象调用要慢。换句话说，`std::function`方法比`auto`方法要更耗空间且更慢，还可能有*out-of-memory*异常。并且正如上面的例子，比起写`std::function`实例化的类型来，使用`auto`要方便得多。在这场存储闭包的比赛中，`auto`无疑取得了胜利（也可以使用`std::bind`来生成一个闭包，但在[Item34](https://cntransgroup.github.io/EffectiveModernCppChinese/6.LambdaExpressions/item34.html)我会尽我最大努力说服你使用*lambda*表达式代替`std::bind`)

#### 类型快捷方式（type shortcuts）

你将看到这样的代码——甚至你会这么写：

```cpp
std::vector<int> v;
…
unsigned sz = v.size();
```

`v.size()`的标准返回类型是`std::vector<int>::size_type`，但是只有少数开发者意识到这点。`std::vector<int>::size_type`实际上被指定为无符号整型，所以很多人都认为用`unsigned`就足够了，写下了上述的代码。这会造成一些有趣的结果。举个例子，在**Windows 32-bit**上`std::vector<int>::size_type`和`unsigned`是一样的大小，但是在**Windows 64-bit**上`std::vector<int>::size_type`是64位，`unsigned`是32位。这意味着这段代码在Windows 32-bit上正常工作，但是当把应用程序移植到Windows 64-bit上时就可能会出现一些问题。谁愿意花时间处理这些细枝末节的问题呢？

所以使用`auto`可以确保你不需要浪费时间：

```cpp
auto sz =v.size();                      //sz的类型是std::vector<int>::size_type
```

你还是不相信使用`auto`是多么明智的选择？

**请记住：**

- `auto`变量必须初始化，通常它可以避免一些移植性和效率性的问题，也使得重构更方便，还能让你少打几个字。
- 正如[Item2](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item2.html)和[6](https://cntransgroup.github.io/EffectiveModernCppChinese/2.Auto/item6.html)讨论的，`auto`类型的变量可能会踩到一些陷阱。

### Item 6: Use the explicitly typed initializer idiom when `auto` deduces undesired types

#### 代理类(proxy class)

更进一步假设第5个*bit*表示`Widget`是否具有高优先级，我们可以写这样的代码：

```cpp
Widget w;
…
bool highPriority = features(w)[5];     //w高优先级吗？
…
processWidget(w, highPriority);         //根据它的优先级处理w
```

这个代码没有任何问题。它会正常工作，但是如果我们使用`auto`代替`highPriority`的显式指定类型做一些看起来很无害的改变：

```cpp
auto highPriority = features(w)[5];     //w高优先级吗？
```

情况变了。所有代码仍然可编译，但是行为不再可预测：

```cpp
processWidget(w,highPriority);          //未定义行为！
```

就像注释说的，这个`processWidget`是一个未定义行为。为什么呢？答案有可能让你很惊讶，使用`auto`后`highPriority`不再是`bool`类型。虽然从概念上来说`std::vector<bool>`意味着存放`bool`，但是`std::vector<bool>`的`operator[]`不会返回容器中元素的引用（这就是`std::vector::operator[]`可返回**除了`bool`以外**的任何类型），取而代之它返回一个`std::vector<bool>::reference`的对象（一个嵌套于`std::vector<bool>`中的类）。`std::vector<bool>::reference`之所以存在是因为`std::vector<bool>`规定了使用一个打包形式（packed form）表示它的`bool`，每个`bool`占一个*bit*。

`highPriority`是这个`std::vector<bool>::reference`的拷贝，所以`highPriority`也包含一个指针，指向`temp`中的这个*word*，加上相应于第5个*bit*的偏移。在这个语句结束的时候`temp`将会被销毁，因为它是一个临时变量。因此`highPriority`包含一个悬置的（*dangling*）指针，如果用于`processWidget`调用中将会造成未定义行为：

```cpp
processWidget(w, highPriority);         //未定义行为！
                                        //highPriority包含一个悬置指针！
```

`std::vector<bool>::reference`是一个代理类（*proxy class*）的例子：所谓代理类就是以模仿和增强一些类型的行为为目的而存在的类。

实际上， 很多开发者都是在跟踪一些令人困惑的复杂问题或在单元测试出错进行调试时才看到代理类的使用。不管你怎么发现它们的，一旦看到`auto`推导了代理类的类型而不是被代理的类型，解决方案并不需要抛弃`auto`。`auto`本身没什么问题，问题是`auto`不会推导出你想要的类型。解决方案是强制使用一个不同的类型推导形式，这种方法我通常称之为显式类型初始器惯用法（*the explicitly typed initialized idiom*)。

显式类型初始器惯用法使用`auto`声明一个变量，然后对表达式强制类型转换（*cast*）得出你期望的推导结果。举个例子，我们该怎么将这个惯用法施加到`highPriority`上？

```cpp
auto highPriority = static_cast<bool>(features(w)[5]);
```

这里，`features(w)[5]`还是返回一个`std::vector<bool>::reference`对象，就像之前那样，但是这个转型使得表达式类型为`bool`，然后`auto`才被用于推导`highPriority`。在运行时，对`std::vector<bool>::operator[]`返回的`std::vector<bool>::reference`执行它支持的向`bool`的转型，在这个过程中指向`std::vector<bool>`的指针已经被解引用。这就避开了我们之前的未定义行为。然后5将被用于指向*bit*的指针，`bool`值被用于初始化`highPriority`。

## Chapter 3 移步现代C++

### Item 7: Distinguish between `()` and `{}` when creating objects

