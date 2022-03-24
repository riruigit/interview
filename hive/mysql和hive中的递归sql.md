# 简单记录一下如何用 SQL 去做递归操作。

MySQL8.0 可以直接支持递归的操作，Hive中并不支持这个语法，但是可以通过开窗函数去解决。

## 何为SQL的递归操作

好比如新增一列，id 根据某种规则做对应的变化。

## MySQL8.0的实现

```sql
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 5
)
SELECT * FROM cte;
```

## Hive的实现

```sql
select row_number() over() -1 as rn 
from (select explode(split(space(23), ''))) t
```

第一步：space 函数。这一个函数是返回指定数量的空格。但是不能直使用 explode 。所以这里就需要 spilt 去操作了。因为spilt 返回的就是数组。

explode 的操作就是把一个数组的每一个元素生成一列数据。

