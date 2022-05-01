# Hive sql 的执行顺序

执行的顺序如下：

from	where	join	on	select	group by	select	having	distinct	order by	limit	union/union all

## 顺序详解

1. 执行 from ,表都没有找到，怎么查询。
2. 执行 where ，注意一点，在有 join 的操作的时候，hive 会默认进行谓词下推的操作。谓词下推就是先执行 where 的条件，然后减少 join 的数据量。
3. 执行 join 的操作

