---
layout: post
title: 语言特征与模式- λ演算
tags: 
    - lisp
    - lambda
    - FP
---

# 语言特征与模式- λ演算

## λ演算

wiki定义

> Lambda演算可以被称为最小的通用程序设计语言。它包括一条变换规则（变量替换）和一条函数定义方式，Lambda演算之通用在于，任何一个可计算函数都能用这种形式来表达和求值。因而，它是等价于图灵机的。尽管如此，Lambda演算强调的是变换规则的运用，而非实现它们的具体机器。可以认为这是一种更接近软件而非硬件的方式。


## λ 表达式通俗定义

**$$λ$$**表达式所属空间Λ描述为：


1. 变量  $$a ∈ Λ$$
2. 对于任何 $$M ∈ Λ$$，若变量$$a ∈ Λ$$， 则$$λa . M ∈ Λ$$
3. 对于$$ F，M ∈ Λ$$， 有 $$(FM) ∈ Λ$$


规则一中说明了$$a$$是一个**标识符**，代表了一个元素。

规则二中说明了可以从$$ λ$$ 表达式中**构造**出一个函数$$f(a)$$，这个函数以参数$$a$$为自变量，以$$λ$$表达式描述映射关系，构造出的函数$$f$$也同属于$$λ $$表达式

规则三中说明了$$M$$可以作用在$$F$$上，也即以$$M$$作为函数$$F$$的实参数进行计算，返回的值也属于$$λ$$表达式

三条规则说明了定义了从$$Λ$$空间到$$Λ$$空间的映射关系的函数作为$$λ $$表达式，其所有表达式的集合构成了$$Λ$$空间。

λ 表达式描述了这样的集合$$E$$，对于函数$$y = f(x)$$, 其输入$$x$$，映射关系$$f$$，输出$$y$$均属于集合$$E$$

在函数式编程思想中，说明了这么一个理念，**变量和函数等同**，可以作为变量参与运算，运算结果也可是函数。这就是**高阶函数**的定义。

 我们需要以lambda演算为基础构建出可用的基本编程语句模式：包括简单的自然数及其运算、布尔运算、条件语句和循环（递归）语句，由此体现出lambda演算的图灵完备性。lambda演算的图灵完备性需要更为严谨的证明，故此作“体现”两字。

## α-变换 和 β-规约

对应于程序函数中的入参声明和实参替入概念，不冲突的情况下，入参名称的改变不影响函数的描述，而应用函数时将实参代入形参进行计算。


## 单位元

类比实数集合中的$$1$$，对任意$$a∈R$$，均有 $$ 1 *  a= a$$
λ 表达式的单位元为

$$
e = λx.x
$$

有 

$$e\ a = ( λx.x)\ a = a$$

特别的，有

$$
 e \ e = ( λx.x)( λx.x) = ( λx.x) = e
$$

## η-变换

$$
f  = λx.(f \  x)
$$

这个等价变换有个好处，若$$f = g\ y$$，则可以延迟到需要调用函数$f$的时候再计算$$g(y)$$

## 柯里化
纯粹的λ 表达式不支持多个参数的函数表达式，可以通过逐步通过返回接受一个参数的函数间接表达多参数函数。如：

```javascript
function add(a, b) { return a + b; };
var add = a => b => a + b;
```

这样，通过这种转换（翻译），从原理上λ 表达式可以支持多参数函数表达式，我们可以使用多参数函数表达式简化表达。

作用：

- 描述λ 表达式支持多参数函数表达式的机制。
- 可以**解耦**函数中的各个参数，在调用时不必一次性全部将各个参数值给出，可以将未给出全部实参的“不完全”表达式当作变量使用。
- 封装和信息隐藏，将一些参数进行隐藏，仅给出需要传参的参数，而内部已经绑定的实参对于调用者说不可见。
- 组装，通过各个算子的组合构建出更高层次的函数，提高函数的复用性。


## 布尔类型构建

#### 布尔变量

布尔全集仅有 true 和 false 两个，考虑设计一个二元组描述整个全集，并将第一个设为false，第二个设为true
  
```scheme
(define true  (lambda (x y) x))
(define false (lambda (x y) y))
```

#### 布尔运算-或
注意到`true`和`false`有选择功能，对于`OR(x, y)`, 当`x`为真时选择`x`，否则选择`y`，故可以这样定义`OR`

```scheme
(define OR (lambda (x y) (x true y)))
```

#### 布尔运算-与
 对于`AND(x, y)`, 当`x`为真时选择`y`，否则选择`false`

```scheme
(define AND (lambda (x y) (x y false)))
```

#### 布尔运算-非
当`x`为真时选择`false`，否则选择`true`

```scheme
(define NOT (lambda (x) (x false true)))
```

## 自然数类型构建


#### 零
考虑这一函数，其代表的数字和参数`f`被应用的次数相同

```scheme
(define ZERO  (lambda (f x) x))
```
则有

```scheme
(define ONE  (lambda (f x) (f x)))
(define TWO  (lambda (f x) (f (f x))))
(define N    (lambda (f x) (f (f (....  (f x))))))
```
#### 递增算子

我们需要一个从$N$递推到$N+1$的函数`INC`，有

```scheme
INC N = N + 1
```
我们需要解出INC算子，尝试通过递推解决

每次都是通过一个lambda表达式来表达一种类型，因为lambda表达式是函数，故一个类型的特征通常表现在**调用**该lambda表达式描述的函数后所体现的行为，那么要了解自然数`N`的特征，则看看调用N的函数所表现出的行为：

```scheme
(TWO f x) = (f (f x))
(N   f x) = (f (f (....  (f x))))
```

注意到`N`应用到`(f x)`后是将参数`x`应用到函数`f` N次, 故`N+ 1` 应该是在`N`的基础上多调用`f`一次，固有

```scheme
((INC N) f x) =  (f (N f x))
```

利用η-变换解出INC算子，需要将参数`f`、`x` 柯里化，隐藏到内部lambda的参数列表中，因为`f`、`x` 是N的参数，不应是`INC`的参数

```scheme
((INC N) f x) = (f (N f x))
(lambda (N f x) ((INC N) f x)) = (lambda (N f x)(f (N f x)))
INC = (lambda (N f x)(f (N f x)))
INC = (lambda N (lambda (f x)(f (N f x))))
INC = (lambda N (lambda f (lambda x (f ((N f) x)))))
```

#### 加法算子

现在定义两个整数的加法ADD，明显有

```scheme
(define ADD (lambda (N M) (lambda (f x) (N f (M f x)))))
```

####  乘法算子
显然，有

```scheme
(define MUL (lambda (N M) (lambda (f x) (N (M f) x))))
```

#### 递减算子

类似于求链表中一个节点的前置节点，在单链表的情况下，可以通过设置两个指针p，q，每次走时p紧跟q后面，当q到达某节点时，p即指向该节点的前置节点。故我们需要定义二元组存放这样的两个指针，同时预留一个参数f以便可以编写关于PAIR实例操作的方法。

```scheme
(define (PAIR a b)(lambda f (f a b)))
```

以及访问第一个元素和第二个元素的方法

```scheme
(define L (lambda (a b) (a)))
(define R (lambda (a b) (b)))
```

还有一个后驱算法`Trans`使得

$$
(lastNumber,currentNumber)⇒(currentNumber,SuccNumber)
$$

有

```scheme
(define TRANS (lambda (p) (PAIR (p R) (INC (p R)))))
```

故递减算子PRED为

```scheme
(define PRED (lambda (N) ((N TRANS (PAIR 0 0)) L)))
```

#### 减法算子
于是减法就很简单了，将PRED调用M次应用在N上得到N  - M

$$
Subt:=λnm.m Pred n
$$

## 谓语和条件语句
在条件语句中，`if (condition) then {...} else {...}`,`condition`称为谓语，谓语是一个依据其变量的值来判定真或假的方法。

从判断一个数是否是零的谓语开始，利用自然数的性质，当函数`f`被应用时（即数大于等于1）立即返回`false`，`x`为初始值`true`，有

```scheme
(define (isZero N) (N (lambda (x) false) true))
```

等于、不等于、大于、小于也能从`isZero `和之前的布尔运算构建出，不再叙述。

而条件语句其实就是单位元`e`。

```scheme
(define (IF condition then else) (condition then else))
```


## 递归（循环）语句和Y组合子

在数学上，若对于一个函数$$f(x)$$,存在一个$$x = x0$$,使得$$f(x) = x$$,则称$$x = x0$$为函数$$f$$的一个不动点。

而在lambda函数上，对于任意一个函数$$f$$，我们均能找到一个由$$f$$推导出来的一个不动点$$x =Y(f)$$，使得$$f(Y(f))=Y(f)$$，这个$$Y$$被称作$Y$组合子，我们试求出这样的函数`Y`，使得对于任意一个函数都能通过`Y`得到`f`的不动点。

Y组合子用于实现函数的可递归特性。举例，为了完成如下的函数的定义：

```scheme
(define (fac n) (if (= n 1)
                    1
                    (+ n (fac (- n 1)))))
```

按照lambda表达式的语法来说，函数体里的`fac`是未定义的，没有出现在参数列表中，也不支持对函数命名，我们想借助一个如下的环境为函数提供可递归的定义：

$$
g:=λf\ x.M(f, x)
$$

`f`为自己定义的可递归函数，在函数体`M`内对`f`的调用如同递归调用自身，而`x`则是原本定义递归函数`f`的参数，由于`f`出现在参数列表中，故可以在函数体内使用函数`f`。我们根据这样的规则定义了函数`g`用来实现递归语义，则还需要一个工具把具体的`f`算出来以便代入到`g`的参数中实现递归的功能，而这样的一个工具便是组合子`Y`，它满足`Y g = f`。

关于Y的定义如下:

$$
Y = λ f. (λ x. f (x\ x)) (λ x. f (x\ x))
$$


#### * Y组合子的推导

1. 以斐波那契数列项推导公式为例，目标是消除名字`fac`

   ```scheme
(define (fac n) (if (= n 1)
                    1
                    (+ n (fac (- n 1)))))
   ```

2. 将函数fac放入参数以完成递归调用

   ```scheme
(define (fac f n) (if (= n 1)
                      1
                      (+ n (f f (- n 1)))))
   ```

3. 柯里化，解耦函数f和参数n

   ```scheme
(define (fac f) (lambda (n) (if (= n 1)
                            1
                            (+ n ((f f) (- n 1))))))
;; 调用
((fac fac) 5) ;;15
   ```

4. 将`w = (f f)`抽象出来有

   ```scheme
(define (w x) (x x))
(define (fac f) (lambda (n) (if (= n 1)
                           1
                           (+ n ((w f) (- n 1))))))
;; 调用
((w fac) 5) ;;15
   ```

5. 将`g = (w f) `提取为参数，并根据η-变换处理`g`延迟`g`的计算

   ```scheme
(define (w x) (x x))
(define (fac f) ((lambda (g) (lambda (n) (if (= n 1)
                             1
                             (+ n (g (- n 1))))))
                 (lambda (x) ((w f) x))))
;; 调用
((w fac) 5) ;;15
   ```

6. 提取自定义的匿名递归函数表达式作为参数，参数名为h，重构后的函数名为`Y0`，同时命名斐波那契的匿名递归函数表达式为`fac`，`fac`实现了在函数体内不调用`fac`自身。

   ```scheme
(define (w x) (x x))
(define fac (lambda (g) (lambda (n) (if (= n 1) 1 (+ n (g (- n 1)))))))
(define Y0 (lambda (h) (lambda (f) (h (lambda (x) ((w f) x))))))
;; 调用
((w (Y0 fac)) 5) ;;15
   ```

7. 对于调用表达式，将`fac`从函数`w`中提出作为参数`f`

   ```scheme
(define (w x) (x x))
(define fac (lambda (g) (lambda (n) (if (= n 1) 1 (+ n (g (- n 1)))))))
(define Y0 (lambda (h) (lambda (f) (h (lambda (x) ((w f) x))))))
;; 调用
(((lambda (f) (w (Y0 f))) fac) 5) ;;15
   ```

8. 记`Y = (lambda (f) (w (Y0 f)))`

   ```scheme
(define (w x) (x x))
(define fac (lambda (g) (lambda (n) (if (= n 1) 1 (+ n (g (- n 1)))))))
(define Y0 (lambda (h) (lambda (f) (h (lambda (x) ((w f) x))))))
(define Y  (lambda (f) (w (Y0 f))))
;; 调用
((Y fac) 5) ;;15
   ```

9. 至此，只要将`fac`的定义代入调用式中即可消除名字`fac`，对于`Y`，将`Y0 `展开，把`Y0 ` 中的变量`f`替换为`g`以免与函数`Y`中的`f`产生歧义(α-变换)，有

   ```scheme
(define (w x) (x x))
(define Y  (lambda (f) (w ((lambda (h) (lambda (g) (h (lambda (x) ((w g) x))))) f))))
;; 调用
((Y (lambda (g) (lambda (n) (if (= n 1) 1 (+ n (g (- n 1))))))) 5) ;;15
   ```

10. 对于`Y`，展开最里面的`w`，并将实参`f`代入到参数`h`，有

    ```scheme
(define (w x) (x x))
(define Y  (lambda (f) (w (lambda (g) (f (lambda (x) ((g g) x)))))))
    ```

11. 展开余下的`w`，即为最后的结果

    ```scheme
(define Y  (lambda (f) ((lambda (g) (f (lambda (x) ((g g) x))))
                        (lambda (g) (f (lambda (x) ((g g) x)))))))
;; 调用
((Y (lambda (g) (lambda (n) (if (= n 1) 1 (+ n (g (- n 1))))))) 5) ;;15
    ```

- 另外，在第6步中，可以将参数`h`放在`f`的后面（而不是前面）作为参数，调用时优先与`w`结合，有

  ```scheme
(define (w x) (x x))
(define (fac f) (lambda (h)
                            (h (lambda (x) (((w f) h) x)))))
(((w fac) (lambda (g) (lambda (n) (if (= n 1) 1 (+ n (g (- n 1))))))) 5)
  ```

  将`fac`直接代入到调用式中，得出

  ```scheme
(define Θ (w (lambda (f) (lambda (h) (h (lambda (x) (((f f) h) x)))))))
  ```

   这实际上 Θ与之前推导出的Y算子等价，说明不同的变换和规约顺序可以得出不同的结果。

## 拓展

更深入的主题：

- 负数、有理数的构造。（思路：和前驱算子一样，设计一个二元组，如` -5 == 0 - 5`，`0.5 = 1 / 2`）
- 实数的构造，涉及到实数理论。

## 参考
- λ演算，https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97
- 神奇的λ演算，http://www.jianshu.com/p/e7db2f50b012
- A Tutorial Introduction to the Lambda Calculu，http://www.inf.fu-berlin.de/inst/ag-ki/rojas_home/documents/tutorials/lambda.pdf
- Fixed-point combinators in JavaScript: Memoizing recursive functions，http://matt.might.net/articles/implementation-of-recursive-fixed-point-y-combinator-in-javascript-for-memoization/
- Lambda Calculus: Subtraction is hard, http://gettingsharper.de/2012/08/30/lambda-calculus-subtraction-is-hard/