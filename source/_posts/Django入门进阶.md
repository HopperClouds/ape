title: Django入门进阶
author: Adam
date: 2016-11-18 00:58:51
# 类别
categories:
  - blog
# 标签
tags:
  - Django
  - Python
---
作者：Adam at 2016-11-18 00:58:51

我们前面讲了快速熟悉`Django`[`架构网站的方式`](http://hopperclouds.github.io/2016/10/29/Django%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/)，接下来我们来深入了解一下`Django框架`。它不仅提供了建站的方法，也提供了一整套的从开发简单的网站到一个完整维护迭代的开发模式，下面几个方面非常重要。

<!-- more -->

## 学会调试
* 单步调试
直接在代码中插入下面这一行：
```python
#polls/views.py
from django.shortcuts import get_object_or_404, render
from .models import Question
import pdb; pdb.set_trace() #这一行
```
  运行程序，就可以在服务器端命令行中一步步的跟踪执行结果。
```python
[26/Nov/2016 14:17:49] "GET /polls/ HTTP/1.1" 200 72
Performing system checks...
> /myFirstDjango/mysite/polls/views.py(6)<module>()
-> def index(request):
(Pdb) l ＃这是调试命令：展示当前运行代码
  1  	from django.shortcuts import get_object_or_404, render
  2
  3  	from .models import Question
  4  	import pdb; pdb.set_trace()
  5  	# Create your views here.
  6  ->	def index(request):
  7  	    latest_question_list = Question.objects.order_by('-pub_date')[:5]
  8  	    context = {
  9  	        'latest_question_list': latest_question_list,
 10  	    }
 11  	    return render(request, 'polls/index.html', context)
(Pdb) n ＃这是调试命令: 下一步
> /Users/gzadamlv/myFirstDjango/mysite/polls/views.py(13)<module>()
-> def detail(request, question_id):
(Pdb) h ＃这是调试命令：获取帮助，查看所有可用命令
Documented commands (type help <topic>):
========================================
EOF    bt         cont      enable  jump  pp       run      unt
a      c          continue  exit    l     q        s        until
alias  cl         d         h       list  quit     step     up
args   clear      debug     help    n     r        tbreak   w
b      commands   disable   ignore  next  restart  u        whatis
break  condition  down      j       p     return   unalias  where
...
```

* Shell 调试方式
用下面的命令进入命令行
```bash
(myFirstDjango)➜  mysite git:(master) ✗ python manage.py shell
Python 2.7.10 (default, Oct 23 2015, 19:19:21)
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```
通过在命令行引入程序中使用的模块、函数来跟踪返回结果。
```bash
>>> django.setup()
>>> import mysite
>>> from django.shortcuts import get_object_or_404, render
>>> from polls.models import Question
>>> latest_question_list = Question.objects.order_by('-pub_date')[:5]
>>> print(latest_question_list)
<QuerySet [<Question: 你会学习python吗？>]>
```

## 学会测试
修改models.py
```python
from django.utils import timezone
import datetime
```
给class Question增加一个方法
```python
def was_published_recently(self):
    return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```
添加测试文件：polls/tests.py
```python
import datetime
from django.utils import timezone
from django.test import TestCase
from .models import Question

class QuestionMethodTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() should return False for questions
        whose pub_date is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```
运行测试
```bash
(myFirstDjango)➜  mysite git:(master) ✗ python manage.py test polls
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionMethodTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/gzadamlv/myFirstDjango/mysite/polls/tests.py", line 14, in test_was_published_recently_with_future_question
    self.assertIs(future_question.was_published_recently(), False)
AssertionError: True is not False

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
```
修改was_published_recently方法
```python
#return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
now = timezone.now()
return now - datetime.timedelta(days=1) <= self.pub_date <= now
```
再运行测试
```python
(myFirstDjango)➜  mysite git:(master) ✗ python manage.py test polls
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

## 学会写文档
> 安装： `pip install Sphinx`
> 运行 `sphinx-quickstart` 命令(下面几项要选`yes`)

```bash
Please indicate if you want to use one of the following Sphinx extensions:
> autodoc: automatically insert docstrings from modules (y/n) [n]: y
> doctest: automatically test code snippets in doctest blocks (y/n) [n]: y
> intersphinx: link between Sphinx documentation of different projects (y/n) [n]: y
> todo: write "todo" entries that can be shown or hidden on build (y/n) [n]: y
> coverage: checks for documentation coverage (y/n) [n]: y

```

> 生成文档： `make html`
> 打开网页：`_build/html/index.html`

默认模版不太好看，我们新加一个主题：`pip install sphinx_rtd_theme`
在`conf.py`最后增加：
```python
import sphinx_rtd_theme
html_theme = 'sphinx_rtd_theme'
```
再`make html`，初始化文档生成成功：
![2016-11-30-截图](http://img.pinbot.me:8080/uploads/2016/11/30/blob_1480436514924.png "blob_1480436514924.png")

但是我们会发现并没有看到程序中的文档，别着急，还差两步。

先用`sphinx-apidoc`把`app`目录中`python`程序的文档内容提取出来，保存到`api`这个目录。
```python
sphinx-apidoc -o api ./app -f
```
最后，我们需要编辑index.rst这个文件：
```python
Contents:

.. toctree::
   :maxdepth: 2

   api/polls.rst  #增加这行
```
再次`make html`，大功告成～

![2016-11-30-截图](http://img.pinbot.me:8080/uploads/2016/11/30/blob_1480436951111.png "blob_1480436951111.png")

这是提取出来的具体文档：

![2016-11-30-截图](http://img.pinbot.me:8080/uploads/2016/11/30/blob_1480437014840.png "blob_1480437014840.png")

另外：如果`make html`出现下面错误
```python
sphinx autodoc ImportError: No module named
```
可以试试这个：
```python
export PYTHONPATH=$(pwd)
```
别小看上面这三步，其实还有很多细节没能和大家演示，师傅领进门，修行靠个人哦～