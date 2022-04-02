## Hive查询

### 1、基本查询

#### 1.1 全表和特定列查询

0）数据准备

（0）原始数据

dept：

```xml
10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
40	OPERATIONS	1700
```

emp：

```xml
369	SMITH	CLERK	7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-2-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-2-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-4-2	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-9-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-5-1	2850.00		30
7782	CLARK	MANAGER	7839	1981-6-9	2450.00		10
7788	SCOTT	ANALYST	7566	1987-4-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-9-8	1500.00	0.00	30
7876	ADAMS	CLERK	7788	1987-5-23	1100.00		20
7900	JAMES	CLERK	7698	1981-12-3	950.00		30
7902	FORD	ANALYST	7566	1981-12-3	3000.00		20
7934	MILLER	CLERK	7782	1982-1-23	1300.00		10
```

（1）创建部门表

```sql
create table if not exists dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

（2）创建员工表

```sql
create table if not exists emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

（3）导入数据

```sql
load data local inpath '/opt/module/datas/dept.txt' into table dept;
load data local inpath '/opt/module/datas/emp.txt' into table emp;
```

1）全表查询

```sql
hive (default)> select * from emp;
hive (default)> select empno,ename,job,mgr,hiredate,sal,comm,deptno from emp;
```

2）选择特定列查询

```sql
hive (default)> select empno, ename from emp;
```

注意：

（1）SQL 语言大小写不敏感。 

（2）SQL 可以写在一行或者多行

（3）关键字不能被缩写也不能分行

（4）各子句一般要分行写。

（5）使用缩进提高语句的可读性。

#### 1.2 列别名

1）重命名一个列

2）便于计算

3）紧跟列名，也可以在列名和别名之间加入关键字‘AS’ 

4）案例实操

查询名称和部门

```sql
hive (default)> select ename AS name, deptno dn from emp;
```

#### 1.3 算术运算符

| 运算符 | 描述           |
| ------ | -------------- |
| A+B    | A和B 相加      |
| A-B    | A减去B         |
| A*B    | A和B 相乘      |
| A/B    | A除以B         |
| A%B    | A对B取余       |
| A&B    | A和B按位取与   |
| A\|B   | A和B按位取或   |
| A^B    | A和B按位取异或 |
| ~A     | A按位取反      |

案例实操：查询出所有员工的薪水后加1显示。

```sql
hive (default)> select sal +1 from emp;
```

#### 1.4 常用函数

1）求总行数（count）

```sql
hive (default)> select count(*) cnt from emp;
```

2）求工资的最大值（max）

```sql
hive (default)> select max(sal) max_sal from emp;
```

3）求工资的最小值（min）

```sql
hive (default)> select min(sal) min_sal from emp;
```

4）求工资的总和（sum）

```sql
hive (default)> select sum(sal) sum_sal from emp; 
```

5）求工资的平均值（avg）

```sql
hive (default)> select avg(sal) avg_sal from emp;
```

#### 1.5 Limit语句

典型的查询会返回多行数据。LIMIT子句用于限制返回的行数。

```sql
hive (default)> select * from emp limit 5;
hive (default)> select * from emp limit 2,3;
```

#### 1.6 Where语句

1）使用WHERE子句，将不满足条件的行过滤掉

2）WHERE子句紧随FROM子句

3）案例实操

查询出薪水大于1000的所有员工

```sql
hive (default)> select * from emp where sal >1000;
```

注意：where子句中不能使用字段别名。

#### 1.7 比较运算符（Between/In/ Is Null）

1）下面表中描述了谓词操作符，这些操作符同样可以用于JOIN…ON和HAVING语句中。

| 操作符                  | 支持的数据类型 | 描述                                                         |
| ----------------------- | -------------- | ------------------------------------------------------------ |
| A=B                     | 基本数据类型   | 如果A等于B则返回TRUE，反之返回FALSE                          |
| A<=>B                   | 基本数据类型   | 如果A和B都为NULL，则返回TRUE，如果一边为NULL，返回False      |
| A<>B, A!=B              | 基本数据类型   | A或者B为NULL则返回NULL；如果A不等于B，则返回TRUE，反之返回FALSE |
| A<B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于B，则返回TRUE，反之返回FALSE |
| A<=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A小于等于B，则返回TRUE，反之返回FALSE |
| A>B                     | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于B，则返回TRUE，反之返回FALSE |
| A>=B                    | 基本数据类型   | A或者B为NULL，则返回NULL；如果A大于等于B，则返回TRUE，反之返回FALSE |
| A [NOT] BETWEEN B AND C | 基本数据类型   | 如果A，B或者C任一为NULL，则结果为NULL。如果A的值大于等于B而且小于或等于C，则结果为TRUE，反之为FALSE。如果使用NOT关键字则可达到相反的效果。 |
| A IS NULL               | 所有数据类型   | 如果A等于NULL，则返回TRUE，反之返回FALSE                     |
| A IS NOT NULL           | 所有数据类型   | 如果A不等于NULL，则返回TRUE，反之返回FALSE                   |
| IN(数值1, 数值2)        | 所有数据类型   | 使用 IN运算显示列表中的值                                    |
| A [NOT] LIKE B          | STRING 类型    | B是一个SQL下的简单正则表达式，也叫通配符模式，如果A与其匹配的话，则返回TRUE；反之返回FALSE。B的表达式说明如下：‘x%’表示A必须以字母‘x’开头，‘%x’表示A必须以字母’x’结尾，而‘%x%’表示A包含有字母’x’,可以位于开头，结尾或者字符串中间。如果使用NOT关键字则可达到相反的效果。 |
| A RLIKE B, A REGEXP B   | STRING 类型    | B是基于java的正则表达式，如果A与其匹配，则返回TRUE；反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串A相匹配，而不是只需与其字符串匹配。 |

2）案例实操

（1）查询出薪水等于5000的所有员工

```sql
hive (default)> select * from emp where sal =5000;
```

（2）查询工资在500到1000的员工信息

```sql
hive (default)> select * from emp where sal between 500 and 1000;
```

（3）查询comm为空的所有员工信息

```sql
hive (default)> select * from emp where comm is null;
```

（4）查询工资是1500或5000的员工信息

```sql
hive (default)> select * from emp where sal IN (1500, 5000);
```

#### 1.8 Like和RLike

1）使用LIKE运算选择类似的值

2）选择条件可以包含字符或数字：

% 代表零个或多个字符(任意个字符)。

_ 代表一个字符。

3）RLIKE子句

RLIKE子句是Hive中这个功能的一个扩展，其可以通过Java的正则表达式这个更强大的语言来指定匹配条件。

4）案例实操

（1）查找名字以A开头的员工信息

hive (default)> select * from emp where ename LIKE 'A%';

（2）查找名字中第二个字母为A的员工信息

hive (default)> select * from emp where ename LIKE '_A%';

（3）查找名字中带有A的员工信息

hive (default)> select * from emp where ename  RLIKE '[A]';

#### 1.9 逻辑运算符（And/Or/Not）

| 操作符 | 含义   |
| ------ | ------ |
| AND    | 逻辑并 |
| OR     | 逻辑或 |
| NOT    | 逻辑否 |

1）案例实操

（1）查询薪水大于1000，部门是30

```sql
hive (default)> select * from emp where sal>1000 and deptno=30;
```

（2）查询薪水大于1000，或者部门是30

```sql
hive (default)> select * from emp where sal>1000 or deptno=30;
```

（3）查询除了20部门和30部门以外的员工信息

```sql
hive (default)> select * from emp where deptno not IN(30, 20);
```

### 2、分组

#### 2.1 Group By语句

GROUP BY语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然后对每个组执行聚合操作。

1）案例实操：

（1）计算emp表每个部门的平均工资

```sql
hive (default)> select t.deptno, avg(t.sal) avg_sal from emp t group by t.deptno;
```

（2）计算emp每个部门中每个岗位的最高薪水

```sql
hive (default)> select t.deptno, t.job, max(t.sal) max_sal from emp t group by t.deptno, t.job;
```

#### 2.2 Having语句

1）having与where不同点

（1）where后面不能写分组函数，而having后面可以使用分组函数。

（2）having只用于group by分组统计语句。

2）案例实操

（1）求每个部门的平均薪水大于2000的部门

求每个部门的平均工资

```sql
hive (default)> select deptno, avg(sal) from emp group by deptno;
```

求每个部门的平均薪水大于2000的部门

```sql
hive (default)> select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000;
```

### 3、Join语句

#### 3.1 等值Join

Hive支持通常的SQL JOIN语句。 

1）案例实操

（1）根据员工表和部门表中的部门编号相等，查询员工编号、员工名称和部门名称；

```sql
hive (default)> select e.empno, e.ename, d.deptno, d.dname from emp e join dept d on e.deptno = d.deptno;
```

#### 3.2 表的别名

1）好处

（1）使用别名可以简化查询。

（2）使用表名前缀可以提高执行效率。

2）案例实操

合并员工表和部门表

```sql
hive (default)> select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno = d.deptno;
```

#### 3.3 内连接

内连接：只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来。

```sql
hive (default)> select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno = d.deptno; 
```

#### 3.4 左外连接

左外连接：JOIN操作符左边表中符合WHERE子句的所有记录将会被返回。

```sql
hive (default)> select e.empno, e.ename, d.deptno from emp e left join dept d on e.deptno = d.deptno;
```

#### 3.5 右外连接

右外连接：JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。

```sql
hive (default)> select e.empno, e.ename, d.deptno from emp e right join dept d on e.deptno = d.deptno;
```

#### 3.6 满外连接

满外连接：将会返回所有表中符合WHERE语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL值替代。

```sql
hive (default)> select e.empno, e.ename, d.deptno from emp e full join dept d on e.deptno = d.deptno;
```

#### 3.7 多表连接

注意：连接 n个表，至少需要n-1个连接条件。例如：连接三个表，至少需要两个连接条件。

数据准备

```xml
1700	Beijing
1800	London
1900	Tokyo
```

1）创建位置表

```sql
create table if not exists location(
loc int,
loc_name string
)
row format delimited fields terminated by '\t';
```

2）导入数据

```sql
hive (default)> load data local inpath '/opt/module/datas/location.txt' into table location;
```

3）多表连接查询

```sql
hive (default)>SELECT e.ename, d.dname, l.loc_name 
FROM emp e
JOIN   dept d
ON     d.deptno = e.deptno 
JOIN   location l
ON     d.loc = l.loc;
```

大多数情况下，Hive会对每对JOIN连接对象启动一个MapReduce任务。本例中会首先启动一个MapReduce job对表e和表d进行连接操作，然后会再启动一个MapReduce job将第一个MapReduce job的输出和表l;进行连接操作。

注意：为什么不是表d和表l先进行连接操作呢？这是因为Hive总是按照从左到右的顺序执行的。

优化：当对3个或者更多表进行join连接时，如果每个on子句都使用相同的连接键的话，那么只会产生一个MapReduce job。

#### 3.8 笛卡尔积

1）笛卡尔积会在下面条件下产生

（1）省略连接条件

（2）连接条件无效

（3）所有表中的所有行互相连接

2）案例实操

```sql
hive (default)> select empno, dname from emp, dept;
```

### 4、排序

#### 4.1 全局排序（Order By）

Order By：全局排序，只有一个Reducer

1）使用 ORDER BY 子句排序

- ASC（ascend）：升序（默认）

- DESC（descend）：降序

2）ORDER BY 子句在SELECT语句的结尾

3）案例实操 

（1）查询员工信息按工资升序排列

```sql
hive (default)> select * from emp order by sal;
```

（2）查询员工信息按工资降序排列

```sql
hive (default)> select * from emp order by sal desc;
```

#### 4.2 按照别名排序

按照员工薪水的2倍排序

```sql
hive (default)> select ename, sal*2 twosal from emp order by twosal;
```

#### 4.3 多个列排序

按照部门和工资升序排序

```sql
hive (default)> select ename, deptno, sal from emp order by deptno, sal;
```

#### 4.4 每个Reduce内部排序（Sort By）

Sort By：对于大规模的数据集order by的效率非常低。在很多情况下，并不需要全局排序，此时可以使用sort by。

Sort by为每个reducer产生一个排序文件。每个Reducer内部进行排序，对全局结果集来说不是排序。

1）设置reduce个数

```sql
hive (default)> set mapreduce.job.reduces=3;
```

2）查看设置reduce个数

```sql
hive (default)> set mapreduce.job.reduces;
```

3）根据部门编号降序查看员工信息

```sql
hive (default)> select * from emp sort by deptno desc;
```

4）将查询结果导入到文件中（按照部门编号降序排序）

```sql
hive (default)> insert overwrite local directory '/opt/module/hive/datas/sortby-result'
 select * from emp sort by deptno desc;
```

#### 4.5 分区（Distribute By）

Distribute By：在有些情况下，我们需要控制某个特定行应该到哪个reducer，通常是为了进行后续的聚集操作。distribute by 子句可以做这件事。distribute by类似MR中partition（自定义分区），进行分区，结合sort by使用。 

对于distribute by进行测试，一定要分配多reduce进行处理，否则无法看到distribute by的效果。

1）案例实操：

（1）先按照部门编号分区，再按照员工编号降序排序。

```sql
hive (default)> set mapreduce.job.reduces=3;
hive (default)> insert overwrite local directory '/opt/module/hive/datas/distribute-result' select * from emp distribute by deptno sort by empno desc;
```

注意：

- distribute by的分区规则是根据分区字段的hash码与reduce的个数进行模除后，余数相同的分到一个区。

- Hive要求DISTRIBUTE BY语句要写在SORT BY语句之前。

#### 4.6 Cluster By

当distribute by和sort by字段相同时，可以使用cluster by方式。

cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是升序排序，不能指定排序规则为ASC或者DESC。

（1）以下两种写法等价

```sql
hive (default)> select * from emp cluster by deptno;
hive (default)> select * from emp distribute by deptno sort by deptno;
```

注意：按照部门编号分区，不一定就是固定死的数值，可以是20号和30号部门分到一个分区里面去。

### 5、抽样查询

对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive可以通过对表进行抽样来满足这个需求。

查询表stu_buck中的数据。

```sql
hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);
```

注：tablesample是抽样语句，语法：TABLESAMPLE(BUCKET x OUT OF y) 。

y必须是table总bucket数的倍数或者因子。hive根据y的大小，决定抽样的比例。例如，table总共分了4份，当y=2时，抽取(4/2=)2个bucket的数据，当y=8时，抽取(4/8=)1/2个bucket的数据。

x表示从哪个bucket开始抽取，如果需要取多个分区，以后的分区号为当前分区号加上y。例如，table总bucket数为4，tablesample(bucket 1 out of 2)，表示总共抽取（4/2=）2个bucket的数据，抽取第1(x)个和第3(x+y)个bucket的数据。

注意：x的值必须小于等于y的值，否则

```sql
FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck
```


