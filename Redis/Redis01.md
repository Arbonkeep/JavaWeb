# Redis

## Nosql概述

    1. 概念
        * NoSQL用于超大规模数据的存储。现在大多与关系数据库管理系统（RDBMS）结合使用
        * NoSQL，指的是非关系型的数据库。NoSQL有时也称作Not Only SQL的缩写

    2. 为什么使用Nosql
        
        * 用户的个人信息，社交网络，地理位置，用户生成的数据和用户操作日志已经成倍的增加。Nosql能很好处理这些数据

    3. 特点

        * 方便扩展，数据之间没有关系，很好扩展
        * 大数据量高性能（Redis 一秒写 8 万次，读取 11 万，NoSQL的缓存记录级，是一种细粒度的缓存，性能比较高）
        * 数据类型是多样型的！（不需要事先设计数据库！随取随用！）

    4. 大数据时代要求
        1. 海量Volume                   1. 高并发
        2. 多样Variety                  2. 高可扩
        3. 实时Velocity                 3. 高性能

    5. 四大数据类型
        * KV键值对
        * 文档型数据库（bson格式 和json一样）
        * 列存储数据库
        * 图形数据库

<img src = "./img/img01.png" width = 800px/>

    6. NoSQL的优点/缺点
        * 优点:
            - 高可扩展性
            - 分布式计算
            - 低成本
            - 架构的灵活性，半结构化数据
            - 没有复杂的关系
        * 缺点:
            - 没有标准化
            - 有限的查询功能（到目前为止）
            - 最终一致是不直观的程序


## Redis概述
    1. 概念
        * Redis（Remote Dictionary Server )，即远程字典服务

    2. 作用
        * 内存存储、持久化数据
        * 效率高，可以用于高速缓存
        * 发布订阅系统
        * 地图信息分析

    3. 特性
        * 多样的数据类型
        * 持久化
        * 集群
        * 事务
    
    4. redis总共包含16个数据库，默认在0，默认端口 6379 单线程操作 对3字节的数据读写能达到 读11万次/秒 8万次/秒

    5. 为什么redis是单线程的，速度还这么快？
        * 因为redis是直接操作内存的，并且单线程相对于多线程来说是没有线程上下文的切换的，所以速度快。
    

## 安装Redis
    1. windows系统
        <1> 下载：https://github.com/dmajkic/redis/releases
        <2> 解压到目录
        <3> 配置文件redis.windows.conf
        <4> 打开redis-server.exe 
        <5> 打开redis-cli.exe，执行redis-cli.exe -h 127.0.0.1 -p 6379

    2. linux系统
        <1> 下载地址：http://redis.io/download，下载最新稳定版本    
        <2> 执行下载命令：wget http://download.redis.io/releases/redis-4.0.13.tar.gz
        <3> 解压： tar -zxvf redis-4.0.13.tar.gz
        <4> 执行： cd redis-4.0.13 
        <5> 编译： make (可以在后面加上PREFIX=路径来指定安装路径)
        <6> 复制一份redis.conf，备份使用：cp redis.conf myredis.conf
        <7> 以配置文件redis.conf启动服务： src/redis-server ./redis.conf
        <8> 设置为后台启动：vi redis.conf  将daemonize设置为yes
        <9> 使配置文件生效：src/redis-server redis.conf
        <10> 客户端连接redis: src/redis-cli
        <11> 查看redis进程是否开启： ps -ef | redis
        <12> 关闭服务 shutdown
        <13> redis-benchmark测试性能，100 个并发连接 100000 请求：
             redis-benchmark -h localhost -p 6379 -c 100 -n 100000


## 基本命令

```
        redis-cli -h 127.0.0.1 -p 6379 -a "mypass"          程连接 h表示主机 p表示端口 a表示密码
        ping                                                检查是否连接，成功返回pong
        select 3                                            切换至3号数据库
        dbsize                                              查看当前数据库的 数据量  一般为 k-v 的对数
        keys *                                              查看当前库中的所有 key
        flushdb                                             清空当前数据库
        flushall                                            清空所有数据库
        dbsize                                              查看数据库数据量

```
## 五大数据类型



### String类型
    1. string类型的操作

```     

    set name zhangsan                              设置值为zhangsan k-v数据
    get name                                       返回这个key对应的值 
    exits name                                     判断当前key是否存在  存在返回1否则返回0
    move name 1                                    移除key
    expire name 10                                 设置这个k-v的过期时间为10秒
    ttl name                                       查看这个k-v剩余的有效期时间
    type name                                      查看当前key的类型
    append name "123"                              将123拼接到name对应的value值后面 如果key不存在则等价于set
    strlen name                                    查看这个key对应的value值的长度
    incr num                                       自增1 下次get num将返回1
    decr num                                       自减1 下次get num将又返回0
    incrby num 5                                   设置自增的步长 自增5 
    decrby num 5                                   设置自减的步长 自减5 
    getrange name 1 2                              范围获取 返回name对应value的一部分 从下标1开始到下标2结束  
    getrange name 0 -1                             范围获取 返回name对应value的一部分 从下标0开始到末尾  
    setrange name 1 aa                             将name下标1开始的内容替换为 aa  
    setex name1 10 "aaa"                           如果name1不存在则创建k-v并指定过期时间10秒，具有原子性。
                                                   如果存在则覆盖value指定过期时间10秒
    setnx name2 bbb                                如果不存在这个key则创建成功返回1，如果存在 则创建失败返回0
    mset k1 v1 k2 v2 k3 v3                         批量设置这3个k-v对
    mget k1 k2 k3                                  批量返回相应k值的相应value值
    msetnx k1 v1 k4 v4                             批量不存在时设置，具有原子性，如此时k1存在k4不存在，全部失败返回0
    set user:1 {name:zhangsan,age:10}              保存对象
    mset user:1:name zhangsan user:1:age 10        也可有上述效果
    getset name ccc                                先get再set 不存在时 返回nil，但set仍然会生效！等价与创建了一个新
                                                   的k-v。存在时返回之前的value值且覆盖掉这个value，下次get会返回刚
                                                   设置的新值
    getset key value                               先获取再设置，若是不存在值，返回nil

```      
    2. String类似的使用场景：value除了是我们的字符串还可以是我们的数字
        * 计数器
        * 统计多单位的数量
        * 粉丝数
        * 对象缓存存储


### List类型
    1. List类型操作

```
        Lpush list xxx                  将一个值或者多个值，插入到列表头部 （左）
        Lrange list 0 -1                获取 list 中所有值（获取指定区间的值）
        Rpush list xxx                  将一个值或者多个值，插入到列表尾部 （右）
        Lpop list                       移除 list 的第一个元素（左）
        Rpop list                       移除 list 的最后一个元素
        lindex list 1                   通过下标获得 list 中的某一个值
        llen list                       查看 list 中元素个数
        lrem list 1 three               移除 list 集合中指定个数的value（移除list中1个three）
        ltrim list 1 2                  通过下标截取指定的长度，list已经被改变了，只剩下截取的元素，即索引12对应的元素
        rpoplpush list other            移除列表的最后一个元素，将他移动到新的列表中，即将list第一个元素移动到other中
        lset list 0 five                列表中指定下标的值替换为另外一个值，如果存在，更新当前下标的值，如果不存在，则
                                        会报错,将list中索引为0的元素替换为five
        linsert list before hello hao   将某个具体的value插入到列把你中某个元素的前面或者后面，即将hao插入到list
                                        中hello的前面，若果将before改为after，就是插入到后面

```
    2. 一般list类型会用于消息队列


### Set类型

    1. Set类型操作

```
        sadd myset hello                向myset集合中添加一个元素
        semebers myset                  获取myset集合中所有元素
        sismember myset hello           判断myset集合中是否存在hello元素，存在返回1，不存在返回0
        scard myset                     查看myset集合中的元素个数
        srem myset hello                移出myset集合中的hello元素
        srandmember myset 2             随机抽取出myset中2个元素
        spop myset 1                    随机删除指定个数个元素
        smove myset myset2 hello        将一个指定的值，移动到另外一个set集合，将myset中的hello移动到myset2中
        sdiff myset myset2              取两个集合中没有重复的元素（取差集）
        sinter myset myset2             取两个集合中重复的元素（取交集）
        sunion myset myset2             取并集

```

    2. Set中的值是不能重复读的

    3. 使用场景
        <1> sinter可以用于共同好友

        <2> 用于共同爱好等等

### Hash类型

    1. Hash类型操作

```
        hset myhash field zhangsan          设置一个具体的key value值
        hget myhash field                   获取myhash中field字段的值
        hmset myhash field one field2 two   设置myhash中多个字段的值（批量设置）
        hmget myhash field field2           获取myhash中多个字段的值（批量获取）
        hgetall myhash                      获取myhash中所有的字段和值
        hdel myhash field                   删除myhash中的字段及该字段对应的值
        hlen myhash                         获取myhash中的字段的数量
        hexists myhash field                判断myhash是否存在filed字段
        hkeys myhash                        获取myhash中所有的field，不包括值
        hvals myhash                        获取myhash中所有field对应的值，只获取值
        hincrby myhash field 2              使得myhash中field的值自增长2
        hsetnx myhash filed zhangsan        如果存在就失败，如果不存在就创建fileld并设置值为zhangsan


```

    2. Hash的使用场景
        <1> Hash一般用于存储经常变动的数据，如user的age等，hash适用于存储对象，string适用于存储字符串


### zSet类型

    1. zSet类型操作

```
    zadd myset 1 value                  在myset中增加一个value值
    zadd myset 2 value1 3 value3        在myset中增加多个值
    zrange myset 0 -1                   获取有序集合myset中的所有元素
    zrem myset xiaoming                 删除有序集合中某个指定的元素   
    zcard myset                         获取有序集合中元素个数 
    zcount myset 1 2                    获取有序集合中指定区间的元素个数   

    zrangebyscore key min max [withscores] [limit offset count]
        返回有序集key中，所有score值介于min和max之间(包括等于min或max)的成员。有序集成员按score值递增(从小到大)
        次序排列。 

    zrevrangebyscore key max min [withscores] [limit offset count]
         同上，改为从大到小排列。 

```
    2. 有序集合的实现

```
    127.0.0.1:6379> zadd grade 100 xiaoming
    (integer) 1
    127.0.0.1:6379> zadd grade 95 xiaohong
    (integer) 1
    127.0.0.1:6379> zadd grade 85 xiaohuan
    (integer) 1
    127.0.0.1:6379> zadd grade 94 xiaotian
    (integer) 1
    127.0.0.1:6379> zrange grade 0 -1                   # 默认从小到大显示
    1) "xiaohuan"
    2) "xiaotian"
    3) "xiaohong"
    4) "xiaoming"
    127.0.0.1:6379> zrangebyscore grade -inf +inf       # 从小到大排序显示所有用户
    1) "xiaohuan"
    2) "xiaotian"
    3) "xiaohong"
    4) "xiaoming"
    127.0.0.1:6379> zrevrange grade 0 -1                # 从大到小排序显示所有用户
    1) "xiaoming"
    2) "xiaohong"
    3) "xiaotian"
    4) "xiaohuan"
    127.0.0.1:6379> zrangebyscore grade -inf +inf withscores    # 从小到大显示所有用户及分数
    1) "xiaohuan"
    2) "85"
    3) "xiaotian"
    4) "94"
    5) "xiaohong"
    6) "95"
    7) "xiaoming"
    8) "100"
    127.0.0.1:6379> zrangebyscore grade -inf 95 withscores      # 从小到大显示成绩小于等于95的用户及分数
    1) "xiaohuan"
    2) "85"
    3) "xiaotian"
    4) "94"
    5) "xiaohong"
    6) "95"

```

    2. zSet是在set的基础上增加一个值

    3. zSet一般用于排序相关的内容，如排行榜，成绩排序等

