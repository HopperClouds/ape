---
title: Python打包发布一条龙
date: 2016-11-29 22:00:00
tags:
  - python
  - setup.py
  - pypi
categories:
  - 技术
author: Kxrr
---


今天介绍一下 Python 项目的打包及分发。

<!--more-->

## 相关工具


### distutils

[distutils](https://docs.python.org/2/distutils/
) 是于2000年发布的 Python 项目创建打包工具。

但现在很少直接使用, 而是使用 setuptools 。

### setuptools

[setuptools](http://setuptools.readthedocs.io/en/latest/) 于2004年发布, 是 distutils 的增强版本, 包含 `setup` 和 `find_packages` 等常见函数。

另外它包含了 easy_install 这个包安装工具, 使用Eggs包来进行安装(扩展名为 .egg ), 如果你不幸用 Windows 做过 Python 开发, 应该对它不陌生。

另外现在在 macOS 的 Python 环境中也附带了这个工具。

常见用法如下:

```
$ easy_install redisdict-0.0.1-py2.7.egg
```

Eggs文件实际是一个压缩包, 解压后目录结构大致如下, 具体可以参看 [The Internal Structure of Python Eggs¶](http://setuptools.readthedocs.io/en/latest/formats.html) 

```
├── EGG-INFO
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   ├── requires.txt
│   ├── top_level.txt
│   └── zip-safe
├── redisdict
│   ├── __init__.py
```

说到 Eggs 就得谈谈 Wheel, [Wheel](https://www.python.org/dev/peps/pep-0427/) 是2012年发布的Python包格式, 类似于 Eggs , 其目录结构大致如下:

```
├── redisdict
│   └── __init__.py
└── redisdict-0.0.1.dist-info
    ├── DESCRIPTION.rst
    ├── METADATA
    ├── RECORD
    ├── WHEEL
    ├── metadata.json
    └── top_level.txt
```

### pip

[pip](https://pip.pypa.io/en/stable/) 是 PyPA(Python Packaging Authority) 推荐、时下最主流的包安装工具, 发布于2013年, pip 支持从常见的VCS系统(例如git)进行安装, 也可以安装Eggs包, Wheel包。

## 使用 setuptools 打包

我们以 RedisDict 这个模块为例, 首先在项目中新建 `setup.py` 文件, 并引入 `setuptools` 。
之后使用 `setuptools.setup` 方法对项目进行打包, 方法的参数设定可以参看 [setup-args](https://packaging.python.org/distributing/#setup-args) 。

示例代码如下:

```
import setuptools
import codecs


def get_requirements(filename):
    return codecs.open('requirements/' + filename, encoding='utf-8').read().splitlines()


setuptools.setup(
    name='redisdict',
    version='0.0.1',
    packages=setuptools.find_packages(exclude=['tests', 'tests.*']),
    url='https://github.com/kxrr/redisdict',
    license='MIT',
    author='kxrr',
    author_email='hi@kxrr.us',
    description='A dict-like object using Redis as the backend.',
    install_requires=get_requirements('default.txt'),
    tests_require=get_requirements('test.txt'),
)
```

之后, 我们可以使用下面命令将项目打包为压缩文件:

```
$ python setup.py check  # 检查
$ python setup.py sdist  # 打包为 .tar.gz 
$ python setup.py bdist_egg  #  创建 Eggs包
$ python setup.py bdist_wheel  # 创建 Wheel包
```


生成的文件均位于 dist 目录下:

```
dist
├── redisdict-0.0.1-py2-none-any.whl
├── redisdict-0.0.1-py2.7.egg
└── redisdict-0.0.1.tar.gz
```

别外也可以通过 setuptools 将项目打包成其它格式(比如exe), 有兴趣的可以自己看看。

## 分发

### 发布到自建 PyPI Server

只需要通过 `scp` 命令将打好的包传到我们的 PyPI Server 上去:

```
$ scp dist/redisdict-0.0.1-py2.py3-none-any.whl deploy@pypi.pinbot.me:/home/deploy/packages
```

上传完成后可以使用 pip 进行安装:

```
$ pip install redisdict -i http://pypi.pinbot.me/simple/ --trusted-host pypi.pinbot.me
```

### 发布到官方 PyPI Server

首先到 [PyPI](https://pypi.python.org/pypi?%3Aaction=register_form) 注册一个帐号, 在邮箱内确认。

之后在家目录新建一个 `.pypirc` 文件, 写入下面内容(注意填入自己的帐号密码):

```
[pypirc]
servers = pypi
[server-login]
username:username
password:password
```

接下来就可以开始上传了:

```
$ python setup.py register  # 将包注册到 PyPI
$ python setup.py register sdist upload  # 上传
```

运行过后根据提示操作即可发布完成。

---

Packaging User Guide: <https://packaging.python.org/key_projects/>
Celery setup.py: <https://github.com/celery/celery/blob/master/setup.py>
What was the problem with packaging?: <http://pythonhosted.org/distlib/overview.html#what-was-the-problem-with-packaging>
Uploading your project to pypi: <https://packaging.python.org/distributing/#uploading-your-project-to-pypi>


