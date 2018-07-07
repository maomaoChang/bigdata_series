## Hive SQL日常使用总结  

### 1.coalesce(exp1,exp2,...expn)  
    返回参数中的第一个非空值；如果所有值都为NULL，那么返回NULL
```
--找到A1,A2,A3中第一个不为空的字段,并返回该表达式的值
select coalesce(A1,A2,A3) from student;
--coalesce也可以实现nvl的功能，比如：
select name,score,coalesce(address,'Shanghai') from student;
```
### 2.regexp_replace（originstr,oldsubstr,newsubstr） 
    字符串替换函数，用newsubstr替换originstr中的oldsubstr
```
--日期格式变动,并取字串转int型
select  cast(substring(regexp_replace('2016-06-05 00:00:00.0', '-', ''),1,8) as int);
--注：substring(s,start,length)中start默认从1开始，第三个字段length是截取的字串长度
--   有别于Java中，str.subString(start,end)，截取的是start至end-1的字串
```