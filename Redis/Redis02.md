# Redis

## Redis的三种特殊数据类型

### Geospatial类型（地理信息）

    1. Geospatial的相关规则
        <1> 两极位置信息无法添加，通常会下载城市数据使用java程序导入

        <2> 有效经度-180°到+180°,有效纬度-85.05112878°到+85.05112878°

        <3> 当坐标位置超出了上述范围，会返回错误信息

    2. Geospatisl的操作

```
        geoadd china:city 113.88 22.25 shengzhen        添加一个地理信息位置
        geopos china:city beijing                       查询一个城市的坐标，china:city中存储的beijing的信息
        geodist china:city beijing shengzhen km         查询城市间的直线距离，如果不指定单位，默认为米
        georadius china:city 110 30 1000 km             查询key中以指定经纬度为中心，半径范围内的所有地理位置
                                                        查询china:city中以经度为110纬度为30的地方方圆1000km的地方
        georadius china:city 110 30 1000 km withdist    查询指定坐标（中心点）到符合范围内城市的直线距离
        georadius china:city 110 30 1000 km withcoord   查询指定坐标（中心点）到符合范围内城市的经纬度（地理位置）
        
        georadius china:city 110 30 1000 km withdist withcoord count 1  综合查询

        georadiusbymember china:city hangzhou 1000 km   查询以指定位置为中心周围的其他元素
        geohash china:city beijing hangzhou             查询出一个或多个元素的geo的hash值将二维的经纬度转换为一维的
                                                        字符串，如果两个字符串越接近，那么则距离越近
                                                        

```
    3. geo底层是通过zset来实现的，可以使用zset的操作方式对geo进行操作

```
        zrange china:city 0 -1                          查询geo中存储的所有元素
        zrem china:city beijing                         删除对应geo中存储的元素

```

    4. 详细操作如下 

```
        127.0.0.1:6379> geoadd china:city 113.88 22.25 shengzhen
        (integer) 1
        127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
        (integer) 1
        127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
        (integer) 1
        127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou
        (integer) 1
        127.0.0.1:6379> keys * 
        1) "myset"
        2) "china:city"
        3) "grade"
        127.0.0.1:6379> geopos china:city beijing
        1) 1) "116.39999896287918091"
        2) "39.90000009167092543"
        127.0.0.1:6379> geopos china:city shanghai
        1) 1) "121.47000163793563843"
        2) "31.22999903975783553"
        127.0.0.1:6379> geodist china:city beijing shengzhen
        "1977525.7252"
        127.0.0.1:6379> geodist china:city beijing shengzhen m
        "1977525.7252"
        127.0.0.1:6379> geodist china:city beijing shengzhen km
        "1977.5257"
        127.0.0.1:6379> geodist china:city shengzhen hangzhou km
        "1086.7847"
        127.0.0.1:6379> georadius china:city 110 30 1000 km
        1) "shengzhen"
        2) "hangzhou"
        127.0.0.1:6379> georadius china:city 110 30 1000 km withdist
        1) 1) "shengzhen"
        2) "944.8686"
        2) 1) "hangzhou"
        2) "977.5143"
        127.0.0.1:6379> georadius china:city 110 30 1000 km withcoord
        1) 1) "shengzhen"
        2) 1) "113.87999922037124634"
            2) "22.2499990553364384"
        2) 1) "hangzhou"
        2) 1) "120.1600000262260437"
            2) "30.2400003229490224"
        127.0.0.1:6379> georadius china:city 110 30 1000 km withdist withcoord count 1
        1) 1) "shengzhen"                   # 查位于指定纬经度地理位置到方圆1000km中的地理位置的直线距离和经纬度
        2) "944.8686"                       # count num 表示查询到从中心点到范围内最近的开始查询
        3) 1) "113.87999922037124634"
            2) "22.2499990553364384"
        127.0.0.1:6379> georadiusbymember china:city shengzhen 1000 km
        1) "shengzhen"
        127.0.0.1:6379> georadiusbymember china:city hangzhou 1000 km
        1) "hangzhou"
        2) "shanghai"
        127.0.0.1:6379> geohash china:city beijing hangzhou
        1) "wx4fbxxfke0"
        2) "wtmkn31bfb0"
        127.0.0.1:6379> zrange china:city 0 -1
        1) "shengzhen"
        2) "hangzhou"
        3) "shanghai"
        4) "beijing"
        127.0.0.1:6379> 

```

### Hyperloglog类型

    1. 简介
        <1> Hyperloglog是基数统计的算法，redis2.8.9开始引入。

        <2> 优点：存储元素占用的内存是固定的，2^64不同元素的存储只需要12kb的内存

        <3> 应用：网页的uv，即一个人多次访问一个网站仍然记录为一个人。传统方式是通过id使用计数思想来完成统计，这种
                  方式需要记录用户id，而使用Hyperloglog虽然有0.81%错误率，但是UV中可以忽略不计

        <4> 如果允许容错那么就可以使用hyperloglog，不允许容错就使用set或者其它

    2. Hyperloglog的操作

```
        pfadd mykey a b c d e f g                       存储一组数据
        pfcount mykey                                   统计mykey中基数的元素个数
        pfmerge mykey3 mykey1 mykey2                    合并两组为一组，将mykey1和mykey2合并到mykey3，取并集

```

### Bitmap类型

    1. 简介
        <1> 位存储，位图数据结构，操作二进制数据来进行记录，只有0和1两个状态

        <2> 应用：一般我们可以用来记录只有两个状态的内容，如：打卡（打卡，未打卡），登录状态（登录，未登录）

        <3> 基本命令
            * setbit key offset value       设置位存储

            * getbit key offset             获取存储的值

            * bitcount key start end        统计某个区间的次数（值为1的次数）

    2. bitmap的操作
        * 案例：使用bitmap来记录一周的打卡次数。0表示未打卡，1表示打卡

```
    127.0.0.1:6379> setbit sign 1 0
    (integer) 0
    127.0.0.1:6379> setbit sign 2 1
    (integer) 0
    127.0.0.1:6379> setbit sign 3 0
    (integer) 0
    127.0.0.1:6379> setbit sign 4 1
    (integer) 0
    127.0.0.1:6379> setbit sign 5 1
    (integer) 0
    127.0.0.1:6379> setbit sign 6 0
    (integer) 0
    127.0.0.1:6379> setbit sign 7 1
    (integer) 0
    127.0.0.1:6379> getbit sign 6           查看星期六的打卡状态
    (integer) 0
    127.0.0.1:6379> bitcount sign 0 -1      查看一周的打卡次数有几天
    (integer) 4
    
```

## Redis的事务

    1. Redis事务的本质：一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行过程中，按照顺序执行。
                       Redis中事务执行是队列结构

    2. Redis的事务中，没有隔离级别的概念。所有的命令在事务中没有被直接执行，只有在发起执行命令exec时，才会
       被执行。redis中单条命令时保持原子性的，但是事务不保持原子性。

    3. 事务的实现

        开启事务：multi
        命令队列：一系列命令
        执行事务：exec
        取消事务：discard

    4. 事务的操作

        <1> 事务的正常执行

```
            127.0.0.1:6379> multi           #开启事务
            OK
            127.0.0.1:6379> set k1 v1
            QUEUED
            127.0.0.1:6379> set k2 v2 
            QUEUED
            127.0.0.1:6379> set k3 v3
            QUEUED
            127.0.0.1:6379> exec            #执行事务
            1) OK
            2) OK
            3) OK
            127.0.0.1:6379> get k1
            "v1"
            127.0.0.1:6379> get k2
            "v2"
            127.0.0.1:6379> get k3
            "v3"

```
        <2> 事务的取消

```
            127.0.0.1:6379> multi
            OK
            127.0.0.1:6379> set k1 v1
            QUEUED
            127.0.0.1:6379> set k2 v2
            QUEUED
            127.0.0.1:6379> set k3 v3
            QUEUED
            127.0.0.1:6379> discard             # 取消事务
            OK
            127.0.0.1:6379> get k1              #取消事务后没有值
            (nil)

```

        <3> Redis中的编译期异常（就是一个命令出错，会导致所有命令都不执行）

```   
            127.0.0.1:6379> multi
            OK
            127.0.0.1:6379> set k1 v1
            QUEUED
            127.0.0.1:6379> set k2 v2
            QUEUED
            127.0.0.1:6379> setaa k3 v3         #错误命令，语法错误
            (error) ERR unknown command `setaa`, with args beginning with: `k3`, `v3`, 
            127.0.0.1:6379> set k4 v4
            QUEUED
            127.0.0.1:6379> exec                # 执行事务失败
            (error) EXECABORT Transaction discarded because of previous errors.
            127.0.0.1:6379> get k1
            (nil)

```

        <4> Redis中运行期异常（如果事务队列中语法都正确，某个命令出现了异常(如：1/0) 在执行命令时，其他的
            命令时可以正常执行的)

```

            127.0.0.1:6379> set k1 "aaa"        #设置k1为字符串aaa
            OK
            127.0.0.1:6379> multi
            OK
            127.0.0.1:6379> incr k1             # 显然字符串不能自增，但是此时语法没有任何问题
            QUEUED
            127.0.0.1:6379> set k2 v2
            QUEUED
            127.0.0.1:6379> set k3 v3
            QUEUED
            127.0.0.1:6379> exec
            1) (error) ERR value is not an integer or out of range      #自增命令出异常，其余命令正常执行
            2) OK
            3) OK
            127.0.0.1:6379> get k2
            "v2"

```

## Redis的乐观锁的实现

    1. 概念
        <1> 乐观锁：是很乐观的，认为什么地方都是安全的，不会上锁，在更新数据时，去判断一下该数据是否被更改过。

        <2> 悲观锁：是很悲观的。认为什么地方都是不安全的，都会上过

        <3> redis中通过watch命令来完成监视，通过unwatch命令来撤销监视

    2. Redis实现乐观锁的测试

        <1> 正常执行

    ```
            127.0.0.1:6379> set zhangsan 100
            OK
            127.0.0.1:6379> set lisi 100
            OK
            127.0.0.1:6379> watch zhangsan             #监视money是否被修改
            OK
            127.0.0.1:6379> multi                   #开启事务
            OK
            127.0.0.1:6379> incrby zhangsan 20
            QUEUED
            127.0.0.1:6379> decrby lisi 20
            QUEUED
            127.0.0.1:6379> exec                    #期间数据没有变动，事务正常结束，也就是说money不会被修改
            1) (integer) 120
            2) (integer) 80

    ```

        <2> 多线程修改值，使用watch命令实现redis的乐观锁

            1) 使用多线程修改值，导致事务执行失败

``` 
        127.0.0.1:6379> set zhangsan 100
        OK
        127.0.0.1:6379> set lisi 100
        OK
        127.0.0.1:6379> watch zhangsan
        OK
        127.0.0.1:6379> multi
        OK
        127.0.0.1:6379> incrby zhangsan 10
        QUEUED
        127.0.0.1:6379> decrby lisi 10
        QUEUED
        127.0.0.1:6379> exec                #在执行事务之前，多开一个客户端修改zhangsan的值（即另外一个线程修改了
        (nil)                                zhangsan的值，会导致事务执行失败）
        
        另外一个线程修改值：
        [root@root local]# redis-cli
        127.0.0.1:6379> set zhangsan 20
        OK
        127.0.0.1:6379> keys * 
        1) "lisi"
        2) "zhangsan"

```
            2) 事务执行失败，即修改失败，我们获取最新的值

```
        127.0.0.1:6379> unwatch                 #事务执行失败，进行解锁
        OK
        127.0.0.1:6379> watch zhangsan          #获取新值，重新监视
        OK
        127.0.0.1:6379> multi
        OK
        127.0.0.1:6379> incrby zhangsan 10
        QUEUED
        127.0.0.1:6379> decrby lisi 10
        QUEUED
        127.0.0.1:6379> exec                    #比对监视的值是否发生改变，如果没有改变则执行成功，如果改变则执行失败
        1) (integer) 30
        2) (integer) 90
        127.0.0.1:6379> 


```

## redis的java客户端Jedis
    1. 使用Jedis连接redis
        <1> 需要导入jedis的jar包

        <2> 需要设置redis的配置文件，redis.conf
            1) 将bind 127.0.0.1 注释掉

            2) 将protect-mode的值置为no

        <3> 关闭防火墙
            CentOS7中使用systemctl stop firewalld.service 关闭，之前的版本使用service iptables stop关闭

## 常用配置文件详解

```
    units                           配置文件中units单位对大小写不敏感

    bind 127.0.0.1                  绑定的ip，只能接收本机访问
    protected-mode yes              保护模式，如果为yes，那么在没有设定bind ip且没有设密码时，Redis只接受本机的响应

    port 6379                       端口设置

    daemonize yes                   以守护进程的方式运行，默认是 no，我们需要自己开启为yes
    pidfile /var/run/redis_6379.pid 如果以后台的方式运行，需要指定一个 pid 文件
    logfile ""                      日志的文件位置名
    databases 16                    数据库的数量，默认是 16 个数据库
    always-show-logo yes            是否总是显示LOGO

    save 900 1                      如果900s内，如果至少有一个1 key进行了修改，就进行持久化操作
    save 300 10                     如果300s内，如果至少10 key进行了修改，就进行持久化操作
    save 60 10000                   如果60s内，如果至少10000 key进行了修改，就进行持久化操作

    stop-writes-on-bgsave-error yes 持久化如果出错，是否还需要继续工作
    rdbcompression yes              是否压缩 rdb 文件，需要消耗一些cpu资源
    rdbchecksum yes                 保存rdb文件的时候，进行错误的检查校验
    dir ./                          rdb 文件保存的目录

    requirepass foobared            设置密码，默认被注释

    maxclients 10000                设置能连接上redis的最大客户端的数量
    maxmemory <bytes>               redis 配置最大的内存容量

    maxmemory-policy noeviction     内存到达上限之后的处理策略
        策略：
        1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）
        2、allkeys-lru ： 删除lru(最近最少使用)算法的key
        3、volatile-random：随机删除即将过期key
        4、allkeys-random：随机删除
        5、volatile-ttl ： 删除即将过期的
        6、noeviction ： 永不过期，返回错误

    appendonly no                   默认是不开启aof模式的，默认是使用rdb方式持久化的，大多数情况使用rdb即可
    appendfilename "appendonly.aof" aof持久化的文件的名字
    appendfsync always              每次修改都会 sync。消耗性能
    appendfsync everysec            每秒执行一次 sync，可能会丢失这1s的数据
    appendfsync no                  不执行 sync，这个时候操作系统自己同步数据，速度最快

