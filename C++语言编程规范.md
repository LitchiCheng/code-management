# C/C++语言编程规范

# <a name="c0-1"></a>背景
正如每个 C/C++ 程序员都知道的，C++ 有很多强大的特性，但这种强大不可避免的导致它走向复杂，使代码更容易产生 bug，难以阅读和维护。使代码易于管理的方法之一是加强代码一致性。让任何程序员都可以快速读懂你的代码这点非常重要。

风格，亦被称作可读性，也就是指导 C/C++ 编程的约定保持统一编程风格并遵守约定意味着可以很容易根据“模式匹配” 规则来推断各种标识符的含义。 创建通用，必需的习惯用语和模式可以使代码更容易理解。 规则并不是完美的，通过禁止在特定情况下有用的特性，可能会对代码实现造成影响。但是我们制定规则的目的“为了大多数程序员可以得到更多的好处”，这些规则在保证代码易于管理的同时，也能高效使用 C/C++ 的语言特性。

该文档结合coding代码管理平台，使用自动化代码扫描平台，对代码加以如下规则进行审查以达到提高团队代码效率及质量。

# <a name="c0-2"></a>约定

1. 规范中含检查规则条目必须遵守。
2. 规范中无检查规则条目应尽可能保持一致。
3. 对于不可避免的特例行为应给出合理解释及必要的讨论。

# <a name="c1"></a>1 头文件

## <a name="c1-1"></a>1-1 #define 保护

>所有头文件都应该使用 #define 来防止头文件被多重包含, 命名格式当是: `<PROJECT>_<PATH>_<FILE>_H_`.
为保证唯一性, 头文件的命名应该基于所在项目源代码树的全路径. 例如, 项目 foo 中的头文件 foo/src/bar/baz.h 可按如下方式保护:
```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
...
#endif // FOO_BAR_BAZ_H_a
```

### <a name="a1-1-1"></a>检查规则

#### HeaderGuard

**所属工具** CppLint

**规则概述** 检查头文件格式

**分类** 功能

**严重级别** 提示

**详细描述** Header guard has wrong style.

1. 检查两点: 

   ​	a. 头文件中是否含有#ifndef、#define

   ​	b. #ifndef和#define中的内容是否一样。 如果上述两条有一条满足，给出警告。

2. 检查#ifndef的格式是否正确，正确的命名格式是：PATH_FILE_H。如果格式有出入，给出警告。 

3. 检查#endif的格式是否正确，正确的格式是#endif // PATHFILE_H。如果不满足这种格式，给出警告。 

4. 对上一点3的补充检查：检查#endif后面的注释，如果没有/* */或者//...格式注释，给出警告。

#### EndifComment

**所属工具** CppLint

**规则概述** 检查#endif后面是否跟有注释

**分类** 功能

**严重级别** 错误

**详细描述** Uncommented text after #endif is non-standard. Use a comment.

1.检查#endif后面是否跟有注释，如果没有注释，给出警告。



## <a name="c1-2"></a>1-2 前置声明

> 尽可能地避免使用前置声明。使用 #include 包含需要的头文件即可。

所谓「前置声明」（forward declaration）是类、函数和模板的纯粹声明，没伴随着其定义.
前置声明隐藏了依赖关系，头文件改动时，用户的代码会跳过必要的重新编译过程。

前置声明可能会被库的后续更改所破坏。前置声明函数或模板有时会妨碍头文件开发者变动其 API. 例如扩大形参类型，加个自带默认参数的模板形参等等。

前置声明来自命名空间 std:: 的 symbol 时，其行为未定义。

尽量避免前置声明那些定义在其他项目中的实体.
函数：总是使用 #include.
类模板：优先使用 #include.

### <a name="a1-2-1"></a>检查规则
#### ForwardDeclaration

**所属工具** CppLint

**规则概述** 检查是否使用了类似“class AA: : tt;”这种格式的前向声明

**分类** 功能

**严重级别** 错误

**详细描述** Inner-style forward declarations are invalid. Remove this line.

1.在作用域内（如namespace作用域），检查是否使用了类似“class AA: : tt;”这种格式的前向声明。如果有这种前向声明，给出警告。



## <a name="c1-3"></a>1-3 #include 的路径及顺序
> 使用标准的头文件包含顺序可增强可读性, 避免隐藏依赖: 相关头文件, C 库, C++ 库, 其他库的 .h, 本项目内的 .h.

项目内头文件应按照项目源代码目录树结构排列, 避免使用 UNIX 特殊的快捷目录 . (当前目录) 或 .. (上级目录). 例如, google-awesome-project/src/base/logging.h 应该按如下方式包含:
```C++
#include "base/logging.h"
```
按字母顺序分别对每种类型的头文件进行二次排序是不错的主意。注意较老的代码可不符合这条规则，要在方便的时候改正它们。

举例来说, google-awesome-project/src/foo/internal/fooserver.cc 的包含次序如下:
```C++
#include "foo/public/fooserver.h" // 优先位置

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/public/bar.h"
```

**例外：**

有时，平台特定（system-specific）代码需要条件编译（conditional includes），这些代码可以放到其它 includes 之后。当然，您的平台特定代码也要够简练且独立，比如：

> ```
> #include "foo/public/fooserver.h"
> 
> #include "base/port.h"  // For LANG_CXX11.
> 
> #ifdef LANG_CXX11
> #include <initializer_list>
> #endif  // LANG_CXX11
> ```

### <a name="a1-3-1"></a> 检查规则

#### IncludeOrder

**所属工具** CppLint

**规则概述** 检查include文件的顺序

**分类** 功能

**严重级别** 错误

**详细描述** Include order should be: .h, c system, c++ system, other.

1.检查include文件的顺序：本文件相应头文件，C系统文件，C++系统文件，其他库文件。

#### IncludeAlpha

**所属工具** CppLint

**规则概述** 检查相同目录下头文件是否按字母序升序引用

**分类** 功能

**严重级别** 错误

**详细描述** Include not in alphabetical order.

1.检查相同目录下头文件是否按字母序升序引用，如果没有，给出警告。

#### IncludeSubdir

**所属工具** CppLint

**规则概述** 检查命名.h文件时是否包含目录

**分类** 功能

**严重级别** 错误

**详细描述** Include the directory when naming .h files.

命名.h文件时包含目录。

#### IncorrectInclude

**所属工具**  CppLint

**规则概述** 检查include的头文件不正确

**分类** 功能

**严重级别** 错误

**详细描述** Incorrect include header file.

1.检查每一个C++源文件是否都有一个对应的.h头文件，如果没有，给出警告。 2. 检查include的头文件是否加上路径，如果没有，给出警告。 3. 检查是否include了多次同一个头文件，如果是，给出警告。 4. 不要include其他包里面的.cc文件。如果include了，给出警告。



# <a name="c2"></a>2 作用域

## <a name="c2-1"></a> 2-1 命名空间

> 鼓励在 `.cpp` 文件内使用匿名命名空间或 `static` 声明. 使用具名的命名空间时, 其名称可基于项目名或相对路径. 禁止使用 using 指示（using-directive）。禁止使用内联命名空间（inline namespace）。

> 命名空间将全局作用域细分为独立的, 具名的作用域, 可有效防止全局作用域的命名冲突.

**缺点:**

> 命名空间具有迷惑性, 因为它们使得区分两个相同命名所指代的定义更加困难。
>
> 内联命名空间很容易令人迷惑，毕竟其内部的成员不再受其声明所在命名空间的限制。内联命名空间只在大型版本控制里有用。
>
> 有时候不得不多次引用某个定义在许多嵌套命名空间里的实体，使用完整的命名空间会导致代码的冗长。
>
> 在头文件中使用匿名空间导致违背 C++ 的唯一定义原则 (One Definition Rule (ODR)).

**结论:**

> 根据下文将要提到的策略合理使用命名空间.
>
> - 像之前的几个例子中一样，在命名空间的最后注释出命名空间的名字。
>
> - 用命名空间把文件包含, 以及类的前置声明以外的整个源文件封装起来, 以区别于其它命名空间:
>
>   > ```
>   > // .h 文件
>   > namespace mynamespace {
>   > 
>   > // 所有声明都置于命名空间中
>   > // 注意不要使用缩进
>   > class MyClass {
>   >     public:
>   >     ...
>   >     void Foo();
>   > };
>   > 
>   > } // namespace mynamespace
>   > ```
>   >
>   > ```
>   > // .cpp 文件
>   > namespace mynamespace {
>   > 
>   > // 函数定义都置于命名空间中
>   > void MyClass::Foo() {
>   >     ...
>   > }
>   > 
>   > } // namespace mynamespace
>   > ```
>   >
>   > 更复杂的 `.cc` 文件包含更多, 更复杂的细节, 比如 gflags 或 using 声明。
>   >
>   > ```
>   > #include "a.h"
>   > 
>   > DEFINE_FLAG(bool, someflag, false, "dummy flag");
>   > 
>   > namespace a {
>   > 
>   > ...code for a...                // 左对齐
>   > 
>   > } // namespace a
>   > ```
>
> - 不要在命名空间 `std` 内声明任何东西, 包括标准库的类前置声明. 在 `std` 命名空间声明实体是未定义的行为, 会导致如不可移植. 声明标准库下的实体, 需要包含对应的头文件.
>
> - 不应该使用 *using 指示* 引入整个命名空间的标识符号。
>
>   > ```
>   > // 禁止 —— 污染命名空间
>   > using namespace foo;
>   > ```
>
> - 不要在头文件中使用 *命名空间别名* 除非显式标记内部命名空间使用。因为任何在头文件中引入的命名空间都会成为公开API的一部分。
>
>   > ```
>   > // 在 .cpp 中使用别名缩短常用的命名空间
>   > namespace baz = ::foo::bar::baz;
>   > ```
>   >
>   > ```
>   > // 在 .h 中使用别名缩短常用的命名空间
>   > namespace librarian {
>   > namespace impl {  // 仅限内部使用
>   > namespace sidetable = ::pipeline_diagnostics::sidetable;
>   > }  // namespace impl
>   > 
>   > inline void my_inline_function() {
>   >   // 限制在一个函数中的命名空间别名
>   >   namespace baz = ::foo::bar::baz;
>   >   ...
>   > }
>   > }  // namespace librarian
>   > ```
>
> - 禁止用内联命名空间

不要增加额外的缩进层次, 例如:

```
namespace {

void foo() {  // 正确. 命名空间内没有额外的缩进.
  ...
}

}  // namespace
```

不要在命名空间内缩进:

```
namespace {

  // 错, 缩进多余了.
  void foo() {
    ...
  }

}  // namespace
```

声明嵌套命名空间时, 每个命名空间都独立成行.

```
namespace foo {
namespace bar {
```

### <a name="a2-1-1"></a> 检查规则

#### Namespace

**所属工具** CppLint

**规则概述** 检查使用using std;

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper namespace style

**检查使用** using std; 建议使用具体命名空间，如：using std: string

#### IndentationNamespace

**所属工具** CppLint

**规则概述** 检查是否在namespace中存在缩进

**分类** 功能

**严重级别** 致命

**详细描述** Do not indent within a namespace.

1. 检查是否在namespace中存在缩进。如果有，给出警告。

#### Namespaces

**所属工具** CppLint

**规则概述** 检查命名空间的不当使用

**分类** 功能

**严重级别** 错误

**详细描述** Failed to find complete declaration of namespace.Do not use namespace  using-directives.Use using-declarations instead.Do not use unnamed  namespaces in header files.

1. 检查命名空间定义是否有结束标记，如果没有，给出警告。
2. 检查是否使用了using编译指令，如果使用了，给出警告，提示用户使用using声明指令。
3. 检查在.h文件中是否使用了不具名的命名控件，如果使用了，给出警告。



# <a name="c3"></a> 3 类

## <a name="c3-1"></a> 3-1 声明

>  将相似的声明放在一起, 将 `public` 部分放在最前。定义一般应以 `public:` 开始, 后跟 `protected:`, 最后是 `private:`. 省略空部分。
>
> 在各个部分中, 建议将类似的声明放在一起, 并且建议以如下的顺序: 类型 (包括 `typedef`, `using` 和嵌套的结构体与类), 常量, 工厂函数, 构造函数, 赋值运算符, 析构函数, 其它函数, 数据成员.
>
> 不要将大段的函数定义内联在类定义中. 通常，只有那些普通的, 或性能关键且短小的函数可以内联在类定义中.

### <a name="a3-1-1"></a> 检查规则

#### IncompleteClassDeclaration

**所属工具** CppLint 

**规则概述** 检查类声明是否完整

**分类** 功能

**严重级别** 错误

**详细描述** Failed to find complete declaration of class.

1.检查类声明是否完整，即类声明的结束部位是否含有“}”。如果没有，给出类声明不完整警告。



## <a name="c3-2"></a> 3-2 继承

> 使用组合 (YuleFox 注: 这一点也是 GoF 在 <<Design Patterns>> 里反复强调的) 常常比使用继承更合理. 如果使用继承的话, 定义为 `public` 继承.当子类继承基类时, 子类包含了父基类所有数据及操作的定义. C++ 实践中, 继承主要用于两种场合: 实现继承, 子类继承父类的实现代码; 子类仅继承父类的方法名称.
>
> 所有继承必须是 `public` 的. 如果你想使用私有继承, 你应该替换成把基类的实例作为成员对象的方式.
>
> 不要过度使用实现继承. 组合常常更合适一些. 尽量做到只在 "是一个" ("is-a", YuleFox 注: 其他 "has-a" 情况下请使用组合) 的情况下使用继承: 如果 `Bar` 的确 "是一种" `Foo`, `Bar` 才能继承 `Foo`.
>
> 必要的话, 析构函数声明为 `virtual`. 如果你的类有虚函数, 则析构函数也应该为虚函数.
>
> 对于可能被子类访问的成员函数, 不要过度使用 `protected` 关键字. 
>
> 对于重载的虚函数或虚析构函数, 使用 `override`, 或 (较不常用的) `final` 关键字显式地进行标记. 较早 (早于 C++11) 的代码可能会使用 `virtual` 关键字作为不得已的选项. 因此, 在声明重载时, 请使用 `override`, `final` 或 `virtual` 的其中之一进行标记. 标记为 `override` 或 `final` 的析构函数如果不是对基类虚函数的重载的话, 编译会报错, 这有助于捕获常见的错误. 这些标记起到了文档的作用, 因为如果省略这些关键字, 代码阅读者不得不检查所有父类, 以判断该函数是否是虚函数.

### <a name="a3-2-1"></a> 检查规则

#### Inheritance

**所属工具** CppLint

**规则概述** 检查到final或overrider修饰的函数是否有virtual修饰

**分类** 代码风格

**严重级别** 警告

**详细描述** virtual or override is redundant.

1. 在函数的后面如果有关键字final或者overrider，表示该函数不可以为虚函数，如果检查到final或overrider修饰的函数有virtual修饰，会给出警告。

## <a name="c3-3"></a> 3-3 隐式类型转换	

> 不要定义隐式类型转换. 对于转换运算符和单参数构造函数, 请使用 `explicit` 关键字.
>
> 隐式类型转换允许一个某种类型 (称作 *源类型*) 的对象被用于需要另一种类型 (称作 *目的类型*) 的位置, 例如, 将一个 `int` 类型的参数传递给需要 `double` 类型的函数.
>
> 除了语言所定义的隐式类型转换, 用户还可以通过在类定义中添加合适的成员定义自己需要的转换. 在源类型中定义隐式类型转换, 可以通过目的类型名的类型转换运算符实现 (例如 `operator bool()`). 在目的类型中定义隐式类型转换, 则通过以源类型作为其唯一参数 (或唯一无默认值的参数) 的构造函数实现.

`explicit` 关键字可以用于构造函数或 (在 C++11 引入) 类型转换运算符, 以保证只有当目的类型在调用点被显式写明时才能进行类型转换, 例如使用 `cast`. 这不仅作用于隐式类型转换, 还能作用于 C++11 的列表初始化语法:

```
class Foo {
  explicit Foo(int x, double y);
  ...
};

void Func(Foo f);
```

此时下面的代码是不允许的:

```
Func({42, 3.14});  // Error
```

这一代码从技术上说并非隐式类型转换, 但是语言标准认为这是 `explicit` 应当限制的行为.

- 隐式类型转换会隐藏类型不匹配的错误. 有时, 目的类型并不符合用户的期望, 甚至用户根本没有意识到发生了类型转换.
- 隐式类型转换会让代码难以阅读, 尤其是在有函数重载的时候, 因为这时很难判断到底是哪个函数被调用.
- 单参数构造函数有可能会被无意地用作隐式类型转换.
- 如果单参数构造函数没有加上 `explicit` 关键字, 读者无法判断这一函数究竟是要作为隐式类型转换, 还是作者忘了加上 `explicit` 标记.
- 并没有明确的方法用来判断哪个类应该提供类型转换, 这会使得代码变得含糊不清.
- 如果目的类型是隐式指定的, 那么列表初始化会出现和隐式类型转换一样的问题, 尤其是在列表中只有一个元素的时候.

在类型定义中, 类型转换运算符和单参数构造函数都应当用 `explicit` 进行标记. 一个例外是, 拷贝和移动构造函数不应当被标记为 `explicit`, 因为它们并不执行类型转换. 对于设计目的就是用于对其他类型进行透明包装的类来说, 隐式类型转换有时是必要且合适的. 这时应当联系项目组长并说明特殊情况.

不能以一个参数进行调用的构造函数不应当加上 `explicit`. 接受一个 `std::initializer_list` 作为参数的构造函数也应当省略 `explicit`, 以便支持拷贝初始化 (例如 `MyType m = {1, 2};`).

### <a name="a3-3-1"></a> 检查规则

#### ImproperExplicit

**所属工具** CppLint

**规则概述** 检查explicit关键字的不当使用

**分类** 功能

**严重级别** 致命

**详细描述** Improper explicit.

1. 检查只有一个参数的构造函数是否使用了explicit关键字。如果没有使用，给出警告。
2. 检查没有参数的构造函数是否使用了explicit关键字。如果使用了，给出警告。 
3. 检查有多个参数的构造函数是否使用了explicit关键字。如果使用了，给出警告。



## <a name="c3-3"></a> 3-4 运算符重载

> 除少数特定环境外, 不要重载运算符. 也不要创建用户定义字面量.
>
> - 要提供正确, 一致, 不出现异常行为的操作符运算需要花费不少精力, 而且如果达不到这些要求的话, 会导致令人迷惑的 Bug.
> - 过度使用运算符会带来难以理解的代码, 尤其是在重载的操作符的语义与通常的约定不符合时.
> - 函数重载有多少弊端, 运算符重载就至少有多少.
> - 运算符重载会混淆视听, 让你误以为一些耗时的操作和操作内建类型一样轻巧.
> - 对重载运算符的调用点的查找需要的可就不仅仅是像 grep 那样的程序了, 这时需要能够理解 C++ 语法的搜索工具.
> - 如果重载运算符的参数写错, 此时得到的可能是一个完全不同的重载而非编译错误. 例如: `foo < bar` 执行的是一个行为, 而 `&foo < &bar` 执行的就是完全不同的另一个行为了.
> - 重载某些运算符本身就是有害的. 例如, 重载一元运算符 `&` 会导致同样的代码有完全不同的含义, 这取决于重载的声明对某段代码而言是否是可见的. 重载诸如 `&&`, `||` 和 `,` 会导致运算顺序和内建运算的顺序不一致.
> - 运算符从通常定义在类的外部, 所以对于同一运算, 可能出现不同的文件引入了不同的定义的风险. 如果两种定义都链接到同一二进制文件, 就会导致未定义的行为, 有可能表现为难以发现的运行时错误.
> - 用户定义字面量所创建的语义形式对于某些有经验的 C++ 程序员来说都是很陌生的.

只有对用户自己定义的类型重载运算符. 更准确地说, 将它们和它们所操作的类型定义在同一个头文件中, `.cc`  中和命名空间中. 这样做无论类型在哪里都能够使用定义的运算符, 并且最大程度上避免了多重定义的风险. 如果可能的话, 请避免将运算符定义为模板, 因为此时它们必须对任何模板参数都能够作用. 如果你定义了一个运算符, 请将其相关且有意义的运算符都进行定义, 并且保证这些定义的语义是一致的. 例如, 如果你重载了 `<`, 那么请将所有的比较运算符都进行重载, 并且保证对于同一组参数, `<` 和 `>` 不会同时返回 `true`.

建议不要将不进行修改的二元运算符定义为成员函数. 如果一个二元运算符被定义为类成员, 这时隐式转换会作用域右侧的参数却不会作用于左侧. 这时会出现 `a < b` 能够通过编译而 `b < a` 不能的情况, 这是很让人迷惑的.

不要为了避免重载操作符而走极端. 比如说, 应当定义 `==`, `=`, 和 `<<` 而不是 `Equals()`, `CopyFrom()` 和 `PrintTo()`. 反过来说, 不要只是为了满足函数库需要而去定义运算符重载. 比如说, 如果你的类型没有自然顺序, 而你要将它们存入 `std::set` 中, 最好还是定义一个自定义的比较运算符而不是重载 `<`.

不要重载 `&&`, `||`, `,` 或一元运算符 `&`. 不要重载 `operator""`, 也就是说, 不要引入用户定义字面量.

### <a name="a3-4-1"></a> 检查规则

#### DangerousOperator

**所属工具** CppLint

**规则概述** 操作符&比较危险，建议不要用

**分类** 功能

**严重级别** 致命	

**详细描述** Unary operator& is dangerous. Do not use it.

1. 检查是否重载了操作符&。如果重载了，鉴于该操作符的危险性，给出警告。

   

# <a name="c4"></a> 4 函数

## <a name="c4-1"></a> 4-1 编写简短函数

> 我们倾向于编写简短, 凝练的函数.

我们承认长函数有时是合理的, 因此并不硬性限制函数的长度. 如果函数超过 40 行, 可以思索一下能不能在不影响程序结构的前提下对其进行分割.

即使一个长函数现在工作的非常好, 一旦有人对其修改, 有可能出现新的问题, 甚至导致难以发现的 bug. 使函数尽量简短, 以便于他人阅读和修改代码.

在处理代码时, 你可能会发现复杂的长函数. 不要害怕修改现有代码: 如果证实这些代码使用 / 调试起来很困难, 或者你只需要使用其中的一小段代码, 考虑将其分割为更加简短并易于管理的若干函数.

### <a name="a4-1-1"></a> 检查规则

#### FunctionSize

**所属工具** CppLint

**规则概述** 检查函数的大小

**分类** 代码风格

**严重级别** 警告

**详细描述** Too large size of function. Or Lint failed to find start of function body.

1. 建议编写小巧、功能集中的函数，大于50行开始给出警告，具体警告等级和代码行数关系为：50 => 0, 100 => 1, 200 => 2, 400 => 3, 800 => 4, 1600 => 5；测试代码量可以加倍。
2. 检查到函数定义，但是没有找到函数体的时候，给出警告。



## <a name="c4-2"></a> 4-2 引用参数

**总述**

所有按引用传递的参数必须加上 `const`.

**定义**

在 C 语言中, 如果函数需要修改变量的值, 参数必须为指针, 如 `int foo(int *pval)`. 在 C++ 中, 函数还可以声明为引用参数: `int foo(int &val)`.

**优点**

定义引用参数可以防止出现 `(*pval)++` 这样丑陋的代码. 引用参数对于拷贝构造函数这样的应用也是必需的. 同时也更明确地不接受空指针.

**缺点**

容易引起误解, 因为引用在语法上是值变量却拥有指针的语义.

**结论**

函数参数列表中, 所有引用参数都必须是 `const`:

```
void Foo(const string &in, string *out);
```

事实上这在 Google Code 是一个硬性约定: 输入参数是值参或 `const` 引用, 输出参数为指针. 输入参数可以是 `const` 指针, 但决不能是非 `const` 的引用参数, 除非特殊要求, 比如 `swap()`.

有时候, 在输入形参中用 `const T*` 指针比 `const T&` 更明智. 比如:

- 可能会传递空指针.
- 函数要把指针或对地址的引用赋值给输入形参.

总而言之, 大多时候输入形参往往是 `const T&`. 若用 `const T*` 则说明输入另有处理. 所以若要使用 `const T*`, 则应给出相应的理由, 否则会使得读者感到迷惑.

### <a name="a4-2-1"></a> 检查规则

#### RuntimeReferences

**所属工具** CppLint

**规则概述** 检查是否使用了非const型的指针

**分类** 功能

**严重级别** 致命

**详细描述** Is this a non-const reference? If so, make const or use a pointer: ReplaceAll(' *<', '<', parameter)

1. 在函数参数中，查找是否使用了非const型的指针。如果发现了这种类型的指针，则给出警告，并建议使用const型或者指针代替之。



# <a name="c5"></a> 5 其他C++特性

## <a name="c5-1"></a> 5-1 右值引用

> 右值引用是一种只能绑定到临时对象的引用的一种, 其语法与传统的引用语法相似. 例如, `void f(string&& s)`; 声明了一个其参数是一个字符串的右值引用的函数.
>
> 只在定义移动构造函数与移动赋值操作时使用右值引用, 不要使用 `std::forward` 功能函数. 你可能会使用 `std::move` 来表示将值从一个对象移动而不是复制到另一个对象.

### <a name="a5-1-1"></a> 检查规则

#### UnapprovedCpp11Feature

**所属工具** CppLint

**规则概述** 检查未批准的C++特性的使用

**分类** 功能

**严重级别** 错误

**详细描述** Unapproved C++11 header, class or function.

1. 检查对C++11标准提到的右值引用的使用，如果检查到使用右值引用，给出警告（未批准的C++特性）。 

2. 检查是否使用了默认的lambda捕获，如果使用了，给出警告（未批准的C++特性）。 



## <a name="c5-2"></a> 5-2 Lambda 表达式

>  适当使用 lambda 表达式。别用默认 lambda 捕获，所有捕获都要显式写出来。

> Lambda 表达式是创建匿名函数对象的一种简易途径，常用于把函数当参数传，例如：
>
> ```
> std::sort(v.begin(), v.end(), [](int x, int y) {
>     return Weight(x) < Weight(y);
> });
> ```
>
> C++11 首次提出 Lambdas, 还提供了一系列处理函数对象的工具，比如多态包装器（polymorphic wrapper） `std::function`.

> - Lambdas 的变量捕获略旁门左道，可能会造成悬空指针。
> - Lambdas 可能会失控；层层嵌套的匿名函数难以阅读。

> - 按 format 小用 lambda 表达式怡情。
> - 禁用默认捕获，捕获都要显式写出来。打比方，比起 `[=](int x) {return x + n;}`, 您该写成 `[n](int x) {return x + n;}` 才对，这样读者也好一眼看出 `n` 是被捕获的值。
> - 匿名函数始终要简短，如果函数体超过了五行，那么还不如起名（acgtyrant 注：即把 lambda 表达式赋值给对象），或改用函数。
> - 如果可读性更好，就显式写出 lambd 的尾置返回类型，就像auto.

### <a name="a5-2-1"></a> 检查规则

#### UnapprovedCpp11Feature

**所属工具** CppLint

**规则概述** 检查未批准的C++特性的使用

**分类** 功能

**严重级别** 错误

**详细描述** Unapproved C++11 header, class or function.

1. 检查是否使用了默认的lambda捕获，如果使用了，给出警告（未批准的C++特性）。 

   

## <a name="c5-3"></a> 5-3 整形

> C++ 内建整型中, 仅使用 `int`. 如果程序中需要不同大小的变量, 可以使用 `<stdint.h>` 中长度精确的整型, 如 `int16_t`.如果您的变量可能不小于 2^31 (2GiB), 就用 64 位变量比如 `int64_t`. 此外要留意，哪怕您的值并不会超出 int 所能够表示的范围，在计算过程中也可能会溢出。所以拿不准时，干脆用更大的类型。

缺点:

> C++ 中整型大小因编译器和体系结构的不同而不同.

结论:

> `<stdint.h>` 定义了 `int16_t`, `uint32_t`, `int64_t` 等整型, 在需要确保整型大小时可以使用它们代替 `short`, `unsigned long long` 等. 在 C 整型中, 只使用 `int`. 在合适的情况下, 推荐使用标准类型如 `size_t` 和 `ptrdiff_t`.
>
> 如果已知整数不会太大, 我们常常会使用 `int`, 如循环计数. 在类似的情况下使用原生类型 `int`. 你可以认为 `int` 至少为 32 位, 但不要认为它会多于 `32` 位. 如果需要 64 位整型, 用 `int64_t` 或 `uint64_t`.
>
> 对于大整数, 使用 `int64_t`.
>
> 不要使用 `uint32_t` 等无符号整型, 除非你是在表示一个位组而不是一个数值, 或是你需要定义二进制补码溢出. 尤其是不要为了指出数值永不会为负, 而使用无符号类型. 相反, 你应该使用断言来保护数据.
>
> 如果您的代码涉及容器返回的大小（size），确保其类型足以应付容器各种可能的用法。拿不准时，类型越大越好。
>
> 小心整型类型转换和整型提升（acgtyrant 注：integer promotions, 比如 `int` 与 `unsigned int` 运算时，前者被提升为 `unsigned int` 而有可能溢出），总有意想不到的后果。

关于无符号整数:

> 有些人, 包括一些教科书作者, 推荐使用无符号类型表示非负数. 这种做法试图达到自我文档化. 但是, 在 C 语言中, 这一优点被由其导致的 bug 所淹没. 看看下面的例子:
>
> > ```
> > for (unsigned int i = foo.Length()-1; i >= 0; --i) ...
> > ```
>
> 上述循环永远不会退出! 有时 gcc 会发现该 bug 并报警, 但大部分情况下都不会. 类似的 bug 还会出现在比较有符合变量和无符号变量时. 主要是 C 的类型提升机制会致使无符号类型的行为出乎你的意料.
>
> 因此, 使用断言来指出变量为非负数, 而不是使用无符号型!

### <a name="a5-3-1"></a> 检查规则

#### ImproperInt

**所属工具** CppLint 

**规则概述** 检查int的不当使用

**分类** 功能 

**严重级别** 致命 

**详细描述** Use 'unsigned short' for ports, not 'short'. Use int16/int64/etc, rather than the C type.

1. 检查port前面是否使用了unsigned short修饰。如果不是，给出警告。
2. 检查是否使用了short、long、long long。如果发现使用了这些，给出警告，并建议使用int16、int64等代替之。



## <a name="c5-4"></a> 5-4 printf打印

> 代码应该对 64 位和 32 位系统友好. 处理打印, 比较, 结构体对齐时应切记:

- 对于某些类型, `printf()` 的指示符在 32 位和 64 位系统上可移植性不是很好. C99 标准定义了一些可移植的格式化指示符. 不幸的是, MSVC 7.1 并非全部支持, 而且标准中也有所遗漏, 所以有时我们不得不自己定义一个丑陋的版本 (头文件 `inttypes.h` 仿标准风格):

  > ```
  > // printf macros for size_t, in the style of inttypes.h
  > #ifdef _LP64
  > #define __PRIS_PREFIX "z"
  > #else
  > #define __PRIS_PREFIX
  > #endif
  > 
  > // Use these macros after a % in a printf format string
  > // to get correct 32/64 bit behavior, like this:
  > // size_t size = records.size();
  > // printf("%"PRIuS"\n", size);
  > #define PRIdS __PRIS_PREFIX "d"
  > #define PRIxS __PRIS_PREFIX "x"
  > #define PRIuS __PRIS_PREFIX "u"
  > #define PRIXS __PRIS_PREFIX "X"
  > #define PRIoS __PRIS_PREFIX "o"
  > ```
  >
  > | 类型                      | 不要使用          | 使用                   | 备注           |
  > | ------------------------- | ----------------- | ---------------------- | -------------- |
  > | `void *` (或其他指针类型) | `%lx`             | `%p`                   |                |
  > | `int64_t`                 | `%qd, %lld`       | `%"PRId64"`            |                |
  > | `uint64_t`                | `%qu, %llu, %llx` | `%"PRIu64", %"PRIx64"` |                |
  > | `size_t`                  | `%u`              | `%"PRIuS", %"PRIxS"`   | C99 规定 `%zu` |
  > | `ptrdiff_t`               | `%d`              | `%"PRIdS"`             | C99 规定 `%zd` |
  >
  > 注意 `PRI*` 宏会被编译器扩展为独立字符串. 因此如果使用非常量的格式化字符串, 需要将宏的值而不是宏名插入格式中. 使用 `PRI*` 宏同样可以在 `%` 后包含长度指示符. 例如, `printf("x = %30"PRIuS"\n", x)` 在 32 位 Linux 上将被展开为 `printf("x = %30" "u" "\n", x)`, 编译器当成 `printf("x = %30u\n", x)` 处理 (Yang.Y 注: 这在 MSVC 6.0 上行不通, VC 6 编译器不会自动把引号间隔的多个字符串连接一个长字符串).

### <a name="a5-4-1"></a> 检查规则

#### RuntimePrintfFormat

**所属工具** CppLint

**规则概述** 检查printf打印语句的格式

**分类** 功能

**严重级别** 致命

**详细描述** %q in format strings is deprecated. Use %ll instead. %N$ formats are unconventional. Try rewriting to avoid them.

1. 检查在使用printf打印语句时，是否使用了“%qd”。如果使用了，给出警告，建议使用“%lld”。
2. 检查在使用printf打印语句时，是否使用了“%1$d”这种格式。如果使用了，给出警告。



# <a name="c6"></a> 6 命名约定

最重要的一致性规则是命名管理. 命名的风格能让我们在不需要去查找类型声明的条件下快速地了解某个名字代表的含义: 类型, 变量, 函数, 常量, 宏, 等等, 甚至. 我们大脑中的模式匹配引擎非常依赖这些命名规则.

命名规则具有一定随意性, 但相比按个人喜好命名, 一致性更重要, 所以无论你认为它们是否重要, 规则总归是规则.

## <a name="c6-1"></a> 6-1 通用命名规则

**总述**

函数命名, 变量命名, 文件命名要有描述性; 少用缩写.

**说明**

尽可能使用描述性的命名, 别心疼空间, 毕竟相比之下让代码易于新读者理解更重要. 不要用只有项目开发者能理解的缩写, 也不要通过砍掉几个字母来缩写单词.

```
int price_count_reader;    // 无缩写
int num_errors;            // "num" 是一个常见的写法
int num_dns_connections;   // 人人都知道 "DNS" 是什么
int n;                     // 毫无意义.
int nerr;                  // 含糊不清的缩写.
int n_comp_conns;          // 含糊不清的缩写.
int wgc_connections;       // 只有贵团队知道是什么意思.
int pc_reader;             // "pc" 有太多可能的解释了.
int cstmr_id;              // 删减了若干字母.
```

注意, 一些特定的广为人知的缩写是允许的, 例如用 `i` 表示迭代变量和用 `T` 表示模板参数.

模板参数的命名应当遵循对应的分类: 类型模板参数应当遵循类型命名的规则, 而非类型模板应当遵循变量命名的规则.



## <a name="c6-2"></a> 6-2 文件命名

**总述**

文件名要全部小写, 可以包含下划线 (`_`) 或连字符 (`-`), 依照项目的约定. 如果没有约定, 那么 "`_`" 更好.

**说明**

可接受的文件命名示例:

- `my_useful_class.cc`
- `my-useful-class.cc`
- `myusefulclass.cc`
- `myusefulclass_test.cc` // `_unittest` 和 `_regtest` 已弃用.

C++ 文件要以 `.cc` 结尾, 头文件以 `.h` 结尾. 专门插入文本的文件则以 `.inc` 结尾。

不要使用已经存在于 `/usr/include` 下的文件名 (Yang.Y 注: 即编译器搜索系统头文件的路径), 如 `db.h`.

通常应尽量让文件名更加明确. `http_server_logs.h` 就比 `logs.h` 要好. 定义类时文件名一般成对出现, 如 `foo_bar.h` 和 `foo_bar.cc`, 对应于类 `FooBar`.

内联函数必须放在 `.h` 文件中. 如果内联函数比较短, 就直接放在 `.h` 中.



## <a name="c6-3"></a> 6-3 类型命名

**总述**

类型名称的每个单词首字母均大写, 不包含下划线: `MyExcitingClass`, `MyExcitingEnum`.

**说明**

所有类型命名 —— 类, 结构体, 类型定义 (`typedef`), 枚举, 类型模板参数 —— 均使用相同约定, 即以大写字母开始, 每个单词首字母均大写, 不包含下划线. 例如:

```
// 类和结构体
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// 类型定义
typedef hash_map<UrlTableProperties *, string> PropertiesMap;

// using 别名
using PropertiesMap = hash_map<UrlTableProperties *, string>;

// 枚举
enum UrlTableErrors { ...
```



## <a name="c6-4"></a> 6-4 变量命名

**总述**

变量 (包括函数参数) 和数据成员名一律小写, 单词之间用下划线连接. 类的成员变量以下划线结尾, 但结构体的就不用, 如: `a_local_variable`, `a_struct_data_member`, `a_class_data_member_`.

### 普通变量命名

举例:

```
string table_name;  // 好 - 用下划线.
string tablename;   // 好 - 全小写.

string tableName;  // 差 - 混合大小写
```



### 类数据成员

不管是静态的还是非静态的, 类数据成员都可以和普通变量一样, 但要接下划线.

```
class TableInfo {
  ...
 private:
  string table_name_;  // 好 - 后加下划线.
  string tablename_;   // 好.
  static Pool<TableInfo>* pool_;  // 好.
};
```



### 结构体变量

不管是静态的还是非静态的, 结构体数据成员都可以和普通变量一样, 不用像类那样接下划线:

```
struct UrlTableProperties {
  string name;
  int num_entries;
  static Pool<UrlTableProperties>* pool;
};
```



## 

## <a name="c6-5"></a> 6-5 常量命名

**总述**

声明为 `constexpr` 或 `const` 的变量, 或在程序运行期间其值始终保持不变的, 命名时以 "k" 开头, 大小写混合. 例如:

```
const int kDaysInAWeek = 7;
```

**说明**

所有具有静态存储类型的变量 (例如静态变量或全局变量, 参见 [存储类型](http://en.cppreference.com/w/cpp/language/storage_duration#Storage_duration)) 都应当以此方式命名. 对于其他存储类型的变量, 如自动变量等, 这条规则是可选的. 如果不采用这条规则, 就按照一般的变量命名规则.



## <a name="c6-6"></a> 6-6 函数命名

**总述**

常规函数使用大小写混合, 取值和设值函数则要求与变量名匹配: `MyExcitingFunction()`, `MyExcitingMethod()`, `my_exciting_member_variable()`, `set_my_exciting_member_variable()`.

**说明**

一般来说, 函数名的每个单词首字母大写 (即 "驼峰变量名" 或 "帕斯卡变量名"), 没有下划线. 对于首字母缩写的单词, 更倾向于将它们视作一个单词进行首字母大写 (例如, 写作 `StartRpc()` 而非 `StartRPC()`).

```
AddTableEntry()
DeleteUrl()
OpenFileOrDie()
```

(同样的命名规则同时适用于类作用域与命名空间作用域的常量, 因为它们是作为 API 的一部分暴露对外的, 因此应当让它们看起来像是一个函数, 因为在这时, 它们实际上是一个对象而非函数的这一事实对外不过是一个无关紧要的实现细节.)

取值和设值函数的命名与变量一致. 一般来说它们的名称与实际的成员变量对应, 但并不强制要求. 例如 `int count()` 与 `void set_count(int count)`.



## <a name="c6-7"></a> 6-7 命名空间命名

**总述**

命名空间以小写字母命名. 最高级命名空间的名字取决于项目名称. 要注意避免嵌套命名空间的名字之间和常见的顶级命名空间的名字之间发生冲突.

顶级命名空间的名称应当是项目名或者是该命名空间中的代码所属的团队的名字. 命名空间中的代码, 应当存放于和命名空间的名字匹配的文件夹或其子文件夹中.

注意不能用缩写作名称的规则同样适用于命名空间. 命名空间中的代码极少需要涉及命名空间的名称, 因此没有必要在命名空间中使用缩写.

要避免嵌套的命名空间与常见的顶级命名空间发生名称冲突. 由于名称查找规则的存在, 命名空间之间的冲突完全有可能导致编译失败. 尤其是, 不要创建嵌套的 `std` 命名空间. 建议使用更独特的项目标识符 (`websearch::index`, `websearch::index_util`) 而非常见的极易发生冲突的名称 (比如 `websearch::util`).

对于 `internal` 命名空间, 要当心加入到同一 `internal` 命名空间的代码之间发生冲突 (由于内部维护人员通常来自同一团队, 因此常有可能导致冲突). 在这种情况下, 请使用文件名以使得内部名称独一无二 (例如对于 `frobber.h`, 使用 `websearch::index::frobber_internal`).



## <a name="c6-8"></a> 6-8 枚举命名

**总述**

枚举的命名应当和常量或宏一致: `kEnumName` 或是 `ENUM_NAME`.

**说明**

单独的枚举值应该优先采用常量的命名方式. 但宏方式的命名也可以接受. 枚举名 `UrlTableErrors` (以及 `AlternateUrlTableErrors`) 是类型, 所以要用大小写混合的方式.

```
enum UrlTableErrors {
    kOK = 0,
    kErrorOutOfMemory,
    kErrorMalformedInput,
};
enum AlternateUrlTableErrors {
    OK = 0,
    OUT_OF_MEMORY = 1,
    MALFORMED_INPUT = 2,
};
```

2009 年 1 月之前, 我们一直建议采用宏的方式命名枚举值. 由于枚举值和宏之间的命名冲突, 直接导致了很多问题. 由此, 这里改为优先选择常量风格的命名方式. 新代码应该尽可能优先使用常量风格. 但是老代码没必要切换到常量风格, 除非宏风格确实会产生编译期问题.



## <a name="c6-9"></a> 6-9 宏命名

**总述**

你并不打算使用宏对吧? 如果你一定要用, 像这样命名: `MY_MACRO_THAT_SCARES_SMALL_CHILDREN`.

**说明**

参考预处理宏通常 *不应该* 使用宏. 如果不得不用, 其命名像枚举命名一样全部大写, 使用下划线:

```
#define ROUND(x) ...
#define PI_ROUNDED 3.0
```



## <a name="c6-10"></a> 6-10 命名规则的特例

**总述**

如果你命名的实体与已有 C/C++ 实体相似, 可参考现有命名策略.

`bigopen()`: 函数名, 参照 `open()` 的形式

```
uint`: `typedef
```

`bigpos`: `struct` 或 `class`, 参照 `pos` 的形式

`sparse_hash_map`: STL 型实体; 参照 STL 命名约定

```
LONGLONG_MAX`: 常量, 如同 `INT_MAX
```



# <a name="c7"></a> 7 注释

注释虽然写起来很痛苦, 但对保证代码可读性至关重要. 下面的规则描述了如何注释以及在哪儿注释. 当然也要记住: 注释固然很重要, 但最好的代码应当本身就是文档. 有意义的类型名和变量名, 要远胜过要用注释解释的含糊不清的名字.

你写的注释是给代码读者看的, 也就是下一个需要理解你的代码的人. 所以慷慨些吧, 下一个读者可能就是你!

## <a name="c7-1"></a> 7-1 注释风格

**总述**

使用 `//` 或 `/* */`, 统一就好.

**说明**

`//` 或 `/* */` 都可以; 但 `//` *更* 常用. 要在如何注释及注释风格上确保统一.



### <a name="a7-1-1"></a> 检查规则

#### MultilineComment

**所属工具** CppLint

**规则概述** 检查多行注释的不当使用

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper multiline comment style.

1. 多行注释，如果没有搜索到注释结束标识符，给出警告。
2. 检测到了多行注释/* */，lint工具可能会对此给出警告，建议使用//代替之。

#### WhitespaceComments

**所属工具** CppLint

**规则概述** 检查注释的空格

**分类** 代码风格

**严重级别** 警告

**详细描述** At least two spaces is best between code and comments. Should have a space between // and comment

1. 检查代码和注释之间的空格数量，建议最少空2格。如果没有，给出警告。
2. 检查在注释内容和注释符号//之间的空格。如果没有空格，给出警告。



## <a name="c7-2"></a> 7-2 文件注释

**总述**

在每一个文件开头加入版权公告.

文件注释描述了该文件的内容. 如果一个文件只声明, 或实现, 或测试了一个对象, 并且这个对象已经在它的声明处进行了详细的注释, 那么就没必要再加上文件注释. 除此之外的其他文件都需要文件注释.

### 法律公告和作者信息

每个文件都应该包含许可证引用. 为项目选择合适的许可证版本.(比如, Apache 2.0, BSD, LGPL, GPL)

如果你对原始作者的文件做了重大修改, 请考虑删除原作者信息.

#### MissingCopyright

**所属工具** CppLint

**规则概述** 检查文件中是否包含“Copyright [year]”

**分类** 代码风格

**严重级别** 警告

**详细描述** No copyright message found.You should have a line: Copyright [year](https://seer-group.coding.net/p/src/code-analysis/44751/check-profile/497242/add-rule?pkgId=497283&scheme=497309) 

1. 检查文件中是否包含“Copyright [year](https://seer-group.coding.net/p/src/code-analysis/44751/check-profile/497242/add-rule?pkgId=497283&scheme=497309)”，如果不包含，给出警告。



### 文件内容

如果一个 `.h` 文件声明了多个概念, 则文件注释应当对文件的内容做一个大致的说明, 同时说明各概念之间的联系. 一个一到两行的文件注释就足够了, 对于每个概念的详细文档应当放在各个概念中, 而不是文件注释中.

不要在 `.h` 和 `.cc` 之间复制注释, 这样的注释偏离了注释的实际意义.



## <a name="c7-3"></a> 7-3 类注释

**总述**

每个类的定义都要附带一份注释, 描述类的功能和用法, 除非它的功能相当明显.

```
// Iterates over the contents of a GargantuanTable.
// Example:
//    GargantuanTableIterator* iter = table->NewIterator();
//    for (iter->Seek("foo"); !iter->done(); iter->Next()) {
//      process(iter->key(), iter->value());
//    }
//    delete iter;
class GargantuanTableIterator {
  ...
};
```

**说明**

类注释应当为读者理解如何使用与何时使用类提供足够的信息, 同时应当提醒读者在正确使用此类时应当考虑的因素. 如果类有任何同步前提, 请用文档说明. 如果该类的实例可被多线程访问, 要特别注意文档说明多线程环境下相关的规则和常量使用.

如果你想用一小段代码演示这个类的基本用法或通常用法, 放在类注释里也非常合适.

如果类的声明和定义分开了(例如分别放在了 `.h` 和 `.cc` 文件中), 此时, 描述类用法的注释应当和接口定义放在一起, 描述类的操作和实现的注释应当和实现放在一起.



## <a name="c7-4"></a> 7-4 函数注释

**总述**

函数声明处的注释描述函数功能; 定义处的注释描述函数实现.	

### 函数声明

基本上每个函数声明处前都应当加上注释, 描述函数的功能和用途. 只有在函数的功能简单而明显时才能省略这些注释(例如,  简单的取值和设值函数). 注释使用叙述式 ("Opens the file") 而非指令式 ("Open the file");  注释只是为了描述函数, 而不是命令函数做什么. 通常, 注释不会描述函数如何工作. 那是函数定义部分的事情.

函数声明处注释的内容:

- 函数的输入输出.
- 对类成员函数而言: 函数调用期间对象是否需要保持引用参数, 是否会释放这些参数.
- 函数是否分配了必须由调用者释放的空间.
- 参数是否可以为空指针.
- 是否存在函数使用上的性能隐患.
- 如果函数是可重入的, 其同步前提是什么?

举例如下:

```
// Returns an iterator for this table.  It is the client's
// responsibility to delete the iterator when it is done with it,
// and it must not use the iterator once the GargantuanTable object
// on which the iterator was created has been deleted.
//
// The iterator is initially positioned at the beginning of the table.
//
// This method is equivalent to:
//    Iterator* iter = table->NewIterator();
//    iter->Seek("");
//    return iter;
// If you are going to immediately seek to another place in the
// returned iterator, it will be faster to use NewIterator()
// and avoid the extra seek.
Iterator* GetIterator() const;
```

但也要避免罗罗嗦嗦, 或者对显而易见的内容进行说明. 下面的注释就没有必要加上 "否则返回 false", 因为已经暗含其中了:

```
// Returns true if the table cannot hold any more entries.
bool IsTableFull();
```

注释函数重载时, 注释的重点应该是函数中被重载的部分, 而不是简单的重复被重载的函数的注释. 多数情况下, 函数重载不需要额外的文档, 因此也没有必要加上注释.

注释构造/析构函数时, 切记读代码的人知道构造/析构函数的功能, 所以 "销毁这一对象" 这样的注释是没有意义的.  你应当注明的是注明构造函数对参数做了什么 (例如, 是否取得指针所有权) 以及析构函数清理了什么. 如果都是些无关紧要的内容, 直接省掉注释.  析构函数前没有注释是很正常的.



### 函数定义

如果函数的实现过程中用到了很巧妙的方式, 那么在函数定义处应当加上解释性的注释. 例如, 你所使用的编程技巧, 实现的大致步骤, 或解释如此实现的理由. 举个例子, 你可以说明为什么函数的前半部分要加锁而后半部分不需要.

*不要* 从 `.h` 文件或其他地方的函数声明处直接复制注释. 简要重述函数功能是可以的, 但注释重点要放在如何实现上.



## <a name="c7-5"></a> 7-5 变量注释

**总述**

通常变量名本身足以很好说明变量用途. 某些情况下, 也需要额外的注释说明.

### 类数据成员

每个类数据成员 (也叫实例变量或成员变量) 都应该用注释说明用途. 如果有非变量的参数(例如特殊值, 数据成员之间的关系,  生命周期等)不能够用类型与变量名明确表达, 则应当加上注释. 然而, 如果变量类型与变量名已经足以描述一个变量, 那么就不再需要加上注释.

特别地, 如果变量可以接受 `NULL` 或 `-1` 等警戒值, 须加以说明. 比如:

```
private:
 // Used to bounds-check table accesses. -1 means
 // that we don't yet know how many entries the table has.
 int num_total_entries_;
```



### 全局变量

和数据成员一样, 所有全局变量也要注释说明含义及用途, 以及作为全局变量的原因. 比如:

```
// The total number of tests cases that we run through in this regression test.
const int kNumTestCases = 6;
```



## <a name="c7-6"></a> 7-6 实现注释

**总述**

对于代码中巧妙的, 晦涩的, 有趣的, 重要的地方加以注释.

### 代码前注释

巧妙或复杂的代码段前要加注释. 比如:

```
// Divide result by two, taking into account that x
// contains the carry from the add.
for (int i = 0; i < result->size(); i++) {
  x = (x << 8) + (*result)[i];
  (*result)[i] = x >> 1;
  x &= 1;
}
```



### 行注释

比较隐晦的地方要在行尾加入注释. 在行尾空两格进行注释. 比如:

```
// If we have enough memory, mmap the data portion too.
mmap_budget = max<int64>(0, mmap_budget - index_->length());
if (mmap_budget >= data_size_ && !MmapData(mmap_chunk_bytes, mlock))
  return;  // Error already logged.
```

注意, 这里用了两段注释分别描述这段代码的作用, 和提示函数返回时错误已经被记入日志.

如果你需要连续进行多行注释, 可以使之对齐获得更好的可读性:

```
DoSomething();                  // Comment here so the comments line up.
DoSomethingElseThatIsLonger();  // Two spaces between the code and the comment.
{ // One space before comment when opening a new scope is allowed,
  // thus the comment lines up with the following comments and code.
  DoSomethingElse();  // Two spaces before line comments normally.
}
std::vector<string> list{
                    // Comments in braced lists describe the next element...
                    "First item",
                    // .. and should be aligned appropriately.
"Second item"};
DoSomething(); /* For trailing block comments, one space is fine. */
```



### 函数参数注释

如果函数参数的意义不明显, 考虑用下面的方式进行弥补:

- 如果参数是一个字面常量, 并且这一常量在多处函数调用中被使用, 用以推断它们一致, 你应当用一个常量名让这一约定变得更明显, 并且保证这一约定不会被打破.
- 考虑更改函数的签名, 让某个 `bool` 类型的参数变为 `enum` 类型, 这样可以让这个参数的值表达其意义.
- 如果某个函数有多个配置选项, 你可以考虑定义一个类或结构体以保存所有的选项, 并传入类或结构体的实例. 这样的方法有许多优点,  例如这样的选项可以在调用处用变量名引用, 这样就能清晰地表明其意义. 同时也减少了函数参数的数量, 使得函数调用更易读也易写. 除此之外,  以这样的方式, 如果你使用其他的选项, 就无需对调用点进行更改.
- 用具名变量代替大段而复杂的嵌套表达式.
- 万不得已时, 才考虑在调用点用注释阐明参数的意义.

比如下面的示例的对比:

```
// What are these arguments?
const DecimalNumber product = CalculateProduct(values, 7, false, nullptr);
```

和

```
ProductOptions options;
options.set_precision_decimals(7);
options.set_use_cache(ProductOptions::kDontUseCache);
const DecimalNumber product =
    CalculateProduct(values, options, /*completion_callback=*/nullptr);
```

哪个更清晰一目了然.



### 不允许的行为

不要描述显而易见的现象, *永远不要* 用自然语言翻译代码作为注释, 除非即使对深入理解 C++ 的读者来说代码的行为都是不明显的. 要假设读代码的人 C++ 水平比你高, 即便他/她可能不知道你的用意:

你所提供的注释应当解释代码 *为什么* 要这么做和代码的目的, 或者最好是让代码自文档化.

比较这样的注释:

```
// Find the element in the vector.  <-- 差: 这太明显了!
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
```

和这样的注释:

```
// Process "element" unless it was already processed.
auto iter = std::find(v.begin(), v.end(), element);
if (iter != v.end()) {
  Process(element);
}
```

自文档化的代码根本就不需要注释. 上面例子中的注释对下面的代码来说就是毫无必要的:

```
if (!IsAlreadyProcessed(element)) {
  Process(element);
}
```



## <a name="c7-7"></a> 7-7 标点, 拼写和语法

**总述**

注意标点, 拼写和语法; 写的好的注释比差的要易读的多.

**说明**

注释的通常写法是包含正确大小写和结尾句号的完整叙述性语句. 大多数情况下, 完整的句子比句子片段可读性更高. 短一点的注释, 比如代码行尾注释, 可以随意点, 但依然要注意风格的一致性.

虽然被别人指出该用分号时却用了逗号多少有些尴尬, 但清晰易读的代码还是很重要的. 正确的标点, 拼写和语法对此会有很大帮助.



## <a name="c7-8"></a> 7-8 TODO 注释

**总述**

对那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 `TODO` 注释.

`TODO` 注释要使用全大写的字符串 `TODO`, 在随后的圆括号里写上你的名字, 邮件地址, bug ID, 或其它身份标识和与这一 `TODO` 相关的 issue. 主要目的是让添加注释的人 (也是可以请求提供更多细节的人) 可根据规范的 `TODO` 格式进行查找. 添加 `TODO` 注释并不意味着你要自己来修正, 因此当你加上带有姓名的 `TODO` 时, 一般都是写上自己的名字.

```
// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature
```

如果加 `TODO` 是为了在 "将来某一天做某事", 可以附上一个非常明确的时间 "Fix by November 2005"), 或者一个明确的事项 ("Remove this code when all clients can handle XML  responses.").

### <a name="a7-8-1"></a> 检查规则

#### MissingUsernameInTodo

**所属工具** CppLint

**规则概述** 检查TODO注释的格式是否正确

**分类** 代码风格

**严重级别** 警告

**详细描述** Missing username in TODO; it should look like '// TODO(my_username): Stuff.'.

1. 检查TODO注释的格式是否正确，建议格式为“// TODO(my_username): Stuff.”

#### WhitespaceTodo

**所属工具** CppLint

**规则概述** 检查TODO注释前后的空格数量

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper use of whitespace near TODO.

1. 检查TODO注释前后的空格数量，如果没有空格，会给出警告；如果多余1个空格，也给出警告。



## <a name="c7-9"></a> 7-9 弃用注释

**总述**

通过弃用注释（`DEPRECATED` comments）以标记某接口点已弃用.

您可以写上包含全大写的 `DEPRECATED` 的注释, 以标记某接口为弃用状态. 注释可以放在接口声明前, 或者同一行.

在 `DEPRECATED` 一词后, 在括号中留下您的名字, 邮箱地址以及其他身份标识.

弃用注释应当包涵简短而清晰的指引, 以帮助其他人修复其调用点. 在 C++ 中, 你可以将一个弃用函数改造成一个内联函数, 这一函数将调用新的接口.

仅仅标记接口为 `DEPRECATED` 并不会让大家不约而同地弃用, 您还得亲自主动修正调用点（callsites）, 或是找个帮手.

修正好的代码应该不会再涉及弃用接口点了, 着实改用新接口点. 如果您不知从何下手, 可以找标记弃用注释的当事人一起商量.



# <a name="c8"></a> 8 格式

每个人都可能有自己的代码风格和格式, 但如果一个项目中的所有人都遵循同一风格的话, 这个项目就能更顺利地进行.  每个人未必能同意下述的每一处格式规则, 而且其中的不少规则需要一定时间的适应, 但整个项目服从统一的编程风格是很重要的,  只有这样才能让所有人轻松地阅读和理解代码.

## <a name="c8-1"></a> 8-1 行长度

**总述**

每一行代码字符数不超过 100.

我们也认识到这条规则是有争议的, 但很多已有代码都遵照这一规则, 因此我们感觉一致性更重要.

**优点**

提倡该原则的人认为强迫他们调整编辑器窗口大小是很野蛮的行为. 很多人同时并排开几个代码窗口, 根本没有多余的空间拉伸窗口. 大家都把窗口最大尺寸加以限定, 并且 80 列宽是传统标准. 那么为什么要改变呢?

**缺点**

反对该原则的人则认为更宽的代码行更易阅读.100 列的限制是上个世纪 60 年代的大型机的古板缺陷; 现代设备具有更宽的显示屏, 可以很轻松地显示更多代码.

**结论**

100 个字符是最大值.

如果无法在不伤害易读性的条件下进行断行, 那么注释行可以超过 100 个字符, 这样可以方便复制粘贴. 例如, 带有命令示例或 URL 的行可以超过 80 个字符.

包含长路径的 `#include` 语句可以超出100列.



### <a name="a8-1-1"></a> 检查规则

#### LineLength

**所属工具** CppLint

**规则概述** 检查每一行代码的长度

**分类** 代码风格

**严重级别** 警告

**详细描述** Line too long.

1. 检查每一行代码的长度。对于长度超过120个字符的，给出较严重警告；对于长度超过100个字符的，给出较低级别警告。



## <a name="c8-2"></a> 8-2 非 ASCII 字符

**总述**

尽量不使用非 ASCII 字符, 使用时必须使用 UTF-8 编码.

**说明**

即使是英文, 也不应将用户界面的文本硬编码到源代码中, 因此非 ASCII 字符应当很少被用到. 特殊情况下可以适当包含此类字符. 例如, 代码分析外部数据文件时, 可以适当硬编码数据文件中作为分隔符的非 ASCII 字符串; 更常见的是 (不需要本地化的) 单元测试代码可能包含非 ASCII 字符串. 此类情况下, 应使用 UTF-8 编码, 因为很多工具都可以理解和处理 UTF-8 编码.

十六进制编码也可以, 能增强可读性的情况下尤其鼓励 —— 比如 `"\xEF\xBB\xBF"`, 或者更简洁地写作 `u8"\uFEFF"`, 在 Unicode 中是 *零宽度 无间断* 的间隔符号, 如果不用十六进制直接放在 UTF-8 格式的源文件中, 是看不到的.

(Yang.Y 注: `"\xEF\xBB\xBF"` 通常用作 UTF-8 with BOM 编码标记)

使用 `u8` 前缀把带 `uXXXX` 转义序列的字符串字面值编码成 UTF-8. 不要用在本身就带 UTF-8 字符的字符串字面值上, 因为如果编译器不把源代码识别成 UTF-8, 输出就会出错.

别用 C++11 的 `char16_t` 和 `char32_t`, 它们和 UTF-8 文本没有关系, `wchar_t` 同理, 除非你写的代码要调用 Windows API, 后者广泛使用了 `wchar_t`.

### <a name="a8-2-1"></a> 检查规则

#### InvalidUTF8

**所属工具** CppLint

**规则概述** 检查文件中是否包含非法的UTF-8字符

**分类** 功能

**严重级别** 警告

**详细描述** Line contains invalid UTF-8 (or Unicode replacement character).

1. 检查文件中是否包含非法的UTF-8字符，如果存在，给出警告。



## <a name="c8-3"></a> 8-3 空格还是制表位

**总述**

只使用空格, 每次缩进 2 个空格.

**说明**

我们使用空格缩进. 不要在代码中使用制表符. 你应该设置编辑器将制表符转为空格.

### <a name="a8-2-1"></a> 检查规则

#### WhitespaceTab

**所属工具** CppLint

**规则概述** 检查文中是否使用了Tab

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper use of tab.

1. 检查文中是否使用了Tab。如果使用了，给出警告，建议使用空格代替。



## <a name="c8-4"></a> 8-4 函数声明与定义

**总述**

返回类型和函数名在同一行, 参数也尽量放在同一行, 如果放不下就对形参分行.

**说明**

函数看上去像这样:

```
ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
  ...
}
```

如果同一行文本太多, 放不下所有参数:

```
ReturnType ClassName::ReallyLongFunctionName(Type par_name1, Type par_name2,
                                             Type par_name3) {
  DoSomething();
  ...
}
```

甚至连第一个参数都放不下:

```
ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par_name1,  // 4 space indent
    Type par_name2,
    Type par_name3) {
  DoSomething();  // 2 space indent
  ...
}
```

注意以下几点:

- 使用好的参数名.
- 只有在参数未被使用或者其用途非常明显时, 才能省略参数名.
- 如果返回类型和函数名在一行放不下, 分行.
- 如果返回类型与函数声明或定义分行了, 不要缩进.
- 左圆括号总是和函数名在同一行.
- 函数名和左圆括号间永远没有空格.
- 圆括号与参数间没有空格.
- 左大括号总在最后一个参数同一行的末尾处, 不另起新行.
- 右大括号总是单独位于函数最后一行, 或者与左大括号同一行.
- 右圆括号和左大括号间总是有一个空格.
- 所有形参应尽可能对齐.
- 缺省缩进为 2 个空格.
- 换行后的参数保持 4 个空格的缩进.

未被使用的参数, 或者根据上下文很容易看出其用途的参数, 可以省略参数名:

```
class Foo {
 public:
  Foo(Foo&&);
  Foo(const Foo&);
  Foo& operator=(Foo&&);
  Foo& operator=(const Foo&);
};
```

未被使用的参数如果其用途不明显的话, 在函数定义处将参数名注释起来:

```
class Shape {
 public:
  virtual void Rotate(double radians) = 0;
};

class Circle : public Shape {
 public:
  void Rotate(double radians) override;
};

void Circle::Rotate(double /*radians*/) {}
// 差 - 如果将来有人要实现, 很难猜出变量的作用.
void Circle::Rotate(double) {}
```

属性, 和展开为属性的宏, 写在函数声明或定义的最前面, 即返回类型之前:

```
MUST_USE_RESULT bool IsOK();
```

### <a name="a8-4-1"></a> 检查规则

#### WhitespaceParens

**所属工具** CppLint

**规则概述** 检查括号的空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper use of whitespace near parens.

1. 检查函数名和开始的括号(之间是否有空格，如果有空格，给出警告。
2. 检查函数的结束括号)是否和函数名在同一行上。如果不在同一行，在下一行的话给出警告。
3. 检查函数的结束括号）前面是否有空格，如果有空格，给出警告。



## <a name="c8-5"></a> 8-5 函数调用

**总述**

要么一行写完函数调用, 要么在圆括号里对参数分行, 要么参数另起一行且缩进四格. 如果没有其它顾虑的话, 尽可能精简行数, 比如把多个参数适当地放在同一行里.

**说明**

函数调用遵循如下形式：

```
bool retval = DoSomething(argument1, argument2, argument3);
```

如果同一行放不下, 可断为多行, 后面每一行都和第一个实参对齐, 左圆括号后和右圆括号前不要留空格：

```
bool retval = DoSomething(averyveryveryverylongargument1,
                          argument2, argument3);
```

参数也可以放在次行, 缩进四格：

```
if (...) {
  ...
  ...
  if (...) {
    DoSomething(
        argument1, argument2,  // 4 空格缩进
        argument3, argument4);
  }
```

把多个参数放在同一行以减少函数调用所需的行数, 除非影响到可读性. 有人认为把每个参数都独立成行, 不仅更好读, 而且方便编辑参数. 不过, 比起所谓的参数编辑, 我们更看重可读性, 且后者比较好办：

如果一些参数本身就是略复杂的表达式, 且降低了可读性, 那么可以直接创建临时变量描述该表达式, 并传递给函数：

```
int my_heuristic = scores[x] * y + bases[x];
bool retval = DoSomething(my_heuristic, x, y, z);
```

或者放着不管, 补充上注释：

```
bool retval = DoSomething(scores[x] * y + bases[x],  // Score heuristic.
                          x, y, z);
```

如果某参数独立成行, 对可读性更有帮助的话, 那也可以如此做. 参数的格式处理应当以可读性而非其他作为最重要的原则.

此外, 如果一系列参数本身就有一定的结构, 可以酌情地按其结构来决定参数格式：

```
// 通过 3x3 矩阵转换 widget.
my_widget.Transform(x1, x2, x3,
                    y1, y2, y3,
                    z1, z2, z3);
```



## <a name="c8-6"></a> 8-6 列表初始化格式

**总述**

您平时怎么格式化函数调用, 就怎么格式化

**说明**

如果列表初始化伴随着名字, 比如类型或变量名, 格式化时将将名字视作函数调用名, {} 视作函数调用的括号. 如果没有名字, 就视作名字长度为零.

```
// 一行列表初始化示范.
return {foo, bar};
functioncall({foo, bar});
pair<int, int> p{foo, bar};

// 当不得不断行时.
SomeFunction(
    {"assume a zero-length name before {"},  // 假设在 { 前有长度为零的名字.
    some_other_function_parameter);
SomeType variable{
    some, other, values,
    {"assume a zero-length name before {"},  // 假设在 { 前有长度为零的名字.
    SomeOtherType{
        "Very long string requiring the surrounding breaks.",  // 非常长的字符串, 前后都需要断行.
        some, other values},
    SomeOtherType{"Slightly shorter string",  // 稍短的字符串.
                  some, other, values}};
SomeType variable{
    "This is too long to fit all in one line"};  // 字符串过长, 因此无法放在同一行.
MyType m = {  // 注意了, 您可以在 { 前断行.
    superlongvariablename1,
    superlongvariablename2,
    {short, interior, list},
    {interiorwrappinglist,
     interiorwrappinglist2}};
```



## <a name="c8-7"></a> 8-7 条件语句

**总述**

倾向于不在圆括号内使用空格. 关键字 `if` 和 `else` 另起一行.

**说明**

对基本条件语句有两种可以接受的格式. 一种在圆括号和条件之间有空格, 另一种没有.

最常见的是没有空格的格式. 哪一种都可以, 最重要的是 *保持一致*. 如果你是在修改一个文件, 参考当前已有格式. 如果是写新的代码, 参考目录下或项目中其它文件. 还在犹豫的话, 就不要加空格了.

```
if (condition) {  // 圆括号里没有空格.
  ...  // 2 空格缩进.
} else if (...) {  // else 与 if 的右括号同一行.
  ...
} else {
  ...
}
```

如果你更喜欢在圆括号内部加空格:

```
if ( condition ) {  // 圆括号与空格紧邻 - 不常见
  ...  // 2 空格缩进.
} else {  // else 与 if 的右括号同一行.
  ...
}
```

注意所有情况下 `if` 和左圆括号间都有个空格. 右圆括号和左大括号之间也要有个空格:

```
if(condition)     // 差 - IF 后面没空格.
if (condition){   // 差 - { 前面没空格.
if(condition){    // 变本加厉地差.
if (condition) {  // 好 - IF 和 { 都与空格紧邻.
```

如果能增强可读性, 简短的条件语句允许写在同一行. 只有当语句简单并且没有使用 `else` 子句时使用:

```
if (x == kFoo) return new Foo();
if (x == kBar) return new Bar();
```

如果语句有 `else` 分支则不允许:

```
// 不允许 - 当有 ELSE 分支时 IF 块却写在同一行
if (x) DoThis();
else DoThat();
```

通常, 单行语句不需要使用大括号, 如果你喜欢用也没问题; 复杂的条件或循环语句用大括号可读性会更好. 也有一些项目要求 `if` 必须总是使用大括号:

```
if (condition)
  DoSomething();  // 2 空格缩进.

if (condition) {
  DoSomething();  // 2 空格缩进.
}
```

但如果语句中某个 `if-else` 分支使用了大括号的话, 其它分支也必须使用:

```
// 不可以这样子 - IF 有大括号 ELSE 却没有.
if (condition) {
  foo;
} else
  bar;

// 不可以这样子 - ELSE 有大括号 IF 却没有.
if (condition)
  foo;
else {
  bar;
}
// 只要其中一个分支用了大括号, 两个分支都要用上大括号.
if (condition) {
  foo;
} else {
  bar;
}
```

### <a name="a8-7-1"></a> 检查规则

#### EmptyConditionalBody

**所属工具** CppLint

**规则概述** 检查是否存在空条件体（对应于if）

**分类** 代码风格

**严重级别** 警告

**详细描述** Empty conditional bodies should use {}

1. 检查是否存在空条件体（对应于if）。如果存在，给出警告，建议使用{}。

#### EmptyIfBody

**所属工具** CppLint

**规则概述** 检查if条件体是否为空

**分类** 代码风格

**严重级别** 警告

**详细描述**

1. 检查if语块是否有效，是否有else分支，如果没有，给出警告。

#### WhitespaceParens

**所属工具** CppLint

**规则概述** 检查括号的空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper use of whitespace near parens.

1. 检查if\for\while\switch和开始的括号(之间是否有空格，如果没有空格，给出警告。
2. 检查if\for\while\switch后面的括号()之间的空格是否对称。如果不对称。给出警告（如if( foo )这种情况）。
3. 检查if\for\while\switch后面的括号()内侧的空格情况，建议可以有0个或者1个空格。如果不是0个或者1个，给出警告。

#### WhitespaceBraces

**所属工具** CppLint

**规则概述** 检查大括号的格式

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper braces and whitespace

1. 检查“[”符号前面是否有空白。如果有，给出警告。
2. 检查“{”符号前面是否留有空白。如果没有，给出警告。
3. 检查“}else”这种情况的else前面是否留有空白。如果没有，给出警告。
4. 检查“{”是否接在语句最后，即“{”是否直接跟在语句的后面，而不是单独起一行。如果单独占一行，给出警告。



## <a name="c8-8"></a> 8-8 循环和开关选择语句

**总述**

`switch` 语句可以使用大括号分段, 以表明 cases 之间不是连在一起的. 在单语句循环里, 括号可用可不用. 空循环体应使用 `{}` 或 `continue`.

**说明**

`switch` 语句中的 `case` 块可以使用大括号也可以不用, 取决于你的个人喜好. 如果用的话, 要按照下文所述的方法.

如果有不满足 `case` 条件的枚举值, `switch` 应该总是包含一个 `default` 匹配 (如果有输入值没有 case 去处理, 编译器将给出 warning). 如果 `default` 应该永远执行不到, 简单的加条 `assert`:

```
switch (var) {
  case 0: {  // 2 空格缩进
    ...      // 4 空格缩进
    break;
  }
  case 1: {
    ...
    break;
  }
  default: {
    assert(false);
  }
}
```

在单语句循环里, 括号可用可不用：

```
for (int i = 0; i < kSomeNumber; ++i)
  printf("I love you\n");

for (int i = 0; i < kSomeNumber; ++i) {
  printf("I take it back\n");
}
```

空循环体应使用 `{}` 或 `continue`, 而不是一个简单的分号.

```
while (condition) {
  // 反复循环直到条件失效.
}
for (int i = 0; i < kSomeNumber; ++i) {}  // 可 - 空循环体.
while (condition) continue;  // 可 - contunue 表明没有逻辑.
while (condition);  // 差 - 看起来仅仅只是 while/loop 的部分之一.
```

### <a name="a8-8-1"></a> 检查规则

#### EmptyLoopBody

**所属工具** CppLint

**规则概述** 检查是否存在空循环体

**分类** 代码风格

**严重级别** 警告

**详细描述** Empty loop bodies should use {} or continue.

1. 检查是否存在空循环体(对应for、while)。如果存在，给出警告，建议使用{}。

#### ForColon

**所属工具** CppLint

**规则概述** 检查for循环中冒号前后是否有空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Missing space around colon in range-based for loop.

1. 检查for循环中冒号前后是否有空格，如果没有，给出警告。

#### WhitespaceParens

**所属工具** CppLint

**规则概述** 检查括号的空格

**分类** 代码风格

**严重级别** 警告

详细描述 Improper use of whitespace near parens.

1. 检查if\for\while\switch和开始的括号(之间是否有空格，如果没有空格，给出警告。
2. 检查if\for\while\switch后面的括号()之间的空格是否对称。如果不对称。给出警告（如if( foo )这种情况）。
3. 检查if\for\while\switch后面的括号()内侧的空格情况，建议可以有0个或者1个空格。如果不是0个或者1个，给出警告。

#### WhitespaceBraces

**所属工具** CppLint

**规则概述** 检查大括号的格式

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper braces and whitespace

1. 检查“[”符号前面是否有空白。如果有，给出警告。
2. 检查“{”符号前面是否留有空白。如果没有，给出警告。
3. 检查“}else”这种情况的else前面是否留有空白。如果没有，给出警告。
4. 检查“{”是否接在语句最后，即“{”是否直接跟在语句的后面，而不是单独起一行。如果单独占一行，给出警告。





## <a name="c8-9"></a> 8-9 指针和引用表达式

**总述**

句点或箭头前后不要有空格. 指针/地址操作符 (`*, &`) 之后不能有空格.

**说明**

下面是指针和引用表达式的正确使用范例:

```
x = *p;
p = &x;
x = r.y;
x = r->y;
```

注意:

- 在访问成员时, 句点或箭头前后没有空格.
- 指针操作符 `*` 或 `&` 后没有空格.

在声明指针变量或参数时, 星号与类型或变量名紧挨都可以:

```
// 好, 空格前置.
char *c;
const string &str;

// 好, 空格后置.
char* c;
const string& str;
int x, *y;  // 不允许 - 在多重声明中不能使用 & 或 *
char * c;  // 差 - * 两边都有空格
const string & str;  // 差 - & 两边都有空格.
```

在单个文件内要保持风格一致, 所以, 如果是修改现有文件, 要遵照该文件的风格.



## <a name="c8-10"></a> 8-10 布尔表达式

**总述**

如果一个布尔表达式超过行宽, 断行方式要统一一下.

**说明**

下例中, 逻辑与 (`&&`) 操作符总位于行尾:

```
if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
```

注意, 上例的逻辑与 (`&&`) 操作符均位于行尾. 这个格式在 Google 里很常见, 虽然把所有操作符放在开头也可以. 可以考虑额外插入圆括号, 合理使用的话对增强可读性是很有帮助的. 此外, 直接用符号形式的操作符, 比如 `&&` 和 `~`, 不要用词语形式的 `and` 和 `compl`.



## <a name="c8-11"></a> 8-11 函数返回值

**总述**

不要在 `return` 表达式里加上非必须的圆括号.

**说明**

只有在写 `x = expr` 要加上括号的时候才在 `return expr;` 里使用括号.

```
return result;                  // 返回值很简单, 没有圆括号.
// 可以用圆括号把复杂表达式圈起来, 改善可读性.
return (some_long_condition &&
        another_condition);
return (value);                // 毕竟您从来不会写 var = (value);
return(result);                // return 可不是函数！
```



## <a name="c8-12"></a> 8-12 变量及数组初始化

**总述**

用 `=`, `()` 和 `{}` 均可.

**说明**

您可以用 `=`, `()` 和 `{}`, 以下的例子都是正确的：

```
int x = 3;
int x(3);
int x{3};
string name("Some Name");
string name = "Some Name";
string name{"Some Name"};
```

请务必小心列表初始化 `{...}` 用 `std::initializer_list` 构造函数初始化出的类型. 非空列表初始化就会优先调用 `std::initializer_list`, 不过空列表初始化除外, 后者原则上会调用默认构造函数. 为了强制禁用 `std::initializer_list` 构造函数, 请改用括号.

```
vector<int> v(100, 1);  // 内容为 100 个 1 的向量.
vector<int> v{100, 1};  // 内容为 100 和 1 的向量.
```

此外, 列表初始化不允许整型类型的四舍五入, 这可以用来避免一些类型上的编程失误.

```
int pi(3.14);  // 好 - pi == 3.
int pi{3.14};  // 编译错误: 缩窄转换.
```



## <a name="c8-13"></a> 8-13 预处理指令

**总述**

预处理指令不要缩进, 从行首开始.

**说明**

即使预处理指令位于缩进代码块中, 指令也应从行首开始.

```
// 好 - 指令从行首开始
  if (lopsided_score) {
#if DISASTER_PENDING      // 正确 - 从行首开始
    DropEverything();
# if NOTIFY               // 非必要 - # 后跟空格
    NotifyClient();
# endif
#endif
    BackToNormal();
  }
// 差 - 指令缩进
  if (lopsided_score) {
    #if DISASTER_PENDING  // 差 - "#if" 应该放在行开头
    DropEverything();
    #endif                // 差 - "#endif" 不要缩进
    BackToNormal();
  }
```



## <a name="c8-14"></a> 8-14 类格式

**总述**

访问控制块的声明依次序是 `public:`, `protected:`, `private:`, 每个都缩进 1 个空格.

**说明**

类声明的基本格式如下:

```
class MyClass : public OtherClass {
 public:      // 注意有一个空格的缩进
  MyClass();  // 标准的两空格缩进
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
```

注意事项:

- 所有基类名应在 80 列限制下尽量与子类名放在同一行.
- 关键词 `public:`, `protected:`, `private:` 要缩进 1 个空格.
- 除第一个关键词 (一般是 `public`) 外, 其他关键词前要空一行. 如果类比较小的话也可以不空.
- 这些关键词后不要保留空行.
- `public` 放在最前面, 然后是 `protected`, 最后是 `private`.
- 关于声明顺序的规则请参考声明顺序一节.



## <a name="c8-15"></a> 8-15 构造函数初始值列表

**总述**

构造函数初始化列表放在同一行或按四格缩进并排多行.

**说明**

下面两种初始值列表方式都可以接受:

```
// 如果所有变量能放在同一行:
MyClass::MyClass(int var) : some_var_(var) {
  DoSomething();
}

// 如果不能放在同一行,
// 必须置于冒号后, 并缩进 4 个空格
MyClass::MyClass(int var)
    : some_var_(var), some_other_var_(var + 1) {
  DoSomething();
}

// 如果初始化列表需要置于多行, 将每一个成员放在单独的一行
// 并逐行对齐
MyClass::MyClass(int var)
    : some_var_(var),             // 4 space indent
      some_other_var_(var + 1) {  // lined up
  DoSomething();
}

// 右大括号 } 可以和左大括号 { 放在同一行
// 如果这样做合适的话
MyClass::MyClass(int var)
    : some_var_(var) {}
```



## <a name="c8-16"></a> 8-16 命名空间格式化

**总述**

命名空间内容不缩进.

**说明**

命名空间不要增加额外的缩进层次, 例如:

```
namespace {

void foo() {  // 正确. 命名空间内没有额外的缩进.
  ...
}

}  // namespace
```

不要在命名空间内缩进:

```
namespace {

  // 错, 缩进多余了.
  void foo() {
    ...
  }

}  // namespace
```

声明嵌套命名空间时, 每个命名空间都独立成行.

```
namespace foo {
namespace bar {
```



## <a name="c8-17"></a> 8-17 水平留白

**总述**

水平留白的使用根据在代码中的位置决定. 永远不要在行尾添加没意义的留白.

### 通用

```
void f(bool b) {  // 左大括号前总是有空格.
  ...
int i = 0;  // 分号前不加空格.
// 列表初始化中大括号内的空格是可选的.
// 如果加了空格, 那么两边都要加上.
int x[] = { 0 };
int x[] = {0};

// 继承与初始化列表中的冒号前后恒有空格.
class Foo : public Bar {
 public:
  // 对于单行函数的实现, 在大括号内加上空格
  // 然后是函数实现
  Foo(int b) : Bar(), baz_(b) {}  // 大括号里面是空的话, 不加空格.
  void Reset() { baz_ = 0; }  // 用空格把大括号与实现分开.
  ...
```

添加冗余的留白会给其他人编辑时造成额外负担. 因此, 行尾不要留空格. 如果确定一行代码已经修改完毕, 将多余的空格去掉;  或者在专门清理空格时去掉（尤其是在没有其他人在处理这件事的时候). (Yang.Y 注: 现在大部分代码编辑器稍加设置后,  都支持自动删除行首/行尾空格, 如果不支持, 考虑换一款编辑器或 IDE)

#### WhitespaceSemicolon

**所属工具** CppLint

**规则概述** 检查分号的空格

**分类** 代码风格

**严重级别** 警告

详细描述 Improper use of whitespace near semicolon.

1. 在分号“;”之后需要有空格。如果没有，给出警告。
2. 检查使用分号“;”表示空状态的语句。如果检查到了，给出警告，并提示使用{}代替。
3. 检查每行最后一个分号“;”，看其前面是否有空格。如果有空格，给出警告。

#### LineEndWhitespace

**所属工具** CppLint

**规则概述** 检查每一行的末尾是否有空格

**分类** 代码风格

**严重级别** 警告

详细描述 Line ends in whitespace. Consider deleting these extra spaces.

1. 检查每一行的末尾是否有空格。如果有空格，给出警告，建议删除这些空格。

#### MissingSpaceAfterComma

**所属工具** CppLint

**规则概述** 在逗号“,”之后需要有空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Missing space after ,

1. 在逗号“,”之后需要有空格。如果没有，给出警告。



### 循环和条件语句

```
if (b) {          // if 条件语句和循环语句关键字后均有空格.
} else {          // else 前后有空格.
}
while (test) {}   // 圆括号内部不紧邻空格.
switch (i) {
for (int i = 0; i < 5; ++i) {
switch ( i ) {    // 循环和条件语句的圆括号里可以与空格紧邻.
if ( test ) {     // 圆括号, 但这很少见. 总之要一致.
for ( int i = 0; i < 5; ++i ) {
for ( ; i < 5 ; ++i) {  // 循环里内 ; 后恒有空格, ;  前可以加个空格.
switch (i) {
  case 1:         // switch case 的冒号前无空格.
    ...
  case 2: break;  // 如果冒号有代码, 加个空格.
```

#### EmptyLoopBody

**所属工具** CppLint

**规则概述** 检查是否存在空循环体

**分类** 代码风格

**严重级别** 警告

**详细描述** Empty loop bodies should use {} or continue.

1. 检查是否存在空循环体(对应for、while)。如果存在，给出警告，建议使用{}。

#### ForColon

**所属工具** CppLint

**规则概述** 检查for循环中冒号前后是否有空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Missing space around colon in range-based for loop.

1. 检查for循环中冒号前后是否有空格，如果没有，给出警告。

#### WhitespaceParens

**所属工具** CppLint

**规则概述** 检查括号的空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper use of whitespace near parens.

1. 检查if\for\while\switch和开始的括号(之间是否有空格，如果没有空格，给出警告。
2. 检查if\for\while\switch后面的括号()之间的空格是否对称。如果不对称。给出警告（如if( foo )这种情况）。
3. 检查if\for\while\switch后面的括号()内侧的空格情况，建议可以有0个或者1个空格。如果不是0个或者1个，给出警告。

#### WhitespaceBraces

**所属工具** CppLint

**规则概述** 检查大括号的格式

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper braces and whitespace

1. 检查“[”符号前面是否有空白。如果有，给出警告。
2. 检查“{”符号前面是否留有空白。如果没有，给出警告。
3. 检查“}else”这种情况的else前面是否留有空白。如果没有，给出警告。
4. 检查“{”是否接在语句最后，即“{”是否直接跟在语句的后面，而不是单独起一行。如果单独占一行，给出警告。



### 操作符

```
// 赋值运算符前后总是有空格.
x = 0;

// 其它二元操作符也前后恒有空格, 不过对于表达式的子式可以不加空格.
// 圆括号内部没有紧邻空格.
v = w * x + y / z;
v = w*x + y/z;
v = w * (x + z);

// 在参数和一元操作符之间不加空格.
x = -5;
++x;
if (x && !y)
  ...
```

#### AltTokens

**所属工具** CppLint

**规则概述** 检查符号（&&、/、//、^、~、&、&=、|=、^=、!、!=）的使用

**分类** 代码风格

**严重级别** 警告

**详细描述** Use another operator instead.

1. 检查符号（and、bitor、or、xor、compl、bitand、and_eq、or_eq、xor_eq、not和not_eq）的使用，建议使用（&&、/、//、^、~、&、&=、|=、^=、!、!=）代替以上几类符号。

#### WhitespaceOperators

**所属工具** CppLint

**规则概述** 检查运算符的空格

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper use of whitespace near operators.

1. 检查“=”号两边是否有空格。如果没有，给出警告。
2. 检查“==|!=|<=|>=||”双目运算符两边是否有空格。如果没有，给出警告。
3. 检查“<|>|<<|>>|!|~|--|++”等单目运算符两边是否有空格。如果没有，给出警告。
4. 检查“&&”两边是否有空格。如果没有，给出警告。



### 模板和转换

```
// 尖括号(< and >) 不与空格紧邻, < 前没有空格, > 和 ( 之间也没有.
vector<string> x;
y = static_cast<char*>(x);

// 在类型与指针操作符之间留空格也可以, 但要保持一致.
vector<char *> x;
```



## <a name="c8-18"></a> 8-18 垂直留白

**总述**

垂直留白越少越好.

**说明**

这不仅仅是规则而是原则问题了: 不在万不得已, 不要使用空行. 尤其是: 两个函数定义之间的空行不要超过 2 行, 函数体首尾不要留空行, 函数体中也不要随意添加空行.

基本原则是: 同一屏可以显示的代码越多, 越容易理解程序的控制流. 当然, 过于密集的代码块和过于疏松的代码块同样难看, 这取决于你的判断. 但通常是垂直留白越少越好.

下面的规则可以让加入的空行更有效:

- 函数体内开头或结尾的空行可读性微乎其微.
- 在多重 if-else 块里加空行或许有点可读性.



## <a name="c8-19"></a> 8-19 其他格式规则

### <a name="a8-19-1"></a> 检查规则

#### ImproperBraces

**所属工具** CppLint

**规则概述** 检查大括号的不当使用

**分类** 代码风格

**严重级** 警告

**详细描述** Improper ues of braces.

1. 检查if ... elseif ... elseif这种结构，如果其中有if或者else if使用了大括号({})，则其他的if或者else if没有使用大括号（{}），则给出警告。
2. 检查if或else体中有多条语句时，是否有大括号{}。如果没有，则给出警告。
3. 检查else是否和与之匹配的if有同样的缩进，如果没有，给出警告；同时建议用户，对于嵌套关系比较模糊的情况，使用{}标示。
4. 检查右大括号}后面是否有“;”，如果有，给出警告。PS：对于namespace、class等正确的情况，不会给出警告。
5. 检查if是否在单独一行，如果不在单独一行，给出警告。

#### ImproperIndent

**所属工具** CppLint

**规则概述** 检查缩进

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper indent.

1. 检查每一行开始的缩进数量是否合法。如果出现奇数个缩进的情况，给出警告。建议使用2个空格缩进。
2. 结束的括号（如)、}）应该和开始的括号对齐。如果不对其，给出警告。
3. 检查public、private、protected、signals和slots的缩进是否合理。建议缩进一个空格，如果不是，给出警告。

#### ImproperNewline

**所属工具** CppLint

**规则概述** 检查不当的换行风格

**分类** 代码风格

**严重级别** 警告

**详细描述** Improper newline.

1. 检查一行上是否有多条语句。如果出现，给出警告。
2. 检查else语句的位置，建议和}在一行上。如果不在一行上，给出警告。
3. 检查是否出现“else{”这种语句。如果出现，给出警告，建议将{放到下一行。
4. 检查{是否和do\while在同一行上。如果出现，给出警告，建议将{放到下一行。
5. 检查在换行的时候，是否使用了回车\r。如果使用了，给出警告，建议使用\n换行。

#### NULByte

**所属工具** CppLint

**规则概述** 检查文件中是否存在'\0'字符

**分类** 代码风格

**严重级别** 警告 

**详细描述** Line contains NUL byte.

1. 检查文件中是否存在'\0'字符，即NUL字符，如果存在，给出警告。

   

# <a name="c9"></a> 9 安全标准

**待续。**

