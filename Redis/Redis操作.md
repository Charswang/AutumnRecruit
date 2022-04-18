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

## 常用命令

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

## redis可以当作数据库，缓存，消息中间件MQ

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

# setex setnx
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

#### Hash

#### Set

##### Sort Set