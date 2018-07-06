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