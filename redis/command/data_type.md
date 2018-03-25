### [字符串](#1)
### [散列](#2)
### [列表类型](#3)
### [集合Set](#4)
### [有序集合](#5)
# <span id = "1">字符串</span>
* 例如用户邮箱，json格式，图片等，都可以保存为字符串格式
* 最大为512M
## 命令

### 赋值取值
```bash
set key value
get key
```
### 递增数字
```bash
incr key
```
### 增加指定的整数
```bash
incrby key increment
```
### 减少指定的整数
```bash
decr key
decrby key decrement
```
### 增加指定的浮点数
```bash
incrbyfloat key increment
```
### 向尾部追加值
```bash
append key value
```
### 获取字符串长度
```bash
strlen key
```
### 同时获得/设置多个key
```bash
mget key [key ...]
mset key value [key value ...]
```
### 位操作
```bash
getbit key offset
setbit key offset value
bitcount key [start] [end]
bitop operation destkey key [key...]
bitpos key offset [start] [end]
```
> 这些位操作有什么用呢,如记录用户性别
```
setbit key offset，
其中offset用userid取代，
如果ID为1的用户是男性就setbit key 1 1，
如果是女性就setbit key 1 0,
获取ID为1的性别就getbit key 1
```
## 对象:id:属性 来命名key
```bash
user:1:friends
```
## 生成自增id
使用对象类型（负数形式）:count来存储当前类型对象的数量
```bash
users:count
```
## 存储数据
因为每个字符串类型的key只能存储一个字符串，可以将众多的要存储的元素，序列化（json，二进制等）


# <span id = "2">散列</span>
适合存储对象：使用对象类别和id构成key
```bash
car:1
```
使用field表示属性

使用value表示属性值
## 赋值与取值
```bash
127.0.0.1:6379> hset car:1 price 500
(integer) 1
127.0.0.1:6379> hset car:1 name BMW
(integer) 1
```
```bash
127.0.0.1:6379> hget car:1 price
"500"
```
```bash
127.0.0.1:6379> hmset car:2 price 600 name benz
OK
```
```bash
127.0.0.1:6379> hmget car:2 price name
1) "600"
2) "benz"
```
```bash
127.0.0.1:6379> hgetall car:2
1) "price"
2) "600"
3) "name"
4) "benz"
```
## 判断字段是否存在
```bash
127.0.0.1:6379> hexists car:2 price
(integer) 1
```
## 当字段不存在时赋值
```bash
127.0.0.1:6379> hsetnx car:3 price 100
(integer) 1
127.0.0.1:6379> hmget car:3 price
1) "100"
```

## 增加数字
```bash
127.0.0.1:6379> hincrby car:3 price 100
(integer) 200
```
## 删除字段
```bash
127.0.0.1:6379> hmget car:1 price name
1) "500"
2) "BMW"
127.0.0.1:6379> hdel car:1 price name
(integer) 2
127.0.0.1:6379> hmget car:1 price name
1) (nil)
2) (nil)
```

## 存储数据
相比于字符串类型，多个元素不需要再进行序列化了。

## 只获取字段名或字段值
```bash
127.0.0.1:6379> hkeys car:2
1) "price"
2) "name"
127.0.0.1:6379> hvals car:2
1) "600"
2) "benz"
```
## 只获取字段数量
```bash
127.0.0.1:6379> hlen car:2
(integer) 2
```
# <span id = "3">列表类型</span>
* 列表内部使用双向链表（double linked list）实现的，所以向列表两端添加元素的时间复杂度为O(1)，获取越接近两端的元素速度就越快。

* 代价是通过索引访问元素比较慢。

* 使用场景
    * 如社交网站的新鲜事（关注最新内容）
    * 记录日志

## 向列表两端增加元素
### 向列表左边加入元素
```bash
127.0.0.1:6379> lpush numbers 1
(integer) 1
127.0.0.1:6379> lpush numbers 2 3
(integer) 3
```
### 向列表右边加入元素
```bash
127.0.0.1:6379> rpush numbers 0 -1
(integer) 5
```

## 从列表两端弹出元素
### 从左边弹出元素
```bash
127.0.0.1:6379> lpop numbers
"3"
```
### 从右边弹出元素
```bash
127.0.0.1:6379> rpop numbers
"-1"
```

## 获取列表中元素的个数
```bash
127.0.0.1:6379> llen numbers
(integer) 3
```
时间复杂度为O(1),使用Redis会直接读取现成的值，而不需要像InnoDB那样遍历一遍数据表来统计条目数量
## 获得列表片段
获取从start到stop之间的所有元素(包含两端)，且不会像lpop那样删除元素
```bash
127.0.0.1:6379> lrange numbers 0 2
1) "2"
2) "1"
3) "0"
```
支持负索引，表示从右边开始计算序数，如-1表示最右边第一个元素，-2表示最右边第二个元素
```bash
127.0.0.1:6379> lrange numbers -2 -1
1) "1"
2) "0"
```
显然 lrange numbers 0 -1能获取所有元素
```bash
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "0"
```
如果start的索引比stop的索引位置还靠后，则会返回空列表

如果stop大于实际的索引范围，则会返回到列表最右的元素：
```bash
127.0.0.1:6379> lrange numbers 1 999
1) "1"
2) "0"
```

## 删除列表中指定的值
lrem key count value会删除列表中前count个值为value的元素

返回值是实际删除的元素个数

当count>0时，lrem从列表左边开始删除前count个值为value的元素


当count<0时，lrem从列表右边开始删除前count个值为vaule的元素

当count=0时，lrem删除所有值为value的元素

```bash
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "2"
3) "2"
4) "2"
5) "1"
6) "0"
7) "2"
127.0.0.1:6379> lrem numbers 2 2
(integer) 2
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "2"
3) "1"
4) "0"
5) "2"
127.0.0.1:6379> lrem numbers -1 2
(integer) 1
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "2"
3) "1"
4) "0"
127.0.0.1:6379> lrem numbers 0 2
(integer) 2
127.0.0.1:6379> lrange numbers 0 -1
1) "1"
2) "0"
```
## 获取/设置指定索引的元素值
### 获取
索引从0开始
```bash
127.0.0.1:6379> lrange numbers 0 -1
1) "1"
2) "0"
127.0.0.1:6379> lindex numbers 0
"1"
```
如果索引是负数，则表示从右边开始
```bash
127.0.0.1:6379> lindex numbers -1
"0"
```
### 设置
```bash
127.0.0.1:6379> lset numbers 1 7
OK
127.0.0.1:6379> lindex numbers 1
"7"
```
## 只保留列表指定片段
删除指定索引范围之外的所有元素
```bash
127.0.0.1:6379> lrange numbers 0 -1
 1) "5"
 2) "4"
 3) "3"
 4) "2"
 5) "1"
 6) "0"
 7) "-1"
 8) "-2"
 9) "-3"
10) "-4"
11) "-5"
127.0.0.1:6379> ltrim numbers 3 -2
OK
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "0"
4) "-1"
5) "-2"
6) "-3"
7) "-4"
```

### ltrim lpush一起使用来限制列表中元素的数量

## 向列表中插入元素
linsert key before|after pivot value
```bash
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "0"
4) "-1"
5) "-2"
6) "-3"
7) "-4"
127.0.0.1:6379> linsert numbers before 0 7
(integer) 8
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "7"
4) "0"
5) "-1"
6) "-2"
7) "-3"
8) "-4"
127.0.0.1:6379> linsert numbers after 0 7
(integer) 9
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "7"
4) "0"
5) "7"
6) "-1"
7) "-2"
8) "-3"
9) "-4"
```
## 将元素从一个列表转移到另一个列表
rpoplpush source destination
原子操作，从source的右边弹出一个元素，加入到destination的左边
```bash
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "7"
4) "0"
5) "7"
6) "-1"
7) "-2"
8) "-3"
9) "-4"
127.0.0.1:6379> rpoplpush numbers count
"-4"
127.0.0.1:6379> lrange numbers 0 -1
1) "2"
2) "1"
3) "7"
4) "0"
5) "7"
6) "-1"
7) "-2"
8) "-3"
127.0.0.1:6379> lrange count 0 -1
1) "-4"
```
当把列表作为队列使用时，该命令可以很直观地在多个队列中传递数据（先进先出）
## 阻塞弹出
```
BLPOP key [key ...] timeout
BRPOP key [key ...] timeout
BRPOPLPUSH source destination timeout
```
列表的阻塞式(blocking)弹出原语。

### 任务队列bash
```
loop 
    task = brpop queue, 0
    execute(task)
```
### 优先级队列 
```
loop 
    task = brpop queue:confirmation.email queue:notification.email 0
    execute(task)
```
# <span id = "4">集合Set</span>
* 集合中的元素都是不同的，且没有顺序。
* Redis中使用空的散列表实现
* 常见操作为添加删除或判断某个元素是否存在
* 时间复杂度O(1)
## 增加/删除元素
```bash
127.0.0.1:6379> sadd letters a b c
(integer) 3
127.0.0.1:6379> smembers letters
1) "a"
2) "c"
3) "b"
127.0.0.1:6379> srem letters a b
(integer) 2
127.0.0.1:6379> smembers letters
1) "c"
```
## 获取集合中的所有元素
```bash
127.0.0.1:6379> smembers letters
1) "c"
```
## 判断元素是否在集合中
```bash
127.0.0.1:6379> smembers letters
1) "c"
127.0.0.1:6379> sismember letters c
(integer) 1
127.0.0.1:6379> sismember letters d
(integer) 0
```

## 集合运算
### 差集 A-B
```bash
127.0.0.1:6379> sadd setA 1 2 3
(integer) 3
127.0.0.1:6379> sadd setB 2 3 4
(integer) 3
127.0.0.1:6379> sdiff setA setB
1) "1"
127.0.0.1:6379> sdiff setB setA
1) "4"
```
```bash
127.0.0.1:6379> sadd setC 2 3
(integer) 2
127.0.0.1:6379> sdiff setA setB setC
1) "1"
```
### 交集
```bash
127.0.0.1:6379> smembers setA
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> smembers setB
1) "2"
2) "3"
3) "4"
127.0.0.1:6379> sinter setA setB 
1) "2"
2) "3"
```
### 并集
```bash
127.0.0.1:6379> smembers setA
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> smembers setB
1) "2"
2) "3"
3) "4"
127.0.0.1:6379> sunion setA setB
1) "1"
2) "2"
3) "3"
4) "4"
```
## 获取集合中元素个数
```bash
127.0.0.1:6379> scard letters
(integer) 1
```
## 进行集合运算并将结果存储
```bash
127.0.0.1:6379> sdiffstore setD setA setB
(integer) 1
127.0.0.1:6379> smembers setD
1) "1"
```
## 随机获取集合中的元素
```bash
127.0.0.1:6379> srandmember setA
"1"
127.0.0.1:6379> srandmember setA
"3"
127.0.0.1:6379> srandmember setA
"3"
```
```bash
127.0.0.1:6379> smembers letters
1) "a"
2) "c"
3) "d"
4) "b"
127.0.0.1:6379> srandmember letters 2
1) "a"
2) "b"
127.0.0.1:6379> srandmember letters 2
1) "a"
2) "d"
127.0.0.1:6379> srandmember letters 100
1) "a"
2) "c"
3) "b"
4) "d"
127.0.0.1:6379> srandmember letters -2
1) "b"
2) "b"
127.0.0.1:6379> srandmember letters -2
1) "d"
2) "d"
127.0.0.1:6379> srandmember letters -2
1) "a"
2) "a"
127.0.0.1:6379> srandmember letters -2
1) "c"
2) "d"
127.0.0.1:6379> srandmember letters -10
 1) "c"
 2) "d"
 3) "c"
 4) "c"
 5) "b"
 6) "c"
 7) "a"
 8) "d"
 9) "b"
10) "b"
```
## 从集合中弹出一个元素
```bash
127.0.0.1:6379> smembers letters
1) "a"
2) "c"
3) "d"
4) "b"
127.0.0.1:6379> spop letters
"a"
```
# <span id = "5">有序集合TreeSet</span>
* 使用散列表和跳跃表实现O(logN)
* 分数（comparator）
## 增加元素
```bash
127.0.0.1:6379> zadd scoreboard 89 tom 67 peter 100 david
(integer) 3
```
## 获取元素的分数
```bash
127.0.0.1:6379> zscore scoreboard tom
"89"
```

## 获取排名在某个范围的元素列表Olog((n+m))
* zrange 从小到达
* zrevrange 从大到小
```bash
127.0.0.1:6379> zrange scoreboard 0 2
1) "peter"
2) "tom"
3) "david"
127.0.0.1:6379> zrange scoreboard 1 -1
1) "tom"
2) "david"
127.0.0.1:6379> zrange scoreboard 1 -1 withscores
1) "tom"
2) "89"
3) "david"
4) "100"
```
### 分数相同，按照字典顺序排序
```bash
127.0.0.1:6379> zadd test 0 0 0 9 0 a 0 A 0 z 0 Z
(integer) 6
127.0.0.1:6379> zrange test 0 -1 
1) "0"
2) "9"
3) "A"
4) "Z"
5) "a"
6) "z"
127.0.0.1:6379> zadd chinese 0 张三 0 李四 0 赵五
(integer) 3
127.0.0.1:6379> zrange chinese 0 -1
1) "\xe5\xbc\xa0\xe4\xb8\x89"
2) "\xe6\x9d\x8e\xe5\x9b\x9b"
3) "\xe8\xb5\xb5\xe4\xba\x94"
```
## 获得指定分数范围的元素
```bash
127.0.0.1:6379> zrangebyscore scoreboard 80 100
1) "tom"
2) "david"
```
```bash
127.0.0.1:6379> zrangebyscore scoreboard 80 (100
1) "tom"
```
### limit
```bash
127.0.0.1:6379> zrange scoreboard 0 -1 withscores
 1) "jerry"
 2) "56"
 3) "yvonne"
 4) "67"
 5) "peter"
 6) "76"
 7) "tom"
 8) "89"
 9) "wendy"
10) "92"
11) "david"
12) "100"
127.0.0.1:6379> zrangebyscore scoreboard 60 +inf limit 1 3
1) "peter"
2) "tom"
3) "wendy"
```
类似于
```sql
mysql> SELECT * FROM orange LIMIT 2 OFFSET 3;//查询4-5两条记录
等价于
mysql> SELECT * FROM orange  LIMIT 3,2;
```
```bash
127.0.0.1:6379> zrange scoreboard 0 -1 withscores
 1) "jerry"
 2) "56"
 3) "yvonne"
 4) "67"
 5) "peter"
 6) "76"
 7) "tom"
 8) "89"
 9) "wendy"
10) "92"
11) "david"
12) "100"
127.0.0.1:6379> zrevrangebyscore scoreboard 100 0 limit 0 3
1) "david"
2) "wendy"
3) "tom"
```
## 增加某个元素的分数
```bash
127.0.0.1:6379> zincrby scoreboard -4 jerry
"52"
```
## 获取集合中元素的数量
```bash
127.0.0.1:6379> zcard scoreboard
(integer) 6
```

## 获取指定分数范围内的元素个数
```bash
127.0.0.1:6379> zcount scoreboard 90 100
(integer) 2
```

## 删除一个或多个元素
```bash
127.0.0.1:6379> zrem scoreboard wendy
(integer) 1
```

## 按照排名范围删除元素
```bash
127.0.0.1:6379> zadd testRem 1 a 2 b 3 c 4 d 5 e 6 f
(integer) 6
127.0.0.1:6379> zremrangebyrank testRem 0 2
(integer) 3
127.0.0.1:6379> zrange testRem 0 -1
1) "d"
2) "e"
3) "f"
```

## 按照分数范围删除元素
```bash
127.0.0.1:6379> zremrangebyscore testRem (4 5
(integer) 1
127.0.0.1:6379> zrange testRem 0 -1
1) "d"
2) "f"
```
## 获得元素的排名,从0开始
```bash
127.0.0.1:6379> zrange scoreboard 0 -1
1) "jerry"
2) "yvonne"
3) "peter"
4) "tom"
5) "david"
127.0.0.1:6379> zrank scoreboard peter
(integer) 2
```
## 计算有序集合的交集

```bash
zinterstore destination numkeys key [key...][weights weight [weight...]][aggregate sum|min|max]
```
### aggregate为sum(default)
结果中的分数为每个参与计算的集合中该元素分数的和
```bash
127.0.0.1:6379> zadd sortedSets1 1 a 2 b
(integer) 2
127.0.0.1:6379> zadd sortedSets2 10 a 20 b
(integer) 2
127.0.0.1:6379> zinterstore sortedSetsResult 2 sortedSets1 sortedSets2
(integer) 2
127.0.0.1:6379> zrange sortedSetsResult 0 -1 withscores
1) "a"
2) "11"
3) "b"
4) "22"
```
### aggregate为min
结果中的分数为每个参与计算的集合中该元素分数的最小值
```bash
127.0.0.1:6379> zadd sortedSets1 1 a 2 b
(integer) 2
127.0.0.1:6379> zadd sortedSets2 10 a 20 b
(integer) 2
127.0.0.1:6379> zinterstore sortedSetsResult 2 sortedSets1 sortedSets2 aggregate min
(integer) 2
127.0.0.1:6379> zrange sortedSetsResult 0 -1 withscores
1) "a"
2) "1"
3) "b"
4) "2"
```
### aggregate为max
结果中的分数为每个参与计算的集合中该元素分数的最大值
```bash
27.0.0.1:6379> zadd sortedSets1 1 a 2 b
(integer) 2
127.0.0.1:6379> zadd sortedSets2 10 a 20 b
(integer) 2
127.0.0.1:6379> zinterstore sortedSetsResult 2 sortedSets1 sortedSets2 weights 1 0.1
(integer) 2
127.0.0.1:6379> zrange sortedSetsResult 0 -1 withscores
1) "a"
2) "2"
3) "b"
4) "4"
```
## 计算有序集合的并集
zunionstore 规则类似于计算有序集合的交集