2021-09-20
15:25:58
author:陈建浩


--- 
# Redis的数据类型
Redis的数据类型有5种，分别是 String，List，Set，Zset，Hash

## String类型
String是最常用的一种数据类型，普通的key/value存储都可以归为此类，value其实不仅是String，也可以是数字，以下是String类型的内存模型以及命令表格

内存模型图：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109201525910.png)

常用操作命令表格：

| 命令                                       | 说明                                       |
| ------------------------------------------ | ------------------------------------------ |
| set                                        | 设置一个key/value                          |
| get                                        | 根据key获得对应的value                     |
| mset                                       | 一次设置多个key value （mset key value key value..）                     |
| mget                                       | 一次获得多个key的value (mget key key ...)                    |
| getset                                     | 获得原始key的值，同时设置新值              |
| strlen                                     | 获得对应key存储value的长度                 |
| append                                     | 为对应key的value追加内容                   |
| getrange 索引0开始                         | 截取value的内容                            |
| setex                                      | 设置一个key存活的有效期（秒）              |
| psetex                                     | 设置一个key存活的有效期（毫秒）            |
| setnx                                      | 存在不做任何操作,不存在添加                |
| msetnx原子操作(只要有一个存在不做任何操作) | 可以同时设置多个key,只有有一个存在都不保存 |
| decr                                       | 进行数值类型的-1操作                       |
| decrby                                     | 根据提供的数据进行减法操作                 |
| Incr                                       | 进行数值类型的+1操作                       |
| incrby                                     | 根据提供的数据进行加法操作                 |
| Incrbyfloat                                | 根据提供的数据加入浮点数                   |



## List类型
list 列表 相当于java中list 集合 特点 元素有序 且 可以重复

下面是List的工作示意图
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109201541189.png)

常用的工作指令

| 命令    | 说明                                 |
| ------- | ------------------------------------ |
| lpush   | 将某个值加入到一个key列表头部        |
| lpushx  | 同lpush,但是必须要保证这个key存在    |
| rpush   | 将某个值加入到一个key列表末尾        |
| rpushx  | 同rpush,但是必须要保证这个key存在    |
| lpop    | 返回和移除列表左边的第一个元素       |
| rpop    | 返回和移除列表右边的第一个元素       |
| lrange  | 获取某一个下标区间内的元素(lrange 起始下标 终点下标) 0 -1 表示返回所有元素           |
| llen    | 获取列表元素个数                     |
| lset    | 设置某一个指定索引的值(索引必须存在) |
| lindex  | 获取某一个指定索引位置的元素         |
| lrem    | 删除重复元素 （lerm 要删除重复元素的个数 要删除的重复值）                        |
| ltrim   | 保留列表中特定区间内的元素           |
| linsert | 在某一个元素之前，之后插入新元素(linsert [要插入的列表名] [before或者after] 要插入位置元素名 要插入的元素值)     |

> linsert 命令示例
> ```bash
> linsert list before test test1
> ```
> 以上代码表示的意思是：在 `list` 中，在 `test` 值的前面插入一个名为 `test1` 的值
> `bofore` 是元素之前插入， `after` 是在元素之后插入

**应用场景**

Redis list的应用场景非常多，也是Redis最重要的数据结构之一。  
我们可以轻松地实现最新消息排行等功能。  
Lists的另一个应用就是消息队列，可以利用Lists的PUSH操作，将任务存在Lists中，然后工作线程再用POP操作将任务取出进行执行。


## Set类型
Set类型 Set集合 元素无序 不可以重复

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202109201605746.png)

Set的常用命令列表

| 命令        | 说明                                               |
| ----------- | -------------------------------------------------- |
| sadd        | 为集合添加元素                                     |
| smembers    | 显示集合中所有元素 无序                            |
| scard       | 返回集合中元素的个数                               |
| spop        | 随机返回一个元素 并将元素在集合中删除              |
| smove       | 从一个集合中向另一个集合移动元素  必须是同一种类型 |
| srem        | 从集合中删除一个元素                               |
| sismember   | 判断一个集合中是否含有这个元素                     |
| srandmember | 随机返回元素                                       |
| sdiff       | 去掉第一个集合中其它集合含有的相同元素             |
| sinter      | 求交集                                             |
| sunion      | 求和集                                             |

