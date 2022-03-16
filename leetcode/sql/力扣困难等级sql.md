# [1369.获取最近第二次的活动](https://zqt0.gitbook.io/leetcode/sql/1369.get-the-second-most-recent-activity)

这一个题目，我是真的憨，场景是选择第二条记录的，这里用了两个开窗函数，最后一句其实可以优化成  tmp2=1 or tmp=2 。因为只有一条记录，无论你取第一条还是取第二条，都是 1 。

```sql
# Write your MySQL query statement below
with a as (
select * , ROW_NUMBER() over (partition by username order by startDate desc) as tmp , count(username) over (partition by username ) as tmp2 
from useractivity)
select username   , activity     , startDate   , endDate  from a where if (tmp2=1,tmp=1,tmp=2)
```







# [1384.按年度列出销售总额](https://zqt0.gitbook.io/leetcode/sql/1384.total-sales-amount-by-year)

这个题目我的思路是在原有的表上面加上2018，2019，2020这三列，然后根据开始和结束的时间，先把全部情况给列举出来，然后在计算具体一年的值，最后把结果union all 出来。

```sql
with a as (
select product_id ,
case when period_end < '2018-12-31' then (DATEDIFF(period_end,period_start)+1)*average_daily_sales else if(DATEDIFF('2018-12-31',period_start)+1<0,0,(DATEDIFF('2018-12-31',period_start)+1)*average_daily_sales) end as tmp1 ,

case when period_start > '2018-12-31' and  period_end < '2019-12-31' then  if((DATEDIFF(period_end,period_start)+1)<0,0,(DATEDIFF(period_end,period_start)+1))*average_daily_sales 
when  period_start < '2018-12-31' and  period_end < '2019-12-31' then if((DATEDIFF(period_end,'2019-01-01')+1)<0,0,(DATEDIFF(period_end,'2019-01-01')+1))*average_daily_sales 
when period_start > '2019-01-01' and  period_end > '2019-12-31' then if((DATEDIFF('2019-12-31',period_start)+1)<0,0,(DATEDIFF('2019-12-31',period_start)+1))*average_daily_sales 
else 365*average_daily_sales end as tmp2 ,

case when period_end < '2020-12-31' then if(DATEDIFF(period_end,'2020-01-01')+1 < 0 , 0 ,(DATEDIFF(period_end,'2020-01-01')+1)*average_daily_sales) end as tmp3 
from sales)

select b.product_id product_id , product_name, report_year , tmp1 total_amount from (select product_id , '2018' as report_year , tmp1
from a where tmp1 != 0 
union all 
select product_id , '2019' as report_year , tmp2
from a where tmp2 != 0 
union all 
select product_id , '2020' as report_year , tmp3
from a where tmp3 != 0 ) b left JOIN product p on b.product_id = p.product_id 
order by product_id , report_year

```





# [1412.查找成绩处于中游的学生](https://zqt0.gitbook.io/leetcode/sql/1412.find-the-quiet-students-in-all-exams)

这个题目没有难度，一次过了。一个 left join 的操作，把没有参加考试的给过滤掉了。然后在窗口函数排序，选取最大的和最小的，然后对这些id去重，在使用一个 left join 然后就会剩下2这一条没有 join 上，把为空的选上去重就可以了。

```sql
# Write your MySQL query statement below
with a as (
select exam_id , e.student_id , score , student_name  , max(score) over (partition by exam_id) max_score ,  min(score) over (partition by exam_id) min_score 
from exam e left join student s on e.student_id = s.student_id
order by exam_id 
)
select distinct a.student_id student_id, student_name 
from a left join (select distinct student_id  from a 
where score = max_score or  score = min_score ) b on a.student_id = b.student_id 
where b.student_id is null 
order by student_id
```



# [1479.周内每天的销售情况](https://zqt0.gitbook.io/leetcode/sql/1479.sales-by-day-of-the-week)

没看评论区的情况 ，这个我想把数据统计好，然后在做一个行专列。这种结果表要增加列的，可以往case when 这里考虑。行专列的关键在于 max 跟 case when

```sql
# Write your MySQL query statement below
with a as (
select sum(quantity) quantity, item_category  , if (DAYOFWEEK(order_date)-1 % 7 =0,7,DAYOFWEEK(order_date)-1 % 7) as tmp 
from orders o left join items i on o.item_id = i.item_id
group by item_category , tmp 
)
select i.item_category  Category, 
max(case when tmp = 1 then quantity else 0 end) as Monday,
max(case when tmp = 2 then quantity else 0 end) as Tuesday   ,
max(case when tmp = 3 then quantity else 0 end) as Wednesday ,
max(case when tmp = 4 then quantity else 0 end) as Thursday  ,
max(case when tmp = 5 then quantity else 0 end) as Friday    ,
max(case when tmp = 6 then quantity else 0 end) as Saturday  ,
max(case when tmp = 7 then quantity else 0 end) as Sunday    
from items i left join a on i.item_category = a.item_category 
group by Category
order by Category
```



# [569.员工薪水中位数](https://zqt0.gitbook.io/leetcode/sql/569.median-employee-salary)

中位数的判断：在中间的那一个数，如果是偶数就取中间的两个， 如果是奇数，取中间的数。中位数必定产生在这个范围里面(tmp/2 , tmp/2 + 1, tmp/2 + 0.5 )。所以两个开窗，一个排序做标记，一个算总数。

```sql
# Write your MySQL query statement below
with a as (
	SELECT
		*,
		ROW_NUMBER() over ( PARTITION by company ORDER BY salary ) as tmp1 ,
				COUNT(id) over ( PARTITION by company) as tmp 

	FROM
	employee 
	)
SELECT id , company , salary
from a
where tmp1 in (tmp/2 , tmp/2 + 1, tmp/2 + 0.5 )
```



# [571.给定数字的频率查询中位数](https://zqt0.gitbook.io/leetcode/sql/571.find-median-given-frequency-of-numbers)

这个题目跟上面的有点像，也是算中位数，但是这个给出的是数字出现的频率，就不能打标记，取出中间的那条。还有一点就是取平均的话，用sql中的avg函数。

评论区：中位数的意思是在中间，那就是至少大于等于前面一半的数，倒着排也是。

重点在掌握思路 中位数的累计频数必须大于或等于总个数的一半 中位数前一个数（比中位数小）的累计频数必须小于或等于总个数的一半

```sql
# Write your MySQL query statement below
with a as (SELECT  * , sum(frequency) over (ORDER BY num ) as tmp1 , sum(frequency) over (ORDER BY num desc) as tmp2 , sum(frequency) over () as tmp3
FROM numbers ) 
select round(avg(num),1) as median from a 
where tmp1 >= tmp3/2
and tmp2 >= tmp3/2

```



# [579.查询员工的累计薪水](https://zqt0.gitbook.io/leetcode/sql/579.find-cumulative-salary-of-an-employee)

请你编写 SQL 语句，对于每个员工，查询他除最近一个月（即最大月）之外，剩下每个月的近三个月的累计薪水（不足三个月也要计算）

这个是题目出去最大的一个月，说明要开窗了，然后每个月近三月，是逻辑上的近三月，不是表的数据里面的近三行数据。逻辑上的就需要在开窗的时候使用 range了。同时这个 range 还是有注意点的！！！

```sql
with a as (
	SELECT
		id,
		month,
		sum( salary ) over ( partition by id order by month range 2 preceding ) as Salary,
		ROW_NUMBER() over ( partition by id order by month desc ) as month_rank 
	FROM
		`employee` 
	) select
	Id,
	Month,
	Salary 
from
	a 
where
	month_rank > 1
```



# [601.体育馆的人流量](https://zqt0.gitbook.io/leetcode/sql/601.human-traffic-of-stadium)

每天只有一行的记录，而且日期随着id自增。选择大于100的出来做开窗，如果是连续的，tmp会是一样的，在开窗一次，看分区内的count总数是否大于3。大于三就符合要求了。

```sql
# Write your MySQL query statement below
with a as (
SELECT * ,id - ROW_NUMBER() over (order by id ) as tmp 
FROM `stadium`
WHERE people >= 100
) , b as (
SELECT * , count(id) over (partition by tmp) as tmp1
from a )
SELECT id , visit_date ,people from b 
where tmp1 >= 3
order by visit_date  
```



# [618.学生地理信息报告](https://zqt0.gitbook.io/leetcode/sql/618.students-report-by-geography)

sql中的经典行转列问题，max 加 if  可以，当然 case when 也是可以的

```sql
# Write your MySQL query statement below
select max(if(continent='america',name,null)) america,
max(if(continent='asia',name,null)) asia,
max(if(continent='europe',name,null)) europe
from (
select *,row_number() over(partition by continent order by name) rk from student) t1
group by rk
```



# [1097.游戏玩法分析 V](https://zqt0.gitbook.io/leetcode/sql/1097.game-play-analysis-v)

先对用户做一个distinct 的操作，然后 left join 原始表。条件就是 id 匹配，再加上一个时间相差，如果差是1的话，就是第一天留存了，然后除以总数，就可以得出一个次日留存率

```sql
# Write your MySQL query statement below
select
	first_date as install_dt,
	count(*) installs,
	round(count(activity.event_date) / count(*), 2) as day1_retention
from (
	select 
		player_id,
		min(event_date) as first_date
	from activity group by player_id
) t1 left join activity
on t1.player_id = activity.player_id 
and datediff(activity.event_date, t1.first_date) = 1
group by first_date;

```



# [1127.用户购买平台](https://zqt0.gitbook.io/leetcode/sql/1127.user-purchase-platform)

看结果表，肯定是要原有的基础上增加的。t2就是我们构造的表，最终的结果需要在这个表的基础上去  left join 。而且先 group by  spend_date, user_id 一个用户要是有两条记录的话，说明这个用户是 both 。if(count(*)=1,platform,'both')  就是这一条语句的作用。因为要 group by 这里注意select 的不能太多。

```sql
-- # Write your MySQL query statement below
select t2.spend_date, t2.platform, ifnull(sum(amount), 0) as total_amount,
ifnull(count(distinct user_id), 0) as total_users
from
(   #1.构造所需的表
    select distinct spend_date, "desktop" as platform
    from Spending
    union
    select distinct spend_date, "mobile" as platform
    from Spending
    union
    select distinct spend_date, "both" as platform
    from Spending    
) as t2
left join 
(   #2.查询每个用户，每个日期，每个平台类型，总金额
    select spend_date, user_id, sum(amount) as amount,
    if(count(*)=1,platform,'both') as platform
    from Spending 
    group by spend_date, user_id
) as t1
#3.左连接，并按日期和平台分组
on t2.spend_date = t1.spend_date and t2.platform = t1.platform
group by t2.spend_date, t2.platform
```



