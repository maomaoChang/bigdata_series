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

### 2.Order By子句  
order by子句会让输入的数据强制排序,Order By子句对于诸如Row_Number()，Lead()，LAG()等函数是必须的，因为如果数据无序，这些函数的结果就没有任何意义。使用方式如下：  
```
select name,
       orderdate,
       cost,
       sum(cost) over(partition by month(orderdate) order by orderdate )
from t_window
``` 
得到的结果为:(order by默认情况下聚合从起始行当当前行的数据)
```
name    orderdate   cost    sum_window_0
jack    2015-01-01  10  10
tony    2015-01-02  15  25
tony    2015-01-04  29  54
jack    2015-01-05  46  100
tony    2015-01-07  50  150
jack    2015-01-08  55  205
jack    2015-02-03  23  23
jack    2015-04-06  42  42
mart    2015-04-08  62  104
mart    2015-04-09  68  172
mart    2015-04-11  75  247
mart    2015-04-13  94  341
neil    2015-05-10  12  12
neil    2015-06-12  80  80
```  

### 3. window 字句  
上面已经通过partition by对数据做了分组，但是如果想要更细的粒度，就需要借助于window字句来实现:  
明确两个概念：  
1.只使用partition by而不制定order by,聚合为分组内聚合
2.使用order by,未使用window字句的情况下，默认从起点到当前行  
**当同一个select查询中存在多个窗口函数时,他们相互之间是没有影响的.每个窗口函数应用自己的规则.**  
window字句包含：
1. PRECEDING 往前
2. FOLLOWING 往后
3. CURRENT ROW 当前行
4. UNBOUNDED 起点  

+ UNBOUNDED PRECEDING 从前面的起点
+ UNBOUNDED FOLLOWING 到后面的终点  

我们按照name进行分区,按照购物时间进行排序,做cost的累加：  
``` sql
select name,orderdate,cost,
sum(cost) over() as sample1,--所有行相加
sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加
sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row )  as sample4 ,--和sample3一样,由起点到当前行的聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   and current row) as sample5, --当前行和前面一行做聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   AND 1 FOLLOWING  ) as sample6,--当前行和前边一行及后面一行
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行
from t_window;
```
得到的查询结果为：  
```
name    orderdate   cost    sample1 sample2 sample3 sample4 sample5 sample6 sample7
jack    2015-01-01  10  661 176 10  10  10  56  176
jack    2015-01-05  46  661 176 56  56  56  111 166
jack    2015-01-08  55  661 176 111 111 101 124 120
jack    2015-02-03  23  661 176 134 134 78  120 65
jack    2015-04-06  42  661 176 176 176 65  65  42
mart    2015-04-08  62  661 299 62  62  62  130 299
mart    2015-04-09  68  661 299 130 130 130 205 237
mart    2015-04-11  75  661 299 205 205 143 237 169
mart    2015-04-13  94  661 299 299 299 169 169 94
neil    2015-05-10  12  661 92  12  12  12  92  92
neil    2015-06-12  80  661 92  92  92  92  92  80
tony    2015-01-02  15  661 94  15  15  15  44  94
tony    2015-01-04  29  661 94  44  44  44  94  79
tony    2015-01-07  50  661 94  94  94  79  79  50
```  

### 4.窗口函数中的序列函数
