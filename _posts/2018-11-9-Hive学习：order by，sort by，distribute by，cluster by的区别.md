---
title: Hive学习：order by，sort by，distribute by，cluster by的区别
key: 20181109
tags: Hive 大数据 机器学习 Hive学习笔记 SQL 数据库
---
源数据：

    tb.id	tb.name	tb.job	tb.m_id	tb.day	tb.salary	tb.salary2	tb.deptno
    7369	SMITH	CLERK	7902	1980-12-17	800.0	NULL	20
    7499	ALLEN	SALESMAN	7698	1981-2-20	1600.0	300.0	30
    7521	WARD	SALESMAN	7698	1981-2-22	1250.0	500.0	30
    7566	JONES	MANAGER	7839	1981-4-2	2975.0	NULL	20
    7654	MARTIN	SALESMAN	7698	1981-9-28	1250.0	1400.0	30
    7698	BLAKE	MANAGER	7839	1981-5-1	2850.0	NULL	30
    7782	CLARK	MANAGER	7839	1981-6-9	2450.0	NULL	10
    7788	SCOTT	ANALYST	7566	1987-4-19	3000.0	NULL	20
    7839	KING	PRESIDENT	NULL	1981-11-17	5000.0	NULL	10
    7844	TURNER	SALESMAN	7698	1981-9-8	1500.0	0.0	30
    7876	ADAMS	CLERK	7788	1987-5-23	1100.0	NULL	20
    7900	JAMES	CLERK	7698	1981-12-3	950.0	NULL	30
    7902	FORD	ANALYST	7566	1981-12-3	3000.0	NULL	20
    7934	MILLER	CLERK	7782	1982-1-23	1300.0	NULL	10

1. `order by`

`order by`会对输入做全局排序，因此只有一个Reducer(多个Reducer无法保证全局有序)，然而只有一个Reducer，会导致当输入规模较大时，消耗较长的计算时间。
注意：

(1)：`order by`后面可以有多列进行排序，默认按字典排序。

(2)：`order by`为全局排序。

(3)：`order by`需要reduce操作，且只有一个reduce，无法配置(因为多个reduce无法完成全局排序)。

2.`sort by`

`sort by`不是全局排序，其在数据进入reducer前完成排序，因此，如果用`sort by`进行排序，并且设置`mapred.reduce.tasks > 1`，则`sort by`只会保证每个reducer的输出有序（sorts the data per reducer. ），并不保证全局有序。`sort by`不同于`order by`，它不受`hive.mapred.mode`属性的影响，`sort by`的数据只能保证在同一个reduce中的数据可以按指定字段排序。使用`sort by`你可以指定执行的reduce个数(通过`set mapred.reduce.tasks=n`来指定)，对输出的数据再执行归并排序，即可得到全部结果。
设置reducers数为3的时候
```angular2html
> insert overwrite local directory '/opt/module/datas/results_sort_by'
> select * from tb sort by salary;

```
输出的三个文件中其中之一的`salary`（倒数第三列）

    7654MARTINSALESMAN76981981-9-28 1250.0 1400.030
    7844TURNERSALESMAN76981981-9-8 1500.0 0.030
    7782CLARKMANAGER78391981-6-9 2450.0 \N10
    7698BLAKEMANAGER78391981-5-1 2850.0 \N30
    7788SCOTTANALYST75661987-4-19 3000.0 \N20
    7839KINGPRESIDENT\N1981-11-17 5000.0 \N10
    
可见是局部有序的
当设置`mapred.reduce.tasks=1`的时候同样输出全局有序的单个文件

    7369SMITHCLERK79021980-12-17800.0\N20
    7900JAMESCLERK76981981-12-3950.0\N30
    7876ADAMSCLERK77881987-5-231100.0\N20
    7521WARDSALESMAN76981981-2-221250.0500.030
    7654MARTINSALESMAN76981981-9-281250.01400.030
    7934MILLERCLERK77821982-1-231300.0\N10
    7844TURNERSALESMAN76981981-9-81500.00.030
    7499ALLENSALESMAN76981981-2-201600.0300.030
    7782CLARKMANAGER78391981-6-92450.0\N10
    7698BLAKEMANAGER78391981-5-12850.0\N30
    7566JONESMANAGER78391981-4-22975.0\N20
    7788SCOTTANALYST75661987-4-193000.0\N20
    7902FORDANALYST75661981-12-33000.0\N20
    7839KINGPRESIDENT\N1981-11-175000.0\N10
    
3. `distribute by`

`distribute by`是控制在map端如何拆分数据给reduce端的。hive会根据`distribute by`后面列，对应reduce的个数进行分发，默认是采用hash算法。`sort by`为每个reduce产生一个排序文件。在有些情况下，你需要控制某个特定行应该到哪个reducer，这通常是为了进行后续的聚集操作。`distribute by`刚好可以做这件事。因此，`distribute by`经常和`sort by`配合使用。 类似mapreduce中的分区函数

注：`Distribute by`和`sort by`的使用场景：

    1.Map输出的文件大小不均。

    2.Reduce输出文件大小不均。
    
    3.小文件过多。
    
    4.文件超大。
    
根据最后一列的`deptno`进行`distribute by`:

```angular2html
hive (default)> set mapreduce.job.reduces=3;
hive (default)> select * from tb distribute by deptno sort by salary;

```
结果会分成根据deptno分成三个文件，每个文件的deptno分别为30\10\20（根据哈希模余reducer的个数得到分区号），并根据salary排序（可以看出`distribute by`的作用类似分区函数。


    Query ID = hadoop_20181109205054_bd370d06-dd98-44b7-87a4-6cafdb0b4364
    Total jobs = 1
    Launching Job 1 out of 1
    Number of reduce tasks not specified. Defaulting to jobconf value of: 3
    In order to change the average load for a reducer (in bytes):
      set hive.exec.reducers.bytes.per.reducer=<number>
    In order to limit the maximum number of reducers:
      set hive.exec.reducers.max=<number>
    In order to set a constant number of reducers:
      set mapreduce.job.reduces=<number>
    Starting Job = job_1539486568772_0009, Tracking URL = http://hadoop101:8088/proxy/application_1539486568772_0009/
    Kill Command = /opt/module/hadoop-2.7.2/bin/hadoop job  -kill job_1539486568772_0009
    Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 3
    2018-11-09 20:51:04,151 Stage-1 map = 0%,  reduce = 0%
    2018-11-09 20:51:12,513 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.3 sec
    2018-11-09 20:51:22,006 Stage-1 map = 100%,  reduce = 33%, Cumulative CPU 2.65 sec
    2018-11-09 20:51:27,183 Stage-1 map = 100%,  reduce = 67%, Cumulative CPU 3.99 sec
    2018-11-09 20:51:28,236 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 5.29 sec
    MapReduce Total cumulative CPU time: 5 seconds 290 msec
    Ended Job = job_1539486568772_0009
    MapReduce Jobs Launched: 
    Stage-Stage-1: Map: 1  Reduce: 3   Cumulative CPU: 5.29 sec   HDFS Read: 16307 HDFS Write: 661 SUCCESS
    Total MapReduce CPU Time Spent: 5 seconds 290 msec
    OK
    tb.id	tb.name	tb.job	tb.m_id	tb.day	tb.salary	tb.salary2	tb.deptno
    7900	JAMES	CLERK	7698	1981-12-3	950.0	NULL	30
    7521	WARD	SALESMAN	7698	1981-2-22	1250.0	500.0	30
    7654	MARTIN	SALESMAN	7698	1981-9-28	1250.0	1400.0	30
    7844	TURNER	SALESMAN	7698	1981-9-8	1500.0	0.0	30
    7499	ALLEN	SALESMAN	7698	1981-2-20	1600.0	300.0	30
    7698	BLAKE	MANAGER	7839	1981-5-1	2850.0	NULL	30
    7934	MILLER	CLERK	7782	1982-1-23	1300.0	NULL	10
    7782	CLARK	MANAGER	7839	1981-6-9	2450.0	NULL	10
    7839	KING	PRESIDENT	NULL	1981-11-17	5000.0	NULL	10
    7369	SMITH	CLERK	7902	1980-12-17	800.0	NULL	20
    7876	ADAMS	CLERK	7788	1987-5-23	1100.0	NULL	20
    7566	JONES	MANAGER	7839	1981-4-2	2975.0	NULL	20
    7788	SCOTT	ANALYST	7566	1987-4-19	3000.0	NULL	20
    7902	FORD	ANALYST	7566	1981-12-3	3000.0	NULL	20


4. `cluster by`
`cluster by`除了具有`distribute by`的功能外还兼具`sort by`的功能。也就是说进入每个reducer之后，键可以是有序的。然而单靠`distribute by`则不能做到，他可以保证相同的键进入相同的分区，但是未必有序。
但是排序只能是倒序排序，不能指定排序规则为`ASC`或者`DESC`。可以认为是是`distribute by` + `sort by`的结合。

**所以`distribute by`和`sort by`总在一起使用，当二者的字段名相同的时候就可以写成`cluster by`**

例子：

    x1
    x2
    x4
    x3
    x1
    
设置两个reducer，
经过`distribute by`：

    SELECT ... FROM t1 DISTRIBUTE BY col1
    
两个reducer会分别得到

    x1
    x2
    x1
    
和

    x4
    x3
    
    
但是`cluster by`则可以得到：


    x1
    x1
    x2

和

    x3
    x4