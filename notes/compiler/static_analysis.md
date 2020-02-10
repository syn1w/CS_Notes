《Secure Programming with Static Analysis》



# 一、Software Security

术语静态分析(*static analysis*)是指在不执行代码的情况下评估代码的任何过程。

## 1.   defensive programming

防御式编程(*defensive programming*) 越来越具有安全含义，之前仅仅指因为思维方式的编码实践导致不可避免的错误，还有在程序内部迟早发生错误和导致未预期的异常。

防御式编程不会保证软件的安全性。

> 防御式编程与非防御式编程之间的区别在于，程序员不会对特定的函数调用或库的使用情况做假设。( [https://zh.wikipedia.org/wiki/%E9%98%B2%E5%BE%A1%E6%80%A7%E7%BC%96%E7%A8%8B](https://zh.wikipedia.org/wiki/防御性编程) )

考虑下面的代码：

```c
void printMsg(FILE* file, char* msg) {
    fprintf(file, msg);
}
```

如果参数是 null 的话程序将崩溃。

改进一下：

```c
void printMsg(FILE* file, char* msg) {
	if (file == NULL) {
		logError("attempt to print message to null file");
	} else if (msg == NULL) {
		logError("attempt to print null message");
	} else {
		fprintf(file, msg);
	}
}
```

从安全方面考虑，这些检查远远不够，虽然阻止了 null 值情况下崩溃的问题，但 `msg` 参数自身就可能是恶意的。

可能会进行格式化字符串攻击，具体见缓冲区溢出(*buffer overflow*)。

安全考虑的改进：

```c
void printMsg(FILE* file, char* msg) {
	if (file == NULL) {
		logError("attempt to print message to null file");
	} else if (msg == NULL) {
		logError("attempt to print null message");
	} else {
		fprintf(file, "%.128s", msg);
	}
}
```

在考虑一段代码出错的范围时，程序员倾向于坚持自己的经验，这导致代码可以很好地防御程序员所熟悉的问题类型，但是对于攻击者仍然很容易攻击。



## 2. Quality Fallacy

渗透测试(*penetration test*)是有经验与知识，为雇主的网络设备、主机、模拟黑客的手法进行**攻击测试**，为了发掘系统漏洞，并提出改善方法，通常出于善意。

黑盒测试(*black-box test*)不会向攻击者提供有关系统构建方式的信息。也许类似现实场景，但是既不充分又没有效率。测试在系统构建完成才能开始。

模糊测试(*fuzzing*)，提供给程序随机生成的输入。



## 3. Classifying Vulnerabilities

将缺陷分为两组：通用(*generic defeat*)的和特定上下文的(*context-specific defects*)。

通用缺陷能够发生在几乎任何被某种给定的编程语言的程序中。buffer overflow 是一个 C/C++ 程序极好的例子。buffer overflow 代表的安全问题几乎能发生在任何的上下文中。

特定上下文缺陷需要关于程序语义特定的知识。

Common Weakness Enumeration (CWE)是一个对软件脆弱性和易受攻击性的一个分类系统。

开放式 Web 应用程序安全项目(OWASP)使用基于社区的方法来定义在安全原则、威胁、攻击、漏洞和对策之间的术语和关系。

7种缺陷分类：

- Input Validation and Representation: encodings, XSS, SQL injection, etc.
- API Abuse
- Security Features: authentication, access control, confidentiality, cryptography, and privilege management.
- Time and State: unexpected interactions between threads, processes, time, and data
- Error Handling
- Code Quality
- 封装(Encapsulation)

还有一个是源代码外部因素，比如环境(Environment)。



# 二、Intro

不执行代码对源代码进行分析的工具都进行静态检测。



静态分析适用于安全问题的原因：

- 静态分析工具进行彻底地和一致地检测，没有程序员认为哪段代码是有趣的这种偏见。
- 通过测试代码本身，静态分析工具通常可以指出导致安全问题的根本原因，而不是现象之一。
- 静态分析能够在开发初期找到错误，甚至在程序在第一次运行之前。尽早地发现错误能够引导程序员的工作。
- 当安全研究员发现了新的攻击类型时，静态分析工具能够很容易对大规模代码进行重新检测。

静态分析工具最普遍的缺点是会产生过多误报(*false positives*, also *false alarms*)，大量的误报可能会导致忽略重要的结果。

误报肯定是不希望的，但是对于安全问题来说，漏报(*false negatives*)是更糟糕的。误报浪费了排除的时间。如果是漏报，程序中存在问题，但是没有报出问题，需要付出与代码中存在漏洞相关的代价，而且会产生一种错误的安全感。

代码质量工具产生更少的误报并更愿意接受漏报。安全工具则相反。

静态分析工具的使用比人们意识到的要广泛，静态分析工具的种类有很多，每种工具都有不同的目标，比如：

- 类型检查(type checking)
- 风格检查(style checking)
- 程序理解(program understanding)
- 程序验证(program verification)
- 属性检查(property checking)
- 寻找缺陷(bug finding)
- 安全审查(security review)



静态分析工具面临的挑战：

- 理解程序(构建精确的程序模型)
- 在准确性、深度和扩展性之间权衡
- 寻找正确缺陷集合
- 呈现出易于理解的结果和错误
- 集成在构建系统或者 IDE 中



# 三、Code Review

Static analysis as a part of the code review

## 1. Performing

出于多种原因，进行注重安全的 code review

- 寻找一些可利用的漏洞，来证明额外的安全投资是合理的；
- 多于每一个没有考虑安全的大型项目，团队不得不对代码进行初步检查并做安全性改进；
- 在每一个 release 周期，项目最少进行一次 security review



review cycle

- 建立目标
- 运行静态检测工具
- review 代码
- 进行修复



## 2. Metrics

静态测试的指标：误报率和漏报率(人工测试足够样本进行估算)。

一个更深的问题是无法估计一组漏洞带来的风险。



重点指标：

- 计算漏洞密度
- 按照严重程度比较项目
- 按照类别细分结果
- 监测趋势



# 四、Internal

静态分析工具的内部工作原理，包括数据结构、分析技术、规则、报告bug的方法。

以安全功能为目标的静态分析工具大概是相同的方法。

```txt

 Source Code --> Build Model --> Perform Analysis --> Present results
                                         ^
                                         |
             Security Knowledge ---------|
```



## 1. Build Model

静态分析的第一步需要把被分析的源代码转化为程序模型(*program model*)，表示代码的数据结构。

静态分析工具一般借用编译工具的部分。许多的静态分析技术被用在编译器和编译器优化的问题。

接下来是编译器和静态分析工具的技术和数据结构

**词法分析(Lexical Analysis)**

把源代码转化为一系列的 *tokens*。



**语法分析(Parsing)**

parser 使用 *context-free grammar*(CFG) 匹配 token 流。形成了解析树(*parse tree*)。



**抽象语法树(Abstract Syntax Tree)**

在 parse tree 上进行分析是可行的，因为它包含了最直接的源码的表示。

不过在 parse tree 上进行大量分析是不方便的，因为树中的节点直接从语法产生式规则中派生而来，这些规则会引入非终结符，它们存在只是为了使解析变得容易且明确。

抽象语法树(*Abstract Syntax Tree*, AST) 为之后的分析提供了标准化的版本。



**语义分析(Semantic Analysis)**

在构建 AST 时，该工具会构建一个符号表(*symbol table*)，符号表将标识符与其类型、指向声明或定义的指针相关联。借助 AST 和符号表，现在被编译器用来做类型检查。

在编译器的世界中，符号解析和类型检查被称为语义分析，使用这些数据结构而不是静态分析工具具有明显的优势。

在语义检查之后，编译器和更高级的静态分析工具分道扬镳。编译器使用 AST、符号表、类型信息生成中间表示(*intermediate representation*, IR)，是一种适用于优化的机器代码的通用版本，然后转换为特定平台的目标代码。



**跟踪控制流(Tracking Control Flow)**

许多静态分析算法(编译器优化技术)探索执行函数时可能发生不同的执行路径。

为了使这些算法高效，大多数的工具构建控制流图(*control flow graph*)在 AST 或 IR 上。

控制流图的节点是基本块(*basic blocks*)，从第一条指令到最后一条指令的指令序列。控制流图中的边是有向的，表示基本块之间的控制流路径

比如：

```c
if (a > b) {
  nConsec = 0;
} else {
  s1 = getHexChar(1);
}
return nConsec;
```

```txt

                     +------------------bb0
                     |    if (a > b)     |
                     +-------------------+
                              |
          ---------------------------------------------
          ↓                                           ↓
  +----------------bb1                    +---------------------bb2
  |   nConsec = 0;  |                     | s1 = getHexChar(1);  |
  + ----------------+                     +----------------------+
          |                                           |
          ---------------------------------------------
                              |
                              ↓
                     +----------------bb3
                     | return nConsec; |
                     +-----------------+
```

*trace* 是一系列基本块，定义了通过代码的路径，上图中有两条 trace，分别是 `[bb0, bb1, bb3]` 和 `[bb0, bb2, bb3]`

调用图(*call graph*) 代表了在函数或方法之间潜在的控制流。

当函数指针或虚方法被调用时，需要结合数据流分析(dataflow analysis)和数据类型分析来限制可能调用的函数集。如果代码在运行时动态加载，没有办法保证控制流图是完整的



**跟踪数据流(Tracking Dataflow)**

数据流分析算法检查数据在程序中的移动方式。编译器执行数据流分析分配寄存器，移除 dead code。

数据流分析通常涉及到遍历函数的控制流图，并注明数据在哪儿被产生和在哪儿被使用。

把函数转化为 静态单赋值形式(*Static Single Assignment form*, SSA form)。SSA 是 IR 的特性，每个变量仅被赋值一次。比如在代码中

```c
y = 1;
y = 2;
x = y;
```

在 SSA 中会变成

```c
y1 = 1;
y2 = 2;
x1 = y2;
```

编译器或静态分析可以做到以下的改进：

- 常量传播(折叠)(*constant propagation*)
- 值域传播(*value range propagation*)
- 消除无用代码(*remove dead code*)
- 寄存器分配(register allocation)
- ...

如果一个变量在不同控制流路径上被赋了不同的值，则必须在控制流路径合并处协调该变量。

SSA 通过引入该变量的新版本并为新版本的变量分配两条控制流路径之一的值来完成合并，此合并点的符号速记为 *ф 函数*。ф 函数代表选择适当的值，取决于被执行的控制流图。比如：

源代码：

```c
if (bytesRead < 8) {
    tail = (byte)bytesRead;
}
```

SSA form:

```c
if (bytesRead1 < 8) {
    tail2 = (byte) bytesRead;
}
tail3 = ф(tail1, tail2);
```



**污点传播(Taint Propagation)**

安全工具需要知道攻击者能够可以控制程序的哪个值。使用数据流图来确定攻击者可以控制什么，被称为控制什么，称为污点传播(*Taint Propagation*)。需要知道信息在哪儿进入程序和如何在程序中移动。污点传播是需要输入验证(*input validation*) 和 表示缺陷(representation defects)的关键。



**指针别名(Pointer Aliasing)**

指针别名(*Pointer Aliasing*)是另一个数据流问题。别名分析的目的是了解哪些指针可能引用相同的内存位置，使用 “一定别名(must alias)”，“可能别名(may alias)”，“不可能别名(cannot alias)” 等术语描述指针关系。

许多编译器优化需要某种形式的别名分析来保证正确性。

对于安全工具来说，别名分析对于检测污点传播是重要的。流敏感的污点跟踪算法需要执行别名分析来理解获取用户输入和处理用户输入的数据流。



## 2. Analysis Algorithm

使用高级静态分析算法的动机是为了提升上下文敏感性，即确定特定代码段运行的环境和条件。更好的上下文敏感性可以更好地评估代码表现出的危险。

任何高级分析策略包含两个主要的部分：过程内分析(*intraprocedural analysis*)是为了独立函数的分析；过程间分析(*interprocedural analysis*)是为了函数间的分析，因为名字比较相似，也可以使用 *local analysis* 作为过程内分析(使用 CFG)，*global analysis* 作为过程间分析(使用 Call Graph)。

**断言检查**

可以将一些安全属性被声明为断言，比如下面的缓冲区溢出的例子：

```c
strcpy(dest, src);
```

添加以下的断言：

```c
assert(alloc_size(dest) > strlen(src));
```

如果程序的逻辑保证断言总是成功的，则 buffer overflow 是不可能发生的。如果有一组条件，断言可能会失败，则 analyzer 应该报告潜在的 buffer overflow。这种基于断言的方法适用于 SQL 注入，XSS 和其他漏洞。

本节主要是如何进行断言检查。

因为安全属性引起的三种断言：

- 程序员在不应该信任输入的情况下信任了它们。比如 SQL 注入、XSS 等。
- 可利用的缓冲区溢出导致与污点传播类似。
- 在某种情况下，在程序执行时，不太关注特定数值，而是关注对象的状态。变量在代码的每个点上具有不同的类型，比如一片内存区域处于分配状态还是释放状态。可能会引起内存泄露或多次释放。这样的安全特性可以表示为小型有限状态自动机。