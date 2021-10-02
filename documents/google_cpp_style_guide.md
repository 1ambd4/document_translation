## 内容清单

- [C++版本](#C++版本)
- 头文件
  - [Self-contained 头文件](#Self-containted头文件)
  - [头文件保护](#头文件保护)
  - [只引入你需要的](#只引入你需要的)
  - [前置声明](#前置声明)
  - [内联函数](#内联函数)
  - [引入的路径和顺序](#引入的路径和顺序)
- 作用域
  - [名字空间](#名字空间)
- 类
- 函数
- C++其他特性
- [命名](#命名)
  - [通用命名规则](#通用命名规则)
  - [文件命名](#文件命名)
  - [类型命名](#类型命名)
  - [变量命名](#变量命名)
  - [常量命名](#常量命名)
  - [函数命名](#函数命名)
  - [名字空间命名](#名字空间命名)
  - [枚举命名](#枚举命名)
  - [宏命名](#宏命名)
  - [命名规则的特例](#命名规则的特例)
- 注释
- 格式
- 例外
- 写在最后


## C++版本

目前，应该拥抱C++17标准，也就是说，还不应该使用C++2x的新特性。

不要使用非标准的编译器扩展功能。

在使用C++14和C++17特性之前，慎重考虑代码的可移植性。


## 头文件

通常来说，每一个`.cc`文件都应该有一个与其关联的`.h`头文件。但如果是单元测试或者特别小的`.cc`文件，小到仅有一个`main()`函数，那么可以不创建对应的头文件。

正确的使用头文件，可以让代码更具可读性，同时性能和文件大小也有极大的改善。

接下来的规则将指导跨越头文件使用过程中常见的陷阱。

### Self-contained头文件

头文件应当是自洽的，并且都以`.h`结尾，单纯用来插入的文本，应当以`.inc`结尾，不允许使用`-inl.h`。

自洽使得用户和重构工具不需要为特定情况另外引入头文件，一般用#define guard做头文件保护，在其间引入所有需要引入的头文件。

模板和内联函数的定义应当和声明写在一起，任何使用它们的地方都应当引入头文件，否则链接器会报错。如果声明和定义放在了不同的文件中，引入前者也会相应的引入后者。

有一个例外情况，如果某函数模版为所有相关的模板参数显示实例化，或者其自身是某个类的私有成员，那么，就只能在实例化该模板的`.cc`文件中定义。

很少有头文件不是自洽的，如果有的话，常常以`.inc`作为扩展名，并且往往是在非常规位置引入，比如说另一个文件的中间。此时它们有可能不使用头文件保护，且未包含自身需要的头文件。

### 头文件保护

所有的头文件都应该有头文件保护以避免多次引入和循环引入，命名格式：`<PROJECT>_<PATH>_<FILE>_H_`。

为了确保唯一性，命名应当基于项目源代码树的全路径，如，`foo`项目里的文件`foo/src/bar/baz.h`应当如下做头文件保护：

``` c++
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
```

### 只引入你需要的

如果源文件或者头文件使用了定义在它处的符号，应当在恰当的位置引入提供此符号声明或定义的头文件，除此之外，不应为任何理由多引入其他任何头文件。

不要依赖包含的传递性，因为你不能确保头文件的作者是否某天会移除掉某些个头文件。同样，即使`bar.h`引入了`foo.h`，`foo.cc`若使用到了定义于`bar.h`中的某个符号，仍然需要引入`bar.h`。

### 前置声明

如无必要，减少前置声明的使用，使用`#include`引入需要的头文件即可。

定义：

前置声明仅仅只是声明，不伴随定义出现。
``` c++
// In a C++ source file:
class B;
void FuncInB();
extern int variable_in_b;
ABSL_DECLARE_FLAG(flag_in_b);
```

优点：

- 前置声明可以节省编译时间，不像`#include`那样强迫编译器打开多个文件处理读入
- 前置声明可以节省不必要的重复编译，而使用`#include`引入头文件，当头文件发生无关的变动时，会迫使编译器重新编译

缺点：

- 前置声明会隐藏依赖，当引入的头文件发生变化时，用户代码不会重新编译。
- 前置声明和头文件引入相比，不利于自动化工具查找符号在何模块中定义。
- 前置声明可能会被库的后续变动破坏。前置声明函数或模版可能会妨碍头文件开发人员改动API，像拓展参数类型、添加带默认值的模板参数，以及迁移到另一个名字空间。
- 前置声明`std`名字空间里的符号时，其行为是未定义的。
- 很难判断什么时候需要前置声明，用前置声明替代头文件引入会改变代码逻辑。
``` c++
// b.h:
struct B {};
struct D : B {};

// good_user.cc:
#include "b.h"
void f(B*);
void f(void*);
void test(D* x) { f(x); }   // call f(B*)

// 如果使用前置声明替代头文件引入，test()将会调用f(void*)
// 译者：是说前置声明隐藏掉了继承关系么？
```
- 前置声明过多来自同一个头文件的符号比头文件引入看起来杂乱。
- 出于使用前置声明的原因进行重构，比如使用指针成员代替对象成员，会让代码变得复杂且影响效率。

结论：

尽量避免前置声明那些定义在其他项目中的实体。

### 内联函数

仅当函数体不足十行的时候，才将其声明为内联函数。

定义：

你可以声明函数为内联来建议编译器对函数进行展开，以绕开常规函数调用机制。

优点：

当被内联的函数足够小时可以生成更高效的代码，对于`set()/get()`函数、效率敏感的函数以及足够短的函数，请尽情声明其为内联。

缺点：

滥用内联可能会拖慢程序性能。视函数体大小，将其声明为内联可能会让代码的总体积发生变化。比如，声明`set()`为内联往往会减小代码体积，而声明函数体很大的函数往往会急剧增大代码体积。现代处理器得益于指令缓存，运行小巧代码的效率要高得多。

结论：

一条好用的经验法则是不要将超过十行的代码声明为内联。谨慎对待析构函数，它们往往比看起来长的多，因为有隐含的成员和基类析构函数调用。


另一条经验法则是：内联包含循环或者选择结构的函数往往得不偿失，除非这些语句很少被执行。

即使将函数声明为内联，编译器也并不一定会将其内联，这点很重要。例如，虚函数和递归函数就不会被内联。编译时编译器无法明确递归深度，所以不会对递归函数进行内联。虚函数内联的主要原因是想把其函数体放到类的定义内，或者当作文档描述其行为，比如短小的`set()/get()`。

#### 注
GCC支持强制内联，如下：
``` c++
inline void Test::set_value(int value) __attribute__((always_inline))
{
    this->value = value;
}

当然，也可以宏替换
#define inline __attribute((always_inline))
```

### 引入的路径和顺序 

引入头文件应当遵循如下规则：相关头文件、C库、C++库、其他库的头文件、项目内的头文件。

项目内的头文件应当按照项目源代码目录结构排列，避免使用unix风格的`.`或者`..`。例如，`google-awesome-project/src/base/logging.h`应当按如下方式引入：

``` c++
#include "base/logging.h"
```

在`dir2/foo2.h`的实现文件`dir/foo.cc`和测试文件`dir/foo_test.cc`中，应当按如下的顺序包含：
1. `dir/foo2.h`
2. 空一行
3. C库，如`<unistd.h>`、`<stdlib.h>`
4. 空一行
5. C++库，如`<algorithm>`、`<cstddef>`
6. 空一行
7. 其他库的头文件
8. 项目内的头文件

按照如上推荐的顺序引入头文件，如果`dir/foo2.h`遗漏了必要的头文件，`dir/foo.cc`和`dri/foo_test.cc`将无法通过编译，这使得维护这些文件的人首先看到编译错误的原因，而不是将这些原因扔给维护其他包的人。

`dir/foo.cc`和`dir2/foo2.h`通常位于同一路径下（如`base/basictypes_test.cc`和`basictypes.h`），但更多时候分开存放。

注意C库中很多头文件，如`stddef.h`可以和C++库中的`cstddef`呼唤，两种都可以接受，优先保持和项目原风格一致。

按照字典序排列头文件，当发现不符合此风格的老旧代码，应当给予修正。

例如，`google-awesome-project/src/foo/internal/fooserver.cc`包含顺序如下：

``` c++
#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "base/basictypes.h"
#include "base/commandlinflags.h"
#include "foo/server/bar.h"
```

例外：

有时候，平台相关的代码需要条件编译，可以将这些代码放到每个部分的后面。当然，要确保这部分代码小巧。
``` c++
#include "foo/public/fooserver.h"

#include "base/port.h"      // For LANG_CXX11.

#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
```


## 作用域

### 名字空间

如无例外，请将代码置于名字空间内。名字空间命名应当基于项目名和路径，以减少发生冲突的可能。不用使用`using`指令，如`using namespace foo`，也不要使用内联名字空间。匿名名字空间，参考内部链接。

定义：

名字空间将全局作用域细分成独立的具名作用域，有助于避免命名冲突。

优点：

名字空间的出现给代码体量大的项目减少命名冲突提供了可能。

例如，两个不同的项目各自有一个命名为`Foo`的类，在编译的时候会发生冲突。而当把类置于名字空间之下，两个类可以使用`project1::Foo`和`project2::Foo`进行区分。

内联名字空间会自动将名字置于外层作用域，参考如下代码段：
``` c++
namespace outer {
inline namespace inner {
    void foo();
}   // namespace inner
}   // namespace outer
```
`outer::inner::foo()`和`outer::foo()`都是正确的，内联名字空间的出现是为了解决跨版本的ABI兼容问题。

缺点：

名字空间把指明名字意思的机制变得复杂，很容易使人困惑，特别是内联名字空间。

有时候使用多层嵌套的名字空间，写出完整的名字空间名称，会让代码变得杂乱不堪。

建议：

名字空间的使用应当遵循如下规则：
- 准照[名字空间命名](#名字空间名字)规则
- 像给出的例子一样，在名字空间的结尾用注释标明名字空间名
- 用名字空间把`includes`、`gflags`的声明和定义以及从其他名字空间中引入的类的前置声明之外的所有代码都包裹起来
``` c++
// .h文件
namespace mynamespace {

// 所有的声明都写在名字空间内
// 注意不要缩进

class MyClass {
public:
    ...
    void Foo();
};
}   // namespace mynamespace
```

``` c++
// .cc文件
namespace mynamespace {
// 所有的定义都写在名字空间内
void MyClass::Foo() {
    ...
}
}   // namespace mynamespace
```

结构更复杂的`.cc`文件如下：
``` c++
#include "a.h"

ABSL_FLAG(bool, someflag, false, "dummy flag");

namespace mynamespace {

using ::foo::Bar;

... code for mynamespace...     // 左对齐
}   // namespace mynamespace
```
- 将生成协议消息码置于名字空间中，在`.proto`文件中使用包标识符，具体参照[协议缓冲包](#)
- 不要在`std`命名空间内声明任何实体，这种行为是未定义的
- 不要使用`using`指令引入整个名字空间内的标识符
``` c++
// 禁止的行为，会污染名字空间
using namespace foo;
```
- 因为任何在头文件中引入的名字空间都成为API的一部分，所以不要在头文件中使用名字空间别名，除非显式地标记了仅供内部名字空间使用。
``` c++
// 在.cc中使用别名缩短名字空间
namespace baz = ::foo::bar::baz;
```

``` c++
// 在.h中使用别名缩短名字空间
namespace librarian {
namespace impl {    // 仅限内部使用
namespace sidetable = ::pipeline_diagnostics:::sidetable;
}   // namespace impl
inline void my_inline_function() {
    // 仅限于在一个函数中使用的名字空间别名 
    namespace baz = ::foo::bar::baz;
}

}   // namespace impl
}   // namespace librarian
```
- 不用使用内联名字空间


## 命名

在代码中最应当遵循规则保持一致的莫过于各种命名了。好的命名风格可以顾名思义，而不至于向前翻找代码看声明部分。大脑的匹配机制很大程度上依赖于命名。

尽管可以随意命名，但当编写代码的时候，保持命名风格的一致比个人喜恶重要得多。众口难调，故无论你觉得这里列举的规则是否合理，规则就是规则。

### 通用命名规则

命名要具有良好的可读性，这样其他人读代码的时候也能清楚知道其含义。

使用名字描述对象的目的或意图时，尽量不使用缩写，不要担心过长，那远不如让人可以快速读懂代码重要。经验法则来说，出现在维基百科上的缩写可以使用。另外，命名的详细程度应与其作用范围正相关。例如，三五行的函数内，局部变量不必特意命名，简单的使用`n`即可。但如果是类的成员变量，命名为`n`会让人摸不着头脑。

还有一些普遍知晓的缩写也是可以使用的，典型的有循环控制变量命名为`i`，以及模版参数命名为`T`。

``` c++
// 良好的命名风格
class MyClass {
public:
    int CountFooErrors(const std::vector<Foo>& foos) {
        int n = 0;      // 代码逻辑足够简单，简短的命名也可以立刻读懂
        for (const auto& foo : foos) {
            ...
            ++n;
        }
        return n;
    }

    void DoSomeThingImportant() {
        std::string fqdn = ...;    // fqdn: Fully Qualified Domain Name，众所周知的缩写，可以使用
        //然而我并不知道，或许此处指fqdn截至目前不会引起歧义吧
    }

private:
    const int kMaxAllowedConnections = ...;   // 这个命名真的好，清晰明了，甚至有可能借此推断出fqdn
};

// 不良的命名风格
class MyClass {
public:
    int CountFooErrors(const std::vector<Foo>& foos) {
        int total_number_of_foo_errors = 0; // 此处命名冗长得多余
        for (int foo_index = 0; foo_index < foos.size(); ++foo_index) {  // 作用域极小的循环控制变量可以自然的命名为：i
            total_number_of_foo_errors++;
        }
        return total_number_of_foo_errors;
    }
};
```

对于接下来要列举的命名规则使用的例子，"word"表示任何不包含空格的英文，也包括缩写在内。当采用驼峰命名法等大小写混合的命名风格时，推荐直接缩写的第一个字母大写，比如，`StartRpc()`比`StartRPC()`要好。

模版参数要按照其分类对应不同的规则：类型模版参数应该遵照[类型命名规则](#类型命名)，非类型模版参数应该遵照[变量命名规则](#变量命名)。

### 文件命名

文件名采用英文小写加下划线或者破折号，如无特殊需要，优先加下划线的方式。

下面是可以接收的文件命名的例子：
- my_useful_class.cc
- my-useful-class.cc
- myusefulclass.cc
- myusefulclass_test.cc

C++文件应该以`.cc`为扩展名，头文件应当以`.h`为扩展名。依赖于某个特定位置被引入的文件应当以`.inc`为扩展名（详细请参照[自包含头文件](#)）。

不要使用在`/usr/include`路径下已经存在的文件，比如`db.h`。

通常来说，命名文件的时候尽量详细且明确，例如，`http_server_logs.h`优于`logs.h`。常用的做法是，当定义一个名为`FooBar`的类时，使用`foo_bar.h`和`foo_bar.cc`命名相应的代码文件。

### 类型命名

所有的类型：类、结构体、类型别名、枚举和类型模版参数遵循相同的命名习惯，以大写字母开头，并且后续每一个单词的首字母都大写，不使用下划线，如：`MyExcitingClasss`、`MyExcitingEnum`。

``` c++
// 类和结构体
class UrlTable { ... };
class UrlTabletester { ... };
struct UrlTableProperties { ... };

// 类型别名
typedef hash_map<UrlTableProperties *, std::string> PropertiesMap;
using PropertiesMap = hash_map<UrlTableProperties *, std::string>;

// 枚举
enum class UrlTableError { ... };
```

### 变量命名

变量、函数参数、成员变量命名都采用小写加下划线的方式。其中，类（不包括结构体）的成员变量在末尾多加一个下划线。如：`a_local_variable`、`a_struct_data_member`、`a_class_data_member_`。

#### 普通的变量

``` c++
std::string table_name;     // 推荐，小写字母加下划线。

std::string tableName;      // 不推荐
```

#### 类的成员变量

类的成员变量，不管是静态还是非静态，都像普通的变量一样命名，然后在末尾加一个下划线以示区分。

``` c++
class TableInfo {
    ...
private:
    std::string table_name_;        // 推荐，小写字母加下划线，并在末尾多加一个下划线
    static Pool<TableInfo>* pool_;  // 推荐，静态成员变量遵循相同的命名规则
};
```

#### 结构体的成员变量

结构体的成员变量，不管是静态还是非静态，都像普通变量一样命名，且不用像类成员变量一样在末尾多加一个下划线。

``` c++
struct UrlTableProperties {
    std::string name;
    int num_entries;
    static Pool<UrlTableProperties>* pool;
};
```

当你不确定使用类还是结构体的时候，请看[类和结构体的对比](#)。

### 常量命名

常量遵从每个单词的首字母大写，并在最前面加一个小写的k，当必须大写字母无法分隔时，使用下划线。

``` c++
const int kDaysInAWeek = 7;     // 每个单词首字母大小，并在前面加一个小写k
const int KAndroid8_0_0 = 24;   // 首字母大写不足以分隔数字 
```

所有静态存储周期变量（如静态变量和全局变量，详见[存储期](#)），都应当遵循此规则。

### 函数命名

一般函数采用大小写混合，每个单词首字母大写的方式。此规则同样适用于类内和名字空间内对外像API一样的常量，因为它们是对象还是函数是不重要的实现细节。

``` c++
AddTableEntry()
DeleteUrl()
OpenFileOrDie()
```

`set`和`get`函数应当像变量一样命名。

``` c++
int count();

void set_count(int count);
int get_count();
```

### 名字空间命名

名字空间采用小写加下划线的方式，最外层的名字空间应该基于项目名。多层嵌套时注意不要冲突，也不要和标准库内的名字空间冲突。

### 枚举命名

枚举只应该和常量的命名规则相同，即每个单词的首字母大写，并在开头加一个小写k。

``` c++
// 推荐，首字母大写，并在开头加一个小写k
enum class UrlTableError {
    kOk = 0,
    kOutOfMemory,
    kMalformedInput,
};

// 不推荐
enum class AlternateUrlTableError {
    OK = 0;
    OUT_OF_MEMORY = 1;
    MALFORMED_INPUT = 2;
};
```

### 宏命名

你真的要定义一个宏吗？如果是的话，它们应该长这样；`MY_MACRO_THAT_SCARES_SMALL_CHILDREN_AND_ADULIS_ALIKE`。

请先阅读[宏](#)，通常不建议使用宏。但如果你必须使用的话，命名时全部字母大写，并以下划线分隔单词。

``` c++
#define ROUND(x) ...
#define PI_ROUNDED 3.0
```

### 命名规则的特例

当你命名的实体类似C/C++中存在的实体，那么，你可以跟随它们的命名习惯。
- `bigopen()`
  函数名，跟随`open()`
- `uint`
  跟随`typedef`
- `bigpos`
  结构体或类，跟随`pos`
- `sparse_hash_map`
  像STL中的实体，跟随STL的命名风格
- `LONGLONG_MAX`
  常量，跟随`INT_MAX`
