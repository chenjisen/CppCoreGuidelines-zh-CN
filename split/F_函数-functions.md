# <a id="s-functions"></a>F: 函数

函数指定了一个活动或者一次计算，以将系统从一种一致的状态转移到另一种一致的状态。函数是程序的基础构造块。

应当使函数的名字有意义，说明对其参数的必要条件，并清晰地规定参数和其结果之间的关系。函数的实现本身并不是规格说明。请尝试同时对函数应当做什么和函数应当怎样做来进行思考。
函数在大多数接口中都是最关键的部分，请参考接口的规则。

函数规则概览：

函数定义式的规则：

* [F.1: 把有意义的操作“打包”成为精心命名的函数](#rf-package)
* [F.2: 一个函数应当实施单一一项逻辑操作](#rf-logical)
* [F.3: 保持函数短小简洁](#rf-single)
* [F.4: 如果函数可能必须在编译期进行求值，就将其声明为 `constexpr`](#rf-constexpr)
* [F.5: 如果函数非常小，并且是时间敏感的，就将其声明为 `inline`](#rf-inline)
* [F.6: 如果函数必然不会抛出异常，就将其声明为 `noexcept`](#rf-noexcept)
* [F.7: 对于常规用法，应当接受 `T*` 或 `T&` 参数而不是智能指针](#rf-smart)
* [F.8: 优先采用纯函数](#rf-pure)
* [F.9: 未使用的形参应当没有名字](#rf-unused)
* [F.10: 若操作可被重用，则应为其命名](#rf-name)
* [F.11: 当需要仅在一处使用的简单函数对象时使用无名 lambda]([#Rf-lambda)

参数传递表达式的规则：

* [F.15: 优先采用简单的和传统的信息传递方式](#rf-conventional)
* [F.16: 对于“输入（in）”参数，把复制操作廉价的类型按值进行传递，把其他类型按 `const` 引用进行传递](#rf-in)
* [F.17: 对于“输入/输出（in-out）”参数，按非 `const` 引用进行传递](#rf-inout)
* [F.18: 对于“将被移动（will-move-from）”参数，按 `X&&` 进行传递并对参数 `std::move`](#rf-consume)
* [F.19: 对于“转发（forward）”参数，按 `TP&&` 进行传递并只对参数 `std::forward`](#rf-forward)
* [F.20: 对于“输出（out）”值，采用返回值优先于输出参数](#rf-out)
* [F.21: 要返回多个“输出”值，优先返回结构体或元组（tuple）](#rf-out-multi)
* [F.60: 当“没有参数”是有效的选项时，采用 `T*` 优先于 `T&`](#rf-ptr-ref)

参数传递语义的规则：

* [F.22: 用 `T*` 或 `owner<T*>` 来代表单个对象](#rf-ptr)
* [F.23: 用 `not_null<T>` 来表明“空值（null）”不是有效的值](#rf-nullptr)
* [F.24: 用 `span<T>` 或者 `span_p<T>` 来代表一个半开序列](#rf-range)
* [F.25: 用 `zstring` 或者 `not_null<zstring>` 来代表 C 风格的字符串](#rf-zstring)
* [F.26: 当需要指针时，用 `unique_ptr<T>` 来传递所有权](#rf-unique_ptr)
* [F.27: 用 `shared_ptr<T>` 来共享所有权](#rf-shared_ptr)

<a id="rf-value-return"></a>值返回语义的规则：

* [F.42: 返回 `T*` 来（仅仅）给出一个位置](#rf-return-ptr)
* [F.43: 不要（直接或间接）返回指向局部对象的指针或引用](#rf-dangle)
* [F.44: 当不想进行复制，且不需要“没有对象被返回”时，返回 `T&`](#rf-return-ref)
* [F.45: 不要返回 `T&&`](#rf-return-ref-ref)
* [F.46: `int` 是 `main()` 的返回类型](#rf-main)
* [F.47: 赋值运算符返回 `T&`](#rf-assignment-op)
* [F.48: 不要用 `std::move(local)`](#rf-return-move-local)
* [F.49: 不要返回 `const T`](#rf-return-const)

其他函数规则：

* [F.50: 当函数不适用时（不能俘获局部变量，或者不能编写局部函数），就使用 Lambda](#rf-capture-vs-overload)
* [F.51: 如果需要作出选择，采用默认实参应当优先于进行重载](#rf-default-args)
* [F.52: 对于局部使用的（也包括传递给算法的）lambda，优先采用按引用俘获](#rf-reference-capture)
* [F.53: 对于非局部使用的（包括被返回的，在堆上存储的，或者传递给别的线程的）lambda，避免采用按引用俘获](#rf-value-capture)
* [F.54: 当俘获了 `this` 时，显式俘获所有的变量（不使用默认俘获）](#rf-this-capture)
* [F.55: 不要使用 `va_arg` 参数](#f-varargs)
* [F.56: 避免不必要的条件嵌套](#f-nesting)

函数和 Lambda 表达式以及函数对象有很强的相似性。

**参见**：[C.lambdas: 函数对象和 lambda](#ss-lambdas)

## <a id="ss-fct-def"></a>F.def: 函数的定义式

函数的定义式就是一并指定了函数的实现（函数体）的函数声明式。

### <a id="rf-package"></a>F.1: 把有意义的操作“打包”成为精心命名的函数

##### 理由

把公共的代码分解出去，将使代码更易于阅读，更可能被重用，并能够对源于复杂代码的错误有所限制。
如果某部分是一个明确指定的活动，就将之从其包围代码中分离出来，并为其进行命名。

##### 示例，请勿这样做

    void read_and_print(istream& is)    // 读取并打印一个 int
    {
        int x;
        if (is >> x)
            cout << "the int is " << x << '\n';
        else
            cerr << "no int on input\n";
    }

`read_and_print` 的几乎每件事都有问题。
它进行了读取，它（向一个固定 `ostream`）进行了写入，它（向一个固定的 `ostream`）写入了错误消息，它只能处理 `int`。
这里没有可以重用的东西，逻辑上分开的操作被搅拌到了一起，而局部变量在其逻辑上使用完毕之后仍处于作用域中。
作为一个小例子的话还好，但如果输入操作、输出操作和错误处理更加复杂的话，
这个纠缠混乱的代码就会变得难于理解了。

##### 注解

如果你编写的一个有些价值的 lambda 可能潜在地被用于多处，那就为它进行命名并将其赋值给一个（通常非局部的）变量。

##### 示例

    sort(a, b, [](T x, T y) { return x.rank() < y.rank() && x.value() < y.value(); });

对 lambda 进行命名，将会把这个表达式进行逻辑上的分解，还会为 lambda 的含义给出有力的提示。

    auto lessT = [](T x, T y) { return x.rank() < y.rank() && x.value() < y.value(); };

    sort(a, b, lessT);

对于性能和可维护性来说，最简短的代码并不总是最好的选择。

##### 例外

循环体，包括用作循环体的 lambda，很少需要进行命名。
然而，大型的循环体（比如好多行或者好多页）也是个问题。
规则“[保持函数短小简洁](#rf-single)”暗含有“保持循环体短小”。
与此相似，用作回调参数的 lambda 有时候也是有意义的，虽然它们不大可能被重用。

##### 强制实施

* 参见“[保持函数短小简洁](#rf-single)”
* 把不同地方所用的同样和非常相似的 lambda 标记出来。

### <a id="rf-logical"></a>F.2: 一个函数应当实施单一一项逻辑操作

##### 理由

仅实施单一操作的函数易于理解，测试和重用。

##### 示例

考虑：

    void read_and_print()    // 不好
    {
        int x;
        cin >> x;
        // 检查错误
        cout << x << "\n";
    }

这是一整块被绑定到一个特定的输入的代码，而且无法为其找到另一种（不同的）用途。作为代替，我们把函数分解为合适的逻辑部分并进行参数化：

    int read(istream& is)    // 有改进
    {
        int x;
        is >> x;
        // 检查错误
        return x;
    }

    void print(ostream& os, int x)
    {
        os << x << "\n";
    }

这样的话，就可以在需要时进行组合：

    void read_and_print()
    {
        auto x = read(cin);
        print(cout, x);
    }

如果有需要，我们还可以进一步把 `read()` 和 `print()` 针对数据类型，I/O 机制，以及对错误的反应等等方面进行模板化。例如：

    auto read = [](auto& input, auto& value)    // 有改善
    {
        input >> value;
        // 检查错误
    };

    void print(auto& output, const auto& value)
    {
        output << value << "\n";
    }

##### 强制实施

* 把具有多个“输出”参数的函数当作有问题的。使用返回值来代替，包括以 `tuple` 用作多个返回值。
* 把无法装入编辑器的一屏之内的“大型”函数当作有问题的。考虑把这种函数分解为较小的恰当命名的子操作。
* 把有七个或更多参数的函数当作有问题的。

### <a id="rf-single"></a>F.3: 保持函数短小简洁

##### 理由

大型函数难于阅读，更有可能包含复杂的代码，而且更有可能含有其作用域超过最低限度的变量。
带有复杂的控制结构的函数更有可能变长，也更有可能隐藏逻辑错误于其中。

##### 示例

考虑：

    double simple_func(double val, int flag1, int flag2)
        // simple_func: 接受一个值并计算所需的 ASIC 值，
        // 依赖于两个模式标记。
    {
        double intermediate;
        if (flag1 > 0) {
            intermediate = func1(val);
            if (flag2 % 2)
                 intermediate = sqrt(intermediate);
        }
        else if (flag1 == -1) {
            intermediate = func1(-val);
            if (flag2 % 2)
                 intermediate = sqrt(-intermediate);
            flag1 = -flag1;
        }
        if (abs(flag2) > 10) {
            intermediate = func2(intermediate);
        }
        switch (flag2 / 10) {
        case 1: if (flag1 == -1) return finalize(intermediate, 1.171);
                break;
        case 2: return finalize(intermediate, 13.1);
        default: break;
        }
        return finalize(intermediate, 0.);
    }

这个函数过于复杂了。
要如何判断是否所有的可能性都被正确处理了呢？
当然，它也同样违反了别的规则。

我们可以进行重构：

    double func1_muon(double val, int flag)
    {
        // ???
    }

    double func1_tau(double val, int flag1, int flag2)
    {
        // ???
    }

    double simple_func(double val, int flag1, int flag2)
        // simple_func: 接受一个值并计算所需的 ASIC 值，
        // 依赖于两个模式标记。
    {
        if (flag1 > 0)
            return func1_muon(val, flag2);
        if (flag1 == -1)
            // 由 func1_tau 来处理: flag1 = -flag1;
            return func1_tau(-val, flag1, flag2);
        return 0.;
    }

##### 注解

“无法放入一屏显示”通常是对“太长了”的一种不错的实际定义方式。
一行到五行大小的函数应当被当作是常态。

##### 注解

把大型函数分解成较小的紧致的有名字的函数。
小型的简单函数在函数调用的代价比较明显时很容易被内联。

##### 强制实施

* 标记无法“放入一屏”的函数。
  一屏有多大？可以试试 60 行，每行 140 个字符；这大致上就是书本页面能够适于阅读的最大值了。
* 标记过于复杂的函数。多复杂算是过于复杂呢？
  应当用圈复杂度来度量。可以试试“多于 10 个逻辑路径”。一个简单的开关算作一条路径。

### <a id="rf-constexpr"></a>F.4: 如果函数可能必须在编译期进行求值，就将其声明为 `constexpr`

##### 理由

需要用 `constexpr` 来告诉编译器允许对其进行编译期求值。

##### 示例

（不）著名的阶乘例子：

    constexpr int fac(int n)
    {
        constexpr int max_exp = 17;      // constexpr 使得可以在 Expects 中使用 max_exp
        Expects(0 <= n && n < max_exp);  // 防止犯糊涂和发生溢出
        int x = 1;
        for (int i = 2; i <= n; ++i) x *= i;
        return x;
    }

这个是 C++14。
对于 C++11，请使用递归形式的 `fac()`。

##### 注解

`constexpr` 并不会保证发生编译期求值；
它只能保证函数可以在当程序员需要或者编译器为优化而决定时，对常量表达式实参进行编译期求值。

    constexpr int min(int x, int y) { return x < y ? x : y; }

    void test(int v)
    {
        int m1 = min(-1, 2);            // 可能进行编译期求值
        constexpr int m2 = min(-1, 2);  // 编译期求值
        int m3 = min(-1, v);            // 运行期求值
        constexpr int m4 = min(-1, v);  // 错误: 无法在编译期求值
    }

##### 注解

不要试图让所有函数都变成 `constexpr`。
大多数计算都最好在运行时进行。

##### 注解

任何可能最终将依赖于高层次的运行时配置或者
业务逻辑的API，都不应当是 `constexpr` 的。这种定制化是无法
由编译期来求值的，并且依赖于这种 API 的任何 `constexpr` 函数
也都应当进行重构，或者抛弃掉 `constexpr`。

##### 强制实施

不可能也不必要。
当在要求常量的地方调用了非 `constexpr` 函数时，编译器会报告错误。

### <a id="rf-inline"></a>F.5: 如果函数非常小，并且是时间敏感的，就将其声明为 `inline`

##### 理由

有些优化器可以不依赖于程序员的提示就能很好地进行内联，但请不要依赖这点。
请测量！至少超过 40 年，我们一直在允诺编译器可以不依赖于人类的提示而做到比人类更好地内联。
可是我们还在等。
（显式地，或于类定义体中编写成员函数而隐式地）将其指定为内联能够促进编译器工作得更好。

##### 示例

    inline string cat(const string& s, const string& s2) { return s + s2; }

##### 例外

不要把 `inline` 函数加入需要变得稳定的接口中，除非你十分确定它不会再发生变化。
内联函数是 ABI的一部分。

##### 注解

`constexpr` 蕴含 `inline`。

##### 注解

在类之中所定义的成员函数默认是 `inline` 的。

##### 例外

函数模板（包括类模板的成员函数 `A<T>::function()` 和成员函数模板 `A::function<T>()`）一般都定义于头文件中，因此是内联的。

##### 强制实施

对超过三条语句，并且本可以声明为非内联的 `inline` 函数（比如类成员函数）标记为 `inline`。

### <a id="rf-noexcept"></a>F.6: 如果函数必然不会抛出异常，就将其声明为 `noexcept`

##### 理由

如果不打算抛出异常的话，程序就会认为无法处理这种错误，并且应当尽早终止。把函数声明为 `noexcept` 对优化器有好处，因为其减少了可能的执行路径的数量。它也能使发生故障之后的退出动作有所加速。

##### 示例

给完全以 C 或者其他任何没有异常的语言编写的每个函数都标上 `noexcept`。
C++ 标准库隐含地对 C 标准库中的所有函数做了这件事。

##### 注解

`constexpr` 函数在运行时执行时可能抛出异常，因此可能需要对其中的一些使用有条件 `noexcept`。

##### 示例

对能够抛出异常的函数也可以使用 `noexcept`：

    vector<string> collect(istream& is) noexcept
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }

如果 `collect()` 耗光了内存，程序就会崩溃。
除非这个程序特别精心编写成不会耗尽内存，否则这也许正是正确的方式；
`terminate()` 能够产生合适的错误日志信息（但当内存耗尽时是很难做出任何巧妙的事情的）。

##### 注解

当你想决定是否要给函数标上 `noexcept` 时，一定要特别
注意你的代码的执行环境，尤其是与抛出异常和内存分配
相关的情形。打算成为完全通用的代码（比如像
标准库和其他类似的工具代码），应当支持那些
可以有意义地处理 `bad_alloc` 异常的执行环境。
不过，大多数程序和执行环境都不能有意义地
处理内存分配失败，而中止程序则是在这些情况中
应对分类失败的最干净和最简单的方式。如果已知
应用程序代码无法应对分配失败的话，对于即使
确实会进行分配的函数，添加 `noexcept` 也是适当的。

换一种方式来说：在大多数程序中，大多数函数都会抛出异常（比如说，
它们可能使用 `new`，调用会抛出异常的函数，或者使用通过抛出异常
来报告失败的库函数），因此请勿随意到处散布 `noexcept` 而不
考虑清楚是否有异常是可以被处理的。

`noexcept` 对于常用的，底层的函数是最有用处的（并且几乎
显然是正确的）。

##### 注解

析构函数，`swap` 函数，移动操作，以及默认构造函数不应当抛出异常。
另请参见 [C.44](#rc-default00)。

##### 强制实施

* 标记不是 `noexcept`，而又不能抛出异常的函数。
* 标记抛出异常的 `swap`，`move`，析构函数，以及默认构造函数。

### <a id="rf-smart"></a>F.7: 对于常规用法，应当接受 `T*` 或 `T&` 参数而不是智能指针

##### 理由

智能指针的传递会转移或者共享所有权，因此应当仅在有意要实现所有权语义时才能使用。
不操作生存期的函数应当接受原始指针或引用。

使用按智能指针传递方式把函数限制为只能服务于使用智能指针的调用方。
需要一个 `widget` 的函数应当能够接受任何 `widget` 对象，而不只是由某种特定种类的智能指针管理其生存期的那些。

智能指针的传递（比如 `std::shared_ptr`）暗含了一些运行时成本。

##### 示例

    // 接受任何的 int*
    void f(int*);

    // 只能接受你想转移所有权的 int
    void g(unique_ptr<int>);

    // 只能接受你想共享所有权的 int
    void g(shared_ptr<int>);

    // 不会改变所有权，但要求调用方对其具有特定的所有权。
    void h(const unique_ptr<int>&);

    // 接受任何的 int
    void h(int&);

##### 示例，不好

    // 被调用方
    void f(shared_ptr<widget>& w)
    {
        // ...
        use(*w); // w 的唯一使用点 -- 其生存期是完全未被涉及到的
        // ...
    };

    // 调用方
    shared_ptr<widget> my_widget = /* ... */;
    f(my_widget);

    widget stack_widget;
    f(stack_widget); // 错误

##### 示例，好

    // 被调用方
    void f(widget& w)
    {
        // ...
        use(w);
        // ...
    };

    // 调用方
    shared_ptr<widget> my_widget = /* ... */;
    f(*my_widget);

    widget stack_widget;
    f(stack_widget); // ok -- 这样就有效了

##### 注解

我们可以静态地找出悬挂指针的许多常见情况（参见[生存期安全性剖面配置](#ss-lifetime)）。函数实参天然存活于函数调用的生存期，因而具有更少的生存期问题。

##### 强制实施

* 【简单】若函数接受可复制的智能指针类型（即重载了 `operator->` 或 `operator*`），但该函数仅调用了：`operator*`、`operator->` 或 `get()`，则给出警告。
  建议代之以 `T*` 或 `T&`。
* 对于智能指针类型（重载了 `operator->` 或 `operator*` 的类型）的参数，若它是可复制/可移动的，但从没有从函数体中被复制/移动出来，且从未对其进行修改，且未将其传递给会修改它其他函数，对之进行标记。这意味着并未使用其所有权语义。
  建议代之以 `T*` 或 `T&`。

**参见**

* [当“无实参”是有效情形时，优先采用 `T*` 而不是 `T&`](#rf-ptr-ref)
* [智能指针规则概述](#rr-summary-smartptrs)

### <a id="rf-pure"></a>F.8: 优先采用纯函数

##### 理由

纯函数更容易进行推导，有时候也更易于优化（甚至并行化），有时候还可以进行存储。

##### 示例

    template<class T>
    auto square(T t) { return t * t; }

##### 强制实施

不可能进行强制实施。

### <a id="rf-unused"></a>F.9: 未使用的形参应当没有名字

##### 理由

可读性。
抑制未使用形参的警告消息。

##### 示例

    widget* find(const set<widget>& s, const widget& w, Hint);   // 这里曾经使用过一个提示

##### 注解

为解决这个问题，在 1980 年代早期就引入了允许形参无名的规则。

如果形参是根据条件不被使用的，可以用 `[[maybe_unused]]` 特性来声明它们。
例如：

    template <typename Value>
    Value* find(const set<Value>& s, const Value& v, [[maybe_unused]] Hint h)
    {
        if constexpr (sizeof(Value) > CacheSize)
        {
            // 仅当 Value 具有特定大小时才使用提示参数
        }
    }

##### 强制实施

对有名字的未使用形参进行标记。

### <a id="rf-name"></a>F.10: 若操作可被重用，则应为其命名

##### 理由

文档，可读性，重用机会。

##### 示例

    struct Rec {
        string name;
        string addr;
        int id;         // 唯一标识符
    };

    bool same(const Rec& a, const Rec& b)
    {
        return a.id == b.id;
    }

    vector<Rec*> find_id(const string& name);    // 寻找“name”的所有记录

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) {
            if (r.name.size() != n.size()) return false; // 要比较的名字在 n 里
            for (int i = 0; i < r.name.size(); ++i)
                if (tolower(r.name[i]) != tolower(n[i])) return false;
            return true;
        }
    );

这里蕴含着一个有用的函数（大小写不敏感的字符串比较），lambda 的参数变大时总会这样。

    bool compare_insensitive(const string& a, const string& b)
    {
        if (a.size() != b.size()) return false;
        for (int i = 0; i < a.size(); ++i) if (tolower(a[i]) != tolower(b[i])) return false;
        return true;
    }

    auto x = find_if(vr.begin(), vr.end(),
        [&](Rec& r) { return compare_insensitive(r.name, n); }
    );

或者可以这样（如果你倾向于避免隐含绑定到 `n` 的名字）：

    auto cmp_to_n = [&n](const string& a) { return compare_insensitive(a, n); };

    auto x = find_if(vr.begin(), vr.end(),
        [](const Rec& r) { return cmp_to_n(r.name); }
    );

##### 注解

函数、lambda 或运算符均如此。

##### 例外

* 逻辑上仅在局部使用的 lambda，比如作为 `for_each` 或类似控制流算法的实参。
* 作为[初始化](#???)的 lambda。

##### 强制实施

* 【困难】 标记相似的 lambda
* ???

### <a id="rf-lambda"></a>F.11: 当需要仅在一处使用的简单函数对象时使用无名 lambda

##### 理由

使代码精简，提供比其他方式更好的局部性。

##### 示例

    auto earlyUsersEnd = std::remove_if(users.begin(), users.end(),
                                        [](const User &a) { return a.id > 100; });


##### 例外

为 lambda 命名有助于明晰代码，即便它仅用一次。

##### 强制实施

* 寻找相同或几乎相同的 lambda（以将它们替换为具名的函数或者具名的 lambda）。

## <a id="ss-call"></a>F.call: 参数传递

存在各种不同的向函数传递参数和返回值的方式。

### <a id="rf-conventional"></a>F.15: 优先采用简单的和传统的信息传递方式

##### 理由

使用“与众不同和精巧”的技巧会带来意外，其他程序员的理解减慢，并促进 BUG 的发生。
如果你确实想要比常规技巧更好的优化，请进行测量以确保它真的有所提升，并为其写下文档/注释，因为这种提升可能无法移植。

下面的表格总结了以下 F.16-21 的各个指导方针中的建议。

一般性参数传递：

![一般性参数传递表](./images/param-passing-normal.png "一般性参数传递")

高级参数传递：

![高级参数传递表](./images/param-passing-advanced.png "高级参数传递")

只有在进行论证必要之后再使用高级技巧，并将其必要性注明在代码注释中。

对于字符序列的传递，参见 [字符串](#ss-string)。

##### 例外

使用 `shared_ptr` 等类型来表达共享所有权时，不应遵循指导方针 F.16-21，
而应遵循 [R.34](#rr-sharedptrparam-owner)，[R.35](#rr-sharedptrparam)，以及 [R.36](#rr-sharedptrparam-const)。

### <a id="rf-in"></a>F.16: 对于“输入（in）”参数，把复制操作廉价的类型按值进行传递，把其他类型按 `const` 引用进行传递

##### 理由

既能让调用者了解函数不会修改其参数，也使得参数能够以右值初始化。

何谓“复制操作廉价”依赖于机器的架构，不过只有两三个机器字（Word）的类型（double，指针，引用等）一般最好按值传递。
当可以廉价复制时，没什么比得过进行复制的简单性和安全性，而且对于小型对象（最多两三个机器字）来说，也比按引用传递更快，因为它不需要在函数中进行一次额外的间接访问。

##### 示例

    void f1(const string& s);  // OK: 按 const 引用传递; 总是廉价的

    void f2(string s);         // bad: 可能是昂贵的

    void f3(int x);            // OK: 无可比拟

    void f4(const int& x);     // bad: f4() 中的访问带来开销

（仅）对于高级的运用，如果你确实需要为“只当作输入”的参数的按右值传递进行优化的话：

* 如果函数需要无条件地从参数进行移动，那就按 `&&` 来接受参数。参见 [F.18](#rf-consume)。
* 如果函数需要保留参数的一个副本，那就在按 `const&` 接受参数（对于左值）之外，
  添加一个按 `&&` 传递参数（对于右值）的重载，并在函数体中将之 `std::move` 到其目标之中。基本上，这个重载是“将被移动（will-move-from）”；参见 [F.18](#rf-consume)。
* 在特殊情况中，比如有多个“输入+复制”的参数时，考虑采用完美转发。参见 [F.19](#rf-forward)。

##### 示例

    int multiply(int, int); // 仅输入了 int，按值传递

    // suffix 仅作输入，但并不如 int 那样廉价，因此按 const& 传递
    string& concatenate(string&, const string& suffix);

    void sink(unique_ptr<widget>);  // 仅作输入，但移动了这个 widget 的所有权

避免“为了效率”而按 `T&&` 来传递参数这类的“玄奥技巧”。
关于按 `&&` 传递带来性能好处的大多数传言都是假的或者是脆弱的（不过也请参考 [F.18](#rf-consume) 和 [F.19](#rf-forward)）。

##### 注解

可以假定引用都指代了某个有效对象（语言规则）。
“空引用”（正规地说）是不存在的。
如果要表示一个非强制的值，请使用指针，`std::optional`，或者一个用以代表“没有值”的特殊值。

##### 强制实施

* 【简单】〔基础〕 当按值传递的参数的大小大于 `2 * sizeof(void*)` 时给出警告。
  建议代之以 `const` 的引用。
* 【简单】〔基础〕 当按 `const` 引用传递的参数的大小小于或等于 `2 * sizeof(void*)` 时给出警告。建议代之以按值传递。
* 【简单】〔基础〕 当按 `const` 引用传递的参数被 `move` 时给出警告。

##### 例外

使用 `shared_ptr` 等类型来表达共享所有权时，应遵循 [R.34](#rr-sharedptrparam-owner) 或 [R.36](#rr-sharedptrparam-const)，
取决于函数是否无条件地获取实参的引用。

### <a id="rf-inout"></a>F.17: 对于“输入/输出（in-out）”参数，按非 `const` 引用进行传递

##### 理由

让调用者明了这个对象假定将会被改动。

##### 示例

    void update(Record& r);  // 假定 update 将会写入 r

##### 注解

一些用户定义和标准程序库的类型，如 `span<T>` 或迭代器等，
是[可廉价复制](#rf-in)的，并可按值传递，
这样做时具有可改动（in-out）引用语义：

    void increment_all(span<int> a)
    {
      for (auto&& e : a)
        ++e;
    }

##### 注解

`T&` 参数既可以向函数中传递信息，也可以传递出来。
因此 `T&` 能够作为“输入/输出”参数。这点本身就可能是一种错误的来源：

    void f(string& s)
    {
        s = "New York";  // 不明显的错误
    }

    void g()
    {
        string buffer = ".................................";
        f(buffer);
        // ...
    }

这里，`g()` 的作者提供了一个缓冲区让 `f()` 来填充，但 `f()` 仅仅替换掉了它（以多少比简单的字符复制高一些的成本）。
如果 `g()` 的作者对 `buffer` 的大小作出了错误的假设的话，就会发生糟糕的逻辑错误。

##### 强制实施

* 【中等】〔基础〕 对带有指向非 `const` 的引用参数但又*不*向其进行写入的函数给出警告。
* 【简单】〔基础〕 当按引用传递的非 `const` 参数被进行 `move` 时给出警告。

### <a id="rf-consume"></a>F.18: 对于“将被移动（will-move-from）”参数，按 `X&&` 进行传递并对参数 `std::move`

##### 理由

这样做很高效，并且消除了调用点的 BUG：`X&&` 绑定到右值，而要传递左值的话则要求在调用点明确进行 `std::move`。

##### 示例

    void sink(vector<int>&& v)  // 无论参数所拥有的是什么，sink 都获得了其所有权
    {
        // 通常这里可能有对 v 的 const 访问
        store_somewhere(std::move(v));
        // 通常这里不再会使用 v 了；它已经被移走
    }

注意，`std::move(v)` 使得 `store_somewhere()` 可以把 `v` 遗留为被移走的状态。
[这可能很危险](#rc-move-semantic)。


##### 例外

只能移动并且移动廉价的唯一拥有者类型，比如 `unique_ptr`，也可以按值传递，这样写起来更简单而且效果相同。按值传递确实产生了一次额外的（廉价）移动操作，但我们更加优先于简单性和清晰性。

例如：

    template<class T>
    void sink(std::unique_ptr<T> p)
    {
        // 使用 p ... 可能在之后的什么地方 std::move(p)
    }   // p 被销毁

##### 例外

当“将被移动”的形参是 `shared_ptr` 时，应遵循 [R.34](#rr-sharedptrparam-owner)，并按值传递 `shared_ptr`。

##### 强制实施

* 对于所有 `X&&` 参数（其中的 `X` 不是模板类型参数的名字），如果函数体中使用它时没有用 `std::move`，就将其标明。
* 标明对已经被移动过的对象的访问。
* 不要有条件地从对象进行移动。

### <a id="rf-forward"></a>F.19: 对于“转发（forward）”参数，按 `TP&&` 进行传递并只对参数 `std::forward`

##### 理由

如果一个对象要被传递给其他代码而并不在本函数中直接使用，我们就想让这个函数对于该参数的 `const` 性质和右值性质来说是中立的。

这种情况下，而且只有这种情况下，才应当让参数为 `TP&&`，其中 `TP` 为模板类型参数——它既*忽略*了也*保持*了 `const` 性质和右值性质。因而使用 `TP&&` 的任何代码都隐含地声称它自己并不关心变量的 `const` 性质和右值性质（因为这被忽略了），但它有意把值继续传递给其他确实关心 `const` 性质和右值性质的代码（因为这也是被保持的）。把 `TP&&` 用于参数类型上是安全的，因为从调用方传递来的任何临时对象都会在函数调用期间一直存活。基本上 `TP&&` 类型的参数应当总是在函数体中通过 `std::forward` 来继续传递。

##### 示例

你通常在每个静态控制流路径中恰好进行一次完整的形参（或形参包组，通过 `...`）的转发：

    template<class F, class... Args>
    inline auto invoke(F f, Args&&... args)
    {
        return f(forward<Args>(args)...);
    }

##### 示例

有时候，你可能会在每个静态控制流路径中按每个子对象一次的方式分段转发一个组合形参：

    template<class PairLike>
    inline auto test(PairLike&& pairlike)
    {
        // ...
        f1(some, args, and, forward<PairLike>(pairlike).first);           // 转发 .first
        f2(and, forward<PairLike>(pairlike).second, in, another, call);   // 转发 .second
    }

##### 强制实施

* 对于接受 `TP&&` 参数的函数（其中的 `TP` 不是模板类型参数的名字），如果函数对它做了任何别的事，而不是在每个静态路径中都正好进行一次 `std::forward`，或者在每个静态路径中对其进行多次 `std::forward` 但限定为不同的数据成员均正好进行一次，就将函数进行标明。

### <a id="rf-out"></a>F.20: 对于“输出（out）”值，采用返回值优先于输出参数

##### 理由

返回值是自我说明的，而 `&` 参数则既可能是输入/输出的也可能是仅输出的，并且倾向于被误用。

适用的情况也包括如标准容器这样的大型对象，它们为性能因素使用了隐式的移动操作，并且避免进行显式的内存管理。

当有多个值要返回时，[使用元组](#rf-out-multi)或者类似的多成员类型。

##### 示例

    // OK: 返回指向具有 x 值的元素的指针
    vector<const int*> find_all(const vector<int>&, int x);

    // 不好: 把指向具有 x 值的元素的指针放入 out
    void find_all(const vector<int>&, vector<const int*>& out, int x);

##### 注解

含有许多（每个都廉价移动的）元素的 `struct`，聚合起来则可能是移动操作昂贵的。

##### 例外

* 对于非具体类型，比如继承层次中的类型来说，可以用 `unique_ptr` 或 `shared_ptr` 来返回对象。
* 如果类型的移动操作昂贵（比如 `array<BigPOD>`），就考虑将其分配在自由存储中并返回一个句柄（比如 `unique_ptr`），或者传递一个指代用以填充的非 `const` 目标对象的引用（将其用作输出参数）。
* 对于内部循环中的多次函数调用之间重用自带容量的对象（比如 `std::string` 和 `std::vector`）：[将其按照输入/输出参数处理，并按引用传递](#rf-out-multi)。

##### 示例

假设 `Matrix` 带有移动操作（可能它将其元素都保存在一个 `std::vector` 中）：

    Matrix operator+(const Matrix& a, const Matrix& b)
    {
        Matrix res;
        // ... 用二者的和填充 res ...
        return res;
    }

    Matrix x = m1 + m2;  // 移动构造函数

    y = m3 + m3;         // 移动赋值


##### 注解

返回值优化无法处理赋值的情况，不过移动赋值却可以。

##### 示例

    struct Package {      // 特殊情况: 移动操作昂贵的对象
        char header[16];
        char load[2024 - 16];
    };

    Package fill();       // 不好: 大型的返回值
    void fill(Package&);  // OK

    int val();            // OK
    void val(int&);       // 不好: val 会不会读取参数？

##### 强制实施

* 对于指代非 `const` 的引用参数，如果其被写入之前未进行过读取，而且其类型能够廉价地返回，则标记它们；它们应当是“输入”的返回值。

### <a id="rf-out-multi"></a>F.21: 要返回多个“输出”值，优先返回结构体或元组（tuple）

##### 理由

返回值是自我说明为“仅输出”值的。
注意，C++ 是支持多返回值的，按约定使用的是 `tuple`（包括 `pair`），并可以在调用点使用 `tie` 或结构化绑定（C++17）以带来更多的便利。
优先使用具名的结构体，使其返回值具有语义。不过，没有名字的 `tuple` 在泛型代码中则很有用。

##### 示例

    // 不好: 在代码注释作用说明仅作输出的参数
    int f(const string& input, /*output only*/ string& output_data)
    {
        // ...
        output_data = something();
        return status;
    }

    // 好: 自我说明的
    tuple<int, string> f(const string& input)
    {
        // ...
        return {status, something()};
    }

C++98 的标准库已经使用这种风格了，因为 `pair` 就像一种两个元素的 `tuple` 一样。
例如，给定一个 `set<string> my_set`，请考虑：

    // C++98
    result = my_set.insert("Hello");
    if (result.second) do_something_with(result.first);    // 变通方案

在 C++11 中我们可以这样写，将结果直接放入现存的局部变量中：

    Sometype iter;                                // 如果我们还未因为别的目的而使用
    Someothertype success;                        // 这些变量，则进行默认初始化

    tie(iter, success) = my_set.insert("Hello");   // 普通的返回值
    if (success) do_something_with(iter);

而在 C++17 中，我们可以使用“结构化绑定”对多个变量进行声明和初始化：

    if (auto [ iter, success ] = my_set.insert("Hello"); success) do_something_with(iter);

##### 例外

有时候需要把对象传递给函数让其操纵它的状态。
这种情况下，按引用传递对象 [`T&`](#rf-inout) 通常是恰当的技巧。
显式传递一个输入/输出参数再让其作为返回值返回出来通常是没必要的。
例如：

    istream& operator>>(istream& in, string& s);    // 与 std::operator>>() 很相似

    for (string s; in >> s; ) {
        // 对文本行做些事
    }

这里，`s` 和 `in` 都用作了输入/输出参数。
`in` 按（非 `const`）引用来传递，以便可以操作其状态。
`s` 的传递是为避免重复进行分配。
通过重用 `s`（按引用传递），我们只需要在为扩展 `s` 的容量时才会分配新的内存。
这种技巧有时候被称为“调用方分配的输出参数”模式，它特别适合于
诸如 `string` 和 `vector` 这样需要进行自由存储分配的类型。

比较一下，如果所有的值都按返回值传递出来的话，得像如下这样做：

    pair<istream&, string> get_string(istream& in)  // 不建议这样做
    {
        string s;
        in >> s;
        return {in, move(s)};
    }

    for (auto p = get_string(cin); p.first; ) {
        // 对 p.second 做些事
    }

我们觉得这样明显不够简洁，而且性能明显更差。

当严格理解这条规则（F.21）时，这些例外并不真的算是例外，因为它依赖于输入/输出参数，
而不是规则中提到的单纯的输出参数。
不过我们倾向于进行明确而不是精巧的说明。

##### 注解

许多情况下，返回某种用户定义的某个专门的类型是有好处的。
例如：

    struct Distance {
        int value;
        int unit = 1;   // 1 表示一米
    };

    Distance d1 = measure(obj1);        // 访问 d1.value 和 d1.unit
    auto d2 = measure(obj2);            // 访问 d2.value 和 d2.unit
    auto [value, unit] = measure(obj3); // 访问 value 和 unit；
                                        // 对于了解 measure() 的人来说有点多余
    auto [x, y] = measure(obj4);        // 请勿如此；这很可能造成混乱

只有当返回的值表现的是几个无关实体而不是某个抽象的时候，才应使用过于通用的 `pair` 和 `tuple`。

作为另一个例子，应当使用像 `variant<T, error_code>` 这样的专门的类型，而不使用通用的 `tuple`。

##### 注解

当所要返回的元组是从复制操作昂贵的局部变量进行初始化时，
可以用显式 `move` 有效避免复制：

    pair<LargeObject, LargeObject> f(const string& input)
    {
        LargeObject large1 = g(input);
        LargeObject large2 = h(input);
        // ...
        return { move(large1), move(large2) }; // 没有复制
    }

还可以：

    pair<LargeObject, LargeObject> f(const string& input)
    {
        // ...
        return { g(input), h(input) }; // 没有复制，没有移动
    }

请注意这与 [ES.56](#res-move) 的 `return move(...)` 反模式是不同的。

##### 强制实施

* 输出参数应当被替换为返回值。
  输出参数时由函数写入的，调用了非 `const` 成员函数的，或者将它作为非 `const` 参数继续传递的参数。

### <a id="rf-ptr-ref"></a>F.60: 当“没有参数”是有效的选项时，采用 `T*` 优先于 `T&`

##### 理由

指针（`T*`）可能为 `nullptr`，而引用（`T&`）则不能，不存在合法的“空引用”。
有时候用 `nullptr` 作为一种代表“没有对象”的方式是有用处的，但若是没有这种情况的话，使用引用的写法更简单，而且可能会产生更好的代码。

##### 示例

    string zstring_to_string(zstring p) // zstring 就是 char*; 这是一个 C 风格的字符串
    {
        if (!p) return string{};    // p 可能为 nullptr; 别忘了要检查
        return string{p};
    }

    void print(const vector<int>& r)
    {
        // r 指代一个 vector<int>; 不需要检查
    }

##### 注解

构造出一个本质上是 `nullptr` 的引用是可能的，但不是合法的 C++ 代码（比如，`T* p = nullptr; T& r = *p;`）。
这种错误非常罕见。

##### 注解

如果你更喜欢指针写法（`->` 以及 `*` vs. `.`）的话，`not_null<T*>` 可以提供和 `T&` 相同的保证。

##### 强制实施

* Flag ???

### <a id="rf-ptr"></a>F.22: 用 `T*`，`owner<T*>` 或者智能指针来代表一个对象

##### 理由

可读性：这样能够明确普通指针的含义。
带来了显著的工具支持。

##### 注解

在传统的 C 和 C++ 代码中，普通的 `T*` 有各种互相没什么关联的用法，比如：

* 标识单个对象（本函数内不会进行 delete）
* 指向分配于自由存储之中的一个对象（随后将会 delete）
* 持有 `nullptr` 值
* 标识一个 C 风格字符串（以零结尾的字符数组）
* 标识一个数组，其长度被分开指明
* 标识数组中的一个位置

这样就难于了解代码真正做了什么和打算做什么。
它也会使检查工作和工具支持复杂化。

##### 示例

    void use(int* p, int n, char* s, int* q)
    {
        p[n - 1] = 666; // 不好: 不知道 p 是不是指向了 n 个元素；
                        // 应当假定它并非如此，否则应当使用 span<int>
        cout << s;      // 不好: 不知道 s 指向的是不是以零结尾的字符数组；
                        // 应当假定它并非如此，否则应当使用 zstring
        delete q;       // 不好: 不知道 *q 是不是在自由存储中分配的；
                        // 否则应当使用 owner
    }

更好的做法

    void use2(span<int> p, zstring s, owner<int*> q)
    {
        p[p.size() - 1] = 666; // OK, 会造成范围错误
        cout << s; // OK
        delete q;  // OK
    }

##### 注解

`owner<T*>` 表示所有权，`zstring` 表示 C 风格的字符串。

**再者**: 应当假定从指向 `T` 的智能指针（比如 `unique_ptr<T>`）中获得的 `T*`是指向单个元素的。

**参见**: [支持程序库](#gsl-guidelines-support-library)

**参见**: [请勿将数组作为单个指针来传递](#ri-array)

##### 强制实施

* 【简单】〔边界〕 对指针类型的表达式的算术操作，若其结果为指针类型的值，就给出警告。

### <a id="rf-nullptr"></a>F.23: 用 `not_null<T>` 来表明“空值（null）”不是有效的值

##### 理由

清晰性。以 `not_null<T>` 为参数的函数很明确地说明，应当由该函数的调用者来负责进行任何必要的 `nullptr` 检查。
相似地，以 `not_null<T>` 为返回值的函数很明确地说明，该函数的调用者无须检查 `nullptr`。

##### 示例

`not_null<T*>` 让读者（人类或机器）明了，在进行解引用前不需要检查 `nullptr`。
而且当进行调试时，可以对 `owner<T*>` 和 `not_null<T>` 进行植入来进行正确性检查。

考虑：

    int length(Record* p);

当调用 `length(p)` 时，我应该先检查 `p` 是否为 `nullptr` 吗？是不是应当由 `length()` 的实现来检查 `p` 是否为 `nullptr`？

    // 确保 p != nullptr 是调用者的任务
    int length(not_null<Record*> p);

    // length() 的实现者必须假定可能出现 p == nullptr
    int length(Record* p);

##### 注解

假定 `not_null<T*>` 不可能是 `nullptr`；而 `T*` 则可能为 `nullptr`；二者都可以在内存中表示为 `T*`（因此不会带来运行时开销）。

##### 注解

`not_null` 不仅对内建指针有效。它也能在 `unique_ptr`，`shared_ptr`，以及其他指针式的类型上使用。

##### 强制实施

* 【简单】 当函数中的一个原始指针在未测试 `nullptr`（或等价形式）之前就被解引用时，就给出警告。
* 【简单】 当函数中的一个原始指针有时候会在测试 `nullptr`（或等价形式）后进行解引用，而有时候不会时，就报错。
* 【简单】 当函数中的一个 `not_null` 指针进行了 `nullptr` 测试时，就给出警告。

### <a id="rf-range"></a>F.24: 用 `span<T>` 或者 `span_p<T>` 来代表一个半开序列

##### 理由

非正式和不明确的范围（range）是一种错误来源。

##### 示例

    X* find(span<X> r, const X& v);    // 在 r 中寻找 v

    vector<X> vec;
    // ...
    auto p = find({vec.begin(), vec.end()}, X{});  // 在 vec 中寻找 X{}

##### 注解

范围（Range）在 C++ 代码中十分常见。典型情况下，它们都是隐含的，且非常难于保证它们能够被正确使用。
特别地，给定一对儿参数 `(p, n)` 来代表数组 `[p:p+n)`，
通常来说不可能确定 `*p` 后面是不是真的存在 `n` 个元素。
`span<T>` 和 `span_p<T>` 两个简单的辅助类，分别用于代表范围 `[p:q)`，以及一个以 `p` 开头并以使谓词为真的第一个元素结尾的范围。

##### 示例

`span` 代表元素的范围，我们应当如何操作范围的各个元素呢？

    void f(span<int> s)
    {
        // 范围的遍历（保证正确进行）
        for (int x : s) cout << x << '\n';

        // C 风格的遍历（可能带有检查）
        for (gsl::index i = 0; i < s.size(); ++i) cout << s[i] << '\n';

        // 随机访问（可能带有检查）
        s[7] = 9;

        // 截取指针（可能带有检查）
        std::sort(&s[0], &s[s.size() / 2]);
    }

##### 注解

`span<T>` 对象并不拥有其元素，而且很小，可以按值传递。

把一个 `span` 对象作为参数传递的效率完全等同于传递一对儿指针参数或者传递一个指针和一个整数计数值。

**参见**: [支持程序库](#gsl-guidelines-support-library)

##### 强制实施

【复杂】 当对指针参数的访问是以其他整型类型的参数为边界限定时，就给出警告并建议改用 `span`。

### <a id="rf-zstring"></a>F.25: 用 `zstring` 或者 `not_null<zstring>` 来代表 C 风格的字符串

##### 理由

C 风格的字符串非常普遍。它们是按一种约定方式定义的：就是以零结尾的字符数组。
我们必须把 C 风格的字符串从指向单个字符的指针或者指向字符数组的老式的指针当中区分出来。

当不需要零结尾时，请使用 'string_view'。

##### 示例

考虑：

    int length(const char* p);

当调用 `length(p)` 时，我应该先检查 `p` 是否为 `nullptr` 吗？是不是应当由 `length()` 的实现来检查 `p` 是否为 `nullptr`？

    // length() 的实现者必须假定可能出现 p == nullptr
    int length(zstring p);

    // it is the caller's job to make sure p != nullptr
    int length(not_null<zstring> p);

##### 注解

`zstring` 不含有所有权。

**参见**: [支持程序库](#gsl-guidelines-support-library)

### <a id="rf-unique_ptr"></a>F.26: 当需要指针时，用 `unique_ptr<T>` 来传递所有权

##### 理由

使用 `unique_ptr` 是安全地传递指针的最廉价的方式。

**参见**：[C.50](#rc-factory)关于何时从一个工厂中返回 `shared_ptr`。

##### 示例

    unique_ptr<Shape> get_shape(istream& is)  // 从输入流中装配一个形状
    {
        auto kind = read_header(is); // 从输入中读取头部并识别下一个形状
        switch (kind) {
        case kCircle:
            return make_unique<Circle>(is);
        case kTriangle:
            return make_unique<Triangle>(is);
        // ...
        }
    }

##### 注解

当要传递的对象属于某个类层次，且将要通过接口（基类）来使用它时，你需要传递一个指针而不是对象。

##### 强制实施

【简单】 当函数返回了局部分配了的原始指针时就给出警告。建议改为使用 `unique_ptr` 或 `shared_ptr`。

### <a id="rf-shared_ptr"></a>F.27: 用 `shared_ptr<T>` 来共享所有权

##### 理由

使用 `std::shared_ptr` 是表示共享所有权的标准方式。其含义是，最后一个拥有者负责删除对象。

##### 示例

    shared_ptr<const Image> im { read_image(somewhere) };

    std::thread t0 {shade, args0, top_left, im};
    std::thread t1 {shade, args1, top_right, im};
    std::thread t2 {shade, args2, bottom_left, im};
    std::thread t3 {shade, args3, bottom_right, im};

    // 脱离各线程
    // 最后执行完的线程会删除这个图像

##### 注解

如果同时不可能超过一个所有者的话，优先采用 `unique_ptr` 而不是 `shared_ptr`。
`shared_ptr` 的作用是共享所有权。

注意，过于普遍的使用 `shared_ptr` 是有成本的（`shared_ptr` 的引用计数上的原子性操作会产生可测量的总体花费）。

##### 替代方案

让单个对象来拥有这个共享对象（比如一个有作用域的对象），并当其所有使用方都完成工作后（最好隐含地）销毁它。

##### 强制实施

【无法强制实施】 这种模式过于复杂，无法可靠地进行检测。

### <a id="rf-return-ptr"></a>F.42: 返回 `T*` 来（仅仅）给出一个位置

##### 理由

指针就是用来干这个的。
使用 `T*` 来传递所有权其实是一种误用。

##### 示例

    Node* find(Node* t, const string& s)  // 在 Node 组成的二叉树中寻找 s
    {
        if (!t || t->name == s) return t;
        if ((auto p = find(t->left, s))) return p;
        if ((auto p = find(t->right, s))) return p;
        return nullptr;
    }

`find` 所返回的指针如果不是 `nullptr` 的话，就指定了一个含有 `s` 的 `Node`。
重要的是，这里面并没有暗含着把所指向的对象的所有权传递给调用者。

##### 注解

迭代器、索引值和引用也可以用来传递位置。
[当不需要使用 `nullptr`](#rf-ptr-ref)，或者[当不会改变被指代的对象](???)时，用引用通常比用指针更好。

##### 注解

不要返回指向某个不在调用方的作用域中的东西的指针；参见 [F.43](#rf-dangle)。

**参见**: [有关如何避免悬挂指针的讨论](#???)

##### 强制实施

* 标记出施加在普通 `T*` 上的 `delete`，`std::free()` 等等。
只有所有者才能被删除。
* 标记出赋值给普通 `T*` 的 `new`，`malloc()` 等等。
只有所有者才应当负责进行删除。

### <a id="rf-dangle"></a>F.43: 不要（直接或间接）返回指向局部对象的指针或引用

##### 理由

避免由于使用了这种悬挂指针而造成的程序崩溃和数据损坏。

##### 示例, 不好

从函数返回后，其中的局部对象就不再存在了：

    int* f()
    {
        int fx = 9;
        return &fx;  // 不好
    }

    void g(int* p)   // 貌似确实是无辜的
    {
        int gx;
        cout << "*p == " << *p << '\n';
        *p = 999;
        cout << "gx == " << gx << '\n';
    }

    void h()
    {
        int* p = f();
        int z = *p;  // 从已经丢弃的栈帧中读取（不好）
        g(p);        // 把指向已丢弃栈帧的指针传递给函数（不好）
    }

我在一种流行的实现上得到了以下输出：

    *p == 999
    gx == 999

我预期这样的结果是因为，对 `g()` 的调用重用了被 `f()` 的调用所丢弃的栈空间，因此 `*p` 所指代的空间应当会被 `gx` 所占据。

* 请想象一下当 `fx` 和 `gx` 类型不同时会发生什么。
* 请想象一下当 `fx` 或 `gx` 的类型带有不变式时会发生什么。
* 请想象一下当在更大的一组函数之间传递的不止是悬挂指针时会发生什么。
* 请想象一下一个攻击者能够利用悬挂指针干些什么。

幸运的是，大多数（全部？）的当代编译器都可以识别这种简单的情况并给出警告。

##### 注解

这同样适用于引用：

    int& f()
    {
        int x = 7;
        // ...
        return x;  // 不好: 返回指代即将被销毁的对象的引用
    }

##### 注解

这条仅适用于非 `static` 的局部变量。
所有的 `static` 变量都是（顾名思义）静态分配的，因此指向它们的指针不可能变为悬挂的。

##### 示例, 不好

并非所有的局部变量指针的泄漏都是那么明显的：

    int* glob;       // 全局变量的不好的方面太多了

    template<class T>
    void steal(T x)
    {
        glob = x();  // 不好
    }

    void f()
    {
        int i = 99;
        steal([&] { return &i; });
    }

    int main()
    {
        f();
        cout << *glob << '\n';
    }

我这次成功从 `f` 的调用所丢弃的位置上读到了数据。
存于 `glob` 中的指针可能在很晚才被使用，并可能以无法预测的方式造成各种麻烦。

##### 注解

局部变量的地址的“返回”或者泄漏方式，可能是返回语句，以 `T&` 输出参数，以所返回对象的成员，以所返回数组的元素，还有更多其他方式。

##### 注解

还可以构造出相似的从内部作用域“泄漏”到外部作用域的例子；
对这样的例子应当按照与从函数中泄漏指针的相同方式来处理。

这个问题的一个略有不同的变体是，把指针放入容器使其生存期超过其所指向的对象。

**参见**: 另一种获得悬挂指针的方式是[指针失效](#???)。
这种情况也可以用相似的技术来检测和避免。

##### 强制实施

* 编译器通常可以发现返回局部对象 引用，许多情况下也可以发现返回指向局部对象的指针。
* 静态分析可以发现许多常见的确定指针位置的使用模式（因而可以消除掉悬挂指针）

### <a id="rf-return-ref"></a>F.44: 当不想进行复制，而“没有对象被返回”不是有效的选项时，返回 `T&`

##### 理由

语言规则保证 `T&` 会指代对象，因此不需要对其测试 `nullptr`。

**参见**: 所返回的引用诀不能蕴含所有权的传递：
[有关如何避免悬挂指针的讨论](#???)以及[有关所有权的讨论](#???)。

##### 示例

    class Car
    {
        array<wheel, 4> w;
        // ...
    public:
        wheel& get_wheel(int i) { Expects(i < w.size()); return w[i]; }
        // ...
    };

    void use()
    {
        Car c;
        wheel& w0 = c.get_wheel(0); // w0 与 c 的生存期相同
    }

##### 强制实施

对不存在可能产生 `nullptr` 的 `return` 表达式的函数进行标记。

### <a id="rf-return-ref-ref"></a>F.45: 不要返回 `T&&`

##### 理由

它要求返回对已销毁的临时对象的引用。
`&&` 是吸引临时对象的符号。

##### 示例

返回的右值引用超出了返回的完整表达式结束的范围：

    auto&& x = max(0, 1);   // 到目前为止，没问题
    foo(x);                 // 未定义的行为

这种用法是频繁产生 bug 的根源，经常错误地报告为编译器错误。
函数的实现者应避免为用户设置此类陷阱。

[生存期安全性](#ss-lifetime)完全执行时，会捕捉到这个问题。


##### 示例

当临时的引用”向下”传递给被调用对象时，返回右值引用是正常的；
然后，临时对象保证比函数调用生命期更长（参见 [F.18](#rf-consume) 和 [F.19](#rf-forward)）。
但是，将这样的引用“向上”传递给更大的调用范围时，不好。
对于通过普通引用或完美传递方式传递参数，并希望返回值的通过函数，使用简单的 `auto` 而不是 `auto &&` 返回推导的类型。

假定 “F” 按值返回：

    template<class F>
    auto&& wrapper(F f)
    {
        log_call(typeid(f)); // 或者别的什么测量手段
        return f();          // 不好：返回一个临时对象的引用
    }

更好的方式：

    template<class F>
    auto wrapper(F f)
    {
        log_call(typeid(f)); // 或者别的什么测量手段
        return f();          // 好
    }


##### 例外

`std::move` 和 `std::forward` 确实会返回 `&&`，但它们只不过是强制转换 —— 只会按惯例在某些表达式上下文中使用，其中指代临时对象的引用只会在该临时对象被销毁之前在同一个表达式中被传递。我们不知道还存在任何别的返回 `&&` 的好例子。

##### 强制实施

对除了 `std::move` 和 `std::forward` 之外的任何把 `&&` 作为返回类型的情况都进行标记。

### <a id="rf-main"></a>F.46: `int` 是 `main()` 的返回类型

##### Reason

这是一条语言规则，但通常被“语言扩展”所违反，因此值得一提。
把 `main`（即程序中的那个全局的 `main`）声明为 `void` 会限制其可移植性。

##### 示例

        void main() { /* ... */ };  // 不好，不符合 C++

        int main()
        {
            std::cout << "This is the way to do it\n";
        }

##### 注解

我们提出这条规则，只是因为这种错误持续存在于大众之间。
注意，虽然其返回类型不是 `void`，但主函数并不需要明确的返回语句。

##### 强制实施

* 编译器应当做到。
* 如果编译器做不到，就让工具把它标记出来。

### <a id="rf-assignment-op"></a>F.47: 赋值运算符返回 `T&`

##### 理由

运算符重载的惯例（尤其是对于具体类型来说），是让
`operator=(const T&)` 实施赋值之后返回（非 `const`）的
`*this`。这就确保了与标准库类型之间的一致性，并遵从了
“像 `int` 一样工作”的原则。

##### 注解

历史上曾有过一些建议让赋值运算符返回 `const T&`。
这主要是为了避免 `(a = b) = c` 形式的代码 —— 这种代码其实并不常见到足以成为违反与标准类型之间一致性的理由。

##### 示例

    class Foo
    {
     public:
        ...
        Foo& operator=(const Foo& rhs)
        {
          // 复制各个成员。
          ...
          return *this;
        }
    };

##### 强制实施

应当通过工具对所有赋值运算符的返回类型（和返回值）进行检查
来强制实施。

### <a id="rf-return-move-local"></a>F.48: 不要用 `return std::move(local)`

##### 理由

有了确保进行的副本消除之后，现在在返回语句中明确使用 `std::move` 几乎总是不良的实践。

##### 示例，不好

    S f()
    {
      S result;
      return std::move(result);
    }

##### 示例，好

    S f()
    {
      S result;
      return result;
    }

##### 强制实施

应当通过工具对返回语句进行检查来强制实施。

### <a id="rf-return-const"></a>F.49: 不要返回 `const T`

##### 理由

不建议返回 `const` 值。
这种老旧的建议已经过时了；它并不会带来什么价值，而且还会对移动语义造成影响。

##### 示例

    const vector<int> fct();    // 不好: 这个 "const" 带来的麻烦超过其价值

    void g(vector<int>& vx)
    {
        // ...
        fct() = vx;   // 被 "const" 所禁止
        // ...
        vx = fct(); // 昂贵的复制："const" 抑制掉了移动语义
        // ...
    }

要求对返回值添加 `const` 的理由是可以防止（非常少见的）对临时对象的意外访问。
而反对的理由则是它妨碍了（非常常见的）对移动语义的利用。

**另见**: [F.20，有关“out”输出值的一般条款](#rf-out)

##### 强制实施

* 标记 `const` 返回值。修正方法：移除 `const` 使其变为返回非 `const` 值。


### <a id="rf-capture-vs-overload"></a>F.50: 当函数不适用时（不能俘获局部变量，或者不能编写局部函数），就使用 Lambda

##### 理由

函数不能俘获局部变量且不能在局部作用域中进行定义；当想要这些能力时，如果可能就应当使用 lambda，不行的就用手写的函数对象。另一方面，lambda 和函数对象是不能重载的；如果想要重载，就优先使用函数（让 lambda 重载的变通方案相当繁复）。如果两种方式都不行的话，就优先写一个函数；应当只使用所需的最简工具。

##### 示例

    // 编写只会接受 int 或 string 的函数
    // -- 重载是很自然的
    void f(int);
    void f(const string&);

    // 编写需要俘获局部状态的函数对象，可以出现于
    // 语句或者表达式作用域中 -- lambda 更自然
    vector<work> v = lots_of_work();
    for (int tasknum = 0; tasknum < max; ++tasknum) {
        pool.run([=, &v] {
            /*
            ...
            ... 处理 v 的 1 / max, 即第 tasknum 个部分
            ...
            */
        });
    }
    pool.join();

##### 例外

泛型的 lambda 可以提供一种更精简的编写函数模板的方式，因此会比较有用，虽然普通的函数模板用稍多一点儿的语法可以做到同样的事情。这种优势在未来一旦所有的函数都获得了 Concept 参数的能力之后就可能会消失。

##### 强制实施

* 有名字的非泛型 lambda（比如 `auto x = [](int i) { /*...*/; };`），而其并未发生俘获并且出现于全局作用域，对它们给出警告。代之以编写常规的函数。

### <a id="rf-default-args"></a>F.51: 如果需要作出选择，采用默认实参应当优先于进行重载

##### 理由

默认实参本就是为一个单一实现提供替代的接口的。
无法保证一组重载函数全部都实现相同的语义。
使用默认实参可以避免出现代码重复。

##### 注解

当变化来自相同类型的一组参数时，需要在默认实参和重载两种方案之间进行选择。
例如：

    void print(const string& s, format f = {});

相对的则是

    void print(const string& s);  // 使用默认的 format
    void print(const string& s, format f);

如果要为一组不同类型来实现语义上等价的操作，就不需要进行选择了。例如：

    void print(const char&);
    void print(int);
    void print(zstring);

##### 参见


[虚函数的默认实参](#rf-virtual-default-arg}

##### 强制实施

* 如果某个重载集合中的各个重载具有一个共同的形参前缀（例如 `f(int)`，`f(int, const string&)`，`f(int, const string&, double)`），则为其给出警告。（注意：如果这条强制措施实践中产生太多消息，请对此进行复查。）

### <a id="rf-reference-capture"></a>F.52: 对于局部使用的（也包括传递给算法的）lambda，优先采用按引用俘获

##### 理由

为了效率和正确性，当使用局部的 lambda 时，你基本上总是需要进行按引用俘获。这也包括编写或者调用并行算法的情形，因为它们在返回前会进行联结。

##### 讨论

效率方面的考虑是，大多数的类型都是按引用传递比按值传递更便宜。

正确性方面的考虑是，许多的调用都希望在调用点对原本的对象实施副作用（参见下面的示例）。而按值传递妨碍了这点。

##### 注解

不幸的是，并没有一种简单的方法，能够按 `const` 引用来捕获以获得其局部调用的效率，同时又妨碍其副作用。

##### 示例

此处，一个大型对象（网络消息）被传递给一个迭代算法，而它也许不够高效，或者能够正确复制这个消息（它可能是无法复制的）：

    std::for_each(begin(sockets), end(sockets), [&message](auto& socket)
    {
        socket.send(message);
    });

##### 示例

下面是一个简单的三阶段并行管线。每个 `stage` 对象封装了一个工作线程和一个队列，有一个用来把任务入队的 `process` 函数，且其析构函数会自动进行阻塞以在线程结束前等待队列变空。

    void send_packets(buffers& bufs)
    {
        stage encryptor([](buffer& b) { encrypt(b); });
        stage compressor([&](buffer& b) { compress(b); encryptor.process(b); });
        stage decorator([&](buffer& b) { decorate(b); compressor.process(b); });
        for (auto& b : bufs) { decorator.process(b); }
    }  // 自动阻塞以等待管线完成

##### 强制实施

对于按引用捕获的 lambda，若其并非局部地用在函数作用域中，或者若其被按引用传递给某个函数，则对其进行标记。（注意：这条规则是一种近似，但确实对按指针传递进行标记，它们更可能被受调方所保存，通过某个参数来向某个堆位置进行写入，返回 lambda，等等。生存期方面的规则也会提供一般性的规则，以针对包括通过 lambda 脱离的指针和引用进行标记。）

### <a id="rf-value-capture"></a>F.53: 对于非局部使用的（包括被返回的，在堆上存储的，或者传递给别的线程的）lambda，避免采用按引用俘获

##### 理由

指向局部对象的指针和引用不能超出它们的作用域而存活。按引用捕获的 lambda 恰是另外一种保存指向局部对象的引用的地方，因而当它们（或其副本）存活超出作用域的话，也不应该这样做。

##### 示例，不好

    int local = 42;

    // 需要局部对象的引用。
    // 注意，当程序离开作用域时，
    // 局部对象不再存在，因此
    // process() 的调用将带有未定义行为！
    thread_pool.queue_work([&] { process(local); });

##### 示例，好

    int local = 42;
    // 需要局部对象的副本。
    // 由于为局部变量建立了副本，它将在
    // 函数调用的全部时间内可用。
    thread_pool.queue_work([=] { process(local); });

##### 注解

如果必须捕获非局部指针，则应考虑使用 `unique_ptr`；它会处理生存期和同步问题。

如果必须捕获 `this` 指针，则应考虑使用 `[*this]` 捕获，它会创建整个对象的一个副本。

##### 强制实施

* 【简单】 当捕获列表中包含指代局部声明的变量的引用时给出警告。
* 【复杂】 当捕获列表中包含指代局部声明的变量的引用，而 lambda 被传递给非 `const` 且非局部的上下文时，进行标记。

### <a id="rf-this-capture"></a>F.54: 当俘获了 `this` 时，显式俘获所有的变量（不使用默认俘获）

##### 理由

这是容易混淆的。在成员函数里边写 `[=]` 貌似会按值来俘获，但其实会按引用俘获数据成员，因为它实际上按值俘获了不可见的 `this` 指针。如果你确实要这样做的话，请把 `this` 写明。

##### 示例

    class My_class {
        int x = 0;
        // ...

        void f()
        {
            int i = 0;
            // ...

            auto lambda = [=] { use(i, x); };   // 不好: “貌似”按复制/按值俘获
            // [&] 在当前的语言规则下的语义是一样的，也会复制 this 指针
            // [=,this] 和 [&,this] 也没好多少，并且也会导致混淆

            x = 42;
            lambda(); // 调用 use(0, 42);
            x = 43;
            lambda(); // 调用 use(0, 43);

            // ...

            auto lambda2 = [i, this] { use(i, x); }; // ok, 最明确并且最不混淆

            // ...
        }
    };

##### 注解

这在标准化之中正在进行积极的讨论，而且很可能在未来版本的标准中通过增加一种新的俘获模式或者调整 `[=]` 的含义而得到结局。当前的话，还是应当明确为好。

##### 强制实施

* 若指定了默认俘获（如 `=` 或 `&`）的 lambda 俘获列表并且还俘获了 `this` 的情况——无论是如 `[&, this]` 这样显式，还是通过 `[=]` 这样的默认俘获而又在函数体中使用了 `this`——对此进行标识。

### <a id="f-varargs"></a>F.55: 不要使用 `va_arg` 参数

##### 理由

从 `va_arg` 中读取时需要假定确实传递了正确类型的参数。
而向变参传递时则需要假定将会读取正确的类型。
这样是很脆弱的，因为其在语言中无法一般性地强制其安全，因而需要靠程序员的纪律来保证其正确。

##### 示例

    int sum(...)
    {
        // ...
        while (/*...*/)
            result += va_arg(list, int); // 不好，假定所传递的是 int
        // ...
    }

    sum(3, 2); // ok
    sum(3.14159, 2.71828); // 不好，未定义的行为

    template<class ...Args>
    auto sum(Args... args) // 好，而且更灵活
    {
        return (... + args); // 注意：C++17 的“折叠表达式”
    }

    sum(3, 2); // ok: 5
    sum(3.14159, 2.71828); // ok: ~5.85987

##### 替代方案

* 重载
* 变参模板
* `variant` 参数
* `initializer_list`（同质的）

##### 注解

有时候，对于并不涉及实际的参数传递的技巧来说，声明 `...` 形参有其作用，比如当声明“接受任何东西”的函数，以在重载集合中禁止“其他所有东西”，或在模板元程序中表达一种“全覆盖（catchall）”情况时。

##### 强制实施

* 为 `va_list`，`va_start`，或 `va_arg` 的使用给出诊断。
* 如果 vararg 参数的函数并未提供重载以为该参数位置指定更加特定的类型，则当其传递参数时给出诊断。修正：使用别的函数，或标明 `[[suppress(types)]]`。


### <a id="f-nesting"></a>F.56: 避免不必要的条件嵌套

##### 理由

浅层嵌套的条件语句使代码容易理解。也会使缩进结构清楚明了。
力求将基础代码放在最外层作用域，除非这样做会搞乱缩进。

##### 示例

使用防卫代码块来处理异常情况，并提早返回。

    // 不好：深层嵌套
    void foo() {
        ...
        if (x) {
            computeImportantThings(x);
        }
    }

    // 不好：还有多余的 else。
    void foo() {
        ...
        if (!x) {
            return;
        }
        else {
            computeImportantThings(x);
        }
    }

    // 好：提早返回，无多余 else
    void foo() {
        ...
        if (!x)
            return;

        computeImportantThings(x);
    }

##### 示例

    // 不好：不必要的条件嵌套
    void foo() {
        ...
        if (x) {
            if (y) {
                computeImportantThings(x);
            }
        }
    }

    // 好：合并条件 + 提早返回
    void foo() {
        ...
        if (!(x && y))
            return;

        computeImportantThings(x);
    }

##### 强制实施

标记多余的 `else`。
对函数体仅为包含一个代码块的条件语句的函数进行标记。
