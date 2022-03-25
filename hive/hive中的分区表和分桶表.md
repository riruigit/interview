## 分区表的概念

分区表实际上就是对应一个 HDFS 文件系统上的独立的文件夹，该文件夹下是该分区所 有的数据文件。Hive 中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据 集。在查询时通过 WHERE 子句中的表达式选择查询所需要的指定的分区，这样的查询效率 会提高很多。

where字段有分区字段的时候，就不会走全表扫描。

## 分区的增删查改

### 创建分区

增加分区可以增加一个，也支持增加多个。

```sql
 alter table dept_partition add partition(day='20200405') 
partition(day='20200406');
```

### 删除分区

同上，可以删除多个。

```sql
alter table dept_partition drop partition 
(day='20200404'), partition(day='20200405');
```

## 二级分区

二级分区，其实就是多个 partition 条件。

