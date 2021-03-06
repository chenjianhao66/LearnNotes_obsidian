2021-09-13
20:03:36
author:陈建浩


--- 

## 什么是NoSQL
在了解什么是Redis之前，先了解下什么是 `NoSQL`

**NoSQL**(`Not Only SQL` )，意即**不仅仅是SQL**, 泛指非关系型的数据库。Nosql这个技术门类,早期就有人提出,发展至2009年趋势越发高涨。

随着互联网网站的兴起，传统的关系数据库在应付动态网站，特别是超大规模和高并发的纯动态网站已经显得力不从心，暴露了很多难以克服的问题。如`商城网站中对商品数据频繁查询`、`对热搜商品的排行统计`、`订单超时问题`、以及微信朋友圈（音频，视频）存储等相关使用传统的关系型数据库实现就显得非常复杂，虽然能实现相应功能但是在性能上却不是那么乐观。nosql这个技术门类的出现，更好的解决了这些问题，它告诉了世界不仅仅是sql。

### NoSQL的发展演化

#### 单机Mysql时期
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109132010801.png)

所有站点的数据都存放到一个数据里面

遇到的瓶颈：
-   数据库总大小一台机器硬盘内存放不下
-   数据的索引（B + tree）一个机器的运行内存放不下
-   访问量（读写混合）一个实例不能承受

#### Memcached（缓存）+ MySql + 垂直拆分
通过缓存来缓解数据库的压力，优化数据库的结构和索引

垂直拆分指的是：分成多个数据库存储数据（如：卖家库与买家库）

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109132011392.png)


#### MySql主从复制读写分离
-   主从复制：主库来一条数据，从库立刻插入一条。
-   读写分离：读取（从库Master），写（主库Slave）

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109132012345.png)


#### 分表分库+水平拆分+MySql集群

主库的写压力出现瓶颈（行锁InnoDB取代表锁MyISAM）
分库：根据业务相关紧耦合在同一个库，对不同的数据读写进行分库（如注册信息等不常改动的冷库与购物信息等热门库分开）
分表：切割表数据（例如90W条数据，id 1-30W的放在A库，30W-60W的放在B库，60W-90W的放在C库）
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109132013742.png)


而上述瓶颈都是Mysql下面所会遇到的瓶颈，不单单上面的瓶颈还会产生以下扩展瓶颈“

1.  大数据下IO压力大
2.  表结构更改困难

--- 


在实际开发中，高并发环境下，不同的用户会需要相同的数据。因为每次请求，在后台我们都会创建一个线程来处理，这样造成，同样的数据从数据库中查询了N次。而数据库的查询本身是ＩＯ操作，效率低，频率高也不好。
总而言之，一个网站总归是有大量的数据是用户共享的，但是如果每个用户都去数据库查询效率就太低了。


Redis是一款基于键值对的NoSQL数据库，它的值支持多种数据结构：
字符串(strings)、哈希(hashes)、列表(lists)、集合(sets)、有序集合(sorted sets)等。
Redis将所有的数据都存放在内存中，所以它的读写性能十分惊人，用作数据库，缓存和消息代理。

Redis具有内置的复制，Lua脚本，LRU逐出，事务和不同级别的磁盘持久性，并通过Redis Sentinel和Redis Cluster自动分区提供了高可用性。

Redis典型的应用场景包括：缓存、排行榜、计数器、社交网络、消息队列等


## Redis的特点
-   Redis是一个高性能key/value内存型数据库
    
-   Redis支持丰富的数据类型
    
-   Redis支持持久化
    
-   Redis单线程,单进程


## Redis的应用场景
 - 网站高并发的主页数据  
 - 网站数据的排名  
 - 消息订阅
 - 缓存
 - 分布式锁
 - 全局ID
 - 用户消息事件线timeline
