---
layout: post  
title:  Lambda演算学习笔记（01） 
date:   2017-6-25  
category: "lambda"  
---
最近读了[康托尔、哥德尔、图灵——永恒的金色对角线](http://mindhacks.cn/2006/10/15/cantor-godel-turing-an-eternal-golden-diagonal/)，其中提到了Y combinator，突然很感兴趣。
本来昨天都写完了，结果保存时不小心误操作删掉了，元气大伤。。。
## 基本语法（BNF）
λ演算的语法规则极其简单，用[BNF](http://baike.baidu.com/item/BNF)的方式表述：
1. \<expr> ::= \<identifier>
2. \<expr> ::= (lambda \<identifier-list>. \<expr>)
3. \<expr> ::= (\<expr> \<expr>)  


第一条规则叫*变量*( `Variable` ):表示参数或数值的标识符。最简单的表达式（λ项），通常用小些字母表示。例如:x,y.  
第二条规则叫*抽象*( `Abstraction` ):用于定义函数，参数与作用域绑定（参数作用域的问题）。例如函数 `f(x) = x + 2` 的定义: `lambda x . x + 2` ( 或者是 `λx.x+2` )   
第三条规则叫*应用*( `Application` ):以给定的参数调用函数。前者为函数，后者为参数。例如 `(f x)` 表示把函数f应用于( apply to )参数x。  
在不会产生歧义的情况下，可以省略抽象和应用两条规则中的括号。  
如下假定保证不会产生歧义：
1. λ操作符被绑定到它后面的整个表达式(The scope of abstraction extends to the rightmost.)例如  `λx.M N` 表示`(λx.M N)` 而不是 `(λx.M) N`
2. 应用是左结合的(Application is left associati𝛼ve.)例如 `f x y` 表示 `(f x) y `而不是 `f (x y）`

## 柯里化（Currying）
基本语法的第二条规则——抽象，其实是有问题的。严格来说函数必须是单参函数，格式统一为 `λx.M` 。所以多参数的函数应该是这样的：`λx.λy.λz.M` ，即柯里化的形式。
为了方便书写和理解，λ演算引入了缩写：可以把 `λx.λy.λz.M` 缩写为 `λxyz.M` ，这于前文所述的规则一致。
但是要记住，`λxyz.M` 只是缩写，本质上它还是柯里化的函数。

## 自由变量和有界变量
这就是*抽象*语法所说的：参数与作用域绑定。函数体中与参数同名的变量，是有界变量，反之，则是自由变量。
具体对于到三条语法规则中：
1. 对于*变量*表达式V，V是变量，则这个表达式里自由变量的集合只有V。例如 `x` 是变量表达式，自由变量也是 `x` .
2. 对于*抽象*表达式 `λV.E` （V是变量，E是另一个表达式），自由变量的集合是E中自由变量的集合减去变量V。例如 `λx.x + y` 中自由变量为 `y` .
3. 对于应用表达式 `(E.E')` ，自由变量的集合是E和E'中自由变量集合的并集。例如 `(λx.x) x` 的自由变量为`x`(空集和 `x` 的并集)

一个λ表达式，当且仅当其自由变量集合为空时（没有自由变量），才是有效的λ表达式。注意，如果是复杂表达式中的一个子表达式，在不考虑其上下文的情况下，它可以有自由变量。

## 归约
在λ演算中只有两条规则：
1. Alpha转换：简而言之，就是参数名不重要。例如 `λxy.x + y` 等同于 `λab.a + b`。
2. Beta简化：这是λ演算的计算规则。简而言之，对于*应用*表达式，等同于用实参替换的函数体。例如 `(λx.x + 2) 3` 等同于 `3 + 2` 。

注意，仅当Beta化简不会引起有界变量和自由变量的冲突时，才可以Beta化简。
例如 `(λxy.x + y) (y + 1)` 直接Beta简化得到：`λy. y + 1 + y`。发现原来的自由变量y变为了有界变量，
所以这种情况下，需要先做Alpha转换: `(λxz.x + z)(y + 1)`.再做Beta简化: `λz.y + 1 + z`.

推荐一个lambda表达式的可视化小工具[Lambda Viwer](http://projectultimatum.org/cgi-bin/lambda).可以帮助我们理解复杂的lambda表达式。

## 邱奇数(Church numerals)
仔细看看λ演算的语法就能发现，整个系统中只有一个类型-单参函数。所以λ演算没有数值，只有邱奇数，一种用函数定义的数值。
邱奇数都是两个参数的函数：
1. `0 := λsz.z;`
2. `1 := λsz.s z;`
3. `2 := λsz.s (s z);`
4. `...`

对任意一个数 `n` ，它的邱齐数都是一个函数。这个函数把它的第一个参数应用到第二个参数上 `n` 次。
可以这样理解，`λz.z` 表示零值。那么 `0` 可以看作是返回零值的函数。
`s` 是后继函数(successor function),类似`++`操作，接受 `n` 返回 `n + 1` .所以 `1` 就可以看作是 `s` 应用于 `0` 的结果。
````javascript
let ZERO_VALUE = λz.z;//零值
let zero = λsz.z; //返回零值的函数
let succ = λnsz.s (n s z); //后继函数
let one = succ(zero);//即λsz.s z
let two = succ(one); //即λsz.s (s z)
````
上面的let只是语法糖，便于书写和理解，其实没有这种东西。
接着可以定义加法：`λxy.(λsz.(x s (y s z)))`.
加法都可以到Lambda Viwer里试试，便于理解。。

## 参考资料
1. [My Favorite Calculus: Lambda](http://goodmath.blogspot.jp/2006/05/my-favorite-calculus-lambda-part-1.html)
2. [λ演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)
3. [lambda-calculus](https://en.wikipedia.org/wiki/Lambda_calculus#Free_variables)