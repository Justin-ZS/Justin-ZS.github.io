---
layout: post  
title: 《JavaScript高级程序设计》学习笔记（01） 
date:   2017-6-9  
category: "JavaScript"
---
# JavaScript简介  
## JavaScript由三部分组成
 1. **ECMAScript**：由ECMA-262定义，提供核心功能。  
 2. **DOM**：文档对象模型， 提供访问和操作网页的方法和接口。
 3. **BOM**：浏览器对象模型，提供与浏览器交互的方法和接口。
## ECMAScript
`ECMAScript`是一门弱类型的动态脚本语言，本身不包含输入输出。  
同时也是一门"嵌入式"语言，可以嵌入不同的宿主环境。宿主环境不仅提供`ECMAScript`的实现，还会提供语言的扩展。  
`ECMAScript`的前端宿主环境通常是浏览器，所以真正意义上的前端`JavaScript`由上述三个部分组成。    
## DOM
`DOM`是针对`XML`但经过扩展用于`HTML`的`API`。将整个页面映射成一个多节点的树，页面中的每个部分都是某种类型的节点。  
`DOM`有三个级别: [详细内容](https://developer.mozilla.org/fr/docs/DOM_Levels)
 - **DOM Level 1**  
   最基础的`DOM`标准，98年发布。
    1. *DOM Core 1*：规定如何映射基于XML的文档结构。
    2. *DOM HTML*：在前者的基础上加以扩展，添加了针对HTML的对象和方法。  

 - **DOM Level 2**  
   一级的目标主要是映射文档结构，二级增加了很多扩展和模块，一共六个部分：
    1. *DOM Core 2*：扩展了`DOM Core 1`,专门为XML增加了新的接口。例如`getElementById`方法。
    2. *DOM HTML 2*: 基于`DOM Core 2`，允许脚本动态地访问和更新HTML文档的内容和结构。例如`contentDocument`属性。  
    3. *DOM View*：定义了跟踪不同文档视图的接口。例如应用`CSS`前后的文档。
    4. *DOM Event*：定义了事件和处理事件的接口。
    5. *DOM Style*: 定义了基于`CSS`为元素应用样式的接口。
    6. *DOM Traversal and Range*：定义了遍历和操作文档树的接口。前者例如,`NodeIterator`和`TreeWalker`。  

 - **DOM Level 3**
   三级进一步扩展引入了更多模块：
    1. *DOM Core 3*：扩展了`DOM Core 2`。例如新增的`adoptNode`方法`textContent`属性。
    2. *DOM Load and Save*：定义了加载和保存文档的方法,即`XML`文档和`DOM`文档间的相互转换。
    3. *DOM Validation*：定义了验证文档的方法。
    4. *DOM Event 3*：扩展了`DOM Event 2`。主要关注键盘事件及处理方法。
    5. *DOM XPath*：提供了使用`XPath 1.0`来访问`DOM`树的功能.  

各个浏览器对`DOM`标准的支持里的不同。以`Mozilla`为例，对一，二级`DOM`标准的所有模块都支持得很好，部分支持三级`DOM`标准的某些模块，不支持`DOM Validation`，用不标准的方法(自身实现的`DOMParser`和`XMLSerializer`对象)支持`DOM Load and Save`。
  
## BOM
可以使用`BOM`访问浏览器对象，保护一系列的扩展：
- 弹出浏览器窗口
- 移动，缩放，关闭浏览器窗口
- 提供浏览器详细信息的`navigator`对象
- 提供加载页面详细信息的`location`对象
- 提供用户显示器分辨率详细信息的`screen`对象
- 支持`cookies`
- `XMLHttpRequest`等自定义对象  

没有统一的`BOM`标准，所以每个浏览器都有自己的实现。

# HTML中的JavaScript

## \<script\>标签  
主要使用\<script\>元素向HTML中插入JavaScript。  
有两种方式插入脚本：*直接嵌入脚本*和*加载外部脚本*
- 当解释器遇到script标签时，文档的解析将停止，并立即下载和执行脚本，执行完毕后继续解析文档。
- scritpt元素主要有4个属性: 
  1. **type**: 通常是`text/javascript` 
  2. **src**: 表示要执行的外部脚本地址，可以是外部域的文档(绕过同源策略)。如果包含了嵌入代码，则只会下载和执行外部脚本，忽略嵌入的代码。
  3. **defer**: 遇到设置了`defer`属性的\<script\>元素，文档的解析不会停止，其他线程下载脚本，待到文档解析完毕后，脚本才会执行。(对于嵌入脚本无效)
  4. **async**: 遇到设置了`async`属性的\<script\>元素，文档的解析不会停止，其他线程下载脚本，下载完毕后立即执行脚本并停止文档解析。待到脚本执行完毕后，继续解析文档。(对于嵌入脚本无效)

如果没有`defer`和`async`属性，浏览器会按照\<script\>元素在页面中的出现的顺序依次解析执行脚本。  
`defer`的脚本也是按照声明顺序执行的;而`async`的脚本不同，只要脚本下载完成，将会立即执行，未必会按照声明顺序执行。  
如果同时设置了`defer`和`async`，默认使用`async`。  

## 其他地方
除了\<script\>元素之外，JavaScript还可以出现在HTML标签的on事件中，以及一些标签的href，src等属性的伪协议(javascript:等)中。
````HTML
<img src="#" onerror="alert(1)" />
<iframe src="javascript:alert(1)"></iframe>
<a href="javascript:alert(1)">x</a>
````
Web安全需要关注这部分。
