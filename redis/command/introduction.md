# 特性
## 存储结构
* 字典式存储结构和对多种键值数据类型的支持，使得数据在Redis中的存储形式和其在程序中的使用方式非常接近。
* 可进行交集并集这样的集合运算操作

## 内存存储与持久化
* Redis数据库中的所有数据存储在内存中，同时提供异步持久化支持

## 功能
* 缓存
    * 为每个键值设置生存时间TTL
    * 可以限定数据占用的最大内存，之后按照一定的规律自动淘汰不需要的键
* 队列
    * 列表类型键
    * 支持阻塞式读取
    * 高性能的优先级队列
    * 发布/订阅的消息模式（聊天室）
## 多数据库实例
* 默认支持16个数据库，每一个以数字命名（0-15）
* 不能更改名字
* 所有数据库拥有同样的权限
* 不要在同一个redis实例上，存储多个应用的数据

## 不区分大小写

# 常用命令
## 获取keys 列表

```bash
127.0.0.1:6379> set bar 1
OK
127.0.0.1:6379> keys *
1) "bar"
```
可使用glob风格通配符
```
*：匹配一个路径部分中0个或多个字符，注意不匹配以.开始的路径，如文件.a。
?：匹配一个字符。
[…]：匹配一系列字符，如[abc]匹配字符a, b, c，在[^…]和[!…]表示匹配不在列表中的字符，如[^abc]匹配除了a, b, c以外的字符。
**：匹配0个或多个子文件夹。
{a,b}：匹配a或则b，a和b也是通配符，可以由其他通配符组成。
!：排除文件，如!a.js表示排除文件a.js。 
```
## 判断key是否存在
```bash
127.0.0.1:6379> exists bar
(integer) 1
127.0.0.1:6379> exists noexists
(integer) 0
```
## 删除key
```bash
127.0.0.1:6379> del bar
(integer) 1
127.0.0.1:6379> del bar
(integer) 0
```

del命令的参数不支持通配符，但可以结合linux的管道和xargs来删除所有符合规则的keys

例：删除所有以user开头的key
```bash
127.0.0.1:6379> set user1 1
OK
127.0.0.1:6379> set user2 2
OK
```
```bash
xinyuazh-mac:~ xinyuan.zhang$ redis-cli keys "user*" |xargs redis-cli del
(integer) 2
```
del命令支持多个keys作为参数，因此，也可以这样删除（推荐）
```bash
xinyuazh-mac:~ xinyuan.zhang$ redis-cli del `redis-cli keys "user*"`
(integer) 2
```
## 获取keys的数据类型
```bash
127.0.0.1:6379> set foo 1
OK
127.0.0.1:6379> type foo
string
```
```bash
127.0.0.1:6379> lpush bar 1
(integer) 1
127.0.0.1:6379> type bar
list
```

































