---
layout: post  
title:  JavaScript高级程序设计(02) 
date:   2017-6-18  
category: "JavaScript"  
---
## 基本概念
### 标识符
- 区分大小写  
- 第一个字符必须是字母，下划线（_）或美元符号（$)。
- 其他字符可以是字母，下划线，美元符号或*数字* 
- 采用`camelCase`的格式。

### 严格模式
在严格模式下code的解析和执行完全不同。  
#### 启用
- 整个脚本启用：在脚本等顶部添加`"use strict"`
- 指定函数启用：在函数体所有语句前添加`"use strict"` 

#### 区别
严格模式同时改变了语法和运行时行为，主要体现在以下几点：  
- 将问题转化为错误：
    1. 无法意外创建全局变量：
    ````javascript
    "use strict"
    mistypedVariable = 10;
    //ReferenceError
    ````
    2. 导致静默失败（不报错也没任何效果）的赋值语句抛出异常
    ````javascript
    "use strict";
    // 给不可写属性赋值
    var obj1 = {};
    Object.defineProperty(obj1, "x", { value: 42, writable: false });
    obj1.x = 9; // 抛出TypeError错误

    // 给只读属性赋值
    var obj2 = { get x() { return 17; } };
    obj2.x = 5; // 抛出TypeError错误

    // 给不可扩展对象的新属性赋值
    var fixed = {};
    Object.preventExtensions(fixed);
    fixed.newProp = "ohai"; // 抛出TypeError错误
    ````
    3. 函数的参数名唯一
    4. 禁止八进制语法
- 简化变量的使用
    1. 禁止使用with
    2. eval不再为上层上下文引入新变量。
    3. 禁止删除变量  
- 让eval和arguments更简单 
- 等等

#### 建议  
尽可能的采用严格模式  
不要使用`with,eval`  
使用`rest(...)`来替代`arguments`
 
### 变量
三种声明变量的方式：
- var：声明一个变量。
- let：声明一个块作用域的变量
- const：声明一个块作用域，只读的静态变量。

最初只有var操作符。它有一些缺陷：
- 变量提升（variable hoisting）
- 无块作用域（只有全局作用域和函数作用域两种）

ES2015新加入的let和const操作符改进了这两点。
- 无变量提升  

````javascript
//在声明前引用会抛出错误
"use strict"
console.log(x);
let x = 1;
//ReferenceError

//同一个作用域内无法重复声明
"use strict"
let x = 1;
let x = 2;
//SyntaxError
````  

- 引入块作用域：一对大括号就是一个块作用域。

#### 建议
不使用var，只使用let和const。
## 数据类型
基本类型6种：
- boolean,true和false
- null,表示空对象，注意Null，NULL是错误的写法  
- undefined,表示未被初始化的变量或者尚未声明对变量
- number，数值
- string，字符串
- symbol，ES2015新加入的类型

复杂类型1种
- object

### typeof 
typeof操作符可以查看变量类型
````javascript
"use strict";

typeof 1 === "number";
typeof 'word' === "string";
typeof true === "boolean";
typeof symbol() === "symbol";
//undefined
typeof undefined === 'undefined';
typeof declaredButUndefinedVariable === 'undefined';
typeof undeclaredVariable === 'undefined';

typeof {} === "object";
typeof null === "object"; //所以说null表示空对象

typeof function() {} === "function";
//基本类型的包装对象
typeof new Boolean(true) === 'object'; 
typeof new Number(1) === 'object'; 
typeof new String('abc') === 'object'; 
````
对于尚未声明的变量，只能执行一项操作，即使用typeof操作符检测其数据类型。
### Number
JavaScript中使用`IEEE754`格式来表示整数和浮点数（双精度）。  
采用64位存储，1位符号位，11位阶码（表示指数），52位尾数（表精度）。  
下面是JavaScript中数值的计算方法。
> (-1)^符号位 * 1.xxx...xxx * 2^尾数

因为总是1.xxx的形式，所以1不需要存储，只需要存储小数点后的尾数即可（因此精度为52 + 1 = 53位）。  
计算阶码时需要加上基数，为*2^(n - 1) - 1*,其中n表示阶码位数。11位阶码的基数为*2^10 - 1*, 即*1023*.  
所以指数范围为*2^(-1023) ~ 2^1024*
````javascript
Math.MAX_VALUE; //1.7976931348623157e+308,即（2 - 2^-52）* 2^1023 === 2^1024 - 2^971
Number.MAX_SAFE_INTEGER //9007199254740991,即2^53 -1
````
如果数值大小超过了最大值，则会被转化为`Infinity`;
> `Math.pow(10, 1000); //Infinity`
并不是所有的浮点数都能被准确的存储,很多时候只能是一个近似值，所以判断浮点数相等与否时要万分小心。
> `0.1 + 0.2 === 0.3; //false`

ES2015中引入了epsilon属性来解决这个问题  
> `Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; //true `  

整数也是同样的存储方式，用尾数来表示。所以最大的整数`Number.MAX_SAFE_INTEGER`为(2^53 - 1)，最大精度为15位。


#### 类型转换
有三个函数可以把非数值转换为数值：
- Number()
- parseInt()
- parseFloat()

>  一元操作符（+）的操作与Number()函数相同  

转换字符串时，Number()函数的转换规则不够合理，建议使用专门转换字符串的parseInt()函数.  
````javascript
'use strict';
//空字符串
Number(''); //0
parseInt('', 10); //NaN
//遇到非数值字符停止解析
Number('123abc'); //NaN
parseInt('123abc', 10); //123

````
parseInt()会识别多种整数格式（十进制，十六进制），务必传入第二参数，指定以哪种格式解析。  
parseInt()无法解析浮点数（.会被视为非数字字符），所以解析浮点数时使用pareseFloat()函数。  
parseFloat()只解析十进制数值,所以只有一个参数。如果解析结果为整数，则返回整数。

#### NaN  
即非数值（not a number），用来表示一个本来要返回数值的操作数位返回数值的情况。作为一个特殊数值，它有两个特点：
- 任何涉及NaN的操作都会返回NaN：
- NaN于任何值都不相等，包括其本身。  

使用isNaN函数来判断参数是否是NaN。


### String
字符串字面量即零个或多个以双引号（“）或单引号（‘）括起来的Unicode字符。
有一些特殊的字符：
- \n: 换行
- \r: 回车
- \xnn: 以十六进制代码nn表示的一个字符
- \unnnn: 以十六进制nnnn表示的一个字符

ECMAScript中的字符串时不可变的。所以要改变某个变量保存的字符串，先要销毁原来的字符串，然后创建新的字符串填充该变量。

#### 类型转换
有两种方法可以转换非字符串到字符串
- 基本上所有变量都有的toString()方法（除了null和undefined）
- String()函数。

toSting()方法会返回相应的字符串实现。  
String()方法可以转换任何类型的值：
- 如果值有toString()方法，就调用该方法
- 如果值是null，返回"null"
- 如果值时undefined，返回"undefined"  
 
## 参考资料
1. [Adding to Number.MAX_VALUE](https://stackoverflow.com/questions/10837670/adding-to-number-max-value)
2. [为什么0.1 + 0.2不等于0.3？](http://www.renfed.com/2017/05/13/float-number/)