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

