title: JavaScript之递归
date: 2016-09-02 00:52:45
author: 纯利
# 类别
categories:
  - blog
# 标签
tags:
  - javascript
---
作者：纯利

那么什么叫递归呢？所谓**递归函数就是在函数体内调用本函数**。最简单的例子就是计算阶乘。0和1的阶乘都会被定义为1，更大的数的阶乘是通过计算1*1*...来求得的，每次增加1，直至达到要计算阶乘的那个数。

递归的缺点：如果递归函数的终止条件不明确或者缺少终止条件会导致函数长时间运行，是用户界面处于假死状态。值得注意的是：浏览器对递归的支持熟练与JS调用栈大小直接相关，当使用太多递归甚至超过最大调用栈容量时，浏览器会报错误信息，各个浏览器对报错的提示信息也不一样。

下面我们先来看一下一个经典的递归阶乘函数：
```javascript
    function test(num){
        if(num <= 1){
            return 1;
        }else{
            return num * test(num-1);
        }
    }
```
上面的的这个函数表面上没有什么问题，但是以下的代码却可能会导致问题：
```javascript
    var f = test;
    test = null;
    console.log(f(2));//报错 Uncaught TypeError: test is not a function
```
指向原始函数的引用就剩下一个，当调用f()函数时，而test已经不再是一个函数了，所以会导致错误，但是我们可以使用arguments.callee来解决这个问题。

大家都知道，arguments.callee是一个指向正在执行的函数的指针，因此可以用它来实现函数的递归调用,看如下代码：
```javascript
    function test(num){
        if(num <= 1){
            return 1;
        }else{
            return num * arguments.callee(num-1);
        }
    }
```
这样即使函数赋值给了另外一个变量，f()函数依然是有效的，所以递归调用能正常完成。而且这种方式在严格模式和非严格模式下都可以使用哦。
