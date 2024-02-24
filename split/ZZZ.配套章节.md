

# <a id="s-a"></a>A: 架构设计的观念

本部分包括有关高层次的架构性观念和程序库的观念。

架构性规则概览：

* [A.1: 分离稳定的代码和不稳定的代码](#ra-stable)
* [A.2: 将潜在可复用的部分作为程序库](#ra-lib)
* [A.4: 程序库之间不能有循环依赖](#ra-dag)
* [???](#???)
* [???](#???)
* [???](#???)
* [???](#???)
* [???](#???)
* [???](#???)

### <a id="ra-stable"></a>A.1: 分离稳定的代码和不稳定的代码

对较不稳定的代码进行隔离，有助于其单元测试，接口改进，重构，以及最终弃用。

### <a id="ra-lib"></a>A.2: 将潜在可复用的部分作为程序库

##### 理由

##### 注解

程序库是一些共同进行维护，文档化，并发布的声明式和定义式的集合体。
程序库可以是一组头文件（“仅有头文件的程序库”），或者一组头文件加上一组目标文件构成。
你可以静态或动态地将程序库连接到程序中，或者你还可以 `#included` 仅头文件的库。


### <a id="ra-dag"></a>A.4: 程序库之间不能有循环依赖

##### 理由

* 循环依赖导致构建过程变得复杂。
* 循环依赖难于理解，可能会引入不确定性（未定义行为）。

##### 注解

一个程序库可以在它的组件的定义之间包含循环引用。
例如：

    ???

不过，程序库不能对依赖于它的其他程序库产生依赖。


# <a id="s-not"></a>NR: 伪规则和错误的看法

本部分包含一些在不少地方流行的规则和指导方针，但是我们慎重地建议不要采纳它们。
我们完全了解这些规则曾经在某些时间和场合是有意义的，而且我们自己也曾经采用过它们。
不过，在我们所推荐并以各项指导方针所支持的编程风格的情况中，这些“伪规则”是有害的。

即便是今天，仍有一些情况下这些规则是有意义的。
比如说，缺少合适的工具支持会导致异常在硬实时系统中的不适用，
但请不要盲目地信任“通俗智慧”（比如有关“效率”的未经数据支持的观点）；
这种“智慧”也许是基于几十年前的信息，或者是来自于与 C++ 有非常不同性质的语言的经验
（比如 C 或者 Java）。

对于这些伪规则的替代方案的正面观点都在各个规则的“替代方案”部分中给出。

伪规则概览：

* [NR.1: 请勿坚持认为声明都应当放在函数的最上面](#rnr-top)
* [NR.2: 请勿坚持使函数中只保留一个 `return` 语句](#rnr-single-return)
* [NR.3: 请勿避免使用异常](#rnr-no-exceptions)
* [NR.4: 请勿坚持把每个类定义放在其自己的源文件中](#rnr-lots-of-files)
* [NR.5: 请勿采用两阶段初始化](#rnr-two-phase-init)
* [NR.6: 请勿把所有清理操作放在函数末尾并使用 `goto exit`](#rnr-goto-exit)
* [NR.7: 请勿使所有数据成员 `protected`](#rnr-protected-data)
* ???

### <a id="rnr-top"></a>NR.1: 请勿坚持认为声明都应当放在函数的最上面

##### 理由

“所有声明都在开头”的规则，是来自不允许在语句之后对变量和常量进行初始化的老编程语言的遗产。
这样做会导致更长的程序，以及更多由于未初始化的或者错误初始化的变量所导致的错误。

##### 示例，不好

    int use(int x)
    {
        int i;
        char c;
        double d;

        // ... 做一些事 ...

        if (x < i) {
            // ...
            i = f(x, d);
        }
        if (i < x) {
            // ...
            i = g(x, c);
        }
        return i;
    }

未初始化变量和其使用点的距离越长，出现 BUG 的机会就越大。
幸运的是，编译器可以发现许多“设值前使用”的错误。
不幸的是，编译器无法捕捉到所有这样的错误，而且一些 BUG 并不都像这个小例子中的这样容易发现。


##### 替代方案

* [坚持为对象进行初始化](#res-always)。
* [ES.21: 不要在确实需要使用变量（或常量）之前就引入它](#res-introduce)。

### <a id="rnr-single-return"></a>NR.2: 请勿坚持使函数中只保留一个 `return` 语句

##### 理由

单返回规则会导致不必要地复杂的代码，并引入多余的状态变量。
特别是，单返回规则导致更难在函数开头集中进行错误检查。

##### 示例

    template<class T>
    //  requires Number<T>
    string sign(T x)
    {
        if (x < 0)
            return "negative";
        if (x > 0)
            return "positive";
        return "zero";
    }

为仅使用一个返回语句，我们得做类似这样的事：

    template<class T>
    //  requires Number<T>
    string sign(T x)        // 不好
    {
        string res;
        if (x < 0)
            res = "negative";
        else if (x > 0)
            res = "positive";
        else
            res = "zero";
        return res;
    }

这不仅更长，而且很可能效率更差。
越长越复杂的函数，对其进行变通就越是痛苦。
当然许多简单的函数因为它们本来就简单的逻辑都天然就只有一个 `return`。

##### 示例

    int index(const char* p)
    {
        if (!p) return -1;  // 错误指标：替代方案是 "throw nullptr_error{}"
        // ... 进行查找以找出 p 的索引
        return i;
    }

如果我们采纳这条规则的话，得做类似这样的事：

    int index2(const char* p)
    {
        int i;
        if (!p)
            i = -1;  // 错误指标
        else {
            // ... 进行查找以找出 p 的索引
        }
        return i;
    }

注意我们（故意地）违反了禁止未初始化变量的规则，因为这种风格通常都会导致这样。
而且，这种风格也会倾向于采用 [goto exit](#rnr-goto-exit) 伪规则。

##### 替代方案

* 保持函数短小简单。
* 随意使用多个 `return` 语句（以及抛出异常）。

### <a id="rnr-no-exceptions"></a>NR.3: 请勿避免使用异常

##### 理由

一般有四种主要的不用异常的理由：

* 异常是低效的
* 异常会导致泄漏和错误
* 异常的性能无法预测
* 异常处理的运行时支持耗费过多空间

我们没有能够满足所有人的解决这个问题的办法。
无论如何，针对异常的讨论已经持续了四十多年了。
一些语言没有异常就无法使用，而另一些并不支持异常。
这在使用和不使用异常的方面都造成了强大的传统，并导致激烈的争论。

不过，我们可以简要说明，为什么我们认为对于通用编程以及这里的指导方针的情况来说，
异常是最佳的候选方案。
简单的论据，无论支持还是反对，都是缺乏说服力的。
确实存在一些特殊的应用，其中异常就是不合适的。
（例如，硬实时系统，且缺乏可靠的对于异常处理的耗费进行估计的支持）。

我们依次来考虑针对异常的主要反对观点：

* 异常是低效的：
和什么相比？
当进行比较时，请确保处理了同样的错误集合，并且它们都进行了等价的处理。
尤其是，不要对一个见到异常就立刻终止的程序和一个在记录错误日志之前
小心地进行资源清理的程序之间进行比较。
确实，某些系统的异常处理实现很糟糕；有时候，这样的实现迫使我们使用
其他错误处理方案，但这并不是异常的基本问题。
当使用某个有效的论据时——无论什么样的上下文——请小心你能拿出确实提供了所讨论的问题的
内部情况的健全的数据。
* 异常会导致泄漏和错误。
不会。
如果你的程序时一大堆乱糟糟的指针而没有总体的资源管理策略，
那么无论你干什么都会有问题。
如果你的系统是由上百万行这样的代码构成的，
那你可能是无法使用异常的，
不过这是一个有关过度和放纵的使用指针的问题，而不是异常的问题。
我们的观点是，你需要用 RAII 来让基于异常的错误处理变得简单且安全——比其他方案都要更简单和安全。
* 异常的性能无法预测。
如果你是在硬实时系统上，而你必须确保一个任务要在给定的时间内完成，
你需要一些工具来支撑这样的保证。
就我们所知，还没有出现这样的工具（至少对大多数程序员没有）。
* 异常处理的运行时支持耗费过多空间
小型（通常为嵌入式）系统中可能如此。
不过在放弃异常之前，请考虑采用统一的利用错误码的错误处理将耗费的空间有多少，
以及错误未被捕获将造成的损失由多少。

许多（可能是大多数）的和异常有关的问题都源自于需要和杂乱的老代码进行交互的历史性原因。

而支持使用异常的基本论点是：

* 它们把错误返回和普通返回进行了清晰的区分
* 它们无法被忘记或忽略
* 它们可以系统化地使用

请记住

* 异常是用于报告错误的（C++ 中；其他语言可能有异常的不同用法）。
* 异常不是用于可以局部处理的错误的。
* 不要试图在每个函数中捕获每一种异常（这样做是冗长的，臃肿的，而且会导致代码缓慢）。
* 异常不是用于那些当发生无法恢复的错误之后需要立即终止模块或系统的错误的。

##### 示例

    ???

##### 替代方案

* [RAII](#re-raii)
* 契约/断言：使用 GSL 的 `Expects` 和 `Ensures`（直到对契约的语言支持可以使用）

### <a id="rnr-lots-of-files"></a>NR.4: 请勿坚持把每个类定义放在其自己的源文件中

##### 理由

将每个类都放进其自己的文件所导致的文件数量难于管理，并会拖慢编译过程。
单个的类很少是一种良好的维护和发布的逻辑单位。

##### 示例

    ???

##### 替代方案

* 使用命名空间来包含逻辑上聚合的类和函数。

### <a id="rnr-two-phase-init"></a>NR.5: 请勿采用两阶段初始化

##### 理由

将初始化拆分为两步会导致不变式的弱化，
更复杂的代码（必须处理半构造对象），
以及错误（当未能一致地正确处理半构造对象时）。

##### 示例，不好

    // 老式传统风格：有许多问题

    class Picture
    {
        int mx;
        int my;
        int * data;
    public:
        // 主要问题：构造函数未进行完全构造
        Picture(int x, int y)
        {
            mx = x;         // 也不好：在构造函数体中而非
                            // 成员初始化式中进行赋值
            my = y;
            data = nullptr; // 也不好：在构造函数中而非
                            // 成员初始化式中进行常量初始化
        }

        ~Picture()
        {
            Cleanup();
        }

        // ...

        // 不好：两阶段初始化
        bool Init()
        {
            // 不变式检查
            if (mx <= 0 || my <= 0) {
                return false;
            }
            if (data) {
                return false;
            }
            data = (int*) malloc(mx*my*sizeof(int));   // 也不好：拥有原始指针，还用了 malloc
            return data != nullptr;
        }

        // 也不好：没有理由让清理操作作为单独的函数
        void Cleanup()
        {
            if (data) free(data);
            data = nullptr;
        }
    };

    Picture picture(100, 0); // 此时 picture 尚未就绪可用
    // 这里将失败
    if (!picture.Init()) {
        puts("Error, invalid picture");
    }
    // 现在有一个无效的 picture 对象实例。

##### 示例，好

    class Picture
    {
        int mx;
        int my;
        vector<int> data;

        static int check_size(int size)
        {
            // 不变式检查
            Expects(size > 0);
            return size;
        }

    public:
        // 更好的方式是以一个 2D 的 Size 类作为单个形参
        Picture(int x, int y)
            : mx(check_size(x))
            , my(check_size(y))
            // 现在已知 x 和 y 为有效的大小
            , data(mx * my) // 出错时将抛出 std::bad_alloc
        {
            // 图片就绪可用
        }

        // 编译器生成的析构函数会完成工作。（另见 C.21）

        // ...
    };

    Picture picture1(100, 100);
    // picture1 已就绪可用……

    // y 并非有效大小值，
    // 缺省的契约违规行为将会调用 std::terminate
    Picture picture2(100, 0);
    // 不会抵达这里……

##### 替代方案

* 始终在构造函数中建立类不变式。
* 不要在需要对象之前就定义它。

### <a id="rnr-goto-exit"></a>NR.6: 请勿把所有清理操作放在函数末尾并使用 `goto exit`

##### 理由

`goto` 是易错的。
这种技巧是进行 RAII 式的资源和错误处理的前异常时代的技巧。

##### 示例，不好

    void do_something(int n)
    {
        if (n < 100) goto exit;
        // ...
        int* p = (int*) malloc(n);
        // ...
        if (some_error) goto_exit;
        // ...
    exit:
        free(p);
    }

请找出其中的 BUG。

##### 替代方案

* 使用异常和 [RAII](#re-raii)
* 对于非 RAII 资源，使用 [`finally`](#re-finally)。

### <a id="rnr-protected-data"></a>NR.7: 请勿使所有数据成员 `protected`

##### 理由

`protected` 数据是一种错误来源。
`protected` 数据可以被各种地方的无界限数量的代码所操纵。
`protected` 数据是在类层次中等价于全局对象的东西。

##### 示例

    ???

##### 替代方案

* [使成员数据 `public` 或者（更好地）`private`](#rh-protected)。


# <a id="s-references"></a>RF: 参考材料

已经为 C++，尤其是对 C++ 的使用编写过了许多的编码标准、规则和指导方针。
它们中许多都

* 关注的是低级问题，比如标识符的拼写
* 是由 C++ 的新手编写的
* 将“禁止程序员作出不常见行为”作为其首要目标
* 将维持许多编译器的可移植性作为目标（有些已经是 10 年前的了）
* 是为了维持好几十年的代码库而编写的
* 是仅关注单一的应用领域的
* 只会产生反效果
* 被忽略了（程序员为了完成工作不得不忽略它们）

不良的编码标准要比没有编码标准还要差。
不过一组恰当的指导方针比没有标准要好得多：“形式即解放。”

我们为什么不能有一种允许所有我们想要的同时又禁止所有我们不期望的东西的语言（“完美的语言”）呢？
本质上说，这是由于可负担的语言（及其工具链）同时也要为那些需求与你不同的人提供服务，并且要为你今后比今天更多的需求提供服务。
而且，你的需求会随时间而改变，而为此你则需要采用一种通用语言。
今日貌似理想的语言在未来可能会变得过于受限了。

编码指导方针可以使语言能够适应于特定的需求。
因此，并不存在适用于每个人的单一编码风格。
我们预计不同的组织会提供更具限制性和更严格的编码风格附加规定。

参考材料部分：

* [RF.rules: 编码规则](#ss-rules)
* [RF.books: 带有编码指导方针的书籍](#ss-books)
* [RF.C++: C++ 编程 (C++11/C++14/C++17)](#ss-cplusplus)
* [RF.web: 网站](#ss-web)
* [RS.video: 有关“当代 C++”的视频](#ss-vid)
* [RF.man: 手册](#ss-man)
* [RF.core: 核心指导方针相关材料](#ss-core)

## <a id="ss-rules"></a>RF.rules: 编码规则

* [AUTOSAR Guidelines for the use of the C++14 language in critical and safety-related systems v17.10](https://www.autosar.org/fileadmin/user_upload/standards/adaptive/17-10/AUTOSAR_RS_CPP14Guidelines.pdf)
* [Boost Library Requirements and Guidelines](http://www.boost.org/development/requirements.html).
  ???.
* [Bloomberg: BDE C++ Coding](https://github.com/bloomberg/bde/wiki/CodingStandards.pdf).
  着重强调了代码的组织和布局。
* Facebook: ???
* [GCC Coding Conventions](https://gcc.gnu.org/codingconventions.html).
  C++03 以及（相当）一部分向后兼容。
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
  面向 C++17 和（同样）较老的代码库。Google 的专家们现在正展开活跃的合作，以改进这里的各项指导方针，有希望能够合并这些成果，以使它们能够成为他们也同样推荐采纳的一组现代的通用指导方针。
* [JSF++: JOINT STRIKE FIGHTER AIR VEHICLE C++ CODING STANDARDS](http://www.stroustrup.com/JSF-AV-rules.pdf).
  文档编号 2RDU00001 Rev C. December 2005.
  针对飞行控制软件。
  针对硬实时。
  这意味着它需要非常多的限制（“程序如果发生故障就会有人挂掉”）。
  例如，飞机起飞后禁止进行任何自由存储的分配和回收（禁止内存溢出并禁止发生碎片化）。
  禁止使用异常（因为没有可用工具可以保证异常能够在固定的短时间段内被处理）。
  所使用的程序库必须是已被证明可以用于关键任务应用的。
  它和这个指导方针集合的相似性并不让人惊讶，因为 Bjarne Stroustrup 正是 JSF++ 的作者之一。
  建议采纳，但请注意其非常特定的关注领域。
* [MISRA C++ 2008: Guidelines for the use of the C++ language in critical systems] (https://www.misra.org.uk/Buyonline/tabid/58/Default.aspx)。
* [Using C++ in Mozilla Code](https://firefox-source-docs.mozilla.org/code-quality/coding-style/using_cxx_in_firefox_code.html).
  如其名称所示，它关注于跨许多（老）编译器的兼容性。
  因此，它是很具有限制性的。
* [Geosoft.no: C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html).
  ???.
* [Possibility.com: C++ Coding Standard](http://www.possibility.com/Cpp/CppCodingStandard.html).
  ???.
* [SEI CERT: Secure C++ Coding Standard](https://wiki.sei.cmu.edu/confluence/x/Wnw-BQ).
  针对安全关键代码所编写的一组非常好的规则（还带有示例和原理说明）。
  它们的许多规则都广泛适用。
* [High Integrity C++ Coding Standard](http://www.codingstandard.com/).
* [llvm](http://llvm.org/docs/CodingStandards.html).
  有些简略，基于 C++14，而且是（有理由地）针对其应用领域的。
* ???

## <a id="ss-books"></a>RF.books: 带有编码指导方针的书籍

* [Meyers96](#meyers96) Scott Meyers: *More Effective C++*. Addison-Wesley 1996.
* [Meyers97](#meyers97) Scott Meyers: *Effective C++, Second Edition*. Addison-Wesley 1997.
* [Meyers01](#meyers01) Scott Meyers: *Effective STL*. Addison-Wesley 2001.
* [Meyers05](#meyers05) Scott Meyers: *Effective C++, Third Edition*. Addison-Wesley 2005.
* [Meyers15](#meyers15) Scott Meyers: *Effective Modern C++*. O'Reilly 2015.
* [SuttAlex05](#suttalex05) Sutter and Alexandrescu: *C++ Coding Standards*. Addison-Wesley 2005. 与其说是一组规则，不如说是一组元规则。前 C++11 时代。
* [Stroustrup05](#stroustrup05) Bjarne Stroustrup: [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
  LCSD05. October 2005.
* [Stroustrup14](#stroustrup05) Stroustrup: [A Tour of C++](http://www.stroustrup.com/Tour.html).
  Addison Wesley 2014.
  每章的结尾都有一个包含一组建议的忠告部分。
* [Stroustrup13](#stroustrup13) Stroustrup: [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html).
  Addison Wesley 2013.
  每章的结尾都有一个包含一组建议的忠告部分。
* Stroustrup: [Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
  for [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).
  大多是一些低级的命名和代码布局规则。
  主要作为教学工具。

## <a id="ss-cplusplus"></a>RF.C++: C++ 编程 (C++11/C++14)

* [TC++PL4](http://www.stroustrup.com/4th.html):
  面向有经验的程序员的，对 C++ 语言和标准库的全面彻底的描述。
* [Tour++](http://www.stroustrup.com/Tour.html):
  面向有经验的程序员的，对 C++ 语言和标准库的简介。
* [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html):
  面向初学者和新手们的教材。

## <a id="ss-web"></a>RF.web: 网站

* [isocpp.org](https://isocpp.org)
* [Bjarne Stroustrup 的个人主页](http://www.stroustrup.com)
* [WG21](http://www.open-std.org/jtc1/sc22/wg21/)
* [Boost](http://www.boost.org)<a id="boost"></a>
* [Adobe open source](https://opensource.adobe.com/)
* [Poco libraries](http://pocoproject.org/)
* Sutter's Mill?
* ???

## <a id="ss-vid"></a>RS.video: 有关“当代 C++”的视频

* Bjarne Stroustrup: [C++11?Style](http://channel9.msdn.com/Events/GoingNative/GoingNative-2012/Keynote-Bjarne-Stroustrup-Cpp11-Style). 2012.
* Bjarne Stroustrup: [The Essence of C++: With Examples in C++84, C++98, C++11, and?C++14](http://channel9.msdn.com/Events/GoingNative/2013/Opening-Keynote-Bjarne-Stroustrup). 2013
* [CppCon '14](https://isocpp.org/blog/2014/11/cppcon-videos-c9) 的全部演讲
* Bjarne Stroustrup: [The essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE) 在爱丁堡大学。2014
* Bjarne Stroustrup: [The Evolution of C++ Past, Present and Future](https://www.youtube.com/watch?v=_wzc7a3McOs). CppCon 2016 keynote.
* Bjarne Stroustrup: [Make Simple Tasks Simple!](https://www.youtube.com/watch?v=nesCaocNjtQ). CppCon 2014 keynote.
* Bjarne Stroustrup: [Writing Good C++14](https://www.youtube.com/watch?v=1OEu9C51K2A). CppCon 2015 keynote about the Core Guidelines.
* Herb Sutter: [Writing Good C++14... By Default](https://www.youtube.com/watch?v=hEx5DNLWGgA). CppCon 2015 keynote about the Core Guidelines.
* CppCon 15
* ??? C++ Next
* ??? Meting C++
* ??? more ???

## <a id="ss-man"></a>RF.man: 手册

* ISO C++ Standard C++11.
* ISO C++ Standard C++14.
* [ISO C++ Standard C++17](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4606.pdf). 委员会草案。
* [Palo Alto "Concepts" TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).
* [ISO C++ Concepts TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
* [WG21 Ranges report](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf). 草案。


## <a id="ss-core"></a>RF.core: 核心指导方针相关材料

这个部分包含一些用于展示核心指导方针及其背后的思想的有用材料：

* [Our documents directory](https://github.com/isocpp/CppCoreGuidelines/tree/master/docs)
* Stroustrup, Sutter, and Dos Reis: [A brief introduction to C++’s model for type- and resource-safety](http://www.stroustrup.com/resource-model.pdf). A paper with lots of examples.
* Sergey Zubkov: [a Core Guidelines talk](https://www.youtube.com/watch?v=DyLwdl_6vmU)
and here are the [slides](http://2017.cppconf.ru/talks/sergey-zubkov). In Russian. 2017.
* Neil MacIntosh: [The Guideline Support Library: One Year Later](https://www.youtube.com/watch?v=_GhNnCuaEjo). CppCon 2016.
* Bjarne Stroustrup: [Writing Good C++14](https://www.youtube.com/watch?v=1OEu9C51K2A). CppCon 2015 keynote.
* Herb Sutter: [Writing Good C++14... By Default](https://www.youtube.com/watch?v=hEx5DNLWGgA). CppCon 2015 keynote.
* Peter Sommerlad: [C++ Core Guidelines - Modernize your C++ Code Base](https://www.youtube.com/watch?v=fQ926v4ZzAM). ACCU 2017.
* Bjarne Stroustrup: [No Littering!](https://www.youtube.com/watch?v=01zI9kV4h8c). Bay Area ACCU 2016.
It gives some idea of the ambition level for the Core uidelines.

CppCon 的展示的幻灯片是可以获得的（其链接，还有上传的视频）。

极大欢迎对于这个列表的贡献。

## <a id="ss-ack"></a>鸣谢

感谢对规则、建议、支持信息和参考材料等等作出了各种贡献的许多人：

* Peter Juhl
* Neil MacIntosh
* Axel Naumann
* Andrew Pardoe
* Gabriel Dos Reis
* Zhuang, Jiangang (Jeff)
* Sergey Zubkov

请查看 github 的贡献者列表。

# <a id="s-profile"></a>Pro: 剖面配置

理想情况，我们应当遵循所有这些指导方针。
这样能够得到最简洁，最规范，最少易错性，而且通常是最快的代码。
不幸的是这通常是不可能的，因为我们不得不让代码适合于大型的代码库并使用一些现存的程序库。
常常是，这样的代码已经编写好几十年了，并且并不遵循这些指导方针。
我们必须以[渐次采纳](#s-modernizing)为目标。

无论采用何种渐次采纳的策略，我们都应当能够首先采用一些相关指导方针的集合来
处理某些问题的集合，遗留其他的以后处理。
当发现某些而不是全部的指导方针对于代码库有关时，以及当在某个专门化的应用领域
采用一组专门化的指导方针的集合时，会出现类似的“相关指导方针”的主意。
我们称这样的相关指导方针的集合为一个“剖面配置”。
我们对这种指导方针集合的目标是其内聚性，它们一起可以有助于我们达成某个特定的目标，如“消除范围错误”
或“静态类型安全性”。
每个剖面配置都被设计用于消除一个类别的错误。
而“随意”实施一些独立的规则，相对于提供确定的改善来说，更像是对代码库的破坏。

“剖面配置”是确定的并且可移植实施的规则（限制）子集，它们是专门设计以达成某项特定保证的。
“确定性”意味着它们仅需要进行局部分析，并且可以在一台计算机中进行实现（虽然并不比如此）。
“可移植实施性”表明它们和语言规则相似，因而程序员们可以期望不同实施工具对于相同的代码给出相同的答案。

编写成在这样的语言剖面配置下仍免于警告的代码，可以认为是遵循这个剖面配置的。
而遵从的代码则可以认为对于该剖面配置的目标安全属性来说是安全的。
遵从的代码不会成为这种性质的错误的根本原因，
虽然程序中可能从其他的代码引入这样的错误。
剖面配置还可能会引入一些额外的库类型，以简化遵从性并鼓励编写正确的代码。

剖面配置概览：

* [Pro.type: 类型安全性](#ss-type)
* [Pro.bounds: 边界安全性](#ss-bounds)
* [Pro.lifetime: 生存期安全性](#ss-lifetime)

未来，我们打算定义更多的剖面配置，并向现有剖面配置中添加更多的检查。
候选者有：

* 窄化算术提升和转换（可能会成为一个单独的安全算术剖面配置的一部分）
* 从负浮点数向无符号整型类型进行算术强制转换（同上）
* 经选择的未定义行为：从 Gabriel Dos Reis 为 WG21 研究小组开发的 UB 列表入手
* 经选择的未指明行为：处理可移植性问题。
* `const` 违反：大多数情况已经由编译器完成，但我们可以捕捉不适当的强制转换和 `const` 的不当使用。

剖面配置的开启是由实现所定义的；典型情况下，是在分析工具之中进行的设置。

要抑制对某个剖面配置检查，可以在语言构造上放一个 `suppress` 标注。例如：

    [[suppress(bounds)]] char* raw_find(char* p, int n, char x)    // 在 p[0]..p[n - 1] 中寻找 x
    {
        // ...
    }

这样 `raw_find()` 就可以在内存中到处爬了。
显然，进行抑制应当是非常罕见的。

## <a id="ss-type"></a>Pro.safety: 类型安全性剖面配置

这个剖面配置将能够简化正确使用类型的代码编写，并避免因疏忽产生类型双关。
它是关注于移除各种主要的类型违例的因素（包括对强制转换和联合的不安全使用）而达成这点的。

针对本部分的目的而言，
类型安全性被定义为这样的性质：对变量的使用不会不遵守其所定义的类型的规则。
通过类型 `T` 所访问的内存，不应该是某个实际上包含了无关类型 `U` 的对象的有效内存。
注意，当和[边界安全性](#ss-bounds)、[生存期安全性](#ss-lifetime)组合起来时，安全性才是完整的。

这个剖面配置的实现应当在源代码中识别出下列模式，将之作为不符合并给出诊断信息。

类型安全性剖面配置概览：

* <a id="pro-type-avoidcasts"></a>Type.1: [避免强制转换](#res-casts)：

  1. <a id="pro-type-reinterpretcast"></a>请勿使用 `reinterpret_cast`；此为[避免强制转换](#res-casts)和[优先使用具名的强制转换](#res-casts-named)的严格的版本。
  2. <a id="pro-type-arithmeticcast"></a>请勿在算术类型上使用 `static_cast`；此为[避免强制转换](#res-casts)和[优先使用具名的强制转换](#res-casts-named)的严格的版本。
  3. <a id="pro-type-identitycast"></a>当源指针类型和目标类型相同时，请勿进行指针强制转换；此为[避免强制转换](#res-casts)的严格的版本。
  4. <a id="pro-type-implicitpointercast"></a>当指针转换可以隐式转换时，请勿使用指针强制转换；此为[避免强制转换](#res-casts)的严格的版本。
* <a id="pro-type-downcast"></a>Type.2: 请勿使用 `static_cast` 进行向下强制转换：
[代之以使用 `dynamic_cast`](#rh-dynamic_cast)。
* <a id="pro-type-constcast"></a>Type.3: 请勿使用 `const_cast` 强制掉 `const`（亦即不要这样做）：
[不要强制掉 `const`](#res-casts-const)。
* <a id="pro-type-cstylecast"></a>Type.4: 请勿使用  C 风格的强制转换 `(T)expression` 和函数式风格强制转换 `T(expression)`：
优先使用[构造语法](#res-construct)，[具名的强制转换](#res-casts-named)，或 `T{expression}`。
* <a id="pro-type-init"></a>Type.5: 请勿在初始化之前使用变量：
[坚持进行初始化](#res-always)。
* <a id="pro-type-memberinit"></a>Type.6: 坚持初始化成员变量：
[坚持进行初始化](#res-always)，
可以采用[默认构造函数](#rc-default0)或者
[默认成员初始化式](#rc-in-class-initializer)。
* <a id="pro-type-union"></a>Type.7: 避免裸 union：
[代之以使用 `variant`](#ru-naked)。
* <a id="pro-type-varargs"></a>Type.8: 避免 varargs：
[不要使用 `va_arg` 参数](#f-varargs)。

##### 影响

在类型安全性剖面配置下，你可以相信每个操作都将在有效的对象上进行。
可能抛出异常以报告无法（在编译时）被静态地检测到的错误。
要注意的是，这种类型安全性仅当我们同样具有[边界安全性](#ss-bounds)和[生存期安全性](#ss-lifetime)时才是完整的。
而没有这些保证的话，一个内存区域可能以与其所存储的单个或多个对象，或对象的一部分无关的方式被访问。


## <a id="ss-bounds"></a>Pro.bounds: 边界安全性剖面配置

这个剖面配置将能简化对于在分配的内存块的边界之中进行操作的编码工作。
它是通过关注于移除边界违例的主要根源——即指针算术和数组索引——而做到这点的。
这个剖面配置的核心功能之一就是限制指针只能指向单个对象而不是数组。

我们将边界安全性定义为这样一种性质：程序不通过一个对象来对分配给这个对象的内存范围之外的内存进行访问。
仅当边界安全性与[类型安全性](#ss-type)和[生存期安全性](#ss-lifetime)组合起来时才是完整的，
它们还会包含其他允许发生边界违例的不安全操作。

边界安全性剖面配置概览：

* <a id="pro-bounds-arithmetic"></a>Bounds.1: 请勿使用指针算术。请使用 `span` 代替：
[（仅）传递单个对象的指针](#ri-array)，并[保持指针算术的简单性](#res-ptr)。
* <a id="pro-bounds-arrayindex"></a>Bounds.2: 仅使用常量表达式对数组进行索引操作：
[（仅）传递单个对象的指针](#ri-array)，并[保持指针算术的简单性](#res-ptr)。
* <a id="pro-bounds-decay"></a>Bounds.3: 避免数组向指针的退化：
[（仅）传递单个对象的指针](#ri-array)，并[保持指针算术的简单性](#res-ptr)。
* <a id="pro-bounds-stdlib"></a>Bounds.4: 请勿使用不进行边界检查的标准库函数和类型：
[以类型安全的方式使用标准库](#rsl-bounds)

##### 影响

边界安全性意味着，当访问对象（尤其是数组）时不会越过对象的内存分配范围。
这消除了一大类的隐伏且难于发现的错误，包括（不）著名的“缓冲区溢出”错误。
这避免了安全漏洞，以及（当越界写入时发生）内存损坏错误的大量来源。
即使越界访问“只是读取操作”，它也可能导致不变式的违反（当所访问的不是预期的类型时）
和“神秘的值”。


## <a id="ss-lifetime"></a>Pro.lifetime: 生存期安全性剖面配置

通过已经不指向任何东西的指针进行访问，是错误的一种主要来源，
而且在许多传统的 C 或 C++ 风格的编程中这很难避免。
例如，指针可能未初始化，值为 `nullptr`，指向越过指针范围，或者指向已删除的对象。

[参见此处的设计说明书的当前版本](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Lifetime.pdf)

生存期安全性剖面配置概览：

* <a id="pro-lifetime-invalid-deref"></a>Lifetime.1: 不要解引用无效指针：
[检测或避免](#res-deref)。

##### 影响

一旦强制实施了编码风格规则，静态分析，以及程序库支持的组合方案之后，本剖面配置将能

* 消除 C++ 中的恶劣错误的一种主要来源
* 消除潜在安全漏洞的一种主要来源
* 通过消除多余的“偏执”检查而改善性能
* 提升代码正确性的信心
* 通过强制遵循一种关键的 C++ 语言规则而避免未定义的行为


# <a id="s-gsl"></a>GSL: 指导方针支持库

GSL 是一个小型的程序库，其中的设施被设计用于支持本指导方针。
不使用这些设施的话，这些指导方针不得不变得对语言细节过于限制。

核心指导方针支持库是定义在 `gsl` 命名空间中的，其中的名字可能是对标准库和其他著名程序库的名字的别名。通过 `gsl` 命名空间进行的（编译期）间接，使我们可以进行试验，以及对这些支持设施提供局部变体。

GSL 只有头文件，可以在 [GSL: 指导方针支持库](https://github.com/Microsoft/GSL)找到。
支持库中的设施被设计为极为轻量化（零开销），它们相比于传统方案并不会带来任何开销。
当需要时，它们还可以用其他功能“工具化”（比如一些检查）来帮助进行诸如调试等任务。

各指导方针中，除了使用 GSL 中的类型之外，还使用了标准程序库（如 C++17）中的类型。
例如，我们假定有一个 `variant` 类型，但它当前尚未在 GSL 中。
总之，请使用[通过表决进入 C++17 的版本](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0088r3.html)。

由于诸如当前 C++ 版本的限制等技术原因，您所使用的程序库中可能不支持下面列出的某些 GSL 类型。
请查阅您的 GSL 文档以获得更多信息。

对于以下的每个 GSL 类型，我们都为该类型给出了不变式。只要用户代码仅使用类型所提供的成员或自由函数（就是说，用户代码不会以违反任何其他指南规则的方式，绕过类型的接口来改动对象的值或位），该不变式均有效。

GSL 组件概览：

* [GSL.view: 视图](#ss-views)
* [GSL.owner](#ss-ownership)
* [GSL.assert: 断言](#ss-assertions)
* [GSL.util: 工具](#ss-utilities)
* [GSL.concept: 概念](#ss-gsl-concepts)

我们计划提供一个“ISO C++ 标准风格的”半正式的 GSL 规范。

我们依赖于 ISO C++ 标准库，并希望 GSL 的一些部分能够被吸收到标准库之中。

## <a id="ss-views"></a>GSL.view: 视图

这些类型使用户可以区分带有和没有所有权的指针，并区分指向单个对象的指针和指向序列的第一个元素的指针。

“视图”都不是所有者。

引用都不是所有者（参见 [R.4](#rr-ref)）。注意：有许多机会能让引用存活超过其所指代的对象，如按引用返回局部变量，持有 vector 的某个元素的引用然后进行 `push_back`，绑定到  `std::max(x, y + 1)`，等等。生存期安全性剖面配置的目标就是处理这些事情，但即便如此 `owner<T&>` 也没有意义且不建议使用。

它们的名字基本上遵循 ISO 标准库风格（小写字母和下划线）：

* `T*`      // `T*` 不是所有者，可能为 null；假定为指向单个元素。
* `T&`      // `T&` 不是所有者，不可能为“null 引用”；引用总是绑定到对象上。

“原生指针”写法（如 `int*`）假定为具有其最常见的含义；亦即指向一个对象的指针，但并不拥有它。
所有者应当被转换为资源包装（如 `unique_ptr` 或 `vector<T>`），或标为 `owner<T*>`。

* `owner<T*>`   // `T*`，拥有所指向/指代的对象；可能为 `nullptr`。

`owner` 用于对代码中有所有权的指针进行标记，它们无法更改为使用适当的资源包装。
其原因可能包括：

* 转换的成本。
* 需要为某个 ABI 使用指针。
* 这个指针时某种资源包装的实现的一部分。

`owner<T>` 和 `T` 的某种资源包装的区别在于它仍然需要明确进行 `delete`。

`owner<T>` 假定为指代自由存储（堆）上的某个对象。

当某个东西不应当为 `nullptr` 时，可以这样做：

* `not_null<T>`   // `T` 通常是某个指针类型（例如 `not_null<int*>` 和 `not_null<owner<Foo*>>`），且不能为 `nullptr`。
  `T` 可以是 `==nullptr` 有意义的任何类型。

* `span<T>`       // `[p:p+n)`，构造函数接受 `{p, q}` 和 `{p, n}`；`T` 为指针类型
* `span_p<T>`     // `{p, predicate}` `[p:q)`，其中 `q` 为首个使 `predicate(*p)` 为真的元素

`span<T>` 指代零或更多可改动的 `T`，除非 `T` 为 `const` 类型。对 `span` 元素的所有访问，尤其是通过 `operator[]` 进行的访问，默认保证进行边界检查。

> 注：有提案将 GSL 的 `span`（起初叫做 `array_view`）加入 C++ 标准库且已被采纳（名字和接口有改动），唯一不同是 `std::span` 不提供边界检查保证。因此，GSL 修改了 `span` 的名字和接口以跟踪 `std::span`，并应当与 `std::span` 完全相同，而其仅有差别应当为，GSL 的 `span` 默认是完全边界安全的。如果边界检查影响其接口，那么应当通过 ISO C++ 委员会带回改动的提案，以保持 `gsl::span` 和 `std::span` 接口兼容。如果 `std::span` 未来的演化添加了边界检查，则 `gsl::span` 即可移除。

“指针算术”最好在 `span` 之内进行。
指向多个 `char` 但并非 C 风格字符串的 `char*`（比如指向某个输入缓冲区的指针）应当表示为一个 `span`。

* `zstring`    // `char*`，假定为 C 风格字符串；亦即以零结尾的 `char` 的序列或者是 `nullptr`
* `czstring`   // `const char*`，假定为 C 风格字符串；亦即以零结尾的 `const` `char` 的序列或者是 `nullptr`

逻辑上来说，最后两种别名是没有必要的，但我们并不总是依照逻辑的，它们可以在指向单个 `char` 的指针和指向 C 风格字符串的指针之间明确地进行区分。
并未假定为零结尾的字符序列应当是 `span<char>`，或当因 ABI 问题而不可能时是 `char*`，而不是 `zstring`。


对于不能为 `nullptr` 的 C 风格字符串，应使用 `not_null<zstring>`。 ??? 我们需要为 `not_null<zstring>` 命名吗？还是说它的难看是有用的？

## <a id="ss-ownership"></a>GSL.owner: 所有权指针

* `unique_ptr<T>`     // 唯一所有权：`std::unique_ptr<T>`
* `shared_ptr<T>`     // 共享所有权：`std::shared_ptr<T>`（引用计数指针）
* `stack_array<T>`    // 栈分配数组。元素的数量在构造时确定并固定下来。其元素可改变，除非 `T` 为 `const` 类型。
* `dyn_array<T>`      // ??? 有必要吗 ??? 堆分配数组。元素的数量在构造时确定并固定下来。
  其元素可改变，除非 `T` 为 `const` 类型。基本上这是一个进行分配并拥有其元素的 `span`。

## <a id="ss-assertions"></a>GSL.assert: 断言

* `Expects`     // 前条件断言。当前放置于函数体内。今后应当移动到声明中。
                // `Expects(p)` 当不满足 `p == true` 时会终止程序
                // `Expects` 处于一组选项的控制之下（强制，错误消息，对终止程序的替代）
* `Ensures`     // 后条件断言。当前放置于函数体内。今后应当移动到声明中。

现在这些断言还是宏（天呐！）而且必须（只）被用在函数定义式之内。
等待标准委员会对于契约和断言语法的确定。
参见使用属性语法的[契约提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)，
比如说，`Expects(p)` 将变为 `[[expects: p]]`。

## <a id="ss-utilities"></a>GSL.util: 工具

* `finally`        // `finally(f)` 创建一个 `final_action{f}`，其析构函数将执行 `f`
* `narrow_cast`    // `narrow_cast<T>(x)` 就是 `static_cast<T>(x)`
* `narrow`         // `narrow<T>(x)` 在满足无符号提升下的 `static_cast<T>(x) == x` 时为 `static_cast<T>(x)`，否则抛出 `narrowing_error`（例如，`narrow<unsigned>(-42)` 会抛出异常）
* `[[implicit]]`   // 放在单参数构造函数上的“记号”，以明确说明它们并非显式构造函数。
* `move_owner`     // `p = move_owner(q)` 含义为 `p = q` 但 ???
* `joining_thread` // RAII 风格版本的进行联结的 `std::thread`
* `index`          // 用于进行所有的容器和数组索引的类型（当前是 `ptrdiff_t` 的别名）

## <a id="ss-gsl-concepts"></a>GSL.concept: 概念

这些概念（类型谓词）借用于
Andrew Sutton 的 Origin 程序库，
Range 提案，
以及 ISO WG21 的 Palo Alto TR。
其中许多都与已在 C++20 中成为 ISO C++ 标准的概念十分相似。

* `String`
* `Number`
* `Boolean`
* `Range`              // C++20 中为 `std::ranges::range`
* `Sortable`           // C++20 中为 `std::sortable`
* `EqualityComparable` // C++20 中为 `std::equality_comparable`
* `Convertible`        // C++20 中为 `std::convertible_to`
* `Common`             // C++20 中为 `std::common_with`
* `Integral`           // C++20 中为 `std::integral`
* `SignedIntegral`     // C++20 中为 `std::signed_integral`
* `SemiRegular`        // C++20 中为 `std::semiregular`
* `Regular`            // C++20 中为 `std::regular`
* `TotallyOrdered`     // C++20 中为 `std::totally_ordered`
* `Function`           // C++20 中为 `std::invocable`
* `RegularFunction`    // C++20 中为 `std::regular_invocable`
* `Predicate`          // C++20 中为 `std::predicate`
* `Relation`           // C++20 中为 `std::relation`
* ...

### <a id="ss-gsl-smartptrconcepts"></a>GSL.ptr: 智能指针概念

* `Pointer`  // 带有 `*`，`->`，`==`，以及默认构造的类型（默认构造被假定为设值为唯一的“null”值）
* `Unique_pointer`  // 符合 `Pointer` 的类型，可移动但不可复制
* `Shared_pointer`   // 符合 `Pointer` 的类型，可复制

# <a id="s-naming"></a>NL: 命名和代码布局建议

维持一致的命名和代码布局是很有用的。
即便不为其他原因，也可以减少“我的代码风格比你的好”这类的纷争。
然而，人们使用许多许多的不同代码风格，并狂热地坚持它们（的优缺点）。
而且，大多数的现实项目都包含来自于许多来源的代码，因而通常不可能把所有的代码都标准化为某个单一的代码风格。
经过许多的用户请求给予指导后，我们给出一组规则，当你没有更好的选择时可以使用它们，但真正的目标在于一致性，而不是任何一组特定的规则。
IDE 和工具可以提供辅助（当然也可能造成妨碍）。

命名和代码布局规则：

* [NL.1: 不要在代码注释中说明可以由代码来清晰表达的东西](#rl-comments)
* [NL.2: 在代码注释中说明意图](#rl-comments-intent)
* [NL.3: 保持代码注释简明干脆](#rl-comments-crisp)
* [NL.4: 保持一种统一的缩进风格](#rl-indent)
* [NL.5: 避免在名字中编码类型信息](#rl-name-type)
* [NL.7: 使名字的长度大约正比于其作用域的长度](#rl-name-length)
* [NL.8: 使用一种统一的命名风格](#rl-name)
* [NL.9: 将 `ALL_CAPS`（全大写）仅用于宏的名字](#rl-all-caps)
* [NL.10: 优先采用 `underscore_style`（下划线风格）的名字](#rl-camel)
* [NL.11: 使字面量可阅读](#rl-literals)
* [NL.15: 节制地使用空格](#rl-space)
* [NL.16: 使用一种常规的类成员声明次序](#rl-order)
* [NL.17: 使用从 K&R 衍生出的代码布局](#rl-knr)
* [NL.18: 使用 C++ 风格的声明符布局](#rl-ptr)
* [NL.19: 避免使用容易误读的名字](#rl-misread)
* [NL.20: 不要把两个语句放在同一行中](#rl-stmt)
* [NL.21: 每个声明式（仅）声明一个名字](#rl-dcl)
* [NL.25: 请勿将 `void` 用作参数类型](#rl-void)
* [NL.26: 采用符合惯例的 `const` 写法](#rl-const)
* [NL.27: 为代码文件使用后缀 `.cpp`，而对接口文件使用后缀 `.h`](#rl-file-suffix)

这些问题的大部分都是审美问题，程序员都有很强的个人倾向。
IDE 也都会提供某些默认方案和一组替代方案。
这些规则是作为缺省建议的，如果没有别的理由，请采用它们。

我们收到一些意见称命名和代码布局非常个人化和任意性，我们不应该试图为之“立法”。
我们并不是在“立法”（参见前一个段落）。
不过，我们也收到了大量的针对某些命名和代码布局约定的请求，要求当没有外来限制的时候应当采用它们。

更专门和详细的规则更加易于强制实施。

这些规则恐怕会和 [PPP Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf) 中的建议有很强的相似性，
它是为支持 Stroustrup 的 [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html) 而编制的。

### <a id="rl-comments"></a>NL.1: 不要在代码注释中说明可以由代码来清晰表达的东西

##### 理由

编译器不会读注释。
注释没有代码那么精确。
注释不会像代码那样进行一致地更新。

##### 示例，不好

    auto x = m * v1 + vv;   // 将 m 乘以 v1 并将其结果加上 vv

##### 强制实施

构建一个 AI 程序来解释口语英文文字，看看它所说的是否可以用 C++ 来更好地表达。

### <a id="rl-comments-intent"></a>NL.2: 在代码注释中说明意图

##### 理由

代码表示的是做了什么，而不是想要做成什么。通常来说意图比实现能够更清晰简明地进行说明。

##### 示例

    void stable_sort(Sortable& c)
        // 对 c 根据由 < 决定的顺序进行排序，保持相等元素（由 == 定义）的
        // 原始相对顺序
    {
        // ... 相当多的不平常的代码行 ...
    }

##### 注解

如果代码注释和代码有冲突，则它们都可能是错的。

### <a id="rl-comments-crisp"></a>NL.3: 保持代码注释简明干脆

##### 理由

冗长啰嗦会拖慢理解速度，而且到处散布在代码文件里也会让代码难于阅读。

##### 注解

使用明白易懂的英文。
也许我可以流利使用丹麦语，但大多数程序员不行；我的代码的维护者也不行。
避免使用网络用语，注意你的文法，标点，以及大小写。
目标是专业性，而不是“够酷”。

##### 强制实施

不可能。

### <a id="rl-indent"></a>NL.4: 保持一种统一的缩进风格

##### 理由

可读性。避免“微妙的错误”。

##### 示例，不好

    int i;
    for (i = 0; i < max; ++i); // 可能出现的 BUG
    if (i == j)
        return i;

##### 注解

总是把 `if (...)`，`for (...)`，以及 `while (...)` 之后的语句进行缩进是一个好主意：

    if (i < 0) error("negative argument");

    if (i < 0)
        error("negative argument");

##### 强制实施

使用一种工具。

### <a id="rl-name-type"></a>NL.5: 避免在名字中编码类型信息

##### 原理

当名字反映类型而不是功能时，它将变得难于为提供其功能而改变其所使用的类型。
而且，当改变变量的类型时，使用它的代码也得修改。
最小化无意进行的转换。

##### 示例，不好

    void print_int(int i);
    void print_string(const char*);

    print_int(1);          // 重复，人工进行类型匹配
    print_string("xyzzy"); // 重复，人工进行类型匹配

##### 示例，好

    void print(int i);
    void print(string_view);    // 对任意字符串式的序列都能工作

    print(1);              // 简洁，自动类型匹配
    print("xyzzy");        // 简洁，自动类型匹配

##### 注解

带有类型编码的名字要么啰嗦要么难懂。

    printS  // 打印一个 std::string
    prints  // 打印一个 C 风格字符串
    printi  // 打印一个 int

在无类型语言中曾经采用过像匈牙利记法这样的技巧来在名字中编码类型，但在像 C++ 这样的强静态类型语言中，这通常是不必要而且实际上是有害的，因为这些标注会过时（这些累赘和注释类似，而且和它们一样会烂掉），而且它们干扰了语言的恰当用法（应当代之以使用相同的名字和重载决议）。

##### 注解

一些代码风格会使用非常一般性的（而不是特定于类型的）前缀来代表变量的一般用法。

    auto p = new User();
    auto p = make_unique<User>();
    // 注："p" 并非是说“User 类型的原始指针”，
    //     而只是一般性的“这是一次间接访问”

    auto cntHits = calc_total_of_hits(/*...*/);
    // 注："cnt" 并非用于编码某个类型，
    //     而只是一般性的“这是某种东西的一个计数”

这样做是没有害处的，且并不属于本条指导方针，因为其并未编码类型信息。

##### 注解

一些代码风格会对成员和局部变量，以及全局变量之间进行区分。

    struct S {
        int m_;
        S(int m) : m_{abs(m)} { }
    };

这样做是没有害处的，且并不属于本条指导方针，因为其并未编码类型信息。

##### 注解

像 C++ 这样，一些代码风格对类型和非类型之间进行区分。
例如，对类型名字首字母大写，而函数和变量名字则不这样做。

    typename<typename T>
    class HashTable {   // 将 string 映射为 T
        // ...
    };

    HashTable<int> index;

这样做是没有害处的，且并不属于本条指导方针，因为其并未编码类型信息。

### <a id="rl-name-length"></a>NL.7: 使名字的长度大约正比于其作用域的长度

**原理**: 作用域越大，搞混的机会和意外的名字冲突的机会就越大。

##### 示例

    double sqrt(double x);   // 返回 x 的平方根；x 必须是非负数

    int length(const char* p);  // 返回零结尾的 C 风格字符串的字符数量

    int length_of_string(const char zero_terminated_array_of_char[])    // 不好: 啰嗦

    int g;      // 不好: 全局变量具有密秘的名字

    int open;   // 不好: 全局变量使用短小且常用的名字

为指针使用 `p`，以及为浮点变量使用 `x` 是符合惯例的，在受限的作用域中不会造成混乱。

##### 强制实施

???

### <a id="rl-name"></a>NL.8: 使用一种统一的命名风格

**原理**: 命名和命名风格的一致性会提高可读性。

##### 注解

命名风格有好多，当你使用多个程序库时，你无法遵循所有它们不同的命名约定。
应当选用一种“自有风格”，但保持“导入”的程序为其原有风格不变。

##### 示例

ISO 标准仅使用小写字母和数字，并用下划线进行词的连接。

* `int`
* `vector`
* `my_map`

避免使用双下划线 `__`。

##### 示例

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf)：
采用 ISO 标准，但在自己的类型和概念上采用大写字母：

* `int`
* `vector`
* `My_map`

##### 示例

CamelCase：多词标识符的每个词首字母大写：

* `int`
* `vector`
* `MyMap`
* `myMap`

一些命名约定会将首字母大写，而另一些不会。

##### 注解

应当试图在对缩略词的使用和标识符长度上保持一致风格。

    int mtbf {12};
    int mean_time_between_failures {12}; // 你自己决定

##### 强制实施

除使用具有不同命名约定的程序库之外应当是可能做到的。

### <a id="rl-all-caps"></a>NL.9: 将 `ALL_CAPS`（全大写）仅用于宏的名字

##### 理由

避免在宏和遵循作用域和类型规则的名字之间造成混乱。

##### 示例

    void f()
    {
        const int SIZE{1000};  // 不好，应代之以 'size'
        int v[SIZE];
    }

##### 注解

这条规则适用于非宏的符号常量：

    enum bad { BAD, WORSE, HORRIBLE }; // 不好

##### 强制实施

* 对带有小写字母的宏进行标记
* 对 `ALL_CAPS` 非宏名字进行标记

### <a id="rl-camel"></a>NL.10: 优先采用 `underscore_style`（下划线风格）的名字

##### 理由

用下划线来分隔名字的各部分就是 C 和 C++ 的原始风格，并被用于 C++ 标准库中。

##### 注解

这条规则仅作为当你有选择权时的缺省方案。
通常你是没有什么选择权的，而只能遵循某个已经设立的风格以维持[一致性](#rl-name)。
对一致性的需要优先于个人喜好。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](#s-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 示例

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf)：
采用 ISO 标准，但在自己的类型和概念上采用大写字母：

* `int`
* `vector`
* `My_map`

##### 强制实施

不可能。

### <a id="rl-literals"></a>NL.11: 使字面量可阅读

##### 理由

可读性。

##### 示例

用数字分隔符来避免长串的数字

    auto c = 299'792'458; // m/s2
    auto q2 = 0b0000'1111'0000'0000;
    auto ss_number = 123'456'7890;

##### 示例

需要清晰性时使用字面量后缀

    auto hello = "Hello!"s; // std::string
    auto world = "world";   // C 风格字符串
    auto interval = 100ms;  // 使用 <chrono>

##### 注解

不能在代码中到处当做[“魔法常量”](#res-magic)一样乱用字面量，
但当定义它们时使它们更可读仍是个好主意。
在较长的整数串中很容易出现拼写错误。

##### 强制实施

标记长数字串。麻烦的是“长”的定义；也许应当是 7。

### <a id="rl-space"></a>NL.15: 节制地使用空格

##### 理由

太多的空格会让文本更大更分散。

##### 示例，不好

    #include < map >

    int main(int argc, char * argv [ ])
    {
        // ...
    }

##### 示例

    #include <map>

    int main(int argc, char* argv[])
    {
        // ...
    }

##### 注解

一些 IDE 有其自己的看法，并会添加分散的空格。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](#s-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 注解

我们将恰当放置的空白评价为能够明显有助于可读性。但请勿过度。

### <a id="rl-order"></a>NL.16: 使用一种常规的类成员声明次序

##### 理由

一种常规的成员次序会提高可读性。

以如下次序声明类

* 类型：类，枚举，别名（`using`）
* 构造函数，赋值，析构函数
* 函数
* 数据

采用先是 `public`，然后是 `protected`，之后是 `private` 的次序。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](#s-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 示例

    class X {
    public:
        // 接口
    protected:
        // 供派生类实现使用的不带检查的函数
    private:
        // 实现细节
    };

##### 示例

有时候，成员的默认顺序，与将公开接口从实现细节中分离出来的需求之间有冲突。
这种情况下，私有类型和函数可以和私有数据放在一起。

    class X {
    public:
        // 接口
    protected:
        // 供派生类实现使用的不带检查的函数
    private:
        // 实现细节（类型，函数和数据）
    };

##### 示例，不好

避免让具有某一种访问（如 `public`）的多个声明块被具有不同访问（如 `private`）的其他声明块分隔开。

    class X {
    public:
        void f();
    public:
        int g();
        // ...
    };

用宏来声明成员组的做法通常会导致违反所有的次序规则。
不过，宏的使用掩盖了其所表达的东西。

##### 强制实施

对背离上述建议次序的代码进行标记。将会有大量的老代码不符合这条规则。

### <a id="rl-knr"></a>NL.17: 使用从 K&R 衍生出的代码布局

##### 理由

这正是 C 和 C++ 的原始代码布局。它很好地保持了纵向空间。它对不同语言构造（如函数和类）进行了很好的区分。

##### 注解

在 C++ 的语境中，这种风格通常被称为“Stroustrup”。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](#s-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 示例

    struct Cable {
        int x;
        // ...
    };

    double foo(int x)
    {
        if (0 < x) {
            // ...
        }

        switch (x) {
        case 0:
            // ...
            break;
        case amazing:
            // ...
            break;
        default:
            // ...
            break;
        }

        if (0 < x)
            ++x;

        if (x < 0)
            something();
        else
            something_else();

        return some_value;
    }

注意 `if` 和 `(` 之间有一个空格

##### 注解

每个语句，`if` 的分支，以及 `for` 的代码体都使用单独的代码行。

##### 注解

`class` 和 `struct` 的 `{` *并不*在单独的代码行上，但函数的 `{` 在单独的代码行上。

##### 注解

对你自定义的类型的名字进行首字母大写，以将其与标准库类型相区分。

##### 注解

不要对函数名大写。

##### 强制实施

如果想要强制实施的话，请使用某个 IDE 进行格式化。

### <a id="rl-ptr"></a>NL.18: 使用 C++ 风格的声明符布局

##### 理由

C 风格的布局强调其在表达式中的用法和文法，而 C++ 风格强调的是类型。
对表达式用法的说辞并不适用于引用。

##### 示例

    T& operator[](size_t);   // OK
    T &operator[](size_t);   // 奇怪
    T & operator[](size_t);   // 不确定

##### 注解

这个推荐适用于[当你没有约束条件或者没有更好的想法时](#s-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 强制实施

由于历史原因而不可能。


### <a id="rl-misread"></a>NL.19: 避免使用容易误读的名字

##### 理由

可读性。
并非每个人都有能将字符轻易区分开的屏幕和打印机。
我们很容易搞混拼写相似和略微拼错的单词。

##### 示例

    int oO01lL = 6; // 不好

    int splunk = 7;
    int splonk = 8; // 不好：splunk 和 splonk 很容易搞混

##### 强制实施

???

### <a id="rl-stmt"></a>NL.20: 不要把两个语句放在同一行中

##### 理由

可读性。
当一行里有多个语句时，相当容易忽视某个语句。

##### 示例

    int x = 7; char* p = 29;    // 请勿如此
    int x = 7; f(x);  ++x;      // 请勿如此

##### 强制实施

容易。

### <a id="rl-dcl"></a>NL.21: 每个声明式（仅）声明一个名字

##### 理由

可读性。
最小化声明符语法造成的混乱。

##### 注解

相关细节，参见 [ES.10](#res-name-one)、


### <a id="rl-void"></a>NL.25: 请勿将 `void` 用作参数类型

##### 理由

这很啰嗦，而且仅在考虑 C 兼容性是才有必要。

##### 示例

    void f(void);   // 不好

    void g();       // 好多了

##### 注解

即便是 Dennis Ritchie 自己都认为 `void f(void)` 很讨厌。
你可以反驳称，在 C 中当函数原型很少见时禁止这样的代码：

    int f();
    f(1, 2, "weird but valid C89");   // 希望 f() 被定义为 int f(a, b, c) char* c; { /* ... */ }

可能造成很大的问题，但这并不适于 21 世纪和 C++。

### <a id="rl-const"></a>NL.26: 采用符合惯例的 `const` 写法

##### 理由

更多程序员更加熟悉惯例写法。
大型代码库中的一致性。

##### 示例

    const int x = 7;    // OK
    int const y = 9;    // 不好

    const int *const p = nullptr;   // OK, 指向常量 int 的常量指针
    int const *const p = nullptr;   // 不好，指向常量 int 的常量指针

##### 注解

我们知道你可能会说“不好”的例子比标有“OK”的更符合逻辑，
但它们会让更多人搞混，尤其是那些依赖于采用了远为常用，符合惯例的 OK 风格的教学材料的新手们。

一如往常，请记住这些命名和代码布局规则的目标在于一致性，而审美则会有广泛的变化。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](#s-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 强制实施

标记用作类型的后缀的 `const`。

### <a id="rl-file-suffix"></a>NL.27: 为代码文件使用后缀 `.cpp`，而对接口文件使用后缀 `.h`

##### 理由

这是一条历史悠久的约定。
不过一致性更加重要，因此如果你的项目用了别的约定的话，应当遵守它。

##### 注解

这项约定反应了一种常见使用模式：
头文件更容易和 C 语言共享使用，并可以作为 C++ 和 C 编译，它们通常用 `.h` 后缀，
并且对于有意要和 C 共用的头文件来说，让所有头文件都使用 `.h` 而不是别的扩展名要更加容易。
另一方面，实现文件则很少会和 C 共用，通常应当和 `.c` 文件相区别，
因此一般最好为所有的 C++ 实现文件用别的扩展名（如 `.cpp`）来命名。

特定的名字 `.h` 和 `.cpp` 并不是必要的（只是作为缺省建议），其他的名字也被广泛采用。
例子包括 `.hh`，`.C`，和 `.cxx` 等。请以类似方式使用这些名字。
本文档中我们把 `.h` 和 `.cpp` 作为头文件和实现文件的简便提法，
虽然实际上的扩展名可能是不同的。

也许你的 IDE（如果你使用的话）对后缀有较强的倾向。

##### 示例

    // foo.h:
    extern int a;   // 声明
    extern void foo();

    // foo.cpp:
    int a;   // 定义
    void foo() { ++a; }

`foo.h` 提供了 `foo.cpp` 的接口。最好避免全局变量。

##### 示例，不好

    // foo.h:
    int a;   // 定义
    void foo() { ++a; }

一个程序中两次 `#include <foo.h>` 将导致因为对唯一定义规则的两次违反而出现一个连接错误。

##### 强制实施

* 对不符合约定的文件名进行标记。
* 检查 `.h` 和 `.cpp`（或等价文件）遵循下列各规则。

# <a id="s-faq"></a>FAQ: 常见问题及其回答

本节中包括了对关于这些指导方针的常见问题的回答。

### <a id="faq-aims"></a>FAQ.1: 这些指导方针的想要达成什么目标？

请参见<a href="#S-abstract">本页面开头</a>。这是一个开源项目，旨在为采用当今的 C++ 标准来编写 C++ 代码而维护的一组现代的权威指导方针。这些指导方针的设计是现代的，尽可能使机器可实施的，并且是为贡献和分支保持开放，以使各种组织机构可以便于将它们整合到其自己组织的编码指导方针之中。

### <a id="faq-announced"></a>FAQ.2: 这项工作是何时何地首次公开的？

是在 [Bjarne Stroustrup 在他为 CppCon 2015 的开场主旨演讲，“Writing Good C++14”](https://isocpp.org/blog/2015/09/stroustrup-cppcon15-keynote)。另请参见[相应的 isocpp.org 博客条目](https://isocpp.org/blog/2015/09/bjarne-stroustrup-announces-cpp-core-guidelines)，关于类型和内存安全性指导方针的原理请参见 [Herb Sutter 的后续 CppCon 2015 演讲，“Writing Good C++14 ... By Default”](https://isocpp.org/blog/2015/09/sutter-cppcon15-day2plenary)。

### <a id="faq-maintainers"></a>FAQ.3: 谁是这些指导方针的作者和维护者？

最初的主要作者和维护者是 Bjarne Stroustrup 和 Herb Sutter，而迄今为止的指导方针则是由来自 CERN，Microsoft，Morgan Stanley，以及许多其他组织机构的专家所贡献的。指导方针发布时，其正处于 "0.6" 状态，我们欢迎人们进行贡献。正如 Stroustrup 在其声明中所说：“我们需要帮助！”

### <a id="faq-contribute"></a>FAQ.4: 我如何进行贡献呢？

参见 [CONTRIBUTING.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CONTRIBUTING.md)。我们感激志愿者的帮助！

### <a id="faq-maintainer"></a>FAQ.5: 怎样成为一名编辑或维护者？

通过先进行大量贡献并使你的贡献被认可具有一致的质量。参见 [CONTRIBUTING.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CONTRIBUTING.md)。我们感激志愿者的帮助！

### <a id="faq-iso"></a>FAQ.6: 这些指导方针被 ISO C++ 标准委员会采纳了吗？它们是否代表委员会的一致意见？

不是这样。这些指导方针不在标准之内。它们是为标准服务的，而当前维护的指导方针是为了更有效地使用当前的标准 C++的。我们的目标是使其与委员会所设计的标准保持同步。

### <a id="faq-isocpp"></a>FAQ.7: 既然这些指导方针并不是委员会所采纳的，它们为何在 `github.com/isocpp` 之下呢？

因为 `isocpp` 是标准 C++ 基金会；而标准委员会的仓库则处于 [github.com/*cplusplus*](https://github.com/cplusplus) 之下。我们需要一个中立组织来持有版权和许可以明确其并不是由某个人或供应商所控制的。这个自然实体就是基金会，其设立是为了推进使用并持续更新对现代标准 C++ 的理解，以及推进标准委员会的工作。其所遵循的正是与 isocpp.org 为 [C++ FAQ](https://isocpp.org/faq) 所做的相同模式，它是有 Bjarne Stroustrup，Marshall Cline，和 Herb Sutter 所发起的工作，并以相同的方式贡献为了开放项目。

### <a id="faq-cpp98"></a>FAQ.8: 会有 C++98 版本的指导方针吗？C++11 版本呢？

不会。这些指导方针的目标是更好地使用现代标准 C++，以及假定你有一个现代的遵循标准的编译器时如何进行代码编写的。

### <a id="faq-language-extensions"></a>FAQ.9: 这些指导方针中会提出新的语言功能吗？

不会。这些指导方针的目标是更好地使用现代标准 C++，它们自我限定为仅建议使用这些功能。

### <a id="faq-markdown"></a>FAQ.10: 这些指导方针的书写使用的是哪个版本的 Markdown？

这些编码指导方针使用的是 [CommonMark](http://commonmark.org)，以及 `<a>` HTML 锚定元素。

我们正在考虑以下这些来自 [GitHub Flavored Markdown (GFM)](https://help.github.com/articles/github-flavored-markdown/) 的扩展：

* 有围栏代码块（正在讨论是否统一使用缩进还是围栏代码块）
* 表格（我们虽然还没用到，但很需要它们，这是一种 GFM 扩展）

避免使用其他 HTML 标签和其他扩展。

注意：我们还没对这种风格达成一致。

### <a id="faq-gsl"></a>FAQ.50: 什么是 GSL（指导方针支持程序库）？

GSL 是在指导方针中所指定的类型和别名的一个小集合。当写下本文时，对它们的说明还过于松散；我们计划添加一个 WG21 风格的接口规范来确保不同实现之间保持一致，并作为一项可能的标准化提案，按常规遵循标准委员会进行采纳、改进、修订或否决。

### <a id="faq-msgsl"></a>FAQ.51: [github.com/Microsoft/GSL](https://github.com/Microsoft/GSL) 是 GSL 吗？

不是。它只是由 Microsoft 所贡献的第一个实现。我们鼓励其他供应商提供其他的实现，对该实现的分支和贡献也是被鼓励的。书写本文作为一项公开项目的一周中，已经出现了至少一个 GPLv3 的开源实现。我们计划制定一个 WG21 风格的接口规范来确保不同实现之间保持一致。

### <a id="faq-gsl-implementation"></a>FAQ.52: 为何不在指导方针之中提供一个真正的 GSL 实现呢？

我们不愿去保佑某个特定的实现，因为我们不希望让人们以为只有一个实现，而疏忽大意地扼杀了其他并行的实现。而如果在指导方针中包含一个真正实现的话，无论是谁提供了它都会变得过于有影响力。我们更倾向于采用委员会的更具长期性的方案，即指定其接口而不是实现。但同时我们也需要至少存在一个实现；希望可以有很多。

### <a id="faq-boost"></a>FAQ.53: 为什么不把 GSL 类型提交给 Boost 呢？

因为我们想要立刻使用它们，也因为我们想要在一旦标准库中出现了满足其需要的类型时立刻将它们撤销掉。

### <a id="faq-gsl-iso"></a>FAQ.54: ISO C++ 标准委员会采纳了 GSL（指导方针支持程序库）吗？

没有。GSL 的存在只为提供少量标准库中还没有的类型和别名。如果委员会决定了（这些类型或者满足其需要的其他类型的）标准化的版本，就可以将它们从 GSL 中删除了。

### <a id="faq-gsl-string-view"></a>FAQ.55: 既然你是尽可能使用标准类型，为什么 GSL 的 `span<char>` 同 Library Fundamentals 1 Technical Specification 和 C++17 工作文本中的 `string_view` 不同呢？为什么不使用委员会采纳的 `string_view`？

有关 C++ 标准库的视图的分类的统一观点是，“视图（view）”意味着“只读”，而“跨距（span）”意味着“可读写”。如果你只需要一组字符的不需要保证边界检查的只读视图，并且你可以用 C++17，那就使用 C++17 的 `std::string_view`。否则，如果你需要的是不需要保证边界检查的可读写视图，并且可以用 C++20，那就用 C++20 的 `std::span<char>`。否则，就用 `gsl::span<char>`。

### <a id="faq-gsl-owner"></a>FAQ.56: `owner` 和提案的 `observer_ptr` 一样吗？

不一样。`owner` 有所有权，它是一个别名，而且适用于任何间接类型。而 `observer_ptr` 的主要意图则是明确某个*没有*所有权的指针。

### <a id="faq-gsl-stack-array"></a>FAQ.57: `stack_array` 和标准的 `array` 一样吗？

不一样。`stack_array` 保证在栈上分配。虽然 `std::array` 直接在其自身内部包含存储，但 `array` 对象可以放在包括堆在内的任何地方。

### <a id="faq-gsl-dyn-array"></a>FAQ.58: `dyn_array` 和 `vector` 或者提案的 `dynarray` 一样吗？

不一样。`dyn_array` 是不可改变大小的，是一种指代堆分配的固定大小数组的一种安全方式。与 `vector` 不同，它是为了取代数组 `new[]` 的。与委员会中提案的 `dynarray` 不同，它并不会参与编译器和语言的魔法，来在当它作为分配于栈上的对象的成员时也在栈上分配；它只不过指代一个“动态的”或基于堆的数组而已。

### <a id="faq-gsl-expects"></a>FAQ.59: `Expects` 和 `assert` 一样吗？

不一样。它是一种对于契约前条件语言支持的占位符。

### <a id="faq-gsl-ensures"></a>FAQ.60: `Ensures` 和 `assert` 一样吗？

不一样。它是一种对于契约后条件语言支持的占位符。

# <a id="s-libraries"></a>附录 A: 程序库

这个部分列出了一些推荐的程序库，并且特别推荐了其中的几个。

??? 这个对一般性指南来说合适吗？我觉得不是 ???

# <a id="s-modernizing"></a>附录 B: 代码的现代化转换

理想情况下，我们的所有代码都应当遵循全部的规则。
而实际情况则是，我们不得不对付大量的老代码：

* 那些在我们的指导方针被建立或者被了解到之前所编写的代码
* 那些依据老的或者不同的标准所编写的程序库
* 那些在“不寻常”的约束下编写的代码
* 那些我们还未来得及使其现代化的代码

如果有上百万行的新代码的话，“立刻改掉它们”的想法一般都是不现实的。
因此，我们需要一种方式能够渐进地对代码库进行现代化转换。

将老代码升级为现代风格可能是很让人却步的工作。
老代码通常即混乱（难于理解）又可以（在当前使用范围内）正确工作。
很有可能，原来的程序员并不在场，而且测试用例也不完整。
代码的混乱性显著地增大了为进行任何改动所需要的工作量和引入错误的风险。
通常，混乱的老代码的运行会有不必要的缓慢，因为它需要过期的编译器，并且无法得益于当代硬件的改进。
许多情况下，都需要某种自动进行“现代化转换”的工具支持来进行主要的升级工作。

对代码现代化转换的目的在于简化新功能的添加，简化维护工作，以及增加性能（吞吐量或延迟），和更好地利用当代的硬件能力。
而让代码“看起来更好”或“遵循现代风格”自身并不能成为改动的理由。
每一种改动都蕴含着风险，而是由过时的代码库则会蕴含一些成本（包含丢失机会带来的成本）。
成本的缩减必须超过风险。

如何做呢？

并不存在唯一的代码现代化转换的方案。
如何最好地执行，依赖于代码，更新的进度压力，开发者的背景，以及可用的工具。
下面是一些（非常一般的）想法：

* 理想情况是“对全部代码一起进行升级”。这将在最短的总时间内获得做大的好处。
  在大多数情况下，这也是不可能的。
* 我们可以对代码库以模块为单位进行转换，不过任何影响接口（尤其是 ABI）的规则，如 [使用 `span`](#ss-views)，都无法按模块来达成。
* 我们可以“自底向上”转换代码，并最先应用我们估计在给定的代码库上将会带来最大好处和最少麻烦的那些规则。
* 我们可以从关注接口开始，比如说，保证没有资源的泄漏，没有指针误用等。
  这可能会导致涉及整个代码库的一些改动，不过它们是最可能会带来巨大好处的改动。
  以后，隐藏在这些接口后面的代码可以渐进地进行现代化转换而不会影响其他的代码。

无论你选择哪种方式，都要注意，对指导方针的最高度的遵循性才会带来大多数的好处。
这些指导方针并不是一组无关规则的随机集合，并不能让你随意选取并期望取得成功。

我们衷心希望听到有关它们的使用经验，以及有关工具是如何使用的。
如果有分析工具（即便是代码变换工具）的支持的话，代码现代化转换后可以变得更快，更简单，而且更安全。

# <a id="s-discussion"></a>附录 C: 讨论

这个部分包含了对规则和规则集合的跟进材料。
尤其是，我们列出了更多的原理说明，更长的例子，以及对替代方案的探讨等。

### <a id="sd-order"></a>讨论: 以成员的声明顺序进行成员变量的定义和初始化

成员变量总是以它们在类定义中的声明顺序进行初始化，因此在构造函数初始化列表中应当以该顺序来书写它们。以别的顺序书写它们只会让代码混淆，因为它并不会以你所见到的顺序来运行，而这会导致难于发现与顺序有关的 BUG。

    class Employee {
        string email, first, last;
    public:
        Employee(const char* firstName, const char* lastName);
        // ...
    };

    Employee::Employee(const char* firstName, const char* lastName)
      : first(firstName),
        last(lastName),
        // 不好: first 和 last 还未构造
        email(first + "." + last + "@acme.com")
    {}

在这个例子中，`email` 比 `first` 和 `last` 构造得早，因为它是先声明的。这意味着其构造函数对 `first` 和 `last` 的使用过早了——不只在它们被设为所需的值之前，而完全是在它们被构造之前就使用了。

如果类定义和构造函数体是在不同文件中的话，这种由成员变量声明顺序对构造函数的正确性造成的远距离影响将更难于发现。

**参考**：

[\[Cline99\]](#cline99) §22.03-11, [\[Dewhurst03\]](#dewhurst03) §52-53, [\[Koenig97\]](#koenig97) §4, [\[Lakos96\]](#lakos96) §10.3.5, [\[Meyers97\]](#meyers97) §13, [\[Murray93\]](#murray93) §2.1.3, [\[Sutter00\]](#sutter00) §47

### <a id="sd-init"></a>讨论：使用 `=`，`{}`，和 `()` 作为初始化式

???

### <a id="sd-factory"></a>讨论: 当需要在初始化过程中使用“虚函数行为”时，使用工厂函数

如果你的设计需要从基类的构造函数或析构函数中对 `f` 或者 `g` 这样的函数向派生类进行虚函数派发的话，你其实需要的是其他技巧，比如后构造函数——一种必须由调用者调用以完成初始化过程的成员函数，它可以安全地调用 `f` 和 `g`，这是由于成员函数中的虚函数调用能够正常工作。“参考”部分中列出了一些这样的技巧。以下是一个不完整的可选项列表：

* *推卸责任：* 仅仅给出文档说明，要求用户代码在对象构造之后必须立刻调用后初始化函数。
* *惰性后初始化：* 在第一个调用的成员函数中进行。用基类中的一个布尔标记说明后初始化是否已经执行过。
* *使用虚基类语义：* 语言规则要求由最终派生类的构造函数来决定调用哪个基类构造函数；你可以利用这点。（参见[\[Taligent94\]](#taligent94)。）
* *使用工厂函数：* 以这种方式，你可以轻易确保进行对后构造函数的调用。

以下是对最后一种选项的一个例子：

    class B {
    public:
        B()
        {
            /* ... */
            f(); // 不好: C.82：不要在构造函数和析构函数中调用虚函数
            /* ... */
        }

        virtual void f() = 0;
    };

    class B {
    protected:
        class Token {};

    public:
        // 需要公开构造函数以使 make_shared 可以访问它。
        // 通过要求一个 Token 达成受保护访问等级。
        explicit B(Token) { /* ... */ }  // 创建不完全初始化的对象
        virtual void f() = 0;

        template<class T>
        static shared_ptr<T> create()    // 创建共享对象的接口
        {
            auto p = make_shared<T>(typename T::Token{});
            p->post_initialize();
            return p;
        }

    protected:
        virtual void post_initialize()   // 构造之后立即调用
            { /* ... */ f(); /* ... */ } // 好: 虚函数分派是安全的
        }
    };


    class D : public B {                 // 某个派生类
    protected:
        class Token {};

    public:
        // 需要公开构造函数以使 make_shared 可以访问它。
        // 通过要求一个 Token 达成受保护访问等级。
        explicit D(Token) : B( B::Token{} ) {}
        void f() override { /* ... */ };

    protected:
        template<class T>
        friend shared_ptr<T> B::create();
    };

    shared_ptr<D> p = D::Create<D>();    // 创建一个 D 对象

这种设计需要遵守以下纪律：

* 像 `D` 这样的派生类不能暴露可公开调用的构造函数。否则的话，`D` 的使用者就能够创建 `D` 对象而不调用 `post_initialize` 了。
* 分配被限定为使用 `operator new`。不过，`B` 可以覆盖 `new`（参见 [SuttAlex05](#suttalex05) 条款 45 和 46）。
* `D` 必须定义一个带有与 `B` 所选择的相同的参数的构造函数。不过，定义多个重载的 `create` 可以缓和这个问题；而且还可以是这些重载对参数类型进行模板化。

一旦满足了上述要求，这个设计就可以保证对于任意完全构造的 `B` 的派生类对象，都将调用 `post_initialize`。`post_initialize` 不必是虚函数；它可以随意进行虚函数调用。

总之，不存在完美的后构造技巧。最差的方式是完全回避问题而只是让调用方来人工调用后构造函数。即便是最佳方案也需要采用一种不同的对象构造语法（易于进行编译期检查）以及需要派生类的作者的协作（这无法进行编译期进行检查）。

**参考**: [\[Alexandrescu01\]](#alexandrescu01) §3, [\[Boost\]](#boost), [\[Dewhurst03\]](#dewhurst03) §75, [\[Meyers97\]](#meyers97) §46, [\[Stroustrup00\]](#stroustrup00) §15.4.3, [\[Taligent94\]](#taligent94)

### <a id="sd-dtor"></a>讨论: 基类的析构函数应当要么是 public 和 virtual，要么是 protected 且非 virtual

析构应不应该表现为虚函数？就是说，是否允许通过指向 `base` 类的指针来进行析构呢？如果是的话，`base` 的析构函数为被调用则必须是 public 的，而且必须 virtual，否则调用就会导致未定义行为。否则的话，它应当是 protected 的，这样就只有派生类可以在它们自己的析构函数中调用它，且应当是非 virtual 的，因为它并不需要表现为虚函数的行为。

##### 示例

基类的一般情况是为了具有 public 的派生类，因而调用方代码基本上可以确定要用到某种比如 `shared_ptr<base>` 这样的东西：

    class Base {
    public:
        ~Base();                   // 不好, 非 virtual
        virtual ~Base();           // 好
        // ...
    };

    class Derived : public Base { /* ... */ };

    {
        unique_ptr<Base> pb = make_unique<Derived>();
        // ...
    } // 只有当 ~Base 是虚函数时 ~pb 才会调用正确的析构函数

少数比如策略类这类的情况下，类被用作基类是为方便起见，而并非是其多态行为。建议将它们的析构函数作为 protected 和非 virtual 函数：

    class My_policy {
    public:
        virtual ~My_policy();      // 不好, public 并且 virtual
    protected:
        ~My_policy();              // 好
        // ...
    };

    template<class Policy>
    class customizable : Policy { /* ... */ }; // 注: private 继承

##### 注解

这个简单的指导方针演示了一种微妙的问题，而且反映了继承的现代用法以及面向对象的设计原则。

对于某个基类 `Base`，调用方代码可能通过指向 `Base` 的指针来销毁派生类对象，比如使用一个 `unique_ptr<Base>`。如果 `Base` 的析构函数是 public 的且非 virtual（默认情况），它就可能意外地在实际指向一个派生类对象的指针上进行调用，这种情况下想要进行的删除的行为是未定义的。这种状况曾导致一些老编码标准提出通用的要求，让所有基类析构函数都必须 virtual。这种做法杀伤力过大了（虽然这是常见的情况）；其实，规则应当是当且仅当基类析构函数是 public 时才要求它是 virtual 的。

编写一个基类就是在定义一种抽象（参见条款 35 到 37）。注意对于参与这个抽象的每个成员函数来说，你都需要作出以下决定：

* 它是否应当表现为虚函数。
* 它是应当对所有使用 `Base` 指针的调用方公开，还是作为隐藏的内部实现细节。

如条款 39 中所述，对于普通成员函数来说，其选择可以是：允许通过 `Base` 指针对其进行非虚调用（但当它调用了虚函数时可具有虚行为，比如在 NVI 或者模板方法模式中那样），进行虚调用，或者完全不能调用。NVI 模式是一种避免公开虚函数的技巧。

析构可以仅仅被看做是另一种操作，虽然它带有特殊的语义，并且非虚调用要么很危险要么是错误的。因而，对于基类析构函数来说，其选择有：允许通过 `Base` 指针进行虚函数调用，或者完全不能调用；“非虚调用”是不行的。这样的话，基类析构函数当其可以被调用（即为 public）时应当是 virtual 的，否则就为非 virtual。

注意 NVI 模式并不适用于析构函数，因为构造函数和析构函数无法进行深析构调用。（参见条款 39 和 55。）

推论：当编写基类时，一定要明确编写析构函数，因为隐式生成的析构函数是 public 和非 virtual 的。当预置函数体没问题是你当然可以用 `=default`，而仅仅为其指定正确的可见性和虚函数性质即可。

##### 例外

某些组件体系架构（如 COM 和 CORBA）并不适用标准的删除机制，而是为对象的处置设立了不同的方案。请遵循相应的模式和惯用法，并在适当时采纳本条指导方针。

再考虑一下这种罕见情况：

* `B` 既是一个基类，也是可以被实例化的具体类，因而其析构函数必须为 public 以便 `B` 的对象可以创建和销毁。
* 而 `B` 也没有虚函数，且并不打算按多态方式使用，因此虽然其析构函数是 public 它也不必是 virtual 的。

这样的话，虽然析构函数必须为 public，也会有很强的压力来阻止它变为 virtual，因为作为第一个 virtual 函数，若其所添加的功能永远不会被利用的话，它就会损害所有的运行时类型开销。

在这种罕见情况下，可以是析构函数 public 且非 virtual，但要明确说明其派生类对象绝不能当作 `B` 来多态地使用。我们对 `std::unary_function` 正是这样做的。

不过，一般来说应当避免具体的基类（参见条款 35）。例如，`unary_function` 不过是聚合了一组 typedef，它不可能会被有意单独实例化。给它提供 public 的析构函数完全没有任何意义；更好的设计应当是遵循本条款的建议来给它一个 protected 非虚析构函数猜到。

**参考**: [\[SuttAlex05\]](#suttalex05) Item 50, [\[Cargill92\]](#cargill92) pp. 77-79, 207? [\[Cline99\]](#cline99) §21.06, 21.12-13? [\[Henricson97\]](#henricson97) pp. 110-114? [\[Koenig97\]](#koenig97) Chapters 4, 11? [\[Meyers97\]](#meyers97) §14? [\[Stroustrup00\]](#stroustrup00) §12.4.2? [\[Sutter02\]](#sutter02) §27? [\[Sutter04\]](#sutter04) §18

### <a id="sd-noexcept"></a>讨论: noexcept 的用法

???

### <a id="sd-never-fail"></a>讨论: 虚构函数，回收函数和 swap 不允许失败

绝不能允许从虚构函数，资源回收函数（如 `operator delete`），或者 `swap` 函数中用 `throw` 来报告错误。如果这些操作可以失败的话，就几乎不可能编写有用的代码了，而且即便真的发生了某种错误，也几乎不可能有进行重试的任何意义。特别是，C++ 标准库是直截了当地禁止使用可能在析构函数中抛出异常的类型的。现在，大多数析构函数缺省就隐含带有 `noexcept` 了。

##### 示例

    class Nefarious {
    public:
        Nefarious() { /* 可能抛出异常的代码 */ }    // 好
        ~Nefarious() { /* 可能抛出异常的代码 */ }   // 不好, 可能抛出异常
        // ...
    };

1. `Nefarious` 对象很难安全地使用，即便是作为局部变量也是如此：


        void test(string& s)
        {
            Nefarious n;          // 要有麻烦了
            string copy = s;      // 复制 string
        } // 先后销毁 copy 和 n

    这里，对 `s` 的复制可能抛出异常，且当其抛出了异常而 `n` 的析构函数也抛出了异常时，程序就会因调用 `std::terminate` 而退出，因为无法同时传播两个异常。

2. 以 `Nefarious` 为成员或者基类的类同样很难安全地使用，因为它们的析构函数必须调用 `Nefarious` 的析构函数，且同样遭受其糟糕行为的毒害：


        class Innocent_bystander {
            Nefarious member;     // 噢，毒害了外围类的析构函数
            // ...
        };

        void test(string& s)
        {
            Innocent_bystander i;  // 要有更多麻烦了
            string copy2 = s;      // 复制 string
        } // 依次销毁 copy 和 i

    这里，当 `copy2` 的构造中抛出了异常时，我们会遇到同样的问题，因为 `i` 的析构函数现在也会抛出异常，且因此会使我们调用 `std::terminate`。

3. 你也无法可靠地创建全局或静态的 `Nefarious` 对象：


        static Nefarious n;       // 噢，无法捕获任何析构函数异常

4. 你无法可靠地创建 `Nefarious` 的数组：


        void test()
        {
            std::array<Nefarious, 10> arr; // 这行代码会导致 std::terminate()
        }

    当出现可能抛出异常的析构函数时，数组的行为是未定义的，因为根本不可能发明出合理的回退行为。请想象一下：编译器如何才能生成用来构造 `arr` 的代码，如果第四个对象的构造函数抛出了异常，这段代码必须放弃，且在其清理模式中将试图调用已经构造完成的每个对象的析构函数……而这些析构函数中的一个或更多会抛出异常呢？不存在令人满意的答案。

5. 你无法在标准容器中使用 `Nefarious`：


        std::vector<Nefarious> vec(10);   // 这行代码会导致 std::terminate()

    标准库禁止其所使用的任何析构函数抛出异常。你无法把 `Nefarious` 对象存储到标准容器中，或者在标准库的任何其他组件上使用它们。

##### 注解

它们是绝不能失败的关键函数，因为在事务性编程中需要它们提供两种关键操作：当处理过程中遇到问题时撤回工作，以及当未发生问题时提交工作。如果没有办法可以用无失败操作来安全地撤回的话，就不可能实现无失败的回滚操作。如果没有办法可以用无失败操作（显然 `swap` 可以，但并不仅限于它）来安全地提交状态的改变的话，就不可能实现无失败的提交操作。

请考虑以下在 C++ 标准中所找到的建议和要求：

> 当在栈回溯过程中所调用的析构函数因为异常而退出时，将调用 terminate (15.5.1)。因此析构函数通常应当捕获异常，并防止它们被传播出析构函数。 --[\[C++03\]](#cplusplus03) §15.2(3)
>
> C++ 标准库中所定义的任何析构函数（也包括用于实例化标准库模板的任何类型的析构函数）的操作都不会抛出异常。 --[\[C++03\]](#cplusplus03) §17.4.4.8(3)

包括专门重载的 `operator delete` 和 `operator delete[]` 在内的回收函数也属于这一类别，因为一般它们也被用在清理过程，尤其是在异常处理过程中，用以对部分完成的工作进行撤回。
除了析构函数和回收函数之外，一般的错误安全性技术也依赖于永不失败的 `swap` 操作——这种情况下，它们不仅用于实现确保成功的回滚操作，也用于实现确保成功的提交操作。例如，以下是对类型 `T` 的一种惯用的 `operator=` 实现，它在复制构造之后，调用了无失败的 `swap`：

    T& T::operator=(const T& other)
    {
        auto temp = other;
        swap(temp);
        return *this;
    }

(另见条款 56。 ???)

幸运的是，当进行资源释放时，发生故障的范围肯定会比较小。如果使用异常作为错误报告机制的话，请确保这样的函数会处理其内部的处理中可能会产生的所有异常和其他错误。（对于异常，可以直接把你的析构函数中的所有相关部分都包围到一个 `try/catch(...)` 块中。）这点非常重要，因为析构函数可能会在某种紧要关头被调用，比如当无法分配某种系统资源（如内存、文件、锁、端口、窗口，或者其他系统对象）的时候。

当使用异常作为错误处理机制的时候，请始终明示这种行为，将这些函数声明为 `noexcept`。（参见条款 75。）

**参考**: [\[SuttAlex05\]](#suttalex05) Item 51; [\[C++03\]](#cplusplus03) §15.2(3), §17.4.4.8(3)? [\[Meyers96\]](#meyers96) §11? [\[Stroustrup00\]](#stroustrup00) §14.4.7, §E.2-4? [\[Sutter00\]](#sutter00) §8, §16? [\[Sutter02\]](#sutter02) §18-19

## <a id="sd-consistent"></a>统一对复制、移动和销毁操作进行定义

##### 理由

 ???

##### 注解

一旦定义了复制构造函数，就也得定义复制赋值运算符。

##### 注解

一旦定义了移动构造函数，就也得定义移动赋值运算符。

##### 示例

    class X {
    public:
        X(const x&) { /* stuff */ }

        // 不好: 未同时定义复制赋值运算符

        X(x&&) noexcept { /* stuff */ }

        // 不好: 未同时定义移动赋值运算符

        // ...
    };

    X x1;
    X x2 = x1; // ok
    x2 = x1;   // 陷阱：要么不能通过编译，要么会做出不好的事

一旦定义了析构函数，就不能再使用编译器所生成的复制或移动操作了；你可能需要定义或者抑制掉移动或复制操作。

    class X {
        HANDLE hnd;
        // ...
    public:
        ~X() { /* 自定义的行为，比如关闭 hnd */ }
        // 可疑: 未提到过复制或移动操作——hnd 会怎么样？
    };

    X x1;
    X x2 = x1; // 陷阱：要么不能通过编译，要么会做出不好的事
    x2 = x1;   // 陷阱：要么不能通过编译，要么会做出不好的事

如果定义了复制操作，且有任何基类或成员的诶性定义了移动操作的话，应当同样定义移动操作。

    class X {
        string s; // 定义了更高效的移动操作
        // ... 其他数据成员 ...
    public:
        X(const X&) { /* stuff */ }
        X& operator=(const X&) { /* stuff */ }

        //    不好: 并未一同定义移动构造函数和移动赋值
        //   （为何不把那些自定义的“stuff”重复一下呢？）
    };

    X test()
    {
        X local;
        // ...
        return local;  // 陷阱：可能会低效甚或产生错误的行为
    }

一旦定义了复制构造函数，复制赋值运算符，或者析构函数中额任何一个，你就可能需要也定义其他的。

##### 注解

如果需要定义这五个函数中的任何一个，这就意味着你需要得到与其预置行为不同的行为——而这五者则是非对称相关的。如下所述：

* 当编写或禁用复制构造函数或复制赋值运算符之一时，很可能需要对另一个同样对待：若其中之一有“特别的”任务，则很可能另一个也应当如此，因为这两个函数应当具有相似的效果。（参见条款 53，其中对这点进行专门的展开说明。）
* 当明确编写复制函数时，很可能需要编写析构函数：若复制构造函数所做的“特别的”任务为分配或复制某中资源（诸如内存、文件、socket等），则需要在析构函数中对其进行回收。
* 当明确编写析构函数时，很可能需要明确编写或禁用复制操作：若不得不编写一个不平凡的析构函数的话，这通常是由于你需要人工释放对象所持有的某个资源。若是如此的话，很可能需要特别小心这些资源的复制，而你就需要关注对象进行复制和赋值的方式，或者完全禁止复制操作。

许多情况下，持有以 RAII 的“拥有者”对象恰当封装了的资源，是能够把自己编写这些操作的需要消除掉的。（参见条款 13。）

应当优先采用编译器生成（包括 `=default`）的特殊成员；只有它们才被归类为“平凡的”，而且至少有一家主要的标准库供应商针对带有平凡特殊成员的类进行了大量地优化。这可能会成为一种常规实践。

**例外**: 当特殊函数的声明仅为了使其非公开或者为虚函数而没有特殊的语义时，它并不导致需要其他的特殊成员。
少数情况下，带有奇怪类型的成员（诸如引用成员）的类也是例外，因为它们的复制语义很古怪。
在持有引用的类中，你可能需要编写复制构造函数和赋值运算符，但预置的析构函数仍能够做出正确的处理。（需要注意，基本上使用引用成员几乎总是错误的。）

**参考**: [\[SuttAlex05\]](#suttalex05) Item 52; [\[Cline99\]](#cline99) §30.01-14? [\[Koenig97\]](#koenig97) §4? [\[Stroustrup00\]](#stroustrup00) §5.5, §10.4? [\[SuttHysl04b\]](#sutthysl04b)

资源管理规则概览：

* [提供强资源安全性；亦即，绝不让你认为是资源的任何东西发生泄漏](#cr-safety)
* [绝不在持有未被句柄所拥有的资源时返回或抛出异常](#cr-never)
* [“原生”的指针或引用不可能是资源句柄](#cr-raw)
* [绝不让指针的生存期超过其所指向的对象](#cr-outlive)
* [用模板来表现容器（和其他资源句柄）](#cr-templates)
* [按值返回容器（依靠移动或复制消除来获得性能）](#cr-value-return)
* [若类为资源句柄，则它需要构造函数，析构函数，复制以及移动操作](#cr-handle)
* [若类为容器，则应为其提供一个初始化式列表构造函数](#cr-list)

### <a id="cr-safety"></a>讨论：提供强资源安全性；亦即，绝不让你认为是资源的任何东西发生泄漏

##### 理由

避免泄漏。泄漏会导致性能损耗，发生神秘的错误，系统崩溃，以及安全性的违犯。

**其他形式**: 使所有资源都表示为某种可以自我管理生存期的类的对象。

##### 示例

    template<class T>
    class Vector {
    private:
        T* elem;   // 自由存储中的 sz 个元素，由类对象所拥有
        int sz;
        // ...
    };

这个类是一个资源句柄。它管理各个 `T` 对象的生存期。为此，`Vector` 必然要对[一组特殊操作](???)（几个构造函数，析构函数，等等）进行定义或弃置。

##### 示例

    ??? “奇异的”非内存资源 ???

##### 强制实施

防止泄漏的基本技巧是让所有的资源都被某种带有回档析构函数的资源句柄所拥有。检查工具能够查找出“裸 `new`”。给定一组 C 风格的分配函数（如 `fopen()`），检查工具也能够查找出未被资源句柄管理的使用点。一般来说，可以带着怀疑看待“裸指针”，对其进行标记和分析。如果没有人为输入的话，时无法产生资源的完整列表的（“资源”的定义有些过于宽泛），不过可以用一个资源列表来对工具进行“参数化”。

### <a id="cr-never"></a>讨论：绝不在持有未被句柄所拥有的资源时返回或抛出异常

##### 理由

这会导致泄漏。

##### 示例

    void f(int i)
    {
        FILE* f = fopen("a file", "r");
        ifstream is { "another file" };
        // ...
        if (i == 0) return;
        // ...
        fclose(f);
    }

当 `i == 0` 时 `a file` 的文件句柄就会泄漏。另一方面，`another file` 的 `ifstream` 则将会（在销毁时）正确关闭它的文件。如果你必须显式使用指针而不是带有特定语义的资源句柄的话，可以使用带有自定义删除器的 `unique_ptr` 或 `shared_ptr`：

    void f(int i)
    {
        unique_ptr<FILE, int(*)(FILE*)> f(fopen("a file", "r"), fclose);
        // ...
        if (i == 0) return;
        // ...
    }

这样更好：

    void f(int i)
    {
        ifstream input {"a file"};
        // ...
        if (i == 0) return;
        // ...
    }

##### 强制实施

检查器必须将任何“裸指针”当作可疑处理。
检查器可能必须依赖于人工提供的资源列表进行工作。
上手时，我们知道标准库容器，`string`，以及智能指针。
`span` 和 `string_view` 的使用能够提供巨大的帮助（它们并非资源句柄）。

### <a id="cr-raw"></a>讨论：“原生”的指针或引用不可能是资源句柄

##### 理由

使得能够区分所有者和视图。

##### 注解

这和你如何“拼写”指针是两回事：`T*`，`T&`，`Ptr<T>` 和 `Range<T>` 都不是所有者。

### <a id="cr-outlive"></a>讨论：绝不让指针的生存期超过其所指向的对象

##### 理由

避免极难找到的错误。这种指针的解引用时未定义行为，能够导致发生对类型系统的违犯。

##### 示例

    string* bad()   // 确实很坏
    {
        vector<string> v = { "This", "will", "cause", "trouble", "!" };
        // 导致指向已经销毁的对象（v）的已经销毁的成员的一个指针被泄漏出去
        return &v[0];
    }

    void use()
    {
        string* p = bad();
        vector<int> xx = {7, 8, 9};
        // 未定义行为: x 可能不是字符串 "This"
        string x = *p;
        // 未定义行为: 我们不知道在位置 p 上分配的到底是什么（如果有的话）
        *p = "Evil!";
    }

`v` 中的各个 `string` 都在 `bad()` 退出之时被销毁了， `v` 自身也是如此。其所返回的指针指向自由存储上的未分配内存。（由 `p` 所指向的）这块内存，在执行 `*p` 之时可能已经被重新分配了。此时很可能并不存在可以读取的 `string` 对象，而通过 `p` 进行写入则会轻易损坏某些无关类型的对象。

##### 强制实施

大多数编译器已经能对简单情况进行警告，而且它们带有可以更进一步的信息。将函数所返回的任何指针都当作是可疑的。用容器、资源句柄和视图（例如 `span`，它不是资源句柄）来减少需要检查的情形。上手时，可将带有析构函数的类都当作是资源句柄处理。

### <a id="cr-templates"></a>讨论：用模板来表现容器（和其他资源句柄）

##### 理由

提供静态类型安全的元素操作。

##### 示例

    template<typename T> class Vector {
        // ...
        T* elem;   // 指向 sz 个 T 类型的元素
        int sz;
    };

### <a id="cr-value-return"></a>讨论：按值返回容器（依靠移动或复制消除来获得性能）

##### 理由

简化代码并消除一种进行显式内存管理的需要。将对象递交给外围作用域，由此扩展其生存期。

**参见**：[F.20，有关“输出（Out）”值的一般条款](#rf-out)

##### 示例

    vector<int> get_large_vector()
    {
        return ...;
    }

    auto v = get_large_vector(); //  按值返回没有问题，大多数现代编译器都会进行复制消除

##### 例外

见 [F.20](#rf-out) 中的例外。

##### 强制实施

检查函数所返回额指针和引用，看看它们是否被赋值给资源句柄（如 `unique_ptr`）。

### <a id="cr-handle"></a>讨论：若类为资源句柄，则它需要构造函数，析构函数，复制以及移动操作

##### 理由

以提供对资源的生存期的完全控制。以提供一组协调的对资源的操作。

##### 示例

    ??? 折腾指针

##### 注解

若所有的成员都为资源句柄，则尽可能要依赖预置的特殊操作。

    template<typename T> struct Named {
        string name;
        T value;
    };

现在 `Named` 带有一个默认构造函数，一个析构函数，以及高效的复制和移动操作，只要 `T` 也提供了它们。

##### 强制实施

一般来说，工具是无法知道类是否是资源句柄的。不过，如果类带有某种[默认操作](#ss-ctor)的话, 它就得拥有全部，而如果类中有成员为资源句柄的话，它也应被当做是资源句柄。

### <a id="cr-list"></a>讨论：若类为容器，则应为其提供一个初始化式列表构造函数

##### 理由

提供一组初始元素是一种常见情形。

##### 示例

    template<typename T> class Vector {
    public:
        Vector(std::initializer_list<T>);
        // ...
    };

    Vector<string> vs { "Nygaard", "Ritchie" };

##### 强制实施

类怎么算作是容器呢？ ???

# <a id="s-tools"></a>附录 D: 支持工具

这个部分列出了直接支持采用 C++ 核心指导方针的一些工具。这个列表并非要穷尽那些有助于编写良好的 C++ 代码的工具。
如果一个工具被专门设计以支持并关联到 C++ 核心指导方针，那它就是包括进来的候选者。

### <a id="st-clangtidy"></a>工具: [Clang-tidy](http://clang.llvm.org/extra/clang-tidy/checks/list.html)

Clang-tidy 有一组专门用于强制实施 C++ 核心指导方针的规则。这些规则的命名模式为 `cppcoreguidelines-*`。

### <a id="st-cppcorecheck"></a>工具: [CppCoreCheck](https://docs.microsoft.com/en-us/visualstudio/code-quality/using-the-cpp-core-guidelines-checkers)

微软编译器的 C++ 代码分析中包含一组专门用于强制实施 C++ 核心指导方针的规则。

# <a id="s-glossary"></a>词汇表

这是在指导方针中用到的一些术语的相对非正式的定义
（基于 [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html) 中的词汇表）。

有关 C++ 的许多主题的更多信息，可以在[标准 C++ 基金会的网站](https://isocpp.org) 找到。

* *ABI*: 应用二进制接口，对于特定硬件平台与操作系统的组合的一种规范。与 API 相对。
* *抽象类（abstract class）*: 不能直接用于创建对象的类；通常用于为派生类定义接口。
  当类带有纯虚函数或只有受保护的构造函数时，它就是抽象的。
* *抽象（abstraction）*: 对事物的描述，有选择并有意忽略（隐藏）了细节（如实现细节）；选择性忽略。
* *地址（address）*: 用以在计算机的内存中找到某个对象的值。
* *算法（algorithm）*: 用以解决某个问题的过程或公式；有限的一系列计算步骤以产生一个结果。
* *别名（alias）*: 指代某个对象的替代方式；通常为名字，指针，或者引用。
* *API*: 应用编程接口，一组构成不同软件之间之间的交互的函数。与 ABI 相对。
* *应用程序（application）*: 程序或程序的集合，用户将其看作一个实体。
* *近似（approximation）*: 事物（比如值或者设计），接近于完美的或者理想的（值或设计）。
  通常近似都是在理想情形中进行各种权衡的结果。
* *参数/实参（argument）*: 传递给函数或模板的值，其中以形参来进行访问。
* *数组（array）*: 同质元素序列，通常是数值，例如 `[0:max)`。
* *断言（assertion）*: 插入到程序中的语句，以声称（断言）在程序的这个位置某事物必定为真。
* *基类（base class）*: 目的是为从其进行派生的类型（比如它带有非 `final` 的虚函数），且有意仅间接使用该类型的对象（如通过指针）。\[严格地说，“基类”可被定义为“从之进行派生的类型”，但我们这里以类设计者意图的角度来给出定义\] 通常基类带有一个或更多的虚函数。
* *位（bit）*: 计算机中信息的基本单位。一个位的值可以为 0 或 1。
* *bug*: 程序中的错误。
* *字节（byte）*: 大多数计算机中进行寻址的基本单位。通常一个字节有 8 位。
* *类（class）*: 一种用户定义的类型，可以包含数据成员，函数成员，以及成员类型。
* *代码（code）*: 程序或程序的部分；有歧义地同时用于源代码和目标代码。
* *编译器（compiler）*: 一种将源代码变为目标代码的程序。
* *复杂度（complexity）*: 对某个问题构造解决方案的难度，或解决方案自身的一种难于精确定义的记法或度量。
  有时候复杂度只是（简单地）表示对执行某个算法所需操作的数量的估计。
* *计算（computation）*: 执行一些代码，通常接受一些输入并产生一些输出。
* *概念（concept）*: (1) 提法，想法；(2) 一组要求，通常针对模板参数。
* *具体类（concrete type）*: 并非基类的类型，且有意直接使用该类型的对象（而非仅通过指针/间接），其大小是已知的，通常可以按程序员的意图在任何地方分配（比如静态地在运行栈上分配）。
* *常量（constant）*: （在给定作用域中）不能改变的值；不可变。
* *构造函数（constructor）*: 初始化（“构造”）一个对象的操作。
  通常构造函数会建立起不变式，并且通常会获取对象被使用时所需的资源（并通常将由析构函数所释放）。
* *容器（container）*: 持有一些元素（其他对象）的对象。
* *复制（copy）*: 制造两个对象使其值比较为相等的操作。另见移动。
* *正确性（correctness）*: 如果程序或程序片段符合其说明，则其为正确的。
  不幸的是，说明可能不完整或不一致，或者也可能无法满足用户的合理预期。
  因此为了产生可接受的代码，我们有时候比仅仅遵守形式说明要做更多的事。
* *成本（cost）*: 产生一个程序，或者执行它的耗费（如开发时间，执行时间或空间等）。
  理想情况下，成本应当是复杂度的函数。
* *定制点（customization point）*:
* *数据（data）*: 计算中所用到的值。
* *调试（debugging）*: 寻找并移除程序中的错误的行为；通常远没有测试那样系统化。
* *声明式（declaration）*: 程序中对一个名字及其类型的说明。
* *定义式（definition）*: 实体的声明式，提供了程序使用该实体所需的所有信息。
  简化版定义：分配了内存额声明。
* *派生类（derived class）*: 派生自一个或多个基类的类。
* *设计（design）*: 对软件的某个片段应当如何运作以满足其说明的一个总体描述。
* *析构函数（destructor）*: 当对象销毁（如在作用域结束时）被隐式执行（调用）的操作。它通常进行资源的释放。
* *封装（encapsulation）*: 将某些事物（如实现细节）保护为私有的，不接受未授权的访问。
* *错误（error）*: 对程序行为的合理期望（通常表现为某种需求或者一份用户指南）和程序的实际行为之间的不一致。
* *可执行程序（executable）*: 预备在计算机上运行（执行）的程序。
* *功能蔓延（feature creep）*: 为“预防万一”而向程序添加过量的功能的倾向。
* *文件（file）*: 计算机中的持久信息的容器。
* *浮点数（floating-point number）*: 计算机对实数（如 7.93 和 10.78e–3）的近似。
* *函数（function）*: 命名的代码单元，可以从程序的不同部分执行（调用）；计算的逻辑单元。
* *泛型编程（generic programming）*: 关注于算法的设计和高效实现的一种编程风格。
  泛型算法能够对所有符合其要求的参数类型正确工作。在 C++ 中，泛型编程通常使用模板进行。
* *全局变量（global variable）*: 技术上说，命名空间作用域中的具名对象。
* *句柄（handle）*: 一个类，允许通过一个成员指针或引用来访问另一个对象。另见资源，复制，移动。
* *头文件（header）*: 包含用于在程序的各个部分中共享接口的声明的文件。
* *隐藏（hiding）*: 防止一个信息片段被直接看到或访问的行为。
  例如，嵌套（内部）作用域中的名字会防止外部（外围）作用域中相同的名字被直接使用。
* *理想的（ideal）*: 我们力争达成的事物的完美版本。我们经常不得不进行各种权衡最后获得一个近似。
* *实现（implementation）*: (1) 编写代码并测试的活动；(2) 用以实现一个程序的代码。
* *无限循环（infinite loop）*: 终止条件永不为真的循环。参见重复。
* *无限递归（infinite recursion）*: 无法终止的递归，直到机器耗尽内存无法维持其调用。
  在现实中这种递归不可能是无限的，它会因某种硬件错误而终止。
* *信息隐藏（information hiding）*: 分离接口和实现，以此将用户不感兴趣的实现细节隐藏起来，并提供一种抽象的活动。
* *初始化（initialize）*: 为一个对象给定其第一个（初始）值。
* *输入（input）*: 计算中所使用的值（比如函数参数以及通过键盘所输入的字符）。
* *整数（integer）*: 整数，比如 42 和 –99。
* *接口（interface）*: 一个或一组声明，说明了一个代码片段（比如函数或者类）应当如何进行调用。
* *不变式（invariant）*: 程序中的某些点必然总为真的事物；通常用于描述对象的状态（值的集合），或者循环进入其重复的语句之前的状态。
* *重复（iteration）*: 重复执行代码片段的行为；参见递归。
* *迭代器（iterator）*: 用以标识序列中的一个元素的对象。
* *ISO*: 国际标准化组织。C++ 语言是一项 ISO 标准：ISO/IEC 14882。更多信息请参考 [iso.org](http://iso.org)。
* *程序库（library）*: 类型、函数、类等等的集合，它们实现了一组设施（抽象），预备可能被用作不止一个程序的组成部分。
* *生存期（lifetime）*: 从对象的初始化直到它变为不可用（离开作用域，被删除，或程序终止）的时间。
* *连接器（linker）*: 用以将目标代码文件和程序库合并构成一个可执行程序的程序。
* *字面量（literal）*: 直接指定一个值的写法，比如 12 指定的是整数值“十二”。
* *循环（loop）*: 重复执行的代码片段；在 C++ 中，通常是 `for` 语句或者 `while` 语句。
* *移动（move）*: 将值从一个对象转移到另一个对象，并遗留一个表示“空”的值的操作。另见复制。
* *仅可移动类型（move-only type）*：可以移动但不能复制的具体类型。
* *可变的（mutable）*: 可以改动；不可变、常量和不变量的反义词。
* *对象（object）*: (1) 已经初始化的一块具有已知类型的内存区域，持有该类型的一个值；(2) 一块内存区域。
* *目标代码（object code）*: 编译器的输出，预备作为连接器的输入（连接器以其产生可执行代码）。
* *目标文件（object file）*: 包含目标代码的文件。
* *面向对象编程（object-oriented programming）*: （OOP）一种关注类和类层次的设计和使用的编程风格。
* *操作（operation）*: 能够实施某种活动的事物，比如函数或运算符。
* *输出（output）*: 由计算所产生的值（例如函数的结果，或者在屏幕上写下的一行行字符等）。
* *溢出（overflow）*: 产生无法被其预期目标所存储的值。
* *重载（overload）*: 定义两个函数或运算符，使其具有相同名字但不同的参数（操作数）类型。
* *覆盖（override）*: 在派生类中用声明和基类中的某个虚函数具有相同名字和参数类型的函数，以此使该函数可以通过由基类所定义的接口来进行调用。
* *所有者（owner）*: 负责释放某个资源的对象。
* *范式（paradigm）*: 设计和编程风格的一种多少有些做作的术语；通常会被用于（错误地）暗示有一种范式被其他的都更优秀。
* *形参（parameter）*: 对函数或模板的一个明确输入的声明。当进行调用时，函数可以通过其形参的名字来访问向其所传递的各个实参。
* *指针（pointer）*: (1) 值，用于标识内存中的一个有类型的对象；(2) 持有这种值的变量。
* *后条件（post-condition）*: 当从一个代码片段（如函数或者循环）退出时必须满足的条件。
* *前条件（pre-condition）*: 当进入一个代码片段（如函数或者循环）时必须满足的条件。
* *程序（program）*: 足够完整以便能够在计算机上执行的代码（可能带有关联的数据）。
* *编程（programming）*: 将问题的解决方案表现为代码的工艺。
* *编程语言（programming language）*: 用于表达程序的语言。
* *伪代码（pseudo code）*: 以非正式的写法而非编程语言所编写的对计算的一种描述。
* *纯虚函数（pure virtual function）*: 必须在派生类中予以覆盖的虚函数。
* *RAII*: （“资源获取即初始化，Resource Acquisition Is Initialization”）一种基于作用域进行资源管理的基本技术。
* *范围（range）*: 值的序列，可以以一个开始点和一个结尾点进行描述。例如，`[0:5)` 的意思是值 0，1，2，3，和 4。
* *递归（recursion）*: 函数调用其自身的行为；另见重复。
* *引用（reference）*: (1) 一种值，描述内存中具有类型的值的位置；(2) 持有这种值的变量。
* *正则表达式（regular expression）*: 对字符串的模式的一种表示法。
* *正规*: 可以进行相等性比较的半正规类型（参见 `std::regular` 概念）。进行复制之后，副本对象与原对象比较为相等。正规类型的行为与如 `int` 这样的内建类型相似，且可以用 `==` 进行比较。
特别是，正规类型的对象可以进行复制，且复制的结果是与原对象比较为相等的一个独立对象。另见*半正规类型*。
* *要求（requirement）*: (1) 对程序或程序的一部分的预期行为的描述；(2) 对函数或模板对其参数所作出的假设的描述。
* *资源（resource）*: 获取而得的并随后必须被释放的事物，比如文件句柄，锁，或者内存。另见句柄，所有者。
* *舍入（rounding）*: 将一个值转换为某个较不精确类型的数学上最接近的值。
* *RTTI*: 运行时类型信息（Run-Time Type Information）。 ???
* *作用域（scope）*: 程序文本（源代码）的区域，在其中可以对一个名字进行涉指。
* *半正规（semiregular）*: 可复制的（也包括可移动的）且可默认构造的具体类型（参见 `std::semiregular` 概念）。复制的结果是一个与原对象具有相同的值的独立类型。半正规类型的行为与像 `int` 这样内建类型大致相似，但可能没有 `==` 运算符。另见*正规类型*。
* *序列（sequence）*: 可以以线性的顺序访问的一组元素。
* *软件（software）*: 代码片段及其关联数据的集合；通常可以和程序互换运用。
* *源代码（source code）*: 由程序员所生产的代码，（原则上）可以被其他程序员阅读。
* *源文件（source file）*: 包含源代码的文件。
* *规范（specification）*: 对代码片段应当做什么的描述。
* *标准（standard）*: 由官方承认的对某事物的定义，比如编程语言。
* *状态（state）*: 一组值。
* *STL*: 标准库中的容器，迭代器，以及算法部分。
* *字符串（string）*: 字符的序列。
* *风格（style）*: 旨在统一语言功能特征的使用的一组编程技巧；有时候以非常限定的方式来仅代表诸如命名和代码展现等的低层次规则。
* *子类型（subtype）*: 派生类型；一个类型具有另一个类型的所有（可能更多）的性质。
* *超类型（supertype）*: 基类型；一个类型具有另一个类型的性质的子集。
* *系统（system）*: (1) 用以在计算机上实施某种任务的一个或一组程序；(2) 对“操作系统”的简称，即计算机的基本执行环境及工具。
* *TS*: [技术规范](https://www.iso.org/deliverables-all.html type=ts)。技术规范所处理的是仍处于技术开发之中的工作，或者是认为这项工作以后可能会被同意采纳为国际标准，但并不会立即处理。技术规范的出版是为了其立即可用，也是为了提供一种获得反馈的方法。其目标是最终能够被转化并重新作为国际标准来出版。
* *模板（template）*: 由一个或多个的类型或（编译时）值进行参数化的类或函数；支持泛型编程的基本 C++ 语言构造。
* *测试（testing）*: 系统化地查找程序中的错误。
* *权衡（trade-off）*: 对多个设计和实现准则进行平衡的结果。
* *截断（truncation）*: 从一个类型转换为另一个无法精确表示被转换的值的类型时发生的信息损失。
* *类型（type）*: 为一个对象定义了一组可能的值和一组操作的事物。
* *未初始化的（uninitialized）*: 对象在初始化之前的（未定义的）状态。
* *单元（unit）*: (1) 为值赋予含义的一种标准度量（例如，距离单位 km）；(2) 较大的整体中的一个可区分的（比如命名的）部分。
* *用例（use case）*: 程序的某个特定（通常简化的）使用，以测试其功能并演示其目的。
* *值（value）*: 根据某个类型所解释的一组内存中的位。
* *值类型（value type）*：一些人用这个术语来表示正规或半正规类型。
* *变量（variable）*: 给定类型的具名对象；除非未初始化否则包含一个值。
* *虚函数（virtual function）*: 可在派生类中进行覆盖的成员函数。
* *字（word）*: 计算机中内存的基本单元，通常是用以持有一个整数的单元。

# <a id="s-unclassified"></a>To-do: 未分类的规则原型

这是我们的未完成列表。
以下各条目最终将成为规则或者规则的一部分。
或者，我们也会决定不需要做出改动并将条目移除。

* 禁止远距离友元关系
* 应不应该处理物理设计（文件里有什么）和大规模设计（程序库，程序库的组合）？
* 命名空间
* 避免在全局作用域中使用 using 指令（但允许如 std 或其他的“基础”命名空间（如 experimental））
* 命名空间应当有什么粒度？是（如 Sutter/Alexandrescu 所定义的）所有被设计为一同工作或者一同发布的类和函数，还是应该更窄或是更宽？
* 应该用内联命名空间吗（比如 `std::literals::*_literals`）？
* 避免隐式转换
* Const 成员函数应当是线程安全的……aka, 但我并不想真的改掉变量，只是在第一次调用它的时候向它赋一个值……argh
* 始终初始化变量，为成员变量使用初始化列表。
* 无论谁编写了接受或返回 `void*` 的公开接口，都应该上火刑。我曾经好多年都以它作为自己的个人喜好来着。 :)
* 尽可能应用 `const`：成员函数，变量，以及 `const_iterators`
* 使用 `auto`
* `(size)` vs. `{initializers}` vs. `{Extent{size}}`
* 不要过度抽象
* 不要沿着调用栈向下传递指针
* 通过函数底部退出
* 应当提供在多态之间进行选择的指导方针吗？是的。经典的（虚函数，引用语义） vs. Sean Parent 风格（值语义，类型擦除，类似 `std::function`）  vs. CRTP/静态的？也许还需要 vs. 标签派发？
* 我们的指导方针是否应当在构造函数或析构函数中禁止进行虚函数调用？是的。许多人都禁止了，虽然我觉得这是 C++ 的一大优势 ??? -保留意见（D 走向 Java 之路太让我失望了）。有好的例子吗？
* 在 lambda 方面，在算法调用和其他回调场景中什么因素会影响决定使用 lambda 还是（局部？）类？
* 讨论一下 `std::bind`，Stephen T. Lavavej 对它有太多批评，使我开始觉得它是不是真的会在未来消失掉。应该建议以 lambda 代替它吗？
* 怎么处理泄漏的临时变量？ : `p = (s1 + s2).c_str();`
* 指针和迭代器的失效会导致悬挂指针：

        void bad()
        {
            int* p = new int[700];
            int* q = &p[7];
            delete p;

            vector<int> v(700);
            int* q2 = &v[7];
            v.resize(900);

            // ... 使用 q 和 q2 ...
        }

* LSP
* 私有继承 vs/and 成员
* 避免静态类成员变量（竞争条件，几乎就是全局变量）

* 使用 RAII 锁定保护（`lock_guard`，`unique_lock`，`shared_lock`），绝不直接调用 `mutex.lock` 和 `mutex.unlock`（RAII）
* 优先使用非递归锁（它们通常用作不良情况的变通手段，有开销）
* 联结（join）你的每个线程！（因为如果没被联结或脱离（detach）的话，析构函数会调用 `std::terminate`……有什么好理由来脱离线程吗？） -- ??? 支持库该不该为 `std::thread` 提供一个 RAII 包装呢？
* 当必须同时获取两个或更多的互斥体时，应当使用 `std::lock`（或者别的死锁免除算法？）
* 当使用 `condition_variable` 时，始终用一个互斥体来保护它（在互斥体外面设置原子 bool 的值的做法是错误的！），并对条件变量自身使用同一个互斥体。
* 绝不对 `std::atomic<user-defined-struct>` 使用 `atomic_compare_exchange_strong`（填充位中的区别会造成影响，而在循环中使用 `compare_exchange_weak` 则能够归于稳定的填充位）
* 单独的 `shared_future` 对象不是线程安全的：两个线程不能等待同一个 `shared_future` 对象（它们可以等待指代相同共享状态的 `shared_future` 的副本）
* 单独的 `shared_ptr` 对象不是线程安全的：不同的线程可以调用指代相同共享对象的*不同* `shared_ptr` 的非 `const` 成员函数，但当一个线程访问一个 `shared_ptr` 对象时，另一个线程不能调用相同 `shared_ptr` 对象的非 `const` 成员函数（如果确实需要，考虑代之以 `atomic_shared_ptr`）

* 算术相关规则

# 参考文献

* <a id="abrahams01"></a>
  \[Abrahams01]:  D. Abrahams. [Exception-Safety in Generic Components](http://www.boost.org/community/exception_safety.html).
* <a id="alexandrescu01"></a>
  \[Alexandrescu01]:  A. Alexandrescu. Modern C++ Design (Addison-Wesley, 2001).
* <a id="cplusplus03"></a>
  \[C++03]:           ISO/IEC 14882:2003(E), Programming Languages — C++ (updated ISO and ANSI C++ Standard including the contents of (C++98) plus errata corrections).
* <a id="cargill92"></a>
  \[Cargill92]:       T. Cargill. C++ Programming Style (Addison-Wesley, 1992).
* <a id="cline99"></a>
  \[Cline99]:         M. Cline, G. Lomow, and M. Girou. C++ FAQs (2ndEdition) (Addison-Wesley, 1999).
* <a id="dewhurst03"></a>
  \[Dewhurst03]:      S. Dewhurst. C++ Gotchas (Addison-Wesley, 2003).
* <a id="henricson97"></a>
  \[Henricson97]:     M. Henricson and E. Nyquist. Industrial Strength C++ (Prentice Hall, 1997).
* <a id="koenig97"></a>
  \[Koenig97]:        A. Koenig and B. Moo. Ruminations on C++ (Addison-Wesley, 1997).
* <a id="lakos96"></a>
  \[Lakos96]:         J. Lakos. Large-Scale C++ Software Design (Addison-Wesley, 1996).
* <a id="meyers96"></a>
  \[Meyers96]:        S. Meyers. More Effective C++ (Addison-Wesley, 1996).
* <a id="meyers97"></a>
  \[Meyers97]:        S. Meyers. Effective C++ (2nd Edition) (Addison-Wesley, 1997).
* <a id="meyers01"></a>
  \[Meyers01]:        S. Meyers. Effective STL (Addison-Wesley, 2001).
* <a id="meyers05"></a>
  \[Meyers05]:        S. Meyers. Effective C++ (3rd Edition) (Addison-Wesley, 2005).
* <a id="meyers15"></a>
  \[Meyers15]:        S. Meyers. Effective Modern C++ (O'Reilly, 2015).
* <a id="murray93"></a>
  \[Murray93]:        R. Murray. C++ Strategies and Tactics (Addison-Wesley, 1993).
* <a id="stroustrup94"></a>
  \[Stroustrup94]:    B. Stroustrup. The Design and Evolution of C++ (Addison-Wesley, 1994).
* <a id="stroustrup00"></a>
  \[Stroustrup00]:    B. Stroustrup. The C++ Programming Language (Special 3rdEdition) (Addison-Wesley, 2000).
* <a id="stroustrup05"></a>
  \[Stroustrup05]:    B. Stroustrup. [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
* <a id="stroustrup13"></a>
  \[Stroustrup13]:    B. Stroustrup. [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html). Addison Wesley 2013.
* <a id="stroustrup14"></a>
  \[Stroustrup14]:    B. Stroustrup. [A Tour of C++](http://www.stroustrup.com/Tour.html).
  Addison Wesley 2014.
* <a id="Stroustrup15></a>
  \[Stroustrup15]:    B. Stroustrup, Herb Sutter, and G. Dos Reis: [A brief introduction to C++'s model for type- and resource-safety](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Introduction%20to%20type%20and%20resource%20safety.pdf).
* <a id="sutthysl04b"></a>
  \[SuttHysl04b]:     H. Sutter and J. Hyslop. [Collecting Shared Objects](https://web.archive.org/web/20120926011837/http://www.drdobbs.com/collecting-shared-objects/184401839) (C/C++ Users Journal, 22(8), August 2004).
* <a id="suttalex05"></a>
  \[SuttAlex05]:      H. Sutter and  A. Alexandrescu. C++ Coding Standards. Addison-Wesley 2005.
* <a id="sutter00"></a>
  \[Sutter00]:        H. Sutter. Exceptional C++ (Addison-Wesley, 2000).
* <a id="sutter02"></a>
  \[Sutter02]:        H. Sutter. More Exceptional C++ (Addison-Wesley, 2002).
* <a id="sutter04"></a>
  \[Sutter04]:        H. Sutter. Exceptional C++ Style (Addison-Wesley, 2004).
* <a id="taligent94"></a>
  \[Taligent94]: Taligent's Guide to Designing Programs (Addison-Wesley, 1994).
