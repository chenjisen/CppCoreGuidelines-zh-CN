
# <a id="s-templates"></a>T: 模板和泛型编程

泛型编程是使用以类型、值和算法进行参数化的类型和算法所进行的编程。
在 C++ 中，泛型编程是以“模板”语言机制来提供支持的。

泛型函数的参数是以针对参数类型及其所涉及的值的规定的集合来刻画的。
在 C++ 中，这些规定是以被称为概念（concept）的编译期谓词来表达的。

模板还可用于进行元编程；亦即由编译期代码所组成的程序。

泛型编程的中心是“概念”；亦即以编译时谓词表现的对于模板参数的要求。
C++20 已经将“概念”标准化了，不过是在 GCC 6.1 中以一种略有不同的语法首次提供的。

模板使用的规则概览：

* [T.1: 用模板来提升代码的抽象层次](#rt-raise)
* [T.2: 用模板来表达适用于许多参数类型的算法](#rt-algo)
* [T.3: 用模板来表达容器和范围](#rt-cont)
* [T.4: 用模板来表达对语法树的操作](#rt-expr)
* [T.5: 结合泛型和面向对象技术来增强它们的能力，而不是它们的成本](#rt-generic-oo)

概念使用的规则概览：

* [T.10: 为所有模板实参指明概念](#rt-concepts)
* [T.11: 尽可能采用标准概念](#rt-std-concepts)
* [T.12: 对于局部变量，优先采用概念名而不是 `auto`](#rt-auto)
* [T.13: 对于简单的单类型参数概念，优先采用简写形式](#rt-shorthand)
* ???

概念定义的规则概览：

* [T.20: 避免没有有意义的语义的“概念”](#rt-low)
* [T.21: 为概念提出一组完整的操作要求](#rt-complete)
* [T.22: 为概念指明公理](#rt-axiom)
* [T.23: 通过添加新的使用模式，从更一般情形的概念中区分出提炼后的概念](#rt-refine)
* [T.24: 用标签类或特征类来区分仅在语义上存在差别的概念](#rt-tag)
* [T.25: 避免互补性的约束](#rt-not)
* [T.26: 优先采用使用模式而不是简单的语法来定义概念](#rt-use)
* [T.30: 节制地采用概念求反（`!C<T>`）来表示微小差异](#rt-???)
* [T.31: 节制地采用概念析取（disjunction）（`C1<T> || C2<T>`）来表示备选项](#rt-???)
* ???

模板接口的规则概览：

* [T.40: 使用函数对象向算法传递操作](#rt-fo)
* [T.41: 在模板的概念上仅提出基本的性质要求](#rt-essential)
* [T.42: 使用模板别名来简化写法并隐藏实现细节](#rt-alias)
* [T.43: 优先使用 `using` 而不是 `typedef` 来定义别名](#rt-using)
* [T.44: （如果可行）使用函数模板来对类模板的参数类型进行推断](#rt-deduce)
* [T.46: 要求模板参数至少是半正规的](#rt-regular)
* [T.47: 避免用常用名字命名高度可见的无约束模板](#rt-visible)
* [T.48: 如果你的编译器不支持概念的话，可以用 `enable_if` 来模拟](#rt-concept-def)
* [T.49: 尽可能避免类型擦除](#rt-erasure)

模板定义的规则概览：

* [T.60: 最小化模板的上下文依赖性](#rt-depend)
* [T.61: 请勿对成员进行过度参数化（SCARY）](#rt-scary)
* [T.62: 将无依赖的类模板成员置于一个非模板基类之中](#rt-nondependent)
* [T.64: 用特化来提供类模板的其他实现](#rt-specialization)
* [T.65: 用标签分派来提供函数的其他实现](#rt-tag-dispatch)
* [T.67: 用特化来提供不规则类型的其他实现](#rt-specialization2)
* [T.68: 在模板中用 `{}` 而不是 `()` 以避免歧义](#rt-cast)
* [T.69: 在模板中，请勿进行未限定的非成员函数调用，除非有意将之作为定制点](#rt-customization)

模板和类型层次的规则概览：

* [T.80: 请勿不成熟地对类层次进行模板化](#rt-hier)
* [T.81: 请勿混合类层次和数组](#rt-array) // ??? 放在“类型层次”部分
* [T.82: 当不想要虚函数时，可以将类层次线性化](#rt-linear)
* [T.83: 请勿声明虚的成员函数模板](#rt-virtual)
* [T.84: 使用非模板的核心实现来提供 ABI 稳定的接口](#rt-abi)
* [T.??: ????](#rt-???)

变参模板的规则概览：

* [T.100: 当需要可以接受可变数量的多种类型参数的函数时，使用变参模板](#rt-variadic)
* [T.101: ??? 如何向变参模板传递参数 ???](#rt-variadic-pass)
* [T.102: ??? 如何处理变参模板的参数 ???](#rt-variadic-process)
* [T.103: 请勿对同质参数列表使用变参模板](#rt-variadic-not)
* [T.??: ????](#rt-???)

元编程的规则概览：

* [T.120: 仅当确实需要时才使用模板元编程](#rt-metameta)
* [T.121: 模板元编程主要用于模拟概念机制](#rt-emulate)
* [T.122: 用模板（通常为模板别名）来在编译期进行类型运算](#rt-tmp)
* [T.123: 用 `constexpr` 函数来在编译期进行值运算](#rt-fct)
* [T.124: 优先使用标准库的模板元编程设施](#rt-std-tmp)
* [T.125: 当需要标准库之外的模板元编程设施时，使用某个现存程序库](#rt-lib)
* [T.??: ????](#rt-???)

其他模板规则概览：

* [T.140: 若操作可被重用，则应为其命名](#rt-name)
* [T.141: 当仅在一个地方需要一个简单的函数对象时，使用无名的 lambda](#rt-lambda)
* [T.142: 使用模板变量以简化写法](#rt-var)
* [T.143: 请勿编写并非有意非泛型的代码](#rt-nongeneric)
* [T.144: 请勿特化函数模板](#rt-specialize-function)
* [T.150: 用 `static_assert` 来检查类是否与概念相符](#rt-check-class)
* [T.??: ????](#rt-???)

## <a id="ss-gp"></a>T.gp: 泛型编程

泛型编程是利用以类型、值和算法进行了参数化的类型和算法所进行的编程。

### <a id="rt-raise"></a>T.1: 用模板来提升代码的抽象层次

##### 理由

通用性。重用。效率。鼓励用户类型的定义一致性。

##### 示例，不好

概念上说，以下要求是错误的，因为我们所要求的 `T` 不止是如“可以增量”或“可以进行加法”这样的非常低级的概念：

    template<typename T>
        requires Incrementable<T>
    T sum1(vector<T>& v, T s)
    {
        for (auto x : v) s += x;
        return s;
    }

    template<typename T>
        requires Simple_number<T>
    T sum2(vector<T>& v, T s)
    {
        for (auto x : v) s = s + x;
        return s;
    }

由于假设 `Incrementable` 并不支持 `+`，而 `Simple_number` 也不支持 `+=`，我们对 `sum1` 和 `sum2` 的实现者过度约束了。
而且，这种情况下也错过了一次通用化的机会。

##### 示例

    template<typename T>
        requires Arithmetic<T>
    T sum(vector<T>& v, T s)
    {
        for (auto x : v) s += x;
        return s;
    }

通过假定 `Arithmetic` 同时要求 `+` 和 `+=`，我们约束 `sum` 的使用者来提供完整的算术类型。
这并非是最小的要求，但它为算法的实现者更多所需的自由度，并确保了任何 `Arithmetic` 类型
都可被用于各种各样的算法。

为达成额外的通用性和可重用性，我们还可以用更通用的 `Container` 或 `Range` 概念而不是仅投入到 `vector` 一个容器上。

##### 注解

如果我们所定义的模板提出的要求，完全是由一个算法的一个实现所要求的操作
（比如说仅要求 `+=` 而不同时要求 `=` 和 `+`），而没有别的要求，就会对维护者提出过度的约束。
我们的目标是最小化模板参数的要求，但一个实现的绝对最小要求很少能作为有意义的概念。

##### 注解

可以用模板来表现几乎任何东西（它是图灵完备的），但（利用模板进行）泛型编程的目标在于，
使操作和算法对于一组带有相似语义性质的类型有效地进行通用化。

##### 强制实施

* 对带有“过于简单”的要求（诸如不用概念而直接使用特定的运算符）的算法进行标记。
* 不要对“过于简单”的概念本身进行标记；它们也许只不过是更有用的概念的构造块。

### <a id="rt-algo"></a>T.2: 用模板来表达适用于许多参数类型的算法

##### 理由

通用性。最小化源代码数量。互操作性。重用。

##### 示例

这正是 STL 的基础所在。一个 `find` 算法可以很轻易地同任何输入范围一起工作：

    template<typename Iter, typename Val>
        // requires Input_iterator<Iter>
        //       && Equality_comparable<Value_type<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

##### 注解

除非确实需要多于一种模板参数类型，否则请不要使用模板。
请勿过度抽象。

##### 强制实施

??? 很难办，可能需要人来进行

### <a id="rt-cont"></a>T.3: 用模板来表达容器和范围

##### 理由

容器需要一种元素类型，而将之表示为一个模板参数则是通用的，可重用的，且类型安全的做法。
这样也避免了采用脆弱的或者低效的变通手段。约定：这正是 STL 的做法。

##### 示例

    template<typename T>
        // requires Regular<T>
    class Vector {
        // ...
        T* elem;   // 指向 sz 个 T
        int sz;
    };

    Vector<double> v(10);
    v[7] = 9.9;

##### 示例，不好

    class Container {
        // ...
        void* elem;   // 指向 sz 个某种类型的元素
        int sz;
    };

    Container c(10, sizeof(double));
    ((double*) c.elem)[7] = 9.9;

这样做无法直接表现出程序员的意图，而且向类型系统和优化器隐藏了程序的结构。

把 `void*` 隐藏在宏之中只会掩盖问题，并引入新的发生混乱的机会。

**例外**: 当需要 ABI 稳定的接口时，也许你不得不提供一个基本实现，在基于它来呈现（类型安全的）目标。
参见[稳定的基类](#rt-abi)。

##### 强制实施

* 对出现于低级实现代码之外的 `void*` 和强制转换进行标记。

### <a id="rt-expr"></a>T.4: 用模板来表达对语法树的操作

##### 理由

 ???

##### 示例

    ???

**例外**: ???

### <a id="rt-generic-oo"></a>T.5: 结合泛型和面向对象技术来增强它们的能力，而不是它们的成本

##### 理由

泛型和面向对象技术是互补的。

##### 示例

静态能够帮助动态：使用静态多态来实现动态多态的接口：

    class Command {
        // 纯虚函数
    };

    // 实现
    template</*...*/>
    class ConcreteCommand : public Command {
        // 实现虚函数
    };

##### 示例

动态有助于静态：提供通用的，便利的，静态绑定的接口，但内部进行动态派发，这样就可以提供统一的对象布局。
这样的例子包括如 `std::shared_ptr` 的删除器的类型擦除。（不过[请勿过度使用类型擦除](#rt-erasure)。）

    #include <memory>

    class Object {
    public:
        template<typename T>
        Object(T&& obj)
            : concept_(std::make_shared<ConcreteCommand<T>>(std::forward<T>(obj))) {}

        int get_id() const { return concept_->get_id(); }

    private:
        struct Command {
            virtual ~Command() {}
            virtual int get_id() const = 0;
        };

        template<typename T>
        struct ConcreteCommand final : Command {
            ConcreteCommand(T&& obj) noexcept : object_(std::forward<T>(obj)) {}
            int get_id() const final { return object_.get_id(); }

        private:
            T object_;
        };

        std::shared_ptr<Command> concept_;
    };

    class Bar {
    public:
        int get_id() const { return 1; }
    };

    struct Foo {
    public:
        int get_id() const { return 2; }
    };

    Object o(Bar{});
    Object o2(Foo{});

##### 注解

类模板之中，非虚函数仅会在被使用时才会实例化——而虚函数则每次都会实例化。
这会导致代码大小膨胀，而且可能因为实例化从不需要的功能而导致对通用类型的过度约束。
请避免这样做，虽然标准的刻面类犯过这种错误。

##### 参见

* ref ???
* ref ???
* ref ???

##### 强制实施

参见参考条目以获得更具体的规则。

## <a id="ss-concepts"></a>T.concepts: 概念规则

概念是一种 C++20 语言设施，用于为模板参数指定要求。
在考虑泛型编程，以及未来的 C++ 程序库（无论标准的还是其他的）的基础时，
概念都是关键性的。

本部分假定有概念支持。

概念使用的规则概览：

* [T.10: 为所有模板实参指明概念](#rt-concepts)
* [T.11: 尽可能采用标准概念](#rt-std-concepts)
* [T.12: 优先采用概念名而不是 `auto`](#rt-auto)
* [T.13: 对于简单的单类型参数概念，优先采用简写形式](#rt-shorthand)
* ???

概念定义的规则概览：

* [T.20: 避免没有有意义的语义的“概念”](#rt-low)
* [T.21: 为概念提出一组完整的操作要求](#rt-complete)
* [T.22: 为概念指明公理](#rt-axiom)
* [T.23: 通过添加新的使用模式，从更一般情形的概念中区分出提炼后的概念](#rt-refine)
* [T.24: 用标签类或特征类来区分仅在语义上存在差别的概念](#rt-tag)
* [T.25: 避免互补性的约束](#rt-not)
* [T.26: 优先采用使用模式而不是简单的语法来定义概念](#rt-use)
* ???

## <a id="ss-concept-use"></a>T.con-use: 概念的使用

### <a id="rt-concepts"></a>T.10: 为所有模板实参指明概念

##### 理由

正确性和可读性。
针对模板参数所假定的含义（包括语法和语义），是模板接口的基础部分。
使用概念能够显著改善模板的文档和错误处理。
为模板参数指定概念是一种强力的设计工具。

##### 示例

    template<typename Iter, typename Val>
        requires input_iterator<Iter>
                 && equality_comparable_with<value_type_t<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

也可以等价地用更为简洁的方式：

    template<input_iterator Iter, typename Val>
        requires equality_comparable_with<value_type_t<Iter>, Val>
    Iter find(Iter b, Iter e, Val v)
    {
        // ...
    }

##### 注解

普通的 `typename`（或 `auto`）是受最少约束的概念。
应当仅在只能假定“这是一个类型”的罕见情况中使用它们。
通常，这只会在当我们（用模板元编程代码）操作纯粹的表达式树，并推迟进行类型检查时才会需要。

**参考**: TC++PL4

##### 强制实施

对没有概念的模板类型参数进行标记。

### <a id="rt-std-concepts"></a>T.11: 尽可能采用标准概念

##### 理由

“标准”概念（即由 [GSL](#gsl-guidelines-support-library) 和 ISO 标准自身所提供的概念)，
避免了我们思考自己的概念，它们比我们匆忙中能够想出来的要好得多，而且还提升了互操作性。

##### 注解

如果你不是要创建一个新的泛型程序库的话，大多数所需的概念都已经在标准库中定义过了。

##### 示例

    template<typename T>
        // 请勿定义这个: <iterator> 中已经有 sortable
    concept Ordered_container = Sequence<T> && Random_access<Iterator<T>> && Ordered<Value_type<T>>;

    void sort(Ordered_container auto& s);

这个 `Ordered_container` 貌似相当合理，但它和标准库中的 `sortable` 概念非常相似。
它是更好？更正确？它真的精确地反映了标准对于 `sort` 的要求吗？
直接使用 `sortable` 则更好而且更简单：

    void sort(sortable auto& s);   // 更好

##### 注解

在我们推进一个包含概念的 ISO 标准的过程中，“标准”概念的集合是不断演进的。

##### 注解

设计一个有用的概念是很有挑战性的。

##### 强制实施

很难。

* 查找无约束的参数，使用“非常规”或非标准的概念的模板，以及使用“自造的”又没有公理的概念的目标。
* 开发一种概念识别工具（例如，参考[一种早期实验](http://www.stroustrup.com/sle2010_webversion.pdf)）。

### <a id="rt-auto"></a>T.12: 对于局部变量，优先采用概念名而不是 `auto`

##### 理由

`auto` 是最弱的概念。概念的名字会比仅用 `auto` 传达出更多的意义。

##### 示例

    vector<string> v{ "abc", "xyz" };
    auto& x = v.front();        // 不好
    String auto& s = v.front(); // 好（String 是 GSL 的一个概念）

##### 强制实施

* ???

### <a id="rt-shorthand"></a>T.13: 对于简单的单类型参数概念，优先采用简写形式

##### 理由

可读性。直接表达意图。

##### 示例

这样来表达“`T` 是一种 `sortable`”：

    template<typename T>       // 正确但很啰嗦：“参数的类型
        requires sortable<T>   // 为 T，这是某个 sortable
    void sort(T&);             // 类型的名字”

    template<sortable T>       // 有改善：“参数的类型
    void sort(T&);             // 为 Sortable 的类型 T”

    void sort(sortable auto&); // 最佳方式：“参数为 sortable”

越简练的版本越符合我们的说话方式。注意许多模板不在需要使用 `template` 关键字了。

##### 强制实施

* 当人们从 `<typename T>` and `<class T>` 写法进行转换时，使用简短形式是不可行的。
* 之后，如果声明中首先引入了一个 `typename`，之后又用简单的单类型参数概念对其进行约束的话，就对其进行标记。

## <a id="ss-concepts-def"></a>T.concepts.def: 概念定义规则

定义恰当的概念并不简单。
概念是用于表现应用领域中的基本概念的（正如其名“概念”）。
只是把用在某个具体的类或算法的参数上的一组语法约束聚在一起，并不是概念所设计的用法，
也无法获得这个机制的全部好处。

显然，定义概念对于那些可以使用某个实现（比如 C++20 或更新版本）的代码是最有用的，
不过定义概念本身就是一种有益的设计技巧，有助于发现概念上的错误并清理实现中的各种概念。

### <a id="rt-low"></a>T.20: 避免没有有意义的语义的“概念”

##### 理由

概念是用于表现语义的观念的，比如“数”，元素的“范围”，以及“全序的”等等。
简单的约束，比如“带有 `+` 运算符”和“带有 `>` 运算符”，是无法独立进行有意义的运用的，
而仅应当被用作有意义的概念的构造块，而不是在用户代码中使用。

##### 示例，不好

    template<typename T>
    // 不好，不充分
    concept Addable = requires(T a, T b) { a + b; };

    template<Addable N>
    auto algo(const N& a, const N& b) // 使用两个数值
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = algo(x, y);   // z = 16

    string xx = "7";
    string yy = "9";
    auto zz = algo(xx, yy);   // zz = "79"

也许拼接是有意进行的。不过更可能的是一种意外。而对减法进行同等的定义则会导致可接受类型的集合的非常不同。
这个 `Addable` 违反了加法应当可交换的数学法则：`a+b == b+a`。

##### 注解

给出有意义的语义的能力，在于定义真正的概念的特征，而不是仅给出语法约束。

##### 示例

    template<typename T>
    // 假定数值的运算符 +、-、* 和 / 都遵循常规的数学法则
    concept Number = requires(T a, T b) { a + b; a - b; a * b; a / b; };

    template<Number N>
    auto algo(const N& a, const N& b)
    {
        // ...
        return a + b;
    }

    int x = 7;
    int y = 9;
    auto z = algo(x, y);   // z = 16

    string xx = "7";
    string yy = "9";
    auto zz = algo(xx, yy);   // 错误：string 不是 Number

##### 注解

带有多个操作的概念要远比单个操作的概念更少和类型发生意外匹配的机会。

##### 强制实施

* 对在其他 `concept` 的定义之外使用的但操作 `concept` 进行标记。
* 对表现为模拟单操作 `concept` 的 `enable_if` 的使用进行标记。


### <a id="rt-complete"></a>T.21: 为概念提出一组完整的操作要求

##### 理由

易于理解。
提升互操作性。
帮助实现者和维护者。

##### 注解

这是对一般性规则[必须让概念有语义上的意义](#rt-low)的一个专门的变体。

##### 示例，不好

    template<typename T> concept Subtractable = requires(T a, T b) { a - b; };

这个是没有语义作用的。
你至少还需要 `+` 来让 `-` 有意义和有用处。

完整集合的例子有

* `Arithmetic`: `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`
* `Comparable`: `<`, `>`, `<=`, `>=`, `==`, `!=`

##### 注解

无论我们是否使用了概念的直接语言支持，本条规则都适用。
这是一种一般性的设计规则，即便对于非模板也同样适用：

    class Minimal {
        // ...
    };

    bool operator==(const Minimal&, const Minimal&);
    bool operator<(const Minimal&, const Minimal&);

    Minimal operator+(const Minimal&, const Minimal&);
    // 没有其他运算符

    void f(const Minimal& x, const Minimal& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // 意外！错误

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // 意外！错误

        x = x + y;          // OK
        x += y;             // 意外！错误
    }

这是最小化的设计，但会使用户遇到意外或受到限制。
可能它还会比较低效。

这条规则支持这样的观点：概念应当反映（数学上）协调的一组操作。

##### 示例

    class Convenient {
        // ...
    };

    bool operator==(const Convenient&, const Convenient&);
    bool operator<(const Convenient&, const Convenient&);
    // ... 其他比较运算符 ...

    Convenient operator+(const Convenient&, const Convenient&);
    // ... 其他算术运算符 ...

    void f(const Convenient& x, const Convenient& y)
    {
        if (!(x == y)) { /* ... */ }    // OK
        if (x != y) { /* ... */ }       // OK

        while (!(x < y)) { /* ... */ }  // OK
        while (x >= y) { /* ... */ }    // OK

        x = x + y;     // OK
        x += y;        // OK
    }

定义所有的运算符也许很麻烦，但并不困难。
理想情况下，语言应当默认提供比较运算符以支持这条规则。

##### 强制实施

* 如果类所支持的运算符是运算符集合的“奇异”子集，比如有 `==` 但没有 `!=` 或者有 `+` 但没有 `-`，就对其进行标记。
  确实，`std::string` 也是“奇异”的，但要修改它太晚了。


### <a id="rt-axiom"></a>T.22: 为概念指明公理

##### 理由

有意义或有用的概念都有语义上的含义。
以非正式、半正式或正式的方式表达这些语义可以使概念对于读者更加可理解，而且对其进行表达的工作也能发现一些概念上的错误。
对语义的说明是一种强大的设计工具。

##### 示例

    template<typename T>
        // 假定数值的运算符 +、-、* 和 / 都遵循常规的数学法则
        // axiom(T a, T b) { a + b == b + a; a - a == 0; a * (b + c) == a * b + a * c; /*...*/ }
        concept Number = requires(T a, T b) {
            { a + b } -> convertible_to<T>;
            { a - b } -> convertible_to<T>;
            { a * b } -> convertible_to<T>;
            { a / b } -> convertible_to<T>;
        };

##### 注解

这是一种数学意义上的公理：可以假定成立而无需证明。
通常公理是无法证明的，而即便可以证明，通常也超出了编译器的能力。
一条公理也许并不是通用的，不过模板作者可以假定其对于其所实际使用的所有输入均成立（类似于一种前条件）。

##### 注解

这种上下文中的公理都是布尔表达式。
参见 [Palo Alto TR](#s-references) 中的例子。
当前，C++ 并不支持公理（即便 ISO Concepts TS 也不支持），我们不得不长期用代码注释来给出它们。
语言支持一旦出现，公理前面的 `//` 就可以删掉了。

##### 注解

GSL 中的概念都具有恰当定义的语义；请参见 Palo Alto TR 和 Ranges TS。

##### 例外

一个处于开发之中的新“概念”的早期版本，经常会仅仅定义了一组简单的约束而并没有恰当指定的语义。
为其寻找正确的语义可能是很费功夫和时间的。
不过不完整的约束集合仍然是很有用的：

    // 对于一般二叉树的平衡器
    template<typename Node> concept Balancer = requires(Node* p) {
        add_fixup(p);
        touch(p);
        detach(p);
    };

这样 `Balancer` 就必须为树的 `Node` 至少提供这些操作，
但我们还是无法指定详细的语义，因为一种新种类的平衡树可能需要更多的操作，
而在设计的早期阶段，很难确定把对于所有节点的精确的一般语义所确定下来。

不完整的或者没有恰当指定的语义的“概念”仍然是很有用的。
比如说，它可能允许在初始的试验之中进行某些检查。
不过，请不要把它当作是稳定的。
它的每个新的用例都可能导致不完整概念的改善。

##### 强制实施

* 在概念定义的代码注释中寻找单词“axiom”。

### <a id="rt-refine"></a>T.23: 通过添加新的使用模式，从更一般情形的概念中区分出提炼后的概念

##### 理由

否则编译器是无法自动对它们进行区分的。

##### 示例

    template<typename I>
    // 注：<iterator> 中定义了 input_iterator
    concept Input_iter = requires(I iter) { ++iter; };

    template<typename I>
    // 注：<iterator> 中定义了 forward_iterator
    concept Fwd_iter = Input_iter<I> && requires(I iter) { iter++; };

编译器可以基于所要求的操作的集合（这里为前缀 `++`）来确定提炼关系。
这样做减少了这些类型的实现者的负担，
因为他们不再需要任何特殊的声明来“打入概念内部”了。
如果两个概念具有完全相同的要求的话，它们在逻辑上就是等价的（不存在提炼）。

##### 强制实施

* 对与已经出现的另一个概念具有完全相同的要求的概念进行标记（它们中不存在更精炼的概念）。
  为对它们进行区分，参见 [T.24](#rt-tag)。

### <a id="rt-tag"></a>T.24: 用标签类或特征类来区分仅在语义上存在差别的概念

##### 理由

要求相同的语法但具有不同语义的两个概念之间会造成歧义，除非程序员对它们进行区分。

##### 示例

    template<typename I>    // 提供随机访问的迭代器
    // 注：<iterator> 中定义了 random_access_iterator
    concept RA_iter = ...;

    template<typename I>    // 提供对连续数据的随机访问的迭代器
    // 注：<iterator> 中定义了 contiguous_iterator
    concept Contiguous_iter =
        RA_iter<I> && is_contiguous_v<I>;  // 使用 is_contiguous 特征

程序员（在程序库中）必须适当地定义（特征） `is_contiguous`。

把标签类包装到概念中可以得到这个方案的更简单的表达方式：

    template<typename I> concept Contiguous = is_contiguous_v<I>;

    template<typename I>
    concept Contiguous_iter = RA_iter<I> && Contiguous<I>;

程序员（在程序库中）必须适当地定义（特征） `is_contiguous`。

##### 注解

特征可以是特征类或者类型特征。
它们可以是用户定义的，或者标准库中的。
优先采用标准库中的特征。

##### 强制实施

* 编译器会将对相同的概念的有歧义的使用标记出来。
* 对相同的概念定义进行标记。

### <a id="rt-not"></a>T.25: 避免互补性的约束

##### 理由

清晰性。可维护性。
用否定来表达的具有互补要求的函数是很脆弱的。

##### 示例

最初，人们总会试图定义带有互补要求的函数：

    template<typename T>
        requires !C<T>    // 不好
    void f();

    template<typename T>
        requires C<T>
    void f();

这样会好得多：

    template<typename T>   // 通用模板
        void f();

    template<typename T>   // 用概念进行特化
        requires C<T>
    void f();

仅当 `C<T>` 无法满足时，编译器将会选择无约束的模板。
如果你并不想（或者无法）定义无约束版本的
`f()` 的话，你可以删掉它。

    template<typename T>
    void f() = delete;

编译器将会选取这个重载，或者给出一个适当的错误。

##### 注解

很不幸，互补约束在 `enable_if` 代码中很常见：

    template<typename T>
    enable_if<!C<T>, void>   // 不好
    f();

    template<typename T>
    enable_if<C<T>, void>
    f();


##### 注解

有时候会（不正确地）把对单个要求的互补约束当做是可以接受的。
不过，对于两个或更多的要求来说，所需要的定义的数量是按指数增长的（2，4，8，16，……）：

    C1<T> && C2<T>
    !C1<T> && C2<T>
    C1<T> && !C2<T>
    !C1<T> && !C2<T>

这样，犯错的机会也会倍增。

##### 强制实施

* 对带有 `C<T>` 和 `!C<T>` 约束的函数对进行标记。

### <a id="rt-use"></a>T.26: 优先采用使用模式而不是简单的语法来定义概念

##### 理由

其定义更可读，而且更直接地对应于用户需要编写的代码。
其中同时兼顾了类型转换。你再不需要记住所有的类型特征的名字。

##### 示例

你可能打算这样来定义概念 `Equality`：

    template<typename T> concept Equality = has_equal<T> && has_not_equal<T>;

显然，直接使用标准的 `equality_comparable` 要更好而且更容易，
但是——只是一个例子——如果你不得不定义这样的概念的话，应当这样：

    template<typename T> concept Equality = requires(T a, T b) {
        { a == b } -> std::convertible_to<bool>;
        { a != b } -> std::convertible_to<bool>;
        // axiom { !(a == b) == (a != b) }
        // axiom { a = b; => a == b }  // => 的意思是“意味着”
    };

而不是定义两个无意义的概念 `has_equal` 和 `has_not_equal` 仅用于帮助 `Equality` 的定义。
“无意义”的意思是我们无法独立地指定 `has_equal` 的语义。

##### 强制实施

???

## <a id="ss-temp-interface"></a>模板接口

这些年以来，使用模板的编程一直忍受着模板的接口及其实现之间的
微弱区分性。
在引入概念之前，这种区分是没有直接的语言支持的。
不过，模板的接口是一个关键概念——是用户和实现者之间的一种契约——而且应当进行周密的设计。

### <a id="rt-fo"></a>T.40: 使用函数对象向算法传递操作

##### 理由

函数对象比“普通”的函数指针能够向接口传递更多的信息。
一般来说，传递函数对象比传递函数指针能带来更好的性能。

##### 示例

    bool greater(double x, double y) { return x > y; }
    sort(v, greater);                                    // 函数指针：可能较慢
    sort(v, [](double x, double y) { return x > y; });   // 函数对象
    sort(v, std::greater{});                             // 函数对象

    bool greater_than_7(double x) { return x > 7; }
    auto x = find_if(v, greater_than_7);                 // 函数指针：不灵活
    auto y = find_if(v, [](double x) { return x > 7; }); // 函数对象：携带所需数据
    auto z = find_if(v, Greater_than<double>(7));        // 函数对象：携带所需数据

当然，也可以使用 `auto` 或概念来使这些函数通用化。例如：

    auto y1 = find_if(v, [](totally_ordered auto x) { return x > 7; }); // 要求一种有序类型
    auto z1 = find_if(v, [](auto x) { return x > 7; });                 // 期望类型带有 >

##### 注解

Lambda 会生成函数对象。

##### 注解

性能表现依赖于编译器和优化器技术。

##### 强制实施

* 标记以函数指针作为模板参数。
* 标记将函数指针作为模板的参数进行传递（存在误报风险）。


### <a id="rt-essential"></a>T.41: 在模板的概念上仅提出基本的性质要求

##### 理由

保持接口的简单和稳定。

##### 示例

考虑一个带有（过度简化的）简单调试支持的 `sort`：

    void sort(sortable auto& s)  // 对序列 s 进行排序
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }

是否该把它重写为：

    template<sortable S>
        requires Streamable<S>
    void sort(S& s)  // 对序列 s 进行排序
    {
        if (debug) cerr << "enter sort( " << s <<  ")\n";
        // ...
        if (debug) cerr << "exit sort( " << s <<  ")\n";
    }

毕竟，`sortable` 里面并没有要求任何 `iostream` 支持。
而另一方面，在排序的基本概念中也没有任何东西是有关于调试的。

##### 注解

如果我们要求把用到的所有操作都在要求部分中列出的话，接口就会变得不稳定：
每当我们改动了调试设施，使用情况数据收集，测试支持，错误报告等等，
模板的定义就都得进行改动，而模板的所有使用方都不得不进行重新编译。
这样做很是累赘，而且在某些环境下是不可行的。

与之相反，如果我们在实现中使用了某个并未被概念检查提供保证的操作的话，
我们可能会遇到一个延后的编译时错误。

通过不对模板参数的那些不被认为是基本的性质进行概念检查，
我们把其检查推迟到了进行实例化的时候。
我们认为这是值得做出的折衷。

注意，使用非局部的，非待决的名字（比如 `debug` 和 `cerr`），同样会引入可能导致“神秘的”错误的某些上下文依赖。

##### 注解

可能很难确定一个类型的哪些性质是基本的，而哪些不是。

##### 强制实施

???

### <a id="rt-alias"></a>T.42: 使用模板别名来简化写法并隐藏实现细节

##### 理由

提升可读性。
隐藏实现。
注意，模板别名取代了许多用于计算类型的特征。
它们也可以用于封装一个特征。

##### 示例

    template<typename T, size_t N>
    class Matrix {
        // ...
        using Iterator = typename std::vector<T>::iterator;
        // ...
    };

这免除了 `Matrix` 的用户必须了解其元素是存储于 `vector` 之中，而且也免除了用户重复书写 `typename std::vector<T>::`。

##### 示例

    template<typename T>
    void user(T& c)
    {
        // ...
        typename container_traits<T>::value_type x; // 不好，啰嗦
        // ...
    }

    template<typename T>
    using Value_type = typename container_traits<T>::value_type;


这免除了 `Value_type` 的用户必须了解用于实现 `value_type` 的技术。

    template<typename T>
    void user2(T& c)
    {
        // ...
        Value_type<T> x;
        // ...
    }

##### 注解

一种简洁的常用说法是：“包装特征！”

##### 强制实施

* 将 `using` 声明之外用于消除歧义的 `typename` 进行标记。
* ???

### <a id="rt-using"></a>T.43: 优先使用 `using` 而不是 `typedef` 来定义别名

##### 理由

提升可读性：使用 `using` 时，新名字在前面，而不是被嵌在声明中的什么地方。
通用性：`using` 可用于模板别名，而 `typedef` 无法轻易作为模板。
一致性：`using` 在语法上和 `auto` 相似。

##### 示例

    typedef int (*PFI)(int);   // OK, 但很别扭

    using PFI2 = int (*)(int);   // OK, 更好

    template<typename T>
    typedef int (*PFT)(T);      // 错误

    template<typename T>
    using PFT2 = int (*)(T);   // OK

##### 强制实施

* 标记 `typedef` 的使用。不过这样会出现大量的“命中” :-(

### <a id="rt-deduce"></a>T.44: （如果可行）使用函数模板来对类模板的参数类型进行推断

##### 理由

显式写出模板参数类型既麻烦又无必要。

##### 示例

    tuple<int, string, double> t1 = {1, "Hamlet", 3.14};   // 明确类型
    auto t2 = make_tuple(1, "Ophelia"s, 3.14);         // 更好；推断类型

注意这里用 `s` 后缀来确保字符串是 `std::string` 而不是 C 风格字符串。

##### 注解

既然你可以轻易写出 `make_T` 函数，编译器也可以。以后 `make_T` 函数也许将会变得多余。

##### 例外

有时候没办法对模板参数进行推断，而有时候你则想要明确指定参数：

    vector<double> v = { 1, 2, 3, 7.9, 15.99 };
    list<Record*> lst;

##### 注解

注意，C++17 强允许模板参数直接从构造函数参数进行推断，而使这条规则变得多余：
[构造函数的模板形参推断(Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0091r1.html)。
例如：

    tuple t1 = {1, "Hamlet"s, 3.14}; // 推断为：tuple<int, string, double>

##### 强制实施

当显式指定的类型与所使用的类型精确匹配时进行标记。

### <a id="rt-regular"></a>T.46: 要求模板参数至少是半正规的

##### 理由

可读性。
避免意外和错误。
大多数用法都支持这样做。

##### 示例

    class X {
    public:
        explicit X(int);
        X(const X&);            // 复制
        X operator=(const X&);
        X(X&&) noexcept;        // 移动
        X& operator=(X&&) noexcept;
        ~X();
        // ... 没有别的构造函数了 ...
    };

    X x {1};              // 没问题
    X y = x;              // 没问题
    std::vector<X> v(10); // 错误: 没有默认构造函数

##### 注解

`SemiRegular` 要求可以默认构造。

##### 强制实施

* 对并非至少为 `SemiRegular` 的类型进行标记。

### <a id="rt-visible"></a>T.47: 避免用常用名字命名高度可见的无约束模板

##### 示例

无约束的模板参数和任何东西都能完全匹配，因此这样的模板相对于需要少量转换的更特定类型来说可能更优先。
而当使用 ADL 时，这一点将会更加麻烦和危险。
而常用的名字则会让这种问题更易于出现。

##### 示例

    namespace Bad {
        struct S { int m; };
        template<typename T1, typename T2>
        bool operator==(T1, T2) { cout << "Bad\n"; return true; }
    }

    namespace T0 {
        bool operator==(int, Bad::S) { cout << "T0\n"; return true; }  // 与 int 比较

        void test()
        {
            Bad::S bad{ 1 };
            vector<int> v(10);
            bool b = 1 == bad;
            bool b2 = v.size() == bad;
        }
    }

这将会打印出 `T0` 和 `Bad`。

这里 `Bad` 中的 `==` 有意设计为造成问题，不过你是否在真实代码中发现过这个问题呢？
问题在于 `v.size()` 返回的是 `unsigned` 整数，因此需要进行转换才能调用局部的 `==`；
而 `Bad` 中的 `=` 则不需要任何转换。
实际的类型，比如标准库的迭代器，也可以被弄成类似的反社会倾向。

##### 注解

如果在相同的命名空间中定义了一个无约束模板类型，
这个无约束模板是可以通过 ADL 所找到的（如例子中所发生的一样）。
就是说，它是高度可见的。

##### 注解

这条规则应当是没有必要的，不过委员会在是否将无约束模板从 ADL 中排除上无法达成一致。

不幸的是，这会导致大量的误报；标准库也大量地违反了这条规则，它将许多无约束模板和类型都放入了单一的命名空间 `std` 之中。


##### 强制实施

如果定义模板的命名空间中同样定义了具体的类型，就对其进行标记（可能在我们有概念支持之前都是不可行的）。


### <a id="rt-concept-def"></a>T.48: 如果你的编译器不支持概念的话，可以用 `enable_if` 来模拟

##### 理由

因为这是我们没有直接的概念支持时能够做到的最好的方式。
`enable_if` 可被用于有条件地定义函数，以及用于在一组函数中进行选择。

##### 示例

    template<typename T>
    enable_if_t<is_integral_v<T>>
    f(T v)
    {
        // ...
    }

    // 等价于：
    template<Integral T>
    void f(T v)
    {
        // ...
    }

##### 注解

请当心[互补约束](#rt-not)。
当用 `enable_if` 来模拟概念重载时，有时候会迫使我们使用这种易错的设计技巧。

##### 强制实施

???

### <a id="rt-erasure"></a>T.49: 尽可能避免类型擦除

##### 理由

类型擦除通过在一个独立的编译边界之后隐藏类型信息而招致一层额外的间接。

##### 示例

    ???

**例外**: 有时候类型擦除是合适的，比如 `std::function`。

##### 强制实施

???


##### 注解


## <a id="ss-temp-def"></a>T.def: 模板定义

模板的定义式（无论是类还是函数）都可能包含任意的代码，因而只有相当范围的 C++ 编程技术才能覆盖这个主题。
不过，这个部分所关注的是特定于模板的实现的问题。
特别地说，这里关注的是模板定义式对其上下文之间的依赖。

### <a id="rt-depend"></a>T.60: 最小化模板的上下文依赖性

##### 理由

便于理解。
最小化源于意外依赖的错误。
便于创建工具。

##### 示例

    template<typename C>
    void sort(C& c)
    {
        std::sort(begin(c), end(c)); // 必要并且有意义的依赖
    }

    template<typename Iter>
    Iter algo(Iter first, Iter last)
    {
        for (; first != last; ++first) {
            auto x = sqrt(*first); // 潜在的意外依赖：哪个 sqrt()？
            helper(first, x);      // 潜在的意外依赖：
                                   // helper 是基于 first 和 x 所选择的
            TT var = 7;            // 潜在的意外依赖：哪个 TT？
        }
    }

##### 注解

通常模板是放在头文件中的，因此相对于 `.cpp` 文件之中的函数来说，其上下文依赖更加会受到 `#include` 的顺序依赖的威胁。

##### 注解

让模板仅对其参数进行操作是一种把依赖减到最少的方式，但通常这是很难做到的。
比如说，一个算法通常会使用其他的算法，并且调用一些并非仅在参数上做动作的一些操作。
而且别再让我们从宏开始干活！

**参见**：[T.69](#rt-customization)。

##### 强制实施

??? 很麻烦

### <a id="rt-scary"></a>T.61: 请勿对成员进行过度参数化（SCARY）

##### 理由

不依赖于模板参数的成员，除非给定某个特定的模板参数，否则也是无法使用的。
这样会限制其使用，而且通常会增加代码大小。

##### 示例，不好

    template<typename T, typename A = std::allocator<T>>
        // requires Regular<T> && Allocator<A>
    class List {
    public:
        struct Link {   // 并未依赖于 A
            T elem;
            Link* pre;
            Link* suc;
        };

        using iterator = Link*;

        iterator first() const { return head; }

        // ...
    private:
        Link* head;
    };

    List<int> lst1;
    List<int, My_allocator> lst2;

这看起来没什么问题，但现在 `Link` 形成依赖于分配器（尽管它不使用分配器）。 这迫使冗余的实例化在某些现实场景中可能造成出奇的高的成本。
通常，解决方案是使用自己的最小模板参数集使嵌套类非局部化。

    template<typename T>
    struct Link {
        T elem;
        Link* pre;
        Link* suc;
    };

    template<typename T, typename A = std::allocator<T>>
        // requires Regular<T> && Allocator<A>
    class List2 {
    public:
        using iterator = Link<T>*;

        iterator first() const { return head; }

        // ...
    private:
        Link<T>* head;
    };

    List2<int> lst1;
    List2<int, My_allocator> lst2;

人们发现 `Link` 不再隐藏在列表中很可怕，所以我们命名这个技术为 [SCARY](http://www.open-std.org/jtc1/sc22/WG21/docs/papers/2009/n2911.pdf)。
引自该学术论文：“首字母缩略词 SCARY 描述了看似错误的赋值和初始化（受冲突的通用参数的约束），
但实际上使用了正确的实现（由于最小化的依赖而不受冲突的约束）。”

##### 注解

这同样适用于那些并不依赖于其全部模板形参的 lambda 表达式。

##### 强制实施

* 对并未依赖于全部模板形参的成员类型进行标记。
* 对并未依赖于全部模板形参的成员函数进行标记。
* 对并未依赖于全部模板形参的 lambda 表达式或变量模板进行标记。

### <a id="rt-nondependent"></a>T.62: 将无依赖的类模板成员置于一个非模板基类之中

##### 理由

可以在使用基类成员时不需要指定模板参数，也不需要模板实例化。

##### 示例

    template<typename T>
    class Foo {
    public:
        enum { v1, v2 };
        // ...
    };

???

    struct Foo_base {
        enum { v1, v2 };
        // ...
    };

    template<typename T>
    class Foo : public Foo_base {
    public:
        // ...
    };

##### 注解

这条规则的更一般化的版本是，
“如果类模板的成员依赖于 M 个模板参数中的 N 个，就将它置于只有 N 个参数的基类之中。”
当 N == 1 时，可以如同 [T.61](#rt-scary) 一样在一个基类和其外围作用域中的一个类之间进行选择。

??? 常量的情况如何？类的静态成员呢？

##### 强制实施

* 标记 ???

### <a id="rt-specialization"></a>T.64: 用特化来提供类模板的其他实现

##### 理由

模板定义了通用接口。
特化是一种为这个接口提供替代实现的强大机制。

##### 示例

    ??? 字符串的特化 (==)

    ??? 表示特化？

##### 注解

???

##### 强制实施

???

### <a id="rt-tag-dispatch"></a>T.65: 用标签分派来提供函数的其他实现

##### 理由

* 模板定义了通用接口。
* 标签派发允许我们基于参数类型的特定性质选择不同的实现。
* 性能。

##### 示例

这是 `std::copy` 的一个简化版本（忽略了非连续序列的可能性）

    struct pod_tag {};
    struct non_pod_tag {};

    template<class T> struct copy_trait { using tag = non_pod_tag; };   // T 不是“朴素数据”

    template<> struct copy_trait<int> { using tag = pod_tag; };         // int 是“朴素数据”

    template<class Iter>
    Out copy_helper(Iter first, Iter last, Iter out, pod_tag)
    {
        // 使用 memmove
    }

    template<class Iter>
    Out copy_helper(Iter first, Iter last, Iter out, non_pod_tag)
    {
        // 使用调用复制构造函数的循环
    }

    template<class Iter>
    Out copy(Iter first, Iter last, Iter out)
    {
        return copy_helper(first, last, out, typename copy_trait<Value_type<Iter>>::tag{})
    }

    void use(vector<int>& vi, vector<int>& vi2, vector<string>& vs, vector<string>& vs2)
    {
        copy(vi.begin(), vi.end(), vi2.begin()); // 使用 memmove
        copy(vs.begin(), vs.end(), vs2.begin()); // 使用调用复制构造函数的循环
    }

这是一种进行编译时算法选择的通用且有力的技巧。

##### 注解

当可以广泛使用 `concept` 之后，这样的替代实现就可以直接进行区分了：

    template<class Iter>
        requires Pod<Value_type<Iter>>
    Out copy_helper(In, first, In last, Out out)
    {
        // 使用 memmove
    }

    template<class Iter>
    Out copy_helper(In, first, In last, Out out)
    {
        // 使用调用复制构造函数的循环
    }

##### 强制实施

???


### <a id="rt-specialization2"></a>T.67: 用特化来提供不规则类型的其他实现

##### 理由

 ???

##### 示例

    ???

##### 强制实施

???

### <a id="rt-cast"></a>T.68: 在模板中用 `{}` 而不是 `()` 以避免歧义

##### 理由

`()` 会带来文法歧义。

##### 示例

    template<typename T, typename U>
    void f(T t, U u)
    {
        T v1(T(u));    // 错误：啊，v1 是函数而不是变量
        T v2{u};       // 清晰：显然是变量
        auto x = T(u); // 不清晰：构造还是强制转换？
    }

    f(1, "asdf"); // 不好：从 const char* 强制转换为 int

##### 强制实施

* 标记 `()` 初始化式。
* 标记函数风格的强制转换。


### <a id="rt-customization"></a>T.69: 在模板中，请勿进行未限定的非成员函数调用，除非有意将之作为定制点

##### 理由

* 仅提供预计之内的灵活性。
* 避免源于意外的环境改变的威胁。

##### 示例

主要有三种方法使调用代码对模板进行定制化。

    template<class T>
        // 调用成员函数
    void test1(T t)
    {
        t.f();    // 要求 T 提供 f()
    }

    template<class T>
    void test2(T t)
        // 不带限定地调用非成员函数
    {
        f(t);     // 要求 f(/*T*/) 在调用方的作用域或者 T 的命名空间中可用
    }

    template<class T>
    void test3(T t)
        // 调用一个“特征”
    {
        test_traits<T>::f(t); // 要求定制化 test_traits<>
                              // 以获得非默认的函数和类型
    }

特征通常是用以计算一个类型的类型别名，
用以计算一个值的 `constexpr` 函数，
或者针对用户的类型进行特化的传统的特征模板。

##### 注解

当你打算为依赖于某个模板类型参数的值 `t` 调用自己的辅助函数 `helper(t)` 时，
请将函数放入一个 `::detail` 命名空间中，并把调用限定为 `detail::helper(t);`。
无限定的调用将成为一个定制点，它将会调用处于 `t` 的类型所在命名空间中的任何 `helper` 函数；
这可能会导致诸如[意外地调用了无约束函数模板](#rt-visible)这样的问题。


##### 强制实施

* 在模板中，如果非成员函数的无限定调用传递了具有依赖类型的变量，而在该模板的命名空间中存在相同名字的非成员函数，则对其进行标记。


## <a id="ss-temp-hier"></a>T.temp-hier: 模板和类型层次规则：

模板是 C++ 为泛型编程提供支持的基石，而类层次则是 C++ 为面向对象编程提供
支持的基石。
这两种语言机制可以有效地组合起来，但必须避免一些设计上的陷阱。

### <a id="rt-hier"></a>T.80: 请勿不成熟地对类层次进行模板化

##### 理由

使一个带有许多函数，尤其是有许多虚函数的类层次进行模板化，会导致代码膨胀。

##### 示例，不好

    template<typename T>
    struct Container {         // 这是一个接口
        virtual T* get(int i);
        virtual T* first();
        virtual T* next();
        virtual void sort();
    };

    template<typename T>
    class Vector : public Container<T> {
    public:
        // ...
    };

    Vector<int> vi;
    Vector<string> vs;

把 `sort` 定义为容器的成员函数可能是一个比较糟糕的主意，不过这样做并不鲜见，而且它是一个展示不应当做的事情的好例子。

在这之中，编译器不知道 `Vector<int>::sort()` 是不是会被调用，因此它必须为之生成代码。
`Vector<string>::sort()` 也与此相似。
除非这两个函数被调用，否则这就是代码爆炸。
不难想象当一个带有几十个成员函数和几十个派生类的类层次被大量实例化时会怎么样。

##### 注解

许多情况下都可以通过不为基类进行参数化而提供一个稳定的接口；
参见[“稳定的基类”](#rt-abi)和 [OO 与 GP](#rt-generic-oo)。

##### 强制实施

* 对依赖于模板参数的虚函数进行标记。 ??? 误报

### <a id="rt-array"></a>T.81: 请勿混合类层次和数组

##### 理由

派生类的数组可以隐式地“退化”成指向基类的指针，并带来潜在的灾难性后果。

##### 示例

假定 `Apple` 和 `Pear` 是两种 `Fruit`。

    void maul(Fruit* p)
    {
        *p = Pear{};     // 把一个 Pear 放入 *p
        p[1] = Pear{};   // 把一个 Pear 放入 p[1]
    }

    Apple aa [] = { an_apple, another_apple };   // aa 包含的是 Apple （显然！）

    maul(aa);
    Apple& a0 = &aa[0];   // 是 Pear 吗？
    Apple& a1 = &aa[1];   // 是 Pear 吗？

`aa[0]` 可能会变为 `Pear`（并且没进行过强制转换！）。
当 `sizeof(Apple) != sizeof(Pear)` 时，对 `aa[1]` 的访问就是并未跟数组中的对象的适当起始位置进行对齐的。
这里出现了类型违例，以及很可能出现的内存损坏。
决不要写这样的代码。

注意，`maul()` 违反了 [`T*` 应指向独立对象的规则](#rf-ptr)。

**替代方案**: 使用适当的（模板化）容器：

    void maul2(Fruit* p)
    {
        *p = Pear{};   // 把一个 Pear 放入 *p
    }

    vector<Apple> va = { an_apple, another_apple };   // va 包含的是 Apple （显然！）

    maul2(va);       // 错误: 无法把 vector<Apple> 转换为 Fruit*
    maul2(&va[0]);   // 这是你明确要做的

    Apple& a0 = &va[0];   // 是 Pear 吗？

注意，`maul2()` 中的赋值违反了[避免发生切片的规则](#res-slice)。

##### 强制实施

* 对这种恐怖的东西进行检测！

### <a id="rt-linear"></a>T.82: 当不想要虚函数时，可以将类层次线性化

##### 理由

 ???

##### 示例

    ???

##### 强制实施

???

### <a id="rt-virtual"></a>T.83: 请勿声明虚的成员函数模板

##### 理由

C++ 是不支持这样做的。
如果支持的话，就只能等到连接时才能生成 VTBL 了。
而且一般来说，各个实现还要搞定动态连接的问题。

##### 示例，请勿如此

    class Shape {
        // ...
        template<class T>
        virtual bool intersect(T* p);   // 错误：模板不能为虚
    };

##### 注解

我们保留这条规则是因为人们总是问这个问题。

##### 替代方案

双派发或访问器模式，或者计算出所要调用的函数。

##### 强制实施

编译器会处理这个问题。

### <a id="rt-abi"></a>T.84: 使用非模板的核心实现来提供 ABI 稳定的接口

##### 理由

提升代码的稳定性。
避免代码爆炸。

##### 示例

这个应当是基类：

    struct Link_base {   // 稳定
        Link_base* suc;
        Link_base* pre;
    };

    template<typename T>   // 模板化的包装带来了类型安全性
    struct Link : Link_base {
        T val;
    };

    struct List_base {
        Link_base* first;   // 第一个元素（如果有）
        int sz;             // 元素数量
        void add_front(Link_base* p);
        // ...
    };

    template<typename T>
    class List : List_base {
    public:
        void put_front(const T& e) { add_front(new Link<T>{e}); }   // 隐式强制转换为 Link_base
        T& front() { static_cast<Link<T>*>(first).val; }   // 显式强制转换回 Link<T>
        // ...
    };

    List<int> li;
    List<string> ls;

这样的话就只有一份用于对 `List` 的元素进行入链和解链的操作的代码了。
而类 `Link` 和 `List` 除了进行类型操作之外什么也没做。

除了使用一个独立的“base”类型外，另一种常用的技巧是对 `void` 或 `void*` 进行特化，并让针对 `T` 的通用模板成为在从或向 `void` 的核心实现进行强制转换的一层类型安全封装。

**替代方案**: 使用一个 [PImpl](#ri-pimpl) 实现。

##### 强制实施

???

## <a id="ss-variadic"></a>T.var: 变参模板规则

???

### <a id="rt-variadic"></a>T.100: 当需要可以接受可变数量的多种类型参数的函数时，使用变参模板

##### 理由

变参模板是做到这点的最通用的机制，而且既高效又类型安全。请不要使用 C 的变参。

##### 示例

    ??? printf

##### 强制实施

* 对用户代码中 `va_arg` 的使用进行标记。

### <a id="rt-variadic-pass"></a>T.101: ??? 如何向变参模板传递参数 ???

##### 理由

 ???

##### 示例

    ??? 当心仅能移动参数和引用参数

##### 强制实施

???

### <a id="rt-variadic-process"></a>T.102: 如何处理变参模板的参数

##### 理由

 ???

##### 示例

    ??? 转发参数，类型检查，引用

##### 强制实施

???

### <a id="rt-variadic-not"></a>T.103: 请勿对同质参数列表使用变参模板

##### 理由

存在更加正规的给出同质序列的方式，比如使用 `initializer_list`。

##### 示例

    ???

##### 强制实施

???

## <a id="ss-meta"></a>T.meta: 模板元编程（TMP）

模板提供了一种编译期编程的通用机制。

元编程，是其中至少一项输入或者一项输出是类型的编程。
模板提供了编译期的图灵完备（除了内存消耗外）的鸭子类型系统。
其语法和所需技巧都相当可怕。

### <a id="rt-metameta"></a>T.120: 仅当确实需要时才使用模板元编程

##### 理由

模板元编程很难做对，它会拖慢编译速度，而且通常很难维护。
不过，现实世界有些例子中模板元编程提供了比其他专家级的汇编代码替代方案还要更好的性能。
而且，也存在现实世界的例子用模板元编程做到比运行时代码更好地表达基本设计意图的情况。
例如，当确实需要在编译期进行 AST 操作时，（比如说对矩阵操作进行可选的折叠），C++ 中可能没有其他的实现方式。

##### 示例，不好

    ???

##### 示例，不好

    enable_if

请使用概念来替代它。不过请参见[如何在没有语言支持时模拟概念](#rt-emulate)。

##### 示例

    ??? 好例子

**替代方案**: 如果结果是一个值而不是类型，请使用 [`constexpr` 函数](#rt-fct)。

##### 注解

当你觉得需要把模板元编程代码隐藏到宏之中时，你可能已经跑得太远了。

### <a id="rt-emulate"></a>T.121: 模板元编程主要用于模拟概念机制

##### 理由

不能使用 C++20 时，我们需要用 TMP 来模拟它。
对概念给出要求的用例（比如基于概念进行重载）是 TMP 的最常见（而且最简单）的用法。

##### 示例

    template<typename Iter>
        /*requires*/ enable_if<random_access_iterator<Iter>, void>
    advance(Iter p, int n) { p += n; }

    template<typename Iter>
        /*requires*/ enable_if<forward_iterator<Iter>, void>
    advance(Iter p, int n) { assert(n >= 0); while (n--) ++p;}

##### 注解

这种代码使用概念时将更加简单：

    void advance(random_access_iterator auto p, int n) { p += n; }

    void advance(forward_iterator auto p, int n) { assert(n >= 0); while (n--) ++p;}

##### 强制实施

???

### <a id="rt-tmp"></a>T.122: 用模板（通常为模板别名）来在编译期进行类型运算

##### 理由

模板元编程是在编译期进行类型生成的受到直接支持和部分正规化的唯一方式。

##### 注解

“特征（Trait）”技术基本上在计算类型方面被模板别名所代替，而在计算值方面则被 `constexpr` 函数所代替。

##### 示例

    ??? 大型对象 / 小型对象的优化

##### 强制实施

???

### <a id="rt-fct"></a>T.123: 用 `constexpr` 函数来在编译期进行值运算

##### 理由

函数是用于表达计算一个值的最显然和传统的方式。
通常 `constexpr` 函数都比其他的替代方式具有更少的编译期开销。

##### 注解

“特征（Trait）”技术基本上在计算类型方面被模板别名所代替，而在计算值方面则被 `constexpr` 函数所代替。

##### 示例

    template<typename T>
        // requires Number<T>
    constexpr T pow(T v, int n)   // 幂/指数
    {
        T res = 1;
        while (n--) res *= v;
        return res;
    }

    constexpr auto f7 = pow(pi, 7);

##### 强制实施

* 对产生值的模板元程序进行标记。它们应当被替换成 `constexpr` 函数。

### <a id="rt-std-tmp"></a>T.124: 优先使用标准库的模板元编程设施

##### 理由

标准中所定义的设施，诸如 `conditional`，`enable_if`，以及 `tuple` 等，是可移植的，可以假定为大家所了解。

##### 示例

    ???

##### 强制实施

???

### <a id="rt-lib"></a>T.125: 当需要标准库之外的模板元编程设施时，使用某个现存程序库

##### 理由

要搞出高级的 TMP 设施是很难的，而使用一个库则可让你进入某个（有希望收到支持的）社区。
只有当确实不得不编写自己的“高级 TMP 支持”时，才应当这样做。

##### 示例

    ???

##### 强制实施

???

## <a id="ss-temp-other"></a>其他模板规则

### <a id="rt-name"></a>T.140: 若操作可被重用，则应为其命名

参见 [F.10](#rf-name)

### <a id="rt-lambda"></a>T.141: 当仅在一个地方需要一个简单的函数对象时，使用无名的 lambda

参见 [F.11](#rf-lambda)

### <a id="rt-var"></a>T.142?: 使用模板变量以简化写法

##### 理由

改善可读性。

##### 示例

    ???

##### 强制实施

???

### <a id="rt-nongeneric"></a>T.143: 请勿编写并非有意非泛型的代码

##### 理由

一般性。可重用性。请勿无必要地陷入技术细节之中；请使用最广泛可用的设施。

##### 示例

用 `!=` 而不是 `<` 来比较迭代器；`!=` 可以在更多对象上正常工作，因为它并未蕴含有序性。

    for (auto i = first; i < last; ++i) {   // 通用性较差
        // ...
    }

    for (auto i = first; i != last; ++i) {   // 好; 通用性较强
        // ...
    }

当然，范围式 `for` 在符合需求的时候当然是更好的选择。

##### 示例

使用能够提供所需功能的最接近基类的类。

    class Base {
    public:
        Bar f();
        Bar g();
    };

    class Derived1 : public Base {
    public:
        Bar h();
    };

    class Derived2 : public Dase {
    public:
        Bar j();
    };

    // 不好，除非确实有特别的原因来将之仅限制为 Derived1 对象
    void my_func(Derived1& param)
    {
        use(param.f());
        use(param.g());
    }

    // 好，仅使用 Base 的接口，且保证了这个类型
    void my_func(Base& param)
    {
        use(param.f());
        use(param.g());
    }

##### 强制实施

* 对使用 `<` 而不是 `!=` 的迭代器比较进行标记。
* 当存在 `x.empty()` 或 `x.is_empty()` 时，对 `x.size() == 0` 进行标记。`empty()` 比 `size()` 能够对于更多的容器工作，因为某些容器是不知道自己的大小的，甚至概念上就是大小无界的。
* 如果函数接受指向更加派生的类型的指针或引用，但仅使用了在某个基类中所声明的函数，则对其进行标记。

### <a id="rt-specialize-function"></a>T.144: 请勿特化函数模板

##### 理由

根据语言规则，函数模板是无法被部分特化的。函数模板可以被完全特化，不过你基本上需要的都是进行重载而不是特化——因为函数模板特化并不参与重载，它们的行为和你想要的可能是不同的。少数情况下，应当通过你可以进行适当特化的类模板来进行真正的特化。

##### 示例

    ???

**例外**: 当确实有特化函数模板的恰当理由时，请只编写一个函数模板，并使它委派给一个类模板，然后对这个类模板进行特化（这提供了编写部分特化的能力）。

##### 强制实施

* 标记出所有的函数模板特化。代之以函数重载。


### <a id="rt-check-class"></a>T.150: 用 `static_assert` 来检查类是否与概念相符

##### 理由

当你打算使一个类符合某个概念时，应该提早进行验证以减少麻烦。

##### 示例

    class X {
    public:
        X() = delete;
        X(const X&) = default;
        X(X&&) = default;
        X& operator=(const X&) = default;
        // ...
    };

在别的地方，也许是某个实现文件中，可以让编译器来检查 `X` 的所需各项性质：

    static_assert(Default_constructible<X>);    // 错误: X 没有默认构造函数
    static_assert(Copyable<X>);                 // 错误: 忘记定义 X 的移动构造函数了


##### 强制实施

不可行。
