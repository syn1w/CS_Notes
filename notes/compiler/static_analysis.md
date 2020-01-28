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



## 3. Classifying   Vulnerabilities

