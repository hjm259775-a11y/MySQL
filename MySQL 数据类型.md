# MySQL 数据类型



## 数值类型

数值类型分**整数**和**小数**

### 1. 整数类型

整数就是存整数的，区别只在于**能存多大**。选的时候别一律 `INT` 或 `BIGINT`，够用就行，字段小一点，索引和存储都省。

整数类型默认是**有符号**的，也就是**能存负数**。加 `UNSIGNED` 属性后变成**无符号**，只能存非负数，但正数上限翻倍。**`UNSIGNED` 不改变占用的二进制位数，也不额外占空间**，它改变的是最高位怎么用：有符号时最高位是符号位，无符号时最高位也拿来表示数值。

| 类型        | 占几个二进制位  | 有符号范围（默认，带负号） | 无符号范围（`UNSIGNED`，无负号） |
| :---------- | :-------------- | :------------------------- | :------------------------------- |
| `TINYINT`   | 8 位（1 字节）  | -128 ~ 127                 | 0 ~ 255                          |
| `SMALLINT`  | 16 位（2 字节） | -32768 ~ 32767             | 0 ~ 65535                        |
| `MEDIUMINT` | 24 位（3 字节） | -8388608 ~ 8388607         | 0 ~ 16777215                     |
| `INT`       | 32 位（4 字节） | -2147483648 ~ 2147483647   | 0 ~ 4294967295                   |
| `BIGINT`    | 64 位（8 字节） | 约 -9.22×10¹⁸ ~ 9.22×10¹⁸  | 0 ~ 约 1.84×10¹⁹                 |

拿 `TINYINT` 举例：一共 8 个二进制位，默认有符号时最高位是符号位，剩下 7 位表示数值，范围 -128 ~ 127；加 `UNSIGNED` 后 8 位全部表示数值，范围 0 ~ 255。**位数没变，变的是最高位的解释方式。**

有符号 `TINYINT` 建表并插入：

```
CREATE TABLE t_signed (age TINYINT);
INSERT INTO t_signed (age) VALUES (100);   -- 成功，100 在 -128 ~ 127 之间
INSERT INTO t_signed (age) VALUES (-129);  -- 报错：Out of range value，-129 小于最小值 -128
```



无符号 `TINYINT` 建表并插入：

```
CREATE TABLE t_unsigned (age TINYINT UNSIGNED);
INSERT INTO t_unsigned (age) VALUES (-1);   -- 报错：Out of range value，无符号不允许负数
INSERT INTO t_unsigned (age) VALUES (256);  -- 报错：Out of range value，256 大于最大值 255
```





>   建表直接写 `INT`，别写 `INT(11)`——括号里的数字是**废弃的显示宽度**，不是长度限制。
>
>   -   `UNSIGNED` 是**列属性**，不是约束。约束是 `PRIMARY KEY`、`NOT NULL`、`UNIQUE`、`CHECK`、`DEFAULT`、`FOREIGN KEY` 这些。
>   -   加 `UNSIGNED` 后**负数存不进去**：严格模式下插 `-1` 会报错，非严格模式下可能被截断成 0。金额、差值这种可能为负的字段，别加。
>   -   有符号和无符号**混着比较或运算时容易出问题**，结果可能被当成无符号处理，出现意想不到的大数。

### 2. 小数类型：DECIMAL 和 FLOAT/DOUBLE

这里分两条路：

-   **`DECIMAL(M,D)`**：定点数，**精确存储**。`M` 是总位数，`D` 是小数位，整数位 = `M - D`。比如 `DECIMAL(5,2)` 就是总共 5 位、小数 2 位、整数 3 位，范围 `-999.99 ~ 999.99`。`DECIMAL` 默认也能存负数。
-   **`FLOAT` / `DOUBLE`**：浮点数，**近似存储**。`FLOAT` 占 4 字节，`DOUBLE` 占 8 字节，后者精度更高，但都可能出现舍入误差。同样默认有符号。

```
CREATE TABLE salary_table (
    salary DECIMAL(10,2),   -- 金额，必须精确，可正可负
    rate FLOAT,             -- 近似值场景，可正可负
    ratio DOUBLE
);
```



**为什么浮点数会有误差？**

因为 `FLOAT`/`DOUBLE` 底层是用**二进制**存小数的，而很多十进制小数（比如 0.1、0.2、3.1415926）转成二进制是**无限循环小数**，存不下，只能截断成近似值。十进制里 1/3 = 0.333... 写不完，二进制里 0.1 = 0.0001100110011... 也写不完，道理一样。

```
CREATE TABLE t7 (n1 FLOAT(10,8), n2 DECIMAL(10,8));
INSERT INTO t7 VALUES (3.1415926, 3.1415926);
SELECT * FROM t7;
-- n1 显示 3.14159250，有偏差：FLOAT 用二进制近似存，末尾被舍入
-- n2 显示 3.14159260，完全精确：DECIMAL 按十进制逐位存，不转二进制
```



`FLOAT` 大约只有 6~7 位有效数字，`DOUBLE` 大约 15~16 位，超过这个精度就可能失真。`DECIMAL` 不转二进制，直接按十进制存每一位，所以金额用它不会出现这种误差。

>   **注意**：金额、财务、积分这种**不能有误差**的，必须用 `DECIMAL`。用 `FLOAT`/`DOUBLE` 存钱，算着算着就多出个 0.0000001，对账的时候能让人崩溃。浮点类型只适合科学计算、统计近似值这类场景。

------

## 字符串类型

字符串最常用的是 `CHAR` 和 `VARCHAR`，核心区别就一个：**定长还是变长**。

### 1. CHAR 与 VARCHAR

-   **`CHAR(N)`**：固定长度，最大 255 字符。存的时候不够长会用空格补满，取的时候去掉尾部空格。适合长度非常固定的短字符串，比如性别、国家代码这种。检索效率略高，因为长度固定。
-   **`VARCHAR(N)`**：可变长度，只占实际字符需要的空间（外加 1~2 字节记录长度）。绝大多数变长字符串都用它。

```
CREATE TABLE users (
    gender CHAR(1),          -- 固定 1 位，M/F
    country_code CHAR(2),    -- 固定 2 位，CN/US
    name VARCHAR(50)         -- 名字长度不固定
);
```



超长会直接报错，不是截断：

```
CREATE TABLE t8 (str CHAR(6));
INSERT INTO t8 VALUES ('minbit');      -- 6 个字符，成功
INSERT INTO t8 VALUES ('minbit1');     -- 7 个字符，报错：Data too long
INSERT INTO t8 VALUES ('小米里的大麦'); -- 6 个汉字，成功（CHAR 的 N 是字符数）
INSERT INTO t8 VALUES ('1');           -- 1 个字符，成功，存储时补 5 个空格，查询时去掉
```



>   **易错点**：`VARCHAR(N)` 的 N 是**字符数**，不是字节数。一个字符占几个字节由字符集决定：`latin1` 是 1 字节，`utf8mb3` 最多 3 字节，`utf8mb4` 最多 4 字节。所以同样写 `VARCHAR(100)`，`latin1` 下最多占 100 字节，`utf8mb4` 下最多占 400 字节。
>
>   而且 MySQL 规定**整行所有列加起来不能超过 65535 字节**（`TEXT`/`BLOB` 除外），这个额度整行共享，`VARCHAR` 还要额外花 1~2 字节记长度。就算用最省的 `latin1`，写 `VARCHAR(65535)` 也会报 `Row size too large`，必须降到 65533 以下才建得出来。换成 `utf8mb4`，理论上限大约 16383 个字符。别以为写了 65535 就真能存 65535 个字符；真要存大文本，用 `TEXT` / `MEDIUMTEXT`。

### 2. TEXT 与 BLOB

存大块数据用的。

-   **`TEXT`**：大文本，有字符集，适合文章内容、日志、备注。
-   **`BLOB`**：二进制数据，无字符集，适合图片、文件流。

按容量还分 `TINYTEXT`、`TEXT`、`MEDIUMTEXT`、`LONGTEXT`，`BLOB` 同理。

```
CREATE TABLE articles (
    id INT,
    title VARCHAR(200),
    content TEXT,              -- 文章正文，用 TEXT
    cover_image BLOB           -- 图片二进制，用 BLOB
);
```



>   **易错点**：`TEXT`/`BLOB` 不能像普通 `VARCHAR` 那样随便用。它们通常不能有默认值，建索引时也得指定前缀长度，而且数据不一定和行存在一起，查询时可能多一次磁盘读取。能短就短，别滥用；真要存大文本，用 `TEXT` / `MEDIUMTEXT`。

------

## 日期与时间类型

`DATE`、`TIME`、`DATETIME`、`TIMESTAMP` 最常用，选型关键看两点：**要不要时区转换**、**范围够不够**。

| 类型        | 格式                  | 范围                                                  | 时区处理                     |
| :---------- | :-------------------- | :---------------------------------------------------- | :--------------------------- |
| `DATE`      | `YYYY-MM-DD`          | 1000-01-01 ~ 9999-12-31                               | 无                           |
| `TIME`      | `hh:mm:ss`            | -838:59:59 ~ 838:59:59                                | 无                           |
| `DATETIME`  | `YYYY-MM-DD hh:mm:ss` | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59             | **不做转换**，存什么读什么   |
| `TIMESTAMP` | `YYYY-MM-DD hh:mm:ss` | 1970-01-01 00:00:01 UTC ~ **2038-01-19 03:14:07 UTC** | 存时转 UTC，读时转回当前时区 |



```
CREATE TABLE orders (
    order_date DATE,           -- 只要日期
    order_time TIME,           -- 只要时间
    created_at TIMESTAMP,      -- 跨时区显示，用 TIMESTAMP
    contract_date DATETIME     -- 字面时间，范围超 2038，用 DATETIME
);
```



日期时间值必须用引号包裹，不然会被当成数学表达式：

```
INSERT INTO orders (order_date) VALUES ('2025-01-01');   -- 成功
INSERT INTO orders (order_date) VALUES (2025-01-01);     -- 报错：被解析成 2025-1-1 的减法
```



>   **易错点**：
>
>   -   `TIMESTAMP` 有 **2038 年问题**，上限就是 2038-01-19，超过就溢出。出生日期、合同日期这种必须用 `DATE` 或 `DATETIME`。
>   -   `TIMESTAMP` 会**随时区转换**，存时转 UTC，读时转回当前时区。跨时区显示时间（比如用户发布时间）用它更合适；但要“字面时间”（比如营业时间），用 `DATETIME` 更稳。

------

## 其他实用类型

-   **`BIT(M)`**：位字段，存 0/1 或少量标志位，比 `TINYINT` 更省空间。
-   **`ENUM`**：枚举，从预定义列表里选一个值。适合状态、类型这种固定选项。但后续要改枚举列表，得改表结构，不够灵活。
-   **`JSON`**：MySQL 5.7+ 支持，存 JSON 文档，可以配合虚拟列和 JSON 函数查询。适合结构不固定的扩展字段，但别拿它替代正经的关系表设计。

```
CREATE TABLE flags (
    is_active BIT(1),
    status ENUM('pending','paid','shipped'),
    extra JSON
);
```



`ENUM` 插入不在列表里的值会报错：

```
INSERT INTO flags (status) VALUES ('paid');     -- 成功
INSERT INTO flags (status) VALUES ('unknown');  -- 报错：Data truncated
```



>   **易错点**：`ENUM` 看着方便，但排序是按**定义顺序**排的，不是按字母；改枚举值要 `ALTER TABLE`，线上表大了有锁表风险。状态字段如果以后可能频繁增加选项，不如用 `TINYINT` + 代码里维护映射。

------

## 选型速查

-   **字符型**：绝大多数用 `VARCHAR`；极短且定长用 `CHAR`；大文本用 `TEXT`。`VARCHAR(N)` 的 N 是字符数，实际字节看字符集，整行不能超 65535 字节。
-   **整数**：够用就好，别一律 `BIGINT`；默认有符号能存负数，明确非负再加 `UNSIGNED`，位数不变、正数上限翻倍。建表直接写 `INT`，别写 `INT(11)`。
-   **小数**：金额必须 `DECIMAL(M,D)`，`M` 总位数、`D` 小数位、整数位 = `M - D`；`FLOAT`/`DOUBLE` 是二进制近似存储，十进制小数转二进制可能无限循环，所以有误差，别用来存钱。
-   **时间**：`DATE` 只存日期，`TIME` 只存时间，`DATETIME` 存字面日期时间、范围到 9999 年，`TIMESTAMP` 随时区转换、上限 2038-01-19。
-   **省空间**：状态用 `TINYINT`、布尔用 `BIT(1)`，比 `INT` 高效得多。
-   **别滥用**：`TEXT`/`BLOB` 别当普通字段用，`ENUM` 别当万能状态字段用。

------

这版把浮点数误差的原因写进正文了：二进制存十进制小数可能无限循环，只能截断成近似值，`FLOAT` 约 6~7 位有效数字、`DOUBLE` 约 15~16 位，`DECIMAL` 按十进制逐位存所以精确。其余部分保持不变。