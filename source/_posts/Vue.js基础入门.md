title: Vue.js基础入门
date: 2016-11-09 14:08:08
# 类别
categories:
  - 技术
# 标签
tags:
  - Vue.js
---
作者: 李纯利

Vue.js是一套构建用户界面的`渐进式框架`,Vue的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合;Vue.js 的目标是通过尽可能简单的 API 实现`响应的数据绑定和组合的视图组件`。
<!--more-->

一、模板语法


```javascript

        //数据绑定最常见的形式:{{ }}的文本插入.
        //HTML
        <div id='app'>
            <!-- 文本 -->
            <!-- v-once指令只能执行一次性地插值 -->
            <div v-once>{{message | capitalize}}</div>
            <!-- 纯HTML -->
            <div v-html="htmlStr"></div>
            <!-- 属性 -->
            <div v-bind:class="vueJs"></div>
            <!-- 使用JavaScript表达式 -->
            <div v-bind:title="num + 1"></div>
            <!-- 修饰符 -->
            <form v-on:submit.prevent="onSubmit"></form>
            <!-- 缩写 -->
            <!-- 完整语法 -->
            <div v-bind:class="vueJs"></div>
            <div v-on:click="greet"></div>
            <!-- 缩写 -->
            <div :class="vueJs"></div>
            <div @click="greet"></div>
        </div>
        //js
        var app = new Vue({
            el: '#app',
            data: {
                message: 'Hello Vue',
                htmlStr: '<span>HTML</span>',
                vueJs: 'vue-div',
                num: 2
            },
            filters: {
                capitalize: function (value) {
                  if (!value) return ''
                  value = value.toString()
                  return value..split('');
                }
            }
            methods: {
                onSubmit:function(event){
                    alert('hello!')
                },
                greet: function(){
                    alert('缩写.')
                }
            }
        })
    </script>

```

`v-`前缀在模板中是作为一个标示 Vue 特殊属性的明显标识。
二.计算属性

```javascript

    <div id='app'>
        <div>{{message}}</div>
    </div>
    var app = new Vue({
        el: '#app',
        data: {
            message: 'HelloVue'
        },
        reverseMessage: {
            greet: function(){
                return this.message.split('').reverse().join('');
            }
        }
    })

```
三.class与style绑定

语法:`v-bind:class`, `v-bind:style`


```javascript

    <div class="static"
     v-bind:class="{ active: isActive }">
    </div>
    <div v-bind:class="[activeClass, errorClass]">
    <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

    var app = new Vue({
        el: '#app',
        data: {
            activeClass: 'active',
            errorClass: 'errors',
            activeColor: 'red',
            fontSize: 30
        }
    })

```

这里也可以绑定返回对象的计算属性。
```javascript

    <div v-bind:class="classObject"></div>

    var app = new Vue({
        el: '#app',
        data: {
            classObject: {
                active: true,
                'text-danger': false
              }
        }
    })

```
四.条件渲染
语法:v-show,v-if,v-else
v-else必须紧跟在v-if,v-show后面,不然就不能被识别.
v-if与v-show的区别v-show始终保持在Dom中,用简单的display来切换的.
v-if有更高的切换消耗,如果频繁的切换还是使用v-show比较好.

```javascript

    <h1 v-if="ok">Yes</h1>
    <h1 v-else>No</h1>
    <h1 v-show="ok">Yes</h1>
    <template v-if="ok">
      <h1>1</h1>
    </template>
```
注意 v-show 不支持 `<template>` 语法.
五.列表渲染
语法: v-for
```javascript

    <div v-for="item of items"></div>
    <div v-for="item in items"></div>

```
如同 v-if 模板，你也可以用带有 v-for 的 `<template>` 标签来渲染多个元素块.
在自定义组件里，你可以像任何普通元素一样用 v-for.
```javascript

    <my-component v-for="item in items"></my-component>

```
六.事件处理器
语法:v-on:事件

事件修饰符:
.submit提交事件不再重载页面
.stop阻止单击事件冒泡
.prevent修饰符可以串联
.capture添加事件侦听器时使用事件捕获模式
.self只当事件在该元素本身（而不是子元素）触发时触发回调

按键修饰符:
全部的按键别名：
.enter,.tab,.delete (捕获 “删除” 和 “退格” 键),.esc,.space,.up,.down,.left,.right.

可以通过全局 config.keyCodes 对象自定义按键修饰符别名：
```javascript

    // 可以使用 v-on:keyup.f
    Vue.config.keyCodes.f = 113

```
七.表单控件绑定
v-model 指令在表单控件元素上创建双向数据绑定.

修饰符:
.lazy在默认情况下， v-model 在 input 事件中同步输入框的值与数据，但你可以添加一个修饰符 lazy ，从而转变为在 change 事件中同步.
.number只能输入number类型的值.
.trim去掉首尾空格




