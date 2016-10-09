title: 使用Hexo作为博客
date: 2015-09-24 15:29:25
categories:
  - 其他
tags:
  - 其他
  - 开始
---

## 安装和部署请参考下面或自行google

1.  [官网](https://hexo.io/)
2.  <http://wsgzao.github.io/post/hexo-guide/>

<!--more-->
## 基本使用

### 准备

    # 安装node 和 cnpm
    brew install node
    npm install cnpm -g

    # 将项目clone 下来
    git clone git@github.com:HopperClouds/hopperclouds.github.io.git

    # 安装hexo 依赖的node库
    cnpm install

    # 遇到问题
    # { [Error: Cannot find module './build/Release/DTraceProviderBindings' ] code: 'MODULE_NOT_FOUND'  }
    # { [Error: Cannot find module './build/default/DTraceProviderBindings' ] code: 'MODULE_NOT_FOUND'  }
    # { [Error: Cannot find module './build/Debug/DTraceProviderBindings' ] code: 'MODULE_NOT_FOUND'  }
    # 使用
    cnpm install --no-optional

### 开始写文章

    hexo new "your title"

    # 在source/_posts/your\ title.md 文件
    # 在里面使用markdown编辑博客

    # 生成文件格式
    title: 使用Hexo作为博客
    date: 2015-09-24 15:29:25
    # 类别
    categories:
      - 其他
    # 标签
    tags:
      - 其他
      - 开始
    ---
    markdown格式正文内容

### 生成文章

    hexo generate

    # 使用--watch 参数检测文件更新
    hexo generate --watch

### 预览

    hexo server

### 发布

    hexo deploy

### 将markdown源码push到source分支

    git push origin master:source

## 总结

静态博客才是写博客的正确姿势

初次使用觉得不像[octopress](http://octopress.org/) 那样完善，至于为什么不用octopress, 是因为我们是使用Python和JS的团队，Node对我们来说更友好一些。

对于使用Emacs的用户还没有org mode支持，可以hack一下了。
