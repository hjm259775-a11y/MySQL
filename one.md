# MySQL



SQL:一门用于管理操作关系型数据库的语言



​	SQL：基于功能划分出如下的种类：



DDL:数据定义语言：定义数据库、表\删除、修改    

​	如：CREATE \ALTER\DROP

DML :数据操作语言：表中数据的增删改查的   

​	如：INSERT\DELETE\UPDATE\SELECT(查找)DQL

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
如果表名/字段名和关键字冲突，或含特殊字符，才需要反引号
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

**（所以你还是注意下大小写吧）**



````sql
DESCRIBE employees;

查看 employees 表的结构（表定义），并非查数据
````







## 导入数据库文件



```
 mysql -P 3306 localhost -u root -p
 
 端口号3306，本机，root用户，密码
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





## 基本操作 SELECT



### SELECT

```
SELECT 1+2;
SELECT first_name,salary; ————————————————————————列就为first_name和salary，当然，正常下面应该有from


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







```
SELECT 1 + NULL;

NULL 参与运算结果为 NULL
```









------------





## 进阶操作 SELECT







### 算数运算符





| 运算符  | 说明         |
| :-----: | :----------- |
|    +    | 加法         |
|    -    | 减法         |
|    *    | 乘法         |
|    /    | 除法         |
|   DIV   | 整除（取商） |
| % / MOD | 取余         |







注意：整数尝试和字符串进行加法运算的时候，字符串的数据类型会尝试转换为整型。如果字符串类型无法转换为整数，那么直接转换为0

注意：NULL参与运算直接输出NULL



```
SELECT 1+1,1+0.111,1+'1',10+'a',2-1.121,1-'a',12*3,12*-11,12/3.1

```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014221603452.png" alt="image-20261014221603452" style="zoom:80%;" />











### 比较运算符





| =    | 等于运算符（判断左右两边的值是否相等的，会进行数据类型的隐式转换 ，无法进行关于NULL的判断） |
| ---- | ------------------------------------------------------------ |
| <=>  | 安全等于运算符（可以用于进行NULL的判断）                     |
| ！=  | 这边几个不会的话可以重修C语言🤪                               |
| >    |                                                              |
| <    |                                                              |
| >=   |                                                              |
| <=   |                                                              |



```
SELECT NULL * 12  ,  0 = NULL  ,  1=2  ,  NULL = NULL  ,  NULL <=> NULL , 1 <> 1 , 1 != 2;
SELECT 'a' != 'b'  ,  (3+4) <> (2+6)  ,  'a' != NULL  ,  NULL <> NULL  ,  12>1  ,  12>'13'  ,  'a'>'b'  ,  'c'>'b'  ,  1>NULL;
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014222958239.png" alt="image-20261014222958239" style="zoom:67%;" />

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014223006283.png" alt="image-20261014223006283" style="zoom:67%;" />



```
SELECT employee_id AS "eid",last_name,commission_pct
FROM employees
WHERE NOT commission_pct <=> NULL ;

查找绩效的值不是NULL的员工的姓名  <=>
```













### 逻辑运算符



| &&   | 与   |
| ---- | ---- |
| \|\| | 或   |
| NOT  | 非   |

不懂的话还是去重修C语言（



```
SELECT last_name,department_id
FROM employees
WHERE department_id = 20 || department_id = 50;

查询department_id = 20或者department_id = 50的
```

```
SELECT last_name,department_id,commission_pct
FROM employees
WHERE salary > 1000 && commission_pct > 0.1;

从员工表中查找salary的值大于1000，并且绩效的值是>0.1的员工的姓名和部门id
```







## 关键字





### IS NULL



针对性太高，感觉不如 <=> 或者 NOT <> ，也有可能还有其他作用



| IS NULL     | 判断某个值是否为NULL值 |
| ----------- | ---------------------- |
| IS NOT NULL | 判断不是NULL值         |



```
SELECT employee_id,last_name
FROM employees
WHERE commission_pct IS NOT NULL;

条件为commission_pct不为NULL
```









### 最大最小运算符



| LEAST()    | 取括号内最小值（出现NULL值，直接返回NULL） |
| ---------- | ------------------------------------------ |
| GREATEST() | 取括号内最大值（出现NULL值，直接返回NULL） |



```
SELECT LEAST(1,12,-106,NULL),GREATEST(12,14,106,NULL);
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014223857257.png" alt="image-20261014223857257" style="zoom:67%;" />







### BETWEEN AND



```
a BETWEEN B  AND C  
```



抓取a的取值介于b-c之间的

（作者之前打算法的，感觉这东西纯纯鸡肋，比下不如&&好记，比上不如IN上限高）一坨😑



```
SELECT last_name,department_id
FROM employees
WHERE salary BETWEEN 5000 AND 7000;

查询salary在5000到7000之间的
```





```
SELECT  1 BETWEEN 0 AND 1   ,   10 BETWEEN 11 AND 12;
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014224840797.png" alt="image-20261014224840797" style="zoom:67%;" />









### IN 运算符



判断某个值是否存在于某个列表中



-- IN运算符  ：判断某个值是否存在于某个列表中  NOT IN



```
SELECT department_id
FROM employees
WHERE department_id NOT IN (10,50);

抓取department_id不在10到50内的数据
```









### LIKE



模糊匹配

| %    | 表示0个或多个字符 |
| ---- | ----------------- |
| _    | 表格一个字符      |

LIKE ：条件是满足的，则返回的是1   





```
SELECT * FROM employees
WHERE first_name LIKE '%a%'; ——————————————————————————————————————筛选first_name里面有a的

SELECT * FROM employees
WHERE first_name LIKE 'S____'; ————————————————————————————————————筛选first_name里面S开头，一共5个字符的
```



**`\`转义字符**

```
SELECT * FROM employees
WHERE first_name LIKE 's\_ab_' ——————————————————————— \表示的是去除掉后一个字符的特殊含义的
这个就是单纯的找first_name开头是s_ab，一共5个字符的

SELECT job_id
FROM jobs
WHERE job_id LIKE 'IT$_%' escape '$'; ———————————————— $代替了\
表示的是，在'IT$_%'字符串中'$'符号作为转义字符存在，相当于找IT_开头的，适用于不能打出\的场景
```





### REGEXP

REGEXP 配合正则使用的正则表达式

如果匹配成立，则返回的是1，否则的话，返回的是0





```
SELECT 'zhangsan' REGEXP '^a', 'zhangsan' REGEXP 'n$', 'zhangsan' REGEXP 'an';
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014230647377.png" alt="image-20261014230647377" style="zoom:67%;" />





### 位运算



| 运算符 | 名称     | 说明                                   |
| :----: | :------- | :------------------------------------- |
|   &    | 按位与   | 两位都为 1 时结果才为 1                |
|   \|   | 按位或   | 两位有一个为 1 结果就为 1              |
|   ^    | 按位异或 | 两位不同时为 1，相同时为 0             |
|   ~    | 按位取反 | 0 变 1，1 变 0                         |
|   <<   | 左移     | 二进制整体左移，右边补 0（数值变大）乘 |
|   >>   | 右移     | 二进制整体右移，左边补 0               |

2<<3就是2*2^3=16

16>>2就是16/2^2=4





------------------

运算符的优先级（—–->越来越高）

=(赋值)  or  and  not between 、比较运算符  ……  ()  -- 

（我也不知道这东西放哪，有或者说是有什么用，毕竟我一个括号甩过去谁管他优先级）

------------------------



**小练习**

```
#### 1
SELECT last_name,salary
FROM employees
WHERE NOT (salary >= 5000 && salary <=12000);

#### 2
SELECT last_name,department_id
FROM employees
WHERE department_id = 20 || department_id = 50;

#### 3
SELECT last_name,job_id
FROM employees
WHERE manager_id <=> NULL;

#### 4
SELECT last_name,salary,commission_pct
FROM employees
WHERE not commission_pct  <=> NULL;

#### 5
SELECT last_name
FROM employees
WHERE last_name LIKE '__a%';

#### 6
SELECT last_name
FROM employees
WHERE last_name LIKE '%a%' && last_name LIKE '%k%';

#### 7
SELECT first_name
FROM employees
WHERE first_name REGEXP 'e$';

#### 8
SELECT last_name,department_id
FROM employees
WHERE department_id >= 80 && department_id <=100; 

#### 9
SELECT last_name,salary,manager_id
FROM employees
WHERE manager_id = 100 || department_id = 101 || department_id =102; 
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014230318546.png" alt="image-20261014230318546" style="zoom: 67%;" />









### 排序

ORDER BY  排序依据的字段 ASC(表示的是升序默认的排序就是使用的升序) 、 DESC  降序



```
SELECT * FROM employees;
如果没有进行排序，那么默认查询出的额结果是当初插入数据的时候的结果

SELECT last_name,salary
FROM employees
WHERE department_id = 80
ORDER BY salary ASC;
查询部门id是80的员工的姓名、工资，并且按照工资升序

SELECT last_name,salary
FROM employees
WHERE department_id = 80
ORDER BY salary DESC;
查询部门id是80的员工的姓名、工资，并且按照工资降序


```



>注意： WHERE紧跟着FROM



```
SELECT last_name,salary AS gz
FROM employees
WHERE department_id = 80
ORDER BY 2;
查询部门id是80的员工的姓名、工资，并且按照第二列升序排序
```

注意：WHERE中不能使用别名  ORDER BY中可以使用别名（好像并没有什么用）





### 限定区域查找



```
SELECT * FROM employees LIMIT 0,3;
跳过前0条，看10条

SELECT * FROM employees LIMIT 11,3;
跳过前11条，看10条
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014231328736.png" alt="image-20261014231328736" style="zoom:67%;" />

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261014231432460.png" alt="image-20261014231432460" style="zoom:67%;" />

