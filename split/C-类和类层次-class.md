
# <a id="s-class"></a>C: 类和类层次

类是一种自定义类型，程序员可以定义它的表示，操作和接口。
类层次用于把相关的类组织到层次化的结构当中。

类的规则概览：

* [C.1: 把相关的数据组织到结构中（`struct` 或 `class`）](#rc-org)
* [C.2: 当类具有不变式时使用 `class`；当数据成员可以独立进行变动时使用 `struct`](#rc-struct)
* [C.3: 用类来表示接口和实现之间的区别](#rc-interface)
* [C.4: 仅当函数直接访问类的内部表示时才让函数作为其成员](#rc-member)
* [C.5: 把辅助函数放在其所支持的类相同的命名空间之中](#rc-helper)
* [C.7: 不要在同一个语句中同时定义类或枚举并声明该类型的变量](#rc-standalone)
* [C.8: 当有任何非公开成员时使用 `class` 而不是 `struct`](#rc-class)
* [C.9: 让成员的暴露最小化](#rc-private)

子章节：

* [C.concrete: 具体类型](#ss-concrete)
* [C.ctor: 构造函数，赋值和析构函数](#s-ctor)
* [C.con: 容器和其他资源包装](#ss-containers)
* [C.lambdas: 函数对象和 lambda](#ss-lambdas)
* [C.hier: 类层次（OOP）](#ss-hier)
* [C.over: 重载和运算符重载](#ss-overload)
* [C.union: 联合体](#ss-union)

### <a id="rc-org"></a>C.1: 把相关的数据组织到结构中（`struct` 或 `class`）

##### 理由

易理解性。
如果数据之间（以基本的原因而）相关，应当在代码中体现这点。

##### 示例

    void draw(int x, int y, int x2, int y2);  // 不好: 不必要的隐含式的关系
    void draw(Point from, Point to);          // 好多了

##### 注解

没有虚函数的简单的类是不会带来空间或时间开销的。

##### 注解

从语言的角度看，`class` 和 `struct` 的差别只有其成员的默认可见性不同。

##### 强制实施

也许不可能做到。也许对总是一起使用的数据项目进行启发式查找是一种可能方式。

### <a id="rc-struct"></a>C.2: 当类具有不变式时使用 `class`；当数据成员可以独立进行变动时使用 `struct`

##### 理由

可读性。
易理解性。
`class` 的使用会提醒程序员需要考虑不变式。
这是一种很有用的惯例。

##### 注解

不变式是对象的成员之间的一种逻辑条件，必须由构造函数建立，并由公开成员函数假定成员。
不变式一旦建立（通常是由构造函数），就可以对对象的各个成员函数进行调用了。
不变式既可以非正式地说明（比如在代码注释中），也可以正式地用 `Expects` 说明。

如果数据成员都可以互相独立地进行改变，则不可能存在不变式。

##### 示例

    struct Pair {  // 成员可以独立地变动
        string name;
        int volume;
    };

但是：

    class Date {
    public:
        // 验证 {yy, mm, dd} 是有效的日期并进行初始化
        Date(int yy, Month mm, char dd);
        // ...
    private:
        int y;
        Month m;
        char d;    // day
    };

##### 注解

如果一个类中有任何的 `private` 数据的话，其使用者就不可能不通过构造函数而完全初始化一个对象。
因此，类的定义者必然提供构造函数且必须明确其含义。
这就相当于表示该定义者需要定义一种不变式。

**参见**：

* [把带有私有数据的类定义为 `class`](#rc-class)
* [优先将接口部分放在类的开头](#rl-order)
* [使成员的暴露最小化](#rc-private)
* [避免 `protected` 数据](#rh-protected)

##### 强制实施

查找所有数据都私有的 `struct` 和带有公开成员的 `class`。

### <a id="rc-interface"></a>C.3: 用类来表示接口和实现之间的区别

##### 理由

接口和实现之间的明确区别能够提升可读性并简化维护工作。

##### 示例

    class Date {
    public:
        Date();
        // 验证 {yy, mm, dd} 是有效的日期并进行初始化
        Date(int yy, Month mm, char dd);

        int day() const;
        Month month() const;
        // ...
    private:
        // ... 一些内部表示 ...
    };

比如说，我们现在可以改变 `Date` 的表示而不对其使用者造成影响（虽然很可能需要重新编译）。

##### 注解

使用这样的类来表示接口和实现之间的区别当然不是唯一可能的方式。
比如说，我们也可以使用命名空间中的一组自由函数，一个抽象基类，或者一个带有概念的函数模板来表示一个接口。
最重要的一点，在于明确地把接口和其实现“细节”区分开来。
理想地，并且典型地，接口要比其实现稳定得多。

##### 强制实施

???

### <a id="rc-member"></a>C.4: 仅当函数直接访问类的内部表示时才让函数作为其成员

##### 理由

比成员函数更少的耦合，减少可能由于改动对象状态而造成问题的函数，减少每当改变内部表示时需要进行修改的函数数量。

##### 示例

    class Date {
        // ... 相对较小的接口 ...
    };

    // 辅助函数:
    Date next_weekday(Date);
    bool operator==(Date, Date);

这些“辅助函数”并不需要直接访问 `Date` 的内部表示。

##### 注解

当 C++ 带来[统一函数调用](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0251r0.pdf)之后，这条规则会更有效。

##### 例外

语言规定 `virtual` 函数为成员函数，而并非所有的 `virtual` 函数都会直接访问数据。
特别是，抽象类的成员很少这样做。

注意 [multi methods](https://web.archive.org/web/20200605021759/https://parasol.tamu.edu/~yuriys/papers/OMM10.pdf)。

##### 例外

语言规定运算符 `=`，`()`，`[]` 和 `->` 是成员函数。

##### 示例

一个重载集合中的一些成员可以不直接访问 `private` 数据：

    class Foobar {
    public:
        void foo(long x) { /* 操作 private 数据 */ }
        void foo(double x) { foo(std::lround(x)); }
        // ...
    private:
        // ...
    };

##### 例外

类似地，一组函数可以被设计为进行链式调用：

    x.scale(0.5).rotate(45).set_color(Color::red);

典型情况下，这些函数中的一些而并非全部会访问 `private` 数据。

##### 强制实施

* 寻找并不直接访问数据成员的非 `virtual` 成员函数。
麻烦的是由许多并不需要直接访问数据成员的函数也会这么做。
* 忽略 `virtual` 函数。
* 忽略至少包含一个访问了 `private` 成员的函数的重载集合中的函数。
* 忽略返回 `this` 的函数。

### <a id="rc-helper"></a>C.5: 把辅助函数放在其所支持的类相同的命名空间之中

##### 理由

辅助函数是（由类的作者提供的）并不需要直接访问类的内部表示的函数，它们也被当作是类的可用接口的一部分。
把它们和类放在相同的命名空间中，使它们与类的关系更明显，并允许通过基于参数的查找机制找到它们。

##### 示例

    namespace Chrono { // 我们在这里放置与时间有关的服务

        class Time { /* ... */ };
        class Date { /* ... */ };

        // 辅助函数:
        bool operator==(Date, Date);
        Date next_weekday(Date);
        // ...
    }

##### 注解

这点对于[重载运算符](#ro-namespace)来说尤其重要。

##### 强制实施

* 对接受某一个命名空间中的参数类型的全局函数进行标记。

### <a id="rc-standalone"></a>C.7: 不要在同一个语句中同时定义类或枚举并声明该类型的变量

##### 理由

在同一个声明式中混合类型的定义和另一个实体的定义会导致混淆，而且不是必要的。

##### 示例，不好

    struct Data { /*...*/ } data{ /*...*/ };

##### 示例，好

    struct Data { /*...*/ };
    Data data{ /*...*/ };

##### 强制实施

* 如果类或者枚举的定义式的 `}` 后面没有跟着 `;` 就标记出来。它缺少了 `;`。

### <a id="rc-class"></a>C.8: 当有任何非公开成员时使用 `class` 而不是 `struct`

##### 理由

可读性。
表明有些东西被隐藏或者进行了抽象。
这是一种有用的惯例。

##### 示例，不好

    struct Date {
        int d, m;

        Date(int i, Month m);
        // ... 许多函数 ...
    private:
        int y;  // year
    };

这段代码在 C++ 语言规则方面没有任何问题，
但从设计角度看则几乎全是错误。
私有数据和公开数据相比藏得太远了。
数据在类的声明式中被分到了不同的部分中。
不同部分的数据具有不同的访问性。
所有这些都减弱了可读性，并使维护变得更复杂。

##### 注解

优先将接口部分放在类的开头，[参见 NL.16](#rl-order)。

##### 强制实施

对于声明为 `struct` 的类，当其带有 `private` 或 `protected` 成员时就进行标记。

### <a id="rc-private"></a>C.9: 让成员的暴露最小化

##### 理由

封装。
信息隐藏。
使发生意外访问的机会最小化。
这会简化维护工作。

##### 示例

    template<typename T, typename U>
    struct pair {
        T a;
        U b;
        // ...
    };

无论我们在 `//` 部分中干什么，`pair` 的任意用户都可以任意地并且不相关地改动其 `a` 和 `b`。
在大型代码库中，我们无法轻易找出哪段代码对 `pair` 的成员都做了什么。
这也许正是我们想要的，但如果想要在成员之间强加某种关系，就需要使它们为 `private`，
并通过构造函数和成员函数来强加这种关系（不变式）。
例如：

    class Distance {
    public:
        // ...
        double meters() const { return magnitude*unit; }
        void set_unit(double u)
        {
                // ... 检查 u 是 10 的倍数 ...
                // ... 适当地改变幅度 ...
                unit = u;
        }
        // ...
    private:
        double magnitude;
        double unit;    // 1 为米，1000 为千米，0.001 为毫米，等等
    };

##### 注解

如果无法轻易确定一组变量的直接用户的集合，那么这个集合的类型或用法也无法被（轻易）改变或改进。
对于 `public` 和 `protected` 数据来说这是常见的情况。

##### 示例

一个类可以向其用户提供两个接口。
一个针对其派生类（`protected`），而另一个针对一般用户（`public`）。
例如，可能允许派生类跳过某种运行时检查，因为其已经确保了正确性：

    class Foo {
    public:
        int bar(int x) { check(x); return do_bar(x); }
        // ...
    protected:
        int do_bar(int x); // 在数据上做些操作
        // ...
    private:
        // ... 数据 ...
    };

    class Dir : public Foo {
        //...
        int mem(int x, int y)
        {
            /* ... 做一些事 ... */
            return do_bar(x + y); // OK：派生类可以略过检查
        }
    };

    void user(Foo& x)
    {
        int r1 = x.bar(1);      // OK，有检查
        int r2 = x.do_bar(2);   // 错误：可能略过检查
        // ...
    }

##### 注解

[`protected` 数据不是好主意](#rh-protected)。

##### 注解

优先让 `public` 成员在前，`protected` 成员其次，`private` 成员在后；参见 [NL.16](#rl-order)。

##### 强制实施

* [标记 `protected` 数据](#rh-protected)。
* 标记混合的 `public` 和 `private` 数据。

## <a id="ss-concrete"></a>C.concrete: 具体类型

具体类型的规则概览：

* [C.10: 优先使用具体类型而不是类继承层次](#rc-concrete)
* [C.11: 使具体类型正规化](#rc-regular)
* [C.12: 不要令可复制或移动类型的数据成员为 `const` 或引用](#rc-constref)


### <a id="rc-concrete"></a>C.10: 优先使用具体类型而不是类继承层次

##### 理由

具体类型在本质上就比类继承层次中的类型更简单：
它们更易于设计，更易于实现，更易于使用，更易于进行推理，更小，也更快。
使用继承层次是需要一些理由（用例）来支持的。

##### 示例

    class Point1 {
        int x, y;
        // ... 一些操作 ...
        // ... 没有虚函数 ...
    };

    class Point2 {
        int x, y;
        // ... 一些操作，其中有些是虚的 ...
        virtual ~Point2();
    };

    void use()
    {
        Point1 p11 {1, 2};   // 在栈上创建一个对象
        Point1 p12 {p11};    // 一个副本

        auto p21 = make_unique<Point2>(1, 2);   // 在自由存储中创建一个对象
        auto p22 = p21->clone();                // 创建一个副本
        // ...
    }

当一个类属于某个继承层次时，我们（即使在小例子中不需要，在真实代码中也）必然要通过指针或者引用来操作它的对象。
这意味着更多的内存开销，更多的分配和回收操作，以及更多的用于实施间接操作所带来的运行时开销。

##### 注解

具体类型可以在栈上分配，也可以成为其他类的成员。

##### 注解

对于运行时多态接口来说，使用间接是一项基本要求。
而分配/回收操作的开销则不是（它们只是最常见的情况而已）。
我们可以使用基类来作为有作用域的派生类对象的接口。
当禁止使用动态分配时（比如硬实时）就可以这样做，为某些种类的插件提供一种稳定的接口。


##### 强制实施

???


### <a id="rc-regular"></a>C.11: 使具体类型正规化

##### 理由

正规类型比不正规的类型更易于理解和进行推导（不正规性会导致理解和使用上花费更多的精力）。

C++ 内建类型都是正规的，标准程序库的一些类型，如 `string`，`vector`，和 `map` 也是如此。可以定义没有赋值和相等比较的具体类，但它们很罕见（理当如此）。

##### 示例

    struct Bundle {
        string name;
        vector<Record> vr;
    };

    bool operator==(const Bundle& a, const Bundle& b)
    {
        return a.name == b.name && a.vr == b.vr;
    }

    Bundle b1 { "my bundle", {r1, r2, r3}};
    Bundle b2 = b1;
    if (!(b1 == b2)) error("impossible!");
    b2.name = "the other bundle";
    if (b1 == b2) error("No!");

特别是，当具体类型可以复制时，也应当为之提供相等比较运算符，并确保 `a = b` 蕴含 `a == b`。

##### 注解

对于用来与 C 代码共用的结构体，为其定义 `operator==` 是不可行的。

##### 注解

无法进行克隆的资源包装类（例如，包含一个 `mutex` 的 `scoped_lock`）是具体类型，不过通常都无法进行复制（但它们一般都可以被移动），
因此它们不是正规类型；但它们倾向于成为“仅可移动类型”。

##### 强制实施

???


### <a id="rc-constref"></a>C.12: 不要令可复制或移动类型的数据成员为 `const` 或引用

##### 理由

`const` 和引用数据成员在可复制或移动类型中没什么用处，还会由于微妙的原因使这种类型变得至少部分地无法复制/无法移动而很难使用。

##### 示例；不好

    class bad {
        const int i;    // 不好
        string& s;      // 不好
        // ...
    };

`const` 和 `&` 数据成员会导致类变成“仅可进行某些复制”——可复制构造但不可复制赋值。

##### 注解

如果需要一个指向某物的成员，就请使用指针（原始的或智能的，而当它不能为 null 时使用 `gsl::not_null`）而不是引用。

##### 强制实施

标记具有任意复制或移动操作的类型中的 `const`，`&`，或者 `&&` 的数据成员。



## <a id="s-ctor"></a>C.ctor: 构造函数，赋值，和析构函数

这些函数控制对象的生存期：创建，复制，移动，以及销毁。
定义构造函数是为了确保以及简化类的初始化过程。

以下被称为*默认操作*：

* 默认构造函数: `X()`
* 复制构造函数: `X(const X&)`
* 复制赋值: `operator=(const X&)`
* 移动构造函数: `X(X&&)`
* 移动赋值: `operator=(X&&)`
* 析构函数: `~X()`

缺省情况下，编译器会为这些操作中被使用的进行定义，但这些默认定义可以被抑制掉。

默认操作是一组互相关联的操作，它们共同实现了对象的生存期语义。
缺省情况下，C++ 按照值类型的方式来对待各个类，但并非所有的类型都与值类型相符。

默认操作的规则集合：

* [C.20: 只要可能，请避免定义任何的默认操作](#rc-zero)
* [C.21: 如果定义或者 `=delete` 了任何复制、移动或析构函数，请定义或者 `=delete` 它们全部](#rc-five)
* [C.22: 使默认操作之间保持一致](#rc-matched)

析构函数的规则：

* [C.30: 如果一个类需要在对象销毁时执行明确的操作，请为其定义析构函数](#rc-dtor)
* [C.31: 类所获取的所有资源，必须都在类的析构函数中进行释放](#rc-dtor-release)
* [C.32: 如果类中带有原始指针（`T*`）或者引用（`T&`），请考虑它是否是所有者](#rc-dtor-ptr)
* [C.33: 如果类中带有所有权的指针成员，请定义析构函数](#rc-dtor-ptr2)
* [C.35: 基类的析构函数应当要么是 public 和 virtual，要么是 protected 且非 virtual](#rc-dtor-virtual)
* [C.36: 析构函数不能失败](#rc-dtor-fail)
* [C.37: 使析构函数 `noexcept`](#rc-dtor-noexcept)

构造函数的规则：

* [C.40: 如果类具有不变式，请为其定义构造函数](#rc-ctor)
* [C.41: 构造函数应当创建经过完整初始化的对象](#rc-complete)
* [C.42: 当构造函数无法构造有效对象时，应当抛出异常](#rc-throw)
* [C.43: 保证可复制类带有默认构造函数](#rc-default0)
* [C.44: 尽量让默认构造函数简单且不抛出异常](#rc-default00)
* [C.45: 不要定义仅对数据成员进行初始化的默认构造函数；应当使用成员初始化式](#rc-default)
* [C.46: 默认情况下，把单参数的构造函数声明为 `explicit`](#rc-explicit)
* [C.47: 按成员声明的顺序对成员变量进行定义和初始化](#rc-order)
* [C.48: 对于常量初始化式来说，优先采用类中的初始化式而不是构造函数中的成员初始化式](#rc-in-class-initializer)
* [C.49: 优先进行初始化而不是在构造函数中赋值](#rc-initialize)
* [C.50: 当初始化过程中需要体现“虚函数行为”时，请使用工厂函数](#rc-factory)
* [C.51: 用委派构造函数来表示类中所有构造函数的共同行为](#rc-delegating)
* [C.52: 使用继承构造函数来把构造函数引入到无须进行其他的明确初始化操作的派生类之中](#rc-inheriting)

复制和移动的规则：

* [C.60: 使复制赋值非 `virtual`，接受 `const&` 的参数，并返回非 `const` 的引用](#rc-copy-assignment)
* [C.61: 复制操作应当进行复制](#rc-copy-semantic)
* [C.62: 使复制赋值可以安全进行自赋值](#rc-copy-self)
* [C.63: 使移动赋值非 `virtual`，接受 `&&` 的参数，并返回非 `const&`](#rc-move-assignment)
* [C.64: 移动操作应当进行移动，并使原对象处于有效状态](#rc-move-semantic)
* [C.65: 使移动赋值可以安全进行自赋值](#rc-move-self)
* [C.66: 使移动操作 `noexcept`](#rc-move-noexcept)
* [C.67: 多态类应当抑制公开的移动/复制操作](#rc-copy-virtual)

其他的默认操作规则：

* [C.80: 当需要明确使用缺省语义时，使用 `=default`](#rc-eqdefault)
* [C.81: 当需要关闭缺省行为（且不需要替代的行为）时，使用 `=delete`](#rc-delete)
* [C.82: 不要在构造函数和析构函数中调用虚函数](#rc-ctor-virtual)
* [C.83: 考虑为值类型提供 `noexcept` 的 `swap` 函数](#rc-swap)
* [C.84: `swap` 不能失败](#rc-swap-fail)
* [C.85: 使 `swap` 函数 `noexcept`](#rc-swap-noexcept)
* [C.86: 使 `==` 对操作数的类型对称，并使之 `noexcept`](#rc-eq)
* [C.87: 请当心基类的 `==`](#rc-eq-base)
* [C.89: 使 `hash` 函数 `noexcept`](#rc-hash)
* [C.90: 依靠构造函数和赋值运算符，不要依靠 `memset` 和 `memcpy`](#rc-memset)

## <a id="ss-defop"></a>C.defop: 默认操作

缺省情况下，语言会提供具有预置语义的默认操作。
不过，程序员可以关闭或者替换掉这些缺省实现。

### <a id="rc-zero"></a>C.20: 只要可能，请避免定义任何的默认操作

##### 理由

这样最简单，而且能提供最清晰的语义。

##### 示例

    struct Named_map {
    public:
        // ... 并未声明任何默认操作 ...
    private:
        string name;
        map<int, int> rep;
    };

    Named_map nm;        // 默认构造
    Named_map nm2 {nm};  // 复制构造

由于 `std::map` 和 `string` 都带有全部的特殊函数，这里并不需要做别的事情。

##### 注解

这被称为“零之准则（The rule of zero）”。

##### 强制实施

【无法强制实施】 虽然无法强制实施，但一个优秀的静态分析器可以检查出一些模式，指出可使之符合本条规则的改进可能性。
例如，一个带有（指针,大小）成员对，同时在析构函数中 `delete` 这个指针的类也许可以被转换为使用一个 `vector`。

### <a id="rc-five"></a>C.21: 如果定义或者 `=delete` 了任何复制、移动或析构函数，请定义或者 `=delete` 它们全部

##### 理由

复制、移动和析构的语义互相之间是紧密相关的，一旦需要声明其中一个，麻烦的是其他的也需要予以考虑。

只要声明了复制、移动或析构函数，
即便是声明为 `=default` 或 `=delete`，也将会抑制掉
移动构造函数和移动赋值运算符的隐式声明。
而声明移动构造函数或移动赋值运算符，
即便是声明为 `=default` 或 `=delete`，也将会导致隐式生成的复制构造函数
或隐式生成的复制赋值运算符被定义为弃置的。
因此，只要声明了它们中的任何一个，就应当将
其他全部都予以声明，以避免出现预期外的效果，比如将所有潜在的移动
都变成了更昂贵的复制操作，或者使类变为只能移动的。

##### 示例，不好

    struct M2 {   // 不好: 不完整的复制/移动/析构操作集合
    public:
        // ...
        // ... 没有复制和移动操作 ...
        ~M2() { delete[] rep; }
    private:
        pair<int, int>* rep;  // pair 的以零终止的集合
    };

    void use()
    {
        M2 x;
        M2 y;
        // ...
        x = y;   // 缺省的赋值
        // ...
    }

既然对于析构函数需要“特殊关照”（这里是要进行回收操作），隐式定义的复制和移动赋值运算符仍保持正确性的可能是很低的（此处会导致双重删除问题）。

##### 注解

这被称为“五之准则（The rule of five）”。

##### 注解

如果想（于定义了别的函数时）保持缺省实现，请写下 `=default` 以表明对这个函数是特意这样做的。
如果不想要一个生成的缺省函数，可以用 `=delete` 来抑制它。

##### 示例，好

如果要声明析构函数仅是为了使其为 `virtual` 的话，
可将其定义为预置的。

    class AbstractBase {
    public:
        virtual void foo() = 0;  // 需要至少一个抽象方法使其为抽象类
        virtual ~AbstractBase() = default;
        // ...
    };

为避免发生如 [C.67](#rc-copy-virtual) 所说的切片，
使复制和移动操作为受保护的或 `=delete`，并添加 `clone`：

    class CloneableBase {
    public:
        virtual unique_ptr<CloneableBase> clone() const;
        virtual ~CloneableBase() = default;
        CloneableBase() = default;
        CloneableBase(const CloneableBase&) = delete;
        CloneableBase& operator=(const CloneableBase&) = delete;
        CloneableBase(CloneableBase&&) = delete;
        CloneableBase& operator=(CloneableBase&&) = delete;
        // ... 其他构造函数和函数 ...
    };

这里仅定义移动操作或者进定义复制操作也可以具有
相同效果，但明确说明每个特殊成员的意图，
可使其对读者更加易于理解。

##### 注解

编译器会很大程度上强制实施这条规则，并在理想情况下会对任何违反都给出警告。

##### 注解

在带有析构函数的类中，依靠隐式生成的复制操作的做法已经被摒弃。

##### 注解

编写这些函数很容易出错。
注意它们的参数类型：

    class X {
    public:
        // ...
        virtual ~X() = default;               // 析构函数 (如果 X 是基类，用 virtual)
        X(const X&) = default;                // 复制构造函数
        X& operator=(const X&) = default;     // 复制赋值
        X(X&&) noexcept = default;            // 移动构造函数
        X& operator=(X&&) noexcept = default; // 移动赋值
    };

一个小错误（例如拼写错误，遗漏 `const`，使用 `&` 而不是 '&&`，或遗漏一个特殊功能）可能导致错误或警告。
为避免单调乏味和出错的可能性，请尝试遵循[零规则](＃Rc-zero)。

##### 强制实施

【简单】 类中应当要么为每个复制/移动/析构函数都提供一个声明（即便是 `=delete`），要么都不这样做。

### <a id="rc-matched"></a>C.22: 使默认操作之间保持一致

##### 理由

默认操作是一个概念上相配合的集合。它们的语义是相互关联的。
如果复制/移动构造和复制/移动赋值所做的是逻辑上不同的事情的话，这会让使用者感觉诡异。如果构造函数和析构函数并不提供一种对资源管理的统一视角的话，也会让使用者感觉诡异。如果复制和移动操作并不体现出构造函数和析构函数的工作方式的话，同样会让使用者感觉诡异。

##### 示例，不好

    class Silly {   // 不好: 复制操作不一致
        class Impl {
            // ...
        };
        shared_ptr<Impl> p;
    public:
        Silly(const Silly& a) : p{make_shared<Impl>()} { *p = *a.p; }   // 深复制
        Silly& operator=(const Silly& a) { p = a.p; return *this; }   // 浅复制
        // ...
    };

这些操作在复制语义上并不统一。这将会导致混乱和出现 BUG。

##### 强制实施

* 【复杂】 复制/移动构造函数和对应的复制/移动赋值运算符，应当在相同的解引用层次上向相同的成员变量进行写入。
* 【复杂】 在复制/移动构造函数中被写入的任何成员变量，在其他构造函数中也都应当进行初始化。
* 【复杂】 如果复制/移动构造函数对某个成员变量进行了深复制，就应当在析构函数中对这个成员变量进行修改。
* 【复杂】 如果析构函数修改了某个成员变量，在任何复制/移动构造函数或赋值运算符中就都应当对该成员变量进行写入。

## <a id="ss-dtor"></a>C.dtor: 析构函数

“这个类需要析构函数吗？”是一个出人意料有洞察力的设计问题。
对于大多数类来说，答案是“不需要”，要么是因为类中并没有保持任何资源，要么是因为销毁过程已经被[零之准则](#rc-zero)处理掉了；
就是说，它的成员在销毁之中可以自己照顾自己。
当答案为“需要”时，类的大部分设计应当遵循下列规则（参见[五之准则](#rc-five)）。

### <a id="rc-dtor"></a>C.30: 如果一个类需要在对象销毁时执行明确的操作，请为其定义析构函数

##### 理由

析构函数是在对象的生存期结束时被隐式执行的。
如果预置的析构函数足堪使用的话，就应当用它。
只有当类需要执行的代码不在其成员的析构函数中时，才需要定义非预置的析构函数。

##### 示例

    template<typename A>
    struct final_action {   // 略有简化
        A act;
        final_action(A a) : act{a} {}
        ~final_action() { act(); }
    };

    template<typename A>
    final_action<A> finally(A act)   // 推断出动作的类型
    {
        return final_action<A>{act};
    }

    void test()
    {
        auto act = finally([] { cout << "Exit test\n"; });  // 设置退出动作
        // ...
        if (something) return;   // 动作在这里得到执行
        // ...
    } // 动作在这里得到执行

`final_action` 的全部目的就是为了在其销毁时执行一段代码（通常是一个 lambda）。

##### 注解

需要自定义析构函数的类大致上有两种：

* 类中具有某个资源，而它并未表示成一个具有析构函数的类，比如 `vector` 或事物类。
* 类的目的主要用于在销毁时执行某个动作，比如一个追踪器，或者 `final_action`。

##### 示例，不好

    class Foo {   // 不好; 使用预置的析构函数
    public:
        // ...
        ~Foo() { s = ""; i = 0; vi.clear(); }  // 清理
    private:
        string s;
        int i;
        vector<int> vi;
    };

预置的析构函数会做得更好，更高效，而且不会出错。

##### 注解

当需要预置的析构函数，但其生成被抑制（比如由于定义了移动构造函数）时，可以使用 `=default`。

##### 强制实施

查找疑似“隐式的资源”，比如指针和引用等。查找带有析构函数的类，即便其所有数据成员都带有自己的析构函数。

### <a id="rc-dtor-release"></a>C.31: 类所获取的所有资源，必须都在类的析构函数中进行释放

##### 理由

避免资源泄漏，尤其是错误情形中。

##### 注解

对于以具有完整的默认操作集合的类来表示的资源来说，这些都是会自动发生的。

##### 示例

    class X {
        ifstream f;   // 可能会拥有某个文件
        // ... 没有任何定义或者声明为 =deleted 的默认操作 ...
    };

`X` 的 `ifstream` 会在其所在 `X` 的销毁时，隐含地关闭任何可能已经被它所打开的文件。

##### 示例，不好

    class X2 {     // 不好
        FILE* f;   // 可能会拥有某个文件
        // ... 没有任何定义或者声明为 =deleted 的默认操作 ...
    };

`X2` 可能会泄漏文件的句柄。

##### 注解

不过关不掉的 socket 怎么办呢？析构函数、close 以及清理操作[不应当失败](#rc-dtor-fail)。
如果它确实这样的话，就出现了一个不存在真正的好解决方案的问题。
对于新手来说，作为析构函数的编写者，无法了解析构函数是因为什么被调用的，而且不能通过抛出异常来“拒绝执行”。
参见[相关讨论](#sd-never-fail)。
让这个问题更加糟糕的，还包括许多的 close/release 操作都是无法重试的。
许多人都曾试图解决这个问题，但仍不存在已知的一般性解决方案。
如果可能的话，可以考虑把 close/cleanup 的失败看成是基本的设计错误，然后终止程序（terminate）。

##### 注解

类之中也可以持有指向它并不拥有的对象的指针和引用。
显然这样的对象是不应当在类的析构函数中被 `delete` 的。
例如：

    Preprocessor pp { /* ... */ };
    Parser p { pp, /* ... */ };
    Type_checker tc { p, /* ... */ };

这里的 `p` 指向 `pp` 但并不拥有它。

##### 强制实施

* 【简单】 当类中所有的指针或引用成员变量是所有者
  （比如通过使用 `gsl::owner` 所断定）时，它们就应当在析构函数中有所引用。
* 【困难】 当在所有权上没有明确的说法时，为指针或引用成员变量确定其是否是所有者
  （比如，检查构造函数的代码）。

### <a id="rc-dtor-ptr"></a>C.32: 如果类中带有原始指针（`T*`）或者引用（`T&`），请考虑它是否是所有者

##### 理由

大量的代码都是和所有权无关的。

##### 示例

    class legacy_class
    {
        foo* m_owning;   // 不好：改为 unique_ptr<T> 或 owner<T*>
        bar* m_observer; // OK：不用改
    }

唯一确定所有权的方式可能就是深入代码中去寻找内存分配了。

##### 注解

所有权在新代码（以及重构的遗留代码）中应当是清晰的：[R.20](#rr-owner) 对于有所有权指针，
[R.3](#rr-ptr) 对于无所有权指针。引用从来不能有所有权，[R.4](#rr-ref)。

##### 强制实施

查看原始指针成员和引用成员的初始化，看看是否进行了分配操作。

### <a id="rc-dtor-ptr2"></a>C.33: 如果类中带有所有权的指针成员，请定义析构函数

##### 理由

被拥有的对象，必须在拥有它的对象销毁时进行 `delete`。

##### 示例

指针成员可以表示某种资源。
[不应该这样使用 `T*`](#rr-ptr)，但老代码中这是很常见的。
请把 `T*` 当作一种可能的所有者的嫌疑。

    template<typename T>
    class Smart_ptr {
        T* p;   // 不好: *p 的所有权含糊不清
        // ...
    public:
        // ... 没有自定义的默认操作 ...
    };

    void use(Smart_ptr<int> p1)
    {
        // 错误: p2.p 泄漏了（当其不为 nullptr 且未被其他代码所拥有时）
        auto p2 = p1;
    }

注意，当你定义析构函数时，你必须定义或者弃置（delete）[所有的默认操作](#rc-five)：

    template<typename T>
    class Smart_ptr2 {
        T* p;   // 不好: *p 的所有权含糊不清
        // ...
    public:
        // ... 没有自定义的复制操作 ...
        ~Smart_ptr2() { delete p; }  // p 是所有者！
    };

    void use(Smart_ptr2<int> p1)
    {
        auto p2 = p1;   // 错误: 双重删除
    }

预置的复制操作仅仅把 `p1.p` 复制给了 `p2.p`，这导致对 `p1.p` 进行双重销毁。请明确所有权的处理：

    template<typename T>
    class Smart_ptr3 {
        owner<T*> p;   // OK: 明确了 *p 的所有权
        // ...
    public:
        // ...
        // ... 复制和移动操作 ...
        ~Smart_ptr3() { delete p; }
    };

    void use(Smart_ptr3<int> p1)
    {
        auto p2 = p1;   // OK: 未发生双重删除
    }

##### 注解

通常最简单的处理析构函数的方式，就是把指针换成一个智能指针（比如 `std::unique_ptr`），并让编译器来安排进行恰当的隐式销毁过程。

##### 注解

为什么不直接要求全部带有所有权的指针都是“智能指针”呢？
这样做有时候需要进行不平凡的代码改动，并且可能会对 ABI 造成影响。

##### 强制实施

* 怀疑带有指针数据成员的类。
* 带有 `owner<T>` 的类应当定义其默认操作。


### <a id="rc-dtor-virtual"></a>C.35: 基类的析构函数应当要么是 public 和 virtual，要么是 protected 且非 virtual

##### 理由

以防止未定义行为。
若析构函数是 public，调用方代码就可以尝试通过基类指针来销毁一个派生类的对象，而如果基类的析构函数是非 virtual，则其结果是未定义的。
若析构函数是 protected，调用方代码就无法通过基类指针进行销毁，而且这个析构函数不需要是 virtual；它应当是 protected 而不是 private，以便它能够在派生类析构函数中执行。
总之，基类的编写者并不知道什么是当进行销毁时要做的适当操作。

##### 探讨

请参见[这条规则](#sd-dtor)中的探讨段落.

##### 示例，不好

    struct Base {  // 不好: 隐含带有 public 的非 virtual 析构函数
        virtual void f();
    };

    struct D : Base {
        string s {"a resource needing cleanup"};
        ~D() { /* ... do some cleanup ... */ }
        // ...
    };

    void use()
    {
        unique_ptr<Base> p = make_unique<D>();
        // ...
    } // p 的销毁调用了 ~Base() 而不是 ~D()，这导致 D::s 的泄漏，也许不止

##### 注解

虚函数针对派生类定义了一个接口，使用它并不需要对派生类有所了解。
如果这个接口允许进行销毁，那么它应当安全地做到这点。

##### 注解

析构函数必须是非私有的，否则它会妨碍使用这个类型：

    class X {
        ~X();   // 私有析构函数
        // ...
    };

    void use()
    {
        X a;                        // 错误: 无法销毁
        auto p = make_unique<X>();  // 错误: 无法销毁
    }

##### 例外

可以构想出一种可能需要受保护虚析构函数的情形：派生类型（且仅限于这种类型）的对象允许通过基类指针来销毁*另一个*对象（而不是其自身）。不过我们在实际中从未见到过这种情况。


##### 强制实施

* 带有任何虚函数的类的析构函数，应当要么是 public virtual，要么是 protected 且非 virtual。
* 如果某个类公开继承于某个基类，则该基类应当具有要么是 public virtual，要么是 protected 且非 virtual 的析构函数。

### <a id="rc-dtor-fail"></a>C.36: 析构函数不能失败

##### 理由

一般来说当析构函数可能失败时我们不知道怎样写出没有错误的代码。
标准库要求它所处理的所有的类所带有的析构函数都应当不会因抛出异常而退出。

##### 示例

    class X {
    public:
        ~X() noexcept;
        // ...
    };

    X::~X() noexcept
    {
        // ...
        if (cannot_release_a_resource) terminate();
        // ...
    }

##### 注解

许多人都曾试图针对析构函数中的故障处理设计一种傻瓜式的方案。
但没有人得到过任何一种通用方案。
这确实是真正的实际问题：比如说，怎么处理无法关闭的 socket？
析构函数的编写者无法了解析构函数为什么会被调用，并且不能通过抛出异常来“拒绝执行”。
参见[探讨](#sd-never-fail)段落。
让问题更麻烦的是，许多的“关闭/释放”操作还都是不能重试的。
如果确实可行的话，请把“关闭/清理”的失败作为一项基本设计错误并终止（terminate）程序。

##### 注解

把析构函数声明为 `noexcept`。这将确保它要么正常完成执行，要么就终止程序。

##### 注解

如果一个资源无法释放而程序不能失败，请尝试把这个故障用某种方式通知给系统中的其他部分
（也许甚或修改某个全局状态，并希望有人能注意到它并有能力处理这个问题）。
请充分警惕，这种技巧是有专门用途的，并且容易出错。
请考虑“连接关闭不了”的那个例子。
这也许是因为连接的另一端出现了问题，但只有对连接的两端同时负责的代码才能恰当地处理这个问题。
析构函数可以向系统中负责管控的部分发送一个消息（或别的什么），然后认为已经关闭了连接并正常返回。

##### 注解

如果析构函数所用的操作可能会失败的话，它可以捕获这些异常，某些时候仍然可以成功完成执行
（例如，换用与抛出异常的清理机制不同的另一种机制）。

##### 强制实施

【简单】 如果析构函数可能抛出异常，就应当将其声明为 `noexcept`。

### <a id="rc-dtor-noexcept"></a>C.37: 使析构函数 `noexcept`

##### 理由

[析构函数不能失败](#rc-dtor-fail)。如果析构函数试图抛出异常来退出，这就是一种设计错误，程序最好终止执行。

##### 注解

当类中的所有成员都带有 `noexcept` 析构函数时，析构函数（无论是自定义的还是编译器生成的）将被隐含地声明为 `noexcept`（这与其函数体中的代码无关）。通过将析构函数明确标记为 `noexcept`，程序员可以防止由于添加或修改类的成员而导致析构函数变为隐含的 `noexcept(false)`。

##### 示例

并非所有析构函数都默认为 noexcept; 一个抛出异常的成员会影响整个类的层级：

    struct X {
        Details x;  // 碰巧有一个抛出析构函数
        // ...
        ~X() { }    // 隐含地 noexcept（false）; 也可以抛出异常
    };

所以，不确定的话，声明一个析构函数 noexcept.

##### 注解

为什么对所有析构函数声明 noexcept？
因为在许多情况下，特别是简单的情况，会分散混乱。

##### 强制实施

【简单】 如果析构函数可能抛出异常，就应当将其声明为 `noexcept`。

## <a id="ss-ctor"></a>C.ctor: 构造函数

构造函数定义对象如何进行初始化（构造）。

### <a id="rc-ctor"></a>C.40: 如果类具有不变式，请为其定义构造函数

##### 理由

这正是构造函数的用途。

##### 示例

    class Date {  // Date 表示从 1900/1/1 到 2100/12/31 范围中
                  // 的一个有效日期
        Date(int dd, int mm, int yy)
            :d{dd}, m{mm}, y{yy}
        {
            if (!is_valid(d, m, y)) throw Bad_date{};  // 不变式的实施
        }
        // ...
    private:
        int d, m, y;
    };

把不变式表达为构造函数上的一个 `Ensures` 通常是一种好做法。

##### 注解

即便类并没有不变式，也可以用构造函数来简化代码。例如：

    struct Rec {
        string s;
        int i {0};
        Rec(const string& ss) : s{ss} {}
        Rec(int ii) :i{ii} {}
    };

    Rec r1 {7};
    Rec r2 {"Foo bar"};

##### 注解

C++11 的初始化式列表规则免除了对许多构造函数的需求。例如：

    struct Rec2{
        string s;
        int i;
        Rec2(const string& ss, int ii = 0) :s{ss}, i{ii} {}   // 多余的
    };

    Rec2 r1 {"Foo", 7};
    Rec2 r2 {"Bar"};

`Rec2` 的构造函数是多余的。
同样的，`int` 的默认值最好用[成员初始化式](#rc-in-class-initializer)来给出。

**参见**: [构造有效对象](#rc-complete)和[构造函数抛出异常](#rc-throw)。

##### 强制实施

* 对带有自定义的复制操作但没有构造函数的类进行标记（自定义的复制操作是类是否带有不变式的良好指示器）

### <a id="rc-complete"></a>C.41: 构造函数应当创建经过完整初始化的对象

##### 理由

构造函数为类设立不变式。类的使用者应当能够假定构造完成的对象是可以使用的。

##### 示例，不好

    class X1 {
        FILE* f;   // 在任何其他函数之前应当调用 init()
        // ...
    public:
        X1() {}
        void init();   // 初始化 f
        void read();   // 从 f 中读取数据
        // ...
    };

    void f()
    {
        X1 file;
        file.read();   // 程序崩溃或者错误的数据读取！
        // ...
        file.init();   // 太晚了
        // ...
    }

编译器读不懂代码注释。

##### 例外

如果无法方便地通过构造函数来构造有效的对象的话，请[使用工厂函数](#rc-factory)。

##### 强制实施

* 【简单】 每个构造函数都应当对每个成员变量进行初始化（明确地，通过委派构造函数调用，或者通过默认构造）。
* 【未知】 如果构造函数带有 `Ensures` 契约的话，尝试确定它给出的是否是一项后条件。

##### 注解

如果构造函数（为创建有效的对象）获取了某个资源，则这个资源应当[由析构函数释放](#rc-dtor-release)。
这种以构造函数获取资源并以析构函数来释放的惯用法被称为 [RAII](#rr-raii)（“资源获取即初始化/Resource Acquisition Is Initialization”）。

### <a id="rc-throw"></a>C.42: 当构造函数无法构造有效对象时，应当抛出异常

##### 理由

留下无效对象不管就是会造成麻烦的做法。

##### 示例

    class X2 {
        FILE* f;
        // ...
    public:
        X2(const string& name)
            :f{fopen(name.c_str(), "r")}
        {
            if (!f) throw runtime_error{"could not open" + name};
            // ...
        }

        void read();      // 从 f 中读取数据
        // ...
    };

    void f()
    {
        X2 file {"Zeno"}; // 当文件打不开时会抛出异常
        file.read();      // 好的
        // ...
    }

##### 示例，不好

    class X3 {     // 不好: 构造函数留下了无效的对象
        FILE* f;   // 在任何其他函数之前应当调用 is_valid()
        bool valid;
        // ...
    public:
        X3(const string& name)
            :f{fopen(name.c_str(), "r")}, valid{false}
        {
            if (f) valid = true;
            // ...
        }

        bool is_valid() { return valid; }
        void read();   // 从 f 中读取数据
        // ...
    };

    void f()
    {
        X3 file {"Heraclides"};
        file.read();   // 程序崩溃或错误的数据读取！
        // ...
        if (file.is_valid()) {
            file.read();
            // ...
        }
        else {
            // ... 处理错误 ...
        }
        // ...
    }

##### 注解

对于变量的定义式（比如在栈上，或者作为其他对象的成员），不存在可以返回错误代码的明确函数调用。
留下无效的对象并依赖使用者能够一贯地在使用之前检查 `is_valid()` 函数是啰嗦的，易错的，并且是低效的做法。

##### 例外

有些领域，比如像飞行器控制这样的硬实时系统中，（在没有其他工具支持下）异常处理在计时方面不具有足够的可预测性。
这样的话就必须使用 `is_valid()` 技巧。这种情况下，可以一贯并即刻地检查 `is_valid()` 来模拟 [RAII](#rr-raii)。

##### 替代方案

如果你觉得想要使用某种“构造函数之后初始化”或者“两阶段初始化”手法，请试着避免这样做。
如果你确实要如此的话，请参考[工厂函数](#rc-factory)。

##### 注解

人们使用 `init()` 函数而不是在构造函数中进行初始化的一种原因是为了避免代码重复。
[委派构造函数](#rc-delegating)和[默认成员初始化式](#rc-in-class-initializer)可以更好地做到这点。
另一种原因是为了把初始化推迟到需要对象的位置；它的解决方法通常为“[直到变量可以正确进行初始化的位置再声明变量](#res-init)”。

##### 强制实施

???

### <a id="rc-default0"></a>C.43: 保证可复制类带有默认构造函数

##### 理由

就是说，确保当具体类可复制时它也满足“半正规”类型的其他规定。

许多的语言和程序库设施都依赖于默认构造函数来初始化其各个元素，比如 `T a[10]` 和 `std::vector<T> v(10)`。
对于同时是可复制的类型来说，默认构造函数通常会简化定义一个适当的[移动遗留状态](#???)的任务。

##### 示例

    class Date { // 不好: 缺少默认构造函数
    public:
        Date(int dd, int mm, int yyyy);
        // ...
    };

    vector<Date> vd1(1000);   // 需要默认的 Date
    vector<Date> vd2(1000, Date{7, Month::October, 1885});   // 替代方式

仅当没有用户声明的构造函数时，默认构造函数才会自动生成，因此上面的例子中的 vector `vdl` 是无法进行初始化的。
缺乏默认值会导致用户感觉奇怪，并且使其使用变复杂，因此如果可以合理定义的话就应当定义默认值。

`Date` 可以推动我们考虑：
“天然的”默认日期是不存在的（大爆炸对大多数人来说在时间上太过久远了），因此这并非是毫无意义的例子。
`{0, 0, 0}` 在大多数历法系统中都不是有效的日期，因此选用它可能会引入某种如同浮点的 `NaN` 这样的东西。
不过，大多数现实的 `Date` 类都有某个“首日”（比如很常见的 1970/1/1），因此以它为默认日期通常很容易做到。

    class Date {
    public:
        Date(int dd, int mm, int yyyy);
        Date() = default; // [参见](#rc-default)
        // ...
    private:
        int dd {1};
        int mm {1};
        int yyyy {1970};
        // ...
    };

    vector<Date> vd1(1000);

##### 注解

所有成员都带有默认构造函数的类，隐含得到一个默认构造函数：

    struct X {
        string s;
        vector<int> v;
    };

    X x; // 意为 X{{}, {}}; 即空字符串和空 vector

需要注意的是，内建类型并不会进行正确的默认构造：

    struct X {
        string s;
        int i;
    };

    void f()
    {
        X x;    // x.s 被初始化为空字符串; x.i 未初始化

        cout << x.s << ' ' << x.i << '\n';
        ++x.i;
    }

静态分配的内建类型对象被默认初始化为 `0`，但局部的内建变量并非如此。
请注意你的编译期也许默认初始化了局部内建变量，而它在优化构建中并不会这样做。
因此，上例这样的代码也许恰好可以工作，但这其实依赖于未定义的行为。
假定你确实需要初始化的话，可以使用明确的默认初始化：

    struct X {
        string s;
        int i {};   // 默认初始化（为 0）
    };

##### 注解

缺乏合理的默认构造的类，通常也都不是可以复制的，因此它们并不受本条指导方针所限。

例如，基类不能进行复制，且因而并不需要一个默认构造函数：

    // Shape 是个抽象基类，而不是可复制类型
    // 它可以有也可以没有默认构造函数。
    struct Shape {
        virtual void draw() = 0;
        virtual void rotate(int) = 0;
        // =delete 复制/移动函数
        // ...
    };

必须在构造过程中获取由调用方提供的资源的类，通常无法提供默认构造函数，但它们并不受本条指导方针所限，因为这样的类通常也不是可复制的：

    // std::lock_guard 不是可复制类型。
    // 它没有默认构造函数。
    lock_guard g {mx};  // 护卫 mutex mx
    lock_guard g2;      // 错误：不护卫任何东西

带有必须由其成员函数或者其用户进行特殊处理的“特殊状态”的类，会带来额外的工作量，
（而且很可能有更多的错误）。这样的类型不管其是否可以复制，都可以以这个特殊状态作为其默认构造的值：

    // std::ofstream 不是可复制类型。
    // 它刚好有一个默认构造函数，
    // 并带来一种特殊的“未打开”状态。
    ofstream out {"Foobar"};
    // ...
    out << log(time, transaction);

一些类似的可复制的具有特殊状态的类型，比如具有特殊状态“==nullptr”的可复制的智能指针，也应该以该特殊状态作为其默认构造的值。

不过，为有意义的状态提供默认构造函数（比如 `std::string` 的 `""` 和 `std::vector` 的 `{}`），也是推荐的做法。

##### 强制实施

* 对于可用 `=` 进行复制的类，若没有默认构造函数则对其进行标记。
* 对于可用 `==` 进行比较但不可复制的类进行标记。


### <a id="rc-default00"></a>C.44: 尽量让默认构造函数简单且不抛出异常

##### 理由

如果可以设置一个“默认”值同时又不会涉及可能失败的操作的话，就可以简化错误处理以及对移动操作的推理。

##### 示例，有问题的

    template<typename T>
    // elem 指向以 new 分配的 space-elem 个元素
    class Vector0 {
    public:
        Vector0() :Vector0{0} {}
        Vector0(int n) :elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem;
        T* space;
        T* last;
    };

这段代码很不错而且通用，不过在发生错误之后把一个 `Vector0` 进行置空会涉及一次分配，而它是可能失败的。
而且把默认的 `Vector` 表示为 `{new T[0], 0, 0}` 也比较浪费。
比如说，`Vector0<int> v[100]` 会耗费 100 次分配操作。

##### 示例

    template<typename T>
    // elem 为 nullptr，否则 elem 指向以 new 分配的 space-elem 个元素
    class Vector1 {
    public:
        // 设置表示为 {nullptr, nullptr, nullptr}; 不会抛出异常
        Vector1() noexcept {}
        Vector1(int n) :elem{new T[n]}, space{elem + n}, last{elem} {}
        // ...
    private:
        own<T*> elem {};
        T* space {};
        T* last {};
    };

表示为 `{nullptr, nullptr, nullptr}` 的 `Vector1{}` 很廉价，但这是一种特殊情况并且隐含了运行时检查。
在检测到错误后可以很容易地把 `Vector1` 置空。

##### 强制实施

* 标记会抛出的默认构造函数

### <a id="rc-default"></a>C.45: 不要定义仅对数据成员进行初始化的默认构造函数；应当使用成员初始化式

##### 理由

使用类内部的成员初始化式，编译器可以据此生成函数。由编译器生成的函数可能更高效。

##### 示例，不好

    class X1 { // 不好: 未使用成员初始化式
        string s;
        int i;
    public:
        X1() :s{"default"}, i{1} { }
        // ...
    };

##### 示例

    class X2 {
        string s {"default"};
        int i {1};
    public:
        // 使用编译期生成的默认构造函数
        // ...
    };

##### 强制实施

【简单】 默认构造函数应当不只是用常量初始化成员变量。

### <a id="rc-explicit"></a>C.46: 默认情况下，把单参数的构造函数声明为 `explicit`

##### 理由

用以避免意外的类型转换。

##### 示例，不好

    class String {
    public:
        String(int);   // 不好
        // ...
    };

    String s = 10;   // 意外: 大小为 10 的字符串

##### 例外

如果确实想要从构造函数参数类型隐式转换为类类型的话，就不使用 `explicit`：

    class Complex {
    public:
        Complex(double d);   // OK: 希望进行从 d 向 {d, 0} 的转换
        // ...
    };

    Complex z = 10.7;   // 无意外的转换

**参见**: [有关隐式转换的讨论](#ro-conversion)

##### 注解

不应当将复制和移动构造函数作为 `explicit` 的，因为它们并不进行转换。显式的复制/移动构造函数会把按值传递和返回变麻烦。

##### 强制实施

【简单】 单参数的构造函数应当声明为 `explicit`。有益的单参数非 `explicit` 构造函数在大多数代码库中都是很少见的。对没在“已确认列表”中列出的每个违规都要给出警告。

### <a id="rc-order"></a>C.47: 按成员声明的顺序对成员变量进行定义和初始化

##### 理由

以尽量避免混淆和错误。该顺序正是初始化的发生顺序（而这与成员初始化式的顺序无关）。

##### 示例，不好

    class Foo {
        int m1;
        int m2;
    public:
        Foo(int x) :m2{x}, m1{++x} { }   // 不好: 有误导性的初始化式顺序
        // ...
    };

    Foo x(1); // 意外: x.m1 == x.m2 == 2

##### 强制实施

【简单】 成员初始化式的列表中应当以成员声明的相同顺序列出各个成员。

**参见**: [讨论](#sd-order)

### <a id="rc-in-class-initializer"></a>C.48: 对于常量初始化式来说，优先采用类中的初始化式而不是构造函数中的成员初始化式

##### 理由

明确所有构造函数都将使用相同的值。避免重复。避免可维护性问题。这样做会产生最简短最高效的代码。

##### 示例，不好

    class X {   // 不好
        int i;
        string s;
        int j;
    public:
        X() :i{666}, s{"qqq"} { }   // j 未初始化
        X(int ii) :i{ii} {}         // s 为 "" 而 j 未初始化
        // ...
    };

维护者如何能看出 `j` 是否是故意未初始化的（尽管这可能是个糟糕的想法），而且是不是故意要使 `s` 的默认值在一种情况下为 `""` 而另一种情况下为 `qqq` 呢（几乎可以肯定是个 Bug）？这里 `j` 的问题（忘记对成员初始化）通常会出现在向现存类中添加新成员的时候。

##### 示例

    class X2 {
        int i {666};
        string s {"qqq"};
        int j {0};
    public:
        X2() = default;        // 所有成员都初始化为默认值
        X2(int ii) :i{ii} {}   // s 和 j 被初始化为默认值
        // ...
    };

**替代方案**: 也可以用构造函数的默认实参来获得一部分的好处，而且这在比较老的代码中也并不少见。不过这种方式不够直白，会导致需要传递较多的参数，并且当有多个构造函数时也会造成重复：

    class X3 {   // 不好: 不明确，参数传递开销
        int i;
        string s;
        int j;
    public:
        X3(int ii = 666, const string& ss = "qqq", int jj = 0)
            :i{ii}, s{ss}, j{jj} { }   // 所有成员都初始化为默认值
        // ...
    };

##### 强制实施

* 【简单】 每个构造函数都应该对所有成员变量进行初始化（明确进行，通过委派构造函数调用，或者通过默认构造）。
* 【简单】 构造函数的默认实参的出现表明类内部的初始化式可能更合适。

### <a id="rc-initialize"></a>C.49: 优先进行初始化而不是在构造函数中赋值

##### 理由

初始化语法明确指出所进行的是初始化而不是赋值，它更加精炼和高效。这样也避免了“未设值前就使用”的错误。

##### 示例，好

    class A {   // 好
        string s1;
    public:
        A(czstring p) : s1{p} { }    // 好: 直接构造（这里明确指名了 C 风格字符串）
        // ...
    };

##### 示例，不好

    class B {   // 不好
        string s1;
    public:
        B(const char* p) { s1 = p; }   // 不好: 执行默认构造函数之后进行赋值
        // ...
    };

    class C {   // 恶劣，非常不好
        int* p;
    public:
        C() { cout << *p; p = new int{10}; }   // 意外，初始化前就被使用了
        // ...
    };

##### 示例，更好的做法

可以使用 C++17 的 `std::string_view` 或 `gsl::span<char>` 代替这些 `const char*`
来作为[一种表示函数实参的更通用的方式](#rstr-view)：

    class D {   // 好
        string s1;
    public:
        D(string_view v) : s1{v} { }    // 好: 直接构造
        // ...
    };

### <a id="rc-factory"></a>C.50: 当初始化过程中需要体现“虚函数行为”时，请使用工厂函数

##### 理由

当基类对象的状态必须依赖于对象的派生部分的状态时，需要使用虚函数（或等价手段），并最小化造成误用和不完全构造的对象的机会窗口。

##### 注解

工厂的返回类型默认情况下通常应当为 `unique_ptr`；如果某些用法需要共享，则调用方可以将这个 `unique_ptr` `move` 到一个 `shared_ptr` 中。但是，如果工厂的作者已知其所返回的对象的所有用法都是共享使用的话，就可返回 `shared_ptr`，并在函数体中使用 `make_shared` 以节省一次分配。

##### 示例，不好

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

##### 示例

    class B {
    protected:
        class Token {};

    public:
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
    };

    class D : public B {                 // 某个派生类
    protected:
        class Token {};

    public:
        explicit D(Token) : B( B::Token{} ) {}
        void f() override { /* ... */ };

    protected:
        template<class T>
        friend shared_ptr<T> B::create();
    };

    shared_ptr<D> p = D::create<D>();  // 创建一个 D 的对象

`make_shared` 要求公开的构造函数。构造函数通过要求一个受保护的 `Token` 而无法再被公开调用，从而避免不完全构造的对象泄漏出去。
通过提供工厂函数 `create()`，（在自由存储上）构造对象变得简便。

##### 注解

根据惯例，工厂方法在自由存储上进行分配，而不是在运行栈或者某个外围对象之内进行。

**参见**: [讨论](#sd-factory)

### <a id="rc-delegating"></a>C.51: 用委派构造函数来表示类中所有构造函数的共同行为

##### 理由

以避免代码重复和意外出现的差异。

##### 示例，不好

    class Date {   // 不好: 有重复
        int d;
        Month m;
        int y;
    public:
        Date(int dd, Month mm, year yy)
            :d{dd}, m{mm}, y{yy}
            { if (!valid(d, m, y)) throw Bad_date{}; }

        Date(int dd, Month mm)
            :d{dd}, m{mm} y{current_year()}
            { if (!valid(d, m, y)) throw Bad_date{}; }
        // ...
    };

写这些共同行为很啰嗦，而且可能意外出现不一致。

##### 示例

    class Date2 {
        int d;
        Month m;
        int y;
    public:
        Date2(int dd, Month mm, year yy)
            :d{dd}, m{mm} y{yy}
            { if (!valid(d, m, y)) throw Bad_date{}; }

        Date2(int dd, Month mm)
            :Date2{dd, mm, current_year()} {}
        // ...
    };

**参见**: 当“重复行为”是简单的初始化时，考虑使用[类内部的成员初始化式](#rc-in-class-initializer)。

##### 强制实施

【中等】 查找相似的构造函数体。

### <a id="rc-inheriting"></a>C.52: 使用继承构造函数来把构造函数引入到无须进行其他的明确初始化操作的派生类之中

##### 理由

当派生类需要这些构造函数时，重新实现它们既啰嗦又容易出错。

##### 示例

`std::vector` 有许多棘手的构造函数，如果我想要创建自己的 `vector` 的话，我并不想重新实现它们：

    class Rec {
        // ... 数据，以及许多不错的构造函数 ...
    };

    class Oper : public Rec {
        using Rec::Rec;
        // ... 没有数据成员 ...
        // ... 许多不错的工具函数 ...
    };

##### 示例，不好

    struct Rec2 : public Rec {
        int x;
        using Rec::Rec;
    };

    Rec2 r {"foo", 7};
    int val = r.x;   // 未初始化

##### 强制实施

确保派生类的每个成员都被初始化。

## <a id="ss-copy"></a>C.copy: 复制和移动

具体类型一般都应当是可以复制的，而类层次中的接口则不应如此。
资源包装可以复制也可以不能复制。
我们可以基于逻辑因素，也可以为性能原因而将类型定义为可移动的。

### <a id="rc-copy-assignment"></a>C.60: 使复制赋值非 `virtual`，接受 `const&` 的参数，并返回非 `const` 的引用

##### 理由

这样做简单且高效。如果想对右值进行优化，则可以提供一个接受 `&&` 的重载（参见 [F.18](#rf-consume)）。

##### 示例

    class Foo {
    public:
        Foo& operator=(const Foo& x)
        {
            // 好: 不需要检查自赋值的情况（除非为性能考虑）
            auto tmp = x;
            swap(tmp); // 参见 C.83
            return *this;
        }
        // ...
    };

    Foo a;
    Foo b;
    Foo f();

    a = b;    // 用左值赋值：复制
    a = f();  // 用右值赋值：可能进行移动

##### 注解

`swap` 实现技巧可以提供[强保证](#abrahams01)。

##### 示例

如果不产生临时副本能够得到明显好得多的性能的话应当怎么办呢？考虑一个简单的 `Vector` 类，其所使用的领域中常常要对大型的、大小相同的 `Vector` 进行赋值。这种情况下，`swap` 实现技巧中所蕴含的元素复制操作将导致运行成本按数量级增长。

    template<typename T>
    class Vector {
    public:
        Vector& operator=(const Vector&);
        // ...
    private:
        T* elem;
        int sz;
    };

    Vector& Vector::operator=(const Vector& a)
    {
        if (a.sz > sz) {
            // ... 使用 swap 技巧，没有更好的方式了 ...
            return *this;
        }
        // ... 从 *a.elem 复制 sz 个元素给 elem ...
        if (a.sz < sz) {
            // ... 销毁 *this 中过剩的元素并调整大小 ...
        }
        return *this;
    }

直接向目标元素中进行写入的话，我们得到的是[基本保证](#abrahams01)而不是 `swap` 技巧所提供的强保证。还要当心[自赋值](#rc-copy-self)。

**替代方案**: 如果你想要 `virtual` 的赋值运算符，并了解为何这样做很有问题的话，请不要使其为 `operator=`。请使用一个命名函数，如 `virtual void assign(const Foo&)`。
参见[复制构造函数 vs. `clone()`](#rc-copy-virtual)。

##### 强制实施

* 【简单】 赋值运算符不能为 `virtual`。有怪兽出没！
* 【简单】 赋值运算符应当返回 `T&` 以支持调用链，不要改为如 `const T&` 等类型，这样会影响可组合性以及把对象放入容器的能力。
* 【中等】 赋值运算符应当（隐式或者显式）调用所有的基类和成员的赋值运算符。
  检查析构函数以分辨类型具有指针语义还是值语义。

### <a id="rc-copy-semantic"></a>C.61: 复制操作应当进行复制

##### 理由

这正是一般假定所具有的语义。执行 `x = y` 之后，应当有 `x == y`。
进行复制之后，`x` 和 `y` 可以是各自独立的对象（值语义，非指针的内建类型和标准库类型的工作方式），也可以代表某个共享的对象（指针语义，就是指针的工作方式）。

##### 示例

    class X {   // OK: 值语义
    public:
        X();
        X(const X&);     // 复制 X
        void modify();   // 改变 X 的值
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };

    bool operator==(const X& a, const X& b)
    {
        return a.sz == b.sz && equal(a.p, a.p + a.sz, b.p, b.p + b.sz);
    }

    X::X(const X& a)
        :p{new T[a.sz]}, sz{a.sz}
    {
        copy(a.p, a.p + sz, p);
    }

    X x;
    X y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x == y) throw Bad{};   // 假定具有值语义

##### 示例

    class X2 {  // OK: 指针语义
    public:
        X2();
        X2(const X2&) = default; // 浅拷贝
        ~X2() = default;
        void modify();          // 改变所指向的值
        // ...
    private:
        T* p;
        int sz;
    };

    bool operator==(const X2& a, const X2& b)
    {
        return a.sz == b.sz && a.p == b.p;
    }

    X2 x;
    X2 y = x;
    if (x != y) throw Bad{};
    x.modify();
    if (x != y) throw Bad{};  // 假定具有指针语义

##### 注解

应当优先采用值语义，除非你要构建某种“智能指针”。值语义是最容易进行推理的，而且也是被标准库设施所期望的。

##### 强制实施

【无法强制实施】

### <a id="rc-copy-self"></a>C.62: 使复制赋值可以安全进行自赋值

##### 理由

如果 `x=x` 会改变 `x` 的值的话，会让人惊异，并导致发生严重的错误（通常会含有资源泄漏）。

##### 示例

标准库的容器类都能优雅且高效地处理自赋值：

    std::vector<int> v = {3, 1, 4, 1, 5, 9};
    v = v;
    // v 的值仍然是 {3, 1, 4, 1, 5, 9}

##### 注解

从可以处理自赋值的成员所生成的默认复制操作是能够正确处理自赋值的。

    struct Bar {
        vector<pair<int, int>> v;
        map<string, int> m;
        string s;
    };

    Bar b;
    // ...
    b = b;   // 正确而且高效

##### 注解

可以通过明确检测自赋值来处理自赋值的情况，不过通常不进行这种检测会变得更快并且更优雅（比如说，[利用 `swap`](#rc-swap)）。

    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(const Foo& a);
        // ...
    };

    Foo& Foo::operator=(const Foo& a)   // OK，但增加了成本
    {
        if (this == &a) return *this;
        s = a.s;
        i = a.i;
        return *this;
    }

这显然是安全的，也貌似高效。
不过，如果一百万次赋值才会做一次自赋值会怎么样呢？
这样就有大约一百万次多余的测试（不过由于基本上每次的答案都相同，计算机的分支预测电路也基本上每次都会猜对）。
考虑：

    Foo& Foo::operator=(const Foo& a)   // 更简单，而且可能也更好
    {
        s = a.s;
        i = a.i;
        return *this;
    }

`std::string` 的自赋值是安全的，`int` 也是如此。所有的成本都将花在（罕见的）自赋值情况中。

##### 强制实施

【简单】 赋值运算符不应当包含 `if (this == &a) return *this;` 这样的代码模式 ???

### <a id="rc-move-assignment"></a>C.63: 使移动赋值非 `virtual`，接受 `&&` 的参数，并返回非 `const&`

##### 理由

这样简单而且高效。

**参见**: [针对复制赋值的规则](#rc-copy-assignment)。

##### 强制实施

和针对[复制赋值](#rc-copy-assignment)所做的相同。

* 【简单】 赋值运算符不能为 `virtual`。有怪兽出没！
* 【简单】 赋值运算符应当返回 `T&` 以支持调用链，不要改为如 `const T&` 等类型，这样会影响可组合性以及把对象放入容器的能力。
* 【中等】 移动赋值运算符应当（隐式或者显式）调用所有的基类和成员的移动赋值运算符。

### <a id="rc-move-semantic"></a>C.64: 移动操作应当进行移动，并使原对象处于有效状态

##### 理由

这正是一般假定所具有的语义。
执行 `y=std::move(x)` 之后，`y` 的值应当为 `x` 曾经的值，而 `x` 应当处于有效状态。

##### 示例

    class X {   // OK: 值语义
    public:
        X();
        X(X&& a) noexcept;  // 移动 X
        X& operator=(X&& a) noexcept; // 移动赋值 X
        void modify();     // 改变 X 的值
        // ...
        ~X() { delete[] p; }
    private:
        T* p;
        int sz;
    };

    X::X(X&& a) noexcept
        :p{a.p}, sz{a.sz}  // 窃取其表示
    {
        a.p = nullptr;     // 设其为“空”
        a.sz = 0;
    }

    void use()
    {
        X x{};
        // ...
        X y = std::move(x);
        x = X{};   // OK
    } // OK: x 可以销毁

##### 注解

理想情况下，被移走的对象应当为类型的默认值。
请确保体现这点，除非有非常好的理由不这样做。
然而，并非所有类型都有默认值，而有些类型建立默认值则是昂贵操作。
标准所要求的仅仅是被移走的对象应当可以被销毁。
我们通常也可以轻易且廉价地做得更好一些：标准库假定它可以向被移走的对象进行赋值。
请保证总是让被移走的对象处于某种（需要明确的）有效状态。

##### 注解

请让 `x = std::move(y); y = z;` 按照惯例约定的语义工作，除非有某个十分强大的理由不这样做。

##### 强制实施

【无法强制实施】 检查移动操作中对成员的赋值。如果有默认构造函数的话，则把这些赋值和默认构造函数中的初始化之间进行比较。

### <a id="rc-move-self"></a>C.65: 使移动赋值可以安全进行自赋值

##### 理由

如果 `x = x` 会改变 `x` 的值的话，会让人惊异，并导致发生严重的错误（通常会含有资源泄漏）。不过，通常不会有人写出能够变成移动操作的自赋值代码，但它确实是会发生的。不管怎样，`std::swap` 就是利用移动操作来实现的，因此如果你不小心写了 `swap(a, b)` 而 `a` 和 `b` 指代相同的对象的话，未能处理自移动情况将是一种严重而且微妙的错误。

##### 示例

    class Foo {
        string s;
        int i;
    public:
        Foo& operator=(Foo&& a) noexcept;
        // ...
    };

    Foo& Foo::operator=(Foo&& a) noexcept  // OK，但增加了成本
    {
        if (this == &a) return *this;  // 这行是多余的
        s = std::move(a.s);
        i = a.i;
        return *this;
    }

[自赋值](#rc-copy-self)中反对 `if (this == &a) return *this;` 测试的“每一百万次有一次”的论点，在自移动的情况中更加适当。

##### 注解

并不存在已知的通用方法，以在移动赋值中避免进行 `if (this == &a) return *this;` 测试，又能使其得到正确的结果（亦即，执行 `x = x` 之后不改变 `x` 的值）。

##### 注解

ISO 标准中对标准库容器类仅仅保证了“有效但未指明”的状态。貌似这样做在差不多十年的实验性和产品级代码的使用中并未造成什么问题。如果你找到了反例的话，请联系各位编辑。本条规则更多的是提醒小心并强调完全的安全性。

##### 示例

下面是一种不进行测试而移动一个指针的方法（请想象这段代码来自某个移动赋值的实现）：

    // 从 other.ptr 移动到 this->ptr
    T* temp = other.ptr;
    other.ptr = nullptr;
    delete ptr; // 在自移动情况中，this->ptr 也为 null；delete 是空操作
    ptr = temp; // 在自移动情况中，恢复了原 ptr

##### 强制实施

* 【中级】 在自赋值的情况中，移动赋值运算符不应当使持有已经被 `delete` 或设为 `nullptr` 的指针成员。
* 【无法强制实施】 查看标准库容器类型（包括 `string`）的使用方式，在普通（非性命攸关）使用中将它们当作是安全的。

### <a id="rc-move-noexcept"></a>C.66: 使移动操作 `noexcept`

##### 理由

能够抛出异常的移动操作将违反大多数人的合理假设。
不会抛出异常的移动操作可以更高效地被标准库和语言设施所利用。

##### 示例

    template<typename T>
    class Vector {
    public:
        Vector(Vector&& a) noexcept :elem{a.elem}, sz{a.sz} { a.elem = nullptr; a.sz = 0; }
        Vector& operator=(Vector&& a) noexcept {
            delete elem;
            elem = a.elem; a.elem = nullptr;
            sz   = a.sz;   a.sz   = 0;
            return *this;
        }
        // ...
    private:
        T* elem;
        int sz;
    };

这些操作不会抛出异常。

##### 示例，不好

    template<typename T>
    class Vector2 {
    public:
        Vector2(Vector2&& a) noexcept { *this = a; }             // 直接利用复制操作
        Vector2& operator=(Vector2&& a) noexcept { *this = a; }  // 直接利用复制操作
        // ...
    private:
        T* elem;
        int sz;
    };

`Vector2` 不仅低效，而且由于向量的复制需要分配内存而使其可能抛出异常。

##### 强制实施

【简单】 移动操作应当被标为 `noexcept`。

### <a id="rc-copy-virtual"></a>C.67: 多态类应当抑制公开的移动/复制操作

##### 理由

*多态类*是定义或继承了至少一个虚函数的类。它很可能要被用作其他具有多态行为的派生类的基类。如果不小心将其按值传递了，如果它带有隐式生成的复制构造函数和赋值的话，它就面临发生切片的风险：只会复制派生类对象的基类部分，但将损坏其多态行为。

如果类中没有数据，则使其复制/移动函数 `=delete`。否则，使它们为受保护的。

##### 示例，不好

    class B { // 不好: 多态基类并未抑制复制操作
    public:
        virtual char m() { return 'B'; }
        // ... 没有提供复制操作，使用预置实现 ...
    };

    class D : public B {
    public:
        char m() override { return 'D'; }
        // ...
    };

    void f(B& b)
    {
        auto b2 = b; // 啊呀，对象切片了；b2.m() 将返回 'B'
    }

    D d;
    f(d);

##### 示例

    class B { // 好: 多态类抑制了复制操作
    public:
        B() = default;
        B(const B&) = delete;
        B& operator=(const B&) = delete;
        virtual char m() { return 'B'; }
        // ...
    };

    class D : public B {
    public:
        char m() override { return 'D'; }
        // ...
    };

    void f(B& b)
    {
        auto b2 = b; // ok，编译器能够检测到不恰当的复制并给出警告
    }

    D d;
    f(d);

##### 注解

当需要创建多态对象的深拷贝副本时，应当使用 `clone()` 函数：参见 [C.130](#rh-copy)。

##### 例外

表示异常对象的类应当既是多态的，也可以进行复制构造。

##### 强制实施

* 对带有公开的复制操作的多态类进行标记。
* 对多态类对象的赋值操作进行标记。

## C.other: 默认操作的其他规则

除了语言为之提供默认实现的操作之外，
还有一些操作也是非常基础的，需要对它们的定义给出专门的规则：
比较，`swap`，以及 `hash`。

### <a id="rc-eqdefault"></a>C.80: 当需要明确使用缺省语义时，使用 `=default`

##### 理由

编译器能更正确地实现缺省语义，你所实现的这些函数也不会比编译器更好。

##### 示例

    class Tracer {
        string message;
    public:
        Tracer(const string& m) : message{m} { cerr << "entering " << message << '\n'; }
        ~Tracer() { cerr << "exiting " << message << '\n'; }

        Tracer(const Tracer&) = default;
        Tracer& operator=(const Tracer&) = default;
        Tracer(Tracer&&) noexcept = default;
        Tracer& operator=(Tracer&&) noexcept = default;
    };

由于定义了析构函数，所以也得定义它的复制和移动操作。最佳且最简单的做法就是 `= default`。

##### 示例，不好

    class Tracer2 {
        string message;
    public:
        Tracer2(const string& m) : message{m} { cerr << "entering " << message << '\n'; }
        ~Tracer2() { cerr << "exiting " << message << '\n'; }

        Tracer2(const Tracer2& a) : message{a.message} {}
        Tracer2& operator=(const Tracer2& a) { message = a.message; return *this; }
        Tracer2(Tracer2&& a) noexcept :message{a.message} {}
        Tracer2& operator=(Tracer2&& a) noexcept { message = a.message; return *this; }
    };

把复制和移动操作的函数体写明的做法，既啰嗦又乏味，而且易于出错。编译器则能干得更好。

##### 强制实施

【中级】 特殊操作的函数体不应当和编译器生成的版本具有同样的访问性和语义，因为这样做是多余的。

### <a id="rc-delete"></a>C.81: 当需要关闭缺省行为（且不需要替代的行为）时，使用 `=delete`

##### 理由

少数情况下是不需要提供默认操作的。

##### 示例

    class Immortal {
    public:
        ~Immortal() = delete;   // 不允许进行销毁
        // ...
    };

    void use()
    {
        Immortal ugh;   // 错误: ugh 无法销毁
        Immortal* p = new Immortal{};
        delete p;       // 错误: 无法销毁 *p
    }

##### 示例

`unique_ptr` 可以移动但不能复制。为达成这点，其复制操作是被弃置的。为了避免发生复制，需要将其从左值进行复制的操作定义为 `=delete`：

    template<class T, class D = default_delete<T>> class unique_ptr {
    public:
        // ...
        constexpr unique_ptr() noexcept;
        explicit unique_ptr(pointer p) noexcept;
        // ...
        unique_ptr(unique_ptr&& u) noexcept;   // 移动构造函数
        // ...
        unique_ptr(const unique_ptr&) = delete; // 关闭从左值进行的复制
        // ...
    };

    unique_ptr<int> make();   // 创建“某个对象”并以移动方式返回

    void f()
    {
        unique_ptr<int> pi {};
        auto pi2 {pi};      // 错误: 不存在从左值进行的移动构造函数
        auto pi3 {make()};  // OK，进行移动: make() 的结果为右值
    }

注意，弃置的函数应当是公开的。

##### 强制实施

消除一个默认操作，是（应当是）基于类所要达成的语义考虑的。应当对这样的类保持怀疑，但可以维护一个“确认列表”，由人工断言其语义是正确的。

### <a id="rc-ctor-virtual"></a>C.82: 不要在构造函数和析构函数中调用虚函数

##### 理由

其中所调用的函数其实是目前所构造的对象的函数，而不是可能在派生类中覆盖它的函数。
这可能是最易混淆的。
更糟的是，从构造函数或析构函数中直接或间接调用未被实现的纯虚函数的话，还会导致未定义的行为。

##### 示例，不好

    class Base {
    public:
        virtual void f() = 0;   // 未实现
        virtual void g();       // 有 Base 版本的实现
        virtual void h();       // 有 Base 版本的实现
        virtual ~Base();        // 有 Base 版本的实现
    };

    class Derived : public Base {
    public:
        void g() override;   // 提供 Derived 版本的实现
        void h() final;      // 提供 Derived 版本的实现

        Derived()
        {
            // 不好: 试图调用未经事先的虚函数
            f();

            // 不好: 想要调用 derived::g，但并未发生虚函数分派
            g();

            // 好: 明确说明想要调用的就是写明的版本
            Derived::g();

            // ok，不需要进行限定，h 为 final
            h();
        }
    };

注意，调用一个明确限定的函数时，即便函数是 `virtual` 的，也不会发生虚函数调用。

**参见** [工厂函数](#rc-factory)，以了解如何获得调用派生类函数的效果又不会引发未定义行为。

##### 注解

其实在构造函数和析构函数中调用虚函数并不存在固有的错误。
这种调用的语义是类型安全的。
然而，经验表明这种调用很少真正需要，易于让维护者混淆，而且当被新手使用之后还会成为一种错误来源。

##### 强制实施

* 标记构造函数和析构函数中对虚函数的调用。

### <a id="rc-swap"></a>C.83: 考虑为值类型提供 `noexcept` 的 `swap` 函数

##### 理由

`swap` 对于实现许多惯用法都很有用，其范围包括从平滑地进行对象移动，到轻易实现提供了受保证的提交功能的赋值操作以允许编写具有强异常安全性的调用代码。考虑利用 `swap` 来基于复制构造实现复制赋值操作。另见[析构函数，回收，以及 `swap` 不允许失败](#re-never-fail)。

##### 示例，好

    class Foo {
    public:
        void swap(Foo& rhs) noexcept
        {
            m1.swap(rhs.m1);
            std::swap(m2, rhs.m2);
        }
    private:
        Bar m1;
        int m2;
    };

为调用者方便起见，可以在类型所在的相同命名空间中提供一个非成员的 `swap` 函数。

    void swap(Foo& a, Foo& b)
    {
        a.swap(b);
    }

##### 强制实施

* 可非平凡复制的类型应当提供成员 `swap` 或自由 `swap` 函数的重载。
* 【简单】 当类带有 `swap` 成员函数时，它应当被声明为 `noexcept`。

### <a id="rc-swap-fail"></a>C.84: `swap` 函数不能失败

##### 理由

`swap` 广泛地以假定永不失败的方式被使用，而且如果存在可能失败的 `swap` 函数的话，程序也很难编写为可以正确工作。如果元素类型的 `swap` 会失败的话，标准库的容器和算法也无法正确工作。

##### 示例，不好

    void swap(My_vector& x, My_vector& y)
    {
        auto tmp = x;   // 复制各元素
        x = y;
        y = tmp;
    }

这样做不仅很慢，而且如果为 `tmp` 中的元素进行了内存分配的话，这个 `swap` 也可能抛出异常，并导致使用它的 STL 算法的失败。

##### 强制实施

【简单】 当类带有 `swap` 成员函数时，它应当被声明为 `noexcept`。

### <a id="rc-swap-noexcept"></a>C.85: 使 `swap` 函数 `noexcept`

##### 理由

[`swap` 不能失败](#rc-swap-fail)。
如果 `swap` 试图用异常来退出的话，这就是严重的设计错误，程序最好理解终止 terminate。

##### 强制实施

【简单】 当类带有 `swap` 成员函数时，它应当被声明为 `noexcept`。

### <a id="rc-eq"></a>C.86: 使 `==` 对操作数的类型对称，并使之 `noexcept`

##### 理由

不对称的操作数是出人意料的，而且当可能发生类型转换时也是一种错误来源。
`==` 是一项基础操作，程序员应当能够随意使用而不担心失败。

##### 示例

    struct X {
        string name;
        int number;
    };

    bool operator==(const X& a, const X& b) noexcept {
        return a.name == b.name && a.number == b.number;
    }

##### 示例，不好

    class B {
        string name;
        int number;
        bool operator==(const B& a) const {
            return name == a.name && number == a.number;
        }
        // ...
    };

`B` 的比较函数接受其第二个操作数上的类型转换，但第一个操作数不可以。

##### 注解

如果类带有比如 `double` 的 `NaN` 这样的故障状态的话，就诱惑人们让与故障状态之间的比较抛出异常。
其替代方案是让两个故障状态的比较相等，而任何有效状态和故障状态的比较都不相等。

##### 注解

本条规则适用于所有的常规比较运算符：`!=`，`<`，`<=`，`>`，以及 `>=`。

##### 强制实施

* 对两个参数类型不同的 `operator==()` 进行标记；其他比较运算符也是如此：`!=`，`<`，`<=`，`>`，和 `>=`。
* 对成员 `operator==()` 进行标记；其他比较运算符也是如此：`!=`，`<`，`<=`，`>`，和 `>=`。

### <a id="rc-eq-base"></a>C.87: 请当心基类的 `==`

##### 理由

为类层次编写一个傻瓜式的并且有用处的 `==` 是相当困难的。

##### 示例，不好

    class B {
        string name;
        int number;
    public:
        virtual bool operator==(const B& a) const
        {
             return name == a.name && number == a.number;
        }
        // ...
    };

`B` 的比较函数接受对其第二个操作数的类型转换，但第一个则并非如此。

    class D : public B {
        char character;
    public:
        virtual bool operator==(const D& a) const
        {
            return B::operator==(a) && character == a.character;
        }
        // ...
    };

    B b = ...
    D d = ...
    b == d;    // 比较了 name 和 number，但忽略了 d 的 character
    d == b;    // 比较了 name 和 number，但忽略了 d 的 character
    D d2;
    d == d2;   // 比较了 name、number 和 character
    B& b2 = d2;
    b2 == d;   // 比较了 name 和 number，但忽略了 d2 和 d 的 character

显然有许多使 `==` 在类层次中可以工作的方式，但不成熟的方案是无法适应范围扩展的。

##### 注解

本条规则适用于所有的常规比较运算符：`!=`，`<`，`<=`，`>`，`>=`，以及 `<=>`。

##### 强制实施

* 对虚的 `operator==()` 进行标记；其他比较运算符也是如此：`!=`，`<`，`<=`，`>`，`>=`，以及 `<=>`。

### <a id="rc-hash"></a>C.89: 使 `hash` 函数 `noexcept`

##### 理由

哈希容器的使用者都会间接地使用 `hash`，并且不会预期简单的访问操作也会抛出异常。
这是标准库的一条要求。

##### 示例，不好

    template<>
    struct hash<My_type> {  // 非常不好的 hash 特化
        using result_type = size_t;
        using argument_type = My_type;

        size_t operator()(const My_type & x) const
        {
            size_t xs = x.s.size();
            if (xs < 4) throw Bad_My_type{};    // "没有人期待西班牙宗教裁判所！"
            return hash<size_t>()(x.s.size()) ^ trim(x.s);
        }
    };

    int main()
    {
        unordered_map<My_type, int> m;
        My_type mt{ "asdfg" };
        m[mt] = 7;
        cout << m[My_type{ "asdfg" }] << '\n';
    }

如果你必须定义 `hash` 的特化的话，请尝试单纯地用 `^`（异或 xor）把标准库的 `hash` 特化进行组合。
这样做对于非专业人士来说往往会更好。

##### 强制实施

* 标记可能抛出异常的 `hash`。

### <a id="rc-memset"></a>C.90: 依靠构造函数和赋值运算符，不要依靠 `memset` 和 `memcpy`

##### 理由

构造某个类型的实例的标准 C++ 机制是调用其构造函数。如指导方针 [C.41](#rc-complete) 所述：构造函数应当创建一个已完全初始化的对象。不应当需要进行如用 `memcpy` 来进行的额外初始化。
为适当地做出一个类的副本并维持类型的不变式，类型将提供复制构造函数和/或复制赋值运算符。使用 `memcpy` 来复制一个非可平凡复制的类型具有未定义的行为。这经常会导致切片，或者数据损坏。

##### 示例，好

    struct base {
        virtual void update() = 0;
        std::shared_ptr<int> sp;
    };

    struct derived : public base {
        void update() override {}
    };

##### 示例，不好

    void init(derived& a)
    {
        memset(&a, 0, sizeof(derived));
    }

这样做类型不安全并且会覆写掉虚表。

##### 示例，不好

    void copy(derived& a, derived& b)
    {
        memcpy(&a, &b, sizeof(derived));
    }

这样做同样类型不安全并且会覆写掉虚表。

##### 强制实施

- 对将非可平凡复制类型传递给 `memset` 或 `memcpy` 进行标记。

## <a id="ss-containers"></a>C.con: 容器和其他资源包装类

容器是一种持有某个类型的对象序列的对象；`std::vector` 就是一种典型的容器。
资源包装类是拥有某个资源的类；`std::vector` 是一种典型的资源包装类；它的资源就是其元素的序列。

容器规则概览

* [C.100: 定义容器的时候要遵循 STL](#rcon-stl)
* [C.101: 为容器提供值语义](#rcon-val)
* [C.102: 为容器提供移动操作](#rcon-move)
* [C.103: 为容器提供一个初始化式列表构造函数](#rcon-init)
* [C.104: 为容器提供一个将之置空的默认构造函数](#rcon-empty)
* ???
* [C.109: 当资源包装类具有指针语义时，应提供 `*` 和 `->`](#rcon-ptr)

**参见**: [资源](#s-resource)


### <a id="rcon-stl"></a>C.100: 定义容器的时候要遵循 STL

##### 理由

大多数 C++ 程序员都熟悉 STL 容器，而且它们具有本质上十分健全的设计。

##### 注解

当然也存在其他的本质上健全的设计风格，有时候也存在不遵循
标准程序库的设计风格的各种理由，但在没有非常坚实的理由的情况下，
让实现者和用户都遵循标准，既简单又容易。

尤其是，`std::vector` 和 `std::map` 都提供了相当简单的模型。

##### 示例

    // 简化版本（比如没有分配器）：

    template<typename T>
    class Sorted_vector {
        using value_type = T;
        // ... 各迭代器类型 ...

        Sorted_vector() = default;
        Sorted_vector(initializer_list<T>);    // 初始化式列表构造函数：进行排序并存储
        Sorted_vector(const Sorted_vector&) = default;
        Sorted_vector(Sorted_vector&&) noexcept = default;
        Sorted_vector& operator=(const Sorted_vector&) = default;     // 复制赋值
        Sorted_vector& operator=(Sorted_vector&&) noexcept = default; // 移动赋值
        ~Sorted_vector() = default;

        Sorted_vector(const std::vector<T>& v);   // 存储并排序
        Sorted_vector(std::vector<T>&& v);        // 排序并“窃取表示”

        const T& operator[](int i) const { return rep[i]; }
        // 不提供非 const 的直接访问，以维持顺序

        void push_back(const T&);   // 在正确位置插入（不一定在末尾）
        void push_back(T&&);        // 在正确位置插入（不一定在末尾）

        // ... cbegin(), cend() ...
    private:
        std::vector<T> rep;  // 用一个 std::vector 来持有各元素
    };

    template<typename T> bool operator==(const Sorted_vector<T>&, const Sorted_vector<T>&);
    template<typename T> bool operator!=(const Sorted_vector<T>&, const Sorted_vector<T>&);
    // ...

这段代码遵循 STL 风格但并不完整。
这种做法并不少见。
仅仅为特定的容器提供足以使其有意义的功能即可。
这里的关键在于，定义（对特定容器来说有意义的）符合约定的构造、赋值、析构函数和各迭代器
并提供它们符合约定的语义。
在此基础上，可以根据需要对这个容器进行扩展。
这里添加了来自 `std::vector` 的一些特殊构造函数。

##### 强制实施

???

### <a id="rcon-val"></a>C.101: 为容器提供值语义

##### 理由

常规对象的理解和推理要比非常规对象更加简单。
使人感觉熟悉。

##### 注解

如果有意义的话，要使容器满足 `Regular`（概念）。
尤其是，确保对象与自身的副本比较时相等。

##### 示例

    void f(const Sorted_vector<string>& v)
    {
        Sorted_vector<string> v2 {v};
        if (v != v2)
            cout << "Behavior against reason and logic.\n";
        // ...
    }

##### 强制实施

???

### <a id="rcon-move"></a>C.102: 为容器提供移动操作

##### 理由

容器都有变大的趋势；没有移动构造函数和复制构造函数的对象
进行到处移动可以很昂贵，因而趋使人们转而传递指向它的指针，
从而带来资源管理方面的问题。

##### 示例

    Sorted_vector<int> read_sorted(istream& is)
    {
        vector<int> v;
        cin >> v;   // 假定存在向量的读取操作
        Sorted_vector<int> sv = v;  // 进行排序
        return sv;
    }

用户可以合理地假设返回一个标准程序库风格的容器是廉价的。

##### 强制实施

???

### <a id="rcon-init"></a>C.103: 为容器提供一个初始化式列表构造函数

##### 理由

人们期望能够以一组值来初始化一个容器。
使人感觉熟悉。

##### 示例

    Sorted_vector<int> sv {1, 3, -1, 7, 0, 0}; // Sorted_vector 按需对其元素进行排序

##### 强制实施

???

### <a id="rcon-empty"></a>C.104: 为容器提供一个将之置空的默认构造函数

##### 理由

使其满足 `Regular`。

##### 示例

    vector<Sorted_sequence<string>> vs(100);    // 100 个 Sorted_sequence，值均为 ""

##### 强制实施

???

### <a id="rcon-ptr"></a>C.109: 当资源包装类具有指针语义时，应提供 `*` 和 `->`

##### 理由

这正是对指针所预期的行为，
使人感觉熟悉。

##### 示例

    ???

##### 强制实施

???

## <a id="ss-lambdas"></a>C.lambdas: 函数对象和 lambda

函数对象是提供了重载的 `()` 的对象，因此可以进行调用。
Lambda 表达式（通常通俗地简称为“lambda”）是一种产生函数对象的写法。
函数对象应当可廉价地复制（因此可以[按值传递](#rf-in)）。

概要：

* [F.10: 若操作可被重用，则应为其命名](#rf-name)
* [F.11: 当需要仅在一处使用的简单函数对象时使用无名 lambda]([#Rf-lambda)
* [F.50: 无法用函数达成（捕捉局部变量，或者编写局部函数）时，应使用 lambda](#rf-capture-vs-overload)
* [F.52: 在将被局部范围内使用（包括将之传递给算法）的 lambda 中优先按引用捕捉](#rf-reference-capture)
* [F.53: 在不被局部范围内使用（包括存储在堆上，或传递给其他线程）的 lambda 中避免按引用捕捉](#rf-value-capture)
* [ES.28: 针对复杂的初始化（尤其是 `const` 变量）使用 lambda](#res-lambda-init)

## <a id="ss-hier"></a>C.hier: 类层次（OOP）

构建类层次（仅）用于表达一组按层次组织的概念。
基类通常都表现为接口。
类层次有两种主要的用法，它们通常被叫做实现继承和接口继承。

类层次规则概览：

* [C.120: 类层次（仅）用于表达具有天然层次化结构的概念](#rh-domain)
* [C.121: 如果基类被用作接口的话，应使其成为纯抽象类](#rh-abstract)
* [C.122: 当需要完全区分接口和实现时，应当用抽象类作为接口](#rh-separation)

类层次的设计规则概览：

* [C.126: 抽象类通常并不需要用户编写的构造函数](#rh-abstract-ctor)
* [C.127: 带有虚函数的类应当带有虚的或受保护的析构函数](#rh-dtor)
* [C.128: 虚函数应当指明 `virtual`、`override`、`final` 三者之一](#rh-override)
* [C.129: 当设计类层次时，应区分实现继承和接口继承](#rh-kind)
* [C.130: 多态类的深拷贝；优先采用虚函数 `clone` 来替代公开复制构造/赋值](#rh-copy)
* [C.131: 避免无价值的取值和设值函数](#rh-get)
* [C.132: 请勿无理由地使函数 `virtual`](#rh-virtual)
* [C.133: 避免 `protected` 数据](#rh-protected)
* [C.134: 确保所有非 `const` 数据成员有相同的访问级别](#rh-public)
* [C.135: 用多继承来表达多个不同的接口](#rh-mi-interface)
* [C.136: 用多继承来表达一些实现特性的合并](#rh-mi-implementation)
* [C.137: 用 `virtual` 基类以避免过于通用的基类](#rh-vbase)
* [C.138: 用 `using` 来为派生类和其基类建立重载集合](#rh-using)
* [C.139: 对类运用 `final` 应当保守](#rh-final)
* [C.140: 不要在虚函数和其覆盖函数上提供不同的默认参数](#rh-virtual-default-arg)

对类层次中的对象进行访问的规则概览：

* [C.145: 通过指针和引用来访问多态对象](#rh-poly)
* [C.146: 当无法避免在类层次上进行导航时应使用 `dynamic_cast`](#rh-dynamic_cast)
* [C.147: 当查找所需类的失败被当做一种错误时，应当对引用类型使用 `dynamic_cast`](#rh-ref-cast)
* [C.148: 当查找所需类的失败被当做一种有效的可能情况时，应当对指针类型使用 `dynamic_cast`](#rh-ptr-cast)
* [C.149: 用 `unique_ptr` 或 `shared_ptr` 来避免忘记对以 `new` 所创建的对象进行 `delete` 的情况](#rh-smart)
* [C.150: 用 `make_unique()` 来构建由 `unique_ptr` 所拥有的对象](#rh-make_unique)
* [C.151: 用 `make_shared()` 来构建由 `shared_ptr` 所拥有的对象](#rh-make_shared)
* [C.152: 禁止把指向派生类对象的数组的指针赋值给指向基类的指针](#rh-array)
* [C.153: 优先采用虚函数而不是强制转换](#rh-use-virtual)

### <a id="rh-domain"></a>C.120: 使用类层次来表达具有天然层次化结构的概念

##### 理由

直接在代码中表达想法可以简化理解和维护工作。应当保证各个基类所表达的想法与全部派生类型精确匹配，并且确实找不到比使用继承所带来的紧耦合方式更好的表达方式。

当单纯使用数据成员就能搞定时请*不要*使用继承。这种情况通常意味着派生类型需要覆盖某个基类虚函数或者需要访问某个受保护成员。

##### 示例

    class DrawableUIElement {
    public:
        virtual void render() const = 0;
        // ...
    };

    class AbstractButton : public DrawableUIElement {
    public:
        virtual void onClick() = 0;
        // ...
    };

    class PushButton : public AbstractButton {
        void render() const override;
        void onClick() override;
        // ...
    };

    class Checkbox : public AbstractButton {
    // ...
    };

##### 示例，不好

请*不要*把非层次化的领域概念表示成类层次。

    template<typename T>
    class Container {
    public:
        // 列表操作：
        virtual T& get() = 0;
        virtual void put(T&) = 0;
        virtual void insert(Position) = 0;
        // ...
        // 向量操作：
        virtual T& operator[](int) = 0;
        virtual void sort() = 0;
        // ...
        // 树操作：
        virtual void balance() = 0;
        // ...
    };

大多数派生类都无法恰当实现这个接口所要求的大多数函数。
因而这个基类成为了一个实现负担。
此外，`Container` 的使用者无法依靠成员函数来相当高效地确实实施某个有意义的操作；
它可能会抛出某个异常。
因此使用者只得诉诸于运行时检查，并且
放弃使用这个（过于）一般化的接口，代之以某种运行时类型查询（比如 `dynamic_cast`）所确定的接口。

##### 强制实施

* 寻找带有许多不干别的只会抛出异常的成员的类。
* 对非公用基类 `B` 的每次使用进行标记，其中派生类 `D` 并未覆盖 `B` 的某个虚函数，或访问某个受保护成员，而 `B` 并非以下情况：为空，为 `D` 的模板参数或参数包组，或者为以 `D` 所特化的类模板。

### <a id="rh-abstract"></a>C.121: 如果基类被用作接口的话，应使其成为纯抽象类

##### 理由

不包含数据的类更加稳定（更不脆弱易变）。
接口通常都应当全部由公开的纯虚函数和一个预置的或为空的虚析构函数组成。

##### 示例

    class My_interface {
    public:
        // ... 只有一个纯虚函数 ...
        virtual ~My_interface() {}   // 或者 =default
    };

##### 示例，不好

    class Goof {
    public:
        // ... 只有一个纯虚函数 ...
        // 没有虚析构函数
    };

    class Derived : public Goof {
        string s;
        // ...
    };

    void use()
    {
        unique_ptr<Goof> p {new Derived{"here we go"}};
        f(p.get()); // 通过 Goof 接口使用 Derived
        g(p.get()); // 通过 Goof 接口使用 Derived
    } // 泄漏

`Derived` 是通过其 `Goof` 接口而被 `delete` 的，而它的 `string` 则泄漏了。
为 `Goof` 提供虚析构函数就能使其都正常工作。


##### 强制实施

* 对任何含有数据成员同时带有并非从基类继承的可被覆盖（非 `final`）的虚函数的类给出警告。

### <a id="rh-separation"></a>C.122: 当需要完全区分接口和实现时，应当用抽象类作为接口

##### 理由

诸如在 ABI（连接）边界这种地方。

##### 示例

    struct Device {
        virtual ~Device() = default;
        virtual void write(span<const char> outbuf) = 0;
        virtual void read(span<char> inbuf) = 0;
    };

    class D1 : public Device {
        // ... 数据 ...

        void write(span<const char> outbuf) override;
        void read(span<char> inbuf) override;
    };

    class D2 : public Device {
        // ... 不同的数据 ...

        void write(span<const char> outbuf) override;
        void read(span<char> inbuf) override;
    };

使用者可以通过由 `Device` 所提供的接口来互换地使用 `D1` 和 `D2`。
而且，只要其访问一直是通过 `Device` 进行的话，也可以以与老版本二进制不兼容的方式来更新 `D1` 和 `D2`。

##### 强制实施

    ???

## C.hierclass: 类层次的设计：

### <a id="rh-abstract-ctor"></a>C.126: 抽象类通常并不需要用户编写的构造函数

##### 理由

通常抽象类并没有任何需要由构造函数来初始化的对象。

##### 示例

    class Shape {
    public:
        // 抽象基类中不需要用户编写的构造函数
        virtual Point center() const = 0;    // 纯虚
        virtual void move(Point to) = 0;
        // ... 其他纯虚函数 ...
        virtual ~Shape() {}                 // 析构函数
    };

    class Circle : public Shape {
    public:
        Circle(Point p, int rad);           // 派生类中的构造函数
        Point center() const override { return x; }
    };

##### 例外

* 有任务的基类构造函数，比如把对象注册到什么地方的时候，可能是需要构造函数的。
* 在极端少见的情况下，你可能发觉让抽象类来包含一点所有派生类都会共享的数据是有意义的
  （比如说，使用情况统计数据，调试信息等）；这样的类倾向于带有构造函数。但应当警醒的是：这样的类也倾向于要求进行虚继承。

##### 强制实施

对带有构造函数的抽象类进行标记。

### <a id="rh-dtor"></a>C.127: 带有虚函数的类应当带有虚的或受保护的析构函数

##### 理由

带有虚函数的类通常是通过指向基类的指针来使用的。一般来说，最后一个使用者必须在基类指针上执行 `delete`，这常常是通过基类智能指针来做到的，因而析构函数应当为 `public` 和 `virtual`。而不那么常见的情况是当并不打算支持通过基类指针来删除时，这时析构函数应当为 `protected` 和非 `virtual`；参见 [C.35](#rc-dtor-virtual)。

##### 示例，不好

    struct B {
        virtual int f() = 0;
        // ... 没有用户编写的析构函数，缺省为 public 非 virtual ...
    };

    // 不好：继承于没有虚析构函数的类
    struct D : B {
        string s {"default"};
        // ...
    };

    void use()
    {
        unique_ptr<B> p = make_unique<D>();
        // ...
    } // 未定义行为。可能仅仅调用了 B::~B 而字符串则被泄漏了

##### 注解

有些人不遵守本条规则，因为他们打算仅通过 `shared_ptr` 来使用这些类：`std::shared_ptr<B> p = std::make_shared<D>(args);` 这种情况下，由共享指针来负责删除对象，因此并不会发生不适当的基类 `delete` 所导致的泄漏。坚持一贯这样做的人可能会得到假阳性的警告，但这条规则其实很重要——当通过 `make_unique` 分配对象时会如何呢？这样的话是不安全的，除非 `B` 的作者保证它不会被误用，比如可以让所有的构造函数为私有的并提供一个工厂函数，来强制保证分配都是通过 `make_shared` 进行。

##### 强制实施

* 带有任何虚函数的类的析构函数应当要么是 `public` 和 `virtual`，要么是 `protected` 和非 `virtual` 的。
* 把对带有虚函数但没有虚析构函数的类的 `delete` 标记出来。

### <a id="rh-override"></a>C.128: 虚函数应当指明 `virtual`、`override`、`final` 三者之一

##### 理由

可读性。
检测错误。
明确写下的 `virtual`、`override` 或 `final` 是自说明的，并使编译器可以检查到基类和派生类之间的类型和/或名字的不匹配。不过写出超过一个则不仅多余而且是潜在的错误来源。

可以遵循简单明确的含义：

* `virtual` 刚好仅仅表明“这是一个新的虚函数”。
* `override` 刚好仅仅表明“这是一个非最终覆盖函数”。
* `final` 刚好仅仅表明“这是一个最终覆盖函数”。

##### 示例，不好

    struct B {
        void f1(int);
        virtual void f2(int) const;
        virtual void f3(int);
        // ...
    };

    struct D : B {
        void f1(int);        // 不好（希望会有警告）: D::f1() 隐藏了 B::f1()
        void f2(int) const;  // 不好（但惯用且合法）: 没有明确 override
        void f3(double);     // 不好（希望会有警告）: D::f3() 隐藏了 B::f3()
        // ...
    };

##### 示例，好

    struct Better : B {
        void f1(int) override;        // 错误（被发现）: Better::f1() 隐藏了 B::f1()
        void f2(int) const override;
        void f3(double) override;     // 错误（被发现）: Better::f3() 隐藏了 B::f3()
        // ...
    };

#### 讨论

我们希望消除两种特定类型的错误：

* **隐式虚函数**: 程序员有意使函数隐含为虚函数，而它确实如此（但代码的读者搞不清楚这点）；或者，程序员有意使函数隐含为虚函数，但它并非如此（例如，由于微妙的参数列表不匹配所导致）；或者，程序员并非有意使函数为虚函数，但它却成为虚函数（由于它刚好与基类中的某个虚函数具有相同的签名）
* **隐式覆盖**: 程序员有意使函数隐式地成为覆盖函数，而它确实如此（但代码的读者搞不清楚这点）；或者，程序员有意使函数隐式地成为覆盖函数，但它并非如此（例如，由于微妙的参数列表不匹配）；或者，程序员并非有意使函数成为覆盖函数，但它却成为覆盖函数（由于它刚好与基类中的某个虚函数具有相同的签名 -- 注意无论这个函数是否被显式声明为虚函数都会发生这个问题，因为程序员的意图既可能是要创建一个新的虚函数也可能要创建一个新的非虚函数）

注意：对于定义为 `final` 的类来说，是否在一个虚函数上标记 `override` 或 `final` 是无所谓的。

注意：对函数使用 `final` 要保守。它不一定会带来优化，但会排除进一步的覆盖。

##### 强制实施

* 比较基类和派生类中的虚函数的名字，并对并未进行覆盖的相同名字的使用进行标记。
* 对既没有 `override` 也没有 `final` 的覆盖函数进行标记。
* 对函数声明中使用 `virtual`、`override` 和 `final` 中超过一个的情况进行标记。

### <a id="rh-kind"></a>C.129: 当设计类层次时，应区分实现继承和接口继承

##### 理由

接口中的实现细节会使接口变得脆弱；
就是说，当实现被改变时其用户不得不进行重新编译。
基类中的数据增加了基类实现的复杂性，而且会导致代码的重复。

##### 注解

定义：

* 接口继承，是使用继承来把用户和实现进行分离，
特别是允许添加和修改派生类而不影响基类的用户。
* 实现继承，是使用继承来简化新设施的实现，
通过将有用的操作提供给相关的新操作的实现者（有时候称作“差异式编程”）。

纯粹的接口类只是一组纯虚函数；参见 [I.25](#ri-abstract)。

在早期的 OOP 时代（比如 80 和 90 年代），实现继承和接口继承通常是混在一起的，
而不良习惯则很难改掉。
即便是现在，这种混合在旧代码和老式的教学材料中也不少见。

对两种继承进行区分的重要性随着以下情形而增长：

* 类层次的大小（比如几十个派生类），
* 类层次的使用时期（比如几十年），以及
* 使用这个类层次的独立团体的数量
（比如，可能对分发和更新某个基类造成困难）


##### 示例，不好

    class Shape {   // 不好，混合了接口和实现
    public:
        Shape();
        Shape(Point ce = {0, 0}, Color co = none): cent{ce}, col {co} { /* ... */}

        Point center() const { return cent; }
        Color color() const { return col; }

        virtual void rotate(int) = 0;
        virtual void move(Point p) { cent = p; redraw(); }

        virtual void redraw();

        // ...
    private:
        Point cent;
        Color col;
    };

    class Circle : public Shape {
    public:
        Circle(Point c, int r) : Shape{c}, rad{r} { /* ... */ }

        // ...
    private:
        int rad;
    };

    class Triangle : public Shape {
    public:
        Triangle(Point p1, Point p2, Point p3); // 计算中心点
        // ...
    };

问题：

* 随着类层次的增长和向 `Shape` 添加更多的数据，构造函数会越发难于编写和维护。
* 为什么要计算 `Triangle` 的中心点？我们也许从不用它。
* 向 `Shape` 添加新的数据成员（比如绘制风格或者画布）
将导致所有派生于 `Shape` 的类和所有使用 `Shape` 的代码都需要进行复审，可能需要修改，而且很可能需要重新编译。

`Shape::move()` 的实现就是实现继承的一个例子：
我们为所有派生类一次性定义 `move()`。
在这种基类成员函数实现中的代码越多，在基类中放入的共享数据越多，
就能获得越多的好处——而类层次则越不稳定。

##### 示例

这个 `Shape` 类层次可以用接口继承重新编写：

    class Shape {  // 纯接口
    public:
        virtual Point center() const = 0;
        virtual Color color() const = 0;

        virtual void rotate(int) = 0;
        virtual void move(Point p) = 0;

        virtual void redraw() = 0;

        // ...
    };

注意纯接口很少会有构造函数：没什么需要构造的。

    class Circle : public Shape {
    public:
        Circle(Point c, int r, Color c) : cent{c}, rad{r}, col{c} { /* ... */ }

        Point center() const override { return cent; }
        Color color() const override { return col; }

        // ...
    private:
        Point cent;
        int rad;
        Color col;
    };

接口不再那么脆弱了，但成员函数的实现需要做更多工作。
比如说，每个派生于 `Shape` 的类都得实现 `center`。

##### 示例，双类层次

如何才能同时获得接口类层次的稳定类层次的好处和实现继承的实现重用的好处呢？
一种流行的技巧是双类层次。
有许多实现双类层次的方式；这里，我们使用一种多重继承形式。

首先规划一个接口类的层次：

    class Shape {   // 纯接口
    public:
        virtual Point center() const = 0;
        virtual Color color() const = 0;

        virtual void rotate(int) = 0;
        virtual void move(Point p) = 0;

        virtual void redraw() = 0;

        // ...
    };

    class Circle : public virtual Shape {   // 纯接口
    public:
        virtual int radius() = 0;
        // ...
    };

为使这个接口有用处，我们必须提供其实现类（我们这里用相同的名字，但放入 `Impl` 命名空间）：

    class Impl::Shape : public virtual ::Shape { // 实现
    public:
        // 构造函数，析构函数
        // ...
        Point center() const override { /* ... */ }
        Color color() const override { /* ... */ }

        void rotate(int) override { /* ... */ }
        void move(Point p) override { /* ... */ }

        void redraw() override { /* ... */ }

        // ...
    };

现在 `Shape` 是一个贫乏的具有一个实现的类的例子，
但还请谅解，因为这只是用来展现一种针对更复杂的类层次的技巧的简单例子。

    class Impl::Circle : public virtual ::Circle, public Impl::Shape {   // 实现
    public:
        // 构造函数，析构函数

        int radius() override { /* ... */ }
        // ...
    };

我们可以通过添加一个 `Smiley` 类来扩展它（:-)）：

    class Smiley : public virtual Circle { // 纯接口
    public:
        // ...
    };

    class Impl::Smiley : public virtual ::Smiley, public Impl::Circle {   // 实现
    public:
        // 构造函数，析构函数
        // ...
    }

这里有两个类层次：

* 接口：Smiley -> Circle -> Shape
* 实现：Impl::Smiley -> Impl::Circle -> Impl::Shape

由于每个实现都同时派生于其接口和其实现基类，我们因此获得了一个晶格（DAG）：

    Smiley     ->         Circle     ->  Shape
      ^                     ^               ^
      |                     |               |
    Impl::Smiley -> Impl::Circle -> Impl::Shape

我们曾经说过，这只是用来构造双类层次的一种方式。

可以直接使用实现类层次，而不用通过抽象接口来进行。

    void work_with_shape(Shape&);

    int user()
    {
        Impl::Smiley my_smiley{ /* args */ };   // 创建具体的形状
        // ...
        my_smiley.some_member();        // 直接使用实现类
        // ...
        work_with_shape(my_smiley);     // 通过抽象接口使用实现
        // ...
    }

这种做法在实现类带有并未由抽象接口提供的成员时，
或者当直接使用某个成员具有优化机会（比如当实现成员函数为 `final`）时，比较有用。

##### 注解

分离接口和实现的另一个（相关的）技巧是 [Pimpl](#ri-pimpl)。

##### 注解

在提供公共的功能时，我们通常需要在作为（有实现的）基类函数和（在某个实现命名空间中的）
自由函数之间进行选择。
基类能够提供更简短的写法，以及更容易访问（基类中的）共享数据，
但所付出的是其功能将仅能被这个类层次的用户所使用。

##### 强制实施

* 若派生类向基类转换的基类同时具有数据和虚函数，则对其进行标记
（但排除在派生类成员中对基类成员的调用）。
* ???


### <a id="rh-copy"></a>C.130: 多态类的深拷贝；优先采用虚函数 `clone` 来替代公开复制构造/赋值

##### 理由

由于切片的问题，不鼓励多态类的复制操作，参见 [C.67](#rc-copy-virtual)。如果确实需要复制语义的话，应当进行深复制：提供一个虚 `clone` 函数，它复制的是真正的最终派生类型，并返回指向新对象的具有所有权的指针，而且在派生类中它返回的也是派生类型（利用协变返回类型）。

##### 示例

    class B {
    public:
        B() = default;
        virtual ~B() = default;
        virtual gsl::owner<B*> clone() const = 0;
    protected:
         B(const B&) = default;
         B& operator=(const B&) = default;
         B(B&&) noexcept = default;
         B& operator=(B&&) noexcept = default;
        // ...
    };

    class D : public B {
    public:
        gsl::owner<D*> clone() override
        {
            return new D{*this};
        }
    };

通常来说，推荐使用智能指针来表示所有权（参见 [R.20](#rr-owner)）。不过根据语言规则，协变返回类型不能是智能指针：当 `B::clone` 返回 `unique_ptr<B>` 时 `D::clone` 不能返回 `unique_ptr<D>`。因此，你得在所有覆盖函数中统一都返回 `unique_ptr<B>`，或者也可以使用[指导方针支持库](#ss-views)中的 `owner<>` 工具类。



### <a id="rh-get"></a>C.131: 避免无价值的取值和设值函数

##### 理由

无价值的取值和设值函数没有提供语义价值；让数据项自己 `public` 是一样的。

##### 示例

    class Point {   // 不好：啰嗦
        int x;
        int y;
    public:
        Point(int xx, int yy) : x{xx}, y{yy} { }
        int get_x() const { return x; }
        void set_x(int xx) { x = xx; }
        int get_y() const { return y; }
        void set_y(int yy) { y = yy; }
        // 没有有行为的成员函数
    };

应当考虑把这个类变为 `struct`——就是一组没有行为的变量，全部都是公开数据而没有成员函数。

    struct Point {
        int x {0};
        int y {0};
    };

注意，我们可以为成员变量提供默认初始化式：[C.49: 优先进行初始化而不是在构造函数中赋值](#rc-initialize).

##### 注解

这条规则的关键在于取值和设值函数的语义是否是平凡的。虽然并非是对“平凡”的完整定义，但我们考虑在取值/设值函数，以及当使用公开数据成员之间除了语法上的差别之外是否存在什么差别。非平凡的语义的例子可能有：维护类的不变式，或者在某种内部类型和接口类型之间进行的转换。

##### 强制实施

对大量仅提供单纯的成员访问而没有其他语义的 `get` 和 `set` 成员函数进行标记。

### <a id="rh-virtual"></a>C.132: 请勿无理由地使函数 `virtual`

##### 理由

多余的 `virtual` 会增加运行时间和对象代码的大小。
虚函数可以被覆盖，因此派生类中可能因此发生错误。
虚函数保证会在模板化的层次中造成代码重复。

##### 示例，不好

    template<class T>
    class Vector {
    public:
        // ...
        virtual int size() const { return sz; }   // 不好：派生类能得到什么好处？
    private:
        T* elem;   // 元素
        int sz;    // 元素的数量
    };

这种类型的“向量”并非是为了让人用作基类的。

##### 强制实施

* 对带有虚函数但没有派生类的类进行标记。
* 对所有成员函数都为虚函数并带有实现的类进行标记。

### <a id="rh-protected"></a>C.133: 避免 `protected` 数据

##### 理由

`protected` 数据是复杂性和错误的一种来源。
`protected` 数据会把不变式的陈述搞复杂。
`protected` 数据天生违反了避免把数据放入基类的指导原则，而这样通常还会导致不得不采用虚继承。

##### 示例，不好

    class Shape {
    public:
        // ... 接口函数 ...
    protected:
        // 为派生类所使用的数据：
        Color fill_color;
        Color edge_color;
        Style st;
    };

这样，由每个所定义的 `Shape` 来保证对受保护数据正确进行操作。
这样做一度很流行，但同样是维护性问题的一种主要来源。
在大型的类层次中，很难保持对受保护数据的一致性使用，因为代码可能有很多，
分散于大量的类之中。
能够触碰这些数据的类的集合是开放的：任何人都可以派生一个新的类并开始操作这些受保护数据。
检查这些类的完整集合通常是不可能做到的，因此对类的表示进行任何改动都是不可行的。
这些受保护数据上并没有强加的不变式；它们更像是一组全局变量。
受保护数据在大块代码中事实上成为了全局变量。

##### 注解

受保护数据经常看起来倾向于允许通过派生来进行任意的改进。
而通常你得到的是肆无忌惮的改动和各种错误。
应当[优先采用 `private` 数据](#rc-private)并提供良好定义并强加的不变式。
或者通常更好的做法是，[不要在用作接口的任何类中存放数据](#rh-abstract)。

##### 注解

受保护的成员函数则没有问题。

##### 强制实施

对带有 `protected` 数据的类进行标记。

### <a id="rh-public"></a>C.134: 确保所有非 `const` 数据成员有相同的访问级别

##### 理由

防止出现会导致错误的逻辑混乱。
当非 `const` 数据成员的访问级别不同时，这个类型就会在它应当做什么上出现混乱。
这个类型是用来维持某个不变式的类型，还是仅仅集合了一组值而已？

##### 探讨

其核心问题是：哪段代码应当负责为变量维护有意义/正确的值？

确切地说存在两种数据成员：

* A: 不参与对象的不变式的数据成员。这些成员的任何值的互相组合都是有效的。
* B: 参与对象不变式的数据成员。并非每种值组合都是有意义的（否则就没有不变式了）。因此所有具有对这些变量的写访问权限的代码都应当了解这个不变式，了解其语义，并了解（而且积极实现并加强）用以维持值的正确性的规则。

A 类别中的数据成员就应当是 `public`（或者很少情况下，当你只想在派生类中见到它们时为 `protected`）。不需要对它们进行封装。系统中的所有代码都可以见到并操控它们。

B 类别中的数据成员应当为 `private` 或 `const`。这是因为封装很重要。让它们非 `private` 且非 `const` 可能意味着对象无法控制自身的状态：这个类以外的无限量的代码可能都需要了解这个不变式，并精确地参与它的维护工作——当这些数据成员都是 `public` 时，这可能包括使用这个对象的所有调用方代码；而当它们为 `protected` 时，这可能包括当前以及未来的派生类的所有代码。这会导致易碎的且紧耦合的代码，而且很快将会成为维护的噩梦。任何代码如果把这些数据成员设值为无效的或者预料之外的值的组合，都可能搞坏对象以及随后对象的所有使用。

大多数的类要么都是 A，要么都是 B：

* *全 public*: 如果编写的是聚集一组变量而没有在这些变量之间维护不变式的话，所有这些变量都应当为 `public`。
  [依照惯例，应当把这样的类声明为 `struct` 而不是 `class`](#rc-struct)
* *全 private*: 如果编写的类型维护了某个不变式，则所有的非 `const` 变量都应当是 `private`——应当对它进行封装。

##### 例外

偶尔会出现混合了 A 和 B 的类，通常是为了方便调试。一个被封装的对象可能包含如非 `const` 调试信息的某种东西，它并不是不变式的一部分，因此属于 A 类别——其实它并非对象的值的一部分，也不是这个对象的有意义的可观察状态。这种情况下，A 类别的部分应当按 A 的方式对待（使之 `public`，或罕见情况下当它们只应对派生类可见时，使之为 `protected`），而 B 类别的部分应当仍按 B 的方式对待（`private` 或 `const`）。

##### 强制实施

对包含具有不同访问级别的非 `const` 数据成员的类给出标记。

### <a id="rh-mi-interface"></a>C.135: 用多继承来表达多个不同的接口

##### 理由

并非所有的类都应当支持全部的接口，而且并非所有的调用方都应当处理所有的操作。
尤其应当把巨大的接口拆解为可以被某个给定派生类所支持的不同的行为“方面”。

##### 示例

    class iostream : public istream, public ostream {   // 充分简化
        // ...
    };

`istream` 提供了输入操作的接口；`ostream` 提供了输出操作的接口。
`iostream` 提供了 `istream` 和 `ostream` 接口的并集，以及在单个流上同时允许二者所需的同步操作。

##### 注解

这是非常常见的继承的用法，因为需要同一个实现提供多个不同接口是很常见的，
而通常把这种接口组织到一个单根层次中都是不容易的或者不自然的。

##### 注解

这样的接口通常都是抽象类。

##### 强制实施

???

### <a id="rh-mi-implementation"></a>C.136: 用多继承来表达一些实现特性的合并

##### 理由

一些形式的混元（Mixin）带有状态，并通常会有对其状态提供的操作。
如果这些操作是虚的，就必须进行继承，而如果不是虚的，使用继承也可以避免例行代码和转发函数。

##### 示例

      class iostream : public istream, public ostream {   // 充分简化
        // ...
    };

`istream` 提供了输入操作的接口（以及一些数据）；`ostream` 提供了输出操作的接口（以及一些数据）。
`iostream` 提供了 `istream` 和 `ostream` 接口的并集，以及在单个流上同时允许二者所需的同步操作。

##### 注解

这是一种相对少见的用法，因为实现通常都可以被组织到一个单根层次之中。

##### 示例

有时候，“实现特性”更像是“混元”，决定实现的行为，
并向其中注入成员以使该实现提供其所要求的策略。
相关的例子可以参考 `std::enable_shared_from_this`
或者 boost.intrusive 中的各种基类（例如 `list_base_hook` 和 `intrusive_ref_counter`）。

##### 强制实施

???

### <a id="rh-vbase"></a>C.137: 用 `virtual` 基类以避免过于通用的基类

##### 理由

允许共享的数据和接口的分离。
避免将所有共享数据都被放到一个终极基类之中。

##### 示例

    struct Interface {
        virtual void f();
        virtual int g();
        // ... 没有数据 ...
    };

    class Utility {  // 带有数据
        void utility1();
        virtual void utility2();    // 定制点
    public:
        int x;
        int y;
    };

    class Derive1 : public Interface, virtual protected Utility {
        // 覆盖了 Iterface 的函数
        // 可能覆盖 Utility 的虚函数
        // ...
    };

    class Derive2 : public Interface, virtual protected Utility {
        // 覆盖了 Iterface 的函数
        // 可能覆盖 Utility 的虚函数
        // ...
    };

如果许多派生类都共享了显著的“实现细节”，弄一个 `Utility` 出来就是有意义的。


##### 注解

很明显，这个例子过于“理论化”，但确实很难找到一个*小型*的现实例子出来。
`Interface` 是一个[接口层次](#rh-abstract)的根，
而 `Utility` 则是一个[实现层次](#rh-kind)的根。
[一个稍微更现实的例子](https://www.quora.com/What-are-the-uses-and-advantages-of-virtual-base-class-in-C%2B%2B/answer/Lance-Diduck)，有一些解释。

##### 注解

将层次结构线性化通常是更好的方案。

##### 强制实施

对接口和实现混合的层次进行标记。

### <a id="rh-using"></a>C.138: 用 `using` 来为派生类和其基类建立重载集合

##### 理由

没有 using 声明的话，派生类的成员函数将会隐藏全部其所继承的重载集合。

##### 示例，不好

    #include <iostream>
    class B {
    public:
        virtual int f(int i) { std::cout << "f(int): "; return i; }
        virtual double f(double d) { std::cout << "f(double): "; return d; }
        virtual ~B() = default;
    };
    class D: public B {
    public:
        int f(int i) override { std::cout << "f(int): "; return i + 1; }
    };
    int main()
    {
        D d;
        std::cout << d.f(2) << '\n';   // 打印 "f(int): 3"
        std::cout << d.f(2.3) << '\n'; // 打印 "f(int): 3"
    }

##### 示例，好

    class D: public B {
    public:
        int f(int i) override { std::cout << "f(int): "; return i + 1; }
        using B::f; // 展露了 f(double)
    };

##### 注解

这个问题对虚的和非虚的成员函数都有影响。

对于可变基类，C++17 引入了一种 using 声明的可变形式：

    template<class... Ts>
    struct Overloader : Ts... {
        using Ts::operator()...; // 展露了每个基类中的 operator()
    };

##### 强制实施

诊断名字隐藏情况

### <a id="rh-final"></a>C.139: 对类运用 `final` 应当保守

##### 理由

用 `final` 类来把类层次封闭很少是由于逻辑上的原因而必须的，并可能破坏掉类层次的可扩展性。

##### 示例，不好

    class Widget { /* ... */ };

    // 没有人会想要改进 My_widget（你可能这么觉得）
    class My_widget final : public Widget { /* ... */ };

    class My_improved_widget : public My_widget { /* ... */ };  // 错误: 办不到了

##### 注解

并非每个类都要作为基类的。
大多数标准库的类都是这样（比如，`std::vector` 和 `std::string` 都并非为了派生而设计）。
这条规则是关于在准备作为某个类层次的接口的带有虚函数的类中有关 `final` 的使用的。

##### 注解

用 `final` 来把单个的虚函数封印则是易错的，因为在定义和覆盖一组函数时，`final` 是很容易被忽视的。
幸运的是，编译器能够捕捉到这种错误：你无法在派生类中重新声明/重新打开一个 `final` 成员。

##### 注解

有关 `final` 带来的性能提升的断言是需要证实的。
非常常见的是，这种断言都是基于推测或者在其他语言上的经验而来的。

有一些例子中的 `final` 对于逻辑和性能因素来说可能都是重要的。
一个例子是编译器和语言分析工具中的性能关键的 AST 层次结构。
其中并非每年都会添加新的派生类，而且只有程序库的实现者会做这种事。
不过，误用（或者至少曾经的误用）的情况远远比这常见。

##### 强制实施

标记出所有在类上使用的 `final`。


### <a id="rh-virtual-default-arg"></a>C.140: 不要在虚函数和其覆盖函数上提供不同的默认参数

##### 理由

这会造成混乱：覆盖函数是不会继承默认参数的。

##### 示例，不好

    class Base {
    public:
        virtual int multiply(int value, int factor = 2) = 0;
        virtual ~Base() = default;
    };

    class Derived : public Base {
    public:
        int multiply(int value, int factor = 10) override;
    };

    Derived d;
    Base& b = d;

    b.multiply(10);  // 这两次调用将会调用相同的函数，但分别
    d.multiply(10);  // 使用不同的默认实参，因此获得不同的结果

##### 强制实施

对虚函数默认参数中，如果其在基类和派生类的声明中是不同的，则对其进行标记。

## C.hier-access: 对类层次中的对象进行访问

### <a id="rh-poly"></a>C.145: 通过指针和引用来访问多态对象

##### 理由

如果类带有虚函数的话，你（通常）并不知道你所使用的函数是由哪个类提供的。

##### 示例

    struct B { int a; virtual int f(); virtual ~B() = default };
    struct D : B { int b; int f() override; };

    void use(B b)
    {
        D d;
        B b2 = d;   // 切片
        B b3 = b;
    }

    void use2()
    {
        D d;
        use(d);   // 切片
    }

两个 `d` 都发生了切片。

##### 例外

你可以安全地访问处于自身定义的作用域之内的具名多态对象，这并不会将之切片。

    void use3()
    {
        D d;
        d.f();   // OK
    }

##### 另见

[多态类应当抑制复制操作](#rc-copy-virtual)

##### 强制实施

对所有切片都进行标记。

### <a id="rh-dynamic_cast"></a>C.146: 当无法避免在类层次上进行导航时应使用 `dynamic_cast`

##### 理由

`dynamic_cast` 是运行时检查。

##### 示例

    struct B {   // 接口
        virtual void f();
        virtual void g();
        virtual ~B();
    };

    struct D : B {   // 更宽的接口
        void f() override;
        virtual void h();
    };

    void user(B* pb)
    {
        if (D* pd = dynamic_cast<D*>(pb)) {
            // ... 使用 D 的接口 ...
        }
        else {
            // ... 通过 B 的接口做事 ...
        }
    }

使用其他的强制转换可能会违反类型安全，导致程序中所访问的某个真实类型为 `X` 的变量，被当做具有某个无关的类型 `Z` 而进行访问：

    void user2(B* pb)   // 不好
    {
        D* pd = static_cast<D*>(pb);    // I know that pb really points to a D; trust me
        // ... 使用 D 的接口 ...
    }

    void user3(B* pb)    // 不安全
    {
        if (some_condition) {
            D* pd = static_cast<D*>(pb);   // I know that pb really points to a D; trust me
            // ... 使用 D 的接口 ...
        }
        else {
            // ... 通过 B 的接口做事 ...
        }
    }

    void f()
    {
        B b;
        user(&b);   // OK
        user2(&b);  // 糟糕的错误
        user3(&b);  // OK，*如果*程序员已经正确检查了 some_condition 的话
    }

##### 注解

和其他强制转换一样，`dynamic_cast` 被过度使用了。
[优先使用虚函数而不是强制转换](#rh-use-virtual)。
如果可行（无须运行时决议）并且相当便利的话，优先使用[静态多态](#???)而不是
继承层次的导航。

##### 注解

一些人会在 `typeid` 更合适的时候使用 `dynamic_cast`；
`dynamic_cast` 是一个通用的“是一个”操作，用以发现对象上的最佳接口，
而 `typeid` 是“报告对象的精确类型”操作，用以发现对象的真实类型。
后者本质上就是更简单的操作，因而应当更快一些。
后者（`typeid`）是可以在需要时进行手工模仿的（比如说，当工作在（因为某种原因）禁止使用 RTTI 的系统上），
而前者（`dynamic_cast`）要正确地实现则要困难得多。

考虑：

    struct B {
        const char* name {"B"};
        // 若 pb1->id() == pb2->id() 则 *pb1 与 *pb2 类型相同
        virtual const char* id() const { return name; }
        // ...
    };

    struct D : B {
        const char* name {"D"};
        const char* id() const override { return name; }
        // ...
    };

    void use()
    {
        B* pb1 = new B;
        B* pb2 = new D;

        cout << pb1->id(); // "B"
        cout << pb2->id(); // "D"


        if (pb2->id() == "D") {         // 貌似没问题
            D* pd = static_cast<D*>(pb2);
            // ...
        }
        // ...
    }

`pb2->id == "D"` 的结果实际上是由实现定义的。
我们加上这个是为了警告自造的 RTTI 中的危险之处。
这个代码可能可以多年正常工作，但只在不会对字符字面量进行唯一化的新机器，新编译器，或者新的连接器上失败。

当实现你自己的 RTTI 时，请当心这一点。

##### 例外

如果你所用的实现提供的 `dynamic_cast` 确实很慢的话，你可能只得使用一种替代方案了。
不过，所有无法静态决议的替代方案都涉及显式的强制转换（通常为 `static_cast`），而且易于出错。
你基本上都将会创建自己的特殊目的 `dynamic_cast`。
因此，首先应当确定你的 `dynamic_cast` 确实和你想的一样慢（其实有相当多的并不被支持的流言），
而且你使用的 `dynamic_cast` 确实是性能十分关键的。

我们的观点是，当前的实现中的 `dynamic_cast` 并非很慢。
比如说，在合适的条件下，是可以以[快速常量时间](http://www.stroustrup.com/fast_dynamic_casting.pdf)来实施 `dynamic_cast` 的。
但是，兼容性问题使得作出改变很难，虽然大家都同意对优化付出的努力是值得的。

在很罕见的情况下，如果你已经测量出 `dynamic_cast` 的开销确实有影响，你也有其他的方式来静态地保证向下转换的成功（比如说小心地应用 CRTP 时），而且其中并不涉及虚继承的话，可以考虑战术性地使用 `static_cast` 并带上显著的代码注释和免责声明，概述这个段落，而且由于类型系统无法验证其正确性而在维护中需要人工的关切。即便是这样，以我们的经验来说，这种“我知道我在干什么”的情况仍然是一种已知的 BUG 来源。

##### 例外

考虑：

    template<typename B>
    class Dx : B {
        // ...
    };

##### 强制实施

* 对所有用 `static_cast` 来进行向下转换进行标记，其中也包括实施 `static_cast` 的 C 风格的强制转换。
* 本条规则属于[类型安全性剖面配置](#pro-type-downcast)。

### <a id="rh-ref-cast"></a>C.147: 当查找所需类的失败被当做一种错误时，应当对引用类型使用 `dynamic_cast`

##### 理由

对引用进行的强制转换，所表达的意图是最终会获得一个有效对象，因此这种强制转换必须成功。如果无法成功的话，`dynamic_cast` 将会抛出异常。

##### 示例

    std::string f(Base& b)
    {
        return dynamic_cast<Derived&>(b).to_string();
    }

##### 强制实施

???

### <a id="rh-ptr-cast"></a>C.148: 当查找所需类的失败被当做一种有效的可能情况时，应当对指针类型使用 `dynamic_cast`

##### 理由

`dynamic_cast` 转换允许测试指针是否指向某个其类层次中包含给定类的多态对象。由于其找不到类时仅会返回空值，因而可以在运行时予以测试。这允许编写的代码可以基于其结果而选择不同的代码路径。

与此相对，[C.147](#rh-ref-cast) 中失败即是错误，而且不能用于条件执行。

##### 示例

下面的例子的 `Shape_owner` 的 `add` 函数获取构造的 `Shape` 对象的所有权。这些对象也根据其几何特性被存储到了不同视图中。
这个例子中的 `Shape` 并不继承于 `Geometric_attributes`，而是其各个子类继承。

    void add(Shape * const item)
    {
      // 总是获得其所有权
      owned_shapes.emplace_back(item);

      // 检查 Geometric_attribute 并将该形状加入到（零个/一个/某些/全部）视图中

      if (auto even = dynamic_cast<Even_sided*>(item))
      {
        view_of_evens.emplace_back(even);
      }

      if (auto trisym = dynamic_cast<Trilaterally_symmetrical*>(item))
      {
        view_of_trisyms.emplace_back(trisym);
      }
    }

##### 注解

找不到所需的类时 `dynamic_cast` 将返回空值，而解引用空指针将导致未定义的行为。
因此 `dynamic_cast` 的结果应当总是当做可能包含空值并进行测试。

##### 强制实施

* 【复杂】 除非在指针类型 `dynamic_cast` 的结果上进行了空值测试，否则就对该指针的解引用给出警告。

### <a id="rh-smart"></a>C.149: 用 `unique_ptr` 或 `shared_ptr` 来避免忘记对以 `new` 所创建的对象进行 `delete` 的情况

##### 理由

避免资源泄漏。

##### 示例

    void use(int i)
    {
        auto p = new int {7};           // 不好: 用 new 来初始化局部指针
        auto q = make_unique<int>(9);   // ok: 保证了为 9 所分配的内存会被回收
        if (0 < i) return;              // 可能会返回并泄漏
        delete p;                       // 太晚了
    }

##### 强制实施

* 标记用 `new` 的结果来对裸指针所进行的初始化。
* 标记对局部变量的 `delete`。

### <a id="rh-make_unique"></a>C.150: 用 `make_unique()` 来构建由 `unique_ptr` 所拥有的对象

参见 [R.23](#rr-make_unique)。

### <a id="rh-make_shared"></a>C.151: 用 `make_shared()` 来构建由 `shared_ptr` 所拥有的对象

参见 [R.22](#rr-make_shared)。

### <a id="rh-array"></a>C.152: 禁止把指向派生类对象的数组的指针赋值给指向基类的指针

##### 理由

对所得到的基类指针进行下标操作，将导致无效的对象访问并且可能造成内存损坏。

##### 示例

    struct B { int x; };
    struct D : B { int y; };

    void use(B*);

    D a[] = {{ "{{" }}1, 2}, {3, 4}, {5, 6}};
    B* p = a;     // 不好: a 退化为 &a[0]，并被转换为 B*
    p[1].x = 7;   // 覆盖了 a[0].y

    use(a);       // 不好: a 退化为 &a[0]，并被转换为 B*

##### 强制实施

* 对任何的数组退化和基类向派生类转换之间的组合进行标记。
* 应当将数组作为 `span` 而不是指针来进行传递，而且不能让数组的名字在放入 `span` 之前经手派生类向基类的转换。


### <a id="rh-use-virtual"></a>C.153: 优先采用虚函数而不是强制转换

##### 理由

虚函数调用安全，而强制转换易错。
虚函数调用达到全派生函数，而强制转换可能达到某个中间类
而得到错误的结果（尤其是当类层次在维护中被修改之后）。

##### 示例

    ???

##### 强制实施

参见 [C.146](#rh-dynamic_cast) 和 ???

## <a id="ss-overload"></a>C.over: 重载和运算符重载

可以对普通函数，函数模板，以及运算符进行重载。
不能重载函数对象。

重载规则概览：

* [C.160: 定义运算符应当主要用于模仿传统用法](#ro-conventional)
* [C.161: 对于对称的运算符应当采用非成员函数](#ro-symmetric)
* [C.162: 重载的操作之间应当大体上是等价的](#ro-equivalent)
* [C.163: 应当仅对大体上等价的操作进行重载](#ro-equivalent-2)
* [C.164: 避免隐式转换运算符](#ro-conversion)
* [C.165: 为定制点采用 `using`](#ro-custom)
* [C.166: 一元 `&` 的重载只能作为某个智能指针或引用系统的一部分而提供](#ro-address-of)
* [C.167: 应当为带有传统含义的操作提供运算符](#ro-overload)
* [C.168: 应当在操作数所在的命名空间中定义重载运算符](#ro-namespace)
* [C.170: 当想要重载 lambda 时，应当使用泛型 lambda](#ro-lambda)

### <a id="ro-conventional"></a>C.160: 定义运算符应当主要用于模仿传统用法

##### 理由

最小化意外情况。

##### 示例

    class X {
    public:
        // ...
        X& operator=(const X&); // 定义赋值的成员函数
        friend bool operator==(const X&, const X&); // == 需要访问其内部表示
                                                    // 执行 a = b 之后将有 a == b
        // ...
    };

这里维持了传统的语义：[副本之间相等](#ss-copy)。

##### 示例，不好

    X operator+(X a, X b) { return a.v - b.v; }   // 不好: 让 + 执行减法

##### 注解

非成员运算符应当要么是友元，要么定义于[其操作数所在的命名空间中](#ro-namespace)。
[二元运算符应当等价地对待其两个操作数](#ro-symmetric)。

##### 强制实施

也许是不可能的。

### <a id="ro-symmetric"></a>C.161: 对于对称的运算符应当采用非成员函数

##### 理由

如果使用的是成员函数的话，就需要两个才行。
如果为（比如）`==` 采用的不是非成员函数的话，`a == b` 和 `b == a` 就会存在微妙的差别。

##### 示例

    bool operator==(Point a, Point b) { return a.x == b.x && a.y == b.y; }

##### 强制实施

标记成员运算符函数。

### <a id="ro-equivalent"></a>C.162: 重载的操作之间应当大体上是等价的

##### 理由

让逻辑上互相等价的操作对不同的参数类型使用不同的名字会带来混乱，导致在函数名字中编码类型信息，并妨碍泛型编程。

##### 示例

考虑：

    void print(int a);
    void print(int a, int base);
    void print(const string&);

这三个函数都对其参数进行（适当的）打印。相对而言：

    void print_int(int a);
    void print_based(int a, int base);
    void print_string(const string&);

这三个函数也都对其参数进行（适当的）打印。在名字上附加仅仅增添了啰嗦程度，而且妨碍了泛型代码。

##### 强制实施

???

### <a id="ro-equivalent-2"></a>C.163: 应当仅对大体上等价的操作进行重载

##### 理由

让逻辑上不同的函数使用相同的名字会带来混乱，并导致在泛型编程时发生错误。

##### 示例

考虑：

    void open_gate(Gate& g);   // 把车库出口通道的障碍移除
    void fopen(const char* name, const char* mode);   // 打开文件

这两个操作本质上就是不同的（而且没有关联），因此让它们的名字相区别是正确的。相对而言：

    void open(Gate& g);   // 把车库出口通道的障碍移除
    void open(const char* name, const char* mode ="r");   // 打开文件

这两个操作仍旧本质不同（且没有关联），但它们的名字缩减成了（共同的）最小词，并带来了发生混乱的机会。
幸运的是，类型系统能够识别出许多这种错误。

##### 注解

对于一般性的和流行的名字，比如 `open`、`move`、`+` 和 `==` 等等，应当特别小心。

##### 强制实施

???

### <a id="ro-conversion"></a>C.164: 避免隐式转换运算符

##### 理由

隐式类型转换可以很基础（比如 `double` 向 `int`），但经常会导致意外（比如 `String` 到 C 风格的字符串）。

##### 注解

优先采用明确命名的转换，除非给出了一条很重要的需求。
我们所谓“很重要的需求”的意思是，它在其应用领域中是基本的（比如整数向复数的转换），
并且经常需要用到。不要仅仅为了微小的便利就（通过转换运算符或非 `explicit` 构造函数）
引入隐式的类型转换。

##### 示例

    struct S1 {
        string s;
        // ...
        operator char*() { return s.data(); } // 不好，可能会引起以外
    }

    struct S2 {
        string s;
        // ...
        explicit operator char*() { return s.data(); }
    };

    void f(S1 s1, S2 s2)
    {
        char* x1 = s1;     // 可以，但在许多情况下会引起意外
        char* x2 = s2;     // 错误，这通常是一件好事
        char* x3 = static_cast<char*>(s2); // 我们可以明确（在你的头脑里）
    }

可能在任意的难以发现的上下文里发生令人惊讶且可能具有破坏性的隐式转换，例如，

    S1 ff();

    char* g()
    {
        return ff();
    }

由`ff（）返回的字符串在返回它的指针之前被销毁。

##### 强制实施

标记所有的非显式转换运算符。

### <a id="ro-custom"></a>C.165: 为定制点采用 `using`

##### 理由

以便找到在一个独立的命名空间中所定义的函数对象或函数，它们对一个一般性函数进行了“定制”。

##### 示例

考虑 `swap`。这是一个通用的（标准库）函数，其定义可以在任何类型上工作。
不过，通常需要为特定的类型定义专门的 `swap()`。
例如说，通用的 `swap()` 会对互相交换的两个 `vector` 的元素进行复制，而一个好的专门实现则完全不会复制任何元素。

    namespace N {
        My_type X { /* ... */ };
        void swap(X&, X&);   // 针对 N::X 所优化的 swap
        // ...
    }

    void f1(N::X& a, N::X& b)
    {
        std::swap(a, b);   // 可能不是我们所要的: 调用 std::swap()
    }

`f1()` 中的 `std::swap()` 严格做了我们所要求的工作：它调用了命名空间 `std` 中的 `swap()`。
不幸的是，这可能并非我们想要的。
怎样让其考虑 `N::X` 呢？

    void f2(N::X& a, N::X& b)
    {
        swap(a, b);   // 调用了 N::swap
    }

但这也许并不是我们在泛型代码中所要的。
通常如果专门的函数存在的话我们就想用它，否则的话我们则需要通用的函数。
这是通过在函数的查找中包含通用函数而达成的：

    void f3(N::X& a, N::X& b)
    {
        using std::swap;  // 使得 std::swap 可用
        swap(a, b);        // 如果存在 N::swap 则调用之，否则为 std::swap
    }

##### 强制实施

不大可能，除非是如 `swap` 这样的已知的定制点。
问题在于无限定和带限定的名字查找都有其作用。

### <a id="ro-address-of"></a>C.166: 一元 `&` 的重载只能作为某个智能指针或引用系统的一部分而提供

##### 理由

`&` 运算符在 C++ 中很基本。
C++ 语义中的很多部分都假定了其默认的含义。

##### 示例

    class Ptr { // 一种智能指针
        Ptr(X* pp) : p(pp) { /* 检查 */ }
        X* operator->() { /* 检查 */ return p; }
        X operator[](int i);
        X operator*();
    private:
        T* p;
    };

    class X {
        Ptr operator&() { return Ptr{this}; }
        // ...
    };

##### 注解

如果你要“折腾”运算符 `&` 的话，请保证其定义和 `->`，`[]`，`*` 和 `.` 在结果类型上具有匹配的含义。
注意，运算符 `.` 现在是无法重载的，因此不可能做出一个完美的系统。
我们打算修正这一点： [Operator Dot (R2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4477.pdf)。
注意 `std::addressof()` 总会产生一个内建指针。

##### 强制实施

比较微妙。如果用户定义了 `&` 而并未同时为其结果类型定义 `->`，则进行警告。

### <a id="ro-overload"></a>C.167: 应当为带有传统含义的操作提供运算符

##### 理由

可读性。约定。可重用性。支持泛型代码。

##### 示例

    void cout_my_class(const My_class& c) // 含糊，并非传统约定，非泛型
    {
        std::cout << /* 此处为类成员 */;
    }

    std::ostream& operator<<(std::ostream& os, const my_class& c) // OK
    {
        return os << /* 此处为类成员 */;
    }

对于其自身而言，`cout_my_class` 也许是没问题的，但它无法用在（或组合到）依赖于 `<<` 的输出用法约定代码之中：

    My_class var { /* ... */ };
    // ...
    cout << "var = " << var << '\n';

##### 注解

大多数运算符都有很强烈和活跃的含义约定用法，比如

* 比较：`==`，`!=`，`<`，`<=`，`>`，`>=`，以及 `<=>`
* 算术操作：`+`，`-`，`*`，`/`，以及 `%`
* 访问操作：`.`，`->`，一元 `*`，以及 `[]`
* 赋值：`=`

请勿定义违反约定的用法，请勿为它们发明自己的名字。

##### 强制实施

比较棘手。需要洞悉其语义。

### <a id="ro-namespace"></a>C.168: 应当在操作数所在的命名空间中定义重载运算符

##### 理由

可读性。
通过 ADL 寻找运算符的能力。
避免不同命名空间中的定义之间不一致。

##### 示例

    struct S { };
    S operator+(S, S);   // OK: 和 S 在相同的命名空间，甚至紧跟着 S
    S s;

    S r = s + s;

##### 示例

    namespace N {
        struct S { };
        S operator+(S, S);   // OK: 和 S 在相同的命名空间，甚至紧跟着 S
    }

    N::S s;

    S r = s + s;  // 通过 ADL 找到了 N::operator+()

##### 示例，不好

    struct S { };
    S s;

    namespace N {
        bool operator!(S a) { return true; }
        bool not_s = !s;
    }

    namespace M {
        bool operator!(S a) { return false; }
        bool not_s = !s;
    }

这里的 `!s` 的含义在 `N` 和 `M` 中是不同的。
这可能是最易混淆的一点。
移除 `namespace M` 的定义之后，混乱就被一种犯错的机会所代替。

##### 注解

当为定义于不同命名空间的两个类型定义一个二元运算符时，无法遵循这条规则。
例如：

    Vec::Vector operator*(const Vec::Vector&, const Mat::Matrix&);

也许最好避免这种情形。

##### 参见

这是这条规则的一种特殊情况：[辅助函数应当定义于它们的类相同的命名空间之中](#rc-helper)。

##### 强制实施

* 对并非处于其操作数的命名空间中的运算符的定义进行标记。

### <a id="ro-lambda"></a>C.170: 当想要重载 lambda 时，应当使用泛型 lambda

##### 理由

无法通过定义两个带有相同名字的不同 lambda 来进行重载。

##### 示例

    void f(int);
    void f(double);
    auto f = [](char);   // 错误: 无法重载变量和函数

    auto g = [](int) { /* ... */ };
    auto g = [](double) { /* ... */ };   // 错误: 无法重载变量

    auto h = [](auto) { /* ... */ };   // OK

##### 强制实施

编译期会查明对 lambda 进行重载的企图。

## <a id="ss-union"></a>C.union: 联合体

`union` 是一种 `struct`，其所有成员都开始于相同的地址，因而它同时只能持有一个成员。
`union` 并不会跟踪其所存储的是哪个成员，因此必须由程序员来保证其正确；
这本质上就是易错的，但有一些弥补的办法。

由一个 `union` 加上一个用于指出其当前持有哪个成员的指示符构成的类型被称为*带标记联合体（tagged union）*，*区分性联合体（discriminated union）*，或者*变异体（variant）*。

联合体规则概览：

* [C.180: 采用 `union` 用以节省内存](#ru-union)
* [C.181: 避免“裸” `union`](#ru-naked)
* [C.182: 利用匿名 `union` 实现带标记联合体](#ru-anonymous)
* [C.183: 不要将 `union` 用于类型双关](#ru-pun)
* ???

### <a id="ru-union"></a>C.180: 采用 `union` 用以节省内存

##### 理由

`union` 可以让一块内存在不同的时间用于不同类型的数据。
因此，当我们有几个不可能同时使用的对象时，可以用它来节省内存。

##### 示例

    union Value {
        int x;
        double d;
    };

    Value v = { 123 };  // 现在 v 持有一个 int
    cout << v.x << '\n';    // 写下 123
    v.d = 987.654;  // 现在 v 持有一个 double
    cout << v.d << '\n';    // 写下 987.654

但请你留意这个警告：[避免“裸”`union`](#ru-naked)。

##### 示例

    // 短字符串优化

    constexpr size_t buffer_size = 16; // 比指针的大小稍大

    class Immutable_string {
    public:
        Immutable_string(const char* str) :
            size(strlen(str))
        {
            if (size < buffer_size)
                strcpy_s(string_buffer, buffer_size, str);
            else {
                string_ptr = new char[size + 1];
                strcpy_s(string_ptr, size + 1, str);
            }
        }

        ~Immutable_string()
        {
            if (size >= buffer_size)
                delete[] string_ptr;
        }

        const char* get_str() const
        {
            return (size < buffer_size) ? string_buffer : string_ptr;
        }

    private:
        // 当字符串足够短时，可以以其自己保存字符串
        // 而不是指向字符串的指针。
        union {
            char* string_ptr;
            char string_buffer[buffer_size];
        };

        const size_t size;
    };

##### 强制实施

???

### <a id="ru-naked"></a>C.181: 避免“裸” `union`

##### 理由

*裸联合体（naked union）*是没有相关的指示其持有哪个成员（如果有）的指示器的联合体，
程序员必须保持对它的跟踪。
裸联合体是类型错误的一种来源。

##### Example, bad

    union Value {
        int x;
        double d;
    };

    Value v;
    v.d = 987.654;  // v 持有一个 double

至此为止还都不错，但我们可能会轻易地误用这个 `union`：

    cout << v.x << '\n';    // 不好，未定义的行为：v 持有一个 double，但我们将之当做一个 int 来读取

注意这个类型错误的发生并没有任何明确的强制转换。
当我们测试程序时，其所打印的最后一个值是 `1683627180`，这是 `987.654` 的位模式的整数值。
我们这里遇到的是一个“不可见”的类型错误，它刚好给出的结果轻易可能被看作是无辜清白的。

对于“不可见”来说，下面的代码不会产生任何输出：

    v.x = 123;
    cout << v.d << '\n';    // 不好：未定义的行为

##### 替代方案

将它们和一个类型字段一起包装到类之中。

可以使用 C++17 的 `variant` 类型（在 `<variant>` 中可以找到）：

    variant<int, double> v;
    v = 123;        // v 持有一个 int
    int x = get<int>(v);
    v = 123.456;    // v 持有一个 double
    double w = get<double>(v);

##### 强制实施

???

### <a id="ru-anonymous"></a>C.182: 利用匿名 `union` 实现带标记联合体

##### 理由

设计良好的带标记联合体是类型安全的。
*匿名*联合体可以简化这种带有 (tag, union) 对的类的定义。

##### 示例

这个例子基本上是从 TC++PL4 pp216-218 借鉴而来的。
你可以查看原文以获得其介绍。

这段代码多少有点复杂。
处理一个带有用户定义的赋值和析构函数的类型是比较麻烦的。
在标准中包含 `variant` 的原因之一就是避免程序员不得不编写这样的代码。

    class Value { // 以一个联合体来表现两个候选表示
    private:
        enum class Tag { number, text };
        Tag type; // 区分

        union { // 表示（注意这是匿名联合体）
            int i;
            string s; // string 带有默认构造函数，复制操作，和析构函数
        };
    public:
        struct Bad_entry { }; // 用作异常

        ~Value();
        Value& operator=(const Value&);   // 因为 string 变体的缘故而需要这个
        Value(const Value&);
        // ...
        int number() const;
        string text() const;

        void set_number(int n);
        void set_text(const string&);
        // ...
    };

    int Value::number() const
    {
        if (type != Tag::number) throw Bad_entry{};
        return i;
    }

    string Value::text() const
    {
        if (type != Tag::text) throw Bad_entry{};
        return s;
    }

    void Value::set_number(int n)
    {
        if (type == Tag::text) {
            s.~string();      // 显式销毁 string
            type = Tag::number;
        }
        i = n;
    }

    void Value::set_text(const string& ss)
    {
        if (type == Tag::text)
            s = ss;
        else {
            new(&s) string{ss};   // 放置式 new: 显式构造 string
            type = Tag::text;
        }
    }

    Value& Value::operator=(const Value& e)   // 因为 string 变体的缘故而需要这个
    {
        if (type == Tag::text && e.type == Tag::text) {
            s = e.s;    // 常规的 string 赋值
            return *this;
        }

        if (type == Tag::text) s.~string(); // 显式销毁

        switch (e.type) {
        case Tag::number:
            i = e.i;
            break;
        case Tag::text:
            new(&s) string(e.s);   // 放置式 new: 显式构造
        }

        type = e.type;
        return *this;
    }

    Value::~Value()
    {
        if (type == Tag::text) s.~string(); // 显式销毁
    }

##### 强制实施

???

### <a id="ru-pun"></a>C.183: 不要将 `union` 用于类型双关

##### 理由

读取一个 `union` 曾写入的成员不同类型的成员是未定义的行为。
这种双关是不可见的，或者至少比具名强制转换更难于找出。
使用 `union` 的类型双关是一种错误来源。

##### 示例，不好

    union Pun {
        int x;
        unsigned char c[sizeof(int)];
    };

`Pun` 的想法是要能够查看 `int` 的字符表示。

    void bad(Pun& u)
    {
        u.x = 'x';
        cout << u.c[0] << '\n';     // 未定义行为
    }

如果你想要查看 `int` 的字节的话，应使用（具名）强制转换：

    void if_you_must_pun(int& x)
    {
        auto p = reinterpret_cast<std::byte*>(&x);
        cout << p[0] << '\n';     // OK；好多了
        // ...
    }

对从对象的声明类型向 `char*`，`unsigned char*`，或 `std::byte*` 进行 `reinterpret_cast` 的结果进行访问是有定义的行为。（不建议使用 `reinterpret_cast`，
但至少我们可以发觉发生了某种微妙的事情。）

##### 注解

不幸的是，`union` 经常被用于类型双关。
我们认为“它有时候能够按预期工作”并不是一种令人信服的理由。

C++17 引入了一个独立类型 `std::byte` 以支持在原始对象表示上进行的操作。在这些操作中应当使用这个类型代替 `unsigned char` 或 `char`。

##### 强制实施

???
