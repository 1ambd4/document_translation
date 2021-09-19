## 内容清单

- C++版本
- 头文件
- 作用域
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
int get_count(int count);
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
