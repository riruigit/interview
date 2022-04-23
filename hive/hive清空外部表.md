# hive清空外部表

删除的注意点：

hive的外部表，删除了，但是hdfs上的文件并不会删除，倘若再次创建表的时候，第二次创建表仍然会指向同一个目录，这样的话，使用load data加载数据就会把之前的文件数据也会加载到表上。

## truncate清空表

truncate 这个指令只适合用在内部表上，在外部表上并不适用。

## 清空外部表的方式

清空表有三种方式，这里只介绍一种。先转成内部表，然后在使用清空表的操作。

```sql
alter table ods_hot_band set TBLPROPERTIES('EXTERNAL'='false');
```

