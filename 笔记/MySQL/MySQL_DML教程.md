---
tags: [MySQL, 数据库, SQL, DML]
---

# MySQL DML 教程（增删改数据）

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"
> 覆盖：12. 基础-SQL-DML-插入 ~ 14. 基础-SQL-DML小结（共 3 集）
> 所有示例和输出都是在本机 **MariaDB 13.0.2** 上实测跑出来的

> 配套：表结构由 [MySQL DDL 教程](MySQL_DDL教程.md) 建的 `emp` 表，本笔记的示例都在这张表上跑。

---

## 0. 一句话认识

**DML（Data Manipulation Language）管"**表里的数据**"：插进去、改一改、删掉。它不碰表的结构。**

| | DDL（上一课） | DML（本课） |
| --- | --- | --- |
| 改什么 | 库、表、字段、约束 | 表里的**行** |
| 典型语句 | `CREATE` / `ALTER` / `DROP` / `TRUNCATE` | `INSERT` / `UPDATE` / `DELETE` |
| 能否回滚 | ❌ 隐式提交 | ✅ 在事务里可以 |

| 操作 | 关键词 | 一句话 |
| --- | --- | --- |
| 增 | `INSERT` | 往表里塞行 |
| 改 | `UPDATE` | 把某些行的某些列改掉 |
| 删 | `DELETE` | 把某些行抹掉 |

---

## 1. 为什么需要它

DDL 只把"空壳子"建好了，表里一行数据都没有。程序跑起来以后：

- 用户注册 → `INSERT` 一条
- 用户改昵称 → `UPDATE` 那一行
- 用户注销 → `DELETE` 那一行

**这三件事每天发生几万次，而 DDL 一辈子就执行几次。** 所以 DML 才是业务代码里出现最多的 SQL。

---

## 2. 核心用法

本次实测用的表（沿用 DDL 课的结构，简化版）：

```sql
CREATE TABLE emp (
  id          INT            COMMENT '编号',
  workno      VARCHAR(10)    COMMENT '工号',
  name        VARCHAR(10)    COMMENT '姓名',
  gender      CHAR(1)        COMMENT '性别',
  age         TINYINT UNSIGNED COMMENT '年龄',
  idcard      CHAR(18)       COMMENT '身份证号',
  workaddress VARCHAR(50)    COMMENT '工作地址',
  entrydate   DATE           COMMENT '入职时间'
) COMMENT '员工表';
```

### 2.1 INSERT：三种写法

**写法一：指定字段插入（推荐）**

```sql
INSERT INTO emp (id, name, age) VALUES (1, '甲', 20);
```

只给三个字段，其余字段留 NULL。好处是**不怕表结构变了**——以后加字段也不影响这条语句。

**写法二：全字段插入**

```sql
INSERT INTO emp VALUES (2, '2', '乙', '男', 25, 'x', '北京', '2020-01-01');
```

不给字段列表，就**必须按建表时字段的顺序，一个不少地给全**。少一个就报错：

```
ERROR 1136 (21S01): Column count doesn't match value count at row 1
```

**写法三：批量插入（推荐）**

```sql
INSERT INTO emp (id, name, age) VALUES (3, '丙', 30), (4, '丁', 40);
```

用逗号隔开多组值，一条语句插多行。比写四条快得多（少了三次网络往返）。

实测三种写法混合跑，结果：

| id | name | age |
| --- | --- | --- |
| 1 | 甲 | 20 |
| 2 | 乙 | 25 |
| 3 | 丙 | 30 |
| 4 | 丁 | 40 |

### 2.2 INSERT 踩坑清单

| 坑 | 现象 | 原因 |
| --- | --- | --- |
| 字段数对不上 | `ERROR 1136 Column count doesn't match` | 全字段插入时值少了/多了 |
| 类型超范围 | `ERROR 1264 Out of range` | 实测：`TINYINT UNSIGNED` 列插 `-1` 直接报错 |
| 撞唯一键 | `ERROR 1062 Duplicate entry` | 主键/唯一列的值重复了 |
| 挨必填列不赐值 | `ERROR 1364 doesn't have a default value` | 该列 NOT NULL 且没 DEFAULT（见 DDL 教程） |

> 实测：往 `TINYINT UNSIGNED` 的 age 列插 `-1` →
> `ERROR 1264 (22003): Out of range value for column 'age' at row 1`
> 因为 UNSIGNED 把范围改成了 0~255，负数越界。

### 2.3 UPDATE：改数据

语法：

```sql
UPDATE 表名 SET 字段1 = 值1, 字段2 = 值2, ... [WHERE 条件];
```

**带条件改一行**：

```sql
UPDATE emp SET age = 26 WHERE id = 2;
```

实测结果：id=2 那行的 age 从 25 变成 26，其它行不动。

**⭐ 不写 WHERE = 改全表**

```sql
UPDATE emp SET workaddress = '深圳';
```

实测：这张表**每一行**的 workaddress 都被改成了深圳。

| id | name | workaddress |
| --- | --- | --- |
| 1 | 甲 | 深圳 |
| 2 | 乙 | 深圳 |
| 3 | 丙 | 深圳 |
| 4 | 丁 | 深圳 |

> 这不是 bug，是 SQL 的既定行为：**没有 WHERE 就是"全部行"**。所以线上执行 UPDATE 前必须检查 WHERE 条件。老手的口诀是"先 SELECT 一遍确认范围，再改成 UPDATE"。

**一次改多个字段**：

```sql
UPDATE emp SET name = '张三', age = 30, workaddress = '北京' WHERE id = 1;
```

### 2.4 DELETE：删数据

语法：

```sql
DELETE FROM 表名 [WHERE 条件];
```

```sql
DELETE FROM emp WHERE id = 3;   -- 只删 id=3 那行
SELECT id, name FROM emp;       -- 实测：3 没了，其他都在
```

**⭐ 同样：不写 WHERE = 清空全表**

```sql
DELETE FROM emp;                -- 全删，表结构还在
```

### 2.5 小结：DELETE / TRUNCATE / DROP 别再混

这是 12~14 集最容易混的三个。都在 DDL 课里实测过，这里汇总：

| | `DELETE FROM t` | `TRUNCATE TABLE t` | `DROP TABLE t` |
| --- | --- | --- | --- |
| 属于 | **DML** | DDL | DDL |
| 删什么 | 行，可带条件 | 所有行，结构留着 | 整张表（结构+数据） |
| 自增值 | 不重置 | **重置回 1** | 表没了 |
| 能否回滚 | ✅ 能 | ❌ 不能 | ❌ 不能 |
| 速度 | 慢 | 快 | 快 |

实测自增对比（DDL 课那次）：

- `DELETE` 清空后再插入 → id = **4**（接着往下发号）
- `TRUNCATE` 清空后再插入 → id = **1**（从头开始）

**一句话记**：`DELETE` 是"擦掉字"，`TRUNCATE` 是"换张新纸"，`DROP` 是"把本子撕了"。

---

## 3. 常见坑汇总

| # | 坑 | 现象 | 正确做法 |
| --- | --- | --- | --- |
| 1 | 全字段 INSERT 少给值 | `ERROR 1136` | 用指定字段写法 |
| 2 | UPDATE / DELETE 忘写 WHERE | 全表被改 / 全表被删 | 先 `SELECT` 验证范围，再执行 |
| 3 | `WHERE id = 1 = 1` 之类 | 条件恒真，等于没写 | 条件写清楚 |
| 4 | 拿 `DELETE` 当清表工具 | 自增不重置，id 有空洞 | 要重置自增用 `TRUNCATE` |
| 5 | 以为 `DELETE` 删不掉 | 是被外键挡住了（`ERROR 1451`） | 先处理子表数据（见约束教程） |
| 6 | 批量插入超长 | 大事务卡住 | 分批提交 |

---

## 4. 什么时候用 / 不用

| 该用 | 不该用 |
| --- | --- |
| 用户注册、下单、评论 | 用 UPDATE 当"批量导入"工具（那该用 INSERT） |
| 改资料、改库存 | 不带 WHERE 的 UPDATE / DELETE 上生产 |
| 删过期数据 | 用 `DELETE FROM t` 清空几十万行（慢，用 TRUNCATE） |

---

## 5. 自测

- [ ] 用三种写法各插一行，说出全字段写法为什么容易出错
- [ ] 写一条只改一行的 UPDATE，再用 `SELECT` 验证
- [ ] 故意写一条不带 WHERE 的 UPDATE（在测试表上），观察结果
- [ ] `DELETE` 和 `TRUNCATE` 各清空一次，比较下一次插入的 id
- [ ] 往 `TINYINT UNSIGNED` 列插 -1，把报错原文抄下来
