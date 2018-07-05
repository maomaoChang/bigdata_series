## Hive SQL常见面试题  

#### 1.Multi-group by 的使用例子  
```
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