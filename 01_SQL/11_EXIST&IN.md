## 简单介绍

<li>主要用于联合查询

```
一般来说，in是把外表和内表作hash 连接，而exists 是对外表作loop 循环，
假设有A、B两个表，使用时是这样的：

1、select * from A where id in (select id from B)--使用in

2、select * from A where exists(select B.id from B where B.id=A.id)--使用exists
也可以完全不使用in和exists：

3、select A.* from A,B where A.id=B.id--不使用in和exists

具体使用时到底选择哪一个，主要考虑查询效率问题：

第一条语句使用了A表的索引；

第二条语句使用了B表的索引；

第三条语句同时使用了A表、B表的索引；

如果A、B表的数据量不大，那么这三个语句执行效率几乎无差别；

如果A表大，B表小，显然第一条语句效率更高，反之，则第二条语句效率更高；

第三条语句尽管同时使用了A表、B表的索引，单扫描次数是笛卡尔乘积，效率最差。
```

## 详细例子

###  exists

exists表示()内子查询语句返回结果不为空说明where条件成立就会执行主sql语句，如果为空就表示where条件不成立，sql语句就不会执行。not exists和exists相反，子查询语句结果为空，则表示where条件成立，执行sql语句。负责不执行。

之前在学oracle数据库的时候，接触过exists，做过几个简单的例子,，如

1.如果部门名称中含有字母A，则查询所有员工信息(使用exists)
select * from emp where exists (select * from dept where dname like '%A%' and deptno = emp.deptno) temp and deptno=temp.deptno;

结果为:


     EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO
---------- ---------- --------- ---------- -------------- ---------- ---------- ----------
      7369 SMITH      CLERK           7902 17-12月-80            800                    20
      7499 ALLEN      SALESMAN        7698 20-2月 -81           1600        300         30
      7521 WARD       SALESMAN        7698 22-2月 -81           1250        500         30
      7566 JONES      MANAGER         7839 02-4月 -81           2975                    20
      7654 MARTIN     SALESMAN        7698 28-9月 -81           1250       1400         30
      7698 BLAKE      MANAGER         7839 01-5月 -81           2850                    30
      7782 CLARK      MANAGER         7839 09-6月 -81           2450                    10
      7788 SCOTT      ANALYST         7566 19-4月 -87           3000                    20
      7839 KING       PRESIDENT            17-11月-81           5000                    10
      7844 TURNER     SALESMAN        7698 08-9月 -81           1500          0         30
      7876 ADAMS      CLERK           7788 23-5月 -87           1100                    20

     EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO
---------- ---------- --------- ---------- -------------- ---------- ---------- ----------
      7900 JAMES      CLERK           7698 03-12月-81            950                    30
      7902 FORD       ANALYST         7566 03-12月-81           3000                    20
      7934 MILLER     CLERK           7782 23-1月 -82           1300                    10

已选择14行。

2.如果有平均工资不小于1500的部门信息则查询所有部门信息(使用not exists)
select * from dept where not exists (select deptno from emp where deptno = emp.deptno group by deptno having avg(sal) < 1500) and exists (select * from emp where emp.deptno = deptno);

 

关于exists的详细用法转载http://05rjyzl11.iteye.com/blog/699673

 

有两个简单例子，以说明 “exists”和“in”的效率问题

1) select * from T1 where exists(select 1 from T2 where T1.a=T2.a) ;

    T1数据量小而T2数据量非常大时，T1<<T2 时，1) 的查询效率高。

2) select * from T1 where T1.a in (select T2.a from T2) ;

     T1数据量非常大而T2数据量小时，T1>>T2 时，2) 的查询效率高。

exists 用法：

请注意 1）句中的有颜色字体的部分 ，理解其含义；

其中 “select 1 from T2 where T1.a=T2.a” 相当于一个关联表查询，相当于

“select 1 from T1,T2     where T1.a=T2.a”

但是，如果你当当执行 1） 句括号里的语句，是会报语法错误的，这也是使用exists需要注意的地方。

“exists（xxx）”就表示括号里的语句能不能查出记录，它要查的记录是否存在。

因此“select 1”这里的 “1”其实是无关紧要的，换成“*”也没问题，它只在乎括号里的数据能不能查找出来，是否存在这样的记录，如果存在，这 1） 句的where 条件成立。

 

### in 

继续引用上面的例子

“2) select * from T1 where T1.a in (select T2.a from T2) ”

这里的“in”后面括号里的语句搜索出来的字段的内容一定要相对应，一般来说，T1和T2这两个表的a字段表达的意义应该是一样的，否则这样查没什么意义。

打个比方：T1，T2表都有一个字段，表示工单号，但是T1表示工单号的字段名叫“ticketid”，T2则为“id”，但是其表达的意义是一样的，而且数据格式也是一样的。这时，用 2）的写法就可以这样：

“select * from T1 where T1.ticketid in (select T2.id from T2) ”

Select name from employee where name not in (select name from student);

Select name from employee where not exists (select name from student);

第一句SQL语句的执行效率不如第二句。

通过使用EXISTS，Oracle会首先检查主查询，然后运行子查询直到它找到第一个匹配项，这就节省了时间。Oracle在执行IN子查询时，首先执行子查询，并将获得的结果列表存放在一个加了索引的临时表中。在执行子查询之前，系统先将主查询挂起，待子查询执行完毕，存放在临时表中以后再执行主查询。这也就是使用EXISTS比使用IN通常查询速度快的原因
