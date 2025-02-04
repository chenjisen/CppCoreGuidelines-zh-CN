
# <a id="s-concurrency"></a>CP: 并发与并行

我们经常想要我们的计算机同时运行许多任务（或者至少表现为同时运行它们）。
这有许多不同的原因（例如，需要仅用一个处理器等待许多事件，同时处理许多数据流，或者利用大量的硬件设施）
因此也由许多不同的用以表现并发和并行的基本设施。
我们这里将说明使用 ISO 标准 C++ 中用以表现基本并发和并行的设施的原则和规则。

线程是对并发和并行编程支持的机器级别的基础。
使用线程允许互不相关地运行程序的多个部分，
同时共享相同的内存。并发编程很麻烦，
因为在线程之间保护共享的数据说起来容易做起来难。
使现存的单线程代码可以并发执行，
可以通过策略性地添加 `std::async` 或 `std::thread` 这样简单做到，
也可能需要进行完全的重写，这依赖于原始代码是否是以线程友好
的方式编写的。

本文档中的并发/并行规则的设计有三个
目标：

* 有助于编写可以被修改为能够在线程环境中使用
  的代码。
* 展示使用标准库所提供的线程原语的简洁，
  安全的方式。
* 对当并发和并行无法提供所需要的性能增益时应当如何做
  提供指导。

同样重要的一点，是要注意 C++ 的并发仍然是未完成的工作。
C++11 引入了许多核心并发原语，C++14 和 C++17 对它们进行了改进，
而且在使 C++ 编写并发程序更加简单的方面
仍有许多关注。我们预计这里的一些与程序库相关
的指导方针会随着时间有大量的改动。

这一部分需要大量的工作（显然如此）。
请注意我们的规则是从相对非专家们入手的。
真正的专家们还请稍等；
我们欢迎贡献者，
但请为正在努力使他们的并发程序正确且高效的大多数程序员着想。

并发和并行规则概览：

* [CP.1: 假定你的代码将作为多线程程序的一部分而运行](#rconc-multi)
* [CP.2: 避免数据竞争](#rconc-races)
* [CP.3: 最小化可写数据的明确共享](#rconc-data)
* [CP.4: 以任务而不是线程的角度思考](#rconc-task)
* [CP.8: 不要为同步而使用 `volatile`](#rconc-volatile)
* [CP.9: 只要可行，就使用工具对并发代码进行验证](#rconc-tools)

**参见**：

* [CP.con: 并发](#sscp-con)
* [CP.coro: 协程](#sscp-coro)
* [CP.par: 并行](#sscp-par)
* [CP.mess: 消息传递](#sscp-mess)
* [CP.vec: 向量化](#sscp-vec)
* [CP.free: 无锁编程](#sscp-free)
* [CP.etc: 其他并发规则](#sscp-etc)

### <a id="rconc-multi"></a>CP.1: 假定你的代码将作为多线程程序的一部分而运行

##### 理由

很难说现在或者未来什么时候会不会需要使用并发。
代码是会被重用的。
程序的其他使用了线程的部分可能会使用某个未使用线程的程序库。
请注意这条规则对于程序库代码来说最紧迫，而对独立的应用程序来说则最不紧迫。
不过，久而久之，代码片段可能出现在意想不到的地方。

##### 示例，不好

    double cached_computation(int x)
    {
        // 不好：这些静态变量导致多线程的使用情况中的数据竞争
        static int cached_x = 0.0;
        static double cached_result = COMPUTATION_OF_ZERO;

        if (cached_x != x) {
            cached_x = x;
            cached_result = computation(x);
        }
        return cached_result;
    }

虽然 `cached_computation` 在单线程环境中可以正确工作，但在多线程环境中，其两个 `static` 变量将导致数据竞争进而发生未定义的行为。

##### 示例，好

    struct ComputationCache {
        int cached_x = 0;
        double cached_result = COMPUTATION_OF_ZERO;

        double compute(int x) {
            if (cached_x != x) {
                cached_x = x;
                cached_result = computation(x);
            }
            return cached_result;
        }
    };

这里的缓存作为 `ComputationCache` 对象的成员数据保存，而非共享的静态状态。
这项重构本质上是将关注点委派给了调用方：
单线程程序可能仍会选择采用一个全局的 `ComputationCache` 对象，
而多线程程序可能会为每个线程准备一个 `ComputationCache`，或者为任意定义的“上下文”每个准备一个。
重构后的函数不再试图管理 `cached_x` 的分配。
这方面可以看做是对单一职责原则（SRP）的一次应用。

在这个特定的例子中，为线程安全性进行的重构同样改进了其在单线程程序中的可用性。
不难想象，某个单线程程序可能需要在程序的不同部分中使用两个 `ComputationCache` 实例，
并且它们并不会覆盖掉互相的缓冲数据。

还有其他的一些方法，可以为针对标准多线程环境（就是唯一的并发形式是 `std::thread` 的环境）编写的
代码添加线程安全性：

* 将状态变量标为 `thread_local` 而非 `static`。
* 实现并发控制逻辑，例如，用一个 `static std::mutex` 来保护对这两个 `static` 变量的访问。
* 拒绝在多线程环境中进行构建和/或运行。
* 提供两个实现，一个用在单线程环境中，另一个用在多线程环境中。

##### 例外

永远不会在多线程环境中执行的代码。

要小心的是：有许多例子，“认为”永远不会在多线程程序中执行的代码
却真的在多线程程序中执行了，通常是在多年以后。
一般来说，这种程序将导致进行痛苦的移除数据竞争的工作。
因此，确实有意不在多线程环境中执行的代码，应当清晰地进行标注，而且理想情况下应当利用编译或运行时的强制机制来提早检测到这种使用情况。

### <a id="rconc-races"></a>CP.2: 避免数据竞争

##### 理由

不这样的话，则任何东西都不保证能工作，而且可能出现微妙的错误。

##### 注解

简而言之，当两个线程并发（未同步）地访问同一个对象，且至少一方为写入方（实施某个非 `const` 操作）时，就会出现数据竞争。
有关如何正确使用同步来消除数据竞争的更多信息，请求教于一本有关并发的优秀书籍（参见 [认真学习文献](#rconc-literature)）。

##### 示例，不好

有大量存在数据竞争的例子，其中的一些现在这个时候就运行在
产品软件之中。一个非常简单的例子是：

    int get_id()
    {
      static int id = 1;
      return id++;
    }

这里的增量操作就是数据竞争的一个例子。这可能以许多方式导致发生错误，
包括：

* 线程 A 加载 `id` 的值，OS 上下文切换使 A 离开
  一段时间，其中有其他线程创建了上百个 ID。线程 A
  再次允许执行，而 `id` 被写回到那个位置，其值为 A
  所读取的 `id` 值加一。
* 线程 A 和 B 同时加载 `id` 并进行增量。它们都将获得
  相同的 ID。

局部静态变量是数据竞争的一种常见来源。

##### 示例，不好

    void f(fstream& fs, regex pattern)
    {
        array<double, max> buf;
        int sz = read_vec(fs, buf, max);            // 从 fs 读取到 buf 中
        gsl::span<double> s {buf};
        // ...
        auto h1 = async([&] { sort(std::execution::par, s); });     // 产生一个进行排序的任务
        // ...
        auto h2 = async([&] { return find_all(buf, sz, pattern); });   // 产生一个查找匹配的任务
        // ...
    }

这里，在 `buf` 的元素上有一个（很讨厌的）数据竞争（`sort` 既会读取也会写入）。
所有的数据竞争都很讨厌。
我们这里设法在栈上的数据上造成了数据竞争。
不是所有数据竞争都像这个这样容易找出来的。

##### 示例，不好

    // 未用锁进行控制的代码

    unsigned val;

    if (val < 5) {
        // ... 其他线程可能在这里改动 val ...
        switch (val) {
        case 0: // ...
        case 1: // ...
        case 2: // ...
        case 3: // ...
        case 4: // ...
        }
    }

这里，若编译器不知道 `val` 可能改变，它最可能将这个 `switch` 实现为一个有五个项目的跳转表。
然后，超出 `[0..4]` 范围的 `val` 值将会导致跳转到程序中的任何可能位置的地址，从那个位置继续执行。
真的，当你遇到数据竞争时什么都可能发生。
实际上可能更为糟糕：通过查看生成的代码，是可以确定这个走偏的跳转针对给定的值将会跳转到哪里的；
这就是一种安全风险。

##### 强制实施

有些事是可能做到的，至少要做一些事。
有一些商用和开源的工具试图处理这种问题，
但要注意的是任何工具解决方案都有其成本和盲点。
静态工具通常会有许多漏报，而动态工具则通常有显著的成本。
我们希望有更好的工具。
使用多个工具可以找到比单个工具更多的问题。

还有其他方式可以缓解发生数据竞争的机会：

* 避免全局数据
* 避免 `static` 变量
* 更多地使用栈上的具体类型（且不要过多地把指针到处传递）
* 更多地使用不可变数据（字面量，`constexpr`，以及 `const`）

### <a id="rconc-data"></a>CP.3: 最小化可写数据的明确共享

##### 理由

如果不共享可写数据的话，就不会发生数据竞争。
越少进行共享，你就越少有机会忘掉对访问进行同步（而发生数据竞争）。
越少进行共享，你就越少有机会需要等待锁定（并改进性能）。

##### 示例

    bool validate(const vector<Reading>&);
    Graph<Temp_node> temperature_gradients(const vector<Reading>&);
    Image altitude_map(const vector<Reading>&);
    // ...

    void process_readings(const vector<Reading>& surface_readings)
    {
        auto h1 = async([&] { if (!validate(surface_readings)) throw Invalid_data{}; });
        auto h2 = async([&] { return temperature_gradients(surface_readings); });
        auto h3 = async([&] { return altitude_map(surface_readings); });
        // ...
        h1.get();
        auto v2 = h2.get();
        auto v3 = h3.get();
        // ...
    }

没有这些 `const` 的话，我们就必须为潜在的数据竞争而为在 `surface_readings` 上的所有异步函数调用进行复审。
使 `surface_readings` （对于这个函数）为 `const` 允许我们仅在函数体代码中进行推理。

##### 注解

不可变数据可以安全并高效地共享。
无须对其进行锁定：不可能在常量上发生数据竞争。
另请参见[CP.mess: 消息传递](#sscp-mess)和[CP.31: 优先采用按值传递](#rconc-data-by-value)。

##### 强制实施

???


### <a id="rconc-task"></a>CP.4: 以任务而不是线程的角度思考

##### 理由

`thread` 是一种实现概念，一种针对机器的思考方式。
任务则是一种应用概念，有时候你可以使任务和其他任务并发执行。
应用概念更容易进行推理。

##### 理由

    void some_fun(const std::string& msg)
    {
        std::thread publisher([=] { std::cout << msg; });      // 不好: 表达性不足
                                                               //       且更易错
        auto pubtask = std::async([=] { std::cout << msg; });  // OK
        // ...
        publisher.join();
    }

##### 注解

除了 `async()` 之外，标准库中的设施都是底层的，面向机器的，线程和锁层次的设施。
这是一种必须的基础，但我们不得不尝试提升抽象的层次：为提供生产率，可靠性，以及性能。
这对于采用更高层次的，更加面向应用的程序库（如果可能就建立在标准库设施上）的强有力的理由。

##### 强制实施

???

### <a id="rconc-volatile"></a>CP.8: 不要为同步而使用 `volatile`

##### 理由

和其他语言不同，C++ 中的 `volatile` 并不提供原子性，不会在线程之间进行同步，
而且不会防止指令重排（无论编译器还是硬件）。
它和并发完全没有关系。

##### 示例，不好

    int free_slots = max_slots; // 当前的对象内存的来源

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }

这里有一个问题：
这在单线程程序中是完全正确的代码，但若有两个线程执行，
并且在 `free_slots` 上发生竞争条件时，两个线程就可能拿到相同的值和 `free_slots`。
这（显然）是不好的数据竞争，受过其他语言训练的人可能会试图这样修正：

    volatile int free_slots = max_slots; // 当前的对象内存的来源

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }

这并没有同步效果：数据竞争仍然存在！

C++ 对此的机制是 `atomic` 类型：

    atomic<int> free_slots = max_slots; // 当前的对象内存的来源

    Pool* use()
    {
        if (int n = free_slots--) return &pool[n];
    }

现在的 `--` 操作是原子性的，
而不是可能被另一个线程介入其独立操作之间的读-增量-写序列。

##### 替代方案

在某些其他语言中曾经使用 `volatile` 的地方使用 `atomic` 类型。
为更加复杂的例子使用 `mutex`。

##### 另见

[`volatile` 的（罕见）恰当用法](#rconc-volatile2)

### <a id="rconc-tools"></a>CP.9: 只要可行，就使用工具对并发代码进行验证

经验表明，让并发代码正确是特别难的，
而编译期检查、运行时检查和测试在找出并发错误方面，
并没有如同在顺序代码中找出错误时那么有效。
一些微妙的并发错误可能会造成显著的不良后果，包括内存破坏，死锁，和安全漏洞等。

##### 示例

    ???

##### 注解

线程安全性是一种有挑战性的任务，有经验的程序员通常可以做得更好一些：缓解这些风险的一种重要策略是运用工具。
现存有不少这样的工具，既有商用的也有开源的，既有研究性的也有产品级的。
遗憾的是，人们的需求和约束条件之间有巨大的差别，使我们无法给出特定的建议，
但我们可以提一些：

* 静态强制实施工具：[clang](http://clang.llvm.org/docs/ThreadSafetyAnalysis.html)
和一些老版本的 [GCC](https://gcc.gnu.org/wiki/ThreadSafetyAnnotation)
都提供了一些针对线程安全性性质的静态代码标注。
统一地采用这项技术可以将许多种类的线程安全性错误变为编译时的错误。
这些代码标注一般都是局部的（将某个特定成员变量标记为又一个特定的互斥体进行防护），
且一般都易于学习使用。但与许多的静态工具一样，它经常会造成漏报；
这些情况应当被发现但却被其忽略了。

* 动态强制实施工具：Clang 的 [Thread Sanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html)（即 TSAN）
是动态工具的一个强有力的例子：它改变你的程序的构建和执行，向其中添加对内存访问的簿记工作，
精确地识别你的二进制程序的一次特定执行中发生的数据竞争。
使用它的代价在内存上（多数情况为五到十倍），也拖慢 CPU（二到二十倍）。
像这样的动态工具，在集成测试，金丝雀推送，以及在多个线程上操作的单元测试上实施是最好的。
它是与工作负载相关的：一旦 TSAN 识别了一个问题，那它就是实际发生的数据竞争，
但它只能识别出一次特定的执行之中发生的竞争。

##### 强制实施

对于特定的应用，应用的构建者来选择哪些支持工具是有价值的。

## <a id="sscp-con"></a>CP.con: 并发

这个部分所关注的是相对比较专门的通过共享数据进行多线程通信的用法。

* 有关并行算法，参见[并行](#sscp-par)。
* 有关不使用明确共享的任务间，通信参见[消息传递](#sscp-mess)。
* 有关向量并行代码，参见[向量化](#sscp-vec)。
* 有关无锁编程，参见[无锁](#sscp-free)。

并发规则概览：

* [CP.20: 使用 RAII，绝不使用普通的 `lock()`/`unlock()`](#rconc-raii)
* [CP.21: 用 `std::lock()` 或 `std::scoped_lock` 来获得多个 `mutex`](#rconc-lock)
* [CP.22: 绝不在持有锁的时候调用未知的代码（比如回调）](#rconc-unknown)
* [CP.23: 把联结的 `thread` 看作是有作用域的容器](#rconc-join)
* [CP.24: 把 `thread` 看作是全局的容器](#rconc-detach)
* [CP.25: 优先采用 `gsl::joining_thread` 而不是 `std::thread`](#rconc-joining_thread)
* [CP.26: 不要 `detach()` 线程](#rconc-detached_thread)
* [CP.31: 少量数据在线程之间按值传递，而不是通过引用或指针传递](#rconc-data-by-value)
* [CP.32: 用 `shared_ptr` 在无关的 `thread` 之间共享所有权](#rconc-shared)
* [CP.40: 最小化上下文切换](#rconc-switch)
* [CP.41: 最小化线程的创建和销毁](#rconc-create)
* [CP.42: 不要无条件地 `wait`](#rconc-wait)
* [CP.43: 最小化临界区的时间耗费](#rconc-time)
* [CP.44: 记得为 `lock_guard` 和 `unique_lock` 命名](#rconc-name)
* [CP.50: `mutex` 要和其所护卫的数据一起定义。一旦可能就使用 `synchronized_value<T>`](#rconc-mutex)
* ??? 何时使用 spinlock
* ??? 何时使用 `try_lock()`
* ??? 何时应优先使用 `lock_guard` 而不是 `unique_lock`
* ??? 时序多工
* ??? 何时/如何使用 `new thread`

### <a id="rconc-raii"></a>CP.20: 使用 RAII，绝不使用普通的 `lock()`/`unlock()`

##### 理由

避免源于未释放的锁定的令人讨厌的错误。

##### 示例，不好

    mutex mtx;

    void do_stuff()
    {
        mtx.lock();
        // ... 做一些事 ...
        mtx.unlock();
    }

或早或晚都会有人忘记 `mtx.unlock()`，在 `... 做一些事 ...` 中放入一个 `return`，抛出异常，或者别的什么。

    mutex mtx;

    void do_stuff()
    {
        unique_lock<mutex> lck {mtx};
        // ... 做一些事 ...
    }

##### 强制实施

标记对成员 `lock()` 和 `unlock()` 的调用。 ???


### <a id="rconc-lock"></a>CP.21: 用 `std::lock()` 或 `std::scoped_lock` 来获得多个 `mutex`

##### 理由

避免在多个 `mutex` 上造成死锁。

##### 示例

下面将导致死锁：

    // 线程 1
    lock_guard<mutex> lck1(m1);
    lock_guard<mutex> lck2(m2);

    // 线程 2
    lock_guard<mutex> lck2(m2);
    lock_guard<mutex> lck1(m1);

代之以使用 `lock()`：

    // 线程 1
    lock(m1, m2);
    lock_guard<mutex> lck1(m1, defer_lock);
    lock_guard<mutex> lck2(m2, defer_lock);

    // 线程 2
    lock(m2, m1);
    lock_guard<mutex> lck2(m2, defer_lock);
    lock_guard<mutex> lck1(m1, defer_lock);

或者（这样更佳，但仅为 C++17）：

    // 线程 1
    scoped_lock<mutex, mutex> lck1(m1, m2);

    // 线程 2
    scoped_lock<mutex, mutex> lck2(m2, m1);

这样，`thread1` 和 `thread2` 的作者们仍然未在 `mutex` 的顺序上达成一致，但顺序不再是问题了。

##### 注解

在实际代码中，`mutex` 的命名很少便于程序员记得某种有意的关系和有意的获取顺序。
在实际代码中，`mutex` 并不总是便于在连续代码行中依次获取的。

##### 注解

在 C++17 中，可以编写普通的

    lock_guard lck1(m1, adopt_lock);

而让 `mutex` 类型被推断出来。

##### 强制实施

检测多个 `mutex` 的获取。
这一般来说是无法确定的，但找出常见的简单例子（比如上面这个）则比较容易。


### <a id="rconc-unknown"></a>CP.22: 绝不在持有锁的时候调用未知的代码（比如回调）

##### 理由

如果不了解代码做了什么，就有死锁的风险。

##### 示例

    void do_this(Foo* p)
    {
        lock_guard<mutex> lck {my_mutex};
        // ... 做一些事 ...
        p->act(my_data);
        // ...
    }

如果不知道 `Foo::act` 会干什么（可能它是一个虚函数，调用某个还未编写的某个派生类成员），
它可能会（递归地）调用 `do_this` 因而在 `my_mutex` 上造成死锁。
可能它会在某个别的 `mutex` 上锁定而无法在适当的时间内返回，对任何调用了 `do_this` 的代码造成延迟。

##### 示例

“调用未知代码”问题的一个常见例子是调用了试图在相同对象上进行锁定访问的函数。
这种问题通常可以用 `recursive_mutex` 来解决。例如：

    recursive_mutex my_mutex;

    template<typename Action>
    void do_something(Action f)
    {
        unique_lock<recursive_mutex> lck {my_mutex};
        // ... 做一些事 ...
        f(this);    // f 将会对 *this 做一些事
        // ...
    }

如果如同其很可能做的那样，`f()` 调用了 `*this` 的某个操作的话，我们就必须保证在调用之前对象的不变式是满足的。

##### 强制实施

* 当持有非递归的 `mutex` 时调用虚函数则进行标记。
* 当持有非递归的 `mutex` 时调用回调则进行标记。


### <a id="rconc-join"></a>CP.23: 把联结的 `thread` 看作是有作用域的容器

##### 理由

为了维护指针安全性并避免泄漏，需要考虑 `thread` 所使用的指针。
如果 `thread` 联结了，我们可以安全地把指向这个 `thread` 所在作用域及其外围作用域中的对象的指针传递给它。

##### 示例

    void f(int* p)
    {
        // ...
        *p = 99;
        // ...
    }
    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        joining_thread t0(f, &x);           // OK
        joining_thread t1(f, p);            // OK
        joining_thread t2(f, &glob);        // OK
        auto q = make_unique<int>(99);
        joining_thread t3(f, q.get());      // OK
        // ...
    }

`gsl::joining_thread` 是一种 `std::thread`，其析构函数进行联结且不可被 `detached()`。
这里的“OK”表明对象能够在 `thread` 可以使用指向它的指针时一直处于作用域（“存活”）。
`thread` 运行的并发性并不会影响这里的生存期或所有权问题；
这些 `thread` 可以仅仅被看成是从 `some_fct` 中调用的函数对象。

##### 强制实施

确保 `joining_thread` 不会 `detach()`。
之后，可以实施（针对局部对象的）常规的生存期和所有权强制实施方案。

### <a id="rconc-detach"></a>CP.24: 把 `thread` 看作是全局的容器

##### 理由

为了维护指针安全性并避免泄漏，需要考虑 `thread` 所使用的指针。
如果 `thread` 脱离了，我们只可以安全地把指向静态和自由存储的对象的指针传递给它。

##### 示例

    void f(int* p)
    {
        // ...
        *p = 99;
        // ...
    }

    int glob = 33;

    void some_fct(int* p)
    {
        int x = 77;
        std::thread t0(f, &x);           // 不好
        std::thread t1(f, p);            // 不好
        std::thread t2(f, &glob);        // OK
        auto q = make_unique<int>(99);
        std::thread t3(f, q.get());      // 不好
        // ...
        t0.detach();
        t1.detach();
        t2.detach();
        t3.detach();
        // ...
    }

这里的“OK”表明对象能够在 `thread` 可以使用指向它的指针时一直处于作用域（“存活”）。
“bad”则表示 `thread` 可能在对象销毁之后使用指向它的指针。
`thread` 运行的并发性并不会影响这里的生存期或所有权问题；
这些 `thread` 可以仅仅被看成是从 `some_fct` 中调用的函数对象。

##### 注解

即便具有静态存储期的对象，在脱离的线程中的使用也会造成问题：
若是这个线程持续到程序终止，则它的运行可能与具有静态存储期的对象的销毁过程之间并发地发生，
而这样对这些对象的访问就可能发生竞争。

##### 注解

如果你[不 `detach()`](#rconc-detached_thread) 并[使用 `gsl::joining_tread`](#rconc-joining_thread) 的话，本条规则是多余的。
不过，将代码转化为遵循这些指导方针可能很困难，而对于第三方库来说更是不可能的。
这些情况下，这条规则对于生存期安全性和类型安全性来说就是必要的了。


一般来说是无法确定是否对某个 `thread` 执行了 `detach()` 的，但简单的常见情况则易于检测出来。
如果无法证明某个 `thread` 并没有 `detach()` 的话，我们只能假定它确实脱离了，且它的存活将超过其构造时所处于的作用域；
之后，可以实施（针对全局对象的）常规的生存期和所有权强制实施方案。

##### 强制实施

当试图将局部变量传递给可能 `detach()` 的线程时进行标记。

### <a id="rconc-joining_thread"></a>CP.25: 优先采用 `gsl::joining_thread` 而不是 `std::thread`

##### 理由

`joining_thread` 是一种在其作用域结尾处进行联结的线程。
脱离的线程很难进行监管。
确保脱离的线程（和潜在脱离的线程）中没有错误则更加困难。

##### 示例，不好

    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() const { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() 在独立线程中执行
        std::thread t2{F()};    // F()() 在独立线程中执行
    }  // 请找出问题

##### 示例

    void f() { std::cout << "Hello "; }

    struct F {
        void operator()() const { std::cout << "parallel world "; }
    };

    int main()
    {
        std::thread t1{f};      // f() 在独立线程中执行
        std::thread t2{F()};    // F()() 在独立线程中执行

        t1.join();
        t2.join();
    }  // 剩下一个糟糕的 BUG

##### 注解

使“不死线程”成为全局的，将其放入外围作用域中，或者放入自由存储中，而不要 `detach()` 它们。
[不要 `detach`](#rconc-detached_thread)。

##### 注解

因为老代码和第三方库也会使用 `std::thread`，本条规则可能很难引入。

##### 理由

标记 `std::thread` 的使用：

* 建议使用 `gsl::joining_thread` 或 C++20 的 `std:jthread`.
* 建议当其脱离时使其[“外放所有权”](#rconc-detached_thread)到某个外围作用域中。
* 如果不明确线程是联结还是脱离，则给出警告。

### <a id="rconc-detached_thread"></a>CP.26: 不要 `detach()` 线程

##### 理由

通常，需要存活超出线程创建的作用域的情况是来源于 `thread` 的任务所决定的，
但用 `detach` 来实现这点将造成更加难于对脱离的线程进行监控和通信。
特别是，要确保线程按预期完成或者按预期的时间存活变得更加困难（虽然不是不可能）。

##### 示例

    void heartbeat();

    void use()
    {
        std::thread t(heartbeat);             // 不联结；打算持续运行 heartbeat
        t.detach();
        // ...
    }

这是一种合理的线程用法，一般会使用 `detach()`。
但这里有些问题。
我们怎么监控脱离的线程以查看它是否存活呢？
心跳里边可能会出错，而在需要心跳的系统中，心跳丢失可能是非常严重的问题。
因此，我们需要与心跳线程进行通信
（例如，通过某个消息流，或者使用 `condition_variable` 的通知事件）。

一种替代的，而且通常更好的方案是，通过将其放入某个其创建点（或激活点）之外的作用域来控制其生存期。
例如：

    void heartbeat();

    gsl::joining_thread t(heartbeat);             // 打算持续运行 heartbeat

这个心跳，（除非出错或者硬件故障等情况）将在程序运行时一直运行。

有时候，我们需要将创建点和所有权点分离开：

    void heartbeat();

    unique_ptr<gsl::joining_thread> tick_tock {nullptr};

    void use()
    {
        // 打算在 tick_tock 存活期间持续运行 heartbeat
        tick_tock = make_unique(gsl::joining_thread, heartbeat);
        // ...
    }

##### 强制实施

标记 `detach()`。


### <a id="rconc-data-by-value"></a>CP.31: 少量数据在线程之间按值传递，而不是通过引用或指针传递

##### 理由

对少量数据进行复制和访问要比使用某种锁定机制进行共享更廉价。
复制天然会带来唯一所有权（简化代码），并消除数据竞争的可能性。

##### 注解

对“少量”进行精确的定义是不可能的。

##### 示例

    string modify1(string);
    void modify2(string&);

    void fct(string& s)
    {
        auto res = async(modify1, s);
        async(modify2, s);
    }

`modify1` 的调用涉及两个 `string` 值的复制；而 `modify2` 的调用则不会。
另一方面，`modify1` 的实现和我们为单线程代码所编写的完全一样，
而 `modify2` 的实现则需要某种形式的锁定以避免数据竞争。
如果字符串很短（比如 10 个字符），对 `modify1` 的调用将会出乎意料地快；
基本上所有的代价都在 `thread` 的切换上。如果字符串很长（比如 1,000,000 个字符），对其两次复制
可能并不是一个好主意。

注意这个论点和 `async` 并没有任何关系。它同等地适用于任何对采用消息传递
还是共享内存的考虑之上。

##### 强制实施

???


### <a id="rconc-shared"></a>CP.32: 用 `shared_ptr` 在无关的 `thread` 之间共享所有权

##### 理由

如果线程之间是无关的（就是说，互相不知道是否在相同作用域中，或者一个的生存期在另一个之内），
而它们需要共享需要删除的自由存储内存，`shared_ptr`（或者等价物）就是唯一的
可以保证正确删除的安全方式。

##### 示例

    ???

##### 注解

* 可以共享静态对象（比如全局对象），因为它并不像是需要某个线程来负责其删除那样被谁所拥有。
* 自由存储上不会被删除的对象可以进行共享。
* 由一个线程所拥有的对象可以安全地共享给另一个线程，只要第二个线程存活不会超过这个拥有者线程即可。

##### 强制实施

???


### <a id="rconc-switch"></a>CP.40: 最小化上下文切换

##### 理由

上下文切换是昂贵的。

##### 示例

    ???

##### 强制实施

???


### <a id="rconc-create"></a>CP.41: 最小化线程的创建和销毁

##### 理由

线程创建是昂贵的。

##### 示例

    void worker(Message m)
    {
        // 处理
    }

    void dispatcher(istream& is)
    {
        for (Message m; is >> m; )
            run_list.push_back(new thread(worker, m));
    }

这会为每个消息产生一个线程，而 `run_list` 则假定在它们完成后对这些任务进行销毁。

我们可以用一组预先创建的工作线程来处理这些消息：

    Sync_queue<Message> work;

    void dispatcher(istream& is)
    {
        for (Message m; is >> m; )
            work.put(m);
    }

    void worker()
    {
        for (Message m; m = work.get(); ) {
            // 处理
        }
    }

    void workers()  // 设立工作线程（这里是 4 个工作线程）
    {
        joining_thread w1 {worker};
        joining_thread w2 {worker};
        joining_thread w3 {worker};
        joining_thread w4 {worker};
    }

##### 注解

如果你的系统有一个好的线程池的话，就请使用它。
如果你的系统有一个好的消息队列的话，就请使用它。

##### 强制实施

???


### <a id="rconc-wait"></a>CP.42: 不要无条件地 `wait`

##### 理由

没有条件的 `wait` 可能会丢失唤醒，或者唤醒时只会发现无事可做。

##### 示例，不好

    std::condition_variable cv;
    std::mutex mx;

    void thread1()
    {
        while (true) {
            // 做一些工作 ...
            std::unique_lock<std::mutex> lock(mx);
            cv.notify_one();    // 唤醒另一个线程
        }
    }

    void thread2()
    {
        while (true) {
            std::unique_lock<std::mutex> lock(mx);
            cv.wait(lock);    // 可能会永远阻塞
            // 做一些工作 ...
        }
    }

这里，如果某个其他 `thread` 消费了 `thread1` 的通知的话，`thread2` 将会永远等待下去。

##### 示例

    template<typename T>
    class Sync_queue {
    public:
        void put(const T& val);
        void put(T&& val);
        void get(T& val);
    private:
        mutex mtx;
        condition_variable cond;    // 这用于控制访问
        list<T> q;
    };

    template<typename T>
    void Sync_queue<T>::put(const T& val)
    {
        lock_guard<mutex> lck(mtx);
        q.push_back(val);
        cond.notify_one();
    }

    template<typename T>
    void Sync_queue<T>::get(T& val)
    {
        unique_lock<mutex> lck(mtx);
        cond.wait(lck, [this] { return !q.empty(); });    // 防止假性唤醒
        val = q.front();
        q.pop_front();
    }

这样当执行 `get()` 的线程被唤醒时，如果队列为空（比如因为别的线程在之前已经 `get()` 过了），
它将立刻回到睡眠中继续等待。

##### 强制实施

对所有没有条件的 `wait` 进行标记。


### <a id="rconc-time"></a>CP.43: 最小化临界区的时间耗费

##### 理由

获取 `mutex` 时耗费的时间越短，其他 `thread` 不得不等待的机会就会越少，
而 `thread` 的挂起和恢复是昂贵的。

##### 示例

    void do_something() // 不好
    {
        unique_lock<mutex> lck(my_lock);
        do0();  // 预备：不需要锁定
        do1();  // 事务：需要锁定
        do2();  // 清理：不需要锁定
    }

这里我们持有的锁定比所需的要长：
我们不应该在必须锁定之前就获取锁定，而且应当在开始清理之前将其释放掉。
可以将其重写为：

    void do_something() // 不好
    {
        do0();  // 预备：不需要锁定
        my_lock.lock();
        do1();  // 事务：需要锁定
        my_lock.unlock();
        do2();  // 清理：不需要锁定
    }

但这样损害了安全性并且违反了[使用 RAII](#rconc-raii) 规则。
我们可以为临界区添加语句块：

    void do_something() // OK
    {
        do0();  // 预备：不需要锁定
        {
            unique_lock<mutex> lck(my_lock);
            do1();  // 事务：需要锁定
        }
        do2();  // 清理：不需要锁定
    }

##### 强制实施

一般来说是不可能的。
对“裸” `lock()` 和 `unlock()` 进行标记。


### <a id="rconc-name"></a>CP.44: 记得为 `lock_guard` 和 `unique_lock` 命名

##### 理由

无名的局部对象时临时对象，会立刻离开作用域。

##### 示例

    // 全局互斥体
    mutex m1;
    mutex m2;

    void f()
    {
        unique_lock<mutex>(m1); // (A)
        lock_guard<mutex> {m2}; // (B)
        // 关键区中的工作
    }

这个貌似足够有效，但其实并非如此。在 (A) 点，`m1` 是一个
默认构造的局部 `unique_lock`，它隐藏了全局的 `::m1`（且并未锁定它）。
在 (B) 点，构造了一个无名 `lock_guard` 临时对象并锁定了 `::m2`，
但它立即离开作用域并再次解锁了 `::m2`。
函数 `f()` 的余下部分中并没有锁定任何互斥体。

##### 强制实施

标记所有的无名 `lock_guard` 和 `unique_lock`。



### <a id="rconc-mutex"></a>CP.50: `mutex` 要和其所保护的数据一起定义，只要可能就使用 `synchronized_value<T>`

##### 理由

对于读者来说，数据应该且如何被保护应当是显而易见的。这可以减少锁定错误的互斥体，或者互斥体未能被锁定的机会。

使用 `synchronized_value<T>` 保证了数据都带有互斥体，并且当访问数据时锁定正确的互斥体。
参见向某个未来的 TS 或 C++ 标准的修订版添加 `synchronized_value` [WG21 提案](http://wg21.link/p0290)。

##### 示例

    struct Record {
        std::mutex m;   // 访问其他成员之前应当获取这个 mutex
        // ...
    };

    class MyClass {
        struct DataRecord {
           // ...
        };
        synchronized_value<DataRecord> data; // 用互斥体保护数据
    };

##### 强制实施

??? 可能吗？


## <a id="sscp-coro"></a>CP.coro: 协程

这一部分关注协程的使用。

协程规则概览：

* [CP.51: 不要使用作为协程的有俘获 lambda 表达式](#rcoro-capture)
* [CP.52: 不要在持有锁或其它同步原语时跨越挂起点](#rcoro-locks)
* [CP.53: 协程的形参不能按引用传递](#rcoro-reference-parameters)

### <a id="rcoro-capture"></a>CP.51: 不要使用作为协程的有俘获 lambda 表达式


##### 理由

对于普通 lambda 来说正确的使用模式，对于协程 lambda 来说是高危的。很明显的变量俘获模式将会造成在首个挂起点之后访问已释放的内存，即便是带引用计数的智能指针和可复制类型也是如此。

lambda 表达式会产生一个带有存储的闭包对象，它通常在运行栈上，并会在某个时刻离开作用域。当闭包对象离开作用域时，它所俘获的也会离开作用域。普通 lambda 的执行在这个时间点都已经完成了，因此这并不是问题。闭包 lambda 则可能会在闭包对象已经销毁之后从挂起中恢复执行，而在这时其所有俘获都将变为“释放后使用”的内存访问。

##### 示例，不好

    int value = get_value();
    std::shared_ptr<Foo> sharedFoo = get_foo();
    {
      const auto lambda = [value, sharedFoo]() -> std::future<void>
      {
        co_await something();
        // "sharedFoo" 和 "value" 已被销毁
        // “共享”指针没有带来任何好处
      };
      lambda();
    } // lambda 闭包对象此时已离开作用域

##### 示例，更好

    int value = get_value();
    std::shared_ptr<Foo> sharedFoo = get_foo();
    {
      const auto lambda = [](auto sharedFoo, auto value) -> std::future<void>  // 以按值传参代替作为俘获
      {
        co_await something();
        // sharedFoo 和 value 此时仍然有效
      };
      lambda(sharedFoo, value);
    } // lambda 闭包对象此时已离开作用域

##### 示例，最佳

使用函数作为协程。

    std::future<void> Class::do_something(int value, std::shared_ptr<Foo> sharedFoo)
    {
      co_await something();
      // sharedFoo 和 value 此时仍然有效
    }

    void SomeOtherFunction()
    {
      int value = get_value();
      std::shared_ptr<Foo> sharedFoo = get_foo();
      do_something(value, sharedFoo);
    }

##### 强制实施

标记作为协程且具有非空俘获列表的 lambda 表达式。


### <a id="rcoro-locks"></a>CP.52: 不要在持有锁或其它同步原语时跨越挂起点

##### 理由

这种模式会导致明显的死锁风险。某些种类的等待允许当前线程在异步操作完成前实施一些额外的工作。如果持有锁的线程实施了需要相同的所的工作，那它就会因为试图获取它已经持有的锁而发生死锁。

如果协程在某个与获得所的线程不同的另一个线程中完成，那就是未定义行为。即使协程中明确返回其原来的线程，仍然有可能在协程恢复之前抛出异常，并导致其锁定防护对象（lock guard）未能销毁。

##### 示例，不好

    std::mutex g_lock;

    std::future<void> Class::do_something()
    {
        std::lock_guard<std::mutex> guard(g_lock);
        co_await something(); // 危险：在持有锁时挂起协程
        co_await somethingElse();
    }

##### 示例，好

    std::mutex g_lock;

    std::future<void> Class::do_something()
    {
        {
            std::lock_guard<std::mutex> guard(g_lock);
            // 修改被锁保护的数据
        }
        co_await something(); // OK：锁已经在协程挂起前被释放
        co_await somethingElse();
    }


##### 注解

这种模式对于性能也不好。每当遇到比如 `co_await` 这样的挂起点时，都会停止当前函数的执行而去运行别的代码。而在协程恢复之前可能会经过很长时间。这个锁会在整个时间段中持有，并且无法被其他线程获得以进行别的工作。

##### 强制实施

标记所有未能在协程挂起前销毁的锁定防护。

### <a id="rcoro-reference-parameters"></a>CP.53: 协程的形参不能按引用传递

##### 理由

一旦协程到达其第一个如 `co_await` 这样的挂起点，其同步执行的部分就会返回。这个位置之后，所有按引用传递的形参都是悬挂引用。此后对它们的任何使用都是未定义行为，可能包括向已释放的内存进行写入。

##### 示例，不好

    std::future<int> Class::do_something(const std::shared_ptr<int>& input)
    {
        co_await something();

        // 危险：对 input 的引用可能不再有效，可能已经是已释放内存
        co_return *input + 1;
    }

##### 示例，好

    std::future<int> Class::do_something(std::shared_ptr<int> input)
    {
        co_await something();
        co_return *input + 1; // input 是一个副本，且到此处仍有效
    }

##### 注解

这个问题并不适用于仅在第一个挂起点之前访问的引用形参。此后对函数的改动中可能会添加或移除挂起点，而这可能会再次引入这一类的缺陷。一些种类的协程，在协程执行第一行代码之前就会有挂起点，这种情况中的引用形参总是不安全的。一直采用按值传递的方式更安全，因为所复制的形参存活于协程的栈帧中，并在整个协程中都可以安全访问。

##### 注解

输出参数也有这种危险。[F.20: 对于“输出（out）”值，采用返回值优先于输出参数](#rf-out) 不建议使用输出参数。协程应当完全避免输出参数。

##### 强制实施

标记协程的所有引用形参。

## <a id="sscp-par"></a>CP.par: 并行

这里的“并行”代表的是对许多数据项（或多或少）同时地（“并行进行”）实施某项任务。

并行规则概览：

* ???
* ???
* 适当的时候，优先采用标准库的并行算法
* 使用为并行设计的算法，而不是不必要地依赖于线性求值的算法



## <a id="sscp-mess"></a>CP.mess: 消息传递

标准库的设施是相当底层的，关注于使用 `thread`，`mutex`，`atomic` 等类型的贴近硬件的关键编程。
大多数人都不应该在这种层次上工作：它是易错的，而且开发很慢。
如果可能的话，应当使用高层次的设施：消息程序库，并行算法，以及向量化。
这一部分关注如何传递消息，以使程序员不必进行显式的同步。

消息传递规则概览：

* [CP.60: 使用 `future` 从并发任务返回值](#rconc-future)
* [CP.61: 使用 `async()` 来产生并发任务](#rconc-async)
* 消息队列
* 消息程序库

???? 是否应该有某个“使用 X 而不是 `std::async`”，其中 X 是某种更好说明的线程池？

??? 在未来的趋势下（甚至是现存的程序库），`std::async` 是否还是值得使用的并行设施？当有人想要对比如 `std::accumulate`（带上额外的累加前条件），或者归并排序进行并行化时，指导方针应该给出什么样的建议呢？


### <a id="rconc-future"></a>CP.60: 使用 `future` 从并发任务返回值

##### 理由

`future` 为异步任务保持了常规函数调用的返回语义。
它没有显式的锁定，而且正确的（值）返回和错误的（异常）返回都能被简单处理。

##### 示例

    ???

##### 注解

???

##### 强制实施

???

### <a id="rconc-async"></a>CP.61: 使用 `async()` 来产生并发任务

##### 理由

[R.12](#rr-immediate-alloc) 告诉我们要避免原始所有权指针，与此相似，
我们也要尽可能避免原始线程和原始承诺（promise）。应使用诸如 `std::async` 之类的工厂函数，
它将处理线程的产生和重用，而不会讲原始线程暴露给你自己的代码。

##### 示例

    int read_value(const std::string& filename)
    {
        std::ifstream in(filename);
        in.exceptions(std::ifstream::failbit);
        int value;
        in >> value;
        return value;
    }

    void async_example()
    {
        try {
            std::future<int> f1 = std::async(read_value, "v1.txt");
            std::future<int> f2 = std::async(read_value, "v2.txt");
            std::cout << f1.get() + f2.get() << '\n';
        } catch (std::ios_base::failure & fail) {
            // 此处处理异常
        }
    }

##### 注解

不幸的是，`async()` 并不完美。比如说，它并不使用线程池，
这意味着它可能会因为资源耗尽而失败，而不会将你的任务放入队列以便随后执行。
不过，即便你不能用 `std::async`，你也应当优先编写自己的返回
`future` 的工厂函数，而非使用原始承诺。

##### 示例（不好）

这个例子展示了两种方式，都使用了 `std::future` 却未能避免对原始
`std::thread` 的管理。

    void async_example()
    {
        std::promise<int> p1;
        std::future<int> f1 = p1.get_future();
        std::thread t1([p1 = std::move(p1)]() mutable {
            p1.set_value(read_value("v1.txt"));
        });
        t1.detach(); // 恶行

        std::packaged_task<int()> pt2(read_value, "v2.txt");
        std::future<int> f2 = pt2.get_future();
        std::thread(std::move(pt2)).detach();

        std::cout << f1.get() + f2.get() << '\n';
    }

##### 示例（好）

这个例子展示一种方法，可以让你在当 `std::async` 自身
在产品中不可接受的情形中，模仿 `std::async` 所设立的
一般模式的方法。

    void async_example(WorkQueue& wq)
    {
        std::future<int> f1 = wq.enqueue([]() {
            return read_value("v1.txt");
        });
        std::future<int> f2 = wq.enqueue([]() {
            return read_value("v2.txt");
        });
        std::cout << f1.get() + f2.get() << '\n';
    }

所有为执行 `read_value` 的代码而产生的线程都被隐藏到对
`WorkQueue::enqueue` 的调用之内。用户代码仅需处理 `future` 对象，
无需处理原始 `thread`，`promise` 或 `packaged_task` 对象。

##### 强制实施

???


## <a id="sscp-vec"></a>CP.vec: 向量化

向量化是一种在不引入显式同步时并发地执行多个任务的技术。
它会并行地把某个操作实施与某个数据结构（向量，数组，等等）的各个元素之上。
向量化引人关注的性质在于它通常不需要对程序进行非局部的改动。
不过，向量化只对简单的数据结构和特别针对它所构造的算法能够得到最好的工作效果。

向量化规则概览：

* ???
* ???

## <a id="sscp-free"></a>CP.free: 无锁编程

使用 `mutex` 和 `condition_variable` 进行同步相对来说很昂贵。
而且可能导致死锁。
为了性能并消除死锁的可能性，有时候我们不得不使用麻烦的底层“无锁”设施，
它们依赖于对内存短暂地获得互斥（“原子性”）访问。
无锁编程也被用于实现如 `thread` 和 `mutex` 这样的高层并发机制。

无锁编程规则概览：

* [CP.100: 除非绝对必要，请勿使用无锁编程](#rconc-lockfree)
* [CP.101: 不要信任你的硬件-编译器组合](#rconc-distrust)
* [CP.102: 仔细研究文献](#rconc-literature)
* 如何/何时使用原子
* 避免饥饿
* 使用无锁数据结构而不是手工构造的专门的无锁访问
* [CP.110: 不要为初始化编写你自己的双检查锁定](#rconc-double)
* [CP.111: 当确实需要双检查锁定时应当采用惯用的模式](#rconc-double-pattern)
* 如何/何时进行比较并交换（CAS）


### <a id="rconc-lockfree"></a>CP.100: 除非绝对必要，请勿使用无锁编程

##### 理由

无锁编程容易出错，要求专家级的语言特性、机器架构和数据结构知识。

##### 示例，不好

    extern atomic<Link*> head;        // 共享的链表表头

    Link* nh = new Link(data, nullptr);    // 为进行插入制作一个连接
    Link* h = head.load();                 // 读取链表中的共享表头

    do {
        if (h->data <= data) break;        // 这样的话，就插入到别处
        nh->next = h;                      // 下一个元素是之前的表头
    } while (!head.compare_exchange_weak(h, nh));    // 将 nh 写入 head 或 h

请找出这里的 BUG。
通过测试找到它是非常困难的。
请阅读有关 ABA 问题的材料。

##### 例外

[原子变量](#???)可以简单并安全地使用，只要你所用的是顺序一致性内存模型（`memory_order_seq_cst`），而这是默认情况。

##### 注解

高层的并发机制，诸如 `thread` 和 `mutex`，是使用无锁编程来实现的。

**替代方案**: 使用由他人实现并作为程序库一部分的无锁数据结构。


### <a id="rconc-distrust"></a>CP.101: 不要信任你的硬件-编译器组合

##### 理由

无锁编程所使用的底层硬件接口，是属于最难正确实现的，
而且属于最可能会发生最微妙的兼容性问题的领域。
如果你为了性能而进行无锁编程的话，你应当进行回归检查。

##### 注解

指令重排（静态和动态的）会让我们很难有效在这个层次上进行思考（尤其当你使用宽松的内存模型的时候）。
经验，（半）形式化的模型以及模型检查可以提供帮助。
测试——通常需要极端程度——是基础。
“不要飞得太靠近太阳。”

##### 强制实施

准备强有力的规则，使得当硬件，操作系统，编译器，和程序库发生任何改变都能重复测试以进行覆盖。


### <a id="rconc-literature"></a>CP.102: 仔细研究文献

##### 理由

除了原子和少数其他的标准模式之外，无锁编程真的是只有专家才懂的议题。
在发布无锁代码给其他人使用之前，应当先成为一名专家。

##### 参考文献

* Anthony Williams: C++ concurrency in action. Manning Publications.
* Boehm, Adve, You Don't Know Jack About Shared Variables or Memory Models , Communications of the ACM, Feb 2012.
* Boehm, "Threads Basics", HPL TR 2009-259.
* Adve, Boehm, "Memory Models: A Case for Rethinking Parallel Languages and Hardware", Communications of the ACM, August 2010.
* Boehm, Adve, "Foundations of the C++ Concurrency Memory Model", PLDI 08.
* Mark Batty, Scott Owens, Susmit Sarkar, Peter Sewell, and Tjark Weber, "Mathematizing C++ Concurrency", POPL 2011.
* Damian Dechev, Peter Pirkelbauer, and Bjarne Stroustrup: Understanding and Effectively Preventing the ABA Problem in Descriptor-based Lock-free Designs. 13th IEEE Computer Society ISORC 2010 Symposium. May 2010.
* Damian Dechev and Bjarne Stroustrup: Scalable Non-blocking Concurrent Objects for Mission Critical Code. ACM OOPSLA'09. October 2009
* Damian Dechev, Peter Pirkelbauer, Nicolas Rouquette, and Bjarne Stroustrup: Semantically Enhanced Containers for Concurrent Real-Time Systems. Proc. 16th Annual IEEE International Conference and Workshop on the Engineering of Computer Based Systems (IEEE ECBS). April 2009.
* Maurice Herlihy, Nir Shavit, Victor Luchangco, Michael Spear, "The Art of Multiprocessor Programming", 2nd ed. September 2020

### <a id="rconc-double"></a>CP.110: 不要为初始化编写你自己的双检查锁定

##### 理由

从 C++11 开始，静态局部变量是以线程安全的方式初始化的。当和 RAII 模式结合时，静态局部变量可以取代为初始化自己编写双检查锁定的需求。`std::call_once` 也可以达成相同的目的。请使用 C++11 的静态局部变量或者 `std::call_once` 来代替自己为初始化编写的双检查锁定。

##### 示例

使用 `std::call_once` 的例子。

    void f()
    {
        static std::once_flag my_once_flag;
        std::call_once(my_once_flag, []()
        {
            // 这个只做一次
        });
        // ...
    }

使用 C++11 的线程安全静态局部变量的例子。

    void f()
    {
        // 假定编译器遵循 C++11
        static My_class my_object; // 构造函数仅调用一次
        // ...
    }

    class My_class
    {
    public:
        My_class()
        {
            // 这个只做一次
        }
    };

##### 强制实施

??? 是否可能检测出这种惯用法？


### <a id="rconc-double-pattern"></a>CP.111: 当确实需要双检查锁定时应当采用惯用的模式

##### 理由

双检查锁定是很容易被搞乱的。如果确实需要编写自己的双检查锁定，而不顾规则 [CP.110: 不要为初始化编写你自己的双检查锁定](#rconc-double)和规则 [CP.100: 除非绝对必要，请勿使用无锁编程](#rconc-lockfree)，那么应当采用惯用的模式。

使用双检查锁定模式而不违反[CP.110: 不要为初始化编写你自己的双检查锁定](#rconc-double)的情形，出现于当某个非线程安全的动作既困难也罕见，并且存在某个快速且线程安全的测试可以用于保证该动作并不需要实施的情况，但反过来的情况则无法保证。

##### 示例，不好

使用 `volatile` 并不能使得第一个检查线程安全，另见[CP.200: `volatile` 仅用于和非 C++ 内存进行通信](#rconc-volatile2)

    mutex action_mutex;
    volatile bool action_needed;

    if (action_needed) {
        std::lock_guard<std::mutex> lock(action_mutex);
        if (action_needed) {
            take_action();
            action_needed = false;
        }
    }

##### 示例，好

    mutex action_mutex;
    atomic<bool> action_needed;

    if (action_needed) {
        std::lock_guard<std::mutex> lock(action_mutex);
        if (action_needed) {
            take_action();
            action_needed = false;
        }
    }

这对于正确调校的内存顺序可能会带来好处，其中的获取加载要比顺序一致性加载更加高效

    mutex action_mutex;
    atomic<bool> action_needed;

    if (action_needed.load(memory_order_acquire)) {
        lock_guard<std::mutex> lock(action_mutex);
        if (action_needed.load(memory_order_relaxed)) {
            take_action();
            action_needed.store(false, memory_order_release);
        }
    }

##### 强制实施

??? 是否可能检测出这种惯用法？


## <a id="sscp-etc"></a>CP.etc: 其他并发规则

这些规则不适于简单的分类：

* [CP.200: `volatile` 仅用于和非 C++ 内存进行通信](#rconc-volatile2)
* [CP.201: ??? 信号](#rconc-signal)

### <a id="rconc-volatile2"></a>CP.200: `volatile` 仅用于和非 C++ 内存进行通信

##### 理由

`volatile` 用于涉指那些与“非 C++”代码之间共享的对象，或者不遵循 C++ 内存模型的硬件。

##### 示例

    const volatile long clock;

这说明了一个被某个时钟电路不断更新的寄存器。
`clock` 为 `volatile` 是因为其值将会在没有使用它的 C++ 程序的任何动作下被改变。
例如，两次读取 `clock` 经常会产生两个不同的值，因此优化器最好不要将下面代码中的第二个读取操作优化掉：

    long t1 = clock;
    // ... 这里没有对 clock 的使用 ...
    long t2 = clock;

`clock` 为 `const` 是因为程序不应当试图写入 `clock`。

##### 注解

除非你是在编写直接操作硬件的最底层代码，否则应当把 `volatile` 当做是某种深奥的功能特性并最好避免使用。

##### 示例

通常 C++ 代码接受的 `volatile` 内存是由别处所拥有的（硬件或其他语言）：

    int volatile* vi = get_hardware_memory_location();
        // 注意：我们获得了指向别人的内存的指针
        // volatile 说明“请特别尊重地对待”

有时候 C++ 代码会分配 `volatile` 内存，并通过故意地暴露一个指针来将其共享给“别人”（硬件或其他语言）：

    static volatile long vl;
    please_use_this(&vl);   // 暴露对这个的一个引用给“别人”（不是 C++）

##### 示例，不好

`volatile` 局部变量几乎都是错误的——既然是短暂的，它们如何才能共享给其他语言或硬件呢？
因为相同的理由，这几乎同样强有力地适用于成员变量。

    void f()
    {
        volatile int i = 0; // 不好，volatile 局部变量
        // etc.
    }

    class My_type {
        volatile int i = 0; // 可以的，volatile 成员变量
        // etc.
    };

##### 注解

于其他一些语言不通，C++ 中的 `volatile` [和同步没有任何关系](#rconc-volatile)。

##### 强制实施

* 对 `volatile T` 的局部成员变量进行标记；几乎肯定你应当用 `atomic<T>` 进行代替。
* ???

### <a id="rconc-signal"></a>CP.201: ??? 信号

???UNIX 信号处理??? 也许值得提及异步信号安全有多么微弱，以及如何同信号处理器进行通信（也许最好应当“完全避免”）
