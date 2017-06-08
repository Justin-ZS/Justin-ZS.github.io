---
layout: post  
title:  JavaScript函数式编程指南(01) 
date:   2017-6-8  
category: "JavaScript FP"  
---
# JavaScript函数式编程指南（01）
[JS函数式编程指南](https://drboolean.gitbooks.io/mostly-adequate-guide/)的学习笔记。  
如果不习惯英文，可以看[中文版](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)。  

## 函数是一等公民
所谓的一等公民，即是函数像普通变量一样，可以是参数，也可以是返回值，当然也可以赋值，没什么特殊的。  
### **不要做无意义的函数包装！**
````javascript
const hi = name => `Hi ${name}`;
const greeting = name => hi(name);
//上面的greeting函数就是毫无意义的函数包装
//完全可以这样
const greeting = hi；
````
````javascript
//下面的几种写法是等价的
const getServerStuff = function(callBack) {
    return ajaxCall(json => callBack(json));
}  
//json => callBack(json) 是对callBack函数的无意义包装。
const getServerStuff = function(callBack) {
    return ajaxCall(callBack);
}  
//继续剔除无意义的函数包装。
const getServerStuff = ajaxCall;
````
### **避免使用this**  
`this`的指向问题不用多说。函数式编程中，`this`永远是最后的选择。 

## 纯函数（Pure Function）
> A pure function is a function that, given the same input, will always return the same output and does not have any observable side effect.  
  
*Eric Elliott*有一篇很好的文章介绍了[什么是纯函数](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)。
### 纯函数的三个特点：
 - 相同的输入总是得到相同的输出
 - 没有副作用
 - 只依赖传入参数


### 什么是副作用？
> A side effect is a change of system state or observable interaction with the outside world that occurs during the calculation of a result.  

简而言之，和函数外部的任何交互都是副作用。
### 数学中的函数
> A function is a special relationship between values: Each of its input values gives back exactly one output value.    

在数学领域，函数是一种映射关系，将一个集合中的元素映射(mapping)成另一个集合里的唯一元素。(唯一意指可以多对一，不能一对多)  
纯函数就是数学中的函数，它只是映射。

### 纯函数的好处

#### 可缓存性(Cacheable)
````javascript
var memoize = function(f) {
  var cache = {};

  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};
````
#### 可移植性／自文档化（Portable / Self-Documenting）
纯函数不依赖于外部环境，意味着可移植性更高。  

#### 可测试性（Testable）
一目了然

#### 合理性（Reasonable）
纯函数具备引用透明性(`referential transparency`)，即可以在不影响程序运行的情况下，用函数返回值直接替换函数调用。  
对于引用透明函数可以用“等式推导”(`equational reasoning`)的方法来分析函数。  
等式推导类似手动带入参数执行函数，将函数调用简化(譬如，删去未进入的`if`分支)。最后得到清晰的函数内容，利于分析和优化。

如果还不理解可以看看[什么是引用透明性](https://stackoverflow.com/questions/210835/what-is-referential-transparency)
和[什么是等式推导](https://www.zhihu.com/question/28179914/answer/39754668).

#### 并行代码
因为不需要访问共享的内存， 可以并行运行任意纯函数,

## 柯里化(curring)  
> You can call a function with fewer arguments than it expects. It returns a function that takes the remaining arguments.
  
即可以传递部分参数来调用函数，它会返回一个接受剩余参数的新函数。
````javascript
const _ = require('ramda');
const sum = _.curry(function(a, b, c) {
    return a + b +　c;
});
//柯里化后的sum函数可以这样调用：
sum(1, 2, 3);
sum(1, 2)(3);
sum(1)(2, 3);
sum(1)(2)(3);

````  
当谈论纯函数的时候，总是说它接受一个输入返回一个输出。柯里化的函数就是这样。

### 柯里化的好处

#### 预加载(Pre-load)
通过简单地传递几个参数，就能动态创建实用的新函数。
传给函数一部分参数通常也叫做局部调用（partial application），能够大量减少样板文件代码（boilerplate code）。

### 柯里化的本质
`curry`函数的本质是闭包(closure)。
````javascript
//自己实现的curry函数，缺陷：函数上的length属性无效。
const curry = function(f, length) {
  const arity = length || f.length;
  return function(...args) {
    const ctx = this || null;
    if (args.length >= arity)
      return f.apply(ctx, args);
    else 
      return curry((...args2) => f.apply(ctx, args.concat(args2)), arity - args.length);
  }
}
````
