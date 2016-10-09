title: Python-Web并发重复数据防守策略
date: 2016-09-02 17:51:45
# 类别
categories:
  - 技术
# 标签
tags:
  - python
  - HTTP
  - 并发
---
作者：jackie

## 1.重复数据提交原因
* 恶意用户脚本攻击
* web页面按钮卡顿重复点击引起

## 2.服务器优化方向
* web服务器层防御，如nginx可以限制单一IP每秒钟的访问次数
* 应用层防御，通过web应用程序进行控制
* 数据层防御

## 3.常规防冲击
* nginx 配置 --访问速率控制
```javascript
server {
        ...
        location /download/ {
            limit_conn addr 100; #单一IP每秒钟最多访问100次
        }
```
在代理层防御主要应对于大规模高并发，例如有恶意用户高速率抓取本网站数据，导致网站服务性能下降时，
就需要进行IP访问速率限制；但是考虑到国内网络环境，基本绝大用户都是共享公共IP进行上网，所以此限制也并不是一定会打开。

* 黑名单机制 --防恶意攻击
    * 网络服务商控制，例如使用阿里云的可以通过阿里云的安全策略配置进行设置黑名单。
    * 服务器 本地防火墙策略
    * web服务器 nginx配置黑名单
    * web应用中通过缓存黑名单进行控制

## 4.异常访问带来的数据重复如何规避
* 数据表多字段进行联合唯一索引，通过数据库的限制进行脏数据的排除。 (推荐)
* 数据库加锁，分悲观锁和乐观锁，具体概念不做讲述，一旦加了锁，也就给开发者自己加了锁，自己琢磨去吧。
* 具体业务进行单一服务化，单实例进行处理，可通过MQ与主业务服务进行交互。（推荐）

## 5.具体的某个应用服务如何进行访问速率限制
* 直接上代码了，通过redis的原子操作机制设定计数器，也可称为限速器。
```python
def limit_api_call(key, limit, timeout):
    """
    API限速器
    :param key:
    :param limit:限制次数
    :param timeout: 单位时间
    :return: True or False
    """
    lua_incr = """
        local current
        current = redis.call("incr",KEYS[1])
        if tonumber(current) == 1 then
            redis.call("expire",KEYS[1],ARGV[1])
        end
        return current
    """
    current = client.eval(lua_incr, 1, key, timeout)
    current = int(current)
    if current > limit:
        return False
    return True
```
Reids官方文档中也提供了其他几种实现方式，但是除了是用lua脚本原子操作进行辅助，其他都只能概率限制，无法准确限速。