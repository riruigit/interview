# hive 企业级调优

Hive的优化主要分为：配置优化、SQL语句优化、任务优化等方案。其中在开发过程中主要涉及到的可能是SQL优化这块。

优化的核心思想是：

- 减少数据量（例如分区、列剪裁）
- 避免数据倾斜（例如加参数、Key打散）
- 避免全表扫描（例如on添加加上分区等）
- 减少job数（例如相同的on条件的join放在一起作为一个任务）

## 查看执行计划

```sql
explain select * from study
```

## fetch抓取

Fetch 抓取是指，Hive 中对某些情况的查询可以不必使用 MapReduce 计算。

在 hive-default.xml.template 文件中 hive.fetch.task.conversion 默认是 more，老版本 hive 默认是 minimal，该属性修改为 more 以后，在全局查找、字段查找、limit 查找等都不走 mapreduce。

## 表的优化

### 小表和大表 join

将 key 相对分散，并且数据量小的表放在 join 的左边，可以使用 map join 让小的维度表 先进内存。在 map 端完成 join。 

实际测试发现：新版的 hive 已经对小表 JOIN 大表和大表 JOIN 小表进行了优化。小表放 在左边和右边已经没有区别。

### 空 KEY 过滤

如果是 inner join 的话，会过滤空值。但是其他的 join 可就不会了。所以的过滤空值。不然容易造成数据倾斜

