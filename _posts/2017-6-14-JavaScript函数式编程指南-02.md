---
layout: post  
title:  JavaScript函数式编程指南(02) 
date:   2017-6-14  
category: "JavaScript FP"  
---
## 组合（compose）
这就是组合：
````javascript
const compose = function(f, g) {
    return x => f(g(x));
}
````
本质上就是从右往左依次执行传入的函数。前一个函数的返回值作为后一个函数的参数。  
从右往左是线性的逻辑，比起从内往外，嵌套执行更清晰。
### 一个例子  
这是一个组合的实例，将`head`函数（取列表的第一个元素）和`reverse`函数（反转列表）组合成新的`last`函数（取列表的最后一个元素）
````JavaScript
const R = require('Ramda');
const reduce = R.curry(function(fn, init, xs) {
    return xs.reduce(fn, init);
})
const head = xs => xs[0];
const reverse = reduce((acc, x) => x.concat(acc), []);

const last = R.compose(head, reverse);
````
组合可以自由得将两个或两个以上函数拼接成新的函数。  
组合的顺序很重要！例子中`head`和`reverse`的顺序不能错。

### 结合律
组合操作满足数学上的结合律
````javascript
//结合律(associativity)
//(a + b) + c = a + (b + c)
compose(compose(f, g), h) == compose(f, compose(g, h));
````
这意味可以自由灵活地将函数抽出组合，再放回执行而不改变最后的结果。 
### pointfree 
`pointfree`是指函数无需提及将要操作的数据是什么样的。  
作为一等公民的函数，柯里化和组合，三着协作实现了这个模式。
````javascript
//非pointfree，提及了string
const snakeCase = function(string) {
    return string.toLowerCase.replace(/\s+/ig, '_');
}

//pointfree
const snakeCase = compose(toLowerCase, replace(/\s+/ig, '_'));
````
`pointfree`模式不关心数据，只关系处理过程。  
可以减少不必要的函数命名，保持代码的简洁和通用。  
但是没必要把一切函数都变成`pointfree`,过犹不及。

### debug  
因为都是纯函数，所以只需要看函数的返回值是否和预期的相同就可以了。  
这里可以用一个很简单的，不纯的`trace`函数来帮助追踪代码的执行情况。  
````javascript
const trace = (des, x) => {
    console.log(des, x);
    return x;
}
````
可以将`trace`插入到任意的函数后面，来查看它执行后的结果。就像下面这样：  
`compose(f, trace('after g'), g);`  
会在控制台输出调用`g`函数后的结果。

### 范畴论（Category theory）
这玩意是很高深的数学内容了，网上搜了一些，要想学懂得先学会抽代，图论，群论，然后再转范畴论。所以就简单的看一下吧。。。  
范畴论中最重要的概念就是范畴。  
一个范畴又下面几个集合组成： 
- 对象的集合(a collection of objects)
- 态射的集合(a collection of morphisms)
- 态射的组合(a notion of composition on the morphisms)
- 特殊的identity态射(a distinguished morphisms called identity)  

对应到编程中：
- 对象的集合就是数据类型，像String，Boolean，Number，Object等，将一个数据类型视作一类可能值的集合。
- 态射就是纯函数。
- 态射的组合就是本文介绍的组合。
- identity这个态射一个特殊的纯函数：  
`const id = x => x;`  
它是组合操作的单位元：  
````javascript
compose(f, id) == compose(id, f) == f;
````
可以作为替代定值的函数。  
据说很有用，暂时还没理解到。
