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



## 1. 简单的表达式

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

---

`main` 函数在哪？

OCaml 没有一个唯一的 `main` 函数，执行 OCaml 文件时，文件中所有语句会按照其链接顺序进行计算  

在 `hello world` 程序中，`let () =` 的声明就扮演着 `main` 函数的角色。不过事实上，整个文件都会在启动时计算  

`let () =` 只是一个习惯用法，确保右边的表达式返回 `unit`，这个用法很常见，`unit` 类似于 `void`，之后详细说明  

---



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



## 4. tuple/option

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

**(2) option**

`option` 表示一个值可能有，也可能没有

```ocaml
let divide x y =
  if y = 0 then None else Some(x/y);;
```



## 5. 显式类型转换

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
Base.Int.max 3 4 (* prefix *)
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
let (|>) x f = f x;; (* 类似于 Unix 中的管道操作，左结合 *)
let path = "/usr/bin:/usr/local/bin:/bin:/sbin";;
String.split ~on: ':' path
  |> List.dedup_and_sort ~compare: String.compare 
  |> List.iter ~f: print_endline;;
```

```ocaml
let (^>) x f = f x;; (* 类似于 |>，不过是右结合 *)
```



## 6. 用函数声明函数

定义函数的另一种方法是使用 `function` 关键字，`function` 内置了模式匹配   

```ocaml
let some_or_zero = function
  | Some x -> x
  | None -> 0;;
```

等价于  

```ocaml
let some_or_zero num_opt
  match num_opt with
  | Some x -> x
  | None -> 0;;
```

把不同形式的函数声明结合在一起  

```ocaml
let some_or_default default = function
  | Some x -> x
  | None -> default;;
some_or_default 3 (Some 5);;  (* 5 *)
Some_or_default 100 None;;    (* 100 *)
```



## 7. 标签参数

之前有简单提到标签参数，按照名字标识一个函数参数，当然也能不写标签参数直接按照参数顺序进行传递  

```ocaml
let ratio ~num ~denom = float num /. float denom;;
ratio ~num:3 ~denom:10;;
ratio ~denom:10 ~num:3;;
```

OCaml 还支持标签“双关”，如果标签名和变量名相同，可以省略 `:` 后面的文本  

```ocaml
let num = 3 in
let denom = 10 in
ratio ~num ~denom;;
```

标签参数的作用：

- 参数多的时候，按名字记参数要比位置容易

- 仅从类型无法了解某个参数的含义时，可以使用标签参数  

- 定义多个参数，参数可能相互混淆，可以使用标签参数

  ```ocaml
  val substr : string -> int -> int -> string
  val substr : string -> pos:int -> len:int -> string
  ```

- 如果希望传递参数的顺序具有灵活性

**高阶函数和标签**  

将标签函数作为参数传入时，需要保证标签参数的顺序一致  

```ocaml
let apply_to_tuple1 f (first, second) = f ~first ~second;;
val apply_to_tuple1 : (first:'a -> second:'b -> 'c) -> 'a * 'b -> 'c = <fun>
let apply_to_tuple2 f (first, second) = f ~second ~first;;
val apply_to_tuple2 : (second:'a -> first:'b -> 'c) -> 'a * 'b -> 'c = <fun>

let divide ~first ~second = first / second;;
apply_to_tuple2 divide (3, 4);;
(* Error: This expression has type first:int -> second:int -> int
       but an expression was expected of type second:'a -> first:'b -> 'c *)
apply_to_tuple1 divide (3, 4);;
(* - : int = 0 *)
```



## 8. 可选参数

可选参数之前也提到过，特殊的标签参数  

```ocaml
let concat ?sep x y =
  let sep = match sep with
  | None -> ""
  | Some x -> x in
  x ^ sep ^ y;;
(* ^ is to concat two strings *)

concat "foo" "bar";;
concat ~sep:":" "foo" "bar";;

(* 可选参数的默认值 *)
let concat ?(sep="") x y =
  x ^ sep ^ y;;
```

可选参数本质上使用 `option` 数据结构，之后会详细介绍，如果调用者没有提供可选参数，函数会接收 None，如果调用者提供了这个参数，则会接收 `Some`  

有时候需要显式传入 Some 或 None，可以使用 `?` 来传递这个参数，而不是 `~`  

```ocaml
concat ?sep:(Some ":") "foo" "bar";;
concat ?sep:None "foo" "bar";;
```

使用可选参数会带来简洁，同时也会损失明确性。很少使用的函数不应该使用可选参数，模块内部使用的函数应该避免使用可选参数



```ocaml
let uppercase_concat ?(sep="") a b =
  concat ~sep (String.uppercase a) b;;
uppercase_concat "foo" "bar";; (* FOObar *)
uppercase_concat "foo" "bar" ~sep:":";; (* FOO:bar *)
```

按照此处逻辑，`uppercase_concat` 和 `concat` 的 `sep` 默认值应该保持一致，或者采取另外一种方式  

```ocaml
let uppercase_concat ?sep a b =
  concat ?sep (String.uppercase a) b;;
```



**标签参数和可选参数的类型推断**  

```ocaml
let numeric_deriv ~delta ~x ~y ~f =
  let x' = x +. delta in
  let y' = y +. delta in
  let base = f ~x ~y in
  let dx = (f ~x:x' ~y -. base) /. delta in
  let dy = (f ~x ~y:y' -. base) /. delta in
  (dx, dy);;
val numeric_deriv :
  delta:float ->
  x:float -> y:float -> f:(x:float -> y:float -> float) -> float * float =
  <fun>
```

由于标签参数可以按照任意顺序传入，原则上，`f` 的参数顺序如何选择并不是很清楚。更糟糕的是，`f` 可以取一个可选参数而不是标签参数。  

OCaml 需要一些启发式的规则，编译器尽量使用标签而不是可选参数，另外选择参数在源代码中出现的顺序。  

下面是 `numeric_deriv` 的另一个版本，不同的  `f` 调用会按不同的顺序列出参数：

```ocaml
let numeric_deriv ~delta ~x ~y ~f =
  let x' = x +. delta in
  let y' = y +. delta in
  let base = f ~x ~y in
  let dx = (f ~y ~x:x' -. base) /. delta in
  let dy = (f ~x ~y:y' -. base) /. delta in
  (dx, dy);;
(* Error: This function is applied to arguments
in an order different from other calls.
This is only allowed when the real type is known. *)
```

正如报错信息，需要提供显式类型信息，指出会以不同的参数顺序调用 `f`

```ocaml
let numeric_deriv ~delta ~x ~y ~(f: x:float -> y:float -> float) =
  let x' = x +. delta in
  let y' = y +. delta in
  let base = f ~x ~y in
  let dx = (f ~y ~x:x' -. base) /. delta in
  let dy = (f ~x ~y:y' -. base) /. delta in
  (dx, dy);;
```



**可选参数与部分应用**

```ocaml
let colon_concat = concat ~sep:":";
```

如果只是部分应用第一个参数会发生什么？

```ocaml
let prepend_pound = concat "#";;
prepend_pound "a BASH comment";; (* # a BASH comment *)
```

向 `prepend_pound` 传入 `~sep` 参数会报错，因为 `?sep` 被“擦除”了。如果可选参数后面第一个位置上定义的参数一旦传入，可选参数就会被擦除，如果可选参数后面没有参数，可选参数就不会被擦除，不过会导致编译器警告  

如果 `concat` 把可选参数定义到第二个位置上：

```ocaml
let concat x ?(sep="") y = x ^ sep ^ y;;
let prepend_pound = concat "#";;
prepend_pound "a BASH comment" ~sep:"---";;
```



# 四、list

## 1. 基础

注：在 `utop` 中自测得使用 `open Base;;` 或使用 `Base.List` 进行操作，具体原因之后探索  

`list` 是相同类型元素构成的一个不可变的有序序列，用`[]`  来生成，list 对应的数据结构是**链表**

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

`[]` 空表是多态的，可以用于任意类型的元素。  

```ocaml
let empty = [];; (* val empty : 'a list = [] *)
```



## 2. list 与模式匹配

**使用模式从列表中抽取数据**  

```ocaml
let rec sum l =
  match l with
  | [] -> 0
  | head :: tail -> head + sum tail
```

如果想要删除某个字面量元素  

```ocaml
let rec drop_zero l = 
  match l with
  | [] -> []
  | 0 :: tail -> drop_zero tail
  | head :: tail -> head :: drop_zero tail
```



**模式匹配的局限性(好处)**  

模式匹配并不能表述任意条件，模式仅可以描述一个数据结构的布局，甚至包含字面量，不过仅此而已。比如模式可以检测一个列表是否包含两个元素，但是不能检查两个元素是否相等。  

不过模式表示一个有限的条件集合，实际上可以在编译器中更好地支持模式。比如 `match` 语句的高效率以及检测匹配错误的能力(冗余、完备性检查等)。  



## 3. 使用 List 模块

绘制字符表格例子  

```ocaml
printf "%s\n" 
  (render_table 
    ["language"; "architect"; "first release"]
    [ ["Lisp"; "John McCarthy";  "1958"];
      ["C";    "Dennis Ritchie"; "1969"];
      ["ML";   "Robin Milner";   "1973"];
      ["OCaml";"Xavier Leroy";   "1996"];
    ]);;
```

需要计算最大列宽，直接使用代码来实现很费功夫，可以使用 `List` 模块的三个函数(`map, map2_exn, fold`)来很简洁地完成这些工作：  

`List.map` 用一个函数 `f` 来对列表地每个元素进行映射转换，返回一个新的列表  

`List.map2_exn` 取两个列表和一个函数作为参数，用于结合这两个列表，`_exn` 表示会抛出异常  

`List.fold` 取三个参数，要处理的列表，累加器初始值，以及更新累加器函数  

```ocaml
List.map ~f:String.length ["hello"; "world!"];; (* [5; 6] *)
List.map2_exn ~f:Int.max [1;2;3] [3;2;1];;      (* [3; 2; 3] *)
List.fold ~init: 0 ~f:(+) [1;2;3;4];;           (* 10 *)
List.fold ~init:[] 
  ~f:(fun list x -> x :: list) [1;2;3;4];;      (* [4;3;2;1] *)
```

结合三个函数，计算最大列宽  

```ocaml
(* calc max widths *)
let max_widths header rows =
  let lengths l = List.map ~f:String.length l in
  List.fold rows
    ~init:(lengths header)
    ~f:(fun acc row -> List.map2_exn ~f:Int.max acc (lengths row));;

(* render sep *)
let render_separator widths =
  let pieces = List.map widths
    ~f:(fun w -> String.make(w + 2) '-') in
    "|" ^ String.concat ~sep:"+" pieces ^ "|";;
(*render_separator [3;6;2];;*) (* "|-----+--------+----|" *)

(* pad content *)
let pad s length =
  " " ^ s ^ String.make (length - String.length s+1) ' ';;
(*pad "hello" 10*)    (* " hello      " *)

(* render row *)
let render_row row widths =
  let padded = List.map2_exn row widths ~f:pad in
  "|" ^ String.concat ~sep:"|" padded ^ "|";;
  
let render_table header rows =
  let widths = max_widths header rows in
  String.concat ~sep:"\n"
    (render_row header widths
      :: render_separator widths
      :: List.map rows ~f:(fun row -> render_row row widths)
    );;
```

`^` 是连接字符串操作，不过在拼接大型字符串会有性能问题，每次拼接都会分配一个新的字符串  



**List 模块更多有用的函数**  

[List](https://ocaml.janestreet.com/ocaml-core/109.20.00/doc/core/List.html)  

**(1) `reduce`**  

`reduce` 是简化版的 `fold` 函数，不需要指定 `init`，累加函数必须消费和生产和列表元素类型相同的值  

返回的是 `option` 结果  

```ocaml
List.reduce ~f:(+) [1;2;3;4;5];; (* Some 15 *)
List.reduce ~f:(+) [];;          (* int option = None *)
```

**(2) `filter`, `filter_map`**   

```ocaml
List.filter ~f:(fun x -> x mod 2 = 0) [1;2;3;4;5];; (* [2;4] *)
List.filter_map (Sys.ls_dir ".")
  ~f:(fun fname -> 
    match String.rsplit2 ~on:'.' fname with
    | None | Some ("",_) -> None
    | Some (_,ext) -> Some ext)
    |> List.dedup;;
```

**(3) `partition_tf`**  

对列表元素计算一个布尔条件，并返回两个列表  

```ocaml
let is_ocaml_source s =
  match String.rsplit2 s ~on:'.' with
  | Some (_,("ml"|"mli")) -> true
  | _ -> false;;
let (ml, other) =
  List.partition_tf (Sys.ls_dir ".") ~f:is_ocaml_source;;
```

**(4) `append/concat`**

```ocaml
List.append [1;2;3] [4;5;6];; (* [1;2;3;4;5;6] *)
(* <=> *)
[1;2;3] @ [4;5;6];;

List.concat [[1;2];[3;4;5];[6];[]];; (* [1;2;3;4;5;6] *)
```



## 4. 尾递归 

```ocaml
let rec length = function
  | [] -> 0
  | _ :: tail -> 1 + length tail;;
length (make_list 1000_0000);;
```

如果对特别大的 list 调用 `length` 函数，会造成递归过多导致栈溢出，可以使用尾递归的方式进行计算  

```ocaml
let rec lengthv2 l n =
  match l with
  | [] -> n
  | _ :: tail -> lengthv2 tail (n+1);;
```



## 5. 更简洁更快速

删除重复的连续元素  

```ocaml
let rec destutter list =
  match list with
  | [] -> []
  | [hd] -> [hd]
  | hd :: hd' :: tl -> 
    | if hd = hd' then destutter (hd' :: tl)
      else hd :: destutter(hd' :: tl);;
```

首先考虑效率  `[hd] -> [hd]` 实际会分配一个新的列表元素，实际上，它应该只返回匹配的列表，这里可以使用  `as` 模式来减少分配，使用 Or 模式来合并前两种情况   

可以使用 `function` 来减少显式使用 `match`   

```ocaml
let rec destutter = function
  | [] | [_] as l -> l
  | hd :: hd' :: tl -> 
    | if hd = hd' then destutter (hd' :: tl)
      else hd :: destutter(hd' :: tl);;
```

使用 `when` 子句为模式增加额外的前置条件  

```ocaml
let rec destutter = function
  | [] | [_] as l -> l
  | hd :: (hd' :: _ as tl) when hd = hd' -> destutter tl
  | hd :: tl -> hd :: destutter tl
```



---

**多态比较**

OCaml 的 `=` 操作符允许我们对不同类型进行比较，查看 `=` 的类型  

```ocaml
(=);;
(* - : 'a -> 'a -> bool = <fun> *) 
```

---



`when` 子句有一些缺点，模式匹配一旦能力增强，有些东西就会丢失，比如检查 `match` 的完备性等等  



# 五、模块

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



把前面的例子进行抽象，模块的声明的基本结构  

```ocaml
module <name> : <signature> = <implementation>

module <name> : sig
  type t
  (* val ... *)
end = struct
  type t
  (* let ... impl *)
end
```

还可以采取不同的写法  

```ocaml
module type <name> = sig
  (* sig *)
  type t
  (* val ... *)
end

module type <name> = struct
  (* impl *)
  type t = ...
  (* let ... *)
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



## 6. 单文件程序

下面的程序是从标准输入获取行，构造一个关联列表(`line -> freq`)。然后通过 `|>` 操作符按照频度对列表进行排序，找出列表前 10 的元素，然后迭代 10 个元素进行输出  

```ocaml
(* freq.ml *)
open Core

let build_counts () =
  In_channel.fold_lines In_channel.stdin ~init:[] ~f:(fun counts line ->
    let count =
      match List.Assoc.find counts (=) line with
      | None -> 0
      | Some x -> x
    in
    List.Assoc.add counts (=) line (count + 1)
  )


let () = build_counts ()
  |> List.sort ~compare:(fun (_,x) (_,y) -> Base.Int.descending x y)
  |> (fun l -> List.take l 10)
  |> List.iter ~f:(fun (line, count) -> printf "%3d: %s\n" count line)
```



进行构建  

```sh
ocamlc freq.ml -o freq.byte
# File "freq.ml", line 1, characters 5-9:
# Error: Unbound module Core
```

但是出现了编译错误，无法找到 `Core` 库，在没有使用 `Core` 或其他外部库的情况下，可以使用  `ocamlc` 进行构建二进制程序  

这里需要找到 `Core` 库并链接 `Core` 库  

```ocaml
ocamlfind ocamlc -linkpkg -thread -package core freq.ml -o freq.byte
```

对于单文件程序这样就可以了，更复杂的项目需要一个 `ocamlbuild` 来协调构建过程，之后进行说明，现在使用 `ocamlbuild` 一个简单的包装器就可以完成构建，叫做 `corebuild`  

```ocaml
corebuild freq.byte       (* byte code *)
corebuild freq.native     (* executable *)
```



---

**byte code and native code**

OCaml 附带了两个编译器，`ocamlc` byte code 和 `ocamlopt` naitve code 编译器，用 `ocamlc` 编译的程序由一个虚拟机进行解释  

除了性能之外，还有几点需要注意：  

- 字节码编译器可以在更多体系结构上使用，OCaml 调试工具只能处理字节码(不过GDB确实能够处理 native code)
- 运行字节码通常需要安装 OCaml，不过也可以通过 `-custom` 编译器标志，也可以用一个运动时系统构建字节码程序
- 产品可执行程序一般当作原生码来构建，不过有时候也可以使用字节码来构建

---



## 7. 多文件程序和模块

之前的程序使用 list 来记录字符串的频数，处理一个文件的时间复杂度类似于是文件行数的二次方，类似于C++ 的 `std::list<std::pair<std::string, std::size_t>`，接下来确定接口然后重构为更高效的实现  

首先将关键功能抽取到一个单独的模块中  

```ocaml
(* counter.ml *)
open Core

let touch table str =
  let count = 
    match List.Assoc.find table (=) str with
    | None -> 0
    | Some x -> x
  in
  List.Assoc.add table (=) str (count + 1)
```

文件 `counter.ml` 将被编译为一个名为 `Counter` 的模块，模块名为文件名首字母大写  

现在重写 `freq.ml`

```ocaml
(* freq.ml *)
open Core

let build_counts () =
  In_channel.fold_lines In_channel.stdin ~init:[] ~f:Counter.touch

let () = build_counts ()
  |> List.sort ~compare:(fun (_,x) (_,y) -> Base.Int.descending x y)
  |> (fun l -> List.take l 10)
  |> List.iter ~f:(fun (line, count) -> printf "%3d: %s\n" count line)
```



## 8. 签名和抽象类型

模块的实现细节可以通过接口(interface，在 OCaml 中 interface/signature/module type 会交替使用)来隐藏，类似于在头文件中进行声明，`filename.ml` 定义的模块会受文件 `filename.mli` 中的签名约束  

也可以使用 `corebuild counter.inferred.mli` 生成 `mli` 文件  

```ocaml
(* counter.inferred.mli *)
open Core
val touch :
  ('a, int) Base__List.Assoc.t -> 'a -> ('a, int) Base__List.Assoc.t
  
(* counter.mli *)
open Core
val touch :
  ('a, int) Core.List.Assoc.t -> 'a -> ('a -> int) Core.List.Assoc.t
```



接下来进行重构，频数的实现为关联列表，我们需要在接口中隐藏这一点  

```ocaml
(* counter.mli *)
(** doc comment *)
(** table *)
type t

(** empty table *)
val empty : t

(* add freq count of the string *)
val touch : t -> string -> t

(* interface table to_list*)
val to_list : t -> (string * int) list

(* counter.ml *)
open Core

type t = (string * int) list

let empty = []

let to_list x = x

let touch t s =
  let count =
    match List.Assoc.find t (=) s with
    | None -> 0
    | Some x -> x
  in
  List.Assoc.add t s (count + 1)

(* freq.ml *)
let build_counts () =
  In_channel.fold_lines In_channel.stdin ~init:Counter.empty ~f:Counter.touch

let () = 
  build_counts ()
  |> Counter.to_list
  |> List.sort ~compare:(fun (_,x) (_,y) -> Base.Int.descending x y)
  |> (fun l -> List.take l 10)
  |> List.iter ~f:(fun (line, count) -> printf "%3d: %s\n" count line)
```

接下来换成更高效的数据结构  

```ocaml
(* counter.ml *)
type t = int String.Map.t
let empty = String.Map.empty
let to_list t = Map.to_alist t
let touch t s =
  let count =
    match Map.find t s with
    | None -> 0
    | Some x -> x
  in
  Map.add t ~key:s ~data:(count + 1)
```

TODO: 不过上述代码因为类型原因未通过编译，之后填坑  



## 9. 签名中的具体类型

在接口中增加具体类型，比如在 `Counter` 模块增加 `median` 函数，返回频数的中位数  

```ocaml
(* counter.mli *)
val median = | Median of string
             | Before_and_after of string * string

val median : t -> median

(* counter.ml *)
type median = | Median of string
               | Before_and_after of string * string

let median t =
  let sorted_strings = List.sort t
    ~compare:(fun (_,x) (_,y) -> Int.descending x y)
  in
  let len = List.length sorted_strings in
  if len = 0 then failwith "median: empty frequency count";
  let nth n = fst (List.nth_exn sorted_strings n) in
  if len mod 2 = 1
  then Median (nth (len/2))
  else Before_and_after (nth (len/2 - 1), nth (len/2))
```



## 10. `open`

`open` 会将一个模块的所有内容添加到一个环境中  

`open` 的建议：

- 尽可能少在模块的顶层 `open` 模块，通常只 `open` 那些专门这样设计的模块

- 如果确实要 `open`，最好采用局部打开

  ```ocaml
  let average x y =
    let open Int64 in
    x + y / of_int 2
    
  (* or *)
  let average x y =
    Int64.(x + y / of_int 2)
  ```

- 局部打开的另一种方法是重新绑定一个模块名  

  ```ocaml
  let print_median m =
    match m with
    | Counter.Median string -> printf "True median:\n %s\n" string
    | Counter.Before_and_after (before, after) -> 
      printf "Before and after median: \n %s\n %s\n" before after
      
  (* -> *)
  
  let print_median m =
    let module C = Counter in
    match m with
    | C.Median string -> printf "True median:\n %s\n" string
    | C.Before_and_after (before, after) -> 
      printf "Before and after median: \n %s\n %s\n" before after
  ```



## 11. `include`

考虑下面的例子  

```ocaml
module Interval = struct
type t = | Interval of int * int
         | Empty

let create low high =
  if high < low then Empty else Interval (low, high) end;;
```

模块 `Interval` 只有 `create` 函数用来创建 `Interval.t`，现在我们想要在已有的 `Interval` 模块增加实现一个 `contains` 函数，判断给定元素是否在区间内，可以使用  `include`，可以实现类似于其他编程语言中的 `extend`  

```ocaml
module Extend_interval = struct
  include Interval
  let contains t x =
    match t with
    | Empty -> false
    | Interval (low, high) -> x >= low && x <= high end;;

Extend_interval.contains (Extend_interval.create 3 10) 4;;
```

这个功能更可能给标准模块或第三方模块进行扩展，比如给 `List` 模块进行扩展  

```ocaml
(* ext_list.mli *)
open Core

include (module type of List)

val intersperse : 'a list -> 'a -> 'a list

(* ext_list.ml *)
open Core

let rec intersperse list el
  match list with
  | [] | [_] -> list
  | x :: y :: tail -> x :: el :: intersperse (y :: tail) el
  
include List
```

可以使用 `Ext_list` 来代替 `List` 模块，创建 `Common` 模块，然后在 `Common` 模块中 `module List = Ext_list`，然后在其他的文件中先 `open Core`，再 `open Ext_list`，这样 `List` 就自动指示 `Ext_list`  



## 12. 模块的常见错误

- 签名和实现类型不匹配

- 缺少定义

- 类型定义不匹配
- 循环依赖



## 13. 基于模块的设计

- 尽量少地提供具体类型(尽量保持为抽象类型)，提供模块的灵活性
- 写模块接口要清晰
- 创建统一的接口
  - 几乎面向每一个类型有一个模块，而且模块的主要类型为 `t`
  - 把 `t` 放在最前面
  - 遵守命名规范，比如抛出异常的函数以 `_exn` 结尾，否则返回 `option` 或 `Or_error.t` 等等 
- 接口先于实现



# 六、记录

记录(`record`)，类似于 `struct`

```ocaml
type <record-name> = {
  <field> : <type>;
  (* ... *)
};;
```

```ocaml
type host_info = {
  hostname : string;
  os_name : string;
  cpu_arch : string;
  timestamp : Time.t;
};;
```

```ocaml
(* utop *)
#require "core_extended";;
open Core_extented

let my_host =
  let sh = Shell.sh_one_line_exn in {
    hostname = sh "hostname";
    os_name = sh "uname -s";
    cpu_arch = sh "uname -p";
    timestamp = Time.now();
  };;
  
(* {hostname = "..."; os_name = "Linux"; cpu_arch = "x86_64";
   timestamp = 2020-05-03 15:54:56.706941+08:00} *)
```

```ocaml
(* access record member *)
my_host.cpu_arch;;
(* "x86_64" *)
```

**多态化声明 `record`**

```ocaml
type 'a timestamped = { item: 'a; time: Time.t };;
let first_timestamped list =
  List.reduce list ~f: (fun a b -> if a.time < b.time then a else b)
```



## 1. 模式和完备性

从 `record` 得到信息，还有一种方法是模式匹配  

```ocaml
let host_info_to_string {
  hostname = h; os_name = os;
  cpu_arch = c; timestamp = ts;
} = 
sprintf "%s (%s / %s, on %s)" h os c (Time.to_string ts);;

host_info_to_string my_host;;
```

如果在 `record host_info` 增加一个字段，上述程序仍能工作，但是这时候需要一个警告信息，在 `utop` 中使用 `# warning "+9"`   来打开警告  

如果确定是不完备的，可以使用 `_` 来禁用警告  

```ocaml
let host_info_to_string {
  hostname = h; os_name = os;
  cpu_arch = c; timestamp = ts; _
} = 
sprintf "%s (%s / %s, on %s)" h os c (Time.to_string ts);;
```



## 2. 字段双关

当变量名与字段名恰好相同时，OCaml 提供了方便的语法快捷方式  

使用字段双关来构建 record  

```ocaml
let host_info_to_string {
  hostname; os_name;
  cpu_arch; timestamp; _
} = 
sprintf "%s (%s / %s, on %s)" hostname os_name
  cpu_arch (Time.to_string time_stamp);;
  
  
let my_host =
  let sh cmd = Shell.sh_one_line_exn cmd in
  let hostname = sh "hostname" in
  (* ... *)
  let timestamp = Time.now () in {
    hostname; os_name; cpu_arch; timestamp
  };;
```

编写构建 record 的函数时，同时利用字段双关和标签双关  

```ocaml
let create_host_info ~hostname ~os_name ~cpu_arch = {
  os_name; cpu_arch; hostname = String.lowercase hostname;
  timestamp = Time.now ()
};;
```



## 3. 重用字段名

不同的记录使用相同的字段名可能会出问题  

```ocaml
type log_entry = { 
  session_id: string;
  time: Time.t;
  important: bool;
  message: string;
}
type heartbeat = {
  session_id: string;
  time: Time.t;
  status_message: string;
}
type logon = {
  session_id: string;
  time: Time.t;
  user: string;
  credentials: string;
};;
```

如果编写一个函数获取 `session_id`，该函数会是什么类型呢？  

```ocaml
let get_session_id t = t.session_id;;
(* val get_session_id : logon -> string = <fun> *)
```

在这种情况下，OCaml 选择了这个记录字段的最新定义，也就是 `logon`  

使用类型标注来定义函数  

```ocaml
let get_heartbeat_session_id (t:heartbeat) = t.session_id;;
(* val get_heartbeat_session_id : heartbeat -> string = <fun> *)
```

尽管可以使用类型标注，不过这种二义性还是让人困惑，从 `heartbeat` 中获取 `session_id` 和 `status`  

```ocaml
let status_and_session t = (t.status_message, t.session_id);;
(* val status_and_session : heartbeat -> string * string = <fun> *)

let session_and_status t = (t.session_id, t.status_message);;
(* Characters 44-58:
Error: The record type logon has no field status_message *)

let session_and_status (t:heartbeat) = (t.session_id, t.status_message);;
(* val session_and_status : heartbeat -> string * string = <fun> *)
```

第一个没有类型标注可以成功编译，但是第二个没有类型标注就失败了，第一个类型先考虑 `status_message`，而第二个会先考虑 `session_id`  

可以使用模块类型避免这种二义性  

```ocaml
module Log_entry = struct
  type t = {
    session_id: string;
    time: Time.t;
    important: bool;
    message: string;
  }
end
module Heartbeat = struct
  type t = {
    session_id: string;
    time: Time.t;
    status_message: string;
  }
end
module Logon = struct
  type t = {
    session_id: string;
    time: Time.t;
    user: string;
    credentials: string;
  }
end;;

(*module Log_entry :
  sig
    type t = {
      session_id : string;
      time : Time.t;
      important : bool;
      message : string;
    }
  end
module Heartbeat :
  sig
    type t = { session_id : string; time : Time.t; status_message : string; }
  end
module Logon :
  sig
    type t = {
      session_id : string;
      time : Time.t;
      user : string;
      credentials : string;
    }
  end
*)
```

创建日志记录  

```ocaml
let create_log_entry ~session_id ~important ~message = {
  Log_entry.time = Time.now (); Log_entry.session_id;
  Log_entry.important; Log_entry.message
};;

(* or *)
let create_log_entry ~session_id ~important ~message = {
  Log_entry.
  time = Time.now (); session_id;
  important; message
};;
```

访问字段  

```ocaml
let is_important t = t.Log_entry.important;;
```



## 4. 函数式更新

当我们需要改变 `record`，原 `record` 和 新的 `record` 只有一两个字段不一样，其他都相同，可以使用 `create` 函数来构造一个新的 `record`，但是这样做比较繁琐，使用 OCaml 的函数式更新(*functional update*)语法，更简洁地达到这个目的  

```ocaml
{
  <record> with 
    <field> = <value>
    <field> = <value>
    ...
}
```

具体的例子  

```ocaml
type client_info = {
  addr: Unix.Inet_addr.t;
  port: int;
  user: string;
  credentials: string;
  last_heartbeat_time: Time.t;
};;

let register_heartbeat t hb = {
  t with last_heartbeat_time = hb.Heartbeat.time
};;
```

*functional update* 允许代码独立于记录中不变的字段，但是也有一些缺点，如果改变了类型定义，增加一些字段，类型系统不会让你重新考虑是否修改代码来处理新增加的字段  



## 5. 可变字段

OCaml 中 `record` 中的字段默认是不可变的，不过可以把字段声明为可变字段  

```ocaml
type client_info = {
  addr: Unix.Inet_addr.t;
  port: int;
  user: string;
  credentials: string;
  mutable last_heartbeat_time: Time.t;
};;
```

`<-` 操作符用来设置可变字段  

```ocaml
let register_heartbeat t hb = {
  t.last_heartbeat_time <- hb.Heartbeat.time
};;
```



## 6. 字段作为一等公民

```ocaml
let get_users logons =
  List.dedup (List.map logons ~f:(fun x -> x.Logon.user))
```

上述函数使用 `(fun x -> x.Logon.user)` 访问 `user` 字段，如果可以自动生成访问函数会很方便  

Core 库带有的 `fieldslib` 扩展就可以做到  

记录声明的末尾添加 `with fields` 标注，会使这个 `fieldslib` 扩展应用到给定的类型声明  

```ocaml
module Logon = struct
    type t =
      { session_id: string;
        time: Time.t;
        user: string;
        credentials: string;
      }
    with fields
  end;;

(* module Logon :
  sig type t = {
    session_id : string;
    time : Time.t;
    user : string;
    credentials : string;
  }
  val credentials : t -> string
  val user : t -> string
  val time : t -> Time.t
  val session_id : t -> string
  module Fields :
    sig
      val names : string list
      val credentials :
        ([< `Read | `Set_and_create ], t, string) Field.t_with_perm
      val user :
        ([< `Read | `Set_and_create ], t, string) Field.t_with_perm
      val time :
        ([< `Read | `Set_and_create ], t, Time.t) Field.t_with_perm
      val session_id : ([< `Read | `Set_and_create ], t, string) Field.t_with_perm
      [ ... many definitions omitted ... ]
    end
end *)
```

会自动生成很多输出，因为 `fieldslib` 产生了一大堆处理记录字段的辅助函数   

首先要提及的函数就是 `Logon.user`，用以从一个 `logon` 消息中提取 `user` 字段：

```ocaml
let get_users logons =
  List.dedup (List.map logons ~f:Logon.user);;
```

除了生成段访问器函数，`fieldslib` 还创建了一个名为 `Fields` 的子模块，此模块包含了每一个字段的一等公民表示，形式是一个类型为 `Field.t` 的值。`Field` 模块提供了下列函数：  

```ocaml
Field.name
Field.get
Field.fset   (* functional update field*)
Field.setter (* return None if field is non-mutable, else return Some f *)
```

`Field.t` 有两个类型参数：第一参数是记录类型，第二个是相关字段的类型。因此，`Logon.Fields.session_id` 的类型是`（Logon.t, string） Field.t`，而 `Logon.Fields.time` 的类型是 `(Logon.t, Time.t) Field.t`。因此，如果你对 `Logon.Fields.user` 调用 `Field.get`，将会得到一个从 `Logon.t` 中提取 `user` 字段的函数：

```ocaml
Field.get Logon.Fields.user;;
(* - : Logon.t -> string = <fun> *)
```



# 七、变体

变体(`variant`)

```ocaml
type <variant_name> = 
  | <Tag> [of <type> [ * <type> ]... ]
  | <Tag> [of <type> [ * <type> ]... ]
  | ...
```

注意变体的首字母必须大写，例如：  

```ocaml
type basic_color =
  | Black | Red | Green | Yellow | Blue | Magenta | Cyan | White;;
Cyan;;
(* - : basic_color = Cyan *)
[Blue; Magenta; Red];;
(* - : basic_color list = [Blue; Magenta; Red] *)
```

和模式匹配一起使用  

```ocaml
let basic_color_to_int = function
  | Black -> 0 | Red -> 1     | Green -> 2 | Yellow -> 3
  | Blue -> 4  | Magenta -> 5 | Cyan -> 6  | White -> 7;;
  
open Core;;
List.map ~f:basic_color_to_int [Blue; Red];;
```

改变终端输出的颜色  

```ocaml
let color_by_number number text =
  sprintf "\027[38;5;%dm%s\027[0m" number text;;

let blue = color_by_number (basic_color_to_int Blue) "Blue";;
printf "%s\n" blue;;
```

`basic_color` 都是只有简单的 `Tag`，没有关联的数据，类似于 C 中的 `enum`  

不过 `enum` 无法枚举终端的所有颜色，现在终端一般支持 256 种颜色，分别以下几种  

- 8 种基本颜色，分别有常规和加粗版本(16)
- 6 * 6 * 6 RGB 颜色(216)
- 24 级灰度色谱(24)

```ocaml
type weight = Regular | Bold
type color =
  | Basic of basic_color * weight
  | RGB of int * int * int
  | Gray of int;;

[RGB (250, 70, 70); Basic (Green, Regular)];;
(* - : color list = [RGB (250, 70, 70); Basic (Green, Regular)] *)

let color_to_int = function
  | Basic (basic_color, weight) -> 
    let base = match weight with Bold -> 8 | Regular -> 0 in
    base + basic_color_to_int basic_color
  | RGB (r, g, b) -> 16 + b + g * 6 + r * 36
  | Gray i -> 232 + i;;
  
let color_print color str =
  printf "%s\n" (color_by_number (color_to_int color) str);;
  
color_print (Basic (Blue, Bold)) "Bold Blue";;
color_print (Gray 4) "Muted Gray";;
```



## 1. 重构

OCaml 的类型系统相当于一个重构工具，会警告代码中哪些地方应当更新来应对接口的改变  

```ocaml
type color =
  | Basic of basic_color
  | Bold of basic_color
  | RGB of int * int * int
  | Gray of int;;
```

再次编译，`color_to_int` 会提示 `Basic Tag` 的参数数量错误，修复之后  

```ocaml
let color_to_int = function
  | Basic basic_color -> basic_color_to_int basic_color
  | RGB (r, g, b) -> 16 + b + g * 6 + r * 36
  | Gray i -> 232 + i;;
```

再次编译，编译器又会提示 `Bold` 没有匹配的警告  

```ocaml
let color_to_int = function
  | Basic (basic_color, weight) -> basic_color_to_int basic_color
  | Bold basic_color -> 8 + basic_color_to_int basic_color
  | RGB (r, g, b) -> 16 + b + g * 6 + r * 36
  | Gray i -> 232 + i;;
```

要避免模式匹配 `catch-all` (全包)的情况  

```ocaml
let old_color_to_int = function
  | Basic (basic_color, weight) -> 
    let base = match weight with Bold -> 8 | Regular -> 0 in
    base + basic_color_to_int basic_color
  | _ -> basic_color_to_int White;;
```

当添加 `Bold` 后，类型系统不会发出警告  



## 2. 结合记录和变体

*代数数据类型(algebraic data types)* 这个术语经常用以描述包括变体、记录和元组这几个类型的一组类型。代数数据类型是一种用以描述数据的有用且强大的语言。核心原因是它们结合了两种不同类型： *积类型（product type）*，像元组和记录，把不同类型组合在一起，数学上类似于笛卡儿积；以及 *和类型 (sum type)*，像变体，它可以把不同的可能组合在一个类型中，数学上类似于不相交并集。

回忆之前的 `Log_entry.t` 的定义  

```ocaml
module Log_entry = struct
  type t = { 
    session_id: string;
    time: Time.t;
    important: bool;
    message: string; 
  }
  end;;
```

更一般的，可以将记录看为合取，将变体看为析取  

```ocaml
type client_message =
  | Logon of Logon.t
  | Hearbeat of Heartbeat.t
  | Log_entry of Log_entry.t;;
```

`client_message` 是一个 `Logon` 或者 `Heartbeat` 或者 `Log_entry`  

考虑下面这个函数，接收一个 `client_message` 列表，返回给定用户的所有消息。

```ocaml
let messages_for_user user messages =
    let (user_messages,_) =
      List.fold messages ~init:([],String.Set.empty)
        ~f:(fun ((messages,user_sessions) as acc) message ->
          match message with
          | Logon m ->
            if m.Logon.user = user then
              (message::messages, Set.add user_sessions m.Logon.session_id)
            else acc
          | Heartbeat _ | Log_entry _ ->
            let session_id = match message with
              | Logon     m -> m.Logon.session_id
              | Heartbeat m -> m.Heartbeat.session_id
              | Log_entry m -> m.Log_entry.session_id
            in
            if Set.mem user_sessions session_id then
              (message::messages,user_sessions)
            else acc
        )
    in
    List.rev user_messages;;
```

上面的代码很难看懂，尤其是决定 session_id 的部分，对上面的功能进行重构  

```ocaml
module Log_entry = struct
  type t = {
    important: bool;
    message: string;
  }
end

module Heartbeat = struct
  type t = { status_message: string; }
end

module Logon = struct
  type t = {
    user: string;
    credentials: string;
  }
end;;

type details =
    | Logon of Logon.t
    | Heartbeat of Heartbeat.t
    | Log_entry of Log_entry.t;;
    
module Common = struct
  type t = { 
    session_id: string;
    time: Time.t;
  }
end;;
```

一个完整的消息可以使用 `Common.t` 和一个 `details` 构成的记录对来表示，把前面的方法重写  

```ocaml
let messages_for_user user messages =
    let (user_messages,_) =
      List.fold messages ~init:([],String.Set.empty)
        ~f:(fun ((messages,user_sessions) as acc) ((common,details) as message) ->
          let session_id = common.Common.session_id in
          match details with
          | Logon m ->
            if m.Logon.user = user then
              (message::messages, Set.add user_sessions session_id)
            else acc
          | Heartbeat _ | Log_entry _ ->
            if Set.mem user_sessions session_id then
              (message::messages,user_sessions)
            else acc
        )
    in
    List.rev user_messages;;
```



## 3. 变体和递归数据结构

变体的另一个常见应用是表示树状递归数据结构  

```ocaml
type 'a expr =
  | Base  of 'a
  | Const of bool
  | And   of 'a expr list
  | Or    of 'a expr list
  | Not   of 'a expr ;;
```

计算表达式  

```ocaml
let rec eval expr base_eval =
  (* a shortcut, so we don't need to repeatedly pass [base_eval]
     explicitly to [eval] *)
  let eval' expr = eval expr base_eval in
  match expr with
  | Base  base   -> base_eval base
  | Const bool   -> bool
  | And   exprs -> List.for_all exprs ~f:eval'
  | Or    exprs -> List.exists  exprs ~f:eval'
  | Not   expr  -> not (eval' expr);;
```



可以对上面的例子进行化简  

```ocaml
let and_ l =
  if List.mem l (Const false) then Const false
  else
    match List.filter l ~f:((<>) (Const true)) with
    | [] -> Const true
    | [ x ] -> x
    | l -> And l

let or_ l =
  if List.mem l (Const true) then Const true
  else
    match List.filter l ~f:((<>) (Const false)) with
    | [] -> Const false
    | [x] -> x
    | l -> Or l

let not_ = function
  | Const b -> Const (not b)
  | (Base _ | And _ | Or _ | Not _) as e -> Not e;;
  
  
let rec simplify = function
  | Base _ | Const _ as x -> x
  | And l -> and_ (List.map ~f:simplify l)
  | Or l  -> or_  (List.map ~f:simplify l)
  | Not e -> not_ (simplify e);;
```



## 4. 多态变体

```ocaml
#let three = `Int 3;;
* val three : [> `Int of int ] = `Int 3 *
# let four = `Float 4.;;
val four : [> `Float of float ] = `Float 4.
# let nan = `Not_a_number;;
val nan : [> `Not_a_number ] = `Not_a_number
# [three; four; nan];;
- : [> `Float of float | `Int of int | `Not_a_number ] list =
[`Int 3; `Float 4.; `Not_a_number]
```

标签名（如` Int`）和类型名 (`int`) 是匹配的，如果同一个类型使用不兼容就会报错  

```ocaml
let five = `Int "five";;
val five : [> `Int of string ] = `Int "five"
# [three; four; five];;
(* Characters 14-18:
Error: This expression has type [> `Int of string ]
       but an expression was expected of type
         [> `Float of float | `Int of int ]
       Types for tag `Int are incompatible *)
```

变体类型最开始的 `>` 标志了类型可以与其他的变体类型结合，可以把 `>` 理解为这些标记或更多，可能包括更多标记，`<` 表示这些或更少的标签  

```ocaml
let is_positive = function
  | `Int   x -> x > 0
  | `Float x -> x > 0. ;;
val is_positive : [< `Float of float | `Int of int ] -> bool = <fun>
```

如果标签集既是上边界又是下边界，我们就得到了一个*确切的*多态变体类型，什么标记都没有。  

```ocaml
let exact = List.filter ~f:is_positive [three;four];;
val exact : [ `Float of float | `Int of int ] list = [`Int 3; `Float 4.]
```

创建来自不同上界和下界的变体类型  

```ocaml
let is_positive = function
  | `Int   x -> Ok (x > 0)
  | `Float x -> Ok (x > 0.)
  | `Not_a_number -> Error "not a number";;
val is_positive :
  [< `Float of float | `Int of int | `Not_a_number ] ->
  (bool, string) Result.t = <fun>
List.filter [three; four] ~f:(fun x ->
  match is_positive x with Error _ -> false | Ok b -> b);;
- : [< `Float of float | `Int of int | `Not_a_number > `Float `Int ] list =
[`Int 3; `Float 4.]
```



## 5. 再看终端颜色

现在需要在扩展原先的 `RGB`，加上一个 alpha 通道  

```ocaml
type extended_color =
  | Basic of basic_color * weight  (* basic colors, regular and bold *)
  | RGB   of int * int * int       (* 6x6x6 color space *)
  | Gray  of int                   (* 24 grayscale levels *)
  | RGBA  of int * int * int * int (* 6x6x6x6 color space *)
```

```ocaml
let extended_color_to_int = function
  | RGBA (r,g,b,a) -> 256 + a + b * 6 + g * 36 + r * 216
  | (Basic _ | RGB _ | Gray _) as color -> color_to_int color;;
(* Characters 154-159: Error: This expression has type extended_color but an expression was expected of type color *)
```

因为类型造成编译错误，`extended_color` 和 `color` 是两个不同的没有关系的类型  

我们想要做的就是在两个不同变体类型之间共享标签，而多态变体正好可以以一种自然的方式做到这一点  

```ocaml
let basic_color_to_int = function
  | `Black -> 0 | `Red     -> 1 | `Green -> 2 | `Yellow -> 3
  | `Blue  -> 4 | `Magenta -> 5 | `Cyan  -> 6 | `White  -> 7

let color_to_int = function
  | `Basic (basic_color,weight) ->
    let base = match weight with `Bold -> 8 | `Regular -> 0 in
    base + basic_color_to_int basic_color
  | `RGB (r,g,b) -> 16 + b + g * 6 + r * 36
  | `Gray i -> 232 + i;;

let extended_color_to_int = function
  | `RGBA (r,g,b,a) -> 256 + a + b * 6 + g * 36 + r * 216
  | (`Basic _ | `RGB _ | `Gray _) as color -> color_to_int color
```



---

**多态变体和 catch-all**

给 `match` 添加 `catch-all` 分支，得到了一个有下界的类型

```ocaml
let is_positive_permissive = function
  | `Int   x -> Ok (x > 0)
  | `Float x -> Ok (x > 0.)
  | _ -> Error "Unknown number type";;
val is_positive_permissive : [> `Float of float | `Int of int ] -> (bool, string)  Result.t = <fun>
is_positive_permissive (`Int 0);;
- : (bool, string) Result.t = Ok false
is_positive_permissive (`Ratio (3,4));;
- : (bool, string) Result.t = Error "Unknown number type"
```

而且 `catch-all` 的情况很容易出错  

```ocaml
is_positive_permissive (`Floot 3.5);;
```

把 `Float` 错误拼写成 `Floot`，仍然可以正常编译，但是隐含了一个 bug

---



把终端颜色的所有工作放到一起  

```ocaml
(* mli *)
open Core.Std

type basic_color =
  [ `Black   | `Blue | `Cyan  | `Green
  | `Magenta | `Red  | `White | `Yellow ]

type color =
  [ `Basic of basic_color * [ `Bold | `Regular ]
  | `Gray of int
  | `RGB  of int * int * int ]

type extended_color =
  [ color
  | `RGBA of int * int * int * int ]

val color_to_int          : color -> int
val extended_color_to_int : extended_color -> int
```

```ocaml
(* ml *)
open Core.Std

type basic_color =
  [ `Black   | `Blue | `Cyan  | `Green
  | `Magenta | `Red  | `White | `Yellow ]

type color =
  [ `Basic of basic_color * [ `Bold | `Regular ]
  | `Gray of int
  | `RGB  of int * int * int ]

type extended_color =
  [ color
  | `RGBA of int * int * int * int ]

let basic_color_to_int = function
  | `Black -> 0 | `Red     -> 1 | `Green -> 2 | `Yellow -> 3
  | `Blue  -> 4 | `Magenta -> 5 | `Cyan  -> 6 | `White  -> 7

let color_to_int = function
  | `Basic (basic_color,weight) ->
    let base = match weight with `Bold -> 8 | `Regular -> 0 in
    base + basic_color_to_int basic_color
  | `RGB (r,g,b) -> 16 + b + g * 6 + r * 36
  | `Gray i -> 232 + i

let extended_color_to_int = function
  | `RGBA (r,g,b,a) -> 256 + a + b * 6 + g * 36 + r * 216
  | `Grey x -> 2000 + x
  | (`Basic _ | `RGB _ | `Gray _) as color -> color_to_int color
```

`extended_color_to_int` 中 `Gray` 不小心拼写成 `Grey`，但是编译器没有任何问题  

如果给代码(ml)添加一个类型注释，那么编译会进行警告  

```ocaml
let extended_color_to_int : extended_color -> int = function
  | `RGBA (r,g,b,a) -> 256 + a + b * 6 + g * 36 + r * 216
  | `Grey x -> 2000 + x
  | (`Basic _ | `RGB _ | `Gray _) as color -> color_to_int color
```



一旦定义了类型，就可以重新审视窄化类型的模式匹配这个问题，可以显式使用模式匹配的一部分，加一个 `#` 前缀  

```ocaml
let extended_color_to_int : extended_color -> int = function
  | `RGBA (r,g,b,a) -> 256 + a + b * 6 + g * 36 + r * 216
  | #color as color -> color_to_int color
```



**何时使用多态变体**

多态变体绝对是普通变体的升级版。你可以做普通变体能做的任何事，还更灵活更简洁  

实际上，多数时候普通变体才是更实际的选择。因为多态变体的灵活性是有代价的  

- 复杂性，值层面上的简洁往往是牺牲了类型层面的复杂性
- 错误查找，多态类型是类型安全的，但是它们的类型规则，因其灵活性使它不容易捕捉你程序中的 bug
- 效率，这一点影响不是非常大，但多态变体会比普通变体重一些





# 附录A references

https://ocaml.org/learn/tutorials/index.zh.html  
《Real World OCaml》