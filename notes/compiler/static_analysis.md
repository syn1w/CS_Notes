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



