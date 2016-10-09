---
title: 从__str__说开去
date: 2016-08-31 16:00:00
tags:
  - python
  - str
  - unicode_literals
  - Kxrr
categories:
  - 技术
author: Kxrr
---


## \__str\__ 和 \__repr\__

### 介绍
[`object.__str__`](https://docs.python.org/2/reference/datamodel.html#object.__str__)是python中一个常见的特殊方法, 会被内置函数被 `str` 和 `print` 调用。
常常与它一起出现的还有[`object.__repr__`](https://docs.python.org/2/reference/datamodel.html#object.__repr__), 类似地, 它会被内置函数 `repr` 调用。



### 区别

那么 `str` 和 `repr` 同样作为"informal string representation of instances", 有何区别?

用一句话概括就是:
> repr is for developers, str is for customers.

这点在IDE中调试时得以展现:

```python
class Student(object):

    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

    def __str__(self):
        return '{0}({1})'.format(self.name, self.grade)

    def __repr__(self):
        return '<Student>'


if __name__ == '__main__':
    student = Student(name='Roy', grade=11)
```

在debug模式下, pycharm将 `student` 展示为:

![](http://img.pinbot.me:8080/uploads/2016/8/31/blob_1472628279443.png)

简洁明了。

## \__unicode\__

出场率同样高的还有[`object.__unicode__`](https://docs.python.org/2/reference/datamodel.html#object.__unicode__), 和`object.__str__`作用类似, 但不同的是, `object.__unicode__` 返回的是一个unicode object, 而 `object.__str__` 返回的是string object。
这会引起一些问题, 特别是当你在使用python2中的unicode_literals时。

## unicode_literals

### UnicodeEncodeError

我们对上面的代码做一些修改:

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

class Student(object):

    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

    def __str__(self):
        return '{0}({1})'.format(self.name, self.grade)

    def __repr__(self):
        return '<Student>'

    def __unicode__(self):
        return '<Student>'
```

改动在于import了[`unicode_literals`](https://docs.python.org/2/reference/datamodel.html#object.__unicode__), 并为 `Student` 添加了一个 `__unicode__` 方法, 看起来好像没有什么问题。 但当你实例化一个 `Student` , 并将 `name` 指定为中文时:

```python
student = Student(name='罗伊', grade=11)
print student
```

报错了, `UnicodeEncodeError` 。 你或许精通python2的中文编码问题, 但也许并没有注意到这个问题。 
在使用django时遇到过 `[Bad Unicode data]` 这个东西, 问题是一样的, django在项目中也使用了 `unicode_literals` 。

### 问题在哪
问题在于 `object.__str__` 返回的必须为string object, 而使用 `unicode_literals` 之后返回的为unicode object, python2解释器会尝试用默认的编码(ascii)对其进行encode, 所以报错。

### 解决问题
`unicode_literals` 在python2中是个利器, 不能不用。 
接下来我们用两种方法来解决上面这个问题。

#### patch
这是一种经典的方法:

```python
import sys
reload(sys)
sys.setdefaultencoding('utf8')
```
重载 `sys` 并将 `defaultencoding` 从 `ascii` 修改为 `utf-8` , 对含中文的unicode object使用utf-8进行encode是可行的。

#### 装饰器
```
def force_encoded_string_output(func):
    if sys.version_info.major < 3:
        def _func(*args, **kwargs):
            return func(*args, **kwargs).encode('utf-8')

        return _func

    else:
        return func
```
使用 `force_encoded_string_output` 装饰 `object.__str__` 即可, 解决的思路和上面类似。


### 最佳实践
当你在python2中同时使用中文, `unicode_literals`, `__str__`, `__unicode__` 可以考虑下面的方式:

```python
from __future__ import unicode_literals


class Best(object):

    def __str__(self):
        return unicode(self).encode('utf-8')

    def __unicode__(self):
        s = 'Put your data here.' 
        assert isinstance(s, unicode)
        return s
```


