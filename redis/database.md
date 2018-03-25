# 数据库

```c
/* Redis database representation. There are multiple databases identified
 * by integers from 0 (the default database) up to the max configured
 * database. The database number is the 'id' field in the structure. */
typedef struct redisDb {

    // 数据库键空间，保存着数据库中的所有键值对
    dict *dict;                 /* The keyspace for this DB */

    // 键的过期时间，字典的键为键，字典的值为过期事件 UNIX 时间戳
    dict *expires;              /* Timeout of keys with a timeout set */

    // 正处于阻塞状态的键
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP) */

    // 可以解除阻塞的键
    dict *ready_keys;           /* Blocked keys that received a PUSH */

    // 正在被 WATCH 命令监视的键
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */

    struct evictionPoolEntry *eviction_pool;    /* Eviction pool of keys */

    // 数据库号码
    int id;                     /* Database ID */

    // 数据库的键的平均 TTL ，统计信息
    long long avg_ttl;          /* Average TTL, just for stats */

} redisDb;
```

## 键空间dict *dict;    

## 过期字典dict *expires;   

### 设置过期时间
```bash
expire
pexpire

expireat
pexpireat
```
### 移除过期时间
```bash
persist 
```

### 过期键删除策略
* 定时删除：内存友好，cpu不友好
* 惰性删除：内存不友好，cpu友好
* 定期删除：每隔一段时间，删除一次

### aof，rdb和复制对过期键的处理
#### rdb
* 生成rdb文件
    * 已过期的键不保存到新的rdb文件中
* load rdb文件
    * 主服务器模式，忽略过期键（检查的原因）
    * 从服务器模式，载入过期键（不检查的原因）

#### aof
* 当过期键被删除后，会对aof append一条命令，显示该记录已经被删除
* aof重写，忽略过期键（检查的原因）

#### 复制
当服务器运行在复制模式下，从服务器的过期键删除动作由主服务器控制，保证主从服务器的一致性

## 数据库通知