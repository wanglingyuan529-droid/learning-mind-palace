---
tags: [MySQL, 数据库, SQL, DDL]
---

# MySQL DDL 教程

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"（底层源码、证明）
> 覆盖：06. 基础-SQL-DDL-数据库操作 ~ 10. 基础-SQL-DDL小结（共 5 集）
> 所有示例和输出都是在本机 **MariaDB 13.0.2** 上实测跑出来的；与 MySQL 8.0 的差异单列在文末附录

---

## 0. 一句话认识

**DDL 用 SQL 语句定义数据库的"骨架"：建库、建表、定字段类型、加约束、改结构、删结构。它管结构，不管数据。**

| 概念 | 解决什么问题 | 类比 |
| --- | --- | --- |
| 建库 / 建表 | 数据往哪儿放 | 盖房子先立框架 |
| 数据类型 | 每列能存什么样的值、多大 | 不同房间配不同家具 |
| 约束（NOT NULL 等） | 拦住脏数据 | 门框上装防盗门 |
| ALTER | 结构变了怎么改 | 房子改造 |
| DROP | 不要了怎么拆 | 拆房子（连地基一起） |

SQL 按用途分四类，先分清谁管结构、谁管数据：

| 类别 | 全称 | 干什么 | 典型语句 |
| --- | --- | --- | --- |
| **DDL** | Data **Definition** Language | 定义**结构** | `CREATE` / `ALTER` / `DROP` / `TRUNCATE` |
| DML | Data **Manipulation** Language | 增删改**数据** | `INSERT` / `UPDATE` / `DELETE` |
| DQL | Data **Query** Language | 查数据 | `SELECT` |
| DCL | Data **Control** Language | 权限 | `GRANT` / `REVOKE` |

> 记忆抓手：**DDL 改的是"说明书"，DML 改的是"库存"。** 所以 DDL 一执行就落定，回不了头（后面 10 集小结会实测）。

---

## 1. 为什么需要它

不解释"为什么"直接背语法，学完就忘。三个真实场景：

1. **项目上线前**：数据库里什么都没有，得先 `CREATE DATABASE`，再 `CREATE TABLE`，把字段、类型、约束全定好，程序才能往里写。
2. **线上表少了个字段**：用户反馈"要记录手机号"，你不可能重建表，得 `ALTER TABLE ... ADD COLUMN`。改错了就得靠 `DESC` 和 `SHOW CREATE TABLE` 查回来。
3. **程序有 bug 写进脏数据**：比如用户名是个空串、价格是 NULL。约束加在**数据库这一层**，就算程序代码写漏了、甚至有人直接连数据库手动改，也会被拦住。

---

## 2. 核心用法

### 2.1 数据库操作（对应 06 集）

| 操作 | 语句 |
| --- | --- |
| 查所有库 | `SHOW DATABASES;` |
| 查当前在哪个库 | `SELECT DATABASE();` |
| 建库 | `CREATE DATABASE [IF NOT EXISTS] 库名 [DEFAULT CHARACTER SET 字符集] [COLLATE 排序规则];` |
| 删库 | `DROP DATABASE [IF EXISTS] 库名;` |
| 切换库 | `USE 库名;` |

中括号 `[ ]` 表示**可选**，不是真的要你敲方括号。

#### `IF NOT EXISTS` / `IF EXISTS` 是干什么的

不是程序里的条件分支，作用是让语句**幂等**——重复执行不报错。

| 写法 | 位置 | 条件不满足时 | 不加的后果 |
| --- | --- | --- | --- |
| `IF NOT EXISTS` | CREATE | 库已存在 → 跳过 | `ERROR 1007: database exists` |
| `IF EXISTS` | DROP | 库不存在 → 跳过 | `ERROR 1008: database doesn't exist` |

实测：

```sql
CREATE DATABASE demo_ddl DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE demo_ddl;
SELECT DATABASE();          -- 返回 demo_ddl
SHOW TABLES;                -- 空
```

⚠️ `IF NOT EXISTS` 只是"跳过"，**不会修改**已存在库的字符集。要改得用 `ALTER DATABASE`。

#### 字符集（CHARACTER SET）和排序规则（COLLATE）

初学者最容易把这两件事混成一件。

| | 字符集 | 排序规则 |
| --- | --- | --- |
| 管什么 | 能存**哪些字符**、每个字符占几字节 | 这些字符**怎么比较**、怎么排序 |
| 例子 | `utf8mb4`、`gbk`、`latin1` | `utf8mb4_general_ci`、`utf8mb4_bin` |

一个字符集挂多个排序规则，所以建库时先给 CHARSET，再指定其中一种 COLLATE。

排序规则后缀含义：

| 后缀 | 全称 | 含义 |
| --- | --- | --- |
| `_ci` | case insensitive | **不**区分大小写（默认） |
| `_cs` | case sensitive | 区分大小写 |
| `_bin` | binary | 按字节比，区分大小写 |
| `_ai` / `_as` | accent insensitive / sensitive | 不区分 / 区分重音 |

它影响的全是比较结果，不是显示：

```sql
-- 排序规则是 utf8mb4_general_ci 时
SELECT 'A' = 'a';        -- 1（认为相等）
-- 换成 utf8mb4_bin 时
SELECT 'A' = 'a';        -- 0（不等）
```

最容易忽略的一条：**唯一索引也受它影响**。`_ci` 规则下 `'admin'` 和 `'Admin'` 会被当成重复值，第二条插入报 `ERROR 1062 Duplicate entry`。

查看命令：

```sql
SHOW CHARACTER SET;                              -- 所有字符集 + 各自默认排序规则
SHOW COLLATION WHERE Charset = 'utf8mb4';        -- 某字符集下所有可用排序规则
```

---

### 2.2 表操作：创建与查询（对应 07 集）

#### 建表语法骨架

```sql
CREATE TABLE [IF NOT EXISTS] 表名 (
    字段名 类型 [约束] [COMMENT '注释'],
    ...
    [PRIMARY KEY (字段)],
    [UNIQUE KEY 索引名 (字段)]
);
```

#### 常用约束一览

| 约束 | 作用 | 不写会怎样 |
| --- | --- | --- |
| `NOT NULL` | 该列必须有值 | 可为 NULL |
| `DEFAULT 值` | 省略该列时自动填 | 省略时没值可填 → 报 1364 |
| `PRIMARY KEY` | 唯一标识一行（隐含非空） | 无法唯一定位行 |
| `UNIQUE` | 值不许重复 | 可重复 |
| `AUTO_INCREMENT` | 自动发号 | 每次都得自己算 id |
| `COMMENT '说明'` | 给列加注释 | 别人看不懂这列干啥的 |

#### 一段完整的建表实测

```sql
CREATE TABLE product (
  id         INT           NOT NULL AUTO_INCREMENT COMMENT '主键',
  name       VARCHAR(100)  NOT NULL COMMENT '商品名',
  price      DECIMAL(10,2) NOT NULL DEFAULT 0.00 COMMENT '单价',
  stock      INT           NOT NULL DEFAULT 0,
  created_at DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  UNIQUE KEY uk_name (name)
);
```

`DESC product;` 的真实输出：

| Field | Type | Null | Key | Default | Extra |
| --- | --- | --- | --- | --- | --- |
| id | int(11) | NO | PRI | NULL | auto_increment |
| name | varchar(100) | NO | UNI | NULL | |
| price | decimal(10,2) | NO | | 0.00 | |
| stock | int(11) | NO | | 0 | |
| created_at | datetime | NO | | current_timestamp() | |

`Extra` 列告诉你"它会不会自己动"：`auto_increment` 是自动发号，`on update CURRENT_TIMESTAMP` 是自动改时间。

#### 三种查看方式的区别

| 语句 | 给什么 | 什么时候用 |
| --- | --- | --- |
| `SHOW TABLES;` | 只有表名列表 | 先知道有哪些表 |
| `DESC 表名;` | 字段、类型、是否可空、是否索引 | 快速扫字段 |
| `SHOW CREATE TABLE 表名\G;` | **完整建表语句**（含引擎、字符集、索引定义） | 要看全貌 / 复制结构 / 排障 |

`SHOW CREATE TABLE` 前面已经用过，它输出的就是 2.2 开头那段骨架，只是被规范化了：表名和列名自动加**反引号**，类型按存储格式显示（`INT` 显示成 `int(11)`）。

> 引号规矩：**单引号 `'...'` 是字符串数据，反引号 `` `...` `` 才是表名/列名。** 写成 `CREATE TABLE 'user'` 直接报 `ERROR 1064`。

---

### 2.3 数据类型（对应 08 集，本集最长，坑最多）

#### 数值类型

| 类型 | 字节 | 范围 | 实测 |
| --- | --- | --- | --- |
| `TINYINT` | 1 | -128 ~ 127 | 插入 127 成功；插入 128 → `ERROR 1264 Out of range` |
| `SMALLINT` | 2 | -32768 ~ 32767 | |
| `MEDIUMINT` | 3 | 约 -838万 ~ 838万 | |
| `INT` | 4 | -2147483648 ~ 2147483647 | 插入 2147483647 成功；再 +1 → `ERROR 1264` |
| `BIGINT` | 8 | 约 ±9.22 × 10^18 | |
| `FLOAT` | 4 | 约 7 位有效数字 | 精度会丢 |
| `DOUBLE` | 8 | 约 15 位有效数字 | 精度会丢 |
| `DECIMAL(M,D)` | 变长 | 精确 | M 是总位数，D 是小数位 |

**⭐ 坑一：`INT(11)` 里的 11 不是"长度限制"**

这是全网最大的误解。实测证明：

```sql
CREATE TABLE t_w (a INT(1));
INSERT INTO t_w VALUES (123456789);   -- 9 位数，插进去了！
SELECT a, LENGTH(a) FROM t_w;         -- 123456789, 9
```

`INT(1)` 都能存 9 位数，说明括号里的数字**只影响显示宽度**（早期命令行客户端会给整数前面补空格对齐用），跟能存多大毫无关系。**真正限制范围的是类型本身（INT 就 4 字节）。**

MySQL 8.0 起这个显示宽度已被弃用，`SHOW CREATE TABLE` 里直接显示成 `int`；MariaDB 仍保留 `int(11)` 的写法。

**DECIMAL 的溢出**：

```sql
CREATE TABLE t_dec (p DECIMAL(5,2));
INSERT INTO t_dec VALUES (999.99);    -- 成功：5 位里 3 位整数 + 2 位小数
INSERT INTO t_dec VALUES (1000.00);   -- ERROR 1264：整数部分超了 3 位
```

**⭐ 坑二：钱不能用 FLOAT/DOUBLE**

实测三种类型各插 0.1 和 0.2 再求和：

| 类型              | `SUM` 结果              |
| --------------- | --------------------- |
| `FLOAT`         | `0.30000000447034836` |
| `DOUBLE`        | `0.30000000000000004` |
| `DECIMAL(10,2)` | `0.30` ✅              |

浮点二进制无法精确表示 0.1，累加就漂了。**金额、计数一律用 DECIMAL。**

#### 字符串类型

| 类型 | 长度 | 特点 |
| --- | --- | --- |
| `CHAR(n)` | 固定 n 个字符，最多 255 | 定长，不足补空格；适合固定长度（性别、状态码） |
| `VARCHAR(n)` | 最多 n 个字符，上限 65535 **字节** | 变长，省空间；最常用 |
| `TEXT` | 最多 65535 字节 | 长文本，不能设默认值 |
| `BLOB` | 二进制 | 存图片、文件 |

**CHAR 和 VARCHAR 的真实差别**（实测）：

```sql
CREATE TABLE t_cv (c CHAR(5), v VARCHAR(5));
INSERT INTO t_cv VALUES ('ab', 'ab');
SELECT c, v, LENGTH(c), LENGTH(v) FROM t_cv;
-- 两者 LENGTH 都是 2
```

看起来一样？因为 CHAR 尾部补的空格在**取出来时被去掉了**。差别藏在别处：

- **存储**：CHAR(5) 存 'ab' 内部固定占 5 个字符位；VARCHAR(5) 只占 2 个字符 + 1 个长度字节
- **超长**：插 7 个字符时，先撞上 CHAR(5) → `ERROR 1406 Data too long for column 'c'`

注意 `VARCHAR(5)` 的 5 是**字符数**，但整体上限是 **字节**（65535 字节）。utf8mb4 下一个汉字占 3 字节，`VARCHAR(100)` 存 100 个汉字要 300 字节，够用；但 `VARCHAR(21845)` 这类凑字节上限的写法很容易踩到行大小限制。

#### 日期时间类型

| 类型 | 格式 | 范围 | 特点 |
| --- | --- | --- | --- |
| `DATE` | YYYY-MM-DD | 1000-01-01 ~ 9999-12-31 | 只有日期 |
| `TIME` | HH:MM:SS | | 只有时间 |
| `DATETIME` | YYYY-MM-DD HH:MM:SS | 1000 ~ 9999 年 | **存什么读什么**，不受时区影响 |
| `TIMESTAMP` | 同上 | 1970 ~ 2038（传统） | **会随时区转换**，4 字节 |
| `YEAR` | YYYY | 1901 ~ 2155 | 只存年份 |

**⭐ 坑三：TIMESTAMP 会跟着时区变，DATETIME 不会**

实测（本机时区 UTC+8）：两列各存同一个时刻 `2038-01-20 00:00:00`，然后把会话时区切成 UTC 再读：

```sql
SET time_zone = '+00:00';
SELECT t, d FROM t_ts;
```

| TIMESTAMP | DATETIME |
| --- | --- |
| `2038-01-19 16:00:00` | `2038-01-20 00:00:00` |

TIMESTAMP 往前退了 8 小时（按 UTC 显示），DATETIME 原样不动。因为 TIMESTAMP 存的是 UTC 时间戳，读的时候按当前会话时区换算。

由此还引出一个坑：**UTC+8 下，TIMESTAMP 的最小值不是 `1970-01-01 00:00:00`**，写它会报 `ERROR 1292 Incorrect datetime value`，得写 `1970-01-01 08:00:01`（换算成 UTC 才落在范围内）。

> 2038 问题：传统上限是 `2038-01-19 03:14:07 UTC`。本机 MariaDB 13 实测**已能插 2038-01-20**，而 MySQL 8.0 仍受 2038 限制。跨版本以你实际环境为准，业务上要存远期日期就用 DATETIME。

**默认值常用写法**：

```sql
created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

第二行意思是：插入时自动填当前时间，**每次更新这一行时自动改成新时间**，不用程序手动维护。

---

### 2.4 表操作：修改与删除（对应 09 集）

#### ALTER TABLE 五种改法

| 想干什么 | 语句 |
| --- | --- |
| 加字段 | `ALTER TABLE t ADD COLUMN c INT;` |
| 改类型/约束（**不改名**） | `ALTER TABLE t MODIFY COLUMN c BIGINT;` |
| 改名 + 改类型 | `ALTER TABLE t CHANGE COLUMN 老名 新名 BIGINT;` |
| 只改名（不写类型） | `ALTER TABLE t RENAME COLUMN 老名 TO 新名;` |
| 删字段 | `ALTER TABLE t DROP COLUMN c;` |

`MODIFY` 和 `CHANGE` 的区别就一句话：**CHANGE 多一个"新名字"参数，顺便改名；MODIFY 只能改类型和约束。**

**⭐ 坑四：MODIFY 时忘了写 NOT NULL，约束会丢**

实测：

```sql
CREATE TABLE t_m (a INT NOT NULL);
ALTER TABLE t_m MODIFY COLUMN a BIGINT;   -- 只写了类型，没写 NOT NULL
DESC t_m;
```

结果 `a` 的 Null 列从 `NO` 变成了 **`YES`** —— 原本的非空约束被悄悄抹掉了。

原因：`MODIFY` 是**整列重新定义**，你写什么它就长什么样，没写的约束等于没要求。所以改类型时要把原约束**照抄一遍**：

```sql
ALTER TABLE t_m MODIFY COLUMN a BIGINT NOT NULL;
```

> 建议流程：改之前先 `SHOW CREATE TABLE t_m\G` 抄一份，改完再 `DESC` 对比一次。

#### 表改名与删表

```sql
ALTER TABLE 老表名 RENAME TO 新表名;   -- 改表名
RENAME TABLE 老表名 TO 新表名;         -- 另一种写法
DROP TABLE [IF EXISTS] 表名;           -- 删表（结构+数据一起没）
```

#### 往有数据的表加 NOT NULL 列

实测：表里已有两行，加一个 `INT NOT NULL` 且没给默认值的列——

```sql
ALTER TABLE t_add ADD COLUMN c INT NOT NULL;
```

本机 MariaDB 13 **不报错**，已有行自动填了 `0`。但这个行为跟严格模式、版本有关，**靠不住**。规范做法是加默认值：

```sql
ALTER TABLE t_add ADD COLUMN c INT NOT NULL DEFAULT 0;
```

---

### 2.5 DDL 小结（对应 10 集）

#### TRUNCATE / DELETE / DROP 三兄弟

| | `DELETE FROM t` | `TRUNCATE TABLE t` | `DROP TABLE t` |
| --- | --- | --- | --- |
| 属于 | DML | **DDL** | **DDL** |
| 删什么 | 行（可带条件），可只删一部分 | **所有行**，结构留着 | **整张表**（结构 + 数据） |
| 自增值 | **不重置** | **重置回 1** | 表都没了 |
| 能否回滚 | ✅ 能 | ❌ 不能 | ❌ 不能 |
| 速度 | 慢（逐行 + 写日志） | 快 | 快 |

自增值实测：

```sql
-- 建表，插 a,b,c（id = 1,2,3），然后 DELETE FROM t_tr（清空）
INSERT INTO t_tr (n) VALUES ('d');
SELECT id FROM t_tr;      -- 4   ← 自增不回退

-- 换成 TRUNCATE TABLE t_tr 后再插
INSERT INTO t_tr (n) VALUES ('e');
SELECT id FROM t_tr;      -- 1   ← 自增被重置
```

回滚实测：

| 操作 | 回滚后剩几行 |
| --- | --- |
| `DELETE` + `ROLLBACK` | **2 行**（数据回来了） |
| `TRUNCATE` + `ROLLBACK` | **0 行**（数据回不来） |

#### ⭐ 坑五：DDL 是隐式提交，回滚不了

上面 TRUNCATE 回不来，根源是：**每条 DDL 语句执行时会自动提交事务**。你写 `START TRANSACTION` 根本不覆盖它，`ROLLBACK` 无从下手。

所以 DDL 操作要格外小心：

- 生产环境 `DROP DATABASE` 前先备份
- `ALTER TABLE` 之前先 `SHOW CREATE TABLE` 抄一份
- 拿不准的语句，先在测试库跑

---

## 3. 常见坑汇总

| # | 坑 | 现象 | 正确做法 |
| --- | --- | --- | --- |
| 1 | 用单引号包表名/列名 | `ERROR 1064`，报错位置指着 `'user'` | 用反引号 `` ` ``，或干脆不加 |
| 2 | 最后一个字段后多逗号 | `ERROR 1064 ... near ')'` | 最后一列不带逗号 |
| 3 | `AUTO_INCREMENT` 没建在键上 | `ERROR 1075 it must be defined as a key` | 加 `PRIMARY KEY (列)` |
| 4 | NOT NULL 列插入时省略 | `ERROR 1364 doesn't have a default value` | 给值，或加 `DEFAULT` |
| 5 | NOT NULL 列显式写 NULL | `ERROR 1048 cannot be null` | 给非空值 |
| 6 | 以为 `INT(11)` 限制长度 | `INT(1)` 照样能存 9 位数 | 范围看类型，不看括号 |
| 7 | 用 FLOAT 存金额 | `0.1 + 0.2 = 0.3000000044...` | 用 `DECIMAL` |
| 8 | `DECIMAL(5,2)` 存 1000 | `ERROR 1264 Out of range` | 算好整数位，或放大 M |
| 9 | `MODIFY` 忘写 NOT NULL | `DESC` 里 Null 变 `YES` | 把原约束照抄一遍 |
| 10 | TIMESTAMP 跨时区 | 读出来时间变了 | 存绝对时间用 DATETIME |
| 11 | 在 UTC+8 下插 1970-01-01 00:00:00 | `ERROR 1292 Incorrect datetime value` | 用 `1970-01-01 08:00:01` |
| 12 | 以为 TRUNCATE 能回滚 | 回滚后表空 | DDL 隐式提交，操作前备份 |
| 13 | `IF NOT EXISTS` 以为能改字符集 | 库已存在时设置不生效 | 用 `ALTER DATABASE` |

---

## 4. 什么时候用 / 不用

| 该用 | 不该用 |
| --- | --- |
| 项目初始化建库建表 | 用 DDL 改**数据**（那是 DML 的活） |
| 加约束挡脏数据（NOT NULL、UNIQUE） | 给大表频繁 ALTER（会锁表、耗时长） |
| 结构变更（加字段、改类型） | 生产库直接 `DROP`，先备份 |
| `CREATE TABLE IF NOT EXISTS` 写初始化脚本 | 每张表都设一堆索引（写性能会掉） |

---

## 5. 自测

- [x] 用 `CREATE DATABASE IF NOT EXISTS` 连执行三次，确认不报错
- [x] 建一张表，同时包含 `NOT NULL`、`DEFAULT`、`AUTO_INCREMENT`、`UNIQUE`、`COMMENT`
- [x] 用 `INT(1)` 存一个 9 位数，说出为什么能成功
- [x] 用 `FLOAT` 和 `DECIMAL` 各算一次 0.1 + 0.2，对比结果
- [x] 把一列 `NOT NULL` 用 `MODIFY` 改成 BIGINT 但不写 NOT NULL，用 `DESC` 验证约束是否丢失
- [x] `DELETE` 和 `TRUNCATE` 各清空一次表，观察下一次插入的 id 是几
- [x] 把会话时区改成 `+00:00`，观察 `TIMESTAMP` 和 `DATETIME` 列读出来的差异
- [x] 故意写一段带单引号表名的 CREATE TABLE，看报错位置指向哪里

---

## 附录：本机实测环境的版本差异

本笔记的实测环境是 **MariaDB 13.0.2（Arch Linux）**。和 MySQL 8.0 的差异：

| 项目 | MariaDB 13（本机实测） | MySQL 8.0 |
| --- | --- | --- |
| `INT` 显示 | `int(11)` | `int`（显示宽度已弃用） |
| 默认排序规则 | 取决于建库指定，实测用的 `utf8mb4_unicode_ci` | `utf8mb4_0900_ai_ci` |
| `TIMESTAMP` 上限 | 实测已可存 2038-01-20 | `2038-01-19 03:14:07 UTC` |
| `RENAME COLUMN` | 支持 | 支持（8.0 起） |

遇到输出不一致时，先看版本：`SELECT VERSION();`
