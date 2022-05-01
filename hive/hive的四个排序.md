# hive中的四个排序

## order by

全局排序，只有一个 reducer。默认升序。

这里注意一点，在严格模式中，order by 语句后面必须跟着一个 limit 的字句。

### 如何开启严格模式

开启严格模式是有利的。

- 在对分区表的查询的时候，必须用有分区的条件
- order by 的时候必须带上 limit
- 禁止笛卡尔织的产生，join的时候必须有on的条件

```sql
# 查询严格模式，如果是is undefined 说明严格模式没有开启。默认是没开启的。
set hive.mapred.mode;

# 开启严格模式
set hive.mapred.mode=strict;

# 关闭严格模式
set hive.mapred.mode=undefined;
```

### 位置编号代替列名

什么是位置编号代替列名呢？这一点很鸡肋，而且不建议写这么捞的东西。

```sql
select *
from ads_total_read
where dt = '2022-02-21'
order by 2 desc
```

上面的 2 就是位置编号了，对应第二列的意思。这种写法可读性实在是不行，但是这种学法，但是官网记录了一下一个点。

### 排序中的空值处理

在排序的过程中，对于空值的处理，在升序和降序之间的处理逻辑也不相同。好比如默认的升序排序中，空值会排在最前面，但是在降序的过程中，空值会在排序的最后面。

升序排序  ====》  空值最前面

降序排序  ====》  空值最后面

## sort by

对于大规模的数据集 order by 的效率非常低，在很多情况下，并不需要全局排序。此时可以使用 sort by。

但是 sort by 为每个 reducer 生成一个排序文件，在每个reducer内部进行排序，对全局结果集来说不是排序的。一般场景下不单独使用。

## distribute by

distribute by 是用来控制 map 的输出，在 reduce 是如何划分的。一般情况下会跟 sort by 起来使用。而且sort by 要跟在 distribute by 一起使用。

## cluster by

Cluster By是 Distribute By 和 Sort By 的捷径。

