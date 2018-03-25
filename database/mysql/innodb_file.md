# 表空间文件tablespace
## 共享表空间/系统表空间
Innodb 将存储的数据按照表空间(tablespace)进行存放，默认配置下，会有一个初始大小10M，名为：ibdata1的文件，这就是默认的表空间文件。
 
* 配置文件中的设置：
    * 1.默认设置
      * innodb_data_file_path = ibdata1:10M:autoextend
        
        生成文件默认是在data目录下。
    * 2.多路径设置
      * innodb_data_file_path = /data1/db1/ibdata1:100M:autoextend; /data2/db2/ibdata2:100M:autoextend
      
        放在不同的磁盘，可以平均磁盘负载，提高数据库性能。
    * 3.记录内容
        * 记录所有基于Innodb存储引擎的表的数据。
 
## 独立表空间
### 配置
```sql
mysql> show variables like 'innodb_file_per%'\G;
```
```
*************************** 1. row ***************************
Variable_name: innodb_file_per_table
Value: ON
```
注意：MySQL5.6.7之后默认开启
### 记录内容
**独立的表空间，仅存储该表的：数据，索引和插入缓冲BITMAP等信息。**
其余信息仍存储在默认表空间。
 
* 启用innodb_file_per_table后，每张表的表空间只存放自己的：数据，索引和插入缓冲BITMAP页。其它信息仍放在默认表空间。
 
* 其它信息如：**回滚(undo)信息、插入缓冲索引页、系统的事物信息、二次写缓冲(Double write buffer)等**。
 
* file_per_table的特点
    * 可以灵活选择：row format和file format。对于数据压缩、truncate table操作会更快写。
 
    * 回收空间可以再次被利用，相反默认表空间不会收缩空间。
 
    * MySQL Enterprise Backu对于拥有独立表空间的表，更灵活。表能从备份中单独出来，适合备份不是太频繁的表。
 
* 表空间：从逻辑存储结构看，所有数据都被逻辑地存放在一个空间中，称之为表空间（tablespace）
 
* 共享表空间大小不会自动收缩。
# 重做日志文件redo log file