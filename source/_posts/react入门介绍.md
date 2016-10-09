title: react入门介绍
date: 2016-09-06 18:30:30
tags:
---


## react.js介绍

### react.js的提出
react.js的首次提出是在2014年Facebook的f8大会上。顺便科普一下f8大会，f8大会是由Facebook组织的年度的技术峰会，之所以叫f8，就是看大家在8小时以内能做出哪些有意思的东西。
react.js称为颠覆式前端UI开发框架。目前基于html的前端开发变得越来越复杂，传统的开发方式基于来自服务器和来自用户输入的交互数据,动态反应到复杂界面的时候，代码量变得越来越大，难以维护。
比如，前端开发框架jquey，每次数据更新，必须手动把数据更新渲染到ui界面上,代码量极大。基于此，google推出的angular.js的双向数据绑定很好的解决了这个问题。但是angular.js也有自身的一些不足。1：angular过重，不适用于对性能要求特别高的站点。2：ui组件封装比较复杂，不利于重用。而react解决了所有的这些问题。
ReactJS官网地址：http://facebook.github.io/react/
Github地址：https://github.com/facebook/react

### react.js的特点
#### 1、就是轻，数据渲染响应非常快。
复杂或频繁的DOM操作通常是性能瓶颈产生的原因。React为此引入了虚拟DOM（Virtual DOM）的机制：在浏览器端用Javascript实现了一套DOM API。基于React进行开发时所有的DOM构造都是通过虚拟DOM进行，每当数据变化时，React都会重新构建整个DOM树，然后React将当前整个DOM树和上一次的DOM树进行对比，得到DOM结构的区别，然后仅仅将需要变化的部分进行实际的浏览器DOM更新。尽管每一次都需要构造完整的虚拟DOM树，但是因为虚拟DOM是内存数据，性能是极高的，而对实际DOM进行操作的仅仅是Diff部分，因而能达到提高性能的目的。
#### 2：组件化开发思想。
React推荐以组件的方式去重新思考UI构成，将UI上每一个功能相对独立的模块定义成组件，然后将小的组件通过组合或者嵌套的方式构成大的组件，最终完成整体UI的构建。

### react试用场景
react 这么厉害到底适用于哪些场景呢？
1、复杂场景下的高性能要求。
2、重用组件库，组件组合。

## react html、css基础实践
下面让我们来看看一组代码：

```html
<html>
<head>
  <script src="../build/react.js"></script>
  <script src="../build/react-dom.js"></script>
  <script src="../build/browser.min.js"></script>
  <style type="text/css">
   .redColor{
    color: red;
   }
  </style>
</head>
<body>
  <div id="example"></div>
  <script type="text/babel">
    var Hello = React.createClass({
      render:function(){
          var styleObj = {
            textDecoration:'underline'
          };
        return <div className="redColor" style={{fontSize:'18px'}}>Hello {this.props.name}</div>
      }
    });
    ReactDOM.render(
      <Hello name="World"/>,
      document.getElementById('example')
    );
  </script>
</body>
</html>
```

现在来解释一下这段代码
1.react用的是jsx，是facebook为react开发的一套语法糖。语法糖是计算机中添加的一种语法，对语言的功能没有影响，但是更方便程序员使用，增加可读性减少程序出错机会。类似的还有CoffeeScript、TypeScript等。最终都被解析库解析成js。这里引入的browser.js 就是jsx的解析库。作用是将 JSX 语法转为 JavaScript 语法。另外 `<script>` 标签的 type 属性为 `text/babel` 。表明这是jsx语法。

2.jsx为我们带来的便利就是，我们可以在js里写类dom的结构，比我们用原生js拼接字符串要简单方便许多。jsx语法允许我们生成原生的dom标签，还可以生成自定义标签。比如hello，这些统称为react components.通过调用ReactDOM将react components呈现在页面上。

3.ReactDOM.render是React的最基本方法，用于将模板转为 HTML 语言，并插入指定的 DOM 节点。第一个参数是要插入的components，第二个参数是要插入的容器。
自定义的标签是通过React.createClass申明，参数是一个js的对象。return的内容就是渲染的结构。遇到 HTML 标签（以 `<` 开头），就用 HTML 规则解析；遇到代码块（以 `{` 开头），就用 JavaScript 规则解析。

4.给标签添加css属性有两种：
 一种：用外联样式，注意这里是className，因为这是jsx语法，class在js中已经是一个保留关键字。
 二种：内联样式。在react中内联样式必须用样式对象来表示，在react中内联样式必须用样式对象来表示，必须用驼峰。且用｛｛｝｝包裹。这里为什么要用｛｛｝｝，让我们再看看另一种写法就一目了然了。

````html
  <div id="example"></div>
  <script type="text/babel">
    var Hello = React.createClass({
      render:function(){
          var styleObj = {
            fontSize:'18px'
          };
        return <div className="redColor" style={styleObj}>Hello {this.props.name}</div>
      }
    });
    ReactDOM.render(
      <Hello name="World"/>,
      document.getElementById('example')
    );
  </script>
````

这里申明一个样式对象，用｛｝包裹就能以js的方式来解析。和`｛｛fontSize:"18px"｝｝`异曲同工。
可以隐约的看到，react的组件通过样式对象的申明可以，react组件是html、css、js的集合，成为真正意义上的独立组件。


这次我们简单介绍了react的由来、特点、应用场景。以及，jsx语法糖，如何生成自定义标签，插入节点，添加css样式，这些都是react的基础，接下来，我们继续react compenents的生命周期。

