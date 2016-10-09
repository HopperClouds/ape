title: 数据处理/分析/可视化飞艇(zeppelin)介绍
date: 2016-09-01 01:35:55
updated: 2016-09-01 01:35:55
# 类别
categories:
  - 大数据
# 标签
tags:
  - spark
  - pyspark
---
author: likaiguo

# Zeppelin(飞艇)

Zeppelin思维导图

![2016/9/1/截图](http://img.pinbot.me:8080/uploads/2016/9/1/blob_1472665482681.png "blob_1472665482681.png")

推荐查看思维导图中的各个链接,尤其是[官方文档](https://zeppelin.apache.org/docs/0.6.1/)和[中文翻译]().

## 快速搭建Zeppelin环境

### 安装过程
- 到官网下载二进制包（http://zeppelin.apache.org/download.html）
- 解压到本地(保证已经设置好Java环境)

### 运行Zeppelin服务

bin/zeppelin-daemon.sh start|stop|restart

浏览器中打开：http://localhost:8080 即可进入Zeppelin首页。

![2016-9-1-截图](http://img.pinbot.me:8080/uploads/2016/9/1/blob_1472668081493.png "blob_1472668081493.png")

## Zeppelin是什么?

A web-based notebook that enables interactive data analytics.
You can make beautiful data-driven, interactive and collaborative documents with SQL, Scala and more.

一款基于web页面的笔记本(类似ipython中的notebook),其提供**交互式**数据分析功能.
使用Zeppelin(飞艇)我们能使用如SQL,Scala等后端语言制作出数据驱动的,交互式的并且易于协作的文档.


## Zeppelin基本概念

### 1.支持多种后端语言,[Interpreter(解释器)](https://zeppelin.apache.org/docs/latest/manual/interpreters.html)

![可以使用的解释器](https://zeppelin.apache.org/assets/themes/zeppelin/img/available_interpreters.png)

抽象出解释器概念,运行各种语言和数据处理后端工具.在Zeppelin中解释器被设计为可插拔的模块.
目前支持各种各样的解释器,如上图所示包括Apache Spark, Python, JDBC, Markdown and Shell等等.

同时也可以[写自己需要的解释器](https://zeppelin.apache.org/docs/0.6.1/manual/interpreters.html).

1. 在现有的解释器的基础上配置对应的参数生成新的解释器
2. 写相关的Java或者scala程序开发更加特定的解释器[参考文献2]


### 2.强大的数据可视化能力

![2016/9/1/截图](http://img.pinbot.me:8080/uploads/2016/9/1/blob_1472669411484.png "blob_1472669411484.png")

Zeppelin具有较为常用的数据可视化的图表. 如上图所示,表格,柱状图,饼图,趋势图,散点图一应俱全.

数据可视化不仅限于Spark SQL,任意一种语言的表格输出都能被完美转译成对应的图表.

并且能够导出对应的CSV等类型数据.


### 3.数据透视表

![](https://zeppelin.apache.org/assets/themes/zeppelin/img/screenshots/pivot.png)
Apache Zeppelin aggregates values and displays them in pivot chart with simple drag and drop. You can easily create chart with multiple aggregated values including sum, count, average, min, max.

飞艇能够在页面上通过简单的拖拽进行各种聚合操作,并且展示出对应的数据透视表.
同时也可以很容易通过求和,计数,平均,最小,最大创建各种聚合值的图表.


### 4.动态表格

![](https://zeppelin.apache.org/assets/themes/zeppelin/img/screenshots/dynamicform.png)
Zeppelin可以通过动态表格方式在notebook中添加诸如: 文本框,复选框,单选框等表单元素.
通过这种方式,我们可以快速进行对应的动态操作.


典型应用:

![](http://img.blog.csdn.net/20150523163620563)

这里${maxAge=30}的写法表示一个文本框元素,并且默认值为30。当修改对应的值是下方的图表会对应产生变化。

https://zeppelin.apache.org/docs/latest/manual/dynamicform.html

文档很重要.

遇到一个奇怪的问题:
当使用下拉框时,对应的值可以实时变化. 其他如文本框,复选框都不实时变化,需要点击三角形run按钮才能生效.


### 5.将notebook共享给他人,更好的协作

![](https://zeppelin.apache.org/assets/themes/zeppelin/img/screenshots/publish.png)

- 可以直接将写好的notebook发送给其他人,放入工程的notebook目录下即可
- 可以将生成的图表共享给他人(复制对应的link,参见文献2和官方文档)



## Zeppelin使用场景(特点)?

apache zeppelin应该会很吸引分布式计算、数据分析从业者,是个值得把玩的算比较前卫的项目。

- 代码量少，
- 模块很清楚，
- 可以尝试接入多种不同计算引擎，
- 实时任务运行、可视化效果
- 没有过多复杂的操作，只是区分了多个notebook，
- 每个notebook里做单独的分析处理工作，流程和结果会被保存下来。
- 此外，为spark做了更好的支持，比如默认是scala环境，默认sc已经创建好，即spark local可跑，默认spark sql有可视化效果。
- 一站式数据分析: 文档,不同工具集一应俱全


## Zeppelin怎样服务于我们的业务?

1. 应用于快速导入数据并且进行可视化
2. 将多种数据处理技术和语言融合在一起
3. 优美文档书写
4. 快速给客户提供数据可视化服务


## Zeppelin常见问题


## 参考文献

- [Apache Zeppelin简介](http://blog.csdn.net/laozhaokun/article/details/44803061)
- [Apache Zeppelin安装及介绍](http://blog.csdn.net/pelick/article/details/45934993)
- [让Spark如虎添翼的Zeppelin – 基础篇](http://www.flyml.net/2016/08/19/reinforce-spark-with-zeppelin-basic/)
- [Zeppelin 小试牛刀 – 使用Zeppelin展示MySQL的数据](http://www.flyml.net/2016/08/22/zeppelin-demo-mysql-data/)
- [Hadoop - Zeppelin 使用心得](http://www.cnblogs.com/smartloli/p/5148941.html)


