---
layout: post
title: "Javascript 闭包"
description: 
headline: 
modified: 2016-02-12
categories: [javascript]
tags: [javascript, closure]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
url: /javascript/javascript-closure
---

#闭包

在开始之前，我们先来一个来自于伟大互联网的面试题，如果你的答案和浏览器的结果一样，那么恭喜你，你已经不需要再继续啦～如果不一样，也恭喜你，希望读完这个文章，你就能够找到自己的答案。

    function fun(variable1, variable2){
        console.log(variable2);
        return {
            fun: function(innerVariable) {
                return fun(innerVariable, variable1);
            }
        };
    }
    var a = fun(0); a.fun(1); a.fun(2); a.fun(3);
    var b = fun(0).fun(1).fun(2).fun(3);

输出的结果是什么？
正确答案是undefined 0 0 0 undefined 0 1 2。

看完上面的题目，你是什么感受呢？不要害怕，看完本文后，就知道为什么这个题目是只是“纸老虎”了。

本文将从以下几个方面来介绍一下javascript中的闭包：

- 定义，什么是闭包？
- 闭包的应用
- 闭包中的bad smell

##什么是闭包呢？
我们来看一个简单的例子。

    function makeAdder(captured){
        return function(free) {
            return free + captured;
        }
    }
    var add10 = makeAdder(10);
    add10(30);

这个makeAdder很奇怪，返回了一个函数，并且还引用了一个自由变量captured，其实这就是一个闭包。

**闭包**其实是词法闭包或函数闭包的简称，它是一个引用了**内部变量**的**函数**。
但它是不是仅仅就是一个函数呢？个人认为，不是的。
在闭包中，被引用的自由变量会和这一函数同时存在，即便离开了创造它的环境也不例外。因此，可以说闭包是由***函数***和与其相关的***引用环境***组合而成的实体。

为了更好的理解这个概念，我们可以再来看几个简单的例子。

1. 闭包不仅可以捕捉变量，还可以捕捉**函数**

        function average(v1, v2) { 
            console.log(v1); console.log(v2); 
            return (v1 + v2) / 2; 
        }
        function averageDamp(fun) {
            return function(n) {
            return average(n, fun(n));
        }
        }
        var averageSq = averageDamp(function(n) {return n * n;});
        averageSq(10);

2. 闭包中使用变量的值，需要到**创建**它的作用域中找
        
        var x = 10;
        function fn() { console.log(x); }
        function show(f) { 
            var x = 20; 
            (function() {
                f();
            }) ();
        }
        show(fn);
        
        根据这个原则，上述函数应当输出10而非20.

3. 闭包中局部变量是**引用**而非拷贝

        function sayHi() {
             var hi = 'hi';
             var say = function() {
                 console.log(hi);
             };
             hi = 'hi world';
             return say;
         }
         
         var temp = sayHi();
         temp();
         
        根据这个原则，上述调用会输出hi world
        
4. 每次函数调用的时候都创建一个**新**的闭包

            function outer() {
                var a = 0;
                function inner() {
                    a++;
                    console.log(a);
                }
                return inner;
            }
            
            var obj = outer();
            obj(); obj();
            var obj1 = outer();
            obj1(); obj1();
        
           根据这个原则，上述调用输出1，2，1，2
5. 变量的遮罩（**shadowing**）

        function captureShadow(shadowed) {
         	return function(shadowed) {
         	return shadowed + 1;
         }
         }
         var closureShadow = captureShadow(101);
         closureShadow(2);
         
        变量的shadowing同样在闭包中适用，所以上述调用的输出结果为3.
        
##闭包的应用
在理解闭包的应用场景之前，我们可以先对比一下如下的两段代码：

        function showObject(obj) {
         	return function() {
         		return obj;
         	}
         }
         var o = {a: 42};
         var showO = showObject(o);
         showO();
         o.newField = 100;
         showO();

在这样的代码段里面，我们可以看到对象o在最初只拥有一个名字为a的属性，但是我们通过o.newField改变了对象o的原有结构，当我们再次调用showO()的时候，可以看到***a:42, newField:100***被打印了出来。如果我们后续再此调用o.newField=200之类的语句，都会导致o的newField属性发生变化。

接下来，再看看一段代码：

        var pingpong =  function() {
         	var pv = 0;
         	return {
         		inc: function(n) {
         			return pv += n;
         		},
         		dec: function(n) {
         			return pv -= n;
         		} 
         	};
         }();
         
         pingpong.inc(10);
         pingpong.dec(2);
         
         pingpong.div = function(n) {
         	return pv/n;
         }
         
在这段代码中，pingpong只是暴露了一些函数，只有在调用这些函数的时候才能获取pv的值，用面向对象的说法来说，pv就说pingpong的私有变量。并且我们在调用pingpong.pv的时候只会得到一个undefined。
通过这两段代码的对比，我们看到，闭包所提供的最大魅力在于**封装**。为javascript的编写带来了另一种思路。

##闭包中的“bad smell”
虽然闭包给我们带来了很大的裨益，但是闭包对脚本性能却有一些负面影响。闭包会带来很大的内存消耗。

对比如下两段代码：

- 代码一：

        function MyObject(name, message) {
         	this.name = name.toString();
         	this.message = message.toString();
         	this.getName = function() {
         		return this.name;
         	};
         	this.getMessage = function() {
         		return this.message;
         	}
         }

- 代码二：

        function MyObject(name, message) {
         	this.name = name.toString();
         	this.message = message.toString();
         }
         MyObject.prototype.getName = function() {
         	return this.name;
         };
         MyObject.prototype.getMessage = function() {
         	return this.message;
         }

在代码一中，如果MyObject被用于生成很多对象，则系统会为这些生成的对象维护一个闭包的上下文，用于存储每个闭包里的name和message等信息。而在代码二中，通过函数原型链的方式则避免了闭包带来的性能问题。

更有意思的话题诸如[ie内存泄漏问题](http://bubkoo.com/2015/01/31/understanding-and-solving-internet-explorer-leak-patterns/){:target="_blank"}，欢迎点击链接研究。

***ps:***回到本文开篇的面试题，题目里面出现了3个fun，第一个fun是最外面的函数，第二个fun是第一个fun返回的对象的一个属性，第三个fun其实就是在调用第一个fun。

对于变量`a`来说，调用`fun(0)`的时候，`variable1`的值是0，`variable2`的值是undefined，所以一上来便输出了undefined。之后a就成了一个对象，拥有一个fun的属性，这个属性是一个函数。`a.fun(1)`即调用了闭包，这个时候`innerVariable`被赋值为1，而`variable1`仍然保持值0。之后的`a.fun(2)`和`a.fun(3)`都是在调用闭包，但是由于是同一个闭包的不同实例（如什么是闭包中的第4点所述）

对于变量`b`来说，调用`fun(0)`时，variable1的值是0，variable2的值是undefined，所以会输出undefined。之后链式调用`fun(1)`这个时候`innerVariable`被赋值为1，所以`fun(innerVariable, variable1)`被调用的时候，会输出1；之后又链式调用`fun(2)`，这个时候variable1的值已经为1，所以会输出1，以此类推，每一次的闭包对应的variable1的值都不一样。所以`b`打印出来的值为undefined 0 1 2。

至此，总结了javascript中关于闭包的比较基本的信息，希望看过本文的你，不再因为闭包而望而生怯。