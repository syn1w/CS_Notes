# 一、基础

注：我本地环境是 WSL Ubuntu 18.04  
安装 OCaml，参考[这里](https://ocaml.org/docs/install.html)  
```sh
sudo apt install ocaml    # Debian/Ubuntu
sudo dnf install ocaml    # Fedora
sudo yum install ocaml    # RH/CentOS
sudo zypper install ocaml # SuSE
```

之后输入 `ocaml` 命令，显示 `OCaml version 4.05.0`  

## 1. 尝试 OCaml 简单的表达式
使用交互式顶层环境(interactive toplevel)或REPL(Read-Eval-Print-Loop)。`ocaml` 命令提供了一个十分基础的顶层环境，最好使用 `utop` 交互环境。  
**安装 `utop`**

```sh
sudo apt install opam # OCaml 包管理工具
sudo apt install m4   # 安装依赖
opam init             # Initialize ~/.opam，需要等待一段时间
opam install utop     # no sudo, [WARNING] Running as root is not recommended
```

我本地环境是 WSL Ubuntu18.04，apt 默认安装的 opam 为 1.2.2 版本，使用下面的方法安装和初始化高版本的 opam(2.0.4, 2020.04)
```sh
sudo add-apt-repository ppa:avsm/ppa
sudo apt update
sudo apt install opam
opam init --disable-sandboxing
eval `opam env`
```

```ocaml
1+1;; (* `;;` 在交互模式下表示计算一个表达式，在独立的程序中不是必要的，不过会改善 OCaml 的错误报告 *)
(* output: - : int = 2 *)
```

## 2. Hello World

```ocaml
let () = print_endline "hello world!"
```

构建&运行
```sh
ocamlbuild hello.native
./hello.native
# hello world!
```
## 3. 基础类型
|   type   |                             desc                             |
| :------: | :----------------------------------------------------------: |
|  `int`   | 31-bit(with sign bit) signed int on 32-bit processors; 63-bit on 64-bit processors |
| `float`  |             IEEE double-precision floating point             |
|  `bool`  |                      `true` or `false`                       |
|  `char`  |                       8-bit character                        |
| `string` |                            string                            |
|  `unit`  |                         Written as ()                        |

OCaml 中的 `int` 中一位用来自动管理内存(GC)，因此 `int` 类型是 31 位，而非 32位。如果整型精度不够，需要使用大数模块(`Nat` and `Big_int`)，如果一定要处理 32 位类型(比如加密程序、网络协议栈)，有 `nativeint` 类型  
char 类型不支持 Unicode 或 UTF-8，需要使用 [comprehensive Unicode libraries](http://camomile.sourceforge.net/) 来处理  
字符串并非字符列表，有自己内部更高效的表示  
`unit` 类似于 C 中的 `void`，之后再详述

## 4. 调用函数
OCaml 调用函数参数没有括号，也没有逗号分隔多个参数，但是括号和逗号有其他的意思  
比如 `repeated` 函数，它的参数是一个字符串 s 和一个数 n，返回值是把 s 重复 n 遍形成的新字符串  
```ocaml
repeated "hello" 3
```
但是 `repeated("hello", 3)` 在 OCaml 中是合法的，意义是调用 `repeated` 函数，参数是一个含有两个元素的 `pair`
```ocaml
repeated (prompt_string "Name please: ") 3
```
`prompt_string` 是返回用户输入的字符串，一般情况下，规则是：“括号只括起整个函数调用，不要括起函数调用的参数。” 下面是更多的例子：
```ocaml
f 5 (g "hello") 3    (* f has three arguments, g has one argument *)
f (g 3 4)            (* f has one argument, g has two arguments *)
```

## 5. 函数定义
```ocaml
let average a b =
    (a +. b) /. 2.0;;
(* val average: float -> float -> float = <fun> *)
```

使用 `let` 创建一个函数变量 `average`，参数为 `a b`，类似于把值绑定到变量名上  

关于 OCaml 的一些特性：
- OCaml 是*强类型语言*
- OCaml 用*类型推导*(type inference)找出类型，所以无需注明类型
- OCaml 不做任何的隐式类型转换
- 由于 type inference 的副作用，OCaml 不允许任何形式的重载(包括运算符重载)。对于整数相加使用 `+`，两个浮点数相加使用 `+.`。其他运算符有 `-.`，`*.`，`/.` 也是这样
- First-class functions
- Parametric polymorphism(参数化多态)，类似与 C++ 中的 `template`

**递归函数**
OCaml 一般函数不允许递归，如果需要定义递归函数，需要 `rec` 关键字  
```ocaml
(* Fibonacci function *)
let rec fib n = if n < 2 then n else (fib (n-1)) + (fib (n-2));;
fib 3;; (* 3 *)
```

## 6. 显式类型转换
上面说到 OCaml 不做任何隐式类型转换，确实需要做类型转换使用显式类型转换  
以整数(`i`)加上一个浮点数(`f`)为例进行说明：
```ocaml
float_of_int i +. f;;
```

`float_of_int` 函数输入整型为参数，返回一个浮点数  
其他类似的函数 `int_of_float, char_of_int, int_of_char, string_of_int` 等等  
由于 `int` 转换为 `float` 是个特别常用的操作，`float_of_int` 有个简短的别名 `float`
```ocaml
float i +. f;;
```

## 7. 函数的类型
对于一个函数 `f`，其参数类型为 `arg1, arg2, ..., argn`, 返回类型是 `rettype`  
编译器会显示其函数类型为
```txt
f: arg1 -> arg2 -> ... argn -> rettype
```

如果一个函数没有返回值，把返回值写成 `unit` 类型，类似于 `void`  
```ocaml
output_char : out_channel -> char -> unit
```
如果一个函数的参数可以是任意类型，比如  
```ocaml
let return_three x = 3;
(* return_three : 'a -> int *)
```

`'a` 代表任意类型，之后会进行详述



# 附录A references

https://ocaml.org/learn/tutorials/index.zh.html  
《Real World OCaml》