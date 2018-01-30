# mysql配置
## 配置文件
```bash
mysql --help |grep my.cnf
```
```
    order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf 
```
Mysql数据库按/etc/my.cnf -> /etc/mysql/my.cnf -> /usr/local/etc/my.cnf -> ~/.my.cnf 的顺序配置文件。**后面的配置会覆盖前面的。**