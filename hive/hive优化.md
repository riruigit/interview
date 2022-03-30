# hive 企业级调优

Hive的优化主要分为：配置优化、SQL语句优化、任务优化等方案。其中在开发过程中主要涉及到的可能是SQL优化这块。

优化的核心思想是：

- 减少数据量（例如分区、列剪裁）
- 避免数据倾斜（例如加参数、Key打散）
- 避免全表扫描（例如on添加加上分区等）
- 减少job数（例如相同的on条件的join放在一起作为一个任务）
- 采用 spark 之类的查询引擎

## 查看执行计划

```sql
explain select * from study
```

## fetch抓取

Fetch 抓取是指，Hive 中对某些情况的查询可以不必使用 MapReduce 计算。

在 hive-default.xml.template 文件中 hive.fetch.task.conversion 默认是 more，老版本 hive 默认是 minimal，该属性修改为 more 以后，在全局查找、字段查找、limit 查找等都不走 mapreduce。

## 配置优化

这一部分主要是开启 hive 的配置，从而达到优化的效果。例如 开启CBO优化，开启在 map 端的优化，调整在 map 端聚合操作的条数，在有数据倾斜的时候开始负载均衡（这个配置默认是 false 的），打开谓词下推，设置自动选择 map join 。设置任务并行执行。开启严格模式。

## 表的优化

### 小表和大表 join

将 key 相对分散，并且数据量小的表放在 join 的左边，可以使用 map join 让小的维度表 先进内存。在 map 端完成 join。 

实际测试发现：新版的 hive 已经对小表 JOIN 大表和大表 JOIN 小表进行了优化。小表放 在左边和右边已经没有区别。

### 空 KEY 过滤

如果是 inner join 的话，会过滤空值。但是其他的 join 可就不会了。所以的过滤空值。不然容易造成**数据倾斜.**

### count(distinct id)

由于 COUNT DISTINCT 操作需要用一个 Reduce Task 来完成，这一个 Reduce 需要处理的数据量太大，就会导致整个 Job 很难完成。

优化办法：采用 group by 去重，然后在 count() 起来

```sql
select count(id) from (select id from bigtable group by id) a;
```

### 行列过滤

列处理：在 select 中，只拿需要的列，不要使用 select \* ，存在分区就使用分区限制条件。

行处理：在使用外关联的时候，如果将副表的过滤条件写在 where 条件之后，那么就会先全表关联，之后在过滤。

#### 优化方案

要根据场景去判断，可以先用子查询减少数据量，然后在做关联的操作。

### 谓词下推

简而言之，就是在不影响结果的情况下，尽量将过滤条件提前执行。谓词下推后，过滤条件在map端执行，减少了map端的输出，降低了数据在集群上传输的量，节约了集群的资源，也提升了任务的性能。

[Hive中的Predicate Pushdown Rules（谓词下推规则)_strongyoung88的博客-CSDN博客_hive谓词下推](https://blog.csdn.net/strongyoung88/article/details/81156271)

谓词下推也会有失效的效果。可以看下表。建议先子查询，减少数据量在关联表。

![image-20220331013855513](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/hive/%E8%B0%93%E8%AF%8D%E4%B8%8B%E6%8E%A8.png)

### 笛卡尔积

join操作不加on条件的时候，或者on的条件无效，hive只能使用一个 reduce 去完成任务，设置严格模式，不允许出现笛卡尔积。

