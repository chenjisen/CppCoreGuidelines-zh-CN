
# <a id="s-expr"></a>ES: 表达式和语句

表达式和语句是用以表达动作和计算的最底层也是最直接的方式。局部作用域中的声明也是语句。

有关命名、注释和缩进的规则，参见 [NL: 命名与代码布局](#s-naming)。

一般规则：

* [ES.1: 优先采用标准库而不是其他的库或者“手工自制代码”](#res-lib)
* [ES.2: 优先采用适当的抽象而不是直接使用语言功能特性](#res-abstr)
* [ES.3: 避免重复（DRY），避免冗余代码](#res-dry)

声明的规则：

* [ES.5: 保持作用域尽量小](#res-scope)
* [ES.6: 在 for 语句的初始化式和条件中声明名字以限制其作用域](#res-cond)
* [ES.7: 保持常用的和局部的名字尽量简短，而让非常用的和非局部的名字较长](#res-name-length)
* [ES.8: 避免使用看起来相似的名字](#res-name-similar)
* [ES.9: 避免 `ALL_CAPS` 式的名字](#res-not-caps)
* [ES.10: 每条声明中（仅）声明一个名字](#res-name-one)
* [ES.11: 使用 `auto` 来避免类型名字的多余重复](#res-auto)
* [ES.12: 不要在嵌套作用域中重用名字](#res-reuse)
* [ES.20: 坚持为对象进行初始化](#res-always)
* [ES.21: 不要在确实需要使用变量（或常量）之前就引入它](#res-introduce)
* [ES.22: 要等到获得了用以初始化变量的值之后才声明变量](#res-init)
* [ES.23: 优先使用 `{}` 初始化式语法](#res-list)
* [ES.24: 用 `unique_ptr<T>` 来保存指针](#res-unique)
* [ES.25: 应当将对象声明为 `const` 或 `constexpr`，除非后面需要修改其值](#res-const)
* [ES.26: 不要用一个变量来达成两个不相关的目的](#res-recycle)
* [ES.27: 使用 `std::array` 或 `stack_array` 作为栈上的数组](#res-stack)
* [ES.28: 为复杂的初始化（尤其是 `const` 变量）使用 lambda](#res-lambda-init)
* [ES.30: 不要用宏来操纵程序文本](#res-macros)
* [ES.31: 不要用宏来作为常量或“函数”](#res-macros2)
* [ES.32: 对所有的宏名采用 `ALL_CAPS` 命名方式](#res-all_caps)
* [ES.33: 如果必须使用宏的话，请为之提供唯一的名字](#res-macros)
* [ES.34: 不要定义（C 风格的）变参函数](#res-ellipses)

表达式的规则：

* [ES.40: 避免复杂的表达式](#res-complicated)
* [ES.41: 对运算符优先级不保准时应使用括号](#res-parens)
* [ES.42: 保持单纯直接的指针使用方式](#res-ptr)
* [ES.43: 避免带有未定义的求值顺序的表达式](#res-order)
* [ES.44: 不要对函数参数求值顺序有依赖](#res-order-fct)
* [ES.45: 避免“魔法常量”，采用符号化常量](#res-magic)
* [ES.46: 避免窄化转换](#res-narrowing)
* [ES.47: 使用 `nullptr` 而不是 `0` 或 `NULL`](#res-nullptr)
* [ES.48: 避免强制转换](#res-casts)
* [ES.49: 当必须使用强制转换时，使用具名的强制转换](#res-casts-named)
* [ES.50: 不要强制掉 `const`](#res-casts-const)
* [ES.55: 避免发生对范围检查的需要](#res-range-checking)
* [ES.56: 仅在确实需要明确移动某个对象到别的作用域时才使用 `std::move()`](#res-move)
* [ES.60: 避免在资源管理函数之外使用 `new` 和 `delete`](#res-new)
* [ES.61: 用 `delete[]` 删除数组，用 `delete` 删除非数组对象](#res-del)
* [ES.62: 不要在不同的数组之间进行指针比较](#res-arr2)
* [ES.63: 不要产生切片](#res-slice)
* [ES.64: 使用 `T{e}` 写法来进行构造](#res-construct)
* [ES.65: 不要解引用无效指针](#res-deref)

语句的规则：

* [ES.70: 面临选择时，优先采用 `switch` 语句而不是 `if` 语句](#res-switch-if)
* [ES.71: 面临选择时，优先采用范围式 `for` 语句而不是普通 `for` 语句](#res-for-range)
* [ES.72: 当存在显然的循环变量时，优先采用 `for` 语句而不是 `while` 语句](#res-for-while)
* [ES.73: 当没有显然的循环变量时，优先采用 `while` 语句而不是 `for` 语句](#res-while-for)
* [ES.74: 优先在 `for` 语句的初始化部分中声明循环变量](#res-for-init)
* [ES.75: 避免使用 `do` 语句](#res-do)
* [ES.76: 避免 `goto`](#res-goto)
* [ES.77: 尽量减少循环中使用的 `break` 和 `continue`](#res-continue)
* [ES.78: 不要依靠 `switch` 语句中的隐含直落行为](#res-break)
* [ES.79: `default`（仅）用于处理一般情况](#res-default)
* [ES.84: 不要试图声明没有名字的局部变量](#res-noname)
* [ES.85: 让空语句显著可见](#res-empty)
* [ES.86: 避免在原生的 `for` 循环中修改循环控制变量](#res-loop-counter)
* [ES.87: 请勿在条件上添加多余的 `==` 或 `!=`](#res-if)

算术规则：

* [ES.100: 不要进行有符号和无符号混合运算](#res-mix)
* [ES.101: 使用无符号类型进行位操作](#res-unsigned)
* [ES.102: 使用有符号类型进行算术运算](#res-signed)
* [ES.103: 避免上溢出](#res-overflow)
* [ES.104: 避免下溢出](#res-underflow)
* [ES.105: 避免除整数零](#res-zero)
* [ES.106: 不要试图用 `unsigned` 来防止负数值](#res-nonnegative)
* [ES.107: 不要对下标使用 `unsigned`，优先使用 `gsl::index`](#res-subscripts)

### <a id="res-lib"></a>ES.1: 优先采用标准库而不是其他的库或者“手工自制代码”

##### 理由

使用程序库的代码要远比直接使用语言功能特性的代码易于编写，更为简短，更倾向于更高的抽象层次，而且程序库代码假定已经经过测试。
ISO C++ 标准库是最广为了解而且经过最好测试的程序库之一。
它是任何 C++ 实现的一部分。

##### 示例

    auto sum = accumulate(begin(a), end(a), 0.0);   // 好

`accumulate` 的范围版本就更好了：

    auto sum = accumulate(v, 0.0); // 更好

但请不要手工编写众所周知的算法：

    int max = v.size();   // 不好：啰嗦，目的不清晰
    double sum = 0.0;
    for (int i = 0; i < max; ++i)
        sum = sum + v[i];

##### 例外

标准库的很大部分都依赖于动态分配（自由存储）。这些部分，尤其是容器但并不包括算法，并不适用于某些硬实时和嵌入式的应用的。在这些情况下，请考虑提供或使用类似的设施，比如说某个标准库风格的采用池分配器的容器实现。

##### 强制实施

并不容易。??? 寻找混乱的循环，嵌套的循环，长函数，函数调用的缺失，缺乏使用内建类型。圈复杂度？

### <a id="res-abstr"></a>ES.2: 优先采用适当的抽象而不是直接使用语言功能特性

##### 理由

“适当的抽象”（比如程序库或者类），更加贴近应用的概念而不是语言概念，将带来更简洁的代码，而且更容易进行测试。

##### 示例

    vector<string> read1(istream& is)   // 好
    {
        vector<string> res;
        for (string s; is >> s;)
            res.push_back(s);
        return res;
    }

与之近乎等价的更传统且更低层的代码，更长、更混乱、更难编写正确，而且很可能更慢：

    char** read2(istream& is, int maxelem, int maxstring, int* nread)   // 不好：啰嗦而且不完整
    {
        auto res = new char*[maxelem];
        int elemcount = 0;
        while (is && elemcount < maxelem) {
            auto s = new char[maxstring];
            is.read(s, maxstring);
            res[elemcount++] = s;
        }
        *nread = elemcount;
        return res;
    }

一旦添加了溢出检查和错误处理，代码就变得相当混乱了，而且还有个要记得 `delete` 其所返回的指针以及数组所包含的 C 风格字符串的问题。

##### 强制实施

并不容易。??? 寻找混乱的循环，嵌套的循环，长函数，函数调用的缺失，缺乏使用内建类型。圈复杂度？

### <a id="res-dry"></a>ES.3: 避免重复（DRY），避免冗余代码

重复或者冗余的代码会干扰编码意图，导致对逻辑的理解变得困难，并使维护变得困难，以及一些其他问题。它经常出现于拷贝粘贴式编程中。

只要合适就使用标准算法，而不是编写自己的实现。

**另请参见**: [SL.1](#rsl-lib), [ES.11](#res-auto)

##### 示例

    void func(bool flag)    // 不好，重复代码
    {
        if (flag) {
            x();
            y();
        }
        else {
            x();
            z();
        }
    }

    void func(bool flag)    // 更好，没有重复代码
    {
        x();

        if (flag)
            y();
        else
            z();
    }


##### 强制实施

* 采用静态分析器。它至少会找出一些重复的代码构造。
* 代码评审

## ES.dcl: 声明

声明也是语句。一条声明向一个作用域中引入一个名字，并可能导致对一个具名对象进行构造。

### <a id="res-scope"></a>ES.5: 保持作用域尽量小

##### 理由

可读性。最小化资源持有率。避免值的意外误用。

**其他形式**: 不要把名字在不必要的大作用域中进行声明。

##### 示例

    void use()
    {
        int i;    // 不好: 循环之后并不需要访问 i
        for (i = 0; i < 20; ++i) { /* ... */ }
        // 此处不存在对 i 的有意使用
        for (int i = 0; i < 20; ++i) { /* ... */ }  // 好: i 局部于 for 循环

        if (auto pc = dynamic_cast<Circle*>(ps)) {  // 好: pc 局部于 if 语句
            // ... 处理 Circle ...
        }
        else {
            // ... 处理错误 ...
        }
    }

##### 示例，不好

    void use(const string& name)
    {
        string fn = name + ".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        // ... 200 行代码，都不存在使用 fn 或 is 的意图 ...
    }

大多数测量都会称这个函数太长了，但其关键点是 `fn` 所使用的资源和 `is` 所持有的文件句柄
所持有的时间比其所需长太多了，而且可能在函数后面意外地出现对 `is` 和 `fn` 的使用。
这种情况下，把读取操作重构出来可能是一个好主意：

    Record load_record(const string& name)
    {
        string fn = name + ".txt";
        ifstream is {fn};
        Record r;
        is >> r;
        return r;
    }

    void use(const string& name)
    {
        Record r = load_record(name);
        // ... 200 行代码 ...
    }

##### 强制实施

* 对声明于循环之外，且在循环之后不再使用的循环变量进行标记。
* 当诸如文件句柄和锁这样的昂贵资源超过 N 行未被使用时进行标记（对某个适当的 N）。

### <a id="res-cond"></a>ES.6: 在 for 语句的初始化式和条件中声明名字以限制其作用域

##### 理由

可读性。
将循环变量的可见性限制到循环范围内。
避免在循环之后将循环变量用于其他目的。
最小化资源持有率。

##### 理由

    void use()
    {
        for (string s; cin >> s;)
            v.push_back(s);

        for (int i = 0; i < 20; ++i) {   // 好: i 局部于 for 循环
            // ...
        }

        if (auto pc = dynamic_cast<Circle*>(ps)) {   // 好: pc 局部于 if 语句
            // ... 处理 Circle ...
        }
        else {
            // ... 处理错误 ...
        }
    }

##### 示例，请勿如此

    int j;                            // 不好：j 在循环之外可见
    for (j = 0; j < 100; ++j) {
        // ...
    }
    // j 在此处仍可见但并不需要

**另请参见**: [不要用一个变量来达成两个不相关的目的](#res-recycle)

##### 强制实施

* 若在 `for` 语句中修改的变量是在循环之外声明的，但并未在循环之外使用，则给出警告。
* 【困难】 对声明与循环之前，且在循环之后用于某个无关用途的循环变量进行标记。

**讨论**：限制循环变量的作用域到循环体还相当有助于代码优化。认识到引入的变量仅在循环体中可以访问，
允许进行诸如外提（hoisting），强度消减，循环不变式代码外提等等优化。

##### C++17 和 C++20 示例

注：C++17 和 C++20 还增加了 `if`、`switch` 和范围式 `for` 的初始化式语句。以下代码要求支持 C++17 和 C++20。

    map<int, string> mymap;

    if (auto result = mymap.insert(value); result.second) {
        // 本代码块中，插入成功且 result 有效
        use(result.first);  // ok
        // ...
    } // result 在此处销毁

##### C++17 和 C++20 强制实施（当使用 C++17 或 C++20 编译器时）

* 选择/循环变量，若其在选择或循环体之前声明而在其之后不再使用，则对其进行标记
* 【困难】 选择/循环变量，若其在选择或循环体之前声明而在其之后用于某个无关目的，则对其进行标记

### <a id="res-name-length"></a>ES.7: 保持常用的和局部的名字尽量简短，而让非常用的和非局部的名字较长

##### 理由

可读性。减低在无关的非局部名字之间发生冲突的机会。

##### 示例

符合管理的简短的局部名字能够增强可读性：

    template<typename T>    // 好
    void print(ostream& os, const vector<T>& v)
    {
        for (gsl::index i = 0; i < v.size(); ++i)
            os << v[i] << '\n';
    }

索引根据惯例称为 `i`，而这个泛型函数中不存在有关这个 vector 的含义的提示，因此和其他名字一样， `v` 也没问题。与之相比，

    template<typename Element_type>   // 不好: 啰嗦，难于阅读
    void print(ostream& target_stream, const vector<Element_type>& current_vector)
    {
        for (gsl::index current_element_index = 0;
             current_element_index < current_vector.size();
             ++current_element_index
        )
        target_stream << current_vector[current_element_index] << '\n';
    }

当然，这是一个讽刺，但我们还见过更糟糕的。

##### 示例

不合惯例而简短的非局部名字则会搞乱代码：

    void use1(const string& s)
    {
        // ...
        tt(s);   // 不好: tt() 是什么？
        // ...
    }

更好的做法是，为非局部实体提供可读的名字：

    void use1(const string& s)
    {
        // ...
        trim_tail(s);   // 好多了
        // ...
    }

这样的话，有可能读者指导 `trim_tail` 是什么意思，而且读者可以找一下它并回忆起来。

##### 示例，不好

大型函数的参数的名字实际上可当作是非局部的，因此应当有意义：

    void complicated_algorithm(vector<Record>& vr, const vector<int>& vi, map<string, int>& out)
    // 根据 vi 中的索引，从 vr 中读取事件（并标记所用的 Record），
    // 向 out 中放置（名字，索引）对
    {
        // ... 500 行的代码，使用 vr，vi，和 out ...
    }

我们建议保持函数短小，但这条规则并不受到普遍坚持，而命名应当能反映这一点。

##### 强制实施

检查局部和非局部的名字的长度。同时考虑函数的长度。

### <a id="res-name-similar"></a>ES.8: 避免使用看起来相似的名字

##### 理由

代码清晰性和可读性。太过相似的名字会拖慢理解过程并增加出错的可能性。

##### 示例，不好

    if (readable(i1 + l1 + ol + o1 + o0 + ol + o1 + I0 + l0)) surprise();

##### 示例，不好

不要在同一个作用域中用和类型相同的名字声明一个非类型实体。这样做将消除为区分它们所需的关键字 `struct` 或 `enum` 等。这同样消除了一种错误来源，因为 `struct X` 在对 `X` 的查找失败时会隐含地声明新的 `X`。

    struct foo { int n; };
    struct foo foo();       // 不好, foo 在作用域中已经是一个类型了
    struct foo x = foo();   // 需要进行区分

##### 例外

很古老的头文件中可能会用在相同作用域中用同一个名字同时声明非类型实体和类型。

##### 强制实施

* 用一个已知的易混淆字母和数字组合的列表来对名字进行检查。
* 当变量、函数或枚举符的声明隐藏了在相同作用域中所声明的类或枚举时，给出警告。

### <a id="res-not-caps"></a>ES.9: 避免 `ALL_CAPS` 式的名字

##### 理由

这样的名字通常是用于宏的。因此，`ALL_CAPS` 这样的名字可能遭受意外的宏替换。

##### 示例

    // 某个头文件中：
    #define NE !=

    // 某个别的头文件中的别处：
    enum Coord { N, NE, NW, S, SE, SW, E, W };

    // 某个糟糕程序员的 .cpp 中的第三处：
    switch (direction) {
    case N:
        // ...
    case NE:
        // ...
    // ...
    }

##### 注解

不要仅仅因为常量曾经是宏，就使用 `ALL_CAPS` 作为常量。

##### 强制实施

对所有的 ALL CAPS 进行标记。对于老代码，则接受宏名字的 ALL CAPS 而标记所有的非 ALL-CAPS 的宏名字。

### <a id="res-name-one"></a>ES.10: 每条声明中（仅）声明一个名字

##### 理由

每行一条声明的做法增加可读性并可避免与 C++ 的文法
相关的错误。这样做也为更具描述性的行尾注释留下了
空间。

##### 示例，不好

    char *p, c, a[7], *pp[7], **aa[10];   // 讨厌！

##### 例外

函数声明中可以包含多个函数参数声明。

##### 例外

结构化绑定（C++17）就是专门设计用于引入多个变量的：

    auto [iter, inserted] = m.insert_or_assign(k, val);
    if (inserted) { /* 已插入新条目 */ }

##### 示例

    template<class InputIterator, class Predicate>
    bool any_of(InputIterator first, InputIterator last, Predicate pred);

用 concept 则更佳：

    bool any_of(input_iterator auto first, input_iterator auto last, predicate auto pred);

##### 示例

    double scalbn(double x, int n);   // OK: x * pow(FLT_RADIX, n); FLT_RADIX 通常为 2

或者：

    double scalbn(    // 有改善: x * pow(FLT_RADIX, n); FLT_RADIX 通常为 2
        double x,     // 基数
        int n         // 指数
    );

或者：

    // 有改善: base * pow(FLT_RADIX, exponent); FLT_RADIX 通常为 2
    double scalbn(double base, int exponent);

##### 示例

    int a = 10, b = 11, c = 12, d, e = 14, f = 15;

在较长的声明符列表中，很容易忽视某个未能初始化的变量。

##### 强制实施

对具有多个声明符的变量或常量的声明式（比如 `int* p, q;`）进行标记。

### <a id="res-auto"></a>ES.11: 使用 `auto` 来避免类型名字的多余重复

##### 理由

* 单纯的重复既麻烦又易错。
* 当使用 `auto` 时，所声明的实体的名字是处于声明的固定位置的，这增加了可读性。
* 函数模板声明的返回类型可能是某个成员类型。

##### 示例

考虑：

    auto p = v.begin();      // vector<DataRecord>::iterator
    auto z1 = v[3];          // 产生 DataRecord 的副本
    auto& z2 = v[3];         // 避免复制
    const auto& z3 = v[3];   // const 并避免复制
    auto h = t.future();
    auto q = make_unique<int[]>(s);
    auto f = [](int x) { return x + 10; };

以上都避免了写下冗长又难记的类型，它们是编译器已知的，但程序员则可能会搞错。

##### 示例

    template<class T>
    auto Container<T>::first() -> Iterator;   // Container<T>::Iterator

##### 例外

当使用初始化式列表，而所需要的确切类型是已知的，同时某个初始化式可能需要转换时，应当避免使用 `auto`。

##### 示例

    auto lst = { 1, 2, 3 };   // lst 是一个 initializer_list
    auto x{1};   // x 是一个 int（C++17；在 C++11 中则为 initializer_list）

##### 注解

C++20 的情况是，我们可以（而且应该）使用概念来更加明确地说明所推断的类型：

    // ...
    forward_iterator auto p = algo(x, y, z);

##### 示例（C++17）

    std::set<int> values;
    // ...
    auto [ position, newly_inserted ] = values.insert(5);   // 展开 std::pair 的成员

##### 强制实施

对声明中多余的类型名字进行标记。

### <a id="res-reuse"></a>ES.12: 不要在嵌套作用域中重用名字

##### 理由

这样很容易把所用的是哪个变量搞混。
会造成维护问题。

##### 示例，不好

    int d = 0;
    // ...
    if (cond) {
        // ...
        d = 9;
        // ...
    }
    else {
        // ...
        int d = 7;
        // ...
        d = value_to_be_returned;
        // ...
    }

    return d;

这是个大型的 `if` 语句，很容易忽视在内部作用域中所引入的新的 `d`。
这是一种已知的 BUG 来源。
这种在内部作用域中的名字重用有时候被称为“遮蔽“。

##### 注解

当函数变得过大和过于复杂时，遮蔽是一种主要的问题。

##### 示例

语言不允许在函数的最外层块中遮蔽函数参数：

    void f(int x)
    {
        int x = 4;  // 错误：重用函数参数的名字

        if (x) {
            int x = 7;  // 允许，但不好
            // ...
        }
    }

##### 示例，不好

把成员名重用为局部变量也会造成问题：

    struct S {
        int m;
        void f(int x);
    };

    void S::f(int x)
    {
        m = 7;    // 对成员赋值
        if (x) {
            int m = 9;
            // ...
            m = 99; // 对局部变量赋值
            // ...
        }
    }

##### 例外

我们经常在派生类中重用基类中的函数名：

    struct B {
        void f(int);
    };

    struct D : B {
        void f(double);
        using B::f;
    };

这样做是易错的。
例如，要是忘了 using 声明式的话，`d.f(1)` 的调用就不会找到 `int` 版本的 `f`。

??? 我们需要为类层次中的遮蔽/隐藏给出专门的规则吗？

##### 强制实施

* 对嵌套局部作用域中的名字重用进行标记。
* 对成员函数中将成员名重用为局部变量进行标记。
* 对把全局名字重用为局部变量或成员的名字进行标记。
* 对在派生类中重用（除函数名之外的）基类成员名进行标记。

### <a id="res-always"></a>ES.20: 坚持为对象进行初始化

##### 理由

避免发生“设值前使用”的错误及其所关联的未定义行为。
避免由复杂的初始化的理解所带来的问题。
简化重构。

##### 示例

    void use(int arg)
    {
        int i;   // 不好: 未初始化的变量
        // ...
        i = 7;   // 初始化 i
    }

错了，`i = 7` 并不是 `i` 的初始化；它是向其赋值。而且 `i` 也可能在 `...` 的部分中被读取。更好的做法是：

    void use(int arg)   // OK
    {
        int i = 7;   // OK: 初始化
        string s;    // OK: 默认初始化
        // ...
    }

##### 注释

我们有意让*总是进行初始化*规则比*对象在使用前必须设值*的语言规则更强。
后者是较为宽松的规则，虽然能够识别出技术上的 BUG，不过：

* 它会导致较不可读的代码，
* 它鼓励人们在比所需的更大的作用域中声明名字，
* 它会导致较难于阅读的代码，
* 它会因为鼓励复杂的代码而导致出现逻辑 BUG，
* 它会妨碍进行重构。

而*总是进行初始化*规则则是以提升可维护性为目标的一条风格规则，同样也是保护避免出现“设值前使用”错误的规则。

##### 示例

这个例子经常被当成是用来展示需要更宽松的初始化规则的例子。

    widget i;    // "widget" 是一个初始化操作昂贵的类型，可能是一种大型 POD
    widget j;

    if (cond) {  // 不好: i 和 j 进行了“延迟”初始化
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }

这段代码是无法简单重写为用初始化式来对 `i` 和 `j` 进行初始化的。
注意，对于带有默认构造函数的类型来说，试图延后初始化只会导致变为一次默认初始化之后跟着一次赋值的做法。
这种例子的一种更加流行的理由是“效率”，不过可以检查出是否出现“设置前使用”错误的编译器，同样可以消除任何多余的双重初始化。

假定 `i` 和 `j` 之间存在某种逻辑关联，则这种关联可能应当在函数中予以表达：

    pair<widget, widget> make_related_widgets(bool x)
    {
        return (x) ? {f1(), f2()} : {f3(), f4()};
    }

    auto [i, j] = make_related_widgets(cond);    // C++17

如果除此之外 `make_related_widgets` 函数是多余的，
可以使用 lambda [ES.28](#res-lambda-init) 来消除之：

    auto [i, j] = [x] { return (x) ? pair{f1(), f2()} : pair{f3(), f4()} }();    // C++17

用一个值代表 `uninitialized` 只是一种问题的症状，而不是一种解决方案：

    widget i = uninit;  // 不好
    widget j = uninit;

    // ...
    use(i);         // 可能发生设值前使用
    // ...

    if (cond) {     // 不好: i 和 j 进行了“延迟”初始化
        i = f1();
        j = f2();
    }
    else {
        i = f3();
        j = f4();
    }

这样的话编译器甚至无法再简单地检测出“设值前使用”。而且我们也在 widget 的状态空间中引入了复杂性：哪些操作对 `uninit` 的 widget 是有效的，哪些不是？

##### 注解

几十年来，精明的程序员中都流行进行复杂的初始化。
这样做也是一种错误和复杂性的主要来源。
而许多这样的错误都是在最初实现之后的多年之后的维护过程中所引入的。

##### 示例

本条规则涵盖成员变量。

    class X {
    public:
        X(int i, int ci) : m2{i}, cm2{ci} {}
        // ...

    private:
        int m1 = 7;
        int m2;
        int m3;

        const int cm1 = 7;
        const int cm2;
        const int cm3;
    };

编译器能够标记 `cm3` 为未初始化（因其为 `const`），但它无法发觉 `m3` 缺少初始化。
通常来说，以很少不恰当的成员初始化来消除错误，要比缺乏初始化更有价值，
而且优化器是可以消除冗余的初始化的（比如紧跟在赋值之前的初始化）。

##### 例外

当声明一个即将从输入进行初始化的对象时，其初始化就可能导致发生双重初始化。
不过，应当注意这也可能造成输入之后留下未初始化的数据——而这已经是一种错误和安全攻击的重大来源：

    constexpr int max = 8 * 1024;
    int buf[max];         // OK, 但是可疑: 未初始化
    f.read(buf, max);

由于数组和 `std::array` 的有所限制的初始化规则，它们提供了对于需要这种例外的大多数有意义的例子。

某些情况下，这个数组进行初始化的成本可能是显著的。
但是，这样的例子确实倾向于留下可访问到的未初始化变量，因而应当严肃对待它们。

    constexpr int max = 8 * 1024;
    int buf[max] = {};   // 某些情况下更好
    f.read(buf, max);

如果可行的话，应当用某个已知不会溢出的库函数。例如：

    string s;   // s 默认初始化为 ""
    cin >> s;   // s 进行扩充以持有字符串

不要把用于输入操作的简单变量作为本条规则的例外：

    int i;   // 不好
    // ...
    cin >> i;

在并不罕见的情况下，当输入目标和输入操作分开（其实不应该）时，就带来了发生“设值前使用”的可能性。

    int i2 = 0;   // 更好，假设 0 是 i2 可接受的值
    // ...
    cin >> i2;

优秀的优化器应当能够识别输入操作并消除这种多余的操作。


##### 注解

有时候，可以用 lambda 作为初始化式以避免未初始化变量：

    error_code ec;
    Value v = [&] {
        auto p = get_value();   // get_value() 返回 pair<error_code, Value>
        ec = p.first;
        return p.second;
    }();

还可以是：

    Value v = [] {
        auto p = get_value();   // get_value() 返回 pair<error_code, Value>
        if (p.first) throw Bad_value{p.first};
        return p.second;
    }();

**参见**: [ES.28](#res-lambda-init)

##### 强制实施

* 标记出每个未初始化的变量。
  不要对具有默认构造函数的自定义类型的变量进行标记。
* 检查未初始化的缓冲区是否在声明后*立即*进行了写入。
  将未初始化变量作为一个非 `const` 的引用参数进行传递可以被假定为向变量进行的写入。

### <a id="res-introduce"></a>ES.21: 不要在确实需要使用变量（或常量）之前就引入它

##### 理由

可读性。限制变量可以被使用的范围。

##### 示例

    int x = 7;
    // ... 这里没有对 x 的使用 ...
    ++x;

##### 强制实施

对离其首次使用很远的声明进行标记。

### <a id="res-init"></a>ES.22: 要等到获得了用以初始化变量的值之后才声明变量

##### 理由

可读性。限制变量可以被使用的范围。避免“设值前使用”的风险。初始化通常比赋值更加高效。

##### 示例，不好

    string s;
    // ... 此处没有 s 的使用 ...
    s = "what a waste";

##### 示例，不好

    SomeLargeType var;  // 很难读的驼峰变量

    if (cond)   // 某个不简单的条件
        Set(&var);
    else if (cond2 || !cond3) {
        var = Set2(3.14);
    }
    else {
        var = 0;
        for (auto& e : something)
            var += e;
    }

    // 使用 var; 可以仅通过控制流而静态地保证这并不会过早进行

如果 `SomeLargeType` 的默认初始化并非过于昂贵的话就没什么问题。
不过，程序员可能十分想知道是否所有的穿过这个条件迷宫的路径都已经被覆盖到了。
如果没有的话，就存在一个“设值前使用”的 BUG。这是维护工作的一个陷阱。

对于具有必要复杂性的初始化式，也包括 `const` 变量的初始化式，应当考虑使用 lambda 来表达它；参见 [ES.28](#res-lambda-init)。

##### 强制实施

* 如果具有默认初始化的声明在其首次被读取前就进行赋值，则对其进行标记。
* 对于任何在未初始化变量之后且在其使用之前进行的复杂计算进行标记。

### <a id="res-list"></a>ES.23: 优先使用 `{}` 初始化式语法

##### 理由

优先使用 `{}`。`{}` 初始化的规则比其他形式的初始化更简单，更通用，更少歧义，而且更安全。

仅当你确定不存在窄化转换时才可使用 `=`。对于内建算术类型，`=` 仅和 `auto` 一起使用。

避免 `()` 初始化，它会导致解析中的歧义。

##### 示例

    int x {f(99)};
    int y = x;
    vector<int> v = {1, 2, 3, 4, 5, 6};

##### 例外

对于容器来说，存在用 `{...}` 给出元素列表而用 `(...)` 给出大小的传统做法：

    vector<int> v1(10);    // vector 有 10 个具有默认值 0 的元素
    vector<int> v2{10};    // vector 有 1 个值为 10 的元素

    vector<int> v3(1, 2);  // vector 有 1 个值为 2 的元素
    vector<int> v4{1, 2};  // vector 有 2 个值为 1 和 2 的元素

##### 注解

`{}` 初始化式不允许进行窄化转换（这点通常都很不错），并允许使用显式构造函数（这没有问题，我们的意图就是初始化一个新变量）。

##### 示例

    int x {7.9};   // 错误: 发生窄化
    int y = 7.9;   // OK: y 变为 7. 希望编译器给出了警告消息
    int z = gsl::narrow_cast<int>(7.9);  // OK: 这个正是你想要的

##### 注解

`{}` 初始化可以用于几乎所有的初始化；而其他的初始化则不行：

    auto p = new vector<int> {1, 2, 3, 4, 5};   // 初始化 vector
    D::D(int a, int b) :m{a, b} {   // 成员初始化式（比如说 m 可能是 pair）
        // ...
    };
    X var {};   // 初始化 var 为空
    struct S {
        int m {7};   // 成员的默认初始化
        // ...
    };

由于这个原因，以 `{}` 进行初始化通常被称为“统一初始化”，
（但很可惜存在少数不符合规则的例外）。

##### 注解

对以 `auto` 声明的变量用单个值进行的初始化，比如 `{v}`，直到 C++17 之前都还具有令人意外的含义。
C++17 的规则多少会少些意外：

    auto x1 {7};        // x1 是一个值为 7 的 int
    auto x2 = {7};      // x2 是一个具有一个元素 7 的 initializer_list<int>

    auto x11 {7, 8};    // 错误: 两个初始化式
    auto x22 = {7, 8};  // x22 是一个具有元素 7 和 8 的 initializer_list<int>

如果确实需要一个 `initializer_list<T>` 的话，可以使用 `={...}`：

    auto fib10 = {1, 1, 2, 3, 5, 8, 13, 21, 34, 55};   // fib10 是一个列表

##### 注解

`={}` 进行的是复制初始化，而 `{}` 则进行直接初始化。
与在复制初始化和直接初始化自身之间存在的区别类似的是，这里也可能带来一些意外情况。
`{}` 可以接受 `explicit` 构造函数；而 `={}` 则不能。例如：

    struct Z { explicit Z() {} };

    Z z1{};     // OK: 直接初始化，使用的是 explicit 构造函数
    Z z2 = {};  // 错误: 复制初始化，不能使用 explicit 构造函数

除非特别要求禁止使用显式构造函数，否则都应当使用普通的 `{}` 初始化。

##### 示例

    template<typename T>
    void f()
    {
        T x1(1);    // T 以 1 进行初始化
        T x0();     // 不好: 函数声明（一般都是一个错误）

        T y1 {1};   // T 以 1 进行初始化
        T y0 {};    // 默认初始化 T
        // ...
    }

**参见**: [讨论](#???)

##### 强制实施

* 当使用 `=` 初始化算术类型并发生窄化转换时予以标记。
* 当使用 `()` 初始化语法但实际上是声明式时予以标记。（许多编译器已经可就此给出警告。）

### <a id="res-unique"></a>ES.24: 用 `unique_ptr<T>` 来保存指针

##### 理由

使用 `std::unique_ptr` 是避免泄漏的最简单方法。它是可靠的，它
利用类型系统完成验证所有权安全性的大部分工作，它
增加可读性，而且它没有或近乎没有运行时成本。

##### 示例

    void use(bool leak)
    {
        auto p1 = make_unique<int>(7);   // OK
        int* p2 = new int{7};            // 不好: 可能泄漏
        // ... 未对 p2 赋值 ...
        if (leak) return;
        // ... 未对 p2 赋值 ...
        vector<int> v(7);
        v.at(7) = 0;                    // 抛出异常
        delete p2;                      // 避免泄漏已太晚了
        // ...
    }

当 `leak == true` 时，`p2` 所指向的对象就会泄漏，而 `p1` 所指向的对象则不会。
当 `at()` 抛出异常时也是同样的情况。两种情况下，都没能到达 `delete p2` 语句。

##### 强制实施

寻找作为这些函数的目标的原生指针：`new`，`malloc()`，或者可能返回这类指针的函数。

### <a id="res-const"></a>ES.25: 应当将对象声明为 `const` 或 `constexpr`，除非后面需要修改其值

##### 理由

这样的话你就不会误改掉这个值。而且这种方式可能会给编译器的带来优化机会。

##### 示例

    void f(int n)
    {
        const int bufmax = 2 * n + 2;  // 好: 无法意外改掉 bufmax
        int xmax = n;                  // 可疑: xmax 是不是会改掉？
        // ...
    }

##### 强制实施

查看变量是不是真的被改动过，若并非如此就进行标记。
不幸的是，也许不可能检测出某个非 `const` 是不是
*有意*要改动，还是仅仅是没被改动而已。

### <a id="res-recycle"></a>ES.26: 不要用一个变量来达成两个不相关的目的

##### 理由

可读性和安全性。

##### 示例，不好

    void use()
    {
        int i;
        for (i = 0; i < 20; ++i) { /* ... */ }
        for (i = 0; i < 200; ++i) { /* ... */ } // 不好: i 重复使用了
    }

+##### 注解

也许你想把一个缓冲区当做暂存器来重复使用以作为一种优化措施，但即便如此也请尽可能限定该变量的作用域，还要当心不要导致由于遗留在重用的缓冲区中的数据而引发的 BUG，这是安全性 BUG 的一种常见来源。

    void write_to_file()
    {
        std::string buffer;             // 以避免每次循环重复中的重新分配
        for (auto& o : objects) {
            // 第一部分工作。
            generate_first_string(buffer, o);
            write_to_file(buffer);

            // 第二部分工作。
            generate_second_string(buffer, o);
            write_to_file(buffer);

            // 等等...
        }
    }

##### 强制实施

标记被重复使用的变量。

### <a id="res-stack"></a>ES.27: 使用 `std::array` 或 `stack_array` 作为栈上的数组

##### 理由

它们是可读的，而且不会隐式转换为指针。
它们不会和内建数组的非标准扩展相混淆。

##### 示例，不好

    const int n = 7;
    int m = 9;

    void f()
    {
        int a1[n];
        int a2[m];   // 错误: 并非 ISO C++
        // ...
    }

##### 注解

`a1` 的定义是合法的 C++ 而且一直都是。
存在大量的这类代码。
不过它是易错的，尤其当它的界并非局部时更是如此。
而且它也是一种“流行”的错误来源（缓冲区溢出，数组退化而成的指针，等等）。
而 `a2` 的定义符合 C 但不符合 C++，而且被认为存在安全性风险。

##### 示例

    const int n = 7;
    int m = 9;

    void f()
    {
        array<int, n> a1;
        stack_array<int> a2(m);
        // ...
    }

##### 强制实施

* 对具有非常量界的数组（C 风格的 VLA）作出标记。
* 对具有非局部的常量界的数组作出标记。

### <a id="res-lambda-init"></a>ES.28: 为复杂的初始化（尤其是 `const` 变量）使用 lambda

##### 理由

它可以很好地封装局部的初始化，包括对仅为初始化所需的临时变量进行清理，而且避免了创建不必要的非局部而且无法重用的函数。它对于应当为 `const` 的变量也可以工作，不过必须先进行一些初始化。

##### 示例，不好

    widget x;   // 应当为 const, 不过:
    for (auto i = 2; i <= N; ++i) {          // 这是由 x 的
        x += some_obj.do_something_with(i);  // 初始化所需的
    }                                        // 一段任意长的代码
    // 自此开始，x 应当为 const，不过我们无法在这种风格的代码中做到这点

##### 示例，好

    const widget x = [&] {
        widget val;                                // 假定 widget 具有默认构造函数
        for (auto i = 2; i <= N; ++i) {            // 这是由 x 的
            val += some_obj.do_something_with(i);  // 初始化所需的
        }                                          // 一段任意长的代码
        return val;
    }();

如果可能的话，应当将条件缩减成一个后续的简单集合（比如一个 `enum`），并避免把选择和初始化相互混合起来。

##### 强制实施

很难。最多是某种启发式方案。查找跟随某个未初始化变量之后的循环中向其赋值。

### <a id="res-macros"></a>ES.30: 不要用宏来操纵程序文本

##### 理由

宏是 BUG 的一个主要来源。
宏不遵守常规的作用域和类型规则。
宏保证会让人读到的东西和编译器见到的东西不一样。
宏使得工具的建造复杂化。

##### 示例，不好

    #define Case break; case   /* 不好 */

这个貌似无害的宏会把某个大写的 `C` 替换为小写的 `c` 导致一个严重的控制流错误。

##### 注解

这条规则并不禁止在 `#ifdef` 等部分中使用用于“配置控制”的宏。

将来，模块可能会消除配置控制中对宏的需求。

##### 注解

此规则也意味着不鼓励使用 `#` 进行字符串化和使用 `##` 进行连接。
照例，宏有一些“无害”的用途，但即使这些也会给工具带来麻烦，
例如自动完成器、静态分析器和调试器。
通常，使用花式宏的欲望是过于复杂的设计的标志。
另外，`＃` 和 `##` 促进了宏的定义和使用：

    #define CAT(a, b) a ## b
    #define STRINGIFY(a) #a

    void f(int x, int y)
    {
        string CAT(x, y) = "asdf";   // 不好: 工具难以处理（也很丑陋）
        string sx2 = STRINGIFY(x);
        // ...
    }

有使用宏进行低级字符串操作的变通方法。例如：

    string s = "asdf" "lkjh";   // 普通的字符串文字连接

    enum E { a, b };

    template<int x>
    constexpr const char* stringify()
    {
        switch (x) {
        case a: return "a";
        case b: return "b";
        }
    }

    void f(int x, int y)
    {
        string sx = stringify<x>();
        // ...
    }

这不像定义宏那样方便，但是易于使用、零开销，并且是类型化的和作用域化的。

将来，静态反射可能会消除对程序文本操作的预处理器的最终需求。

##### 强制实施

见到并非仅用于源代码控制（比如 `#ifdef`）的宏时应当大声尖叫。

### <a id="res-macros2"></a>ES.31: 不要用宏来作为常量或“函数”

##### 理由

宏是 BUG 的一个主要来源。
宏不遵守常规的作用域和类型规则。
宏不遵守常规的参数传递规则。
宏保证会让人读到的东西和编译器见到的东西不一样。
宏使得工具的建造复杂化。

##### 示例，不好

    #define PI 3.14
    #define SQUARE(a, b) (a * b)

即便我们并未在 `SQUARE` 中留下这个众所周知的 BUG，也存在多种表现好得多的替代方式；比如：

    constexpr double pi = 3.14;
    template<typename T> T square(T a, T b) { return a * b; }

##### 强制实施

见到并非仅用于源代码控制（比如 `#ifdef`）的宏时应当大声尖叫。

### <a id="res-all_caps"></a>ES.32: 对所有的宏名采用 `ALL_CAPS` 命名方式

##### 理由

遵循约定。可读性。区分宏。

##### 示例

    #define forever for (;;)   /* 非常不好 */

    #define FOREVER for (;;)   /* 仍然很邪恶，但至少对人来说是可见的 */

##### 强制实施

见到小写的宏时应当大声尖叫。

### <a id="res-macros"></a>ES.33: 如果必须使用宏的话，请为之提供唯一的名字

##### 理由

宏并不遵守作用域规则。

##### 示例

    #define MYCHAR        /* 不好，最终将会和别人的 MYCHAR 相冲突 */

    #define ZCORP_CHAR    /* 还是不好，但冲突的机会较小 */

##### 注解

如果可能就应当避免使用宏：[ES.30](#res-macros)，[ES.31](#res-macros2)，以及 [ES.32](#res-all_caps)。
然而，存在亿万行的代码中包含宏，以及一种使用并过度使用宏的长期传统。
如果你被迫使用宏的话，请使用长名字，而且应当带有唯一前缀（比如你的组织机构的名字）以减少冲突的可能性。

##### 强制实施

对较短的宏名给出警告。

### <a id="res-ellipses"></a> ES.34: 不要定义（C 风格的）变参函数

##### 理由

它并非类型安全。
而且需要杂乱的满是强制转换和宏的代码才能正确工作。

##### 示例

    #include <cstdarg>

    // "severity" 后面跟着以零终结的 char* 列表；将 C 风格字符串写入 cerr
    void error(int severity ...)
    {
        va_list ap;             // 一个持有参数的神奇类型
        va_start(ap, severity); // 参数启动："severity" 是 error() 的第一个参数

        for (;;) {
            // 将下一个变量看作 char*；没有检查：经过伪装的强制转换
            char* p = va_arg(ap, char*);
            if (!p) break;
            cerr << p << ' ';
        }

        va_end(ap);             // 参数清理（不能忘了这个）

        cerr << '\n';
        if (severity) exit(severity);
    }

    void use()
    {
        error(7, "this", "is", "an", "error", nullptr);
        error(7); // 崩溃
        error(7, "this", "is", "an", "error");  // 崩溃
        const char* is = "is";
        string an = "an";
        error(7, "this", "is", an, "error"); // 崩溃
    }

**替代方案**: 重载。模板。变参模板。

    #include <iostream>

    void error(int severity)
    {
        std::cerr << '\n';
        std::exit(severity);
    }

    template<typename T, typename... Ts>
    constexpr void error(int severity, T head, Ts... tail)
    {
        std::cerr << head;
        error(severity, tail...);
    }

    void use()
    {
        error(7); // 不会崩溃！
        error(5, "this", "is", "not", "an", "error"); // 不会崩溃！

        std::string an = "an";
        error(7, "this", "is", "not", an, "error"); // 不会崩溃！

        error(5, "oh", "no", nullptr); // 编译器报错！不需要 nullptr。
    }


##### 注解

这基本上就是 `printf` 的实现方式。

##### 强制实施

* 对 C 风格的变参函数的定义作出标记。
* 对 `#include <cstdarg>` 和 `#include <stdarg.h>` 作出标记。


## ES.expr: 表达式

表达式对值进行操作。

### <a id="res-complicated"></a>ES.40: 避免复杂的表达式

##### 理由

复杂的表达式是易错的。

##### 示例

    // 不好: 在子表达式中藏有赋值
    while ((c = getc()) != -1)

    // 不好: 在一个子表达式中对两个非局部变量进行了赋值
    while ((cin >> c1, cin >> c2), c1 == c2)

    // 有改善，但可能仍然过于复杂
    for (char c1, c2; cin >> c1 >> c2 && c1 == c2;)

    // OK: 若 i 和 j 并非别名
    int x = ++i + ++j;

    // OK: 若 i != j 且 i != k
    v[i] = v[j] + v[k];

    // 不好: 子表达式中“隐藏”了多个赋值
    x = a + (b = f()) + (c = g()) * 7;

    // 不好: 依赖于经常被误解的优先级规则
    x = a & b + c * d && e ^ f == 7;

    // 不好: 未定义行为
    x = x++ + x++ + ++x;

这些表达式中有几个是无条件不好的（比如说依赖于未定义行为）。其他的只不过过于复杂和不常见，即便是优秀的程序员匆忙中也可能会误解或者忽略其中的某个问题。

##### 注解

C++17 收紧了有关求值顺序的规则
（除了赋值中从右向左，以及函数实参求值顺序未指明外均为从左向右，[参见 ES.43](#res-order)），
但这并不影响复杂表达式很容易引起混乱的事实。

##### 注解

程序员应当了解并运用表达式的基本规则。

##### 示例

    x = k * y + z;             // OK

    auto t1 = k * y;           // 不好: 不必要的啰嗦
    x = t1 + z;

    if (0 <= x && x < max)   // OK

    auto t1 = 0 <= x;        // 不好: 不必要的啰嗦
    auto t2 = x < max;
    if (t1 && t2)            // ...

##### 强制实施

很麻烦。多复杂的表达式才能被当成复杂的？把计算写成每条语句一个操作同样是让人混乱的。需要考虑的有：

* 副作用：（对于某种非局部性定义，）多个非局部变量上发生副作用是值得怀疑的，尤其是当这些副作用是在不同的子表达式中时
* 向别名变量的写入
* 超过 N 个运算符（N 应当为多少？）
* 依赖于微妙的优先级规则
* 使用了未定义行为（我们是否应当识别所有的未定义行为？）
* 实现定义的行为？
* ???

### <a id="res-parens"></a>ES.41: 对运算符优先级不保准时应使用括号

##### 理由

避免错误。可读性。不是每个人都能记住运算符表格。

##### 示例

    const unsigned int flag = 2;
    unsigned int a = flag;

    if (a & flag != 0)  // 不好: 含义为 a&(flag != 0)

注意：我们建议程序员了解算术运算和逻辑运算的优先级表，但应当考虑当按位逻辑运算和其他运算符混合使用时需要采用括号。

    if ((a & flag) != 0)  // OK: 按预期工作

##### 注解

你应当了解足够的知识以避免在这样的情况下需要括号：

    if (a < 0 || a <= max) {
        // ...
    }

##### 强制实施

* 当按位逻辑运算符合其他运算符组合时进行标记。
* 当赋值运算符不是最左边的运算符时进行标记。
* ???

### <a id="res-ptr"></a>ES.42: 保持单纯直接的指针使用方式

##### 理由

复杂的指针操作是一种重大的错误来源。

##### 注解

代之以使用 `gsl::span`。
指针[只应当指代单个对象](#ri-array)。
指针算术是脆弱而易错的，是许多许多糟糕的 BUG 和安全漏洞的来源。
`span` 是一种用于访问数组对象的带有边界检查的安全类型。
以常量为下标来访问已知边界的数组，编译器可以进行验证。

##### 示例，不好

    void f(int* p, int count)
    {
        if (count < 2) return;

        int* q = p + 1;    // 不好

        ptrdiff_t d;
        int n;
        d = (p - &n);      // OK
        d = (q - p);       // OK

        int n = *p++;      // 不好

        if (count < 6) return;

        p[4] = 1;          // 不好

        p[count - 1] = 2;  // 不好

        use(&p[0], 3);     // 不好
    }

##### 示例，好

    void f(span<int> a) // 好多了：函数声明中使用了 span
    {
        if (a.size() < 2) return;

        int n = a[0];      // OK

        span<int> q = a.subspan(1); // OK

        if (a.size() < 6) return;

        a[4] = 1;          // OK

        a[a.size() - 1] = 2;  // OK

        use(a.data(), 3);  // OK
    }

##### 注解

用变量做下标，对于工具和人类来说都是很难将其验证为安全的。
`span` 是一种用于访问数组对象的带有运行时边界检查的安全类型。
`at()` 是可以保证单次访问进行边界检查的另一种替代方案。
如果需要用迭代器来访问数组的话，应使用构造于数组之上的 `span` 所提供的迭代器。

##### 示例，不好

    void f(array<int, 10> a, int pos)
    {
        a[pos / 2] = 1; // 不好
        a[pos - 1] = 2; // 不好
        a[-1] = 3;    // 不好（但易于被工具查出） - 没有替代方案，请勿这样做
        a[10] = 4;    // 不好（但易于被工具查出） - 没有替代方案，请勿这样做
    }

##### 示例，好

使用 `span`：

    void f1(span<int, 10> a, int pos) // A1: 将参数类型改为使用 span
    {
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }

    void f2(array<int, 10> arr, int pos) // A2: 增加局部的 span 并使用之
    {
        span<int> a = {arr.data(), pos};
        a[pos / 2] = 1; // OK
        a[pos - 1] = 2; // OK
    }

使用 `at()`：

    void f3(array<int, 10> a, int pos) // 替代方案 B: 用 at() 进行访问
    {
        at(a, pos / 2) = 1; // OK
        at(a, pos - 1) = 2; // OK
    }

##### 示例，不好

    void f()
    {
        int arr[COUNT];
        for (int i = 0; i < COUNT; ++i)
            arr[i] = i; // 不好，不能使用非常量索引
    }

##### 示例，好

使用 `span`：

    void f1()
    {
        int arr[COUNT];
        span<int> av = arr;
        for (int i = 0; i < COUNT; ++i)
            av[i] = i;
    }

使用 `span` 和基于范围的 `for`：

    void f1a()
    {
         int arr[COUNT];
         span<int, COUNT> av = arr;
         int i = 0;
         for (auto& e : av)
             e = i++;
    }

使用 `at()` 进行访问：

    void f2()
    {
        int arr[COUNT];
        int i = 0;
        for (int i = 0; i < COUNT; ++i)
            at(arr, i) = i;
    }

使用基于范围的 `for`：

    void f3()
    {
         int arr[COUNT];
         for (auto& e : arr)
             e = i++;
    }

##### 注解

工具可以提供重写能力，以将涉及动态索引表达式的数组访问替换为使用 `at()` 进行访问：

    static int a[10];

    void f(int i, int j)
    {
        a[i + j] = 12;      // 不好，可以重写为 ...
        at(a, i + j) = 12;  // OK - 带有边界检查
    }

##### 示例

把数组转变为指针（语言基本上总会这样做），移除了进行检查的机会，因此应当予以避免

    void g(int* p);

    void f()
    {
        int a[5];
        g(a);        // 不好：是要传递一个数组吗？
        g(&a[0]);    // OK：传递单个对象
    }

如果要传递数组的话，应该这样：

    void g(int* p, size_t length);  // 老的（危险）代码

    void g1(span<int> av); // 好多了：改动了 g()。

    void f()
    {
        int a[5];
        span<int> av = a;

        g(av.data(), av.size());   // OK, 如果没有其他选择的话
        g1(a);                     // OK - 这里没有退化，而是使用了隐式的 span 构造函数
    }

##### 强制实施

* 对任何在指针类型的表达式上进行的产生指针类型的值的算术运算进行标记。
* 对任何数组类型的表达式或变量（无论是静态数组还是 `std::array`）上进行索引的表达式，若其索引不是值为从 `0` 到数组上界之内的编译期常量表达式，则进行标记。
* 对任何可能依赖于从数组类型向指针类型的隐式转换的表达式进行标记。

本条规则属于[边界安全性剖面配置](#ss-bounds)。


### <a id="res-order"></a>ES.43: 避免带有未定义的求值顺序的表达式

##### 理由

你没办法搞清楚这种代码会做什么。可移植性。
即便它做到了对你可能有意义的事情，它在别的编译器（比如你的编译器的下个版本）或者不同的优化设置中也可能会做出不同的事情。

##### 注解

C++17 收紧了有关求值顺序的规则：
除了赋值中从右向左，以及函数实参求值顺序未指明外均为从左向右。

不过，要记住你的代码可能是由 C++17 之前的编译器进行编译的（比如通过复制粘贴），请勿自作聪明。

##### 示例

    v[i] = ++i;   //  其结果是未定义的

一条不错经验法则是，你不应当在一个表达式中两次读取你所写入的值。

##### 强制实施

可以由优秀的分析器检测出来。

### <a id="res-order-fct"></a>ES.44: 不要对函数参数求值顺序有依赖

##### 理由

因为这种顺序是未定义的。

##### 注解

C++17 收紧了有关求值顺序的规则，但函数实参求值顺序仍然是未指明的。

##### 示例

    int i = 0;
    f(++i, ++i);

在 C++17 之前，其行为是未定义的。因此其行为可能是任何事（比如 `f(2, 2)`）。
自 C++17 起，中这段代码没有未定义行为，但仍未指定是哪个实参被首先求值。这个调用会是 `f(0, 1)` 或 `f(1, 0)`，但你不知道是哪个。

##### 示例

重载运算符可能导致求值顺序问题：

    f1()->m(f2());          // m(f1(), f2())
    cout << f1() << f2();   // operator<<(operator<<(cout, f1()), f2())

在 C++17 中，这些例子将按预期工作（自左向右），而赋值则按自右向左求值（`=` 正是自右向左绑定的）

    f1() = f2();    // C++14 中为未定义行为；C++17 中 f2() 在 f1() 之前求值

##### 强制实施

可以由优秀的分析器检测出来。

### <a id="res-magic"></a>ES.45: 避免“魔法常量”，采用符号化常量

##### 理由

表达式中内嵌的无名的常量很容易被忽略，而且经常难于理解：

##### 示例

    for (int m = 1; m <= 12; ++m)   // 请勿如此: 魔法常量 12
        cout << month[m] << '\n';

不是所有人都知道一年中有 12 个月份，号码是 1 到 12。更好的做法是：

    // 月份索引值为 1..12
    constexpr int first_month = 1;
    constexpr int last_month = 12;

    for (int m = first_month; m <= last_month; ++m)   // 好多了
        cout << month[m] << '\n';

更好的做法是，不要暴露常量：

    for (auto m : month)
        cout << m << '\n';

##### 强制实施

标记代码中的字面量。让 `0`，`1`，`nullptr`，`\n'`，`""`，以及某个确认列表中的其他字面量通过检查。

### <a id="res-narrowing"></a>ES.46: 避免丢失数据（窄化、截断）的算术转换

##### 理由

窄化转换会销毁信息，通常是不期望发生的。

##### 示例，不好

关键的例子就是基本的窄化：

    double d = 7.9;
    int i = d;    // 不好: 窄化: i 变为了 7
    i = (int) d;  // 不好: 我们打算声称这样的做法仍然不够明确

    void f(int x, long y, double d)
    {
        char c1 = x;   // 不好: 窄化
        char c2 = y;   // 不好: 窄化
        char c3 = d;   // 不好: 窄化
    }

##### 注解

指导方针支持库提供了一个 `narrow_cast` 操作，用以指名发生窄化是可接受的，以及一个 `narrow`（“窄化判定”）当窄化将会损失合法值时将会抛出一个异常：

    i = gsl::narrow_cast<int>(d);   // OK (明确需要): 窄化: i 变为了 7
    i = gsl::narrow<int>(d);        // OK: 抛出 narrowing_error

其中还包含了一些含有损失的算术强制转换，比如从负的浮点类型到无符号整型类型的强制转换：

    double d = -7.9;
    unsigned u = 0;

    u = d;                               // 不好：发生窄化
    u = gsl::narrow_cast<unsigned>(d);   // OK (明确需要): u 变为了 4294967289
    u = gsl::narrow<unsigned>(d);        // OK：抛出 narrowing_error

##### 注解

这条规则不适用于[按语境转换为 bool](https://en.cppreference.com/w/cpp/language/implicit_conversion#Contextual_conversions) 的情形：

    if (ptr) do_something(*ptr);   // OK：ptr 被用作条件
    bool b = ptr;                  // 不好：发生窄化

##### 强制实施

优良的分析器可以检测到所有的窄化转换。不过，对所有的窄化转换都进行标记将带来大量的误报。建议的做法是：

* 标记出所有的浮点向整数转换（可能只有 `float`->`char` 和 `double`->`int`。这里有问题！需要数据支持）。
* 标记出所有的 `long`->`char`（我怀疑 `int`->`char` 非常常见。这里有问题！需要数据支持）。
* 在函数参数上发生的窄化转换特别值得怀疑。

### <a id="res-nullptr"></a>ES.47: 使用 `nullptr` 而不是 `0` 或 `NULL`

##### 理由

可读性。最小化意外：`nullptr` 不可能和 `int` 混淆。
`nullptr` 还有一个严格定义的（非常严格）类型，且因此
可以在类型推断可能在 `NULL` 或 `0` 上犯错的场合中仍能
正常工作。

##### 示例

考虑：

    void f(int);
    void f(char*);
    f(0);         // 调用 f(int)
    f(nullptr);   // 调用 f(char*)

##### 强制实施

对用作指针的 `0` 和 `NULL` 进行标记。可以用简单的程序变换来达成这种变换。

### <a id="res-casts"></a>ES.48: 避免强制转换

##### 理由

强制转换是众所周知的错误来源，它们使得一些优化措施变得不可靠。

##### 示例，不好

    double d = 2;
    auto p = (long*)&d;
    auto q = (long long*)&d;
    cout << d << ' ' << *p << ' ' << *q << '\n';

你觉得这段代码会打印出什么呢？结果最好由实现定义。我得到的是

    2 0 4611686018427387904

加上这些

    *q = 666;
    cout << d << ' ' << *p << ' ' << *q << '\n';

得到的是

    3.29048e-321 666 666

奇怪吗？我很庆幸程序没有崩溃掉。

##### 注解

写下强制转换的程序员通常认为他们知道所做的是什么事情，
或者写出强制转换能让程序“更易读”。
而实际上，他们这样经常会禁止掉使用值的一些一般规则。
如果存在正确的函数的话，重载决议和模板实例化通常都能挑选出正确的函数。
如果没有的话，则可能本应如此，而不应该进行某种局部的修补（强制转换）。

##### 注解

强制转换在系统编程语言中是必要的。例如，否则我们怎么
才能把设备寄存器的地址放入一个指针呢？然而，强制转换
却被严重过度使用了，而且也是一种主要的错误来源。

当你觉得需要进行大量强制转换时，可能存在一个基本的设计问题。

[类型剖面配置](#pro-type-reinterpretcast) 禁止使用 `reinterpret_cast` 和 C 风格强制转换。

不要以强制转换为 `(void)` 来忽略 `[[nodiscard]]` 返回值。
当你有意要丢弃这种返回值时，应当首先深入思考这是不是确实是个好主意（通常，这个函数或者使用了 `[[nodiscard]]` 的返回类型的作者，当初确实是有充分理由的）。
要是你仍然觉得这样做是合适的，而且你的代码评审者也同意的话，使用 `std::ignore =` 来关闭这个警告，这既简单，可移植，也易于 grep。

##### 替代方案

强制转换被广泛（误）用了。现代 C++ 已经提供了一些规则和语言构造，消除了许多语境中对强制转换的需求，比如

* 使用模板
* 使用 `std::variant`
* 借助良好定义的，安全的，指针类型之间的隐式转换
* 使用 `std::ignore =` 来忽略 `[[nodiscard]]` 值

##### 强制实施

* 对包括向 `void` 在内的所有 C 风格强制转换进行标记。
* 对使用 `Type(value)` 的函数风格强制转换进行标记。应代之以使用不会发生窄化的 `Type{value}`。（参见 [ES.64](#res-construct)。）
* 对指针类型之间的[同一强制转换](#pro-type-identitycast)，若其中的源类型和目标类型相同(#pro-type-identitycast)则进行标记。
* 对可以作为[隐式转换](#pro-type-implicitpointercast)的显示指针强制转换进行标记。

### <a id="res-casts-named"></a>ES.49: 当必须使用强制转换时，使用具名的强制转换

##### 理由

可读性。避免错误。
具名的强制转换比 C 风格或函数风格的强制转换更加特殊，允许编译器捕捉到某些错误。

具名的强制转换包括：

* `static_cast`
* `const_cast`
* `reinterpret_cast`
* `dynamic_cast`
* `std::move`         // `move(x)` 是指代 `x` 的右值引用
* `std::forward`      // `forward<T>(x)` 是指代 `x` 的左值或右值引用（取决于 `T`）
* `gsl::narrow_cast`  // `narrow_cast<T>(x)` 就是 `static_cast<T>(x)`
* `gsl::narrow`       // `narrow<T>(x)` 在当 `static_cast<T>(x) == x` 时即为 `static_cast<T>(x)` 否则会抛出 `narrowing_error`

##### 示例

    class B { /* ... */ };
    class D { /* ... */ };

    template<typename D> D* upcast(B* pb)
    {
        D* pd0 = pb;                        // 错误：不存在从 B* 向 D* 的隐式转换
        D* pd1 = (D*)pb;                    // 合法，但干了什么？
        D* pd2 = static_cast<D*>(pb);       // 错误：D 并非派生于 B
        D* pd3 = reinterpret_cast<D*>(pb);  // OK：你自己负责！
        D* pd4 = dynamic_cast<D*>(pb);      // OK：返回 nullptr
        // ...
    }

这个例子是从真实世界的 BUG 合成的，其中 `D` 曾经派生于 `B`，但某个人重构了继承层次。
C 风格的强制转换很危险，因为它可以进行任何种类的转换，使我们丧失了今后受保护不犯错的机会。

##### 注解

当在类型之间进行没有信息丢失的转换时（比如从 `float` 到
`double` 或者从 `int32` 到 `int64`），可以代之以使用花括号初始化。

    double d {some_float};
    int64_t i {some_int32};

这样做明确了有意进行类型转换，而且同样避免了
发生可能导致精度损失的结果的类型转换。（比如说，
试图用这种风格来从 `double` 初始化 `float` 会导致
编译错误。）

##### 注解

`reinterpret_cast` 可以很基础，但其基础用法（如将机器地址转化为指针）并不是类型安全的：

    auto p = reinterpret_cast<Device_register>(0x800);  // 天生危险


##### 强制实施

* 对包括向 `void` 在内的所有 C 风格强制转换进行标记。
* 对使用 `Type(value)` 的函数风格强制转换进行标记。应代之以使用不会发生窄化的 `Type{value}`。（参见 [ES.64](#res-construct)。）
* [类型剖面配置](#pro-type-reinterpretcast)禁用了 `reinterpret_cast`。
* [类型剖面配置](#pro-type-arithmeticcast)对于在算术类型之间使用 `static_cast` 时给出警告。

### <a id="res-casts-const"></a>ES.50: 不要强制掉 `const`

##### 理由

这是在 `const` 上说谎。
若变量确实声明为 `const`，修改它将导致未定义的行为。

##### 示例，不好

    void f(const int& x)
    {
        const_cast<int&>(x) = 42;   // 不好
    }

    static int i = 0;
    static const int j = 0;

    f(i); // 暗藏的副作用
    f(j); // 未定义的行为

##### 示例

有时候，你可能倾向于借助 `const_cast` 来避免代码重复，比如两个访问函数仅在是否 `const` 上有区别而实现相似的情况。例如：

    class Bar;

    class Foo {
    public:
        // 不好，逻辑重复
        Bar& get_bar()
        {
            /* 获取 my_bar 的非 const 引用前后的复杂逻辑 */
        }

        const Bar& get_bar() const
        {
            /* 获取 my_bar 的 const 引用前后的相同的复杂逻辑 */
        }
    private:
        Bar my_bar;
    };

应当改为共享实现。通常可以直接让非 `const` 函数来调用 `const` 函数。不过当逻辑复杂的时候这可能会导致下面这样的模式，仍然需要借助于 `const_cast`：

    class Foo {
    public:
        // 不大好，非 const 函数调用 const 版本但借助于 const_cast
        Bar& get_bar()
        {
            return const_cast<Bar&>(static_cast<const Foo&>(*this).get_bar());
        }
        const Bar& get_bar() const
        {
            /* 获取 my_bar 的 const 引用前后的复杂逻辑 */
        }
    private:
        Bar my_bar;
    };

虽然这个模式如果恰当应用的话是安全的（因为调用方必然以一个非 `const` 对象来开始），但这并不理想，因为其安全性无法作为检查工具的规则而自动强制实施。

换种方式，可以优先将公共代码放入一个公共辅助函数中，并将之作为模板以使其推断 `const`。这完全不会用到 `const_cast`：

    class Foo {
    public:                         // 好
              Bar& get_bar()       { return get_bar_impl(*this); }
        const Bar& get_bar() const { return get_bar_impl(*this); }
    private:
        Bar my_bar;

        template<class T>           // 好，推断出 T 是 const 还是非 const
        static auto& get_bar_impl(T& t)
            { /* 获取 my_bar 的可能为 const 的引用前后的复杂逻辑 */ }
    };

注意：不要在模板中编写大型的非待决代码，因为它将导致代码爆炸。例如，进一步改进是当 `get_bar_impl` 的全部或部分代码是非待决代码时将之重构移出到一个公共的非模板函数中去，这可能会使代码大小显著变小。

##### 例外

当调用 `const` 不正确的函数时，你可能需要强制掉 `const`。
应当优先将这种函数包装到内联的 `const` 正确的包装函数中，以将强制转换封装到一处中。

##### 示例

有时候，“强制掉 `const`”是为了允许对本来无法改动的对象中的某种临时性的信息进行更新操作。
其例子包括进行缓存，备忘，以及预先计算等。
这样的例子，通常可以通过使用 `mutable` 或者通过一层间接进行处理，而同使用 `const_cast` 一样甚或比之更好。

考虑为昂贵操作将之前所计算的结果保留下来：

    int compute(int x); // 为 x 计算一个值；假设这是昂贵的

    class Cache {   // 为 int->int 操作实现一种高速缓存的某个类型
    public:
        pair<bool, int> find(int x) const;   // 有针对 x 的值吗？
        void set(int x, int v);             // 使 y 成为针对 x 的值
        // ...
    private:
        // ...
    };

    class X {
    public:
        int get_val(int x)
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache.set(x, val); // 插入针对 x 的值
            return val;
        }
        // ...
    private:
        Cache cache;
    };

这里的 `get_val()` 逻辑上是个常量，因此我们想使其成为 `const` 成员。
为此我们仍然需要改动 `cache`，因此人们有时候会求助于 `const_cast`：

    class X {   // 基于强制转换的可疑的方案
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            const_cast<Cache&>(cache).set(x, val);   // 很难看
            return val;
        }
        // ...
    private:
        Cache cache;
    };

幸运的是，有一种更好的方案：
将 `cache` 称为即便对于 `const` 对象来说也是可改变的：

    class X {   // 更好的方案
    public:
        int get_val(int x) const
        {
            auto p = cache.find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache.set(x, val);
            return val;
        }
        // ...
    private:
        mutable Cache cache;
    };

另一种替代方案是存储指向 `cache` 的指针：

    class X {   // OK，但有点麻烦的方案
    public:
        int get_val(int x) const
        {
            auto p = cache->find(x);
            if (p.first) return p.second;
            int val = compute(x);
            cache->set(x, val);
            return val;
        }
        // ...
    private:
        unique_ptr<Cache> cache;
    };

这个方案最灵活，但需要显式进行 `*cache` 的构造和销毁
（最可能发生于 `X` 的构造函数和析构函数中）。

无论采用哪种形式，在多线程代码中都需要保护对 `cache` 的数据竞争，可能需要使用一个 `std::mutex`。

##### 强制实施

* 标记 `const_cast`。
* 本条规则属于[类型安全性剖面配置](#pro-type-constcast)。

### <a id="res-range-checking"></a>ES.55: 避免发生对范围检查的需要

##### 理由

无法溢出的构造时不会溢出的（而且通常运行得更快）：

##### 示例

    for (auto& x : v)      // 打印 v 的所有元素
        cout << x << '\n';

    auto p = find(v, x);   // 在 v 中寻找 x

##### 强制实施

查找显式的范围检查，并启发式地给出替代方案建议。

### <a id="res-move"></a>ES.56: 仅在确实需要明确移动某个对象到别的作用域时才使用 `std::move()`

##### 理由

我们用移动而不是复制，以避免发生重复并提升性能。

一次移动通常会遗留一个空对象（[C.64](#rc-move-semantic)），这可能令人意外甚至很危险，因此我们试图避免从左值进行移动（它们可能随后会被访问到）。

##### 注解

当来源是右值（比如 `return` 的值或者函数的结果）时就会隐式地进行移动，因此请不要在这些情况下明确写下 `move` 而无意义地使代码复杂化。可以代之以编写简短的返回值的函数，这样的话无论是函数的返回还是调用方的返回值接收，都会很自然地得到优化。

一般来说，遵循本文档中的指导方针（包括不要让变量的作用域无必要地变大，编写返回值的简短函数，返回局部变量等），有助于消除大多数对显式使用 `std::move` 的需要。

显式的 `move` 需要用于把某个对象明确移动到另一个作用域，尤其是将其传递给某个“接收器”函数，以及移动操作自身（移动构造函数，移动赋值运算符）和交换（`swap`）操作的实现之中。

##### 示例，不好

    void sink(X&& x);   // sink 接收 x 的所有权

    void user()
    {
        X x;
        // 错误: 无法将作者绑定到右值引用
        sink(x);
        // OK: sink 接收了 x 的内容，x 随即必须假定为空
        sink(std::move(x));

        // ...

        // 可能是个错误
        use(x);
    }

通常来说，`std::move()` 都用做某个 `&&` 形参的实参。
而这点之后，应当假定对象已经被移走（参见 [C.64](#rc-move-semantic)），而直到首次向它设置某个新值之前，请勿再次读取它的状态。

    void f()
    {
        string s1 = "supercalifragilisticexpialidocious";

        string s2 = s1;             // ok, 接收了一个副本
        assert(s1 == "supercalifragilisticexpialidocious");  // ok

        // 不好, 如果你打算保留 s1 的值的话
        string s3 = move(s1);

        // 不好, assert 很可能会失败, s1 很可能被改动了
        assert(s1 == "supercalifragilisticexpialidocious");
    }

##### 示例

    void sink(unique_ptr<widget> p);  // 将 p 的所有权传递给 sink()

    void f()
    {
        auto w = make_unique<widget>();
        // ...
        sink(std::move(w));               // ok, 交给 sink()
        // ...
        sink(w);    // 错误: unique_ptr 经过严格设计，你无法复制它
    }

##### 注解

`std::move()` 经过伪装的向 `&&` 的强制转换；其自身并不会移动任何东西，但会把具名的对象标记为可被移动的候选者。
语言中已经了解了对象可以被移动的一般情况，尤其是从函数返回时，因此请不要用多余的 `std::move()` 使代码复杂化。

绝不要仅仅因为听说过“这样更加高效”就使用 `std::move()`。
通常来说，请不要相信那些没有支持数据的有关“效率”的断言。(???).
通常来说，请不要无理由地使代码复杂化。(??)
绝不要在 const 对象上 `std::move()`，它只会暗中将其转变成一个副本（参见 [Meyers15](#meyers15) 的条款 23)。

##### 示例，不好

    vector<int> make_vector()
    {
        vector<int> result;
        // ... 加载 result 的数据
        return std::move(result);       // 不好; 直接写 "return result;" 即可
    }

绝不要写 `return move(local_variable);`，这是因为语言已经知道这个变量是移动的候选了。
在这段代码中用 `move` 并不会带来帮助，而且可能实际上是有害的，因为它创建了局部变量的一个额外引用别名，而在某些编译器中这回对 RVO（返回值优化）造成影响。


##### 示例，不好

    vector<int> v = std::move(make_vector());   // 不好; 这个 std::move 完全是多余的

绝不在返回值上使用 `move`，如 `x = move(f());`，其中的 `f` 按值返回。
语言已经知道返回值是临时对象而且可以被移动。

##### 示例

    void mover(X&& x)
    {
        call_something(std::move(x));         // ok
        call_something(std::forward<X>(x));   // 不好, 请勿对右值引用 std::forward
        call_something(x);                    // 可疑  为什么不用std:: move?
    }

    template<class T>
    void forwarder(T&& t)
    {
        call_something(std::move(t));         // 不好, 请勿对转发引用 std::move
        call_something(std::forward<T>(t));   // ok
        call_something(t);                    // 可疑, 为什么不用 std::forward?
    }

##### 强制实施

* 对于 `std::move(x)` 的使用，当 `x` 是右值，或者语言已经将其当做右值，这包括 `return std::move(local_variable);` 以及在按值返回的函数上的 `std::move(f())`，进行标记
* 当没有接受 `const S&` 的函数重载来处理左值时，对接受 `S&&` 参数的函数进行标记。
* 若实参经过 `std::move` 传递给形参则进行标记，除非形参的类型为右值引用 `X&&`，或者类型是只能移动的而该形参为按值传递。
* 当对转发引用（`T&&` 其中 `T` 为模板参数类型）使用 `std::move` 时进行标记。应当代之以使用 `std::forward`。
* 当对并非非 const 右值引用的变量使用 `std::move` 时进行标记。（这是前一条规则的更一般的情况，以覆盖非转发的情况。）
* 当对右值引用（`X&&` 其中 `X` 为非模板形参类型）使用 `std::forward` 时进行标记。应当代之以使用 `std::move`。
* 当对并非转发引用使用 `std::forward` 时进行标记。（这是前一条规则的更一般的情况，以覆盖非移动的情况。）
* 如果对象潜在地被移动走之后的下一个操作是 `const` 操作的话，则进行标记；首先应当交错进行一个非 `const` 操作，最好是赋值，以首先对对象的值进行重置。

### <a id="res-new"></a>ES.60: 避免在资源管理函数之外使用 `new` 和 `delete`

##### 理由

应用程序代码中的直接资源管理既易错又麻烦。

##### 注解

通常也被称为“禁止裸 `new`！”规则。

##### 示例，不好

    void f(int n)
    {
        auto p = new X[n];   // n 个默认构造的 X
        // ...
        delete[] p;
    }

`...` 部分中的代码可能导致 `delete` 永远不会发生。

**参见**: [R: 资源管理](#s-resource)

##### 强制实施

对裸的 `new` 和裸的 `delete` 进行标记。

### <a id="res-del"></a>ES.61: 用 `delete[]` 删除数组，用 `delete` 删除非数组对象

##### 理由

这正是语言的要求，而且所犯的错误将导致资源释放的错误以及内存破坏。

##### 示例，不好

    void f(int n)
    {
        auto p = new X[n];   // n 个默认初始化的 X
        // ...
        delete p;   // 错误: 仅仅删除了对象 p，而并未删除数组 p[]
    }

##### 注解

这个例子不仅像上前一个例子一样违反了[禁止裸 `new` 规则](#res-new)，它还有更多的问题。

##### 强制实施

* 如果 `new` 和 `delete` 在同一个作用域中的话，就可以标记出现错误。
* 如果 `new` 和 `delete` 出现在构造函数/析构函数对之中的话，就可以标记出现错误。

### <a id="res-arr2"></a>ES.62: 不要在不同的数组之间进行指针比较

##### 理由

这样做的结果是未定义的。

##### 示例，不好

    void f()
    {
        int a1[7];
        int a2[9];
        if (&a1[5] < &a2[7]) {}       // 不好: 未定义
        if (0 < &a1[5] - &a2[7]) {}   // 不好: 未定义
    }

##### 注解

这个例子中有许多问题。

##### 强制实施

???

### <a id="res-slice"></a>ES.63: 不要产生切片

##### 理由

切片——亦即使用赋值或初始化而只对对象的一部分进行复制——通常会导致错误，
这是因为对象总是被当成是一个整体。
在罕见的进行蓄意的切片的代码中，其代码会让人意外。

##### 示例

    class Shape { /* ... */ };
    class Circle : public Shape { /* ... */ Point c; int r; };

    Circle c {{ "{{" }}0, 0}, 42};
    Shape s {c};    // 仅复制构造了 Circle 中的 Shape 部分
    s = c;          // 仅复制赋值了 Circle 中的 Shape 部分

    void assign(const Shape& src, Shape& dest)
    {
        dest = src;
    }
    Circle c2 {{ "{{" }}1, 1}, 43};
    assign(c, c2);   // 噢，传递的并不是整个状态
    assert(c == c2); // 如果提供复制操作，就也得提供比较操作，
                     //   但这里很可能返回 false

这样的结果是无意义的，因为不会把中心和半径从 `c` 复制给 `s`。
针对这个的第一条防线是[将基类 `Shape` 定义为不允许这样做](#rc-copy-virtual)。

##### 替代方案

如果确实需要切片的话，应当为之定义一个明确的操作。
这会避免读者产生混乱。
例如：

    class Smiley : public Circle {
        public:
        Circle copy_circle();
        // ...
    };

    Smiley sm { /* ... */ };
    Circle c1 {sm};  // 理想情况下由 Circle 的定义所禁止
    Circle c2 {sm.copy_circle()};

##### 强制实施

针对切片给出警告。

### <a id="res-construct"></a>ES.64: 使用 `T{e}` 写法来进行构造

##### 理由

对象构造语法 `T{e}` 明确了所需进行的构造。
对象构造语法 `T{e}` 不允许发生窄化。
`T{e}` 是唯一安全且通用的由表达式 `e` 构造一个 `T` 类型的值的表达式。
强制转换的写法 `T(e)` 和 `(T)e` 既不安全也不通用。

##### 示例

对于内建类型，构造的写法保护了不发生窄化和重解释

    void use(char ch, int i, double d, char* p, long long lng)
    {
        int x1 = int{ch};     // OK，但多余
        int x2 = int{d};      // 错误：double->int 窄化；如果需要的话应使用强制转换
        int x3 = int{p};      // 错误：指针->int；如果确实需要的话应使用 reinterpret_cast
        int x4 = int{lng};    // 错误：long long->int 窄化；如果需要的话应使用强制转换

        int y1 = int(ch);     // OK，但多余
        int y2 = int(d);      // 不好：double->int 窄化；如果需要的话应使用强制转换
        int y3 = int(p);      // 不好：指针->int；如果确实需要的话应使用 reinterpret_cast
        int y4 = int(lng);    // 不好：long long->int 窄化；如果需要的话应使用强制转换

        int z1 = (int)ch;     // OK，但多余
        int z2 = (int)d;      // 不好：double->int 窄化；如果需要的话应使用强制转换
        int z3 = (int)p;      // 不好：指针->int；如果确实需要的话应使用 reinterpret_cast
        int z4 = (int)lng;    // 不好：long long->int 窄化；如果需要的话应使用强制转换
    }

整数和指针之间的转换，在使用 `T(e)` 和 `(T)e` 时是由实现定义的，
而且在不同整数和指针大小的平台之间不可移植。

##### 注解

[避免强制转换](#res-casts)（显式类型转换），如果必须要做的话[优先采用具名强制转换](#res-casts-named)。

##### 注解

当没有歧义时，可以不写 `T{e}` 中的 `T`。

    complex<double> f(complex<double>);

    auto z = f({2*pi,1});

##### 注解

对象构造语法是最通用的[初始化式语法](#res-list)。

##### 例外

`std::vector` 和其他的容器是在 `{}` 作为对象构造语法之前定义的。
考虑：

    vector<string> vs {10};                           // 十个空字符串
    vector<int> vi1 {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};  // 十个元素 1..10
    vector<int> vi2 {10};                             // 一个值为 10 的元素

如何得到包含十个默认初始化的 `int` 的 `vector`？

    vector<int> v3(10); // 十个值为 0 的元素

使用 `()` 而不是 `{}` 作为元素数量是一种约定（源于 1980 年代早期），很难改变，
但仍然是一个设计错误：对于元素类型与元素数量可能发生混淆的容器，必须解决
其中的歧义。
约定的方案是将 `{10}` 解释为单个元素的列表，而用 `(10)` 来指定大小。

不应当在新代码中重复这个错误。
可以定义一个类型来表示元素的数量：

    struct Count { int n; };

    template<typename T>
    class Vector {
    public:
        Vector(Count n);                     // n 个默认构造的元素
        Vector(initializer_list<T> init);    // init.size() 个元素
        // ...
    };

    Vector<int> v1{10};
    Vector<int> v2{Count{10}};
    Vector<Count> v3{Count{10}};    // 这里仍有一个很小的问题

剩下的主要问题就是为 `Count` 找个合适的名字了。

##### 强制实施

标记 C 风格的 `(T)e` 和函数式风格的 `T(e)` 强制转换。


### <a id="res-deref"></a>ES.65: 不要解引用无效指针

##### 理由

解引用如 `nullptr` 这样的无效指针是未定义的行为，通常会导致程序立刻崩溃，
产生错误结果，或者内存被损坏。

##### 注解

本条规则是显然并且广为知晓的语言规则，但其可能难于遵守。
这需要良好的编码风格，程序库支持，以及静态分析来消除违反情况而不耗费大量开销。
这正是 [C++ 的类型和资源安全性模型](#stroustrup15)中所讨论的主要部分。

**参见**：

* 使用 [RAII](#rr-raii) 以避免生存期问题。
* 使用 [unique_ptr](#rf-unique_ptr) 以避免生存期问题。
* 使用 [shared_ptr](#rf-shared_ptr) 以避免生存期问题。
* 当不可能出现 `nullptr` 时应使用[引用](#rf-ptr-ref)。
* 使用 [not_null](#rf-nullptr) 以尽早捕捉到预期外的 `nullptr`。
* 使用[边界剖面配置](#ss-bounds)以避免范围错误。


##### 示例

    void f()
    {
        int x = 0;
        int* p = &x;

        if (condition()) {
            int y = 0;
            p = &y;
        } // p 失效

        *p = 42;            // 不好，若走了上面的分支则 p 无效
    }

为解决这个问题，要么应当扩展指针打算指代的这个对象的生存期，要么应当缩短指针的生存期（将解引用移动到所指代的对象生存期结束之前进行）。

    void f1()
    {
        int x = 0;
        int* p = &x;

        int y = 0;
        if (condition()) {
            p = &y;
        }

        *p = 42;            // OK，p 可指向 x 或 y，而它们都仍在作用域中
    }

不幸的是，大多数无效指针问题都更加难于定位且更加难于解决。

##### 示例

    void f(int* p)
    {
        int x = *p; // 不好：如何确定 p 是否有效？
    }

有大量的这种代码存在。
它们大多数都能工作（经过了大量的测试），但其各自是很难确定 `p` 是否可能为 `nullptr` 的。
后果就是，这同样是错误的一大来源。
有许多方案试图解决这个潜在问题：

    void f1(int* p) // 处理 nullptr
    {
        if (!p) {
            // 处理 nullptr（分配，返回，抛出，使 p 指向什么，等等
        }
        int x = *p;
    }

测试 `nullptr` 的做法有两个潜在的问题：

* 当遇到 `nullptr` 时应当做什么并不总是明确的
* 其测试可能多余并且相对比较昂贵
* 这个测试是为了保护某种违例还是所需逻辑的一部分并不明显

<!-- comment needed for code block after list -->
    void f2(int* p) // 声称 p 不应当为 nullptr
    {
        Assert(p);
        int x = *p;
    }

这样，仅当打开了断言检查时才会有所耗费，而且会向编译器/分析器提供有用的信息。
当 C++ 出现契约的直接支持后，还可以做的更好：

    void f3(int* p) // 声称 p 不应当为 nullptr
        [[expects: p]]
    {
        int x = *p;
    }

或者，还可以使用 `gsl::not_null` 来保证 `p` 不为 `nullptr`。

    void f(not_null<int*> p)
    {
        int x = *p;
    }

这些只是关于 `nullptr` 的处理办法。
要知道还有其他出现无效指针的方式。

##### 示例

    void f(int* p)  // 老代码，没使用 owner
    {
        delete p;
    }

    void g()        // 老代码：使用了裸 new
    {
        auto q = new int{7};
        f(q);
        int x = *q; // 不好：解引用了无效指针
    }

##### 示例

    void f()
    {
        vector<int> v(10);
        int* p = &v[5];
        v.push_back(99); // 可能重新分配 v 中的元素
        int x = *p; // 不好：解引用了潜在的无效指针
    }

##### 强制实施

本条规则属于[生存期安全性剖面配置](#ss-lifetime)

* 当对指向已经超出作用域的对象的指针进行解引用时进行标记
* 当对可能已经通过赋值 `nullptr` 而无效的指针进行解引用时进行标记
* 当对可能已经因 `delete` 而无效的指针进行解引用时进行标记
* 当对指向可能已经失效的容器元素的指针进行解引用时进行标记


## ES.stmt: 语句

语句控制了控制的流向（除了函数调用和异常抛出，它们是表达式）。

### <a id="res-switch-if"></a>ES.70: 面临选择时，优先采用 `switch` 语句而不是 `if` 语句

##### 理由

* 可读性。
* 效率：`switch` 与常量进行比较，且通常比一个 `if`-`then`-`else` 链中的一系列测试获得更好的优化。
* `switch` 可以启用某种启发式的一致性检查。例如，是否某个 `enum` 的所有值都被覆盖了？如果没有的话，是否存在 `default`？

##### 示例

    void use(int n)
    {
        switch (n) {   // 好
        case 0:
            // ...
            break;
        case 7:
            // ...
            break;
        default:
            // ...
            break;
        }
    }

要好于：

    void use2(int n)
    {
        if (n == 0)   // 不好：以 if-then-else 链和一组常量进行比较
            // ...
        else if (n == 7)
            // ...
    }

##### 强制实施

对以 `if`-`then`-`else` 链条（仅）和常量进行比较的情况进行标记。

### <a id="res-for-range"></a>ES.71: 面临选择时，优先采用范围式 `for` 语句而不是普通 `for` 语句

##### 理由

可读性。避免错误。效率。

##### 示例

    for (gsl::index i = 0; i < v.size(); ++i)   // 不好
        cout << v[i] << '\n';

    for (auto p = v.begin(); p != v.end(); ++p)   // 不好
        cout << *p << '\n';

    for (auto& x : v)    // OK
        cout << x << '\n';

    for (gsl::index i = 1; i < v.size(); ++i) // 接触了两个元素：无法作为范围式的 for
        cout << v[i] + v[i - 1] << '\n';

    for (gsl::index i = 0; i < v.size(); ++i) // 可能具有副作用：无法作为范围式的 for
        cout << f(v, &v[i]) << '\n';

    for (gsl::index i = 0; i < v.size(); ++i) { // 循环体中混入了循环变量：无法作为范围式 for
        if (i % 2 != 0)
            cout << v[i] << '\n'; // 输出奇数元素
    }

人类或优良的静态分析器也许可以确定，其实在 `f(v, &v[i])` 中的 `v` 的上并不真的存在副作用，因此这个循环可以被重写。

在循环体中“混入循环变量”的情况通常是最好进行避免的。

##### 注解

不要在范围式 `for` 循环中使用昂贵的循环变量副本：

    for (string s : vs) // ...

这将会对 `vs` 中的每个元素复制给 `s`。这样好一点：

    for (string& s : vs) // ...

更好的做法是，当循环变量不会被修改或复制时：

    for (const string& s : vs) // ...

##### 强制实施

查看循环，如果一个传统的循环仅会查看序列中的各个元素，而且其对这些元素所做的事中没有发生副作用，则将该循环重写为范围式的 `for` 循环。

### <a id="res-for-while"></a>ES.72: 当存在显然的循环变量时，优先采用 `for` 语句而不是 `while` 语句

##### 理由

可读性：循环的全部逻辑都“直观可见”。循环变量的作用域是有限的。

##### 示例

    for (gsl::index i = 0; i < vec.size(); i++) {
        // 干活
    }

##### 示例，不好

    int i = 0;
    while (i < vec.size()) {
        // 干活
        i++;
    }

##### 强制实施

???

### <a id="res-while-for"></a>ES.73: 当没有显然的循环变量时，优先采用 `while` 语句而不是 `for` 语句

##### 理由

可读性。

##### 示例

    int events = 0;
    for (; wait_for_event(); ++events) {  // 不好，含糊
        // ...
    }

这个“事件循环”会误导人，计数器 `events` 跟循环条件（`wait_for_event()`）并没有任何关系。
更好的做法是

    int events = 0;
    while (wait_for_event()) {      // 更好
        ++events;
        // ...
    }

##### 强制实施

对和 `for` 的条件不相关的 `for` 初始化式和 `for` 增量部分进行标记。

### <a id="res-for-init"></a>ES.74: 优先在 `for` 语句的初始化部分中声明循环变量

参见 [ES.6](#res-cond)

### <a id="res-do"></a>ES.75: 避免使用 `do` 语句

##### 理由

可读性，避免错误。
其终止条件处于尾部（而这可能会被忽略），且其条件不会在第一时间进行检查。

##### 示例

    int x;
    do {
        cin >> x;
        // ...
    } while (x < 0);

##### 注解

确实有一些天才的例子中，`do` 语句是更简洁的方案，但有问题的更多。

##### 强制实施

标记 `do` 语句。

### <a id="res-goto"></a>ES.76: 避免 `goto`

##### 理由

可读性，避免错误。存在对于人类更好的控制结构；`goto` 是用于机器生成的代码的。

##### 例外

跳出嵌套循环。
这种情况下应当总是向前跳出。

    for (int i = 0; i < imax; ++i)
        for (int j = 0; j < jmax; ++j) {
            if (a[i][j] > elem_max) goto finished;
            // ...
        }
    finished:
    // ...

##### 示例，不好

有相当数量的代码采用 C 风格的 goto-exit 惯用法：

    void f()
    {
        // ...
            goto exit;
        // ...
            goto exit;
        // ...
    exit:
        // ... 公共的清理代码 ...
    }

这是对析构函数的一种专门模仿。
应当将资源声明为带有清理的析构函数的包装类。
如果你由于某种原因无法用析构函数来处理所使用的各个变量的清理工作，
请考虑用 `gsl::finally()` 作为 `goto exit` 的一种简洁且更加可靠的替代方案。

##### 强制实施

* 标记 `goto`。更好的做法是标记出除了从嵌套内层循环中跳出到紧跟一组嵌套循环之后的语句的 `goto` 以外的所有 `goto`。

### <a id="res-continue"></a>ES.77: 尽量减少循环中使用的 `break` 和 `continue`

##### 理由

在不平凡的循环体中，容易忽略掉 `break` 或 `continue`。

循环中的 `break` 和 `switch` 语句中的 `break` 的含义有很大的区别，
（而且循环中可以有 `switch` 语句，`switch` 的 `case` 中也可以有循环）。

##### 示例

    switch(x) {
    case 1 :
        while (/* 某种条件 */) {
            // ...
        break;
        } // 噢！打算 break switch 还是 break while？
    case 2 :
        // ...
        break;
    }

##### 替代方案

通常，需要 `break` 的循环都是作为一个函数（算法）的良好候选者，其 `break` 将会变为 `return`。

    // 原始代码：break 内部循环
    void use1()
    {
        std::vector<T> vec = {/* 初始化为一些值 */};
        T value;
        for (const T item : vec) {
            if (/* 某种条件 */) {
                value = item;
                break;
            }
        }
        /* 然后对 value 做些事 */
    }

    // 这样更好：创建一个函数使其从循环中返回
    T search(const std::vector<T> &vec)
    {
        for (const T &item : vec) {
            if (/* 某种条件 */) return item;
        }
        return T(); // 默认值
    }

    void use2()
    {
        std::vector<T> vec = {/* 初始化为一些值 */};
        T value = search(vec);
        /* 然后对 value 做些事 */
    }

通常，使用 `continue` 的循环都可以等价且同样简洁地用 `if` 语句来表达。

    for (int item : vec) { // 不好
        if (item%2 == 0) continue;
        if (item == 5) continue;
        if (item > 10) continue;
        /* 对 item 做些事 */
    }

    for (int item : vec) { // 好
        if (item%2 != 0 && item != 5 && item <= 10) {
            /* 对 item 做些事 */
        }
    }

##### 注解

如果你确实要打断一个循环，使用 `break` 通常比使用诸如[修改循环变量](#res-loop-counter)或 [`goto`](#res-goto) 等其他方案更好：


##### 强制实施

???

### <a id="res-break"></a>ES.78: 不要依靠 `switch` 语句中的隐含直落行为

##### 理由

总是以 `break` 来结束非空的 `case`。意外地遗漏 `break` 是一种相当常见的 BUG。
蓄意的控制直落（fall through）是维护的噩梦，应该罕见并被明确标示出来。

##### 示例

    switch (eventType) {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        // 不好 - 隐式的控制直落
    case Error:
        display_error_window();
        break;
    }

单个语句带有多个 `case` 标签是可以的：

    switch (x) {
    case 'a':
    case 'b':
    case 'f':
        do_something(x);
        break;
    }

在 `case` 标签中使用返回语句也是可以的：

    switch (x) {
    case 'a':
        return 1;
    case 'b':
        return 2;
    case 'c':
        return 3;
    }

##### 例外

在罕见的直落被视为合适行为的情况中。应当明确标示，并使用 `[[fallthrough]]` 标注：

    switch (eventType) {
    case Information:
        update_status_bar();
        break;
    case Warning:
        write_event_log();
        [[fallthrough]];
    case Error:
        display_error_window();
        break;
    }

##### 注解

##### 强制实施

对所有从非空的 `case` 隐式发生的直落进行标记。


### <a id="res-default"></a>ES.79: `default`（仅）用于处理一般情况

##### 理由

代码清晰性。
提升错误检测的机会。

##### 示例

    enum E { a, b, c, d };

    void f1(E x)
    {
        switch (x) {
        case a:
            do_something();
            break;
        case b:
            do_something_else();
            break;
        default:
            take_the_default_action();
            break;
        }
    }

此处很明显有一种默认的动作，而情况 `a` 和 `b` 则是特殊情况。

##### 示例

不过当不存在默认动作而只想处理特殊情况时怎么办呢？
这种情况下，应当使用空的 `default`，否则没办法知道你确实处理了所有情况：

    void f2(E x)
    {
        switch (x) {
        case a:
            do_something();
            break;
        case b:
            do_something_else();
            break;
        default:
            // 其他情况无需动作
            break;
        }
    }

如果没有 `default` 的话，维护者以及编译器可能会合理地假定你有意处理所有情况：

    void f2(E x)
    {
        switch (x) {
        case a:
            do_something();
            break;
        case b:
        case c:
            do_something_else();
            break;
        }
    }

你是忘记了情况 `d` 还是故意遗漏了它？
当有人向枚举中添加一种情况，而又未能对每个针对这些枚举符的 `switch` 中添加时，
容易出现这种遗忘 `case` 的情况。

##### 强制实施

针对某个枚举的 `switch` 语句，若其未能处理其所有枚举符且没有 `default`，则对其进行标记。
这样做对于某些代码库可能会产生大量误报；此时，可以仅标记那些处理了大多数情况而不是所有情况的 `switch` 语句
（这正是第一个 C++ 编译器曾经的策略）。

### <a id="res-noname"></a>ES.84: 不要试图声明没有名字的局部变量

##### 理由

没有这种东西。
我们眼里看起来像是个无名变量的东西，对于编译器来说是一条由一个将会立刻离开作用域的临时对象所组成的语句。

##### 示例，不好

    void f()
    {
        lock_guard<mutex>{mx};   // 不好
        // ...
    }

这里声明了一个无名的 `lock_guard` 对象，它将在分号处立刻离开作用域。
这并不是一种少见的错误。
特别是，这个特别的例子会导致很难发觉的竞争条件。

##### 注解

无名函数实参是没问题的。

##### 强制实施

标记出仅有临时对象的语句。

### <a id="res-empty"></a>ES.85: 让空语句显著可见

##### 理由

可读性。

##### 示例

    for (i = 0; i < max; ++i);   // 不好: 空语句很容易被忽略
    v[i] = f(v[i]);

    for (auto x : v) {           // 好多了
        // 空
    }
    v[i] = f(v[i]);

##### 强制实施

对并非块语句且不包含注释的空语句进行标记。

### <a id="res-loop-counter"></a>ES.86: 避免在原生的 `for` 循环中修改循环控制变量

##### 理由

循环控制的第一行应当允许对循环中所发生的事情进行正确的推理。同时在循环的重复表达式和循环体之中修改循环计数器，是发生意外和 BUG 的一种经常性来源。

##### 示例

    for (int i = 0; i < 10; ++i) {
        // 未改动 i -- ok
    }

    for (int i = 0; i < 10; ++i) {
        //
        if (/* 某种情况 */) ++i; // 不好
        //
    }

    bool skip = false;
    for (int i = 0; i < 10; ++i) {
        if (skip) { skip = false; continue; }
        //
        if (/* 某种情况 */) skip = true;  // 有改善: 为两个概念使用了两个变量。
        //
    }

##### 强制实施

如果变量在循环控制的重复表达式和循环体中都潜在地进行更新（存在非 `const` 使用），则进行标记。


### <a id="res-if"></a>ES.87: 请勿在条件上添加多余的 `==` 或 `!=`

##### 理由

这样可避免啰嗦，并消除了发生某些错误的机会。
有助于使代码风格保持一致性和协调性。

##### 示例

根据定义，`if` 语句，`while` 语句，以及 `for` 语句中的条件，选择 `true` 或 `false` 的取值。
数值与 `0` 相比较，指针值与 `nullptr` 相比较。

    // 这些都表示“当 p 不是 nullptr 时”
    if (p) { ... }            // 好
    if (p != 0) { ... }       // !=0 是多余的；不好：不要对指针用 0
    if (p != nullptr) { ... } // !=nullptr 是多余的，不建议如此

通常，`if (p)` 可解读为“如果 `p` 有效”，这正是程序员意图的直接表达，
而 `if (p != nullptr)` 则只是一种啰嗦的变通写法。

##### 示例

这条规则对于把声明式用作条件时尤其有用

    if (auto pc = dynamic_cast<Circle>(ps)) { ... } // 执行是按照 ps 指向某种 Circle 来进行的，好

    if (auto pc = dynamic_cast<Circle>(ps); pc != nullptr) { ... } // 不建议如此

##### 示例

要注意，条件中会实施向 `bool` 的隐式转换。
例如：

    for (string s; cin >> s; ) v.push_back(s);

这里会执行 `istream` 的 `operator bool()`。

##### 注解

明确地将整数和 `0` 进行比较通常并非是多余的。
因为（与指针和布尔值相反），整数通常都具有超过两个的有效值。
此外 `0`（零）还经常会用于代表成功。
因此，最好明确地进行比较。

    void f(int i)
    {
        if (i)            // 可疑
        // ...
        if (i == success) // 可能更好
        // ...
    }

一定要记住整数可以有超过两个值。

##### 示例，不好

众所周知，

    if(strcmp(p1, p2)) { ... }   // 这两个 C 风格的字符串相等吗？（错误！）

是一种常见的新手错误。
如果使用 C 风格的字符串，那么就必须好好了解 `<cstring>` 中的函数。
即便冗余地写为

    if(strcmp(p1, p2) != 0) { ... }   // 这两个 C 风格的字符串相等吗？（错误！）

也不会有效果。

##### 注解

表达相反的条件的最简单的方式就是使用一次取反：

    // 这些都表示“当 p 为 nullptr 时”
    if (!p) { ... }           // 好
    if (p == 0) { ... }       // ==0 是多余的；不好：不要对指针用 0
    if (p == nullptr) { ... } // ==nullptr 是多余的，不建议如此

##### 强制实施

容易，仅需检查条件中多余的 `!=` 和 `==` 的使用即可。



## <a id="ss-numbers"></a>算术

### <a id="res-mix"></a>ES.100: 不要进行有符号和无符号混合运算

##### 理由

避免错误的结果。

##### 示例

    int x = -3;
    unsigned int y = 7;

    cout << x - y << '\n';  // 无符号结果，可能是 4294967286
    cout << x + y << '\n';  // 无符号结果：4
    cout << x * y << '\n';  // 无符号结果，可能是 4294967275

在更实际的例子中，这种问题更难于被发现。

##### 注解

不幸的是，C++ 使用有符号整数作为数组下标，而标准库使用无符号整数作为容器下标。
这妨碍了一致性。使用 `gsl::index` 来作为下标类型；[参见 ES.107](#res-subscripts)。

##### 强制实施

* 编译器已知这种情况，有些时候会给出警告。
* （避免噪声）有符号/无符号的混合比较，若其一个实参是 `sizeof` 或调用容器的 `.size()` 而另一个是 `ptrdiff_t`，则不要进行标记。


### <a id="res-unsigned"></a>ES.101: 使用无符号类型进行位操作

##### 理由

无符号类型支持位操作而没有符号位的意外。

##### 示例

    unsigned char x = 0b1010'1010;
    unsigned char y = ~x;   // y == 0b0101'0101;

##### 注解

无符号类型对于模算术也很有用。
不过，如果你想要进行模算术时，
应当按需添加代码注释以注明所依赖的回绕行为，因为这样的
代码会让许多程序员感觉意外。

##### 强制实施

* 一般来说基本不可能，因为标准库也使用了无符号下标。
???

### <a id="res-signed"></a>ES.102: 使用有符号类型进行算术运算

##### 理由

因为大多数算术都假定是有符号的；
当 `y > x` 时，`x - y` 都会产生负数，除了罕见的情况下你确实需要模算术。

##### 示例

当你不期望时，无符号算术会产生奇怪的结果。
这在混合有符号和无符号算术时有其如此。

    template<typename T, typename T2>
    T subtract(T x, T2 y)
    {
        return x - y;
    }

    void test()
    {
        int s = 5;
        unsigned int us = 5;
        cout << subtract(s, 7) << '\n';       // -2
        cout << subtract(us, 7u) << '\n';     // 4294967294
        cout << subtract(s, 7u) << '\n';      // -2
        cout << subtract(us, 7) << '\n';      // 4294967294
        cout << subtract(s, us + 2) << '\n';  // -2
        cout << subtract(us, s + 2) << '\n';  // 4294967294
    }

我们这次非常明确发生了什么。
但要是你见到 `us - (s + 2)` 或者 `s += 2; ...; us - s` 时，你确实能够预计到打印的结果将是 `4294967294` 吗？

##### 例外

如果你确实需要模算术的话就使用无符号类型——
根据需要为其添加代码注释以说明其依赖溢出行为，因为这样的
代码会让许多程序员感觉意外。

##### 示例

标准库使用无符号类型作为下标。
内建数组则用有符号类型作为下标。
这不可避免地带来了意外（以及 BUG）。

    int a[10];
    for (int i = 0; i < 10; ++i) a[i] = i;
    vector<int> v(10);
    // 比较有符号和无符号数；有些编译器会警告，但我们不能警告
    for (gsl::index i = 0; i < v.size(); ++i) v[i] = i;

    int a2[-2];         // 错误：负的大小

    // OK，但 int 的数值（4294967294）过大，应当会造成一个异常
    vector<int> v2(-2);

使用 `gsl::index` 作为下标类型；[参见 ES.107](#res-subscripts)。

##### 强制实施

* 对混合有符号和无符号算术进行标记。
* 对将无符号算术的结果作为有符号数赋值或打印进行标记。
* 对负数字面量（比如 `-2`）用作容器下标进行标记。
* （避免噪声）有符号/无符号的混合比较，若其一个实参是 `sizeof` 或调用容器的 `.size()` 而另一个是 `ptrdiff_t`，则不要进行标记。


### <a id="res-overflow"></a>ES.103: 避免上溢出

##### 理由

上溢出通常会让数值算法变得没有意义。
将值增加超过其最大值将导致内存损坏和未定义的行为。

##### 示例，不好

    int a[10];
    a[10] = 7;   // 不好，数组边界上溢出

    for (int n = 0; n <= 10; ++n)
        a[n] = 9;   // 不好，数组边界上溢出

##### 示例，不好

    int n = numeric_limits<int>::max();
    int m = n + 1;   // 不好，数值上溢出

##### 示例，不好

    int area(int h, int w) { return h * w; }

    auto a = area(10'000'000, 100'000'000);   // 不好，数值上溢出

##### 例外

如果你确实需要模算术的话就使用无符号类型。

**替代方案**: 对于可以负担一些开销的关键应用，可以使用带有范围检查的整数和/或浮点类型。

##### 强制实施

???

### <a id="res-underflow"></a>ES.104: 避免下溢出

##### 理由

将值减小超过其最小值将导致内存损坏和未定义的行为。

##### 示例，不好

    int a[10];
    a[-2] = 7;   // 不好

    int n = 101;
    while (n--)
        a[n - 1] = 9;   // 不好（两次）

##### 例外

如果你确实需要模算术的话就使用无符号类型。

##### 强制实施

???

### <a id="res-zero"></a>ES.105: 避免除整数零

##### 理由

其结果是未定义的，很可能导致程序崩溃。

##### 注解

这同样适用于 `%`。

##### 示例，不好

    int divide(int a, int b)
    {
        // 不好, 应当进行检查（比如一条前条件）
        return a / b;
    }

##### 示例，好

    int divide(int a, int b)
    {
        // 好, 通过前条件进行处置（并当 C++ 支持契约后可以进行替换）
        Expects(b != 0);
        return a / b;
    }

    double divide(double a, double b)
    {
        // 好, 通过换为使用 double 来处置
        return a / b;
    }

**替代方案**: 对于可以负担一些开销的关键应用，可以使用带有范围检查的整数和/或浮点类型。

##### 强制实施

* 对以可能为零的整型值的除法进行标记。


### <a id="res-nonnegative"></a>ES.106: 不要试图用 `unsigned` 来防止负数值

##### 理由

选用 `unsigned` 意味着对于包括模算术在内的整数的常规行为的许多改动，
它将抑制掉与溢出有关的警告，
并打开了与混合符号相关的错误的大门。
使用 `unsigned` 并不会真正消除负数值的可能性。

##### 示例

    unsigned int u1 = -2;   // 合法：u1 的值为 4294967294
    int i1 = -2;
    unsigned int u2 = i1;   // 合法：u2 的值为 4294967294
    int i2 = u2;            // 合法：i2 的值为 -2

真实代码中很难找出这样的（完全合法的）语法构造的问题，而它们是许多真实世界错误的来源。
考虑：

    unsigned area(unsigned height, unsigned width) { return height*width; } // [参见](#ri-expects)
    // ...
    int height;
    cin >> height;
    auto a = area(height, 2);   // 当输入为 -2 时 a 为 4294967292

记住把 `-1` 赋值给 `unsigned int` 会变成最大的 `unsigned int`。
而且，由于无符号算术是模算术，其乘法并不会溢出，而是会发生回绕。

##### 示例

    unsigned max = 100000;    // “不小心写错了”，应该写 10'000
    unsigned short x = 100;
    while (x < max) x += 100; // 无限循环

要是 `x` 是个有符号的 `short` 的话，我们就会得到有关溢出的未定义行为的警告了。

##### 替代方案

* 使用有符号整数并检查 `x >= 0`
* 使用某个正整数类型
* 使用某个整数子值域类型
* `Assert(-1 < x)`

例如

    struct Positive {
        int val;
        Positive(int x) :val{x} { Assert(0 < x); }
        operator int() { return val; }
    };

    int f(Positive arg) { return arg; }

    int r1 = f(2);
    int r2 = f(-2);  // 抛出异常

##### 注解

???

##### 强制实施

参见 ES.100 的强制实施。


### <a id="res-subscripts"></a>ES.107: 不要对下标使用 `unsigned`，优先使用 `gsl::index`

##### 理由

避免有符号和无符号混乱。
允许更好的优化。
允许更好的错误检测。
避免 `auto` 和 `int` 有关的陷阱。

##### 示例，不好

    vector<int> vec = /*...*/;

    for (int i = 0; i < vec.size(); i += 2)                    // 可能不够大
        cout << vec[i] << '\n';
    for (unsigned i = 0; i < vec.size(); i += 2)               // 有风险的回绕
        cout << vec[i] << '\n';
    for (auto i = 0; i < vec.size(); i += 2)                   // 可能不够大
        cout << vec[i] << '\n';
    for (vector<int>::size_type i = 0; i < vec.size(); i += 2) // 啰嗦
        cout << vec[i] << '\n';
    for (auto i = vec.size()-1; i >= 0; i -= 2)                // BUG
        cout << vec[i] << '\n';
    for (int i = vec.size()-1; i >= 0; i -= 2)                 // 可能不够大
        cout << vec[i] << '\n';

##### 示例，好

    vector<int> vec = /*...*/;

    for (gsl::index i = 0; i < vec.size(); i += 2)             // ok
        cout << vec[i] << '\n';
    for (gsl::index i = vec.size()-1; i >= 0; i -= 2)          // ok
        cout << vec[i] << '\n';

##### 注解

内建数组允许有符号的下标。
标准库容器使用无符号的下标。
因此没有完美的兼容解决方案（除非将来某一天，标准库容器改为使用有符号下标了）。
鉴于无符号和混合符号方面的已知问题，最好坚持使用（有符号）并且足够大的整数，而 `gsl::index` 保证了这点。

##### 示例

    template<typename T>
    struct My_container {
    public:
        // ...
        T& operator[](gsl::index i);    // 不是 unsigned
        // ...
    };

##### 示例

    ??? 演示改进后的代码生成和潜在可进行的错误检查 ???

##### 替代方案

可以代之以

* 使用算法
* 使用基于范围的 `for`
* 使用迭代器或指针

##### 强制实施

* 非常麻烦，因为标准库容器已经搞错了。
* （避免噪声）有符号/无符号的混合比较，若其一个实参是 `sizeof` 或调用容器的 `.size()` 而另一个是 `ptrdiff_t`，则不要进行标记。
