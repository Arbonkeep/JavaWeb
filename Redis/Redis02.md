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