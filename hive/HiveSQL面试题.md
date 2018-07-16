## Hive SQL常见面试题  

#### 1.Multi-group by 的使用例子  
```
--按照不同的group by 条件将数据插入到不同的表中
from A
insert overwrite table B
 select A.a, count(distinct A.b) group by A.a
insert overwrite table C
  select A.c, count(distinct A.b) group by A.c
```  

#### 2.hive中 Sort By，Order By，Cluster By，Distrbute By的含义  
```
1. order by：会对输入做全局排序，因此只有一个reducer（多个reducer无法保证全局有序）。只
   有一个reducer，会导致当输入规模较大时，需要较长的计算时间。
2. sort by：不是全局排序，其在数据进入reducer前完成排序。
3. distribute by：按照指定的字段对数据进行划分输出到不同的reduce中。
4. cluster by：除了具有 distribute by 的功能外还兼具 sort by 的功能。
```  

#### 3.用hive实现如下语句：  
```
--SQL 查询出在a表不在b表的数据
    SELECT a.key,a.value  FROM a  WHERE a.key not in (SELECT b.key FROM b)
--hive
    select a.key,a.value
    from a 
    left join b
        on a.key = b.key
    where b.key is null;
```  

#### 4.Hive的特点是什么，它与RDBMS有什么异同  
```
答：1.Hive是基于Hadoop的数据仓库工具，可以将结构化的数据文件映射为一张数据库表
      并提供完整的sql查询功能。
    2.它的优点是学习成本低，可以通过类SQL语句快速实现简单的mapreduce统计，而不需要
      开发专门的mapreduce应用，十分适合数据仓库的统计分析
    3.与RDBMS异同：
```

#### 5.Hive保存元数据的方式有哪些，各有什么特点  
```
Hive主要有三种保存元数据的方式，分别为：
1.内嵌的Derby模式(单用户模式)
  Derby是Hive自带的一个关系型数据库，只支持单实例
2.Local模式(多用户模式)
  一般配合mysql服务器使用，支持多实例。需要本地运行一个mysql服务器，并将支持mysql的jar包手动放入$HIVE_HOME/lib中
3.Remote模式
  在服务器端启动一个MetaStoreServer,客户端利用Thriftx协议通过MetaStoreServer访问元数据库。
```
