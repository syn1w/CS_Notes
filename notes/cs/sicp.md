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

