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

### coalesce

返回第一个不为null的值，如果都为null，则返回null

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

## 时间相关函数

- current_timestamp

- from_unixtime
- from_utc_timestamp
- to_unix_timestamp
- to_utc_timestamp
- unix_timestamp

### current_timestamp

返回当前时间。格式是时间类型，准确到秒。

```sql
select current_timestamp;
#  2022-03-27 04:10:01.382
```

### from_unixtime

把时间戳类型的时间转变成时间类型的时间，但是与北京时间缺了8小时。

其中它可以指点输出的格式。

```sql
SELECT from_unixtime(1648323079, 'yyyy-MM-dd HH:mm:ss');
#   2022-03-26 19:31:19
```

### from_utc_timestamp

上面这个转出来的时间是慢了8小时的，这一个函数就是为了解决这个问题的。

参数必须要求是 时间类型的时间，所以需要转一下。后者是指定时区。

GMT+8  == PRC 。注意大写

```sql
select from_utc_timestamp(from_unixtime(1648323079),'GMT+8');
#    2022-03-27 03:31:19
```

### to_unix_timestamp

把时间转成时间戳的形式，但是这个时间戳也是有8小时时差的问题的。

```sql
select to_unix_timestamp('2022-03-27 03:31:19');
#   1648351879   ----》   2022-03-27 11:31:19
```

转成时间戳的类型，但是再把这个时间戳转回到时间类型，他就会多8小时了。

### to_utc_timestamp

返回当前正确的北京时间。

```sql
select to_utc_timestamp(current_timestamp,'RPC');
#    2022-03-27 04:11:18.321
```

### unix_timestamp

返回时间戳。

```sql
select unix_timestamp();
#    1648325494
# 提示被抛弃了，使用 current_timestamp 代替
```



## 时间戳

unix_timestamp 和 to_unix_timestamp 都是返回时间戳类型的函数。一个是生成，一个是转成。

## 正常时间

除了上面的这两个，其他的都是了。。不想打了

## 时间函数

### to_date：抽取日期部分

select to_date('2020-10-28 12:12:12');

### year：获取年

select year('2020-10-28 12:12:12');

### month：获取月

select month('2020-10-28 12:12:12');

### day：获取日

select day('2020-10-28 12:12:12');

### hour：获取时

select hour('2020-10-28 12:12:12');

### minute：获取分

select minute('2020-10-28 12:12:12');

### second：获取秒

select second('2020-10-28 12:12:12');

### weekofyear：当前时间是一年中的第几周

select weekofyear('2020-10-28 12:12:12');

### dayofmonth：当前时间是一个月中的第几天

select dayofmonth('2020-10-28 12:12:12');

### months_between： 两个日期间的月份

select months_between('2020-04-01','2020-10-28');

### add_months：日期加减月

select add_months('2020-10-28',-3);

### datediff：两个日期相差的天数

select datediff('2020-11-04','2020-10-28');

### date_add：日期加天数

select date_add('2020-10-28',4);

### date_sub：日期减天数

select date_sub('2020-10-28',-4);

### last_day：日期的当月的最后一天

select last_day('2020-02-30');

### date_format(): 格式化日期

select date_format('2020-10-28 12:12:12','yyyy/MM/dd HH:mm:ss');

