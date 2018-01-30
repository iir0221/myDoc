# Mysql配置参数

## datadir
指定数据库所在路径

```sql
mysql> show variables like 'datadir'\G
```
```
*************************** 1. row ***************************
Variable_name: datadir
        Value: /usr/local/var/mysql/
1 row in set (0.01 sec)
```

## innodb_file_per_table
5.5.6以后默认为ON，该参数为ON，则每个表单独存放在一个.bd文件中
```sql
mysql> show variables like 'innodb_file_per_table'\G
```
```
*************************** 1. row ***************************
Variable_name: innodb_file_per_table
        Value: ON
1 row in set (0.00 sec)
```