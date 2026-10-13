# MySQL



SQL:一门用于管理操作关系型数据库的语言



​	SQL：基于功能划分出如下的种类：



DDL:数据定义语言：定义数据库、表\删除、修改    

​	如：CREATE \ALTER\DROP

DML :数据操作语言   表中数据的增删改查的   

​	如：INSERT\DELECT\UPDATE\SELECT(查找)DQL

DCL：数据控制语言 （控制权限的下发）



## sql语法的规范
 1、sql语句的最后会以;表示结束，

​	\g  \G  也可以作为语句的结束

2、sql语句为了保证可读性，一般会存在多行,并且使用缩进

3、关键字和保留字不能作为数据库的名字和表的名字 

4、标点符号   

​	'字符串'   

​	"给列起的别名"   

着重号（``）表示的是内部的名称只是普通的名称

```sql
SELECT employee_id,department_id 
FROM `uname`;

可以看到，要是你有一张表的名称是uname这个特殊字段，就得用``包起来
```





```sql
-- 注释
# 注释
/*
注释
*/
```





sql语句是否严格的区分大小写

 WINDOWS下，不区分大小写

服务器一般是在linux下，linux下默认是严格区分大小写的，但实际可以修改成不严格区分





````sql
DESCRIBE employees;
````







## 导入数据库文件



```
 mysql -P 3306 localhost -u root -p
```

先登录上自己的mysql







```sql
mysql> source /路径/文件名.sql
```

将你要导入的数据库文件的路径复制







```
USE example;

```

切到example数据库





## 基本操作



### SELECT

```
SELECT 1+2;
SELECT first_name,salary; ——————————————————————————————————————列就为first_name和salary


SELECT决定了你所要查询的列
```





### 引用表格

```
SELECT job_id
FROM employees;

SELECT job_id,last_name
FROM example.employees; ————————————————————————————————也可以跟上数据库名

从employees表中查询列为job_id字段，不写FROM employees会不知道去哪张表找
```





### 查看表格

```
SELECT * FROM employees; ——————————————————————————————查看整个employees表格

SELECT * FROM departments;——————————————————————————————查看整个departments表格

我喜欢这个，因为作者英语不好，总是需要回去看列名
```



```
DESCRIBE departments;

读的是表定义，不查数据
```







### 别名

```
SELECT job_id AS nailong,last_name AS huawei
FROM employees AS em;

SELECT job_id nailong,last_name huawei
FROM employees em;

给job_id起个别名nailong，后面的以此类推，这样所查询的表格的列名就会改为nailong
```





### 去重

```
SELECT DISTINCT job_id
FROM employees;

从employees表中查询列为job_id字段，同时给job_id去重
```





### 设定条件

```
SELECT first_name,salary
FROM employees
WHERE salary>=12000;

列名是first_name和salary，在employees表中找，并且只要salary>=12000的
```





### 综合

```sql
SELECT * FROM employees;
SELECT * FROM departments;

# 1
SELECT salary * 12 * (1+IFNULL(commission_pct,0)) AS 'ANNUAL SALARY'
FROM employees;

# 2
SELECT DISTINCT job_id
FROM employees;

# 3
SELECT first_name,salary
FROM employees
WHERE salary>=12000;

# 4
SELECT last_name,department_id
FROM employees
WHERE employee_id=176;

# 5
DESCRIBE departments;
SELECT * FROM departments;

```

【题目】
1，查询员工12个月的工资总和，并起别名为ANNUAL SALARY

2，查询employees表中去除重复的job_id以后的数据

3，查询工资大于12000的员工姓名和工资

4，查询员工号为176的员工的姓名和部门号

5，显示表 departments 的结构，并查询其中的全部数据