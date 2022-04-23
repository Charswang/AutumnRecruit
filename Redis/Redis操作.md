> 当前例子在windows下进行
>
> redis官网：https://redis.io

#### 进入redis

```bash
1、进入redis安装目录
2、执行redis.cli出现下面所示
D:\Tools\Redis>redis-cli
127.0.0.1:6379>

redis的端口号：6379
为什么是6379 -- 作者偶像名字在九宫格上的数字
<mysql--3306端口是作者女儿名字在九宫格上对应的数字>

redis使用c写的

redis-cli -p 6379
```

## 常用命令--不区分大小写

设置失败返回0，设置成功返回1

```bash
# 下面的name都是keyName的实例
# 查看命令的帮助
help command

# 切换数据库--redis默认包括16个数据库
select 15

# 查看当前所在数据库大小
DBSIZE

# 设置key-value
set name wh

# 查看指定key的value
get name

# 查看所有的key
keys *

# 清空redis中的所有数据
flushall

# 清空redis中当前数据库的数据
flushdb

# 判断key是否存在
exists name

# 删除指定key，1代表当前的数据库
move name 1

# ttl，设置过期时间
expire name 10  # 设置name的过期时间为10s
# 查看key可以存活的剩余时间
ttl name

# 查看key的数据类型
type name
```

#### <font color="red">为什么redis是单线程的？</font>

>  redis的瓶颈不是CPU，而是根据机器的内存和网络，所以既然可以使用单线程那么也没必要使用多线程了【多线程在cpu上可以体现出优势？】

#### <font color="red">redis使用单线程为什么还快?</font>  --  面试常问

需要先了解的两个误区：

**误区1：**高性能的服务器一定是多线程的

**误区2：**多线程一定会比单线程快 -- 因为多线程会涉及CPU的上下文切换，cpu的上下文切换是耗时的

总体来说在处理速度上来说CPU>内存>硬盘

**<font color="red">核心：</font>**redis是将数据放到内存的，所以使用单线程去操作效率是最高的，多次读写仅仅使用一个CPU。因为使用多线程的话会涉及CPU的上下文切换：耗时的操作。因此对于内存系统来说，没有上下文切换，效率就是最高的。

#### redis可以当作<font color="blue">数据库，缓存，消息中间件MQ</font>

**redis的默认16个数据库：**就跟在mysql中创建16个不同的数据库一样的-一个mysql实例中创建多个互不影响的数据库。

## 5大数据类型

#### String【value除了可以是字符串还可以是数字--做计数器等】

```bash
# 想name对应的value后面追加字符串
append name "hello" # 如果当前的key不存在，那么就相当于set key value

# 获取指定key字符串的长度
strlen name

# i++
incr key

# i--
decr key

# i+=10
incrby key 10

# i-=10
decrby key 5

# 字符串范围range
# 截取指定key的指定序号范围的字符串【包括start和end】
getrange key 0 3
getrange key 0 -1 # 查看指定key的全部字符串

# 替换--replace
# 从指定位置开始替换成指定的字符串【注意！！】
setrange key 4 ss
例子：key原来是abcdef  执行setrange key 1 xx之后，key的value就变成了axxdef

# setex setnx --  分布式锁
setex (set with expire) -- 设置过期时间,设置一个key的value，并指定过期时间
例子：setex key1 10 "hello"

setnx (set if not exist) -- 如果不存在，则xxx  在分布式锁中会常常使用！！！
例子：setnx key2 "world"

# 批量设置/获取[这里设置三个key]
mset k1 v1 k2 v2 k3 v3 # 批量设置
mget k1 k2 k3 # 批量获取

在上面的基础上在使用
msetnx k1 v2 k4 v4 # 会返回0，表示失败。因为redis是原子性的，要成功都成功要失败都失败。但是这里失败的是哪一个？？？

# 对象
set user:1 {name:zhangsan,age:3} # 设置一个对象user:1[表示user的id]
# 对象的使用
mset user:1:name lisi user:1:age 18
mget user:1:name usr:1:age

# 组合命令
getset[先get，再set]
## 如果不存在值，则返回nil，如果存在值，先获取返回原来的值，并设置新的值
getset db redis [返回的是get到的值，即使db不存在，也会将db设置成redis的，因为先get到null，然后set db redis]

# 这里使用到的像get/set/getset...再jedis中都是相应的方法。
```



#### List

> 所有的list命令都是以l开头的
>
> 当作阻塞队列。消息队列。栈

```bash
# 类似栈，不过可以从顶部和底部都可以插入
# 将一个值或者多个值插入到列表的头部--栈
# 如果listName存在，则新增内容，否则创建新的list并新增内容
lpush listName one
# 获取list所有元素
lrange listName 0 -1
# 获取list前2个元素
lrange listName 0 1

# 放到栈的最底部
rpush listName right

# 删除元素
# 从栈顶删除元素
lpop listName
# 从栈底删除元素
rpop listName

# 删除n个指定的值
# 移除listName中的one元素，并且从上到下开始，只删除一个one，因为list中是可以存在相同元素的one可能存在多个，这里只删除1个，就是从上到下开始删除
lrem listName 1 one

# 获取某个list的第index个值  --  按照栈顶元素为序号0
lindex listName 1

# 返回list的长度
llen listName

# 只保留list的指定范围的元素
# 截取1~2序号的元素，同时改变list
ltrim listName 1 2

# 先rpop移除list1栈底元素，并将移除的栈底元素加到另外一个list2的栈顶
rpoplpush list1 list2

# 将列表中指定下标的值替换为另外一个值--update功能
# 需要先确认这个list是否存在
# 如果不存在list，执行lset会报错，如果存在，则更新当前指定的下标值
lset list 0 value
# 但是如果当前list中不存在下标为2的元素，但是仍然执行lset list 2 value的话会报错
lset list 2 value # 如果list不存在序号为2的元素，则报错。

# 将某个具体的Value插入到list中某个元素的前面或者后面
# 在list中的existValue元素的前面插入newValue元素
# existValue是list中已经存在的元素，newValue是要插入的新的值
linsert list before existValue newValue
linsert list after existValue new Value
```

>  实际就是一个链表，left,right都可以插入值
>
> 如果key不存在，则创建新的链表
>
> 如果key存在，则新增内容
>
> 如果移除了所有值，空链表，也代表不存在
>
> 在两边插入或者改动值，效率最高，中间元素进行插入或者改动，相对来说效率会低一些

#### Hash

> Map集合   key-Map集合
>
> 命令以h开头

```bash
# set一个具体的key-value
# 这里的field1是map中的key，value1是field1对应的value
hset hashName field1 value1

# 获取指定hash中指定的key的value
hget hashName field1

# 设置多个字段
hmset hashName field1 hello field2 world
# 获取多个字段值
hmget hashName field1 field2

# 获取所有数据，按照key-value显示，比如序号1为field1，序号2的元素九尾hello
hgetall hashName
# 只获得所有的key
hkeys hashName
# 只获得所有的value
hvals hashName

# 删除hashName中指定的key字段，key对应的value也就消失了
hdel hashName field1

# 获取hashName的key-value个数
hlen hashName

# 判断hashName中指定的key是否存在
hexists hashName field1

# 对数据增加/减少
hincrby hashName field2 1
hdecrby hashName field2 1

# 如果hashName的某个key(field4)不存在，则设置key(field4)的value为hello
hsetnx hashName field4 hello

# hash很适合用户信息之类的，经常变动的信息。
# hash更适合对象的存储，String更适合字符串存储
hset user:1 name zhangsan
hget user:1 name
```

##### hash很适合用户信息之类的，经常变动的信息。

##### hash更适合对象的存储，String更适合字符串存储

#### Set

> set中的值不能重复，无序且不重复
>
> set的命令都是以s开头

```bash
# 向set中添加一个元素
sadd setName hello

# 查看所有元素
smembers setName

# 判断hello元素是否存在于set中
# s is member
sismember setName hello

# 查看set中的元素个数
scard setName

# 删除set中指定元素
# s rem(ove)
srem setName hello

# 随机选出set中指定个数的元素
# s rand member
srandmember setName 2

# 随机删除元素
spop setName

# 将一个指定的值移动到另外一个set中
# 将set1中的hello元素移动到set2中
smove set1 set2 hello

# 两个set的差集
sdiff set1 set2

# 两个set的交集--共同好友的实现
sinter set1 set2

# 两个set的并集
sunion set1 set2
```



##### Zset(Sort Set)

> 在set的基础上增加了一个值，进行排序

```bash
# 这里的1 2 3就是在set的基础上增加了一个值[官方命名为score-后面的zrangebyscore就展示了score]，借此来进行排序
zadd set1 1 one
zadd set1 2 two 3 three

zrange set1 0 -1

# zrangebyscore key min max
# -inf +inf表示负无穷 正无穷
# 表示从最小到最大进行排序 -- 递增显示，使用zrangebyscore set1 +inf -inf递减显示难道不行？如果不行就用zrevrange set1 0 -1，新版本的+inf -inf貌似没法用
zrangebyscore set1 -inf +inf
# zrevrange set1 start end ==> 将zset反过来，按照反过来的顺序获取start~end的数据
zrevrange set1 0 -1 # 从大到小进行排序，【用数字表示是否为递增/递减】

# 连着score一块显示
zrangebyscore set1 -inf +inf withscores

# 限制展示的最大的score值为25，就是递增显示，但是递增的边界最大为25，大于25的不显示了
zrangebyscore set1 -inf 25 withscores

# 显示所有元素
zrange set1 0 -1

# 删除指定元素
zrem set1 value

# 获取zset中的元素个数
zcard set1

# 获取指定score区间的元素数量
# 获取score=1 ~ score=3之间的元素个数
zcount set1 1 3
```

**案例思路：**

存储班级成绩表，工资表排序

普通消息设置score为1，重要消息设置score为2，按权重进行判断信息

排行榜的实现



### 3中特殊数据类型

#### Geospatial

#### Hyperloglog

#### Bitmap

