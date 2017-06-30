title: 认识 Apache Lucene
date: 2016-08-31 18:00:04
categories: 搜索引擎
tags: [搜索引擎, apache lucene]
---

&emsp;&emsp;&emsp;&emsp;为了更深入地理解ElasticSearch的工作原理，特别是索引和查询这两个
过程，理解Lucene的工作原理至关重要。本质上，ElasticSearch是用Lucene
来实现索引的查询功能的。如果读者没有用过Lucene，下面的几个部分将
为您介绍Lucene的基本概念。

<!--more-->

## 熟悉 Lucene

&emsp;&emsp;&emsp;&emsp;读者也许会产生疑问，为什么ElasticSearch 的创造者最终采用Lucene
而不是自己开发相应功能的组件。我们也不知道为什么，因为我们不是决策者。
但是我们可以猜想可能是因为Lucene是一个成熟的、高性能的、可扩展的、轻
量级的，而且功能强大的搜索引擎包。Lucene的核心jar包只有一个文件，而且
不依赖任何第三方jar包。更重要的是，它提供的索引数据和检索数据的功能开
箱即用。当然，Lucene也提供了多语言支持，具有拼写检查、高亮等功能；但
是如果你不需要这些功能，你只需要下载Lucene的核心jar包，应用到你的项目
    中就可以了。

## 总体架构

介绍Lucene架构之前必须理解一些基本的概念,才能更好的理解Lucene的
架构,这些概念是:
___
- **Document**:它是在索引和搜索过程中数据的主要表现形式，或者称“载体”，
承载着我们索引和搜索的数据,它由一个或者多个域(Field)组成。
- **Field***:它是Document的组成部分，由两部分组成，名称(name)和值(value)。
- **Term**:它是搜索的基本单位，其表现形式为文本中的一个词。
- **Token**:它是单个Term在所属Field中文本的呈现形式，包含了Term内容、
Term类型、Term在文本中的起始及偏移位置。
___

Apache Lucene把所有的信息都写入到一个称为**倒排索引**的数据结构中。
这种数据结构把索引中的每个Term与相应的Document映射起来，这与关系型数据
库存储数据的方式有很大的不同。读者可以把倒排索引想象成这样的一种数据结构：
数据以Term为导向，而不是以Document为导向。

*ElasticSearch Servier (document 1)*
*Mastering ElasticSearch (document 2)*
*Apache Solr 4 Cookbook (document 3)*
所以索引(以一种直观的形式)展现如下：

| Term | count | Docs |
| -- | -- | -- |
| 4 | 1 | <3> |
|Apache | 1 | <3> |
| Cookbook | 1 | <3> |
| ElasticSearch | 2 | <1> <2> |
| Mastering | 1 | <1> |
| Server | 1 | <1> |
| Solr | 1 | <3> |

正如所看到的那样，每个词都指向它所在的文档号(Document Number/Document ID)。
这样的存储方式使得高效的信息检索成为可能，比如基于词的检索(term-based query)。
此外，每个词映射着一个数值(Count)，它代表着Term在文档集中出现的频繁程度。
当然，Lucene创建的真实索引远比上文复杂和先进。这是因为在Lucene中，词向量(由单
独的一个Field形成的小型倒排索引，通过它能够获取这个特殊Field的所有Token信息)可
以存储；所有Field的原始信息可以存储；删除Document的标记信息可以存储……。核心在于
了解数据的组织方式，而非存储细节。

每个索引被分成了多个段(Segment)，段具有一次写入，多次读取的特点。只要形成了，
段就无法被修改。例如：被删除文档的信息被存储到一个单独的文件，但是其它的段文件
并没有被修改。

需要注意的是，多个段是可以合并的，这个合并的过程称为segments merge。经过强制
合并或者Lucene的合并策略触发的合并操作后，原来的多个段就会被Lucene创建的更大的
一个段所代替了。很显然，段合并的过程是一个I/O密集型的任务。这个过程会清理一些信息，
比如会删除.del文件。除了精减文件数量，段合并还能够提高搜索的效率，毕竟同样的信息，
在一个段中读取会比在多个段中读取要快得多。但是，由于段合并是I/O密集型任务，建议不好
强制合并，小心地配置好合并策略就可以了。

## 分析你的文本

问题到这里就变得稍微复杂了一些。传入到Document中的数据是如何转变成倒排索引的？
查询语句是如何转换成一个个Term使高效率文本搜索变得可行？这种转换数据的过程就称
为文本分析(analysis)

文本分析工作由analyzer组件负责。analyzer由一个分词器(tokenizer)和0个或者多
个过滤器(filter)组成,也可能会有0个或者多个字符映射器(character mappers)组成。

Lucene中的tokenizer用来把文本拆分成一个个的Token。Token包含了比较多的信息，
比如Term在文本的中的位置及Term原始文本，以及Term的长度。文本经过tokenizer处
理后的结果称为token stream。token stream其实就是一个个Token的顺序排列。
token stream将等待着filter来处理。

除了tokenizer外，Lucene的另一个重要组成部分就是filter链，filter链将用来处理
Token Stream中的每一个token。这些处理方式包括删除Token,改变Token，甚至添加新
的Token。Lucene中内置了许多filter，读者也可以轻松地自己实现一个filter。有如下
内置的filter：

- **Lowercase filter**：把所有token中的字符都变成小写
- **ASCII folding filter**：去除tonken中非ASCII码的部分
- **Synonyms filter**：根据同义词替换规则替换相应的token
- **Multiple language-stemming filters**：把Token(实际上是Token的文本内容)
转化成词根或者词干的形式。

所以通过Filter可以让analyzer有几乎无限的处理能力：因为新的需求添加新的Filter就可以了。

#### ***索引和查询***


在我们用Lucene实现搜索功能时，也许会有读者不明觉历：上述的原理是如何对索引过程
和搜索过程产生影响？

索引过程：Lucene用用户指定好的analyzer解析用户添加的Document。当然Document中
不同的Field可以指定不同的analyzer。如果用户的Document中有title和description
两个Field，那么这两个Field可以指定不同的analyzer。

搜索过程：用户的输入查询语句将被选定的查询解析器(query parser)所解析,生成多个
Query对象。当然用户也可以选择不解析查询语句，使查询语句保留原始的状态。
在ElasticSearch中，有的Query对象会被解析(analyzed)，有的不会，
比如：前缀查询(prefix query)就不会被解析，精确匹配查询(match query)
就会被解析。对用户来说，理解这一点至关重要。

对于索引过程和搜索过程的数据解析这一环节，我们需要把握的重点在于：倒排索引中词应
该和查询语句中的词正确匹配。如果无法匹配，那么Lucene也不会返回我们喜闻乐见的结果。
举个例子：如果在索引阶段对文本进行了转小写(lowercasing)和转变成词根形式(stemming)
处理，那么查询语句也必须进行相同的处理，不然就搜索结果就会是竹篮打水——一场空。

## Lucence查询语言

ElasticSearch提供的一些查询方式(query types)能够被Lucene的查询解析器(query parser)
语法所支持。由于这个原因，我们来深入学习Lucene查询语言，了解其庐山真面目吧。

#### ***基础语法***

用户使用Lucene进行查询操作时，输入的查询语句会被分解成一个或者多个Term以及逻辑运算符号。
一个Term，在Lucene中可以是一个词，也可以是一个短语(用双引号括引来的多个词)。如果事先设
定规则：解析查询语句，那么指定的analyzer就会用来处理查询语句的每个term形成Query对象。

一个Query对象中会存在多个布尔运算符，这些布尔运算符将多个Term关联起来形成查询子句。
布尔运算符号有如下类型：

+ **AND(与)**:给定两个Term(左运算对象和右运算对象)，形成一个查询表达式。只有两个
Term都匹配成功，查询子句才匹配成功。比如：查询语句"apache AND lucene"的意思是匹
配含apache且含lucene的文档。
+ **OR(或)**:给定的多个Term，只要其中一个匹配成功，其形成的查询表达式就匹配成功。
比如查询表达式"apache OR lucene"能够匹配包含“apache”的文档，也能匹配包含"lucene"
的文档，还能匹配同时包含这两个Term的文档。
+ **NOT(非)**: 这意味着对于与查询语句匹配的文档，NOT运算符后面的Term就不能在文档中
出现的。例如：查询表达式“lucene NOT elasticsearch”就只能匹配包含lucene但是不含
elasticsearch的文档。

此外，我们也许会用到如下的运算符：
+ **+**这个符号表明：如果想要查询语句与文档匹配，那么给定的Term必须出现在文档中。
例如：希望搜索到包含关键词lucene,最好能包含关键词apache的文档，可以用如下的查询
表达式："+lucene apache"。
+ **-**这个符号表明：如果想要查询语句与文档匹配，那么给定的Term不能出现在文档中。
例如：希望搜索到包含关键词lucene,但是不含关键词elasticsearch的文档，可以用如下
的查询表达式："+lucene -elasticsearch"。

如果在Term前没有指定运算符，那么默认使用OR运算符。
此外，也是最后一点：查询表达式可以用小括号组合起来，形成复杂的查询表达式。比如：
elasticsearch AND (mastering OR book)

#### 多域查询

当然，跟ElasticSearch一样，Lucene中的所有数据都是存储在一个个的Field中，
多个Field形成一个Document。如果希望查询指定的Field,就需要在查询表达式中指
定Field Name(此域名非彼域名)，后面接一个冒号，紧接着一个查询表达式。例如：
查询title域中包含关键词elasticsearch的文档，查询表达式如下：
    title:elasticsearch
也可以把多个查询表达式用于一个域中。例如：查询title域中含关键词elasticsearch
并且含短语“mastering book”的文档，查询表达式如下：
    title:(+elasticsearch +"mastering book")
当然，也可以换一种写法，作用是一样的：
    +title:elasticsearch +title:"mastering book")

#### 词语修饰符

除了可以应用简单的关键词和查询表达式实现标准的域查询，Lucene还支持往查询表达式中
传入修饰符使关键词具有变形能力。最常用的修饰符，也是大家都熟知的，就是通配符。
Lucene支持?和\*两种通配符。?可以匹配任意单个字符，而\*能够匹配多个字符。


***请注意出于性能考虑，默认的通配符不能是关键词的首字母。***

此外，Lucene支持模糊查询(fuzzy query)和邻近查询(proximity query)。语法规则是查
询表达式后面接一个~符号，后面紧跟一个整数。如果查询表达式是单独一个Term，这表示我们
的搜索关键词可以由Term变形(替换一个字符，添加一个字符，删除一个字符)而来，即与Term
是相似的。这种搜索方式称为模糊搜索(fuzzy search)。在~符号后面的整数表示最大编辑距离。
例如：执行查询表达式 "writer~2"能够搜索到含writer和writers的文档。

当~符号用于一个短语时，~后面的整数表示短语中可接收的最大的词编辑距离(短语中替换一个词，
添加一个词，删除一个词)。举个例子,查询表达式title:"mastering elasticsearch"只能匹
配title域中含"mastering elasticsearch"的文档，而无法匹配含"mastering book elasticsearch"
的文档。但是如果查询表达式变成title:"mastering elasticsearch"~2,那么两种文档就都
能够成功匹配了。

此外，我们还可以使用加权(boosting)机制来改变关键词的重要程度。加权机制的语法是一个^符
号后面接一个浮点数表示权重。如果权重小于1，就会降低关键词的重要程度。同理，如果权重大于
1就会增加关键词的重要程度。默认的加权值为1。可以参考 第2章 活用用户查询语言 的 Lucene
默认打分规则详解 章节部分的内容来了解更多关于加权(boosting)是如何影响打分排序的。

除了上述的功能外，Lucene还支持区间查询(range searching),其语法是用中括号或者}表示区间。
例如：如果我们查询一个数值域(numeric field)，可以用如下查询表达式：

    price:[10.00 TO 15.00]

这条查询表达式能查询到price域的值在10.00到15.00之间的所有文档。
对于string类型的field，区间查询也同样适用。例如：

    name:[Adam TO Adria]

这条查询表达式能查询到name域中含关键词Adam到关键词Adria之间关键词(字符串升序，且闭区间)的文档。
如果希望区间的边界值不会被搜索到，那么就需要用大括号替换原来的中括号。例如，查询price域中价
格在10.00(10.00要能够被搜索到)到15.00(15.00不能被搜索到)之间的文档，就需要用如下的查询表达式：

    price:[10.00 TO 15.00}

处理特殊字符

如果在搜索关键词中出现了如下字符集合中的任意一个字符，就需要用反斜杠(\\)进行转义。字符集合
如下： +, -, &&, || , ! , (,) , { } , [ ] , ^, " , ~, *, ?, : , \, / 。
例如，查询关键词 abc"efg 就需要转义成 abc\"efg。