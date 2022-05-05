# hive中的四个排序

## order by (全局排序)

全局排序，只有一个 reducer。默认升序。此处采用的排序办法是**归并排序**。

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

## sort by （局部排序）

对于大规模的数据集 order by 的效率非常低，在很多情况下，并不需要全局排序。此时可以使用 sort by。

每个 reducer 生成一个排序文件，sort by 只保证每个 reducer 的输出有序，只能保证局部是有序的，对全局结果集来说不是排序的。**一般场景下不单独使用。**

关于分区的规则：

1. 使用 sort by 时，通常会跟 distribute by字段一起连用，通过 distribute by 来指定分区规则。
2. 当不指定分区规则时(只使用sort by )，则使用**默认的内部算法进行分区。**

## distribute by （分区排序）

distribute by 是用来指定分区规则的，如果单独使用 sort by。因为没有分区的规则，所以 sort by 会用内部的随机算法进行分区，但是如果有 distribute by ，会根据这个划分分区。如果还想要排序，那就结合 sort by 即可。

distribute by 是用来控制 map 的输出，在 reduce 是如何划分的。一般情况下会跟 sort by 起来使用。而且sort by 要跟在 distribute by 一起使用。

关于分区的规则：

根据 key 的哈希值取模来划分的。相同的key值会划分到同一个reduce。

## cluster by （簇排序）

Cluster By是 Distribute By 和 Sort By 的结合体。

当两者相同的时候，可以使用，但是有一点就是：不能指定排序的规则，只能做一个升序的。

## 参考文档

[Sort By、Distribute By 使用说明书_扛麻袋的少年的博客-CSDN博客](https://blog.csdn.net/lzb348110175/article/details/117321142)

