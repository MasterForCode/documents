# 通用命令

1. KEYS *

   > 查看当前db的所有键,时间复杂度O(n)，该命令不建议在生产环境使用，当key较多的时候会阻塞redis，建议使用替代命令scan、hscan、sscan、zscan.

2. DBSIZE

   > 查看当前db键总数

3. EXISTS $key

   > 查键是否存在，存在返回1、不存在返回0     

4. DEL $key

   > 通用命令返回结果为删除成功的键个数，删除不存在的key返回0，支持一次删除多个键

5. EXPIRE $key $seconds

   > 当为key设置了过期时间，超过过期时间后，键会被删除

6. TTL $key

   > 查看对应的剩余过期时间，返回值有以下3种：
      * -2 键不存在
      * -1 键未设置过期时间
      * 自然数，键的剩余过期时间

7. TYPE $key

   > 查看键的类型,当键不存在时，返回none

8. OBJECT ENCODING $key

   > 查看键对应的内部编码
   
9. SELECT $index

   > 选择数据库

# String相关命令

1. SET $key $value [EX seconds |PX milliseconds] [NX|XX]

   > 设置字符串,NX表示键不存在时，写入该key，XX表示键存在时写入（实际就是更新操作),如果第一次set了一个key并设置了过期时间，那么再次针对该key执行set操作的话，该key之前的过期时间会被清空
      * SET java jedis ex 120 nx
      * SET java hello,jedis xx

2. GET $key

   > 获取对应键的值，如果键不存在返回nil

3. MSET $key $value  [$key $value …]

   > * 该操作是原子性的
      * MSET中的某个键存在于当前DB中的话，会用新的值替换掉旧的值，过期时间也会被清除（如果存在的话）
      * 如果想要在键不存在的情况下，批量写入，请使用MSETNX 命令
      * MSETNX是原子操作，要么都成功要么都失败

4. MGET $key [$key …]

   > * 批量获取值
      * 当给定的键不存在或者键类型不是字符串的话，对应键返回特殊值nil
      * 使用批量操作，可以提高开发效率，节省来回的网络时间，但是要注意，批量操作的命令数不宜过多，过多会阻塞redis

5. INCR $key/INCRBY $key $increment/INCRBYFLOAT $key $increment、DECR $key/DECRBY $key $decrement

   > * 当键对应的值为正数时,执行成功
      * 自增或者增加指定数/自减或者减去指定数
   
6. APPEND $key $value

   > * 在key对应的值上添加字符串value
   > * 如果key不存在，新增

7. STRLEN  $key

   > 获取key对应值的长度

8. GETRANGE $key $start $end

   > * 截取key对应的值，闭区间
   > * end为-1表示截取从start开始的所有字符

9. SETRANGE $key $offset $value

   > * 从offset处开始将等长字符替换成value

10. SETEX $key $seconds $value

    > * 设置并给定过期时间

11. SETNX $key $value

    > * 库中不存在key时创建
    > * 库中存在则创建失败

12. GETSET $key $value

    > * 返回修改前的值，并修改值

# HASH相关命令

1. HSET $key $field $value [$key $value...]

   > 写入值
     
      * 当新增一个field时，返回结果为1，当更新了一个已经存在的field时，返回0

2. HGET $key $field

   > 获取key下的field对应的值
     
      * field存在，返回对应的值，反之返回nil

3. HMSET $key $field $value [$field $value …]

   > 批量设置key下field

4. HMGET $key $field [$field …]

   > 批量获取key下field

5. HDEL $key $field [$field...]

   > 删除key下field

6. HGETALL $key/HKEYS $key/HVALS $key

   > 获取键的所有键值对/键/值

7. HEXISTS $key $field

   > 键不存在或者field不存在时，返回0


# LIST相关命令

> redis的列表是双端链表，可以再两端进行push、pop操作，一个列表最多可以存2^32-1 个元素。

1. LPUSH/RPUSH $key $value [$value...]

   > 向列表头/尾部插入元素

2. LPOP/RPOP $key

   > 从列表头/尾部删除一个元素

3. LLEN $key

   > 获取列表元素个数

4. LRANGE $key $start $end

   > 获取指定范围内的元素
     
      * $start、$end：0表示列表的第一个元素、1表示第2个元素，-1表示列表的最后一个元素，-2表示倒数第二个元素等等
   
5. LTRIM $key $start $end

   > 清空列表
     
      * 严格满足 $start、$end >0 && $start >$end

6. BLPOP/BRPOP $key [$key...]  $timeout

   > 阻塞式的删除列表元素
     
      * 如果超时，则返回nil
   
7. LINDEX $key $index

   > * 通过下标获取值

8. LLEN $key

   > * 获取列表长度

9. LREM $key $count $value

   > * 移除列表中指定数量的value

10. RPOPLPUSH $key $key1

    > * 将key列表中的最后一个元素放到key1列表的头部
    > * key1列表不存在就创建

11. LSET $key1 $index $value

    > * 将列表中指定下标的值替换成新值
    > * 如果key1不存在，报错

12. LINSRT $key before|after $value $value1

    > * 在指定值的前|后插入value1

# SET相关命令

1. SADD $key $member [$member...]

   > 添加元素

2. SCARD $key

   > 获取集合中元素个数

3. SMEMBERS $key

   > 获取集合中的所有元素

4. SISMEMBER $key $member

   > 判断一个元素是否在集合内

5. SPOP $key [$count]

   > 删除集合中的元素
     
      * 随机删除
   
6. SREM $key $value

   > * 删除元素

7. SRANDMEMBER $key $count

   > * 随机获取指定个数的元素

8. SDIFF $key1 $key2 |SINTER $key1 key2 | SUNION $key1 $key2

   > 差集|交集|并集

# SORTEDSET相关命令

   > 有序集合中的每一个元素有一个score作为排序依据，有序集合中的元素不能重复，但是score可以重复

1. ZADD $key [nx|xx] [ch] [incr] $score $member [$score $member...]

   > 添加元素

2. ZCARD $key

   > 有序集合内元素个数

3. ZSCORE $key $member

   > 获取某个成员的分数
   
4. ZRANK/ZREVRANK $key $member

   > 计算某个成员的排名,分数从低到高排名/分数从高到低排名

5. ZRANGE/ZREVRANGE  $key $start $end [withscores]

   > 指定排名范围内的成员,从低到高/从高到低

6. ZREM $key $member [$member...]

   > 删除元素

7. ZREMRANGEBYSCORE $key $min $max

   > 删除指定分数范围内的元素
   

# GEOSPATIAL 相关命令

1. 

# Config 相关命令

1. 慢查询
   * CONFIG GET slowlog-log-slower-than 100 定义当命令处理时间超过多少，该命令会被记录在慢查询列表中
   * CONFIG GETslowlog-max-len 定义慢查询列表的大小

2. 连接空闲超时
   
* CONFIG GET timeout 600
   
3. 最大连接数
   * CONFIG GET maxclients 100

   

   