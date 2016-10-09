title: 聘宝招聘Python实习生
date: 2015-09-29 10:51:04
categories:
  - 招聘
tags:
  - 招聘
  - Python
---

# 公司情况

现在主要做的产品是 <http://www.pinbot.me/> ,已经拿到A轮

坐标：四川成都高新区府城大道399天府新谷9号楼二单元1505成都浩泊云动科技有限公司

<!--more-->
# 我们的研发团队喜欢什么

1.  K.I.S.S
2.  敏捷开发，快速原型和必要的单元测试
3.  使用\*nix
4.  有创造性思维，喜欢创造的人
5.  函数式编程和各种高并发的编程语言（Scheme、Clojure、Golang、Elixir）

# 为什么招实习生

成都Python的圈子本来就小，看来看去就那么些人，大牛实在难搞定，所以希望找到一些有兴趣往Python方向发展的人，一起成长。

# 在这里你能做什么

1.  了解到Python Web开发常用的工具和流程规范
2.  做一些真正有人使用的产品
3.  一起建设团队，给团队带来更高效的流程和工具
4.  做自己想做的产品，如果你有好的创意都可以跟我们产品经理沟通，将创意实现到产品中

# 我们能提供什么

1.  Mac和双显是我们日常的开发工具
2.  每周免费的零食
3.  技术分享
4.  每月一次的Hack Day
5.  妹子都是女神级别的，养眼提高工作效率

# 工作描述

1.  负责<http://www.pinbot.me/> 网站新功能的开发和日常维护
2.  负责内部CRM管理系统的开发维护

# 技能要求

    # coding: utf-8

    """
    对以下技术熟悉或者有强烈兴趣
    """

    # 基本技能
    BASIC_SKILL = [
        '*nix',
        'Vim or Emacs',
        'Git',
        'Unix哲学',
        '基础算法和数据结构',
        '计算机组成原理',
        '网路协议',
    ]

    # 后端技能
    BACKEND_SKILL = [
        'Python',
        'Django',
        'Python常用库',
        '了解Http 协议',
        'NodeJS',
    ]

    # 前端技能
    FRONTEND_SKILL = [
        'HTML',
        'CSS',
        'JS',
        'AngularJS',
        'React',
        'JQuery',
    ]

    # 运维
    MAINTAIN_SKILL = [
        'bash',
        'Docker',
        'Fabric',
        'Ansible',
        'SaltStack',
    ]

    # 数据库
    DATABASE = [
        'MySQL',
        'Mongo',
    ]

    ALL_SKILL = set(i.lower() for i in (BASIC_SKILL + BACKEND_SKILL + FRONTEND_SKILL + MAINTAIN_SKILL + DATABASE))

    def i_want_you(your_skill):
        """
        符合以上任意关键词就可以了
        后续可以让我们的算法工程师来做一个测试程序

        >>> i_want_you('Python HTML Docker MySQL')
        I want you

        >>> i_want_you('java')
        hehe
        """
        your_skill = [i.lower() for i in your_skill.split()]
        print 'I want you' if set(your_skill).intersection(ALL_SKILL) else 'hehe'


    if __name__ == '__main__':
        your_skill = raw_input('Input your skill: ')
        i_want_you(your_skill)

# 补充

1.  有github或者bitbucket等开源社区账号优先
2.  有自己博客的优先

# 联系我

简历请投至：dengyu@hopperclouds.com
