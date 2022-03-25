# HIve中的函数

## 函数的分类

hive中函数可以分为三类。这里的  一 ， 多代表的是函数

- UDF  一进一出
- UDAF  多进一出
- UDTF  一进多出

## 查看函数的描述

```sql
desc function extended xxx;
```

## 常用函数

### nvl

NVL( value，default_value)。如果前者是空，则返回后者。

### case when

CASE WHEN THEN ELSE END AS XX。老熟悉了不介绍了

### concat

拼接字符串

### concat_ws

按照某一个规则拼接字符串。concat_ws(‘-’,’a’,’b’)。其中第一个是分隔符。算是一个特殊的 concat 函数，不推荐使用。而且指定了参数了类型。字符串或者数组。

### collect_set

把数据放到一个set里面，返回一个去重之后的数组。

### collect_list

跟上面一样，但是不会去重。

## 行转列



## 列转行

EXPLODE(col)：将 hive 一列中复杂的 Array 或者 Map 结构拆分成多行。

但是要求 数组的这个形式，所以要使用 split 这个函数去分割，这个函数分割完了之后返回的是数组的形式。也就符合了 explode 的参数要求了。

案例：

表 6-7 数据准备 

movie category 字段名字

《疑犯追踪》 悬疑,动作,科幻,剧情 

《Lie to me》 悬疑,警匪,动作,心理,剧情 

《战狼 2》 战争,动作,灾难

生成的效果

```sql
《疑犯追踪》 悬疑
《疑犯追踪》 动作
《疑犯追踪》 科幻
《疑犯追踪》 剧情
《Lie to me》 悬疑
《Lie to me》 警匪
《Lie to me》 动作
《Lie to me》 心理
《Lie to me》 剧情
《战狼 2》 战争
《战狼 2》 动作
《战狼 2》 灾难
```

根据上面的分析，我们可以很快的把 category  字段给转成行，但是不能直接 select 出来，因为左边 movie  只有三行，但是右边的数据有 12 行，行数都不一致了，肯定不行的。

```sql
SELECT
movie,
category_name
FROM
movie_info
lateral VIEW
explode(split(category,",")) movie_info_tmp AS category_name;
```

看需求：是否要跟源表的字段做关联，如果要关联，就需要这个写法了。

## 时间戳



## 正常时间



## 时间函数

