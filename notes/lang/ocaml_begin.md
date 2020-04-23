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



## 4. 显式类型转换

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

Note: `float_of_int/float` 函数相当于 Pervasives 库提供的 `Float.of_int`  


# 二、程序结构

## 1. “变量”(表达式)

```ocaml
let average a b =
	let sum = a +. b in
	sum /. 2.0;;
```

`let name = expression in` 用来定义一个命名的**局部表达式**。之后 `name` 就可以代替 `expression`，直到一个 `;;` 结束本代码块。  

`sum` 只是 `a +. b` 的别名，所以无法给 `sum` 赋值。  

并不是每次引用 `sum`，`a +. b` 就会被求值一次，`sum` 只是一个局部绑定(binding)。绑定是不可变的，不是变量，不能被改变，只能被隐藏。  

也可以像定义局部表达式那样在全局区域定义全局表达式  



**let 绑定**：任何 `let ...`，无论在全局 (top level) 还是在函数中，经常被称作是 `let binding`   



## 2. 引用(真正的变量)

```ocaml
ref 0;;
(* - : int ref = {contents = 0} *)
```

`ref 0` 创造了一个引用，不过因为没有命名会被GC回收。  

```ocaml
let x = ref 0;; (* decl *)
(* val x : int ref = {contents = 0} *)
x := 42;;  (* assign *)
!x;;       (* dereference *)
```



## 3. 嵌套函数

嵌套函数可以使用包含它的函数当前可见的所有变量。这就是 "lexical scoping（词法变量域）"。  

一个嵌套函数的例子：  

```ocaml
let read_whole_channel chan =
  let buf = Buffer.create 4096 in
  let rec loop () =
    let newline = input_line chan in
    Buffer.add_string buf newline;
    Buffer.add_char buf '\n';
    loop ()
  in
  try
    loop ()
  with
    End_of_file -> Buffer.contents buf;;
```

`let` 和 `in` 结合使用时，可以在任意局部作用域引入一个新的绑定，`in` 标志这个作用域的开始，到 `;;` 作用域结束。



## 4. 模块和 `open`

OCaml 带有很多模块(代码的库)。例如画图、GUI部件交互、处理大数、数据结构、POSIX 系统调用等模块。  

这些库位于 `/usr/lib/ocaml/` 目录下(Unix 环境下)，比如 `Graphics` 库。    

模块 `Graphics` 有 5 个文件，扩展名分别是 `a, cma, cmi, cmxa, mli`    

只关注 `graphics.mli` 文件，因为其他都不是文本文件。主要文件名开头是小写字母，OCaml 文件名第一个字母大写作为模块名。    

使用 `Graphics` 中的函数，有两种方法。一种是 `open Graphics;;`，另一种在函数调用前加模块前缀，比如 `Graphics.open_graph`。`open` 像 Java 中的 `import` 更像 Perl 中的 `use` 或 Python 中的 `from Foo import *`。  

`Pervasives` 无需使用 `open`，所有的符号被自动导入到 OCaml 程序中。  

**重命名符号**  

如果模块名太长或想换个名字，可以使用 `module Gr = Graphics;;`。实际在引入一个嵌套模块。  



## 5. `;` 和 `;;`

`;;` 表示语句的结束，`;` 被称为连接点(sequence point)，和 C/C++ 中一样的用途。

使用 `;` 和 `;;` 的规则：

- 规则1： 使用 `;;` 在最顶端分隔不同的语句。并且绝对不要再函数定义中或其他语句中使用。
- 规则2：有时候可以省略 `;;`，总结起来就是 OCaml 能猜出这是语句结尾的地方。比如 `let, open, type` 之前，文件的最后，还有其他非常少能够猜到这是语句结尾的地方。
- 规则3：把 `let ... in` 看成一条语句，永远不要在它后面加上 `;`
- 规则4：在所有代码块中其他的语句后面跟上一个单独的 `;`

关于 `;` 的补充：  

`;` 也是一个运算符，`;` 的类型是 `int -> 'b -> 'b`，接受两个值并返回第二个，类似于 C/C++ 中的 `,` 运算符。  

下面的代码完全合法都在做同一件事情：  

```ocaml
let f x b y = if b then x+y else x+0
let f x b y = x + (if b then y else 0)
let f x b y = x + (match b with true -> y | false -> 0)
let f x b y = x + (let g z = function true -> z | false -> 0 in g y b)
let f x b y = x + (let _ = y + 3 in (); if b then y else 0)
```

OCaml 表达式的定义比 C 更广一些，C 有 statements 的概念，但是 C 的 statements 只是 OCaml 中的表达式。  

`;` 不同于 `+` 的一个地方是不能像函数一样引用。比如：

```ocaml
let sum_list = List.fold_left ( + ) 0
```

## 6. 命令式编程

OCaml 支持函数式编程和命令式编程，OCaml 中默认为函数式代码，变量绑定和大多数数据结构都是不可变的，不过 OCaml 对命令式的编程也提供了很好的支持。  

包括可变的数据结构(比如数组和散列表)以及控制流语句(`for/while` 等)  

**(1) 数组(array)**

```ocaml
let numbers = [|1; 2; 3; 4|];;
val numbers : int array = [|1; 2; 3; 4|]
numbers.(2) <- 4;;
(* unit = () *)
numbers;;
(* - : int array = [1; 2; 4; 4] *)
```

`.(i)` 为数组索引，从 0 开始，`<-` 语法表示修改。



**(2) 可变记录字段**

```ocaml
type running_sum = {
  mutable sum : float;
  mutable sum_sq : float; (* sum of squares *)
  mutable samples : int;
};;
```

```ocaml
(* create and update running_sum *)
let create () = { sum = 0.; sum_sq = 0.; samples = 0 }
let update rsum x =
  rsum.samples <- rsum.samples + 1;
  rsum.sum     <- rsum.sum +. x;
  rsum.sum_sq  <- rsum.sum_sq +. x *. x;;

(* use *)
let rsum = create();;
List.iter [1.;3.;2.;-7.;4.;5.] ~f:(fun x -> update rsum x);;
(* - : unit = () *)
let mean rsum = rsum.sum /. float rsum.samples
let stdev rsum = sqrt(rsum.sum_sq /. float float rsum.samples
                   -. (rsum.sum /. float rsum.samples) ** 2.);;
mean rsum;;
(* - : float = 1.33333333333333326 *)
```



**(3) `ref`**

可以使用 `ref` 创建单个可变值，`ref` 类型是标准库中预定义的类型，实际上它是一个包含一个可变字段 `contents` 的记录类型。

```ocaml
let x = { contents = 0 };
x.contents <- x.contents + 1;;
(* - : unit = () *)
x;;
(* - : int ref = { contents = 1 } *)
```

`ref` 的实现：

```ocaml
type 'a ref = { mutable contents : 'a }
let ref x = { contents = x }
let (!) r = r.contents
let (:=) r x = r.contents <- x
```

例子

```ocaml
let sum list =
  let sum = ref 0 in
  List iter list ~f:(fun x -> sum := !sum + x);
  !sum;;
```



**(4) `for/while`**

```ocaml
let permute arr =
  let len = Array.length arr in
  for i = 0 to len - 2 do
    let j = i + Random.int (len - i) in
    let tmp = arr.(i) in
    arr.(i) <- arr.(j);
    arr.(j) <- tmp
  done;;

open Core;;
let arr = Array.init 20 ~f:(fun i -> i);;
(* val arr : int array = [|0, 1, ..., 19|] *)
permute arr;;
```


## 7. 其他

`?foo` 表示可选参数。`~foo` 表示命名参数(标签参数)。`foo#bar` 调用对象 `foo` 的 `bar` 方法，类似于 `foo->bar`  



# 三、函数和模式匹配

## 1. 函数基础

**(1) 调用函数**

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

**(2) 函数定义**

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



**(3) 函数的类型**

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



## 2. 模式匹配基础

**模式匹配**一些简单的例子，为之后的内容做铺垫  

```ocaml
let (ints, strs) = List.unzip [(1, "one"); (2, "two"); (3, "three")];;
(* (ints, strs) is a pattern *)
val ints : int list = [1; 2; 3]
val strs : string list = ["one"; "two"; "three"]
```

```ocaml
let distance (x1, y1) (x2, y2) =
  sqrt((x1 -. x2) ** 2. +. (y1 -. y2) ** 2.);;
val distance : float * float -> float * float -> float = <fun>
```

```ocaml
let my_favorite_language languages =
  match languages with
  | first :: the_rest -> fist
  | [] -> "OCaml";;
(* function call *)
my_favorite_language ["English";"Spanish";"French"];;
(* - : string = "English" *)
my_favorite_language [];;
(* - : string = "OCaml" *)
```

```ocaml
let rec sum arr =
  match arr with
  | [] -> 0
  | head :: tail -> head + sum tail;;
val sum : int list -> int = <fun>
sum [1;2;3];;
(* - : int = 6*)
```



## 3. 匿名函数

```ocaml
(fun x -> x + 1);;
(* - : int -> int = <fun> *)

(fun x -> x + 1) 7;;
(* - : int = 8*)

(* as argument *)
List.map ~f:(fun x -> x + 1) [1;2;3];;
(* - : int list = [2; 3; 4] *)

(* put into data structure *)
let increments = [ (fun x -> x + 1); (fun x -> x + 2)];;
val increments : (int -> int) list = [<fun>; <fun>]
List.map ~f:(fun g -> g 5) increments;;
(* - : int list = [6; 7] *)
```

把两个函数放到数组中，`fun g -> g 5` 是一个函数，这个函数分别用数组中的每个函数作为参数。



## 4. 多参数函数

```ocaml
let average a b =
    (a +. b) /. 2.0;;
(* val average: float -> float -> float = <fun> *)
```

**柯里(curried)函数**

上面的 `average` 函数可以改写成下面的形式  

```ocaml
let average =
  (fun a -> (fun b -> (a +. b) /. 2.0));;
(* val average: float -> float -> float = <fun> *)
```

`average` 是一个单参数的函数，该函数返回一个单参数的函数，内层嵌套函数返回最终结果，这种函数被称为柯里函数(命名源自于 Haskell Curry)，`->` 具有右结合性。



## 4. 递归函数

OCaml 一般函数不允许递归，如果需要定义递归函数，需要 `rec` 关键字  

```ocaml
(* Fibonacci function *)
let rec fib n = if n < 2 then n else (fib (n-1)) + (fib (n-2));;
fib 3;; (* 3 *)
```

```ocaml
let rec find_first_stutter list =
  match list with
  | [] | [_] -> None
  | x :: y :: tail ->
    if x = y then Some x else find_first_stutter (y::tail)
```

`[] | [_]` 称为或模式，两个模式的析取，`[]` 匹配空列表，`[_]` 匹配单元素列表。  

通过 `and` 关键字使用 `let rec` 可以定义多个互相递归的函数。

```ocaml
let rec is_even x =
  if x = 0 then true else is_odd (x - 1)
and is_odd x =
  if x = 0 then false else is_even (x - 1)
```



## 5. 操作符

**前缀和中缀函数**  

```ocaml
Int.max 3 4 (* prefix *)
3 + 4       (* infix *)
```

不过操作符本质上是普通的函数，只是语法上有点区别。  如果中缀操作符两边加上 `()` 就可以当作常规的前缀函数使用了。  

```ocaml
(+) 3 4;;
List.map ~f:((+) 3) [7;8;9];;
```

如果一个函数的函数名(字符串)是从一个特殊的集合中选择的，这个函数在语法上被作为操作符来处理  

```ocaml
! $ % & * + - . / : < = > ? @ ^ | ~
or mod lsl lsr asr land lor lxor
```

比如  

```ocaml
let (+!) (x1, y1) (x2, y2) = (x1 + x2, y1 + y2);;
(3, 2) +! (-2, 4);;
(* - : int * int = (1, 6) *)
```

处理包含 `*`  需要注意，可能被解释为注释。  

**优先级**  

注：`!...` 是以 `!` 开头的操作符

|              操作符前缀              | 结合性 |
| :----------------------------------: | :----: |
|          `!..., ?..., ~...`          |  前缀  |
|              `., (, [`               |        |
|    函数应用，`ctor, assert, lazy`    | 左结合 |
|            `-, -.`(一元)             |  前缀  |
|  `**...`(浮点数幂), `lsl, lsr, asr`  | 左结合 |
|          `+..., -...`(二元)          | 左结合 |
|                 `::`                 | 右结合 |
|       `@...`(列表连接), `^...`       | 右结合 |
| `=..., <..., >..., |..., &..., $...` | 左结合 |
|               `&, &&`                | 右结合 |
|               `or, ||`               | 右结合 |
|                 `,`                  |        |
|               `<-, :=`               | 右结合 |
|                 `if`                 |        |
|                 `;`                  | 右结合 |

`-` 表示负号和减法，负号优先级要小于函数应用，如果说想要传入负值作为参数，需要用括号包围负值  

```ocaml
Int.max 3 -4 
(* Error: This expression has type int -> int
       but an expression was expected of type int *)
Int.max 3 (-4)
(* - : int = 3*)
```



下面是标准库中一些有用的例子：  

`List.dedup` 为删除列表中的重复元素。  由于 `List.dedup` 有 deprecated warning，所以替换为 `dedup_and_sort`

```ocaml
let (|>) x f = f x;; (* 类似于 Unix 中的管道操作 *)
let path = "/usr/bin:/usr/local/bin:/bin:/sbin";;
String.split ~on: ':' path
  |> List.dedup_and_sort ~compare: String.compare 
  |> List.iter ~f: print_endline;;
```





# 四、模块

## 1. 基本用法

```ocaml
(* amodule.ml *)
let hello () = print_endline "hello"

(* bmodule.ml *)
Amodule.hello()
```

```sh
ocamlopt -c amodule.ml
ocamlopt -c bmodule.ml
ocamlopt -o hello amodule.cmx bmodule.cmx
```

得到输出 `hello` 的可执行文件，也可以使用 `open` 指令。倾向于避免使用 `;;`  

其他比较常用的模块函数比如 `List.iter, Printf.printf`  

`ocamlopt` 是 OCaml native-code 编译器



## 2. 接口和签名

一个模块可以给其他程序提供很多功能（函数，类型，子模块，……）。如果没有什么特别指定，在模块中定义的**一切**可以从外部访问。  

比如

```ocaml
(* amodule.ml *)
let message = "Hello"
let hello () = print_endline message
```

事实上，`Amodule` 有下面的接口

```ocaml
val message : string
val hello : unit -> unit
```

但是在很多情况下，一个模块更应该只提供一系列有限（但是有用的）接口，而隐藏一些辅助的函数和类型。  

加入不想让其他模块直接访问 `message`，需要定义一个更严格的接口来隐藏它。

```ocaml
(* amodule.mli *)
val hello : unit -> unit
```

`.mli` 文件必须在对应的 `.ml` 文件之前编译

```sh
ocamlc -c amodule.mli
ocamlopt -c amodule.ml
```

关于 OCaml 文件名后缀约定：

| OCaml 字节码 |           OCaml 原生码            | 类比C  |    用途    |
| :----------: | :-------------------------------: | :----: | :--------: |
|     `ml`     |               `ml`                |  `c`   |   源文件   |
|    `mli`     |               `mli`               |  `h`   |   头文件   |
|    `cmo`     | `cmx/o(实际的机器码，通常可忽略)` |  `o`   |  目标文件  |
|    `cma`     |          `cmxa/a(同上)`           |  `a`   |   库文件   |
|    `prog`    |            `prog.opt`             | `prog` | 可执行程序 |



## 3. 抽象类型

在模块中定义新的类型：

```ocaml
type date = { day : int; month : int; year : int }
```

在编写 `mli` 文件时，类型也可以做成抽象的(只给出名字)

```ocaml
type date
val create : ?days:int -> ?months:int -> ?years:int -> unit -> date
val sub : date -> date -> date
val years : date -> float
```

或 record 的域做成只读的：

```ocaml
type date = private { ... }
```

## 4. 子模块

`example.ml` 文件本身可以隐式代表 `Example` 模块，一个给定的模块也可以在一个文件中显式定义，成为当前模块的一个子模块。  

```ocaml
(* example.ml *)
module Hello = struct
  let message = "hello"
  let hello () = print_endline message
end
let goodbye () = print_endline "goodbye"
let hello_goodbye () =
  Hello.hello ();
  goodbye ()
  
  
(* another.ml *)
let () =
  Example.Hello.hello ();
  Example.goodbye ()
```

**子模块接口**

可以约束一个给子模块的接口，叫做模块类型(Module Types)。

```ocaml
module Hello : sig
 val hello : unit -> unit
end = 
struct
  let message = "hello"
  let hello () = print_endline message
end
  
(* 在这里 Hello.message 不再能被访问。 *)
let goodbye () = print_endline "goodbye"
let hello_goodbye () =
  Hello.hello ();
  goodbye ()
```

把所有东西写在一个代码块里面是不优雅的，所以我们一般选择单独定义模块签名。  

```ocaml
module type Hello_type = sig
 val hello : unit -> unit
end
  
module Hello : Hello_type = struct
  ...
end
```



## 5. Functors

声明 Functors:

```ocaml
module F (Arg : Arg_type) : F_type
```

定义 Functors:

```ocaml
module F (Arg : Arg_type ) = struct
  ...
end
```

或  

```ocaml
module F(Arg : Arg_type) : F_type =
struct 
  ...
end
```



## 6. 模块实际的例子

```ocaml
module M = List;; (* type-level module aliases*)
```

模块扩展(原文是包含，感觉使用 extension 更准确)，对一个已有模块来扩充函数

```ocaml
module List = struct
  include List
  let rec optmap f = function
  | [] -> []
  | head :: tail ->
    match f head with
      | None -> optmap f tail
      | Some x -> x :: optmap f tail
```

创建了  `Extensions.List` 模块，这个模块有标准 `List` 模块所有的东西，加上一个新的 `optmap` 函数，要覆盖默认的 `List` 模块所要做的是在 `ml` 文件的开头  

```ocaml
open Extensions
List.optmap ...
```



# 五、数据结构

## 1. 简单数据结构

**(1) tuple**  

`tuple` 是一个有序集合，其中值可以是不同的类型，比如  

```ocaml
let atuple = (3, "three");;
val atuple : int * string = (3, "three")
```

使用了 `*` 符号，因为所有 `t * s` 类型对集合对应于类型 `t`  的元素集合与类型 `s` 元素集合的笛卡尔积  

获取 `tuple` 中的分量(模式匹配)：

```ocaml
let (x, y) = atuple;;
val x : int = 3
val y : string = "three"
```

**(2) list**

`list` 是可以保存任意数目相同类型的元素，比如  

```ocaml
let languages = ["OCaml";"Perl";"C"]
val languages : string list = ["OCaml";"Perl";"C"]
```

Core 提供了 List 模块，其中有大量用来处理列表的函数，比如  

```ocaml
List.length languages;;
(* - : int = 3 *)

List.map languages ~f:String.length;;
(* - : int list = [5; 4; 1] *)
```

用 `::` (语法糖，且 `::` 具有右结合性)构造 `list`

```ocaml
"French" :: "Spanish" :: languages;;
(* - : string list = ["French"; "Spanish"; "OCaml"; "Perl"; "C"] *)
```

`::` 只能在 `list` 前增加一个元素，列表以 `[]`(空列表)结束，另外还有一个列表连接操作符 `@`，但是 `@` 不是常数时间操作，和第一个列表长度成正比。  

```ocaml
[1;2;3] @ [4;5;6];;
(* - : int list = [1; 2; 3; 4; 5; 6] *)
```



**(3) option**

`option` 表示一个值可能有，也可能没有

```ocaml
let divide x y =
  if y = 0 then None else Some(x/y);;
```



## 2. 可变数据结构

1.8 中提到的数组 `Array` 以及带有 `mutable` 的字段的 `record`



# 附录A references

https://ocaml.org/learn/tutorials/index.zh.html  
《Real World OCaml》