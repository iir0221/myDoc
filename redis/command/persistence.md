# 持久化
## RBD
* Redis将在以下情况下对数据进行快照
    * 根据配置规则自动快照
    * 执行命令save,bgsave
    * 执行命令flushall
    * 执行复制(replication)

### 执行命令save,bgsave
#### save
快照过程中会阻塞客户端的请求，生产环境尽量避免使用
#### bgsave
异步的快照，生产环境中可以使用这个命令
### 根据配置规则自动快照
save 60 10000:
60秒内10000个key被更改，则执行bgsave
### 执行命令flushall
会清楚数据库中所有数据，当配置了自动快照规则，则会执行一次快照操作，如果没有配置，则不执行快照
### 执行复制(replication)
当设置了主从模式时，Redis会在复制初始化时进行自动快照
## AOF

默认关闭的

启动参数：appendonly yes

开启后，每执行一条会更改Redis数据的命令，Redis就会将该命令写入aof文件

同步硬盘数据

appendfsync always
appendfsync everysec
appendfsync no