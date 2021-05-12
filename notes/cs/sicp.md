带着崇敬和赞美，将本书献给活在计算机里的神灵。



# 一、构造过程抽象

**计算过程**是存在于计算机里的一类**抽象**事物，人们创造出称为**程序**，指导着操作一些被称为**数据**的抽象事物。

本书主要使用 Lisp 方言 scheme 编程语言， Lisp 是 20 世纪 50 年代的一种记法形式，为了能对递归方程的使用做推理，递归方程可以被作为计算模型。名字来源于 **Lis**t **P**rocessing。



## 1. 程序设计的基本元素

大多数强有力的语言提供的三种机制：

- 基本的表达式，最简单的单元

- 组合的方法，通过较简单的单元构造出复合的元素

- 抽象的方法，把复合对象命名，把它们当作单元去操作



在程序设计中，我们需要处理两类要素：

- 数据，我们希望去操作的“东西”

- 过程，操作这些数据规则的描述



Lisp 解释器读取(**R**ead) Lisp 表达式，对表达式求值(**E**val)，打印(**P**rint)所得结果，循环(**L**oop)，简称为 **REPL**，经典的交互式编程环境。

**表达式**：

```scheme
42         ; 42
(+ 1 2 3)  ; 6
(- 10 5)   ; 5
(* 5 9)    ; 45
(/ 10 3)   ; 10/3
```

Lisp 使用前缀表达式，好处是可以使用任意参数，可以之间扩充(组合式嵌套)。



**定义变量**：

```scheme
(define size 2)
size              ; 2
```

对一个组合式求值，需要：(1) 对各个子表达式求值；(2) 将作为最左运算符应用到相应的实际参数。该过程是**递归**的。



**过程定义**：

```scheme
;; (define (<name formal parameters>) <body>)
(define (square x) (* x x))
;; use
(square 21)      ; 441
(square (+ 2 5)) ; 49

;; x^2 + y^2
(define (sum_squares x y)
  (+ (square x) (square y)))

;; parameter(s) bind ~> std::bind in C++
;; (a+1)^2 + (a*2)^2
(define (f a)
  (sum_squares (+ a 1) (* a 2)))
```



**表达式和谓词**：

```scheme
;; (cond (<p1> <e1>) (<p2> <e2>) ... (<pn> <en>))
(define (abs x)
  (cond ((> x 0) x)
        ((= x 0) 0)
        ((< x 0) (- x))))

;; another
(define (abs x)
  (cond ((< x 0) (- x))
        (else x)))

;; yet another(if expression)
;; if <predicate> <consequent> <alternative>
(define (abs x)
  (if (< x 0)
     (- x)
     x))

;; complex predicate(and/or/not)
(and <e1> ... <en>)
(or <e1> ... <en>)
(not <e>)
```

依次检查谓词 p1, p2, ..., pn，直到发现某个谓词值为真，返回相应的表达式值。如果无法找到值为真的谓词，`cond` 的值就没有定义



[牛顿法](https://zh.wikipedia.org/wiki/%E7%89%9B%E9%A1%BF%E6%B3%95)求平方根实例：

```scheme
(define (sqrt_iter guess x)
  (if (good_enough guess x)
      guess
      (sqrt_iter (improve guess x) x)
  )
)

;; average(guess, x/guess)
(define (improve guess x)
  (average guess (/ x guess)
)

(define (average x y)
  (/ (+ x y) 2)
)
  
;; |guess^2 - x| < 0.001
(define (good_enough guess x)
  (< (abs (- (square guess) x)) 0.001)
)
  
(define (sqrt x)
  (sqrt_iter 1.0 x)
)
```



需要注意的一个问题(ex1.6 new-if)：

```scheme
(define (new-if predicate then-clause else-clause)
   (cond (predicate then-clause)
         (else else-clause)
   )
)
```

使用 `new-if` 替代 `sqrt_iter` 会造成栈溢出，因为 `if` 表达式仅仅会求值为真的分支，而 `new-if` 是一个普通函数，无论 `predicate` 真假，`then-clause` 和 `else-clause` 都会被求值。



## 2. 过程及产生的计算

**线性的递归和迭代**

阶乘问题 

递归：`n! = n * (n - 1)!` 

```scheme
(define (factorial n)
  (if (= n 1)
      1
      (* n (factorial (- n 1)))
   )
)
```

迭代：使用描述计算状态的**状态变量**、描述从一个状态到下一个状态的**转换规则**、**终结条件** 

```scheme
;; product <- counter * product
;; counter <- counter + 1
(define factorial n
  (fact_iter 1 1 n)
)

(define (fact_iter product counter max_count)
  (if (> counter max_count)
      product
      (fact_iter (* counter product)
                 (+ counter 1)
                 max_count)
  )
)
```



**树形递归**

斐波那契数列

```scheme
(define (fib n)
  (cond ((= n 0) 0)
        ((= n 1) 1)
        (else (+ (fib (- n 1)) 
                 (fib (- n 2))
              )
        )
  )
)
```

时间和空间复杂度和树的结点相关，$T(n) = T(n-1) + T(n-2)$，最后解得 $T(n) = \Theta((\dfrac{1+\sqrt5}{2})^n)≈1.618^n$ ，斐波那契数列递归解法复杂度是指数级别的。其迭代算法是线性算法。

换零钱方式统计问题，设总数为 `a`，零钱的种类为 `k`，则递归解法的空间复杂度为 $\Theta(a^k)$，详情见[这里](https://matrix-sicp.readthedocs.io/en/latest/ch1/ch1-2.html#id10)。



计算幂问题 $b^n$

- 使用 $b^n = b*b^{n-1}$ 算法，递归和迭代法都是线性复杂度
- 快速幂 $\begin{cases} b^n=(b^{n/2})^2, if\;n\;is\;even \\ b^n = b*b^{n-1}, if\;n\;is\;odd \end{cases}$，时间复杂度为 $\Theta(log n)$ 

...



## 3. 用高阶函数做抽象

**函数**就是一类抽象，描述了对于数据的复合操作，但并不依赖具体的数据。需要构造出一个函数，其以函数作为参数或返回值，这类函数被成为**高阶函数**。

考虑计算从 `a` 到 `b` 之间整数之和、立方和的问题

```scheme
(define sum_integers a b
  (if (> a b)
      0
      (+ a (sum_integers (+ a 1) b))
  )
)

(define sum_cubes a b
  (if (> a b)
      0
      (+ (cube a) (sum_cubes(+ a 1) b))
  )
)
```

可以提取此计算过程的出抽象的模式，重新使用高阶函数来实现

```scheme
(define (sum term a next b) 
  (if (> a b)
      0
      (+ (term a) (sum (next a) next b))
  )
)

(define (inc n) (+ n 1))
(define (cube n) (* n n n))
(define (sum_cubes a b) (sum cube a inc b))
```



考虑求 $\pi$ 的问题($\dfrac{\pi}{8}=\dfrac{1}{1*3}+\dfrac{1}{5*7}+\dfrac{1}{9*11}+ ...$)

```scheme
(define (pi_sum a b)
  (if (> a b)
      0
      (+ (/ 1.0 (* a (+ a 2)))
         (pi_sum (+ a 4) b)
      )
  )
)
```



使用 lambda 表达式+高阶函数重写求 $\pi$ 问题，lambda 形式为 `(lambda (<formal-parameters>) <body>)`

```scheme
(lambda (x) (+ x 4))
(lambda (x) (/ 1.0 (* x (+ x 2))))

(define (pi_sum a b)
  (sum (lambda (x) (/ 1.0 (* x (+ x 2))))
       a
       (lambda (x) (+ x 4))
       b
  )
)
```



`let` 创建局部变量的表达式，比如计算 $f(x,y)=x(1+xy)^2+y(1-y)+(1+xy)(1-y)$，可以令 $a=(1+xy)$，$b = (1-y)$，则 $f(x) = xa^2+yb+ab$ 

```scheme
(let ((<var1> <expr1>)
      (<var2> <expr2>)
      ...
      (<varn> <exprn>)
     )
  <body>
)

(define (f x y)
  (let ((a (+ 1 (* x y)))
        (b (- 1 y)))
    (+ (* x (square a))
       (* y b)
       (* a b)
    )
  )
)
```

计算 body 时 `let` 能在尽可能近 的地方建立局部变量约束，比如外部 `x` 为 5，则 `(+ (let ((x 3)) (+ x (* x 10))) x)` 为 38，计算 `let` 表达式得到 33，再加上外部的 `x` 为 38

`let` 约束变量的值是 `let` 之外计算的，比如外部 `x` 为 2，则 `(let ((x 3) (y (+ x 2))) (* x y))` 的值为 12，计算 `y` 使用了外部的 `x` 



给定一个函数 $f$，可以考虑计算 $x$ 和 $f(x)$ 的平均值的函数。

```scheme
(define (average_damp f)
  (lambda (x) (average x (f x)))
)

((average_damp square) 10) ;; => (10+100)/2 => 55
```

`average_damp` 是一个函数，它的参数是一个函数 `f`，返回值也是另一个函数。

程序设计中总会对计算元素的可能使用方式强加上某些限制，带有最少限制的元素被成为第一类(first class)状态，其包括的特权为：

- 可以使用变量命名
- 可以提供给函数作为参数
- 可以由函数作为结果返回
- 可以包含在数据结构中

Lisp 的函数为 First-class function。



# 二、构造数据抽象

讨论将对象组合起来，形成**复合数据**。复合数据的使用提供程序的模块性，提高程序设计语言的表达能力。

还有需要考虑的主要问题，将抽象作为克服复杂度的一种技术。数据抽象可以使我们在不同部分之间建立抽象屏障。

引进了**符号表达式**，进一步扩大了语言的表述能力。

为了维持模块性，处理一个程序的不同部分可能采用不同表示的数据的问题引出了实现**通用型操作**(类似于多态和泛型？)的需要，对面向数据的程序设计进行介绍。



## 1. 数据抽象

数据抽象的基本思想，就是构造出使用复合数据对象的程序，使他们就像在“抽象数据”上操作一样。

抽象数据上的两种接口函数是**构造函数**和**选择函数(selector，选择要为函数执行的方法)**，有点面向对象的味道了。

有理数算术的实例：

$\dfrac{n_x}{d_x} + \dfrac{n_y}{d_y} = \dfrac{n_x d_y + n_y d_x}{d_x d_y}$ ...

```scheme
; make_rat: construct a fraction
; numer:    return the numerator
; denon:    return the denominator
(define (add_rat x y)
  (make_rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x))
            )
            (* (denom x) (denom y))
  )
)
```

为了实现这一数据结构，使用**序对**的复合结构，可以通过 `cons` 构造一个序对，使用 `car` 和 `cdr` 来提取序对的一部分

```scheme
(define x (cons 1 2))
(car x) ; 1
(cdr x) ; 2
```

接下来使用序对来实现刚才有理数数据结构

```scheme
(define (make_rat n d) (cons n d))
(define (numer x) (car x))
(define (denom x) (cdr x))

; print a fraction
(define (print_rat x)
  (newline)
  (display (numer x))
  (display "/")
  (display (denom x))
)
```

上例使用**抽象屏障(封装)**，隔离了系统的不同层次，把使用数据抽象的程序与实现数据抽象的抽象分开。使得程序容易维护和修改。



看一个 `cons` 的合法实现：

```scheme
(define (cons x y)
  (define (dispatch m)
    (cond ((= m 0) x)
          ((= m 1) y)
          (else (error "Argument not 0 or 1 -- CONS" m))
    )
  )
  dispatch
)

(define (car z) (z 0))
(define (cdr z) (z 1))
```

这里的微妙之处是 `cons` 返回了一个函数，该函数根据参数 0 或 1 返回 x 或 y。这个实例说明可以将过程作为对象来操作，这种风格被称为 **消息传递**。



## 2. 层次性数据

实现序对的标准方式是以一种 box and pointer 的方式，每个对象表示为一个指向 box 的 pointer，box 中存放着包含着对象的表示。比如 `(cons 1 2)`

`cons` 序对的是一个方形 box，左边存在着序对 `car` 指针，右边存放着相应的 `cdr` 指针

```txt
         +---+---+       +---+
-------->|   |   |------>| 2 |
         +---+---+       +---+
           |
           v
         +---+
         | 1 |
         +---+
```



利用序对可以实现**链表**：

```scheme
(cons 1 (cons 2 (cons 3 nil)))
```



为了方便的链表的构造，提供了基本操作 `list`：

```scheme
(list <a1> <a2> ... <an>) ; <=> (cons <a1> (cons <a2> (cons ... (cons <an> nil) ... )))
(car alist)    ; a1
(cdr alist)    ; (a2 a3 ... an)

(define (list-ref alist n)
  (if (= n 0)
      (car alist)
      (list-ref (cdr alist) (- n 1))
  )
)

(define (length alist)
  (if (null? alist)
      0
      (+ 1 (length (cdr alist)))
  )
)

;; apply map to a list
(define (map proc alist)
  (if (null? alist)
      nil
      (cons (proc (car alist))
            (map proc (cdr alist))
      )
  )
)

;; e.g.
(define (scale_list alist factor)
  (map (lambda (x) (* x factor))
       alist)
)
(scale_list (list 1 2 3 4 5) 10)   ; (10 20 30 40 50)
```



**层次性结构**

也可以将 `list` 看作是 **树**，序列中的元素是树的分支。比如 `(cons (list 1 2) (list 3 4))` 可以看作树形结构：

```txt
  +---+---+      +---+---+      +---+---+
  |   |   | ---> | 3 |   | ---> | 4 |   | ---> nil
  +---+---+      +---+---+      +---+---+                         .
    |                                                           / | \
    v                                               ====>      .  3  4
  +---+---+      +---+                                        / \
  | 1 |   | ---> | 2 | ---> nil                              1   2
  +---+---+      +---+
```



求树的叶子节点数目：

```scheme
(define (count_leaves x)
  (cond ((null? x) 0)
        ((not (pair? x)) 1)
        (else (+ (count_leaves (car x))
                 (count_leaves (cdr x))
              )
        )
  )
)
```



在复杂的系统中，**分层设计**是广泛使用的设计方法。构造各个层次的方式，就是设法组合起作为这个层次中部件的各种基本元素，这样构成的部件又可以作为另一个层次中的基本元素。



## 3. 符号数据

之前所使用的复合数据，最终都是从数值出发构造的，这一节将引入任意符号作为数据。

比如：

```scheme
(define a 1)
(define b 2)

(list a b)     ; (1 2)
(list 'a 'b)   ; (a b)
(list 'a b)    ; (a 2)
(car '(a b c)) ; a
```

我们希望构造出表 `(a b)`，当然不能使用 `(list a b)` ，因为这个表达式是构造出 `a, b` **值**的链表，而不是 `a, b` **符号**本身的链表。

符号是变量的名字，可以使用单引号，其后的对象将被作为符号，可以将链表和符号标记都作为数据对象看待，而不是作为求值的表达式。

单引号也可以用于复合对象，可以通过 `'()` 得到空表

在符号对象上，定义了 `eq?` 操作，判断两个是否是相同的符号。

实现一个判断一个符号是否在链表中的过程，不存在返回 `false`，否则返回第一次出现的子表：

```scheme
(define (memq item alist)
  (cond ((null? alist) false)
        ((eq? item (car alist)) alist)
        (else (memq item (cdr alist)))
  )
)

;; e.g.
(memq 'apple '(pear banana prune)) ; false
(memq 'apple '(x (apple sauce) y apple pear)) ; (apple pear)
```



**对抽象数据进行求导**：

对于 $\dfrac{dc}{dx}=0$，$\dfrac{dx}{dx}=1$，$\dfrac{d(u+v)}{dx}=\dfrac{du}{dx}+\dfrac{dv}{dx}$，$\dfrac{d(uv)}{dx} = u(\dfrac{dv}{dx}) + v(\dfrac{du}{dx})$ 这几条规则进行求导

有下面的一些谓词：

```scheme
(define (variable? e) (symbol? e))
(define (same_variable? v1 v2) 
  (and (variable? v1) (variable? v2) (eq? v1 v2))
)

(define (make_sum a1 a2) (list '+ a1 a2))
(define (sum? e)
  (and (pair? e) (eq? (car e) '+))
)
(define (addend e) (cadr e))  ; e 的被加数，2th element
(define (augend e) (caddr e)) ; e 的加数，3th element

(define (make_product m1 m2) (list '* a1 a2))
(define (product? e)
  (and (pair? e) (eq? (car e) '*))
)
(define (multiplier e) (cadr e))
(define (multiplicand e) (caddr e))
```

求导过程：

```scheme
(define (deriv exp var)
  (cond ((number? exp) 0)
        ((variable? exp) 
         (if (same_variable? exp var)
             1                            ; dx/dx
             0                            ; dc/dx
         )
        )
        ((sum? exp) 
         (make_sum (deriv (addend exp) var)
                   (deriv (augend exp) var))
        )
        ((product? exp)
         (make_sum 
           (make_product (multiplier exp)
                         (deriv (multiplicand exp) var))
           (make_product (multiplicand exp)
                         (deriv (multiplier exp) var))
         )
        )
        (else (error "unknown expression type -- DERIV" exp))
  )
)

;; e.g. 
(deriv '(+ x 3) 'x)  ; (+ 1 0)
```



求导结果是未化简的，需要重写 `make_sum, make_product` 来化简结果：

```scheme
(define (make_sum a1 a2)
  (cond ((=number? a1 0) a2)
        ((=number? a2 0) a1)
        ((and (number? a1) (number? a2)) (+ a1 a2))
        (else (list '+ a1 a2))
  )
)

(define (make_product m1 m2)
  (cond ((=number? m1 0) 0)
        ((=number? m2 0) 0)
        ((=number? m1 1) m2)
        ((=number? m2 1) m1)
        ((and (number? m1) (number? m2) (* m1 m2)))
        (else (list '* m1 m2))
  ) 
)
```

`=number?` 检查一个表达式是否等于给定的数值。



**未排序集合的表示**：

一种方式是用某元素的链表，其中任何元素的出现都不超过一次。

```scheme
;; contains
(define (element_of_set? x set)
  (cond ((null? set) false)
        ((equal? x (car set)) true)
        (else (element_of_set? x (cdr set)))
  )
)

;; insert(head)
(define (adjoin_set x set)
  (if (element_of_set? x set)
      set
      (cons x set)
  )
)

(define (intersection_set set1 set2)
  (cond ((or (null? set1) (null? set2)) '())
        ((element_of_set? (car set1) set2)
         (cons (car set1) 
               (intersection_set (cdr set1) set2)
         )
        )
        (else (intersection_set (cdr set1) set2))
  )
)
```



**排序集合的表示**

```scheme
;; contains
(define (element_of_set? x set)
  (cond ((null? set) false)
        ((= x (car set)) true)
        ((< x (car set)) false)
        (else (element_of_set? x (cdr set)))
  )
)

(define (intersection_set set1 set2)
  (if (or (null? set1) (null? set2))
      '()
      (let ((x1 (car set1)) ((x2 (car set2))))
        (cond ((= x1 x2) (cons x1 (intersection_set (cdr set1) (cdr set2))))
              ((< x1 x2) (intersection_set (cdr set1) set2))
              ((< x2 x1) (intersection_set set1 (cdr set2)))
        )
      )
  )
)
```



## 4. 抽象数据的多重表示

使用**数据抽象**，可以将一个程序中大部分描述与所操作数据对象的具体表示的选择无关。比如之前的有理数例子，将操作有理数的操作和有理数对象的具体表示分隔开来，构成了**抽象屏障**。数据抽象屏障是控制复杂度的强有力工具。

而且需要去构造**通用型过程**，可以在不止一种数据表示上进行操作的过程。让它们在带有**类型标志**的数据对象上操作。



**复数的表示**：

复数可以以直角坐标系 `(real, image)` 或极坐标系 `(magnitude, angle)` 形式来表示

我们需要构造一个系统，使得这两种表示都适用。需要一种方式，将直角坐标和极坐标两种形式区分开，可以借用**类型标志**

在该实例中，用符号 `rectangular` 或 `polar` 来表示

```scheme
;; data with type_tag
(define (attach_tag type_tag contents) (cons type_tag contents))
(define (type_tag datum)
  (if (pair? datum)
      (car datum)
      (error "Bad tagged datum -- TYPE_TAG" datum)
  )
)

(define (contents datum)
  (if (pair? datum)
      (cdr datum)
      (error "Bad tagged datum -- CONTENTS" datum)
  )
)

;; z = x + yi
;; x(real) = rcos(A)
;; y(image) = rsin(A)
;; r aka. magnitude = sqrt(x^2 + y^2)
;; A = atan(y/x) <=> atan(y, x) in scheme

(define (rectangular? z) (eq? (type_tag z) 'rectangular))
(define (polar? z) (eq? (type_tag z) 'polar))

(define (square x) (* x x))

;; rectangular
(define (real_part_rectangular z) (car z))
(define (imag_part_rectangular z) (cdr z))
(define (magnitude_rectangular z)
  (sqrt (+ (square (real_part_rectangular z))
           (square (imag_part_rectangular z))
        )
  )
)

(define (angle_rectangular z)
  (atan (imag_part_rectangular z)
        (real_part_rectangular z)
  )
)

(define (make_from_real_imag_rectangular x y)
  (attach_tag 'rectangular (cons x y))
)

(define (make_from_mag_ang_rectangular r a)
  (attach_tag 'rectangular 
              (cons (* r (cos a)) (* r (sin a)))
  )
)

;; polar
(define (magnitude_polar z) (car z))
(define (angle_polar z) (cdr z))
(define (real_part_polar z)
  (* (magnitude_polar z) (cos (angle_polar z)))
)
(define (imag_part_polar z)
  (* (magnitude_polar z) (sin (angle_polar z)))
)

(define (make_from_mag_ang_polar r a)
  (attach_tag 'polar (cons r a))
)

(define (make_from_real_imag_polar x y)
  (attach_tag 'polar 
              (cons (sqrt (+ (square x) (square y))))
  )
)

(define (real_part z)
  (cond ((rectangular? z) (real_part_rectangular (contents z)))
        ((polar? z) (real_part_polar (contents z)))
        (else (error "Unknown type -- REAL_PART" z))
  )
)

(define (imag_part z)
  (cond ((rectangular? z) (imag_part_rectangular (contents z)))
        ((polar? z) (imag_part_polar (contents z)))
        (else (error "Unknown type -- IMAG_PART" z))
  )
)

(define (magnitude z)
  (cond ((rectangular? z) (magnitude_rectangular (contents z)))
        ((polar? z) (magnitude_polar (contents z)))
        (else (error "Unknown type -- MAGNITUDE" z))
  )
)

(define (angle z)
  (cond ((rectangular? z) (angle_rectangular (contents z)))
        ((polar? z) (angle_polar (contents z)))
        (else (error "Unknown type -- ANGLE" z))
  )
)

(define (make_from_real_imag x y) (make_from_real_imag_rectangular x y))
(define (make_from_mag_ang r a) (make_from_mag_ang_polar x y))

(define (add_complex z1 z2)
  (make_from_real_imag (+ (real_part z1) (real_part z2))
                       (+ (imag_part z1) (imag_part z2))
  )
)

; ...
```



上述的设计有一些缺点：

- 通用型接口(`real_part, imag_part, magnitude, angle`) 必须知道所有的类型标注
- 如果要添加一种表示方法，需要在每个接口过程中添加一个条件子句
- 各种不同的表示方法融合到整个系统，不能存在两个相同名称的函数，需要修改原先函数的名称



考虑使用数据导向的程序设计，以一种表格的形式：

|              |       polar        |          rectangular          |
| :----------: | :----------------: | :---------------------------: |
| `real_part`  | `real_part_polar`  |    `real_part_rectangular`    |
| `imag_part`  | `image_part_polar` |    `imag_part_rectangular`    |
| `mangnitude` | `mangnitude_polar` | `mangnitude_part_rectangular` |
|   `angle`    | `angle_part_polar` |   `angle_part_rectangular`    |

假定有两个过程操作这一表格：

`(put <op> <type> <item>)`

`(get <op> <type>)`

上述过程可以重构为：

```scheme
;; make_table
;; see https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-22.html#%_sec_3.3.3
(define (make-table)
  (let ((local-table (list '*table*)))
    (define (lookup key-1 key-2)
      (let ((subtable (assoc key-1 (cdr local-table))))
        (if subtable
            (let ((record (assoc key-2 (cdr subtable))))
              (if record
                  (cdr record)
                  false))
            false
        )
      )
    )
    (define (insert! key-1 key-2 value)
      (let ((subtable (assoc key-1 (cdr local-table))))
        (if subtable
            (let ((record (assoc key-2 (cdr subtable))))
              (if record
                  (set-cdr! record value)
                  (set-cdr! subtable
                            (cons (cons key-2 value)
                                  (cdr subtable)))))
            (set-cdr! local-table
                      (cons (list key-1
                                  (cons key-2 value))
                            (cdr local-table)))
        )
      )
      'ok
    )
    (define (dispatch m)
      (cond ((eq? m 'lookup-proc) lookup)
            ((eq? m 'insert-proc!) insert!)
            (else (error "Unknown operation - TABLE" m)))
    )
    dispatch
  )
)

(define operation-table (make-table))
(define get (operation-table 'lookup-proc))
(define put (operation-table 'insert-proc!))


;; data with type_tag
(define (attach_tag type_tag contents) (cons type_tag contents))
(define (type_tag datum)
  (if (pair? datum)
      (car datum)
      (error "Bad tagged datum -- TYPE_TAG" datum)
  )
)

(define (contents datum)
  (if (pair? datum)
      (cdr datum)
      (error "Bad tagged datum -- CONTENTS" datum)
  )
)

;; rectangular
(define (install_rectangular_package)
  ;; internal procedures
  (define (real_part z) (car z))
  (define (imag_part z) (cdr z))
  (define (make_from_real_imag x y) (cons x y))
  (define (magnitude z) 
    (sqrt (+ (square (real_part z)) (square (imag_part z))))
  )
  (define (angle z)
    (atan (imag_part z) (real_part z))
  )
  (define (make_from_mag_ang r a)
    (cons (* r (cos a)) (* r (sin a)))
  )

  ;; interface
  (define (tag x) (attach_tag 'rectangular x))
  (put 'real_part '(rectangular) real_part)         ; ^1
  (put 'imag_part '(rectangular) imag_part)
  (put 'magnitude '(rectangular) magnitude)
  (put 'angle '(rectangular) angle)
  (put 'make_from_real_imag 'rectangular            ; ^2
    (lambda (x y) (tag (make_from_real_imag x y)))
  )
  (put 'make_from_mag_ang 'rectangular 
    (lambda (r a) (tag (make_from_mag_ang r a)))
  )
)

;; polar

(install_rectangular_package)

(define (apply_generic op . args)         ; <~> varargs
  (let ((type_tags (map type_tag args)))  ; type_tags = map(type_tag, args)
    (let ((proc (get op type_tags)))      ; proc = get(op, type_tags)
      (if proc
        (apply proc (map contents args))  ; proc(map(contents, args))
        (error "No method for these types -- APPLY_GENERIC"
          (list op type_tags)
        )
      )
    )
  )
)

(define (real_part z) (apply_generic 'real_part z))
; other operator...
(define (make_from_real_imag x y)
  ((get 'make_from_real_imag 'rectangular) x y)
)

(define z (make_from_real_imag 3 4))
(display (real_part z))
```



关于上面代码 `install_rectangular_package` 标记 1 和 2 处，一个是 `'(rectangular)`，另一个是 `'rectangular`，对于各种操作来说，`key-2` 是 `args` 的 `type_tags` 列表，所以加了括号；对于构造函数而已，该处并不需要 `type_tags` 列表，仅仅一个标记类型即可，所以没有使用括号。

还可以改为消息传递风格的代码：

```scheme
(define (make_from_real_imag x y)
  (define (dispatch op)
    (cond ((eq? op 'real_part) x)
          ((eq? op 'imag_part) y)
          ((eq? op 'magnitude) (sqrt (+ (square x) (square y))))
          ((eq? op 'angle) (atan y x))
          (else (error "Unknown op -- MAKE_FROM_REAL_IMAG", op))
    )
  )
  dispatch
)
```



## 5. 通用型操作

第 4 节主要介绍了一种数据对象以多种方式来表示，使用通用的操作接口，将描述数据操作的代码连接到几种不同的表示上。

该节主要针对不同参数类型的通用型操作（类似泛型？）。比如 `add/sub/mul/div` 能应用到有理数、常规数值、复数上。

```scheme
;; scheme number package
(define (install_scheme_number_package)
  (define (tag x) (attach_tag 'scheme_number x))
  (put 'add '(scheme_number scheme_number) (lambda (x y) (tag (+ x y))))
  ; ...
  (put 'make 'scheme_number (lambda (x) (tag x)))
  'done
)

(install_scheme_number_package)

(define (make_scheme_number n)
  ((get 'make 'scheme_number) n)
)


;; rational package
(define (install_rational_package)
  ; internal
  (define (numer x) (car x))
  (define (demon x) (cdr x))
  (define (make_rat n d)
    (let ((g (gcd n d)))
      (cons (/ n g) (/ d g))
    )
  )

  (define (add_rat x y)
    (make_rat (+ (* (number x) (demon y))
                 (* (number y) (demon x))
              )
              (* (demon x) (demon y))
    )
  )
  ; ...
  
  ; interface
  (define (tag x) (attach_tag 'rational x))
  (put 'add '(rational rational) (lambda (x y) (tag (add_rat x y))))
  ; ...
  (put 'make 'rational ((lambda (n d) (tag (make_rat n d)))))
)

(install_rational_package)

(define (make_rational n d)
  ((get 'make 'rational) n d)
)

(define (add x y) (apply_generic 'add x y))
```



强制类型转换：

```scheme
(define (scheme_number->complex n)
  (make_complex_from_real_imag (contents n) 0)
)
```

接下来对类型的层次结构进行了讨论，有点类似继承？



# 三、模块化、对象、状态

前两章分别介绍了过程的抽象和数据的抽象，将基本过程和基本数据组合起来，构造出复合的实体。在构建大型系统的复杂问题上，抽象起着至关重要的作用。（可以分为不同的抽象层，将一个大问题，逐渐分层为不同的小问题。）

不过在设计程序中，我们还需要一些组织原则，能够指导我们完成系统的整体设计，特别是需要一些能够构造模块化大型系统的策略。（自然地划分一些具有内聚力的部分，使这些部分分别进行开发和维护）

这一章中，主要研究两种策略：

- 面向对象的程序设计：将注意力集中到**对象**上，将一个大型系统看成一批对象，它们的行为随着时间（状态）的改变而不断变化。
- 面向数据流的程序设计：将注意力集中到流过系统的**信息流**上。



## 1. 赋值和局部状态

在面向对象的程序设计中，一个系统由许多对象组成，这些对象极少是完全独立的。每个对象能够通过交互作用，影响其他对象的状态。每一个计算对象有自己的一些**局部状态变量**，用于描述对象的状态。

scheme 中的赋值运算符：

```scheme
(set! <name> <new-value>)
```

`begin` 表达式，按照 `exp1, exp2, ..., expk` 的顺序依次求值，最后 `expk` 的值作为 `begin` 的值返回：

```scheme
(begin <exp1> <exp2> ... <expk>)
```

可以使用 `begin` + `set!` 来实现赋值操作符返回赋值后的值的效果。



取钱实例：

```scheme
(define withdraw
  (let ((balance 100))
    (lambda (amount)
      (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds"
      )
    )
  )
)

(withdraw 25)  ; 75
(withdraw 25)  ; 50
(withdraw 60)  ; "Insufficient funds"
(withdraw 10)  ; 40
```

在语言中引入赋值，代换模型就不再适合作为过程应用的模型了（有了状态，也就是多次调用 `withdraw` 返回值不一样）。

代换模型是先求出各子表达式的值，找到要调用的过程的定义，用求出的实际参数代换过程体里的形式参数，再对过程体进行求值，本质上一种使用等价性的表达式的拆分机制

我们需要提出一个新的模型，这一模型包括对赋值语句和局部变量的解释。



接下来看几种取钱的变形：

```scheme
(define (make_withdraw balance)
  (lambda (amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds"
    )
  )
)

(define w1 (make_withdraw 100))
(define w2 (make_withdraw 100))

(w1 25)
(w2 25)
```

这个过程就类似于创建了两个对象，之后调用 `withdraw` 方法



实现取钱和存钱两个操作：

```scheme
(define (make_account balance)
  (define (withdraw amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds"
    )
  )

  (define (deposit amount)
    (set! balance (+ balance amount))
    balance
  )

  (define (dispatch m)
    (cond ((eq? m 'withdraw) withdraw)
          ((eq? m 'deposit) deposit)
          (else (error "Unknown request -- MAKE_ACCOUNT" m))
    )
  )
  dispatch
)

(define acc1 (make_account 100))
((acc 'withdraw) 50) ; 50
((acc 'deposit) 40)  ; 90
```



将赋值引入到程序设计语言，可以将系统看作一组带有局部状态的对象，也是一种维护模块化强有力的技术。

但是引入之后，不再能用代换模型解释了。

只要不用赋值操作，以同样参数对同一过程求值两次一定是同样的结果，不用任何赋值的程序设计被称为**函数式程序设计**。

从本质来说，代换的最终基础是，这一语言中的符号不过是作为值得名称，一旦引入和赋值和变量，一个变量就不再是简单的名称了。

如果一个语言支持在表达式里“同一的东西可以相互替换”的概念，这样的替换不会改变表达式的值，这个原因就称为**引用透明性**，包含赋值之后，也就打破了引用透明性。

与函数式编程语言相对应，广泛使用赋值、变量的是**命令式编程语言**。



## 2. 求值的环境模型

编程语言在引入赋值和变量的概念之后，代换模型就不再适合。此时一个变量必须以某种方式指定一个位置（内存地址），相应的值可以存储在那里，这种位置将维持在称为**环境**的结构中。

一个环境是框架（翻译为栈帧是不是更好？作用域）的的一个序列，每个框架中包含着一些**约束**的表格，这些约束将一个变量名称关联到相应的值，每个框架还包含一个指针，指向这个框架的**外围环境**（外围作用域）。如果一个框架没有外围环境，则称该框架是全局的。



## 3. 用变动数据做模拟

**变动的表结构**：

`set-car!` 接受两个参数，第一个参数是序对，将 `car` 指针指向第二个参数的地址，返回值依赖于实现。`set-cdr!` 类似。

比如 x 为 `((a b) c d)`，y 为 `(e f)`，使用 `(set-car! x y)`，x 结构变为 `((e f) c d)`  

`cons` 实现：

```scheme
(define (cons x y)
  (let ((new (get-new-pair)))
    (set-car! new x)
    (set-cdr! new y)
    new
  )
)
```



**共享和相等**：

考虑下面的例子：

```scheme
(define x (list 'a 'b))
(define z1 (cons x x))
```

`z1` 的 `car` 和 `cdr` 指针都指向（共享）同一个对象 `x`，之后修改 `x`，`z1` 的值会发生变化。



```scheme
(define z2 (cons (list 'a 'b) (list 'a 'b)))
```

虽然 `z1` 和 `z2` 的值相等，但是 `z2` 的 `car` 和 `cdr` 指针指向的两个不同的对象。



检查是否共享(两个符号是否相同或是否为同一对象或两个指针是否相等)的的一种方式是使用谓词 `eq?`，利用共享结构可以扩充表示数据结构的范围，但是引入共享也会带来危险。使用 `set-car!` 和 `set-cdr!` 需要小心，注意数据结构的共享情况。



**队列的表示**：

```scheme
(make-queue)            ; constructor
(define q (make-queue))

(empty-queue? <queue>)  ; empty
(front-queue <queue>)   ; front
(insert-queue! <queue> <item>) ; push
(delete-queue! <queue>)        ; pop
```

具体实现：

```scheme
(define (front_ptr queue) (car queue))
(define (rear_ptr queue) (cdr queue))

(define (set_front_ptr! queue item) (set-car! queue item))
(define (set_rear_ptr! queue item) (set-cdr! queue item))

(define (empty_queue? queue) (null? (front_ptr queue)))

(define (make_queue) (cons '() '()))
(define (front_queue queue)
  (if (empty_queue? queue)
    (error "FRONT called with an empty queue" queue)
    (car (front_ptr queue))
  )
)

(define (insert_queue! queue item)
  (let ((new_pair (cons item '() )))
    (cond ((empty_queue? queue)
            (set_front_ptr! queue new_pair)
            (set_rear_ptr! queue new_pair)
            queue
          )
          (else
            (set-cdr! (rear_ptr queue) new_pair)  ; insert to rear
            (set_rear_ptr! queue new_pair)
            queue
          )
    )
  )
)

(define (delete_queue! queue)
  (cond ((empty_queue? queue) 
          (error "DELETE! called with an empty queue" queue)
        )
        (else
          (set_front_ptr! queue (cdr (front_ptr queue)))
          queue
        )
  )
)

(define (print_queue queue)
  (display (car queue))
)
```



接下来还有表格数据结构的表示以及数字电路模拟器实例，这两个实例不再深入探讨。



**约束的传播**

计算机程序总被组织为一种单向的计算，对一些事先给定的参数执行某些操作，产出所需要的操作。

描述了一种语言设计，这种语言将基于各种关系进行工作，这一语言中基本元素就是**基本约束**，描述了不同量之间的某种特定关系。例如 `(adder a b c)` 描述 `a + b = c` 的约束关系，`(multiplier x y z)` 描述了 `xy = z` 的约束关系，`(constant 3.14 pi)`  描述了 `pi` 的值永远是 `3.14`。

scheme 提供了一种方法，可以构造**约束网络**，约束通过**连接器**连接起来。连接器是一种对象，可以保存一个值，使之能参与一个或多个约束。

比如华氏温度和摄氏温度之间的关系是：`9C = 5(F - 32)`

![约束网络表示关系 `9C=5(F-32)`](../../imgs/cs/sicp3_1.png)

```scheme
(define (inform-about-value constraint)
  (constraint 'I-have-a-value)
)

(define (inform-about-no-value constraint)
  (constraint 'I-lost-my-value)
)

(define (has-value? connector)
  (connector 'has-value?)
)

(define (get-value connector)
  (connector 'value)
)

(define (set-value! connector new-value informant)
  ((connector 'set-value!) new-value informant)
)

(define (forget-value! connector retractor)
  ((connector 'forget) retractor)
)

(define (connect connector new-constraint)
  ((connector 'connect) new-constraint)
)


(define (make-connector)
  (let ((value false)
        (informant false)
        (constraints '())
       )

    (define (set-my-value newval setter)
      (cond ((not (has-value? me))
             (set! value newval)
             (set! informant setter)
             (for-each-except setter inform-about-value constraints)
            )
            ((not (= value newval))
             (error "Contradiction" (list value newval))
            )
            (else 'ignored)
      )
    )

    (define (forget-my-value retractor)
      (if (eq? retractor informant)
        (begin (set! informant false)
               (for-each-except retractor inform-about-no-value constraints)
        )
        'ignored
      )
    )

    (define (connect new-constraint)
      (if (not (memq new-constraint constraints))
        (set! constraints (cons new-constraint constraints))
      )
      (if (has-value? me)
        (inform-about-value new-constraint)
      )
      'done
    )

    (define (me request)
      (cond ((eq? request 'has-value?)
              (if informant true false)
            )
            ((eq? request 'value) value)
            ((eq? request 'set-value!) set-my-value)
            ((eq? request 'forget) forget-my-value)
            ((eq? request 'connect) connect)
            (else (error "Unknown operation -- CONNECTOR" request))
      )
    )

    me
  )
)

(define (for-each-except exception procedure list)
  (define (loop items)
    (cond ((null? items) 'done)
          ((eq? (car items) exception) (loop (cdr items)))
          (else (procedure (car items)) (loop (cdr items)))
    )
  )

  (loop list)
)

(define (adder a1 a2 sum)
  (define (process-new-value)
    (cond ((and (has-value? a1) (has-value? a2))
           (set-value! sum (+ (get-value a1) (get-value a2)) me)
          )
          ((and (has-value? a1) (has-value? sum))
           (set-value! a2 (- (get-value sum) (get-value a1)) me)
          )
          ((and (has-value? a2) (has-value? sum))
           (set-value! a1 (- (get-value sum) (get-value a2)) me)
          )
    )
  )

  (define (process-forget-value)
    (forget-value! sum me)
    (forget-value! a1 me)
    (forget-value! a2 me)
    (process-new-value)
  )

  (define (me request)
    (cond ((eq? request 'I-have-a-value)
           (process-new-value)
          )
          ((eq? request 'I-lost-my-value)
           (process-forget-value)
          )
          (else (error "Unknown request -- ADDER" request))
    )
  )

  (connect a1 me)
  (connect a2 me)
  (connect sum me)
  me
)

(define (multiplier m1 m2 product)
  (define (process-new-value)
    (cond ((or (and (has-value? m1) (= (get-value m1) 0))
               (and (has-value? m2) (= (get-value m2) 0))
           )
           (set-value! product 0 me)
          )
          ((and (has-value? m1) (has-value? m2))
           (set-value! product (* (get-value m1) (get-value m2)) me)
          )
          ((and (has-value? m1) (has-value? product))
           (set-value! m2 (/ (get-value product) (get-value m1)) me)
          )
          ((and (has-value? m2) (has-value? product))
           (set-value! m1 (/ (get-value product) (get-value m2)) me)
          )
    )
  )

  (define (process-forget-value)
    (forget-value! product me)
    (forget-value! m1 me)
    (forget-value! m2 me)
    (process-new-value)
  )

  (define (me request)
    (cond ((eq? request 'I-have-a-value)
           (process-new-value)
          )
          ((eq? request 'I-lost-my-value)
           (process-forget-value)
          )
          (else (error "Unknown request -- MULTIPLIER" request))
    )
  )

  (connect m1 me)
  (connect m2 me)
  (connect product me)
  me
)

(define (constant value connector)
  (define (me request)
    (error "Unknown request -- CONSTANT" request)
  )
  (connect connector me)
  (set-value! connector value me)
  me
)

(define (probe name connector)
  (define (print-probe value)
    (newline)
    (display "Probe: ")
    (display name)
    (display " = ")
    (display value)
  )

  (define (process-new-value)
    (print-probe (get-value connector))
  )

  (define (process-forget-value)
    (print-probe "?")
  )

  (define (me request)
    (cond ((eq? request 'I-have-a-value)
           (process-new-value)
          )
          ((eq? request 'I-lost-my-value)
           (process-forget-value)
          )
          (else "Unknown request -- PROBE" request)
    )  
  )
  
  (connect connector me)
  me
)

(define (celsius_fahrenheit_converter c f)
  (let ((u (make-connector))
        (v (make-connector))
        (w (make-connector))
        (x (make-connector))
        (y (make-connector))
       )
    (constant 9 w)
    (multiplier c w u)  ; 9c = u
    (constant 5 x)
    (constant 32 y)
    (adder v y f)       ; v + 32 = f => v = f - 32
    (multiplier x v u)  ; 5v = u => 5(f-32) = 9c
    'ok
  )
)

(define C (make-connector))
(define F (make-connector))
(celsius_fahrenheit_converter C F)

(probe "Celsius temp" C)
(probe "Fahrenheit temp" F)

(set-value! C 25 'user)
```

上述小型的约束系统，进一步对消息传递风格进行了灵活的应用，类似于回调的方式进行编程

首先构建约束关系(constant、adder、multiplier 等)，使用 `connect` 把 `connector` 和某个约束(过程)连接起来

当对一个 `connector` 设置值时，就会调用 `connector` 的 `set-my-value` 过程，如果有值且和新值不相等，报出冲突的错误。如果没有值就设置新值，然后遍历 `constraints`，使用 `inform-about-value` 对每个约束进行通知(向每个过程发送 `'I-have-a-value'` 消息)，然后调用 `process-new-value` 进行处理出现新值的情况。失去值也是相似的过程。



## 4. 并发：时间是一个本质问题

注：concurrency 中文版书这里翻译为并发，但是依据下文，构建并发并不太恰当，觉得并发和并行应该更恰当一些，但是下文并没有修改原文的并发。

使用具有内部状态的计算对象作为模拟工具有非常大的威力，但是也丢失了引用透明性，抛弃了代换模型

引入赋值之后，就必须得承认时间在所用的计算模型中的位置。赋值语句的执行描绘出有关值变化的一些时刻，对一个表达式的求值结果，不但依赖于该表达式本身，还依赖于在这些时刻之前还是之后。

在现实世界里的对象并不是一次一个地发生变化，我们看到它们总是并发的。在并发的情况下，由赋值引入的复杂性问题将变得更加严重了。

几个进程或线程共享同一状态变量。多个进程或线程可能同时试图去读写这种共享的状态。



对并发的可能的限制是：

- 修改任意共享状态变量的两个操作不能同时发生。这样做可能会严重影响效率。
- 保证并发系统产生的结果和各个进程按照某种方式顺序运行出的结果完全一致。



设计并发系统时，对于某种由顺序要求的事件可以使用**串行化**方式来解决

scheme 提供了一些机制处理并行以及串行：

```scheme
(parallel-execute <p1> <p2> ... <pn>)

; e.g.
(define x 10)
(parallel-execute (lambda () (set! x (* x x)))
                  (lambda () (set! x (+ x 1)))
)

; possible x = 11, 100, 101, 110, 121
```

```scheme
(make-serializer)

(define x 10)
(define s (make-serializer))
(parallel-execute (s (lambda () (set! x (* x x))))
                  (s (lambda () (set! x (+ x 1))))
)
; possible x = 101, 121
```

使用 `serializer`，其中 `p1` 和 `p2` 不会交错执行，要么先 `p1` 后 `p2`，要么先 `p2` 后 `p1`，最后只有两种结果

`serializer` 通过 `mutex` 实现

```scheme
(define (make-serializer)
  (let ((mutex (make-mutex)))
    (lambda (p)
      (define (serializer-p . args)
        (mutex 'acquire)
        (let ((val (apply p args)))
          (mutex 'release)
          val
        )
      )

      serializer-p
    )
  )
)
```



但是需要注意死锁问题，以某种特定顺序执行避免死锁



## 5. 流

使用**流**数据结构，探索对状态模拟的另一条途径。

考虑以数学函数思考这个问题，可以将一个量 `x` 随着时间的变化的行为，描述为一个时间函数 `x(t)`，需要关注的是这些值的整个时间史，不需要强调其中的变化，因为函数本身没有变化。用离散的步长去度量时间。就可以用一个有穷的序列去模拟时间函数。流就是一个序列。引入一种**延迟求值**的方法，能够用流表示非常长的序列。

**流作为延迟求值的链表**

序列可以作为组合程序的一种标准接口，已经对序列构造出功能强大的抽象机制，比如 `map, filter, accumulate`

但是如果将序列表示为链表，获得这些优雅的结果将付出严重的性能代价，因为工作流程的每一步，都需要构造和复制各种数据结构。

比如：

```scheme
(define (sum_primes1 a b)
  (define (iter count accum)
    (cond ((> count b) accum)
          ((prime? count) (iter (+ count 1) (+ count accum)))
          (else (iter (+ count 1) accum))
    )
  )

  (iter a 0)
)

(define (sum_primes2 a b)
  (accumulate +
              0
              (filter prime? (enumerate-interval a b))
  )
)
```

第二个程序需要构造出区间的所有整数的序列，之后用 `filter` 过滤，最后再把过滤结果累加。

流使我们使用各种序列操作，但又不会带来将序列作为链表去操作而引起的代价。

从表面上看，流也是链表。但是对它们操作的过程的名称不同：

```scheme
(cons-stream <a> <b>)

;; <=>
(cons <a> (delay <b>))

(define (stream-car stream) (car stream))
(define (stream-cdr stream) (force (cdr stream)))
```

流的实现基于 `delay` 的特殊形式，对于 `(delay <exp>)` 表达式，将不会立即对 `exp` 进行求值，而是返回一个延迟求值的对象，可以看作在未来的某个时间求值 `exp` 的允诺。和 `delay` 相对应的是 `force`，也就是迫使 `delay` 表达式完成它允诺的求值。

空的流对象为 `the-empty-stream`，在 MIT-scheme 中实现为 `'()`，这个对象可以使用 `stream-null?` 进行判断。

在流上的操作：

```scheme
(define (stream-ref s n)  ; s[n]
  (if (= n 0)
      (stream-car s)
      (stream-ref (stream-cdr s) (- n 1))
  )
)

(define (stream-map proc s)
  (if (stream-null? s)
      the-empty-stream
      (cons-stream (proc (stream-car s))
                   (stream-map proc (stream-cdr s))
      )
  )
)

(define (stream-for-each proc s)
  (if (stream-null? s)
      'done
      (begin (proc (stream-car s))
             (stream-for-each proc (stream-cdr s))
      )
  )
)
```



再回到刚刚求区间内求素数的程序：

```scheme
(stream-car 
  (stream-cdr
    (stream-filter prime? (stream-enumerate-interval a b))
  )
)

(define (stream-enumerate-interval a b)
  (if (> a b)
      the-empty-stream
      (cons-stream a (stream-enumerate-interval (+ a 1) b))
  )
)

(define (stream-filter pred stream)
  (cond ((stream-null? stream) the-empty-stream)
        ((pred (stream-car stream))
          (cons-stream (stream-car stream)
                       (stream-filter pred (stream-cdr stream))
          )
        )
        (else (stream-filter pred (stream-cdr stream)))
  )
)
```



`delay` 和 `force` 的实现：

```scheme
(define (memo-proc proc)
  (let ((already-run? false) (result false))
    (lambda () 
      (if (not already-run?)
        (begin (set! result (proc))
               (set! already-run? true)
               result
        )
        result
      )
    )
  )
)

(define (delay exp)
  (memo-proc (lambda () exp))
)

(define (force delayed-object)
  (delayed-object)
)
```

`delay` 返回了一个过程(`delayed-object`)，对 `delayed-object` 使用 `force` 时，就相当于调用 `delay` 产生的过程



**无穷流**

例如正整数流：

```scheme
(define (integers-starting-from n)
  (cons-stream n (integers-starting-from (+ n 1)))
)

(define integers (integers-starting-from 1))
```

`integers` 是一个流序对，其中 `car` 是 1，其 `cdr` 为从 2 开始的流序对，这是一个无穷长的流。

我们可以定义其他的无穷流，比如不能被 7 整除的正整数流：

```scheme
(define (divisible? x y) (= (remainder x y) 0))

(define no-sevens
  (stream-filter (lambda (x) (not (divisible? x 7)))
                 integers
  )
)
```

斐波那契数列流：

```scheme
(define (fibgen a b)
  (cons-stream a (fibgen b (+ a b)))
)

(define fibs (fibgen 0 1))
```



**隐式定义流**、

上面的过程都是通过描述生存过程来定义流。还可以通过延时求值隐式定义流。比如：

```scheme
(define ones (cons-stream 1 ones))
```

还可以通过 `add-streams` 等操作，可以做到一些有趣的事情：

```scheme
(define (add-stream s1 s2)
  (stream-map (+ s1 s2))
)

(define integers (cons-stream 1 (add-stream ones integers)))

(define fibs
  (cons-stream 0
               (cons-stream 1
                            (add-stream (stream-cdr fibs)
                                        fibs)
               )
  )
)
```



带有延时求值的流可能成为一种强大的模拟工具， 能够提供局部状态和赋值的许多效益，还避免将引入赋值所带来的一些理论困难。

可以将反复的迭代操作表示为流操作，比如原来的牛顿法求平方根、迭代求 pi 值。



**函数式程序的模块化和对象的模块化**

引入赋值的主要收益是增强系统的模块化，把一个大的系统某些部分进行封装（隐藏到局部变量中）。

流模型可以提供等价的模块化，同时又不需要赋值。

流为模拟具有内部状态的对象提供了另一种方式，用**流去模拟一个变化的量**。





# 四、元语言抽象

为了设法控制设计的复杂度，使用抽象技术。将基本元素组合起来，形成复合元素，从复合元素出发通过抽象形成更高一层的构件，并通过采取某些适当的关于系统结构的观点，保持系统的模块性。

为了阐述这些技术，之前的内容一直使用 Lisp 作为语言来描述计算过程。然而随着问题变得更加复杂，发现 Lisp 以及任何一种确定的程序语言都不足以满足我们的需求。需要转向新的语言，以便能有效表达自己的想法。通常使用一种新语言提升处理复杂问题的能力。

程序设计需要涉及的多种语言：

- 机器语言，数据和指令的一系列二进制表示
- 高级语言，构筑在机器语言之上，提供了一些组合和抽象机制，更适合大规模系统组织



**元语言抽象**就是建立新的语言，在程序设计领域非常重要。不仅能设计新的语言，还可以通过构造**求值器（解释器）**的方式实现这些语言。

程序设计中最基本的一点思想：求值器决定了一个程序设计语言中各种表达式的意义，而且它本身也不过是另一个程序。



这一章讨论在一些语言的基础上构造新的语言的技术。将使用 Lisp 为基础，将各种求值器实现为一些 Lisp 过程。

第一步构造一个针对 Lisp 本身的求值器。对这个求值器进行了一个简化。



## 1. 元循环求值器

把 Lisp 求值器（解释器）实现为一个 Lisp 程序，看似是一个循环矛盾(鸡生蛋蛋生鸡)定义。这种做法叫做**元循环**（**自举**）

元循环求值器模型：

- 求值一个组合式，首先求值其中的子表达式，然后将子表达式的值用于计算组合式
- 将一个复合过程应用于一系列的实际参数时，将一个新的环境里计算这个函数的函数体。

求值器的实现依赖于一些定义了被求值表达式的语法形式的过程。



**求值器内核**

求值过程可以描述为 `eval` 和 `apply` 之间的相互作用。

`eval` 的参数是一个表达式和一个环境，`eval` 对表达式进行分类，针对每类表达式都有一个谓词完成相应的检测，有一套抽象方法选择表达式的各个部分。

对于各种基本表达式：

- 对于自求值的表达式，例如数值字面量，之间返回这个表达式本身
- `eval` 在环境中查找变量，返回它们的值

特殊表达式：

- 对于符号表达式，返回被引的表达式（符号）
- 对于变量的赋值，需要递归调用 `eval` 计算出需要关联于这个变量的新值，并修改环境
- `if` 表达式根据谓词真假返回不同的部分
- `lambda` 表达式会转换为一个可以应用的过程，方式就是将 `lambda` 表达式所描述的参数列表和函数体与相应的求值环境包装起来
- `begin` 表达式按顺序求值其中的一系列表达式，并返回最后一个表达式的值
- `cond` 表达式被转换为一组嵌套的 `if` 表达式，而后求值
- 过程调用表达式，递归使用 `eval` 计算实际参数，然后将过程和参数送入 `apply`



`eval` 和 `apply`的实现：

```scheme
(define (eval exp env)
  (cond ((self-evaluating? exp) exp)
        ((variable? exp) (lookup-variable-value exp env))
        ((quoted? exp) (text-of-quotation exp))
        ((assignment? exp) (eval-assignment exp env))
        ((definition? exp) (eval-definition exp env))
        ((if? exp) (eval-if exp env))
        ((lambda? exp) (make-procedure (lambda-parameters exp) 
                                       (lambda-body exp)
                                       env
                       )
        )
        ((begin? exp) (eval-sequence (begin-actions exp) env))
        ((cond? exp) (eval (cond->if exp) env))
        ((application? exp)
          (apply (eval (operator exp) env)
                 (list-of-values (operands exp) env)
          )
        )
        (else (error "Unknown expression type -- EVAL" exp))
  )
)

(define (apply procedure arguments)
  (cond ((primitive-procedure? procedure)
          (apply-primitive-procedure procedure arguments))
        ((compound-procedure? procedure)
          (eval-sequence
            (procedure-body procedure)
            (extend-environment
              (procedure-parameters procedure)
              arguments
              (procedure-environment procedure)
            )
          )
        )
        (else (error "Unknown procedure type -- APPLY" procedure))
  )
)
```

在 `eval` 实现中，使用 `cond` 分情况对各种类型的表达式进行分析。缺点是如果加入新的表达式类型，必须编辑 `eval` 的定义。在大部分的 Lisp 实现中，对表达式类型使用了数据导向方式，增加新的表达式类型，不必修改 `eval` 实现。



处理过程参数：

```scheme
(define (list-of-values exps env)
  (if (no-operands? exps)
    '()
    (cons (eval (first-operand exps) env)
          (list-of-values (rest-operands exps) env)
    )
  )
)
```

`if` 表达式：

```scheme
(define (eval-if exp env)
  (if (true? (eval (if-predicate exp) env))
    (eval (if-consequent exp) env)
    (eval (if-alternative exp) env)
  )
)
```

`begin` 表达式：

```scheme
(define (eval-sequence exps env)
  (cond ((last-exp? exps) (eval first-exp exps) env)
        (else (eval (first-exp exps) env)
              (eval-sequence (rest-exps) env)
        )
  )
)
```

赋值和定义表达式：

```scheme
(define (eval-assignment exp env)
  (set-variable-value! (assignment-value exp)
                       (eval (assignment-value exp) env) 
                       env
  )
  'ok
)

(define (eval-definition exp env)
  (definition-variable! (definition-variable exp)
                        (eval (definition-value exp) env)
                        env
  )
  'ok
)
```



**表达式的表示**：

`quote`

```scheme
;; quote
;  'a => (quote a)
(define (quoted? exp) (tagged-list? exp 'quote))
(define (text-of-quotation exp) (cadr exp))
```



`set!`

```scheme
;; set!
;  (set! <var> <value>)
(define (assignment? exp) (tagged-list? exp 'set!))
(define (assignment-variable exp) (cadr exp)) ; (cadr exp) => var
(define (assignment-value exp) (caddr exp)) ; (caddr exp) => value
```



`define`

```scheme
;; define
;  case1: (define <var> <value>) or
;  case2: (define (<var> <param-1> ... <param-n>) <body>)
(define (definition? exp)
  (tagged-list? exp 'define)
)

(define (definition-variable exp)
  (if (symbol? (cadr exp))  ; (car (cdr exp)) => var or (var params...)
      (cadr exp)            ; case1
      (caadr exp)           ; case2
  )
)

(define (definition-value exp)
  (if (symbol? (cadr exp))
      (caddr exp)               ; case1: value
      (make-lambda (cdadr exp)  ; case2: formal parameters (param-1 ... param-n)
                   (cddr exp)   ; case2: body
      )
  )
)
```



`lambda`

```scheme
;; lambda
;  (lambda (param-1 ... param-n) <body>)

(define (lambda? exp)
  (tagged-list? exp 'lambda)
)

(define (lambda-parameters exp)
  (cadr exp)
)

(define (lambda-body exp)
  (cddr exp)
)

(define (make-lambda parameters body)
  (cons 'lambda (cons parameters body))
)
```



`if`

```scheme
;; if
;  (if predicate <consequent> [alternative])
(define (if? exp)
  (tagged-list? exp 'if)
)

(define (if-predicate exp)
  (cadr exp)
)

(define (if-consequent exp)
  (cddr exp)
)

(define (if-alternative exp)
  (if (not (null? cdddr exp))
      (cadddr exp)
      'false
  )
)

(define (make-if predicate consequent alternative)
  (list 'if predicate consequent alternative)
)
```



`begin`

```scheme
;; begin
;  (begin seq...)
(define (begin? exp)
  (tagged-list? exp 'begin)
)

(define (begin-actions exp)
  (cdr exp)
)

(define (last-exp? seq)
  (null? (cdr exp))
)

(define (first-exp seq)
  (car seq)
)

(define (rest-exp seq)
  (cdr seq)
)

(define (sequence->exp seq)  ; constructor
  (cond ((null? seq) seq)
        ((last-exp? seq) first-exp seq)
        (else (make-begin seq))
  )
)

(define (make-begin seq)
  (cons 'begin seq)
)
```



过程应用不属于上述各种表达式类型的复合表达式

```scheme
;; application
(define (application? exp) (pair? exp))
(define (operator exp) (car exp))
(define (operands exp) (cdr exp))
(define (no-operands? ops) (null? ops))
(define (first-operands ops) (car ops))
(define (reset-operands ops) (cdr ops))
```



派生的 `cond->if`、`let->lambda` 等表达式代码略



**解释器的数据结构**：

谓词检测，除了 `false` 对象之外的所有都为 `true`

```scheme
(define (true? x)
  (not (eq? x false))
)

(define (false? x)
  (eq? x false)
)
```



环境是一个框架（帧？）的链表，一个环境的外围环境就是表的 `cdr`，每个框架都是一对链表形成的序对，一个是变量的表，另一个是约束值得表

```scheme
(define (enclosing-environment env) (cdr env))
(define (first-frame env) (car env))
(define the-empty-environment '())

(define (make-frame variables values) (cons variables values))
(define (frame-variables frame) (car frame))
(define (frame-values frame) (cdr frame))
(define (add-binding-to-frame! var value frame)
  (set-car! frame (cons var (car frame)))
  (set-cdr! frame (cons value (cdr frame)))
)

(define (extend-environment vars vals base-env)
  (if (= (length vars) (length vals))
      (cons (make-frame vars vals) base-env)
      (if (< (length vars) (length vals))
          (error "Too many arguments supplied" vars vals)
          (error "Too few arguments supplied" vars vals)
      )
  )
)

(define (lookup-variable-value var env)
  ; ...
)

(define (set-variable-value! var val env)
  ; ...
)
```



**作为程序运行这个解释器**

设置全局环境

```scheme
(define (setup-environment)
  (let ((initial-env 
         (extend-environment (primitive-procedure-names) 
                        (primitive-procedure-objects)
                        the-empty-environment)))

    (define-variable! 'true true initial-env)
    (define-variable! 'false false initial-env)
    initial
  )
)

(define the-global-environment (setup-environment))

; primitive-procedure
(define (primitive-procedure? proc) (tagged-list? proc 'primitive))
(define (primitive-implementation proc) (cadr proc))
(define primitive-procedures
  (list (list 'car car)
        (list 'cdr cdr)
        (list 'cons cons)
        (list 'null? null?)
        ; ...
  )
)

(define (primitive-procedure-names)
  (map car primitive-procedures)
)

(define (primitive-procedure-objects)
  (map (lambda (proc) (list 'primitive (cadr proc)))
       primitive-procedures
  )
)

(define (apply-primitive-procedure proc args)
  (apply-in-underlying-scheme (primitive-implementation proc) args)
)
```

主循环

```scheme
;; main loop
(define input-prompt "M-Eval input:")
(define output-prompt "M-Eval value:")

(define (driver-loop)
  (prompt-for-input input-prompt)
  (let ((input (read)))
    (let ((output (eval input the-global-environment)))
      (announce-output output-prompt)
      (user-print output)
    )
  )

  (driver-loop)
)

(define (prompt-for-input string) (newline) (display string) (newline))
(define (announce-output string) (newline) (display string) (newline))

(define the-global-environment (setup-environment))
(driver-loop)
```



**内部定义**

```scheme
(let (a 1)
  (define (f x)
    (define b (+ a x))
    (define a 5)
    (+ a b)
  )
  (f 10)
)
```

关于上面得程序有三种解释：

- 1 按照 `define` 的顺序进行，`b` 为 11，`a` 为 5，最后结果是 16
- 2 相互递归要求内部过程定义的同时性作用域规则，过程和变量名规则不应该不同。（过程不像 C 语言那样非得前向声明，因为内部过程的同时作用域规则）。将会导致在计算 b 时，`a` 还没有赋值导致错误
- 3 `a` 和 `b` 的定义是同时进行的，`a` 被置为 5，`b` 被置为 15，最后的结果为 20

MIT-scheme 实现为第二种，从原则上第三种是正确的，定义应该是同时的，但是又很难有一种机制实现第三种，所以在遇到困难的同时定义时产生一个错误

由于内部变量表面上看上去是顺序的，实际上是同时的，就希望避免这种情况，于是采用 `letrec`。

```scheme
(letrec ((<var1> <exp1>) ... (varn expn)) <body>)
```

这些表达式求值都是在一个包含了所有 `letrec` 约束的环境中完成的，这就允许了约束中的递归



**将语法分析与执行分离**

```scheme
(define (eval exp env)
  ((analyze exp) env)
)

(define (analyze exp)
  (cond ((self-evaluating? exp) (analyze-self-evaluating exp))
        ((quoted? exp) (analyze-quoted exp))
        ; ...
        (else "Unknown expression type -- ANALYZE" exp)
  )
)

(define (analyze-self-evaluating exp)
  (lambda (env) exp)
)

(define (analyze-quoted exp)
  (let ((qval (text-of-quotation exp)))
    (lambda (env) qval)
  )
)

; other analysis
```





## 2. 惰性求值

**正则序与应用序**

采用**应用序**的语言在过程应用（函数调用）的时候，提供给过程的所有参数都需要完成求值。scheme 是应用序的。

采用**正则序**的语言将把过程参数的求值延后到需要这些实际参数的值的时候，将过程参数求值拖到最后的可能时刻。也被叫做惰性求值。

比如

```scheme
(define (try a b)
  (if (= a 0) 1 b)
)

(try 0 (/ 1 0))
```

对于上面的程序，应用序将产生一个错误，但是对于惰性求值不会



如果某个参数还没有完成求值之前就进入一个过程体，说这一过程相对于该参数是**非严格的**。

如果在进入过程体之前，某个参数已经完成求值，说该过程相当于这个参数是**严格的**。

严格和非严格是应用序和正则序是同样的意思。



**采用惰性求值的解释器**

这一节要实现和 scheme 一样的正则序语言，但是其中复合过程对任何参数都是非严格的，基本过程仍然是严格的

基本想法是，在应用过程时，解释器必须缺点哪些参数需要求值，哪些应该延时求值。对于这些需要延时求值的参数不进行求值，这里把它们变换为一种称为 thunk 的对象，在 thunk 里包含着产生这一参数值所有的信息

对 thunk 中的表达式求值的过程称为 force(强迫)，现在要处理的一个选择是采用或不采用记忆性的 thunk



**修改解释器**

原先处理 apply 的部分

```scheme
((application? exp)
  (apply (eval (operator exp) env)
         (list-of-values (operands exp) env)
  )
)
```

现在变成了 

```scheme
((application? exp)
  (apply (actual-value (operator exp) env)
         (operands exp)
         env
  )
)

(define (actual-value exp env)
  (force-it (eval exp env))
)
```

对于惰性求值，使用运算对象表达式去调用 apply，而不是用求值产生的实际参数

```scheme
(define (apply procedure arguments)
  (cond ((primitive-procedure? procedure)
          (apply-primitive-procedure
            procedure
            ; arguments 
            (list-of-arg-values arguments env)   ; changed
          )
        )
        ((compound-procedure? procedure)
          (eval-sequence
            (procedure-body procedure)
            (extend-environment
              (procedure-parameters procedure)
              ; arguments
              (list-of-delayed-args arguments env) ; changed
              (procedure-environment procedure)
            )
          )
        )
        (else (error "Unknown procedure type -- APPLY" procedure))
  )
)

(define (list-of-arg-value exps env)
  (if (no-operands? exp)
    '()
    (cons (action-value (first-operand exps) env)
          (list-of-arg-value (rest-operands exps) env)
    )
  )
)

(define (list-of-delayed-args exps env)
  (if (no-operands exps)
    '()
    (cons (dealy-it (first-operand exps) env)
          (list-of-delayed-args (rest-operands exps) env)
    )
  )
)
```

另一个需要修改的地方是对 `if` 的处理

```scheme
(define (eval-if exp env)
  (if (true? (actual-value (if-predicate exp) env))
      (eval (if-consequent exp) env)
      (eval (if-alternative exp) env)
  )
)
```

最后是主循环部分

```scheme
(define input-prompt "L-Eval input:")
(define value-prompt "L-Eval value:")
(define (driver-loop)
  (promt-for-input input-prompt)
  (let ((input (read)))
    (let ((output (actual-value input the-global-environment)))
      (announce-output output-prompt)
      (user-print output)
    )
  )
  (driver-loop)
)
```



**thunk 的表示**

```scheme
(define (force-it obj)
  (if (thunk? obj)
      (action-value (thunk-exp obj) (thunk-env obj))
      obj
  )
)

(define (delay-it exp env)
  (list 'thunk exp env)
)

(define (thunk? obj)
  (tagged-list? obj 'thunk)
)

(define (thunk-exp thunk) (cadr thunk))
(define (thunk-env thunk) (caddr thunk))
```



thunk 还可以带有记忆功能

有了惰性操作之后，流和惰性操作的链表就是一样的



## 3. 非确定性计算

这一节扩展 scheme 解释器，以便支持另一种称为非确定性计算的程序设计范式。

非确定性计算与流处理类似，对于“生成和检测式”的应用特别有价值。考虑下面的例子，一对正整数链表，从中生成出一对整数，其中第一个取自第一个链表，第二个来自第二个，它们之和是素数。之前所采用的方法是生成所有可能的数对，而后过滤出和为素数的数对。

```scheme
(define (prime-sum-pair list1 list2)
  (let ((a (an-element-of list1))
        (b (an-element-of list2))
       )
    (require (prime? (+ a b)))
    (list a b)
  )
)
```

看起来是问题的一个重新描述，而没有描述一种解决它的一个办法，但这就是一个合法的非确定性程序。

在非确定性语言中，关键想法是，表达式可以有多于一个可能的值，例如 `an-element-of` 可能返回给定表中的任何一个元素。非确定性程序解释器将进行有关的工作，从中自动选出一个可能的值，并维持有关选择的轨迹，如果随后的要求无法满足，解释器就会自动选择另一种不同的选择，并且会不断选择出新的选择，直到求值成功或用光了所有的选择。

这种解释器支持这样一种错觉，好像所有可能的结果以一种无事件顺序的方式摆在我们面前。

下面要实现的非确定性解释器被称为 `amb` 解释器



为了扩充 scheme 支持非确定性，引入 `amb` 的特殊形式，表达式 `(amb <e1> <e2> ... <en>)` 有歧义地返回 `<ei>` 之一的值

比如 `(list (amb 1 2 3) (amb 'a 'b))` 可以有以下的值：`(1 a) (1 b) (2 a) (2 b) (3 a) (3 b)`

没有选择的 `amb` 表达式是一个没有可接受值的表达式，将会导致计算“失败”，不会产生任何值

```scheme
(define (require p)
  (if (not p) (amb))
)

(define (an-element-of items)
  (require (not (null? items)))
  (amb (car items) (an-element-of (cdr items)))
)
```

抽象来看，可以认为 `amb` 表达式将导致时间分裂为不同的分支，而计算将在每一个分支里进行，`amb` 表示了一个确定性的**选择点**。如果有一台机器，可以动态分配处理器，就可以以一种直截了当地方式实现这种搜索。执行就像在一台机器上顺序执行，直至遇到一个 `amb` 表达式，在这个点上，需要分配更多的处理器，并继续这一选择所蕴含的所有并行执行，直到成功执行或失败

另一方面，如果只有一台机器，只能执行一个进程，必须换一种方式实现顺序性的方式，可以设想修改解释器，使之在遇到选择点时，随机选择一个分支走下去。如果导向失败，需要重新允许解释器，再做随机选择，最后找到一个不失败的值。一种更好的方式是系统化搜索所有可能的执行路径



非确定性程序的实例有：逻辑谜题(一系列逻辑对话然后有一个人是真话进行推断等等)、自然语言的语法分析



**`amb` 解释器的实现**

对于常规的 scheme 表达式可能返回一个值，也可能永不终止，或发出一个错误信号。对于非确定性的 scheme，表达式还可能”遇到死胡同“，需要回溯到前面的选择点。`amb` 解释器执行过程取三个参数，执行环境、**成功继续**过程、**失败继续**过程。如果求值得到了一个结果，就用这个结果调用成功继续过程，否则就是遇到了死胡同，调用失败继续过程。

```scheme
(define (amb? exp)
  (tagged-list? exp 'amb)
)

(define (amb-choices exp)
  (cdr exp)
)

((amb? exp) (analyze-amb exp))

(define (amb-eval exp env succeed fail)
  ((analyze exp) env succeed fail)
)
```

执行的一般形式是：

```scheme
(amb-eval exp
          the-global-environment
          (lambda (value fail) value)   ; succeed
          (lambda () 'failed)           ; fail
)
```

成功继续是一个带有两个参数的过程，刚刚得到的值和另一个失败过程，如果这个值导致失败的话，就去调用该失败继续过程



```scheme
(define (analyze-self-evaluated exp)
  (lambda (env succeed fail)
    (succeed exp fail)
  )
)

(define (analyze-quoted exp)
  (let ((qval) (text-of-quotation exp))
    (lambda (env succeed fail)
      (succeed qval fail)
    )
  )
)

;; ...
```

上述其他表达式过程比较复杂而且用处不大，不进行详细探讨



## 4. 逻辑程序设计

计算机处理的是命令式（怎么做）的知识，而数学处理的是说明式（是什么）的知识。

命令式程序设计语言要求程序员以一种形式去表述有关的知识，其中需要指明一种为某一特定问题一步一步的方法。另一方面，高级语言也提供了大量的方法论知识，使用户可以不必关心具体计算如何进行的一些细节。

一部分的程序语言，包括 Lisp，都是围绕数学函数值的计算组织起来的，面向表达式的语言(Lisp, Fortran, Algol)，利用了表达式的一语双关：一个描述了某个函数值的表达式也可以解释为一种计算该值的方法。正因于此，大部分程序设计语言都强烈倾向于单一方向计算（定义清晰的输入和输出）。也存在一些与此不同的程序设计语言，比如之前的约束系统、非确定性计算等。

逻辑程序设计是从有关自动定理证明的长期研究中产生出来的，它们都在穷尽地搜索可能证明的空间。使这种搜索成为可能的重要突破是**合一算法和归结原理**。

逻辑程序设计可以成为一种威力强大的写程序的方式。这种威力来自于：一个关于“是什么”的事实可能被用于解决多个不同的问题。

比如 `append` 操作，以两个链表作为参数，返回组合后的链表。

```scheme
(define (append x y)
  (if (null? x)
      y
      (cons (car x) (append (cdr x) y))
  )
)
```

上面的过程可以翻译为以下两条规则：

- 对于任何一个链表 y，对空链表和 y 进行 append 结果就是 y
- 对于任何 u、v、y、z，如果 v 和 y 进行 `append` 形成 z， 那么将 `(cons u v)` 与 y 进行 `append` 将形成 `(cons u z)`

使用这一过程，可以很方便回答这一类问题，求出 `(append (list a b) (list c d))` 的结果。

但是，上面两条规则也足以回答下面这类问题，而上述过程却无法回答：

- 找出一个表 y，使 `(a b)` 与 y 进行 `append` 产生出 `(a b c d)`
- 找出所有的 x 和 y，进行 `append` 形成 `(a b c d)`

在逻辑式程序设计语言中，程序员实现 `append` 过程的方式就是陈述出有关 `append` 的两条规则。相应的怎样做的知识有解释器自动提供。

当代逻辑程序设计语言，都有一些实质性的缺陷，里面有关怎样做的通用方法，可能使我们陷入谬误性的无穷循环或其他并非我们期待的行为。



这一节讨论逻辑程序语言的解释器，我们一般称逻辑程序语言为**查询语言**，在描述提取数据库信息的查询或其他操作时非常有用。也就相当于介绍一个 SQL DSL(Domain-specific language)。



**演绎信息检索**

一个实例数据库，某个公司的人事数据库包含一些人事的**断言**。比如

```scheme
;; Boss
(address (Warbucks Oliver) (Swellesley (Top Heap Road)))
(job (Warbucks 01 iver) (administration big wheel))
(salary (Warbucks 01iver) 150000)

;; Big Leader
(address (Bitdiddle Ben) (Slumerville (Ridge Road) 10))
(job (Bitdiddle Ben) (computer wizard) ) 
(salary (Bitdiddle Ben) 60000)
(supervisor (Bitdiddle Ben) (Warbucks Oliver))

;; Under "Ben"
(address (Hacker Alyssa P) (Cambridge (Mass Ave) 78))
(job (Hacker Alyssa P) (computer programmer))
(salary (Hacker Alyssa P) 40000)
(supervisor (Hacker Alyssa P) (Bitdiddle Ben))

(address (Fect Cy D) (Cambridge (Ames Street) 3))
(job (Fect Cy D) (computer programmer))
(salary (Fect cy D) 35000) 
(supervisor (Fect Cy D) (Bitdiddle Ben))

(address (Tweakit Lem E) (Boston (Bay State Road) 22))
(job (Tweakit Lem E) (computer technician))
(salary (Tweakit Lem E) 25000)
(supervisor (Tweakit Lem E) (Bitdiddle Ben))

;; Trainee, Under "Alyssa"
(address (Reasoner Louis) (Slumerville (Pine Tree Road) 80))
(job (Reasoner Louis) (computer programmer trainee))
(salary (Reasoner Louis) 30000)
(supervisor (Reasoner Louis) (Hacker Alyssa P))
```

除了公司的大老板之外，其他人都是计算机分布，工作描述中第一个词就是 `computer`

还包含另外一些断言：

```scheme
(can-do-job (computer wizard) (computer programmer))
(can-do-job (computer wizard) (computer technician))

(can-do-job (computer programmer) (commputer programmer trainee))
```



进行简单查询：

为了找到所有的 `computer programmer`

```scheme
(job ?x (computer programmer))

; (job (Hacker Alyssa P) (computer programmer))
; (job (Fect Cy D) (computer programmer))
```

列出所有员工的地址：

```scheme
(address ?x ?y)
```

查询计算机部所有员工：

```scheme
(job ?x (computer ?type))

; (job (Bitdiddle Ben) (computer wizard))
; (job (Hacker Alyssa P) (computer programmer))
; (job (Fect Cy D) (computer programmer))
; (job (Tweakit Lem E) (computer technician))
```

如果想要匹配 `computer programmer trainee` 需要使用 `(job ?x (computer . ?type))`



复合查询：

```scheme
(and <query1> <query2> ... <queryn>)
(or <query1> <query2> ... <queryn>)
(not <query1>)

(and (job ?person (computer programmer))
     (address ?person ?where)
)

; (and (job (Hacker Alyssa P) (computer programmer))
;      (address (Hacker Alyssa P) (Cambridge (Mass Ave) 78))
; )

; (and (job (Fect Cy D) (computer programmer))
;      (address (Fect Cy D) (Cambridge (Ames Street) 3))
; )
```



规则查询：

```scheme
(rule <conclusion> <body>)

(rule (lives-near ?person1 ?person2)
  (and (address ?person1 (?town . ?rest1))
       (address ?person2 (?town . ?rest2))
       (not (same ?person1 ?person2))
  )
)

(lives-near ?x (Bitdiddle Ben))
; (lives-near (Reasoner Louis) (Bitdiddle Ben))
; (lives-near (Aull DeWitt) (Bitdiddle Ben))
```



**查询系统是如何工作的**

这一节将给出查询解释器的实现，查询解释器必须执行某些搜索，以便将有关查询与数据库里的事实做匹配。一种方法是使用 `amb` 解释器，另一种是借助流，设法控制搜索。考虑实现第二种方法。主要围绕**模式匹配和合一**。

---

*模式匹配*

检查数据项是否是一个给定的模式，比如 `((a b) c (a b))` 与模式 `(?x c ?x)` 匹配，其中 `?x` 约束到 `(a b)`；也与 `(?x ?y ?z)` 匹配，其中 `?x/?z` 都约束到 `(a b)`，`?y` 约束到 `c`

模式匹配器以一个模式、一个数据、一个框架作为输入，该框架描述了一些模式变量的约束。模式匹配器检测在框架约束下该数据是否以某种方式与模式匹配。

模式匹配器提供了不涉及规则的简单查询的所有机制。比如在执行下列的查询时：

```scheme
(job ?x (computer programmer))
```

需要对于一个空的框架，扫描数据库里的所有断言，选出其中与模式相匹配的断言。

用模式去检查框架的工作被组织为一种对流的使用。给定一个框架，匹配过程将一个个扫描数据库里的条目，对每个条目，匹配器产生出一个指明匹配失败的特殊符号或者给定框架的扩充。对所有数据库条目的匹配结果被收集到一起，形成一个流，流被送入过滤器，删除所有的失败信息，得到另一个流，其中包含着所有满足条件的框架。

真正优雅的地方是对于复合查询的实现，`(and A B)` 就是先使用 A 条件过滤，再使用 B 条件过滤；`(or A B)` 就是把 A 过滤结果和 B 过滤结果进行归并；`(not A)` 就是相当于一个过滤器，删除非 A 的条目。

---

*合一*

为了处理查询语言中的规则，必须找到这样的规则，其 conclusion 部分与给定的查询模式匹配。规则的 conclusion 很像断言，也可以包含变量，为了处理这种情况，需要模式匹配的一种推广——合一。其中的模式和数据都可以包含变量。

合一器取两个都可以包含常量和变量的模式作为参数，设法去确定找到其中变量的某种赋值，如果能找到，就返回有关约束的框架。比如

`(?x a ?y)` 和 `(?y ?z a)` 的合一将产生 `?x/?y/?z` 都约束到 `a` 的框架

`(?x ?y a)` 和 `(?x b ?y)` 的合一将失败

合一是查询系统最难的部分，对于复杂的模式，合一可能需要做推理。比如 `(?x ?x)` 和 `((a ?y c) (a b ?z))` 必须推断出 `?x` 为 `(a b c)`。

---



**无穷循环问题**

```scheme
(assert! (married Minnie Mickey))
```

使用 `assert!` 将知识或规则加入到数据库中

如果提问 `(married Mickey ?who)` 将无法得到回复，因为系统并不能自动推断 A 与 B 结婚，那么 B 也与 A 结婚

为此我们改为下面的规则：

```scheme
(assert! (rule (married ?x ?y))
               (married ?y ?x)
         )
)
```

再次查询 `(married Mickey ?who)` 很不幸进入了无穷循环，首先系统发现 `(married ?x ?y)` 规则与 `(married Mickey ?who)` 匹配，然后又发现 `(married ?y ?x)` 与 `(married Mickey ?who)` 匹配，造成了无穷循环，为了在陷入无穷循环之前找出回答，需要实现细节上进行一些特殊处理。



由于查询解释器实现内容过多且一般不会遇到，先略过



# 五、寄存器机器的运算

寄存器式虚拟机

这一章主要基于传统计算机的一步一步的操作，描述一些计算过程。这类计算机被称为**寄存器机器**。能够顺序执行一些**指令**，对一组固定称为寄存器的存储单元的内容完成各种操作。对寄存器机器执行的计算过程的描述，很像传统计算机的机器语言程序。在这里不考虑任何架构的机器语言，而是考虑若干 Lisp 过程，并为每个过程设计一部特殊的寄存器机器。

## 1. 寄存器机器的设计

要设计一部寄存器机器，必须设计好**数据通路**（寄存器和操作）和**控制器**，控制器实现操作的顺序执行。

定义一部机器的数据通路，需要描述其中的寄存器和各种操作；控制器定义为一个指令序列，再加上一些标号，标明序列中的一些入口点。

```scheme
(define (gcd a b)
  (if (= b 0)
      a
      (gcd b (remainder a b))
  )
)
```

将上述过程翻译到指令序列。类似于下面的结果：

```scheme
(controller
  test-b  ; label
    (test (op =) (reg b) (const 0))
    (branch (label gcd-done))           ; if b = 0, goto gcd-done
    (assign t (op rem) (reg a) (reg b)) ; t = a % b
    (assign a (reg b))                  ; a = b
    (assign b (reg t))                  ; b = t
    (goto (label test-b))               ; goto test-b
  gcd-done ; label
)
```

现在需要给上面的 `GCD` 机器增加输入输出，使用 `read/print`，`read` 就是读入一个值到一个寄存器中，`print` 不会产生任何可以存入寄存器中的值，我们把这种操作称为**动作**，增加 `perform` 的指令，可以执行动作。

```scheme
(perform (op print) (reg a))
```



**机器设计的抽象**

我们需要定义一些包含着某些基本操作的机器，这样我们可以忽略机器的一些细节，将大量复杂的事物隐藏起来，比如写高级应用程序一般不需要注意其生成的机器语言。

我们还可以使用子程序来避免的重复的数据通路部件，也就是 DRY。



采用栈来实现递归，增加两条指令，原文用的 `save/restore`，这里改用 `push/pop`，还需要一个 `continue` 寄存器，在解决完子问题后跳回到原问题中。

比如递归求阶乘，

```scheme
(define (factorial n)
  (if (= n 1)
      1
      (* (factorial (- n 1)) n)
  )
)

(controller
    (assign continue (label fact-done))
  fact-loop
    (test (op =) (reg n) (const 1))
    (branch (label base-case))
    (push continue)
    (push n)
    (assign n (op -) (reg n) (const 1))
    (assign continue (label after-fact))
    (goto (label fact-label))
  after-fact
    (pop n)
    (pop continue)
    (assign val (op *) (reg n) (reg val))
    (goto (reg continue))
  base-case
    (assign val (const 1))
    (goto (reg continue))
  fact-done
)
```



指令总结

```scheme
(assign <register-name> (reg <register-name>))
(assign <register-name> (const <constant-value>))
(assign <register-name> (op <operation-name> <input1> ... <inputn>))
(assign <register-name> (label <label-name>))
(perform (op <operation-name>) <input1> ... <inputn>)
(test (op <operation-name>) <input1> ... <inputn>)
(branch (label <label-name>))
(goto (label <label-name>))
(goto (reg <register-name>))

(push <register-name>)   ; save
(pop <register-name>)    ; restore
```



## 2. 寄存器机器模拟器

```scheme
(make-machine <register-names> <operations> <controller>)
(set-register-contents! <machine-model> <register-name> <value>)
(get-register-contents! <machine-model> <register-name>)
(start <machine-model>)
```

可以使用上述四个接口来定义一个模拟器，来进行一些计算，比如 GCD 机器：

```scheme
(define gcd-machine
  (make-machine
    '(a b t)   ; register names
    (list (list 'rem remainder) (list '= =))  ; operations
    '(test-b  ; label
      (test (op =) (reg b) (const 0))
      (branch (label gcd-done))           ; if b = 0, goto gcd-done
      (assign t (op rem) (reg a) (reg b)) ; t = a % b
      (assign a (reg b))                  ; a = b
      (assign b (reg t))                  ; b = t
      (goto (label test-b))               ; goto test-b
      gcd-done ; label
     )
  )
)

(set-register-contents! gcd-machine 'a 206)   ; done
(set-register-contents! gcd-machine 'b 40)    ; done
(start gcd-machine)                           ; done
(get-register-contents gcd-machine 'a)        ; 2

```



**机器模型**

```scheme
(define (make-machine register-names ops controller-text)
  (let ((machine (make-new-machine)))
    (for-each (lambda (register-name)
                ((machine 'allocate-register) register-name)
              )
      register-names
    )

    ((machine 'install-operations) ops)
    ((machine 'install-instruction-sequence)
      (assemble controller-text machine)
    )
    machine
  )
)

;; ----------------- register -----------------
(define (make-register name)
  (let ((contents '*unassigned*))
    (define (dispatch message)
      (cond ((eq? message 'get) contents)
            ((eq? message 'set) (lambda (value) (set! contents value)))
            (else (error "Unknown request -- REGISTER" message))
      )
    )
    dispatch
  )
)

(define (get-contents register)
  (register 'get)
)

(define (set-contents register value)
  ((register 'set) value)
)

;; ----------------- stack -----------------
(define (make-stack)
  (let ((s '()))
    (define (push x)
      (set! s (cons x s))
    )

    (define (pop)
      (if (null? s)
        (error "Empty stack -- POP")
        (let ((top (car s)))
          (set! s (cdr s))
          top
        )
      )
    )

    (define (initialize)
      (set! s '())
      'done
    )

    (define (dispatch message)
      (cond ((eq? message 'push) push)
            ((eq? message 'pop) (pop))
            ((eq? message 'initialize) (initialize))
            (else (error "Unknown request -- STACK" message))
      )
    )

    dispatch
  )
)

(define (pop stack)
  (stack 'pop)
)

(define (push stack value)
  ((stack 'push) value)
)

;; ----------------- machine -----------------
(define (make-new-machine)
  (let ((pc (make-register 'pc))
        (flag (make-register 'flag))
        (stack (make-stack))
        (the-instruction-sequence '())
       )
    (let ((the-ops (list (list 'initialize (lambda () (stack 'initialize)))))
          (register-table (list (list 'pc pc) (list 'flag flag)))
         )
      (define (allocate-register name)
        (if (assoc name register-table)
          (error "Multiply defined register: " name)  ; exists
          (set! register-table 
                (cons (list name (make-register name))
                      register-table
                )
          )
        )
        'register-allocated
      )

      (define (lookup-register name)
        (let ((val (assoc name register-table)))
          (if val  ; ('name register)
            (cadr val)
            (error "Unknown register: " name)
          )
        )
      )

      (define (execute)
        (let ((insts (get-contents pc)))
          (if (null? insts)
            'done
            (begin ((instruction-execution-proc) (car insts))
                   (execute)
            )
          )
        )
      )

      (define (dispatch message)
        (cond ((eq? message 'start)
                (set-contents! pc the-instruction-sequence)
                (execute)
              )
              ((eq? message 'install-instruction-sequence)
                (lambda (seq) (set! the-instruction-seq seq))
              )
              ((eq? message 'allocate-register) allocate-register)
              ((eq? message 'get-register) lookup-register)
              ((eq? message 'install-operations)
                (lambda (ops) (set! the-ops (append the-ops ops)))
              )
              ((eq? message 'stack) stack)
              ((eq? message 'operations) the-ops)
              (else (error "Unknown request -- MACHINE" message))
        )
      )

      dispatch
    )
  )
)

(define (get-register machine register-name)
  ((machine 'get-register) register-name)
)
```



**汇编程序**

汇编程序将一部机器的控制器表达式序列翻译为与之对应的机器指令表，每条指令都带有相应的执行过程。

首先将用户的代码 `controller-text` 转换为指令序列，然后通过 `install-instruction-seqence` 把指令序列安装到机器上

```scheme
;; controller-text -> instruction sequence
(define (assemble controller-text machine)
  (extract-labels controller-text
    (lambda (insts labels)
      (update-insts! insts labels machine)
      insts
    )
  )
)
```

`extract-labels` 顺序扫描 `text` 中的各个元素，逐渐扩展 `insts` 和 `labels` 链表。当全部扩展完毕，使用 `(update-insts! insts labels machine)` 更新机器的指令表，因为 `extract-labels` 只获取到指令名字，并不知道指令任何执行，在 `update-insts!` 中需要更新指令表，将每一个指令设置为指令到其执行过程 序对。

```scheme
(define (extract-labels text receive)
  (if (null? text)
    (receive '() '())
    (extract-labels (cdr text)
      (lambda (insts labels)
        (let ((next-inst (car text)))
          (if (symbol? next-inst)
            ; update labels
            (receive insts
                     (cons (make-label-entry next-inst insts)
                           labels
                     )
            )

            ; update insts
            (receive (cons (make-instruction next-inst) 
                           insts
                     )
                     labels
            )
          )
        )
      )
    )
  )
)

(define (update-insts! insts labels machine)
  (let ((pc (get-register machine 'pc))
        (flag (get-register machine 'flag))
        (stack (machine 'stack))
        (ops (machine 'operations))
       )
    (for-each
      (lambda (inst)
        (set-instruction-execution-proc! inst
          (make-execution-procedure (instruction-text inst) labels machine
            pc flag stack ops
          )
        )
      )

      insts
    )
  )
)
```

指令和标签的一些操作：

```scheme
(define (make-instruction text)
  (cons text '())
)

(define (instruction-text inst)
  (car inst)
)

(define (instruction-execution-proc inst)
  (cdr inst)
)

(define (set-instruction-execution-proc! inst proc)
  (set-cdr! inst proc)
)

(define (make-label-entry label-name insts)
  (cons label-name insts)
)

(define (lookup-label labels label-name)
  (let ((val (assoc label-name labels)))
    (if val
        (cdr val)
        (error "Undefined label -- ASSEMBLE" label-name)
    )
  )
)
```

