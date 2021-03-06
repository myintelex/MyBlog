---
title: Google C++ Style Guide
date: 2016-12-02 15:57:09
categories: 学习记录
tags: [Google, C++, Code Style]
---

最近把 Google C++ Style Guide 从头到尾看了一遍，把相应的内容记录在这里，也规范下以后自己的代码。  

<!--more-->
![Google C++ Style Guide](http://ogf054qp1.bkt.clouddn.com/Google%20C++%20Style%20Guide.png)

## Header Files
通常每一个 `.cpp` 文件都应该有一个对应的 `.h` 头文件，对于一个头文件来说（1）需要**Self-contained**，即一个头文件应该有相应的**#define 保护**、并包含所有他需要的其它头文件；（2）尽量应使用头文件而不是**前置声明**；（3）不能滥用**内联函数**只在函数少于10行的时候才使用内联函数；（4）使用标准的**头文件包含顺序**来增强可读性，避免隐藏依赖。  
* #define保护的格式为 `#define <PROJECT>_<PATH>_<FILE>_H_`
* 头文件的包含顺序应该为：
  1. 优先位置，当前文件需要实现或测试的头文件
  2. C 系统文件
  3. C++ 系统文件
  4. 其他库的 .h 文件
  5. 本项目内 .h 文件

## Scoping
作用域主要规范了 `名称空间 `、`非成员函数`、`静态成员函数`、`全局函数`、`局部变量`、`静态和全局变量` 的使用规范：
* 名称空间
  * 推荐在 `.cpp` 文件中使用匿名名称空间
  * 不能在 `.h` 文件中使用匿名名称空间
  * 不要在名称空间 `std` 内声明任何东西
  * 不要使用 `using` 包含整个命名空间
  * 不要使用内联命名空间
  * 可以使用 `using` 声明
  * 可以使用名称空间别名
* 使用静态成员函数或名字空间内的非成员函数, 尽量不要用裸的全局函数。
* 将函数变量尽可能置于最小作用域内, 并在变量声明时进行初始化。
* 禁止使用 `class` 类型的静态或全局变量：它们会导致难以发现的 bug 和不确定的构造和析构函数调用顺序。不过 `constexpr` 变量除外，毕竟它们又不涉及动态初始化或析构。

## Classes
在使用类的时候应遵循以下规则：
* 不要在构造函数中进行复杂的初始化 (尤其是那些有可能失败或者需要调用虚函数的初始化)。
* 对单个参数的构造函数使用 C++ 关键字 `explicit`.不要定义隐式转换。
* 如果你的类型需要, 就让它们支持拷贝 / 移动. 否则, 就把隐式产生的拷贝和移动函数禁用。
* 仅当只有数据时使用 `struct`, 其它一概使用 `class`。
* 使用组合 (`composition`) 常常比使用继承更合理. 如果使用继承的话, 定义为 `public` 继承。
* 只在以下情况我们才允许多重继承: 最多只有一个基类是非抽象类; 其它基类都是以 `Interface` 为后缀的 纯接口类。
* 接口是指满足特定条件的类, 这些类以 `Interface` 为后缀 (不强制)。
* 除少数特定环境外，不要重载运算符。
* 将所有数据成员声明为 `private`, 并根据需要提供相应的存取函数，一般在头文件中把存取函数定义成内联函数。
* 在类中使用特定的声明顺序: `public` 在 `private` 之前, 成员函数在数据成员 (变量) 前;  
  每个区段内的声明通常按以下顺序:
  1. `typedefs` 和枚举
  2. 常量
  3. 构造函数
  4. 析构函数
  5. 成员函数, 含静态成员函数
  6. 数据成员, 含静态数据成员
  友元声明应该放在 `private` 区段. 如果用宏 `DISALLOW_COPY_AND_ASSIGN` 禁用拷贝和赋值, 应当将其置于 `private` 区段的末尾, 也即整个类声明的末尾.  

* `.cc` 文件中函数的定义应尽可能和声明顺序一致。

## Functions
在定义函数时，参数顺序应该是：**先输入，后输出**。所有按引用传递的参数必须加上 `const` 。倾向编写简短, 凝练的函数，**函数一般不应超过40行**。**重载函数**时最好在函数名里加入参数信息。除以下情况，最好不要使用缺省参数：
* 其一，位于 .cc 文件里的静态函数或匿名空间函数，毕竟都只能在局部文件里调用该函数了。
* 其二，可以在构造函数里用缺省参数，毕竟不可能取得它们的地址。
* 其三，可以用来模拟变长数组。

## Other C++ Features
Google设定的C++的新特性规范有很多，从中选取了以下几条：
* `Casting` 使用 C++ 的类型转换, 如 static_cast<>(). 不要使用 int y = (int)x 或 int y = int(x) 等转换方式;

* `Streams` 只在记录日志时使用流.

* `Preincrement and Predecrement` 对于迭代器和其他模板对象使用前缀形式 (`++i`) 的自增, 自减运算符.

* `Use of const` `const` 变量, 数据成员, 函数和参数为编译时类型检测增加了一层保障; 便于尽早发现错误。在任何可能的情况下使用 const:
  * 如果函数不会修改传你入的引用或指针类型参数, 该参数应声明为 const.
  * 尽可能将函数声明为 const. 访问函数应该总是 const. 其他不会修改任何数据成员, 未调用非 const 函数, 不会返回数据成员非 const 指针或引用的函数也应该声明成 const.
  * 如果数据成员在对象构造之后不再发生变化, 可将其定义为 const.

* `Use of constexpr` 在 C++11 里，用 `constexpr` 来定义真正的常量，或实现常量初始化。
 
* `Integer Types` C++ 内建整型中, 仅使用`int`。在需要确保整型大小时, 可以使用 `<stdint.h>` 中长度精确的整型。

* `Preprocessor Macros` 如果要使用宏, 尽可能遵守:
  * 不要在 .h 文件中定义宏.
  * 在马上要使用时才进行 #define, 使用后要立即 #undef.
  * 不要只是对已经存在的宏使用#undef，选择一个不会冲突的名称；
  * 不要试图使用展开后会导致 C++ 构造不稳定的宏, 不然也至少要附上文档说明其行为.
  * 不要用 ## 处理函数，类和变量的名字。


* `nullptr/NULL/0` 整数用 `0`, 实数用 `0.0`, 指针用 `nullptr` 或 `NUL`, 字符 (串) 用 `'\0'`.

* `sizeof` 尽可能用 sizeof(varname) 代替 sizeof(type).

* `auto` 用 `auto` 绕过烦琐的类型名，只要可读性好就继续用，别用在局部变量之外的地方。

## Naming
命名的一致性是规则代码最鲜明的体现，规则的命名可以快速的让阅读者获知名字代表的是函数还是类型还是常量，命名需要遵守的最基本的原则是`要有描述性；少用缩写`。对不同的类型，以不同的方式进行命名，规则如下：
* **文件名**要全部小写, 可以包含下划线 (`_`) 或连字符 (`-`)。`_` 更好：`my_useful_class.cc`。
* **类型名称**（类, 结构体, 类型定义 (typedef), 枚举）的每个单词首字母均大写, 不包含下划线: `MyExcitingClass`, `MyExcitingEnum`。
* **变量名**一律小写, 单词之间用下划线连接. 类的成员变量以下划线结尾, 但结构体的就不用，如: `a_local_variable`, `a_struct_data_member`, `a_class_data_member_`。
* **在全局或类里的常量**名称前加 `k`: `kDaysInAWeek`。且除去开头的 k 之外每个单词开头字母均大写。
* **常规函数**使用大小写混合, **取值和设值函数**则要求与变量名匹配: `MyExcitingFunction()`, `MyExcitingMethod()`, `my_exciting_member_variable()`, `set_my_exciting_member_variable()`。
* **名字空间**用小写字母命名, 并基于项目名称和目录结构: `google_awesome_project`。
* **枚举**的命名应当和**常量**或**宏**一致: `kEnumName` 或是 `ENUM_NAME`。
* 如果你一定要用**宏**, 像这样命名: `MY_MACRO_THAT_SCARES_SMALL_CHILDREN`。

## Comments
注释虽然写起来很痛苦, 但对保证代码可读性至关重要。但同时需要记住的是: 注释固然很重要, 但`最好的代码本身应该是自文档化`。有意义的类型名和变量名, 要远胜过要用注释解释的含糊不清的名字。注释主要有以下7类：

### 1. 文件注释   
在每一个文件开头加入版权公告, 然后是文件内容描述。每个文件都应该包含以下项, 依次是:

 * 版权声明 (比如, `Copyright 2008 Google Inc.`)
 * 许可证. 为项目选择合适的许可证版本 (比如, `Apache 2.0`, `BSD`, `LGPL`, `GPL`)
 * 作者: 标识文件的原始作者.
 * 文件内容:紧接着版权许可和作者信息之后, 每个文件都要用注释描述文件内容。
   通常, `.h` 文件要对所声明的类的功能和用法作简单说明. `.cc` 文件通常包含了更多的实现细节或算法技巧讨论。

### 2. 类注释  
每个类的定义都要附带一份注释, 描述类的功能和用法.

### 3. 函数注释  
函数声明处注释描述函数功能; 定义处描述函数实现.

 * 函数声明:  
    * 函数的输入输出.
    * 对类成员函数而言: 函数调用期间对象是否需要保持引用参数, 是否会释放这些参数.
    * 如果函数分配了空间, 需要由调用者释放.
    * 参数是否可以为 NULL.
    * 是否存在函数使用上的性能隐患.
    * 如果函数是可重入的, 其同步前提是什么?

 * 函数定义:  
  每个函数定义时要用注释说明函数功能和实现要点. 比如说说你用的编程技巧, 实现的大致步骤, 或解释如此实现的理由, 为什么前半部分要加锁而后半部分不需要.

### 4. 变量注释  
通常变量名本身足以很好说明变量用途. 某些情况下, 也需要额外的注释说明.
 * 类数据成员:每个类数据成员 (也叫实例变量或成员变量) 都应该用注释说明用途. 如果变量可以接受 NULL 或 -1 等警戒值, 须加以说明。
 * 全局变量:和数据成员一样, 所有全局变量也要注释说明含义及用途。

### 5. 实现注释
对于代码中巧妙的, 晦涩的, 有趣的, 重要的地方加以注释。
 * 比较隐晦的地方要在行尾加入注释. 在行尾空两格进行注释。
 * 如果你需要连续进行多行注释, 可以使之对齐获得更好的可读性。
 * 向函数传入 NULL, 布尔值或整数时, 要注释说明含义, 或使用常量让代码望文知意。
 * **永远不要**用自然语言翻译代码作为注释. 要假设读代码的人 C++ 水平比你高, 即便他/她可能不知道你的用意。

### 6. `TODO` 注释  
对那些临时的, 短期的解决方案, 或已经够好但仍不完美的代码使用 `TODO` 注释。
`TODO` 注释要使用全大写的字符串 `TODO`, 在随后的圆括号里写上你的大名, 邮件地址, 或其它身份标识. 冒号是可选的. 主要目的是让添加注释的人 (也是可以请求提供更多细节的人) 可根据规范的 `TODO` 格式进行查找. 添加 `TODO` 注释并不意味着你要自己来修正。

### 7. 弃用注释
通过弃用注释（`DEPRECATED comments`）以标记某接口点（`interface points`）已弃用。

## Formatting
代码风格和格式是一个比较个人化的问题, 但整个项目(或一个人的所有项目)服从统一的编程风格是很重要的, 只有这样才能让所有人能很轻松的阅读和理解代码。

### 1. 行长度
每一行代码字符数不超过 80.

### 2. 非 `ASCII` 字符
尽量不使用非 `ASCII` 字符, 使用时必须使用 `UTF-8` 编码.

### 3. 空格还是制表位
只使用空格, 每次缩进 2 个空格.

### 4. 函数声明与定义
返回类型和函数名在同一行, 参数也尽量放在同一行，如果放不下就对形参分行。
注意以下几点:

 * 如果返回类型和函数名在一行放不下，分行。
 * 如果返回类型那个与函数声明或定义分行了，不要缩进。
 * 左圆括号总是和函数名在同一行;
 * 函数名和左圆括号间没有空格;
 * 圆括号与参数间没有空格;
 * 左大括号总在最后一个参数同一行的末尾处;
 * 如果其它风格规则允许的话，右大括号总是单独位于函数最后一行，或者与左大括号同一行。
 * 右大括号和左大括号间总是有一个空格;
 * 函数声明和定义中的所有形参必须有命名且一致;
 * 所有形参应尽可能对齐;
 * 缺省缩进为 2 个空格;
 * 换行后的参数保持 4 个空格的缩进;

### 5. `Lambda` 表达式
其它函数怎么格式化形参和函数体，`Lambda` 表达式就怎么格式化；捕获列表同理。

### 6. 函数调用
要么一行写完函数调用，要么在圆括号里对参数分行，要么参数另起一行且缩进四格。如果没有其它顾虑的话，尽可能精简行数，比如把多个参数适当地放在同一行里。函数调用遵循如下形式：

 * 如果同一行放不下，可断为多行，后面每一行都和第一个实参对齐，左圆括号后和右圆括号前不要留空格：
 * 参数也可以放在次行，缩进四格：
 * 把多个参数放在同一行，是为了减少函数调用所需的行数，除非影响到可读性。有人认为把每个参数都独立成行，不仅更好读，而且方便编辑参数。不过，比起所谓的参数编辑，我们更看重可读性，且后者比较好办：
 * 如果一些参数本身就是略复杂的表达式，且降低了可读性。那么可以直接创建临时变量描述该表达式，并传递给函数,或者放着不管，补充上注释：
 * 如果某参数独立成行，对可读性更有帮助的话，就这么办。
 * 此外，如果一系列参数本身就有一定的结构，可以酌情地按其结构来决定参数格式：

### 7. 列表初始化格式
您平时怎么格式化函数调用，就怎么格式化。

### 8. 条件语句
倾向于不在圆括号内使用空格. 关键字 `if` 和 `else` 另起一行.
对基本条件语句有两种可以接受的格式. 一种在圆括号和条件之间有空格, 另一种没有。只要其中一个分支用了大括号，所有分支都要用上大括号。
````C++
if (condition) {  圆括号里没空格紧邻。
  ...  // 2 空格缩进。
} else {  // else 与 if 的右括号同一行。
  ...
}````

### 9. 循环和开关选择语句
`switch` 语句可以使用大括号分段，以表明 `cases` 之间不是连在一起的。在单语句循环里，括号可用可不用。空循环体应使用 `{}` 或 `continue`.
````C++
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
}````
在单语句循环里，括号可用可不用：
````C++
for (int i = 0; i < kSomeNumber; ++i)
  printf("I love you\n");

for (int i = 0; i < kSomeNumber; ++i) {
  printf("I take it back\n");
}````
空循环体应使用 `{}` 或 `continue`, 而不是一个简单的分号.
````C++
while (condition) {
// 反复循环直到条件失效。
}
for (int i = 0; i < kSomeNumber; ++i) {}  // 可 - 空循环体。
while (condition) continue;  // 可 - contunue 表明没有逻辑。
````
### 10. 指针和引用表达式
句点或箭头前后不要有空格. 指针/地址操作符 (`*`, `&`) 之后不能有空格.
下面是指针和引用表达式的正确使用范例:
````C++
x = *p;
p = &x;
x = r.y;
x = r->y;
````
### 11. 布尔表达式
如果一个布尔表达式超过 标准行宽, 断行方式要统一一下.
下例中, 逻辑与 (`&&`) 操作符总位于行尾:
````C++
if (this_one_thing > this_other_thing &&
  a_third_thing == a_fourth_thing &&
  yet_another & last_one) {
...
}````
    
### 12. 函数返回值
`return` 表达式里时没必要都用圆括号，变量返回时不要使用圆括号，可以用圆括号把复杂表达式圈起来，改善可读性。

### 13. 变量及数组初始化
用 `=`, `()` 和 `{}` 均可。请务必小心列表初始化 `{...}`。 

### 14. 预处理指令
预处理指令不要缩进, 从行首开始。即使预处理指令位于缩进代码块中, 指令也应从行首开始.
````C++
// 可 - directives at beginning of line
if (lopsided_score) {
#if DISASTER_PENDING      // 正确 -- 行开头起。
  DropEverything();
#endif
  BackToNormal();
}````

### 15. 类格式
访问控制块的声明依次序是 `public:`, `protected:`, `private:`, 每次缩进 1 个空格.
类声明 (对类注释不了解的话, 参考 类注释) 的基本格式如下:
````C++
class MyClass : public OtherClass {
 public:      // 注意有 1 空格缩进!
  MyClass();  // 照常，2 空格缩进。
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
  DISALLOW_COPY_AND_ASSIGN(MyClass);
};````
注意事项:

 * 所有基类名应在 80 列限制下尽量与子类名放在同一行.
 * 关键词 public:, protected:, private: 要缩进 1 个空格.
 * 除第一个关键词 (一般是 public) 外, 其他关键词前要空一行. 如果类比较小的话也可以不空.
 * 这些关键词后不要保留空行.
 * public 放在最前面, 然后是 protected, 最后是 private.

### 16. 构造函数初始值列表
构造函数初始值列表放在同一行或按四格缩进并排几行.

### 17. 名字空间格式化
名字空间 不要增加额外的缩进层次,

### 18. 水平留白
水平留白的使用因地制宜. 永远不要在行尾添加没意义的留白.
 * 常规
     * 左大括号前恒有空格。
     * 分号前不加空格。
     * 大括号内部可与空格紧邻也不可，不过两边都要加上。
     * 继承与初始化列表中的冒号前后恒有空格。
     * 内联函数实现，在大括号内部加上空格并编写实现。
     * 大括号里面是空的话，不加空格。
 * 循环和条件语句
     * if 条件语句和循环语句关键字后均有空格。
     * 圆括号内部不紧邻空格。
     * 循环和条件语句的圆括号里可以与空格紧邻。
     * 循环里内 `;` 后恒有空格，`；` 前可以加个空格。
     * `switch` `case` 的冒号前无空格。
 * 操作符:
     *  赋值操作系统前后恒有空格。
     *  其它二元操作符也前后恒有空格，不过对 factors 前后不加空格也可以。
     *  圆括号内部不紧邻空格。
     *  在参数和一元操作符之间不加空格。
 * 模板和转换:
     *  尖叫括号(< and >) 不与空格紧邻，< 前没有空格，>( 之间也没有。
     *  在类型与指针操作符之间留空格也可以，但要保持一致。
     *  您或许可以在 < < 里加上一对对称的空格。

### 19. 垂直留白
垂直留白越少越好.
这不仅仅是规则而是原则问题了: 不在万不得已, 不要使用空行. 
 * 两个函数定义之间的空行不要超过 2 行
 * 函数体首尾不要留空行, 函数体中也不要随意添加空行.
 * 函数体内开头或结尾的空行可读性微乎其微。
 * 在多重 if-else 块里加空行或许有点可读性。
