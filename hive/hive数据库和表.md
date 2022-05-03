## DDL扫盲

- DML（data manipulation language）数据库操作语言：
      它们是SELECT、UPDATE、INSERT、DELETE，就象它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言
- DDL（data definition language）数据库定义语言：
      DDL比DML要多，主要的命令有CREATE、ALTER、DROP等，DDL主要是用在定义或改变表（TABLE）的结构，数据类型，表之间的链接和约束等初始化工作上，他们大多在建立表时使用
- DCL（Data Control Language）数据库控制语言：
      是数据库控制功能。是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。在默认状态下，只有sysadmin,dbcreator,db_owner或db_securityadmin等人员才有权力执行DCL

## 数据库

### 创建数据库

这里要养成一个习惯，加上 if not exists

```sql
create database if not exists xx;
```

### 指定数据库文件路径

在hive中可以指定路径的。

```sql
create database if not exists xx  location 'xxx/xxx';
```

### 列举数据库

查看全部数据库，同时可以后面加上 like 进行过滤的操作。

```sql
show databases;
```

### 查看数据库及进阶版

```sql
desc database xxx;
# 进阶版就是加上 extended 查看更多。
```

### 给数据库加备注

用户可以使用 ALTER DATABASE 命令为某个数据库的 DBPROPERTIES 设置键-值对属性值， 来描述这个数据库的属性信息。

```sql
alter database db_hive set dbproperties('createtime'='20170830');
```

### 删除数据库

```sql
drop database db_hive2;
```

### 强制删除数据库

因为数据库中存在表，所以需要强制删除。

```sql
drop database db_hive cascade;
```

## 表

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name, col_name, ...)
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
[ROW FORMAT row_format]
[STORED AS file_format]
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```

EXTERNAL 关键字可以让用户创建一个外部表，在建表的同时可以指定一个指向实 际数据的路径（LOCATION），在删除表的时候，内部表的元数据和数据会被一起删除，而外 部表只删除元数据，不删除数据。



### 内部表和外部表的相互转换

注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！

```sql
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

### 重命名表

数据库的名字不支持修改，但是表支持重命名。

```sql
alter table xxx rename to new_xxx
```

### 表中修改列名

修改列名的时候记得加上类型。

```sql
alter table student change id student_id string;
```

### 表中增加一列

columns 这一个必不可少。

```sql
alter table student add columns  (id string);
```



## 导入数据

导入数据一共有五种方式。

- 直接 load，这种方式又分为本地和hdfs上的数据。
- 把查询的数据插入到表中，也就是 insert into / overwrite table + select 选择。
- 创建表后直接加载数据，也就是创建表语句 + as select 选择。
- 创建表时通过  Location 指定加载数据路径
-  Import 数据到指定 Hive 表中

虽然方式很多，但是可以总结为三个方式。其中 insert overwrite 既可以导出表的数据，也可以导入数据。

**通过 hive 使用 insert 的语句。**

**通过 hdfs 的 put 的方式。**

**通过 hive 的 load 的命令导入。**

但是这里直演示前面三种操作。

### load数据

```sql
load data [local] inpath '数据的 path' [overwrite] into table 
student [partition (partcol1=val1,…)];
```

其中 local 是本地路径，不是 hdfs 的。overwrite 是表示追加。后面是指定分区。

### 查了数据在插入表

注意指定分区的操作，partition 加括号

```sql
 insert overwrite table student_par partition(month='201707')
 select id, name from student where month='201709';
```

### 创建表后插数据

这里的 as 在创建表的参数里面出现过。

```sql
create table if not exists student3
as select id, name from student;
```

# 导出数据

导出数据的方式其实有两种。一个是使用 hive 的语句实现，一个是 linux 中的重定向  >  。

### 使用 hive 中的 insert 的指令

其中，如果导出到 hdfs 上面，就是不加 local ，如果是本地的话，就加上 local。

```sql
insert overwrite local directory '/opt/module/export/student1'
row format delimited fields terminated by ','
select * from hive.m_student;
```

### 使用linux中的重定向符号

```sql
hive -e 'xxxx' > xxx.txt
```

# 创建表

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name, col_name, ...)
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
[ROW FORMAT row_format]
[STORED AS file_format]
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```

EXTERNAL：如果是外部表，需要加上。内部表（也叫内部表）不加，

COMMENT：表示字段或者表的注释。

PARTITIONED BY：用来创建分区表。

CLUSTERED BY：用来创建分桶表。

第6行：跟分桶表有关。

ROW FORMAT：定义行的格式。

STORED AS：指定文件的格式。

LOCATION：指定文件在hdfs上的路径。

AS：把查询的结果放到表里。

```sql
create table if not exists ads_random_band(
    title string
)
partitioned by (dt string)
row format delimited fields terminated by '\t'
lines terminated by  '\n'
```

常规的建表语句，使用  \t 做列的分隔符，使用 \n 做行的分隔符。 
