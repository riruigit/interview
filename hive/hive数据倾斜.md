# hive数据倾斜

## 什么是倾斜

数据倾斜是 “一个人累死，其他人闲死” 的情况。

### 产生现象

任务长时间维持在99%，无法完成任务。或者任务运行的时间远远超过平均时间。判断的方法可以通过以下的方法去操作。

通过时间去判断，通过counter判断。

### 产生的原因

- 对于join过程来说，如果出项较多的key值为空或异常的记录，或key值分布不均匀，就容易出现数据倾斜。
- 对于group by 过程来说，如果某一个key值有特别的多的记录，其它key值的记录比较少，也容易出项数据倾斜。

### 业务场景

- 空值产生的数据倾斜

对于这种，要么不参与关联，要么赋特殊值。

- 不同数据类型关联产生数据倾斜

这个好像没啥用，毕竟可以转换了。但是都是字符类型的比较的话，是按照一位一位比较的。好比如 3>123

### 实现过程

![image-20220410021716485](https://ssuublog.oss-cn-shenzhen.aliyuncs.com/hive/hive_join_group.png)

### 优化办法

#### 增加reduce个数

配置reduce个数，但是不要配置过多，不然会导致输出小文件过多。

```sql
set mapred.reduce.tasks = 15;
```

#### 开启 map join

map join 原理：使用Map Join将小表装入内存，在map端完成join操作，这样就避免了shuffle，reduce操作。省去了 shuffle 阶段。shuffle过程代价非常昂贵，因为他需要排序以及合并。

**在大表和小表的join操作时**，可以开启 map join。

不过这个配置在新版本中默认开启了。

可以调整小表的大小。

```sql
set hive.auto.convert.join = true;  是否开启自动mapjoin，默认是true

set hive.mapjoin.smalltable.filesize=100000000;   mapjoin的表size大小
```

map join 是把小表分发到每一个map上，然后在把全部表读入读入内存之中。

#### 调整内存

调大内存，防止内存溢出。

#### 严格模式

开启严格模式，防止发生笛卡尔积。

#### 参数调优

跟上面这个方法类似，都是属于参数上的调优。

```sql
set hive.map.aggr=true
set hive.groupby.skewindata=true
```

第一个是在map中会做部分聚集操作，效率更高但需要更多的内存。

第二个是数据倾斜时[负载均衡](https://cloud.tencent.com/product/clb?from=10680)，当选项设定为true，生成的查询计划会有两个MR Job。第一个MR Job 中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。

#### 过滤掉脏数据

如果大 key 是无意义的脏数据，直接过滤掉。

#### 大key单独处理

把大的key单独拿出来单独处理。减少参与关联的数据。

好比如订单场景，某一天在北京上海中做了强力推广，但是其他城市的订单量并没有变化，然后在统计订单情况的时候，做了 group 操作，数据倾斜就发生了。

#### count（distinct）的优化

count distinct 的操作只有一个 reduce，也容易发生数据倾斜的操作。

优化的办法：先进行一个 group 分组，分组了之后在执行 count 的操作。

#### 给key值加随机数

在map阶段时给key加上一个随机数，有了随机数的key就不会被大量的分配到同一节点(小几率)，待到reduce后再把随机数去掉即可。翻译成sql的话就是两次group by，第一次 key 加随机数，第二次使用key。

