## Hive SQL日常使用总结  

#### 1.coalesce(exp1,exp2,...expn)  
    返回参数中的第一个非空值；如果所有值都为NULL，那么返回NULL
```
--找到A1,A2,A3中第一个不为空的字段,并返回该表达式的值
select coalesce(A1,A2,A3) from student;
--coalesce也可以实现nvl的功能，比如：
select name,score,coalesce(address,'Shanghai') from student;
```
#### 2.regexp_replace（originstr,oldsubstr,newsubstr） 
    字符串替换函数，用newsubstr替换originstr中的oldsubstr
```
--日期格式变动,并取字串转int型
select  cast(substring(regexp_replace('2016-06-05 00:00:00.0', '-', ''),1,8) as int);
--注：substring(s,start,length)中start默认从1开始，第三个字段length是截取的字串长度
--   有别于Java中，str.subString(start,end)，截取的是start至end-1的字串
```  

#### 3.collect_list  
    列出字段所有的值，且不做去重操作，如：  
```
select collect_list(id) from students;
```  

#### 4.split(str,regex)
    字符串分割函数，返回一个数组。主要用法如下：
```sql
--(1).常规用法
split("a,b,c,d",',') ==> ["a","b","c","d"]，是一个字符串数组
--(2).截取字符串中的某个值
split("a,b,c,d",',')[0] ==> "a" 
--获取小时数
split(split("2014/7/4 14:50",' ')[1],':')[0] ==> "14"
--(3).特殊分割符号，需要做特殊处理
split('192.168.0.1','.') ==> [] 空字符串
split('192.168.0.1','\\.') ==> ["192","168","0","1"]
```