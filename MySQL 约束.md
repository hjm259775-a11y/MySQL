# MySQL 约束

## 非空约束 NOT NULL

`NOT NULL` 限制某一列**不允许出现 NULL 值**。只能对**单列**设置，不能组合设置。

### 建表时设置

```
CREATE DATABASE yueshu;
USE yueshu;

CREATE TABLE IF NOT EXISTS not_null_test1 (
    id INT NOT NULL
);
```



### 查看约束信息

```
-- 1. 看建表语句
SHOW CREATE TABLE not_null_test1;

-- 2. 看表结构
DESC not_null_test1;
```



`DESC` 输出里 `Key` 列的含义：`PRI` 是主键或主键的组成列，`UNI` 是唯一约束。

```
-- 3. 查系统表
SELECT * FROM information_schema.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'not_null_test1';

-- 4. 查索引信息
SHOW INDEX FROM not_null_test1;
```



### 插入 NULL 会报错

```
CREATE TABLE IF NOT EXISTS not_null_test2 (
    id INT NOT NULL,
    phone VARCHAR(11)
);

INSERT INTO not_null_test2 VALUES (1, NULL);  -- 成功：phone 没加 NOT NULL
```



### 建表后添加非空约束

```
-- 如果这个字段已经有 NULL 值，非空约束会添加失败，必须先清掉 NULL 记录
DELETE FROM not_null_test2;

ALTER TABLE not_null_test2
MODIFY phone VARCHAR(11) NOT NULL;

DESC not_null_test2;
```



>   **易错点**：建表后加 `NOT NULL`，如果列里已有 NULL 值，`ALTER TABLE` 会直接失败。先把 NULL 处理掉再加。

## 唯一约束 UNIQUE

`UNIQUE` 保证列值**不重复**。可以单列设置，也可以多列组合设置。

-   唯一约束如果不起名字，默认用**列名**当约束名。
-   唯一约束中**允许 NULL 值，并且可以出现多个 NULL**（NULL 不等于 NULL）。
-   同一张表可以有**多个**唯一约束。
-   唯一约束默认会生成**唯一索引**。

### 建表时设置

```
CREATE TABLE unique_1 (
    uname VARCHAR(5) UNIQUE,              -- 单列唯一，约束名默认是 uname
    age INT,
    phone_number VARCHAR(11),
    email VARCHAR(11),
    CONSTRAINT uk_phone UNIQUE (phone_number)  -- 起名字的唯一约束
);

DESC unique_1;
```



### 多列组合唯一约束

```
CREATE TABLE unique_2 (
    uname VARCHAR(5) UNIQUE,
    age INT,
    phone_number VARCHAR(11),
    email VARCHAR(11),
    CONSTRAINT uk_phone_email UNIQUE (phone_number, email)  -- 两列组合起来不重复
);
```



组合唯一的意思是：`phone_number + email` 这个组合不能重复，单独一列重复没关系。

### 建表后添加唯一约束

```
-- 方式一：起约束名
ALTER TABLE unique_2
ADD CONSTRAINT UK_email UNIQUE (email);

-- 方式二：直接改列定义
ALTER TABLE unique_2
MODIFY email INT UNIQUE;

DESC unique_2;
SHOW INDEX FROM unique_2;
```



### 删除唯一约束

```
-- 删除唯一约束实际上是删除唯一索引
ALTER TABLE unique_2
DROP INDEX uk_phone_email;
```



>   **易错点**：删除唯一约束用 `DROP INDEX`，不是 `DROP CONSTRAINT`。因为唯一约束在 MySQL 里就是靠唯一索引实现的。

## 主键约束 PRIMARY KEY

**主键 = 非空 + 唯一**

-   一张表**最多一个**主键约束。
-   主键约束的名字**不需要给，给了也没用**。
-   主键 = `NOT NULL` + `UNIQUE`：**值不能重复，NULL 值不能出现**。
-   可以是**单列主键**，也可以是**多列组合式主键**。

### 组合主键

```
CREATE TABLE primary_1 (
    uname VARCHAR(5),
    password VARCHAR(11),
    PRIMARY KEY (uname, password)  -- 两列组合起来唯一且非空
);

DESC primary_1;
```



### 删除主键

```
ALTER TABLE primary_1
DROP PRIMARY KEY;
```



>   **易错点**：
>
>   ​	组合主键的每一列都不能为 NULL
>
>   ​	**单独一列可以重复**，只要组合不重复就行。

## 自增列 AUTO_INCREMENT

`AUTO_INCREMENT` 让整数列自动递增，常配合主键用。

规则：

1.  自增列的列属性**必须是整数类型**，并且**必须是约束列**（主键约束或唯一约束）。
2.  一张表**最多只有一个**自增列。
3.  可以指定自增的**起始点**。

### 建表并使用

```
CREATE TABLE auto_in (
    id INT PRIMARY KEY AUTO_INCREMENT,
    uname VARCHAR(11)
);

INSERT INTO auto_in (uname)
VALUES ('aaaaaa'), ('bbb'), ('wangwu');

SELECT * FROM auto_in;
-- id 自动变成 1、2、3
```



### 删除数据后自增不回头

```
DELETE FROM auto_in WHERE id = 3;
DELETE FROM auto_in;
```



删掉数据后，自增计数器**不会重置**，下次插入还是从之前的值继续往上加。想重置得用 `ALTER TABLE ... AUTO_INCREMENT = 1` 或 `TRUNCATE TABLE`。

### 删除自增属性

```
-- 删除自增的时候，不能重复声明主键
ALTER TABLE auto_in
MODIFY id INT;

DESC auto_in;
```



>   **易错点**：
>
>   -   自增列必须是整数类型，并且必须是主键或唯一约束列，不能裸写 `AUTO_INCREMENT`。
>   -   `MODIFY id INT` 去掉自增时，不能再写 `PRIMARY KEY`，否则会报“重复定义主键”。
>   -   `DELETE` 不会重置自增计数器，只有 `TRUNCATE` 才会。
>   -   `AUTO_INCREMENT` **只能递增，不能递减**。`auto_increment_increment` 控制步长，但值必须大于 0，设成负数会被忽略。

## 外键约束 FOREIGN KEY

外键用来**连接两张表**：子表里的某一列，值必须来自父表的主键或唯一键。作用就是保证引用完整性——不能往子表里插一个父表里不存在的值。

### 建表时设置

```
-- 父表：部门
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);

-- 子表：员工，department_id 引用 departments.id
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    department_id INT,
    CONSTRAINT fk_dept FOREIGN KEY (department_id) REFERENCES departments(id)
);
```



### 插入验证

```
INSERT INTO departments VALUES (1, '研发部');

-- 成功：1 在父表里存在
INSERT INTO employees VALUES (1, '张三', 1);

-- 失败：99 在父表里不存在，外键约束报错
INSERT INTO employees VALUES (2, '李四', 99);
-- ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails
```



### 建表后添加外键

```
ALTER TABLE employees
ADD CONSTRAINT fk_dept FOREIGN KEY (department_id) REFERENCES departments(id);
```



### 删除外键

```
-- 先查外键名
SELECT CONSTRAINT_NAME FROM information_schema.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'employees' AND CONSTRAINT_TYPE = 'FOREIGN KEY';

-- 再删，注意删外键用 DROP FOREIGN KEY，不是 DROP INDEX
ALTER TABLE employees
DROP FOREIGN KEY fk_dept;
```



>   **易错点**：
>
>   -   外键列和父表被引用列**类型要一致**，父表被引用列必须是主键或唯一键。
>   -   子表里已有父表不存在的值时，加外键会失败，得先清掉脏数据。
>   -   删除外键用 `DROP FOREIGN KEY`，但**外键会自动生成一个普通索引**，这个索引要另外用 `DROP INDEX` 删。
>   -   父表里有被引用的行时，直接删父表行会报错，除非设置了 `ON DELETE CASCADE` 或 `ON DELETE SET NULL`。

## 检查约束 CHECK

`CHECK` 用来限制列值必须满足某个条件。MySQL **8.0.16 起才真正生效**，之前版本会解析但忽略。

```
CREATE TABLE users (
    id INT PRIMARY KEY,
    age INT CHECK (age >= 0 AND age <= 150),
    gender CHAR(1) CHECK (gender IN ('M', 'F'))
);

INSERT INTO users VALUES (1, 25, 'M');    -- 成功
INSERT INTO users VALUES (2, -1, 'M');    -- 报错：CHECK 约束失败
INSERT INTO users VALUES (3, 25, 'X');    -- 报错：gender 不在 M/F 里
```



>   **易错点**：MySQL 8.0.16 之前的版本写 `CHECK` 不报错但也不生效，别以为加了就安全。用旧版本得靠触发器或应用层校验。

## 默认值 DEFAULT

`DEFAULT` 给列指定默认值，插入时不写这一列就用默认值。

```
CREATE TABLE orders (
    id INT PRIMARY KEY,
    status VARCHAR(10) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 不写 status，自动用 'pending'
INSERT INTO orders (id) VALUES (1);
SELECT * FROM orders;
-- id=1, status='pending', created_at=当前时间
```



>   **易错点**：`TEXT`/`BLOB` 通常不能有默认值；`DEFAULT CURRENT_TIMESTAMP` 只对 `TIMESTAMP`/`DATETIME` 有效；`DEFAULT` 不阻止显式插入 NULL，除非同时加了 `NOT NULL`。

------

## 约束速查

-   **`NOT NULL`**：单列不允许 NULL，只能单列设置，建表后加之前要先清掉已有 NULL。
-   **`UNIQUE`**：列值不重复，允许多个 NULL，可单列可组合，默认生成唯一索引，删除用 `DROP INDEX`。
-   **`PRIMARY KEY`**：非空 + 唯一，一张表最多一个，名字给了也没用，可单列可组合。
-   **`AUTO_INCREMENT`**：整数列 + 必须是主键或唯一约束，一张表最多一个，`DELETE` 不重置计数器，`TRUNCATE` 才重置，只能增不能减。
-   **`FOREIGN KEY`**：子表引用父表主键/唯一键，类型要一致，删除用 `DROP FOREIGN KEY`，自动生成的索引要另外删。
-   **`CHECK`**：限制列值满足条件，MySQL 8.0.16 起才真正生效。
-   **`DEFAULT`**：不写该列时用默认值，`TEXT`/`BLOB` 通常不能有默认值。

------

这版把你原来的随记内容全部保留，补上了外键、`CHECK`、`DEFAULT`，以及自增只能增不能减的说明。约束就这七种，没有其他大类了。