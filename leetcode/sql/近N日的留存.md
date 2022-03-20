# 近 N 日的留存问题

步骤一：对数据做一个基本的去重操作

步骤二：使用开窗函数中的 first_value 取用户的最早的登陆时间。（加多一列），使用 datediff 两个时间做差值。(加多一列）

步骤三：使用 sum(  case when date_time = 1 then 1 else 0 end ) / count(distinct userId) as day_1

当然这里也可以使用 sum (if ( ))  去计算，count（distinct userId）这里算的就是 有多少人，相当于 uv

