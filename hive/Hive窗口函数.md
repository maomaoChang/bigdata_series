## Hive 常用窗口函数的用法  

参考连接：
1.https://blog.csdn.net/qq_26937525/article/details/54925827
2.https://blog.csdn.net/scgaliguodong123_/article/details/60135385
3.https://blog.csdn.net/guohecang/article/details/51582242
主要介绍的窗口函数有以下几个：
1. 聚合函数 + Over
2. partition by 字句
3. order by 字句
4. window 字句
5. ntile
6. row_number,rank,dense_rank
7. lag,lead函数
8. first_value和last_value
9. cube,rollup,grouping sets,grouping_id
10.cume_dist,percent_rank

#### 基础数据准备  
  准备一张order表,字段分别为name,orderdate,cost.数据内容如下:
```
jack,2015-01-01,10
tony,2015-01-02,15
jack,2015-02-03,23
tony,2015-01-04,29
jack,2015-01-05,46
jack,2015-04-06,42
tony,2015-01-07,50
jack,2015-01-08,55
mart,2015-04-08,62
mart,2015-04-09,68
neil,2015-05-10,12
mart,2015-04-11,75
neil,2015-06-12,80
mart,2015-04-13,94
```  
hive中建表t_window,并将数据插入进去  
```sql
use hive_test_database;
create table t_window
( 
   name string,
   orderdate string,
   cost int
) row format delimited
fields terminated by '';
--导入到hive
load data local inpath 
```

### 1.聚合函数+over  
假如说我们想要查询在2015年4月份购买过的顾客及总人数,我们便可以使用窗口函数去去实现：
```
select name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'
```
得到的结果为：
```
name    count_window_0
mart    5
mart    5
mart    5
mart    5
jack    5
```
2015年4月共发生了5次购买记录，mart购买了4次，jack1次。大多数时候我们只想看去重后的结果，此时可以有两种实现方式：
方式1：distinct
```
select distinct name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'
```
方式2：group by
```
select name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'
group by name
```
执行后的结果都是一样的：
```
name count_window_0 
mart 2 
jack 2
``` 

### 1.Partition By子句  
partition by子句称为查询分区字句，over之前的部分在一个分组内进行，如果超出了分组，则函数会重新计算。  
比如，我要看顾客的购买明细及月购买总额：  
```
select name,
       orderdate,
       cost,
       sum(cost) over (partition by month(orderdate)) 
from t_window
```  
得到的结果为：  
```
name    orderdate   cost    sum_window_0
jack    2015-01-01  10  205
jack    2015-01-08  55  205
tony    2015-01-07  50  205
jack    2015-01-05  46  205
tony    2015-01-04  29  205
tony    2015-01-02  15  205
jack    2015-02-03  23  23
mart    2015-04-13  94  341
jack    2015-04-06  42  341
mart    2015-04-11  75  341
mart    2015-04-09  68  341
mart    2015-04-08  62  341
neil    2015-05-10  12  12
neil    2015-06-12  80  80
```  
可以看出，最后一行数据已经按照月份进行汇总了。