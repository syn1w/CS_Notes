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