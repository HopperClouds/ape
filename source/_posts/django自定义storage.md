---
title: django自定义storage
categories: django
tag: [django, 技术]
---
最近遇到了这样的一个问题，由于某些原因，需要把静态文件放到cdn上，之前使用的是django默认的storage（FileSystemStorage）。于是这里需要自定义storage。
第一次写storage，过程中遇到一些坑，记录下来。
<!--more-->
## 什么是storage
其实这个玩意要我说明白，好像有点难，于是我就按我的方法说，如果有错，谢谢指正！
首先，我们的web中使用了许多的模版文件，静态文件，如js,css，图片这些，我们在配置服务器的时候，让nginx对请求进行分发，将动态请求分发给uWSGI，将静态文件交由nginx处理，这里nginx将从文件系统中读取静态资源，这里的文件系统就是我们当前的storage。
但是这里我们是想用cdn，于是我们这里的静态资源不在从服务器上加载，而是从我们的cdn服务提供商那里加载，这里我们又有一个问题了，cdn服务提供商怎么给出正确的资源，于是我们就需要自己来写一个storage，将原来的文件系统更换为cdn提供商的空间。
## storage的结构和重写
这里就需要参考django的官方文档了([原版](https://docs.djangoproject.com/en/1.9/howto/custom-file-storage/),[翻译版](http://python.usyiyi.cn/django/howto/custom-file-storage.html))，由于我的外语水平不是很好，做的时候为了图效率，就没有花时间去琢磨英文官方文档，所以这里给一个翻译的文档地址，当然，如果你能很快看明白官方的文档，那去读原版是更好的选择。
首先，我们看文件系统的基类，其源码位于`django/core/files/storage.py`文件下。这里我就不贴它的代码了，如果需要查看结构，请自行到该文件下查看！
从文档中，我们可以了解的一些东西，这个类的子类需要不带参数实例化，于是我们需要在settings中加入自定义的参数。
``` python
    STATICFILES_STORAGE = 'project.storage.CustomStorage'
    CUSTOM_STORAGE_OPTIONS = {
        'AccessKeyId': 'your_accesskeyid',
        'AccessKeySecret': 'your_accesskeyidsecret',
        'endpoint': 'oss-cn-hangzhou.aliyuncs.com',
        'oss_url': 'http://oss-cn-hangzhou.aliyuncs.com',
        'bucketname': 'your_bucketname',
    }

    COMPRESS_STORAGE = 'project.storage.CustomCompressorFileStorage'
    STATIC_URL = 'your_cdn_prefix_address'
```
参数说明：
- 这里我们指定了STATICFILES_STORAGE为我们自定义的CustomStorage,如果不指定，那么系统将会使用默认的FileSystemStorage
- CUSTOM_STORAGE_OPTIONS是一个你自定义的Storage初始化参数，这里我用字典来初始化，当然，你可以使用你觉得合理的任何数据类型。
- COMPRESS_STORAGE是我们指定的压缩文件的存放位置，与STATICFILES_STORAGE同理。
- STATIC_URL这个就是静态文件的路由前缀，例如你的文件系统中路径是'aa/bb.js'，你的cdn地址是'static.cdn.com'，这里就使用cdn地址作为你的STATIC_URL

从文档中，我们可以知道，自定义的storage，必须实现_open,_save两个方法，我们参考源码可以知道，这两个方法分别被save和open两个方法调用，而这两个方法的作用分别是‘打开文件，读取内容’、‘将文件保存到指定的位置’，由此，我们需要自己定义的存储就在这里来实现。
由于这里我采用的是阿里云的oss，所以认证的过程，我们放在构造函数中完成，本着`D.R.Y`的原则，为了让多个自定义的storage可以使用，我们将它放在外部，只在构造函数里来使用它。
``` python
def authticate(option):
    auth = oss2.Auth(
        option.get('AccessKeyId'),
        option.get('AccessKeySecret')
    )
    service = oss2.Service(
        auth,
        option.get('endpoint')
    )
    bucket = oss2.Bucket(
        auth,
        option.get('oss_url'),
        option.get('bucketname')
    )
return (auth, service, bucket)
```
认证过后，就可以重写save过程了。
``` python
def _save(self, name, content):
    self.bucket.put_object(
        name,
        content
    )
    return name
```
这里我们使用的是阿里云的oss，它的save就这么简单。文档可以直接google阿里云oss！这里就不贴出来了！
哈哈，这里是不是很简单！其实理解了它的各个方法，真的很简单…………
继续，由于open方法是打开本地文件系统的文件，我们就不重写它了。
其他的方法。文档中说到，delete()，exists()，listdir()，size()，url() 这几个方法都需要被覆写，不然就会抛出NotImplementedError异常。
这里我们通过源码可以解释一下，这些玩意在干嘛。
- delete方法：顾名思义，就是删除，此方法被调用时，从storage中删除文件
- exists方法：额，还是顾名思义，就是判断是否存在该文件，返回布尔值
- listdir方法：返回文件列表
- size方法：返回文件大小
- url方法：这个方法需要提一下，我在之前重写的时候，直接pass了，所以，我在打开xadmin时，就会一直报错，于是我就找了很久的原因，我在这个函数中下了断点，最后发现，这个函数是必须自己重新写的（如果你的文件是静态文件，可以通过url访问的话）。因为如果不重写它，返回的是一个None，于是该文件就没有url可以访问，在某些需要判断的地方，也会报错！

## 最后
不同的云服务提供商的上传方式可能不一样，但是原理都是一样的，重写save方法，改为上传到云端，重写需要使用的方法。最后collectstatic,compress即可。
