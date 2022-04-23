# hive中的四个排序

## order by

全局排序，只有一个 reducer。默认升序。

## sort by

对于大规模的数据集 order by 的效率非常低，在很多情况下，并不需要全局排序。此时可以使用 sort by。

但是 sort by 为每个 reducer 生成一个排序文件，在每个reducer内部进行排序，对全局结果集来说不是排序的。一般场景下不单独使用。

## distribute by

distribute by 是用来控制 map 的输出，在 reduce 是如何划分的。一般情况下会跟 sort by 起来使用。而且sort by 要跟在 distribute by 一起使用。

## cluster by

Cluster By是 Distribute By 和 Sort By 的捷径。

