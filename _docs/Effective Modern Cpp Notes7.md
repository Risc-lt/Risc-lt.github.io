---
title: "Effective Modern C++ Notes (7)"
excerpt: "C++11的伟大成功之一是将并发整合到语言和库中。熟悉其他线程API（比如pthreads或者Windows threads）的开发者有时可能会对C++提供的斯巴达式（译者注：应该是简陋和严谨的意思）功能集感到惊讶，这是因为C++对于并发的大量支持是在对编译器作者约束的层面。"
collection: docs
---

## CHAPTER 7 The Concurrency API

C++11的伟大成功之一是将并发整合到语言和库中。熟悉其他线程API（比如pthreads或者Windows threads）的开发者有时可能会对C++提供的斯巴达式（译者注：应该是简陋和严谨的意思）功能集感到惊讶，这是因为C++对于并发的大量支持是在对编译器作者约束的层面。由此产生的语言保证意味着在C++的历史中，开发者首次通过标准库可以写出跨平台的多线程程序。这为构建表达库奠定了坚实的基础，标准库并发组件（任务*tasks*，期望*futures*，线程*threads*，互斥*mutexes*，条件变量*condition variables*，原子对象*atomic objects*等）仅仅是成为并发软件开发者丰富工具集的基础。

在接下来的条款中，记住标准库有两个*future*的模板：`std::future`和`std::shared_future`。在许多情况下，区别不重要，所以我们经常简单的混于一谈为*futures*。

### Item 35: Prefer task-based programming to thread-based

#### 异步执行的方法

如果开发者想要异步执行`doAsyncWork`函数，通常有两种方式。其一是通过创建`std::thread`执行`doAsyncWork`，这是应用了**基于线程**（*thread-based*）的方式：

```cpp
int doAsyncWork();
std::thread t(doAsyncWork);
```

其二是将`doAsyncWork`传递给`std::async`，一种**基于任务**（*task-based*）的策略：

```cpp
auto fut = std::async(doAsyncWork); //“fut”表示“future”
```

这种方式中，传递给`std::async`的函数对象被称为一个**任务**（*task*）。

基于任务的方法通常比基于线程的方法更优，原因之一上面的代码已经表明，基于任务的方法代码量更少。我们假设调用`doAsyncWork`的代码对于其提供的返回值是有需求的。基于线程的方法对此无能为力，而基于任务的方法就简单了，因为`std::async`返回的*future*提供了`get`函数（从而可以获取返回值）。如果`doAsycnWork`发生了异常，`get`函数就显得更为重要，因为`get`函数可以提供抛出异常的访问，而基于线程的方法，如果`doAsyncWork`抛出了异常，程序会直接终止（通过调用`std::terminate`）。

#### 基于线程与基于任务的区别

基于线程与基于任务最根本的区别在于，基于任务的抽象层次更高。基于任务的方式使得开发者从线程管理的细节中解放出来，对此在C++并发软件中总结了“*thread*”的三种含义：

- **硬件线程**（hardware threads）是真实执行计算的线程。现代计算机体系结构为每个CPU核心提供一个或者多个硬件线程。
- **软件线程**（software threads）（也被称为系统线程（OS threads、system threads））是操作系统（假设有一个操作系统。有些嵌入式系统没有。）管理的在硬件线程上执行的线程。通常可以存在比硬件线程更多数量的软件线程，因为当软件线程被阻塞的时候（比如 I/O、同步锁或者条件变量），操作系统可以调度其他未阻塞的软件线程执行提供吞吐量。
- **`std::thread`** 是C++执行过程的对象，并作为软件线程的句柄（*handle*）。有些`std::thread`对象代表“空”句柄，即没有对应软件线程，因为它们处在默认构造状态（即没有函数要执行）；有些被移动走（移动到的`std::thread`就作为这个软件线程的句柄）；有些被`join`（它们要运行的函数已经运行完）；有些被`detach`（它们和对应的软件线程之间的连接关系被打断）。

**软件线程是有限的资源。**如果开发者试图创建大于系统支持的线程数量，会抛出`std::system_error`异常。即使你编写了不抛出异常的代码，这仍然会发生，比如下面的代码，即使 `doAsyncWork`是 `noexcept`，

```cpp
int doAsyncWork() noexcept;         //noexcept见条款14
```

这段代码仍然会抛出异常：

```cpp
std::thread t(doAsyncWork);         //如果没有更多线程可用，则抛出异常
```

设计良好的软件必须能有效地处理这种可能性，但是怎样做？一种方法是在当前线程执行`doAsyncWork`，但是这可能会导致负载不均，而且如果当前线程是GUI线程，可能会导致响应时间过长的问题。另一种方法是等待某些当前运行的软件线程结束之后再创建新的`std::thread`，但是仍然有可能当前运行的线程在等待`doAsyncWork`的动作（例如产生一个结果或者报告一个条件变量）。

即使没有超出软件线程的限额，仍然可能会遇到**资源超额**（*oversubscription*）的麻烦。这是一种当前准备运行的（即未阻塞的）软件线程大于硬件线程的数量的情况。情况发生时，线程调度器（操作系统的典型部分）会将软件线程时间切片，分配到硬件上。当一个软件线程的时间片执行结束，会让给另一个软件线程，此时发生上下文切换。软件线程的上下文切换会增加系统的软件线程管理开销，当软件线程安排到与上次时间片运行时不同的硬件线程上，这个开销会更高。这种情况下，（1）CPU缓存对这个软件线程很冷淡（即几乎没有什么数据，也没有有用的操作指南）；（2）“新”软件线程的缓存数据会“污染”“旧”线程的数据，旧线程之前运行在这个核心上，而且还有可能再次在这里运行。

避免资源超额很困难，因为软件线程之于硬件线程的最佳比例取决于软件线程的执行频率，那是动态改变的，比如一个程序从IO密集型变成计算密集型，执行频率是会改变的。而且比例还依赖上下文切换的开销以及软件线程对于CPU缓存的使用效率。此外，硬件线程的数量和CPU缓存的细节（比如缓存多大，相应速度多少）取决于机器的体系结构，即使经过调校，在某一种机器平台避免了资源超额（而仍然保持硬件的繁忙状态），换一个其他类型的机器这个调校并不能提供较好效果的保证。

#### `std::async`的调度方式

这种调用方式将线程管理的职责转交给C++标准库的开发者。举个例子，这种调用方式会减少抛出资源超额异常的可能性，因为这个调用可能不会开启一个新的线程。你会想：“怎么可能？如果我要求比系统可以提供的更多的软件线程，创建`std::thread`和调用`std::async`为什么会有区别？”确实有区别，因为以这种形式调用（即使用默认启动策略——见[Item36](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item36.html)）时，`std::async`不保证会创建新的软件线程。然而，他们允许通过调度器来将特定函数（本例中为`doAsyncWork`）运行在等待此函数结果的线程上（即在对`fut`调用`get`或者`wait`的线程上），合理的调度器在系统资源超额或者线程耗尽时就会利用这个自由度。

最前沿的线程调度器使用系统级线程池（*thread pool*）来避免资源超额的问题，并且通过工作窃取算法（*work-stealing algorithm*）来提升了跨硬件核心的负载均衡。C++标准实际上并不要求使用线程池或者工作窃取，实际上C++11并发规范的某些技术层面使得实现这些技术的难度可能比想象中更有挑战。不过，库开发者在标准库实现中采用了这些技术，也有理由期待这个领域会有更多进展。如果你当前的并发编程采用基于任务的方式，在这些技术发展中你会持续获得回报。相反如果你直接使用`std::thread`编程，处理线程耗尽、资源超额、负责均衡问题的责任就压在了你身上，更不用说你对这些问题的解决方法与同机器上其他程序采用的解决方案配合得好不好了。

对比基于线程的编程方式，基于任务的设计为开发者避免了手动线程管理的痛苦，并且自然提供了一种获取异步执行程序的结果（即返回值或者异常）的方式。当然，仍然存在一些场景直接使用`std::thread`会更有优势：

- **你需要访问非常基础的线程API**。C++并发API通常是通过操作系统提供的系统级API（pthreads或者Windows threads）来实现的，系统级API通常会提供更加灵活的操作方式（举个例子，C++没有线程优先级和亲和性的概念）。为了提供对底层系统级线程API的访问，`std::thread`对象提供了`native_handle`的成员函数，而`std::future`（即`std::async`返回的东西）没有这种能力。
- **你需要且能够优化应用的线程使用**。举个例子，你要开发一款已知执行概况的服务器软件，部署在有固定硬件特性的机器上，作为唯一的关键进程。
- **你需要实现C++并发API之外的线程技术**，比如，C++实现中未支持的平台的线程池。

这些都是在应用开发中并不常见的例子，大多数情况，开发者应该优先采用基于任务的编程方式。

**请记住：**

- `std::thread` API不能直接访问异步执行的结果，如果执行函数有异常抛出，代码会终止执行。
- 基于线程的编程方式需要手动的线程耗尽、资源超额、负责均衡、平台适配性管理。
- 通过带有默认启动策略的`std::async`进行基于任务的编程方式会解决大部分问题。

### Item 36: Specify `std::launch::async` if asynchronicity is essential

#### `std::async`的异步执行策略

当你调用`std::async`执行函数时（或者其他可调用对象），你通常希望异步执行函数。但是这并不一定是你要求`std::async`执行的操作。你事实上要求这个函数按照`std::async`启动策略来执行。有两种标准策略，每种都通过`std::launch`这个限域`enum`的一个枚举名表示（关于枚举的更多细节参见[Item10](https://cntransgroup.github.io/EffectiveModernCppChinese/3.MovingToModernCpp/item10.html)）。假定一个函数`f`传给`std::async`来执行：

- **`std::launch::async`启动策略**意味着`f`必须异步执行，即在不同的线程。
- **`std::launch::deferred`启动策略**意味着`f`仅当在`std::async`返回的*future*上调用`get`或者`wait`时才执行。这表示`f`**推迟**到存在这样的调用时才执行（译者注：异步与并发是两个不同概念，这里侧重于惰性求值）。当`get`或`wait`被调用，`f`会同步执行，即调用方被阻塞，直到`f`运行结束。如果`get`和`wait`都没有被调用，`f`将不会被执行。（这是个简化说法。关键点不是要在其上调用`get`或`wait`的那个*future*，而是*future*引用的那个共享状态。（[Item38](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item38.html)讨论了*future*与共享状态的关系。）因为`std::future`支持移动，也可以用来构造`std::shared_future`，并且因为`std::shared_future`可以被拷贝，对共享状态——对`f`传到的那个`std::async`进行调用产生的——进行引用的*future*对象，有可能与`std::async`返回的那个*future*对象不同。这非常绕口，所以经常回避这个事实，简称为在`std::async`返回的*future*上调用`get`或`wait`。）

可能让人惊奇的是，`std::async`的默认启动策略——你不显式指定一个策略时它使用的那个——不是上面中任意一个。相反，是求或在一起的。下面的两种调用含义相同：

```cpp
auto fut1 = std::async(f);                      //使用默认启动策略运行f
auto fut2 = std::async(std::launch::async |     //使用async或者deferred运行f
                       std::launch::deferred,
                       f);
```

因此默认策略允许`f`异步或者同步执行。如同[Item35](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/Item35.html)中指出，这种灵活性允许`std::async`和标准库的线程管理组件承担线程创建和销毁的责任，避免资源超额，以及平衡负载。这就是使用`std::async`并发编程如此方便的原因。

#### 默认启动策略的`std::async`的一些影响

但是，使用默认启动策略的`std::async`也有一些有趣的影响。给定一个线程`t`执行此语句：

```cpp
auto fut = std::async(f);   //使用默认启动策略运行f
```

- **无法预测`f`是否会与`t`并发运行**，因为`f`可能被安排延迟运行。
- **无法预测`f`是否会在与某线程相异的另一线程上执行，这个某线程在`fut`上调用`get`或`wait`**。如果对`fut`调用函数的线程是`t`，含义就是无法预测`f`是否在异于`t`的另一线程上执行。
- **无法预测`f`是否执行**，因为不能确保在程序每条路径上，都会不会在`fut`上调用`get`或者`wait`。

默认启动策略的调度灵活性导致使用`thread_local`变量比较麻烦，因为这意味着如果`f`读写了**线程本地存储**（*thread-local storage*，TLS），不可能预测到哪个线程的变量被访问：

```cpp
auto fut = std::async(f);   //f的TLS可能是为单独的线程建的，
                            //也可能是为在fut上调用get或者wait的线程建的
```

这还会影响到基于`wait`的循环使用超时机制，因为在一个延时的任务（参见[Item35](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/Item35.html)）上调用`wait_for`或者`wait_until`会产生`std::launch::deferred`值。意味着，以下循环看似应该最终会终止，但可能实际上永远运行：

```cpp
using namespace std::literals;      //为了使用C++14中的时间段后缀；参见条款34

void f()                            //f休眠1秒，然后返回
{
    std::this_thread::sleep_for(1s);
}

auto fut = std::async(f);           //异步运行f（理论上）

while (fut.wait_for(100ms) !=       //循环，直到f完成运行时停止...
       std::future_status::ready)   //但是有可能永远不会发生！
{
    …
}
```

如果`f`与调用`std::async`的线程并发运行（即，如果为`f`选择的启动策略是`std::launch::async`），这里没有问题（假定`f`最终会执行完毕），但是如果`f`是延迟执行，`fut.wait_for`将总是返回`std::future_status::deferred`。这永远不等于`std::future_status::ready`，循环会永远执行下去。

**请记住：**

- `std::async`的默认启动策略是异步和同步执行兼有的。
- 这个灵活性导致访问`thread_local`s的不确定性，隐含了任务可能不会被执行的意思，会影响调用基于超时的`wait`的程序逻辑。
- 如果异步执行任务非常关键，则指定`std::launch::async`。

### Item 37: Make `std::thread`s unjoinable on all paths

#### `std::thread`的两种状态

每个`std::thread`对象处于两个状态之一：**可结合的**（*joinable*）或者**不可结合的**（*unjoinable*）。可结合状态的`std::thread`对应于正在运行或者可能要运行的异步执行线程。比如，对应于一个阻塞的（*blocked*）或者等待调度的线程的`std::thread`是可结合的，对应于运行结束的线程的`std::thread`也可以认为是可结合的。

不可结合的`std::thread`正如所期待：一个不是可结合状态的`std::thread`。不可结合的`std::thread`对象包括：

- **默认构造的`std::thread`s**。这种`std::thread`没有函数执行，因此没有对应到底层执行线程上。
- **已经被移动走的`std::thread`对象**。移动的结果就是一个`std::thread`原来对应的执行线程现在对应于另一个`std::thread`。
- **已经被`join`的`std::thread`** 。在`join`之后，`std::thread`不再对应于已经运行完了的执行线程。
- **已经被`detach`的`std::thread`** 。`detach`断开了`std::thread`对象与执行线程之间的连接。

（译者注：`std::thread`可以视作状态保存的对象，保存的状态可能也包括可调用对象，有没有具体的线程承载就是有没有连接）

`std::thread`的可结合性如此重要的原因之一就是当可结合的线程的析构函数被调用，程序执行会终止。比如，假定有一个函数`doWork`，使用一个过滤函数`filter`，一个最大值`maxVal`作为形参。`doWork`检查是否满足计算所需的条件，然后使用在0到`maxVal`之间的通过过滤器的所有值进行计算。如果进行过滤非常耗时，并且确定`doWork`条件是否满足也很耗时，则将两件事并发计算是很合理的。

```cpp
constexpr auto tenMillion = 10000000;           //constexpr见条款15

bool doWork(std::function<bool(int)> filter,    //返回计算是否执行；
            int maxVal = tenMillion)            //std::function见条款2
{
    std::vector<int> goodVals;                  //满足filter的值

    std::thread t([&filter, maxVal, &goodVals]  //填充goodVals
                  {
                      for (auto i = 0; i <= maxVal; ++i)
                          { if (filter(i)) goodVals.push_back(i); }
                  });

    auto nh = t.native_handle();                //使用t的原生句柄
    …                                           //来设置t的优先级

    if (conditionsAreSatisfied()) {
        t.join();                               //等t完成
        performComputation(goodVals);
        return true;                            //执行了计算
    }
    return false;                               //未执行计算
}
```

然而，这份代码在返回`doWork`时是有问题的，如果`conditionsAreSatisfied()`返回`true`，没什么问题，但是如果返回`false`或者抛出异常，在`doWork`结束调用`t`的析构函数时，`std::thread`对象`t`会是可结合的。这造成程序执行中止。

#### `thread`的隐式行为

你可能会想，为什么`std::thread`析构的行为是这样的，那是因为另外两种显而易见的方式更糟：

- **隐式`join`** 。这种情况下，`std::thread`的析构函数将等待其底层的异步执行线程完成。这听起来是合理的，但是可能会导致难以追踪的异常表现。比如，如果`conditonAreStatisfied()`已经返回了`false`，`doWork`继续等待过滤器应用于所有值就很违反直觉。

- **隐式`detach`** 。这种情况下，`std::thread`析构函数会分离`std::thread`与其底层的线程。底层线程继续运行。听起来比`join`的方式好，但是可能导致更严重的调试问题。比如，在`doWork`中，`goodVals`是通过引用捕获的局部变量。它也被*lambda*修改（通过调用`push_back`）。假定，*lambda*异步执行时，`conditionsAreSatisfied()`返回`false`。这时，`doWork`返回，同时局部变量（包括`goodVals`）被销毁。栈被弹出，并在`doWork`的调用点继续执行线程。

  调用点之后的语句有时会进行其他函数调用，并且至少一个这样的调用可能会占用曾经被`doWork`使用的栈位置。我们调用那么一个函数`f`。当`f`运行时，`doWork`启动的*lambda*仍在继续异步运行。该*lambda*可能在栈内存上调用`push_back`，该内存曾属于`goodVals`，但是现在是`f`的栈内存的某个位置。这意味着对`f`来说，内存被自动修改了！想象一下调试的时候“乐趣”吧。

标准委员会认为，销毁可结合的线程如此可怕以至于实际上禁止了它（规定销毁可结合的线程导致程序终止）。

这使你有责任确保使用`std::thread`对象时，在所有的路径上超出定义所在的作用域时都是不可结合的。但是覆盖每条路径可能很复杂，可能包括自然执行通过作用域，或者通过`return`，`continue`，`break`，`goto`或异常跳出作用域，有太多可能的路径。

#### 使用**RAII对象**（*RAII objects*）来解决scope的问题

每当你想在执行跳至块之外的每条路径执行某种操作，最通用的方式就是将该操作放入局部对象的析构函数中。这些对象称为**RAII对象**（*RAII objects*），从**RAII类**中实例化。（RAII全称为 “Resource Acquisition Is Initialization”（资源获得即初始化），尽管技术关键点在析构上而不是实例化上）。RAII类在标准库中很常见。比如STL容器（每个容器析构函数都销毁容器中的内容物并释放内存），标准智能指针（[Item18](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item18.html)-[20](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item20.html)解释了，`std::uniqu_ptr`的析构函数调用他指向的对象的删除器，`std::shared_ptr`和`std::weak_ptr`的析构函数递减引用计数），`std::fstream`对象（它们的析构函数关闭对应的文件）等。但是标准库没有`std::thread`的RAII类，可能是因为标准委员会拒绝将`join`和`detach`作为默认选项，不知道应该怎么样完成RAII。

幸运的是，完成自行实现的类并不难。

在`doWork`的例子上使用`ThreadRAII`的代码如下：

```cpp
bool doWork(std::function<bool(int)> filter,        //同之前一样
            int maxVal = tenMillion)
{
    std::vector<int> goodVals;                      //同之前一样

    ThreadRAII t(                                   //使用RAII对象
        std::thread([&filter, maxVal, &goodVals]
                    {
                        for (auto i = 0; i <= maxVal; ++i)
                            { if (filter(i)) goodVals.push_back(i); }
                    }),
                    ThreadRAII::DtorAction::join    //RAII动作
    );

    auto nh = t.get().native_handle();
    …

    if (conditionsAreSatisfied()) {
        t.get().join();
        performComputation(goodVals);
        return true;
    }

    return false;
}
```

这种情况下，我们选择在`ThreadRAII`的析构函数对异步执行的线程进行`join`，因为在先前分析中，`detach`可能导致噩梦般的调试过程。我们之前也分析了`join`可能会导致表现异常（坦率说，也可能调试困难），但是在未定义行为（`detach`导致），程序终止（使用原生`std::thread`导致），或者表现异常之间选择一个后果，可能表现异常是最好的那个。

**请记住：**

- 在所有路径上保证`thread`最终是不可结合的。
- 析构时`join`会导致难以调试的表现异常问题。
- 析构时`detach`会导致难以调试的未定义行为。
- 声明类数据成员时，最后声明`std::thread`对象。

### Item 38：Be aware of varying thread handle destructor behavior

#### 异步执行程序间的通信机制

在[Item37](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item37.html)中说明，可结合的`std::thread`析构会终止你的程序，因为两个其他的替代选择——隐式`join`或者隐式`detach`都是更加糟糕的。但是，*future*的析构表现有时就像执行了隐式`join`，有时又像是隐式执行了`detach`，有时又没有执行这两个选择。它永远不会造成程序终止。这个线程句柄多种表现值得研究一下。

我们可以观察到实际上*future*是通信信道的一端，被调用者通过该信道将结果发送给调用者。（[Item39](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item39.html)说，与*future*有关的这种通信信道也可以被用于其他目的。但是对于本条款，我们只考虑它们作为这样一个机制的用法，即被调用者传送结果给调用者。）被调用者（通常是异步执行）将计算结果写入通信信道中（通常通过`std::promise`对象），调用者使用*future*读取结果。你可以想象成下面的图示，虚线表示信息的流动方向：

![item38_fig1](../images/item38_fig1.png)

但是被调用者的结果存储在哪里？被调用者会在调用者`get`相关的*future*之前执行完成，所以结果不能存储在被调用者的`std::promise`。这个对象是局部的，当被调用者执行结束后，会被销毁。

结果同样不能存储在调用者的*future*，因为（当然还有其他原因）`std::future`可能会被用来创建`std::shared_future`（这会将被调用者的结果所有权从`std::future`转移给`std::shared_future`），而`std::shared_future`在`std::future`被销毁之后可能被复制很多次。鉴于不是所有的结果都可以被拷贝（即只可移动类型），并且结果的生命周期至少与最后一个引用它的*future*一样长，这些潜在的*future*中哪个才是被调用者用来存储结果的？

因为与被调用者关联的对象和与调用者关联的对象都不适合存储这个结果，所以必须存储在两者之外的位置。此位置称为**共享状态**（*shared state*）。共享状态通常是基于堆的对象，但是标准并未指定其类型、接口和实现。标准库的作者可以通过任何他们喜欢的方式来实现共享状态。

我们可以想象调用者，被调用者，共享状态之间关系如下图，虚线还是表示信息流方向：

![item38_fig2](../images/item38_fig2.png)

共享状态的存在非常重要，因为*future*的析构函数——这个条款的话题——取决于与*future*关联的共享状态。特别地，

- **引用了共享状态——使用`std::async`启动的未延迟任务建立的那个——的最后一个\*future\*的析构函数会阻塞住**，直到任务完成。本质上，这种*future*的析构函数对执行异步任务的线程执行了隐式的`join`。
- **其他所有\*future\*的析构函数简单地销毁\*future\*对象**。对于异步执行的任务，就像对底层的线程执行`detach`。对于延迟任务来说如果这是最后一个*future*，意味着这个延迟任务永远不会执行了。

#### future的一种例外行为

正常行为是*future*析构函数销毁*future*。就是这样。那意味着不`join`也不`detach`，也不运行什么，只销毁*future*的数据成员（当然，还做了另一件事，就是递减了共享状态中的引用计数，这个共享状态是由引用它的*future*和被调用者的`std::promise`共同控制的。这个引用计数让库知道共享状态什么时候可以被销毁。对于引用计数的一般信息参见[Item19](https://cntransgroup.github.io/EffectiveModernCppChinese/4.SmartPointers/item19.html)。）

正常行为的例外情况仅在某个`future`同时满足下列所有情况下才会出现：

- **它关联到由于调用`std::async`而创建出的共享状态**。
- **任务的启动策略是`std::launch::async`**（参见[Item36](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item36.html)），原因是运行时系统选择了该策略，或者在对`std::async`的调用中指定了该策略。
- **这个\*future\*是关联共享状态的最后一个\*future\***。对于`std::future`，情况总是如此，对于`std::shared_future`，如果还有其他的`std::shared_future`，与要被销毁的*future*引用相同的共享状态，则要被销毁的*future*遵循正常行为（即简单地销毁它的数据成员）。

只有当上面的三个条件都满足时，*future*的析构函数才会表现“异常”行为，就是在异步任务执行完之前阻塞住。实际上，这相当于对由于运行`std::async`创建出任务的线程隐式`join`。

**请记住：**

- *future*的正常析构行为就是销毁*future*本身的数据成员。
- 引用了共享状态——使用`std::async`启动的未延迟任务建立的那个——的最后一个*future*的析构函数会阻塞住，直到任务完成。

### Item 39: Consider `void` futures for one-shot event communication

#### 不同任务间进行事件通知的机制

一个明显的方案就是使用条件变量（*condition variable*，简称*condvar*）。如果我们将检测条件的任务称为**检测任务**（*detecting task*），对条件作出反应的任务称为**反应任务**（*reacting task*），策略很简单：反应任务等待一个条件变量，检测任务在事件发生时改变条件变量。代码如下：

```cpp
std::condition_variable cv;         //事件的条件变量
std::mutex m;                       //配合cv使用的mutex
```

检测任务中的代码不能再简单了：

```cpp
…                                   //检测事件
cv.notify_one();                    //通知反应任务
```

如果有多个反应任务需要被通知，使用`notify_all`代替`notify_one`，但是这里，我们假定只有一个反应任务需要通知。

反应任务的代码稍微复杂一点，因为在对条件变量调用`wait`之前，必须通过`std::unique_lock`对象锁住互斥锁。（在等待条件变量前锁住互斥锁对线程库来说是经典操作。通过`std::unique_lock`锁住互斥锁的需要仅仅是C++11 API要求的一部分。）概念上的代码如下：

```cpp
…                                       //反应的准备工作
{                                       //开启关键部分

    std::unique_lock<std::mutex> lk(m); //锁住互斥锁

    cv.wait(lk);                        //等待通知，但是这是错的！
    
    …                                   //对事件进行反应（m已经上锁）
}                                       //关闭关键部分；通过lk的析构函数解锁m
…                                       //继续反应动作（m现在未上锁）
```

这份代码的第一个问题就是有时被称为**代码异味**（*code smell*）的一种情况：即使代码正常工作，但是有些事情也不是很正确。在这个情况中，这种问题源自于使用互斥锁。互斥锁被用于保护共享数据的访问，但是可能检测任务和反应任务可能不会同时访问共享数据，比如说，检测任务会初始化一个全局数据结构，然后给反应任务用，如果检测任务在初始化之后不会再访问这个数据结构，而在检测任务表明数据结构准备完了之前反应任务不会访问这个数据结构，这两个任务在程序逻辑下互不干扰，也就没有必要使用互斥锁。但是条件变量的方法必须使用互斥锁，这就留下了令人不适的设计。

#### 条件变量存在的问题

即使你忽略掉这个问题，还有两个问题需要注意：

- **如果在反应任务`wait`之前检测任务通知了条件变量，反应任务会挂起**。为了能使条件变量唤醒另一个任务，任务必须等待在条件变量上。如果在反应任务`wait`之前检测任务就通知了条件变量，反应任务就会丢失这次通知，永远不被唤醒。

- **`wait`语句虚假唤醒**。线程API的存在一个事实（在许多语言中——不只是C++），等待一个条件变量的代码即使在条件变量没有被通知时，也可能被唤醒，这种唤醒被称为**虚假唤醒**（*spurious wakeups*）。正确的代码通过确认要等待的条件确实已经发生来处理这种情况，并将这个操作作为唤醒后的第一个操作。C++条件变量的API使得这种问题很容易解决，因为允许把一个测试要等待的条件的*lambda*（或者其他函数对象）传给`wait`。因此，可以将反应任务`wait`调用这样写：

  ```cpp
  cv.wait(lk, 
          []{ return whether the evet has occurred; });
  ```

  要利用这个能力需要反应任务能够确定其等待的条件是否为真。但是我们考虑的场景中，它正在等待的条件是检测线程负责识别的事件的发生情况。反应线程可能无法确定等待的事件是否已发生。这就是为什么它在等待一个条件变量！

在很多情况下，使用条件变量进行任务通信非常合适，但是也有不那么合适的情况。

#### 在检测任务设置的*future*上`wait`来避免使用条件变量，互斥锁和flag

一个替代方案是让反应任务通过在检测任务设置的*future*上`wait`来避免使用条件变量，互斥锁和flag。这可能听起来也是个古怪的方案。毕竟，[Item38](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item38.html)中说明了*future*代表了从被调用方到（通常是异步的）调用方的通信信道的接收端，这里的检测任务和反应任务没有调用-被调用的关系。然而，[Item38](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item38.html)中也说说明了发送端是个`std::promise`，接收端是个*future*的通信信道不是只能用在调用-被调用场景。这样的通信信道可以用在任何你需要从程序一个地方传递信息到另一个地方的场景。这里，我们用来在检测任务和反应任务之间传递信息，传递的信息就是感兴趣的事件已经发生。

方案很简单。检测任务有一个`std::promise`对象（即通信信道的写入端），反应任务有对应的*future*。当检测任务看到事件已经发生，设置`std::promise`对象（即写入到通信信道）。同时，`wait`会阻塞住反应任务直到`std::promise`被设置。

现在，`std::promise`和*futures*（即`std::future`和`std::shared_future`）都是需要类型参数的模板。形参表明通过通信信道被传递的信息的类型。在这里，没有数据被传递，只需要让反应任务知道它的*future*已经被设置了。我们在`std::promise`和*future*模板中需要的东西是表明通信信道中没有数据被传递的一个类型。这个类型就是`void`。检测任务使用`std::promise<void>`，反应任务使用`std::future<void>`或者`std::shared_future<void>`。当感兴趣的事件发生时，检测任务设置`std::promise<void>`，反应任务在*future*上`wait`。尽管反应任务不从检测任务那里接收任何数据，通信信道也可以让反应任务知道，检测任务什么时候已经通过对`std::promise<void>`调用`set_value`“写入”了`void`数据。

所以，有

```cpp
std::promise<void> p;                   //通信信道的promise
```

检测任务代码很简洁：

```cpp
…                                       //检测某个事件
p.set_value();                          //通知反应任务
```

反应任务代码也同样简单：

```cpp
…                                       //准备作出反应
p.get_future().wait();                  //等待对应于p的那个future
…                                       //对事件作出反应
```

像使用flag的方法一样，此设计不需要互斥锁，无论在反应线程调用`wait`之前检测线程是否设置了`std::promise`都可以工作，并且不受虚假唤醒的影响（只有条件变量才容易受到此影响）。

**请记住：**

- 对于简单的事件通信，基于条件变量的设计需要一个多余的互斥锁，对检测和反应任务的相对进度有约束，并且需要反应任务来验证事件是否已发生。
- 基于flag的设计避免的上一条的问题，但是是基于轮询，而不是阻塞。
- 条件变量和flag可以组合使用，但是产生的通信机制很不自然。
- 使用`std::promise`和*future*的方案避开了这些问题，但是这个方法使用了堆内存存储共享状态，同时有只能使用一次通信的限制。

### Item 40: Use `std::atomic` for concurrency, `volatile` for special memory

#### RMW操作对`std::atomic`在并发中有效而`volatile`无效

举一个关于在多线程程序中`std::atomic`和`volatile`表现不同的具体例子，考虑这样一个简单的计数器，通过多线程递增。我们把它们初始化为0：

```cpp
std::atomic<int> ac(0);         //“原子性的计数器”
volatile int vc(0);             //“volatile计数器”
```

然后我们在两个同时运行的线程中对两个计数器递增：

```cpp
/*----- Thread 1 ----- */        /*------- Thread 2 ------- */
        ++ac;                              ++ac;
        ++vc;                              ++vc;
```

当两个线程执行结束时，`ac`的值（即`std::atomic`的值）肯定是2，因为每个自增操作都是不可分割的（原子性的）。另一方面，`vc`的值，不一定是2，因为自增不是原子性的。每个自增操作包括了读取`vc`的值，增加读取的值，然后将结果写回到`vc`。这三个操作对于`volatile`对象不能保证原子执行，所有可能是下面的交叉执行顺序：

1. Thread1读取`vc`的值，是0。
2. Thread2读取`vc`的值，还是0。
3. Thread1将读到的0加1，然后写回到`vc`。
4. Thread2将读到的0加1，然后写回到`vc`。

`vc`的最后结果是1，即使看起来自增了两次。

不仅只有这一种可能的结果，通常来说`vc`的最终结果是不可预测的，因为`vc`会发生数据竞争，对于数据竞争造成未定义行为，标准规定表示编译器生成的代码可能是任何逻辑。当然，编译器不会利用这种行为来作恶。但是它们通常做出一些没有数据竞争的程序中才有效的优化，这些优化在存在数据竞争的程序中会造成异常和不可预测的行为。

#### `volatile`无法保证代码重排顺序

假定一个任务计算第二个任务需要的一个重要的值。当第一个任务完成计算，必须传递给第二个任务。[Item39](https://cntransgroup.github.io/EffectiveModernCppChinese/7.TheConcurrencyAPI/item39.html)表明一种使用`std::atomic<bool>`的方法来使第一个任务通知第二个任务计算完成。计算值的任务的代码如下：

```cpp
std::atomic<bool> valVailable(false); 
auto imptValue = computeImportantValue();   //计算值
valAvailable = true;                        //告诉另一个任务，值可用了
```

人类读这份代码，能看到在`valAvailable`赋值之前对`imptValue`赋值很关键，但是所有编译器看到的是给相互独立的变量的一对赋值操作。通常来说，编译器会被允许重排这对没有关联的操作。这意味着，给定如下顺序的赋值操作（其中`a`，`b`，`x`，`y`都是互相独立的变量），

```cpp
a = b;
x = y;
```

编译器可能重排为如下顺序：

```cpp
x = y;
a = b;
```

即使编译器没有重排顺序，底层硬件也可能重排（或者可能使它看起来运行在其他核心上），因为有时这样代码执行更快。

然而，`std::atomic`会限制这种重排序，并且这样的限制之一是，在源代码中，对`std::atomic`变量写之前不会有任何操作（或者操作发生在其他核心上）。（这只在`std::atomic`s使用**顺序一致性**（*sequential consistency*）时成立，对于使用在本书中展示的语法的`std::atomic`对象，这也是默认的和唯一的一致性模型。C++11也支持带有更灵活的代码重排规则的一致性模型。这样的**弱**（*weak*）（亦称**松散的**，*relaxed*）模型使构建一些软件在某些硬件构架上运行的更快成为可能，但是使用这样的模型产生的软件**更加**难改正、理解、维护。在使用松散原子性的代码中微小的错误很常见，即使专家也会出错，所以应当尽可能坚持顺序一致性。）这意味对我们的代码，

```cpp
auto imptValue = computeImportantValue();   //计算值
valAvailable = true;                        //告诉另一个任务，值可用了
```

编译器不仅要保证`imptValue`和`valAvailable`的赋值顺序，还要保证生成的硬件代码不会改变这个顺序。结果就是，将`valAvailable`声明为`std::atomic`确保了必要的顺序——其他线程看到的是`imptValue`值的改变不会晚于`valAvailable`。

将`valAvailable`声明为`volatile`不能保证上述顺序：

```cpp
volatile bool valVailable(false); 
auto imptValue = computeImportantValue();
valAvailable = true;                        //其他线程可能看到这个赋值操作早于imptValue的赋值操作
```

这份代码编译器可能将`imptValue`和`valAvailable`赋值顺序对调，如果它们没这么做，可能不能生成机器代码，来阻止底部硬件在其他核心上的代码看到`valAvailable`更改在`imptValue`之前。

这两个问题——不保证操作的原子性以及对代码重排顺序没有足够限制——解释了为什么`volatile`在多线程编程中没用，但是没有解释它应该用在哪。简而言之，它是用来告诉编译器，它们处理的内存有不正常的表现。

`volatile`

编译器本身需要拜托重复读写的代码（技术上称为**冗余访问**（*redundant loads*）和**无用存储**（*dead stores*））

比如在这段代码中：

```cpp
x = 10;                                 //写x
x = 20;                                 //再次写x
```

如果`x`与无线电发射器的控制端口关联，则代码是给无线电发指令，10和20意味着不同的指令。优化掉第一条赋值会改变发送到无线电的指令流。

`volatile`是告诉编译器我们正在处理特殊内存。意味着告诉编译器“不要对这块内存执行任何优化”。所以如果`x`对应于特殊内存，应该声明为`volatile`：

```cpp
volatile int x;
```

考虑对我们的原始代码序列有何影响：

```cpp
auto y = x;                             //读x
y = x;                                  //再次读x（不会被优化掉）

x = 10;                                 //写x（不会被优化掉）
x = 20;                                 //再次写x
```

如果`x`是内存映射的（或者已经映射到跨进程共享的内存位置等），这正是我们想要的。

**请记住：**

- `std::atomic`用于在不使用互斥锁情况下，来使变量被多个线程访问的情况。是用来编写并发程序的一个工具。
- `volatile`用在读取和写入不应被优化掉的内存上。是用来处理特殊内存的一个工具。