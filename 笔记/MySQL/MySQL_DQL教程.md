---
tags: [MySQL, 数据库, SQL, DQL]
---

# MySQL DQL 教程（查询数据）

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"
> 覆盖：15. 基础-SQL-DQL-基础查询 ~ 23. 基础-SQL-DQL小结（共 9 集）
> 所有示例和输出都是在本机 **MariaDB 13.0.2** 上实测跑出来的

> 配套：用的 `emp` 表在 [MySQL DML 教程](MySQL_DML教程.md)。本笔记在 10 条员工数据的基础上，额外插了一条 `name = NULL` 的记录（共 11 行），用来演示 NULL 相关行为。

---

## 0. 一句话认识

**DQL 就是 `SELECT`：按你的条件从表里捞出想要的行和列，还能算数、分组、排序、分页。**

DQL 是四类 SQL 里**用得最多**的一种。业务代码里十条 SQL 有九条是 `SELECT`。

先记住这张执行顺序图（22 集会细讲）：

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

注意：`SELECT` 排在第 5 位，不是第一位。**这就是为什么有的地方能用别名、有的地方不能用**（见 2.7）。

| 需求 | 用什么 |
| --- | --- |
| 看看有哪些行 | `WHERE` 条件 |
| 求和 / 计数 / 平均 | 聚合函数 `COUNT/SUM/AVG/MAX/MIN` |
| 按类别统计 | `GROUP BY` |
| 排个序 | `ORDER BY` |
| 只看第几页 | `LIMIT` |

---

## 1. 为什么需要它

表里的数据是给人用的，但代码要的是"**符合某个条件的那几行**"：

- 登录：按用户名查一条
- 商品列表：按分类、按价格区间查，还要分页
- 后台统计：按部门算平均工资、按地区数人数

这些全是 DQL 的活。**条件写错 = 查出来的数据全错**，所以 `WHERE` 部分是重点中的重点。

---

## 2. 核心用法

### 2.1 基础查询：捞行和列

```sql
SELECT 列1, 列2 FROM 表;      -- 查指定列
SELECT * FROM 表;             -- 查所有列（* 是通配）
```

**起别名**：让查询结果的表头更好看、更好读。

```sql
SELECT name AS 姓名, age AS 年龄 FROM emp LIMIT 5;
```

| 姓名 | 年龄 |
| --- | --- |
| 柳岩 | 20 |
| 张无忌 | 18 |
| 韦小宝 | 38 |
| 赵敏 | 32 |
| 宋青书 | 18 |

`AS` 可以省略（写成 `SELECT name 姓名`），但**建议保留**，读起来清楚。

**去重**：`DISTINCT`。

```sql
SELECT DISTINCT workaddress FROM emp;
```

| workaddress |
| --- |
| 北京 |
| 上海 |
| NULL |

⚠️ 注意 NULL 也被当成一个独立的值列出来了。`DISTINCT` 去重**只保留一个 NULL**，它不会把 NULL 当异常。

### 2.2 条件查询：WHERE

语法：`SELECT ... FROM 表 WHERE 条件;`

#### 比较运算符

| 运算符 | 意思 | 例子 |
| --- | --- | --- |
| `=` | 等于（不是 `==`） | `WHERE age = 18` |
| `>` `<` `>=` `<=` | 大小 | `WHERE age >= 30` |
| `!=` 或 `<>` | 不等于 | `WHERE age != 18` |

实测：

```sql
SELECT name, age FROM emp WHERE age > 30;
```

| name | age |
| --- | --- |
| 韦小宝 | 38 |
| 赵敏 | 32 |
| 刘德华 | 39 |
| 张三丰 | 40 |

#### 逻辑运算符

| 运算符 | 意思 |
| --- | --- |
| `AND` / `&&` | 都满足 |
| `OR` / `\|\|` | 满足一个就行 |
| `NOT` / `!` | 取反 |

```sql
SELECT name FROM emp WHERE age > 30 AND workaddress = '北京';
SELECT name FROM emp WHERE age = 18 OR age = 40;   -- 18 岁或 40 岁
SELECT name FROM emp WHERE NOT (age = 18);          -- 不是 18 岁
```

> ⚠️ `AND` 的优先级高于 `OR`。写 `WHERE a = 1 OR b = 2 AND c = 3` 时，实际是 `a = 1 OR (b = 2 AND c = 3)`。拿不准就**加括号**。

#### 范围：BETWEEN

```sql
SELECT name, age FROM emp WHERE age BETWEEN 18 AND 20;
```

BETWEEN 是**闭区间**，等于 `age >= 18 AND age <= 20`。

#### 多选一：IN

```sql
SELECT name, workaddress FROM emp WHERE workaddress IN ('北京','上海');
```

`IN` 适合"可能是这几个值之一"的场面，比一长串 `OR` 干净。反向用 `NOT IN`。

#### 模糊匹配：LIKE

两个通配符：

| 符号 | 意思 |
| --- | --- |
| `%` | 任意**多个**字符（包括 0 个） |
| `_` | 任意**一个**字符 |

实测：

```sql
SELECT name FROM emp WHERE name LIKE '张%';    -- 以张开头的
```

| name |
| --- |
| 张无忌 |
| 张三丰 |
| 张若昀 |

```sql
SELECT name FROM emp WHERE name LIKE '张_';    -- 张 + 恰好一个字符（即两个字）
```

实测结果是 **0 行**——因为这三人都是"张 + 两个字"（三字名）。这个空白最有教育意义：`_` 卡的是**字数**，不是"大概匹配"。

#### ⭐ NULL 的判断：只能用 IS

这是 DQL 里最反直觉的一条，实测：

```sql
SELECT id, name FROM emp WHERE name IS NULL;    -- ✅ 查到 1 行
SELECT COUNT(*) FROM emp WHERE name = NULL;     -- ❌ 返回 0
```

| 写法 | 结果 |
| --- | --- |
| `WHERE name IS NULL` | 1 行（真有 NULL 的那条） |
| `WHERE name = NULL` | **0 行** |

原因：NULL 表示"未知"，**任何值和未知比较，结果还是未知**，不是 true。所以 `= NULL` 永远匹配不上任何东西。判断 NULL 只能用 `IS NULL` / `IS NOT NULL`。

### 2.3 聚合函数：把多行压成一个数

| 函数 | 作用 | 例子 |
| --- | --- | --- |
| `COUNT()` | 计数 | `COUNT(*)` 总行数 |
| `SUM()` | 求和 | `SUM(age)` |
| `AVG()` | 平均 | `AVG(age)` |
| `MAX()` | 最大 | `MAX(age)` |
| `MIN()` | 最小 | `MIN(age)` |

实测一条语句全集齐：

```sql
SELECT COUNT(*) AS 总人数, COUNT(name) AS 姓名非空,
       MAX(age) AS 最大年龄, MIN(age) AS 最小年龄,
       ROUND(AVG(age),1) AS 平均年龄, SUM(age) AS 年龄总和
FROM emp;
```

| 总人数 | 姓名非空 | 最大年龄 | 最小年龄 | 平均年龄 | 年龄总和 |
| --- | --- | --- | --- | --- | --- |
| 11 | 10 | 40 | 18 | 27.9 | 307 |

**⭐ `COUNT(*)` 和 `COUNT(列)` 不一样**：

- `COUNT(*)` = 11：**行数**
- `COUNT(name)` = 10：**该列有值的行数**，那条 NULL 被跳过了

这是 NULL 第二次咬人。聚合时 NULL 不会参与计算（不计数、不求和、不进平均），所以**用 `AVG(age)` 时要清楚：分母是"age 有值的行数"，不是总行数"**。

### 2.4 分组查询：GROUP BY

把同类行归一组，对每组分别算聚合值。

```sql
SELECT gender, COUNT(*) AS 人数, ROUND(AVG(age),1) AS 平均年龄
FROM emp GROUP BY gender;
```

| gender | 人数 | 平均年龄 |
| --- | --- | --- |
| NULL | 1 | 25.0 |
| 女 | 3 | 27.0 |
| 男 | 7 | 28.7 |

注意那条 gender 为 NULL 的也被单独归成了一组。**NULL 可以作为一个分组依据**，这一点和"去重"时一样。

#### HAVING：对分组结果再过滤

```sql
SELECT workaddress, COUNT(*) AS 人数
FROM emp
GROUP BY workaddress
HAVING COUNT(*) >= 5;
```

| workaddress | 人数 |
| --- | --- |
| 北京 | 6 |

#### ⭐ WHERE 和 HAVING 的区别

| | WHERE | HAVING |
| --- | --- | --- |
| 作用对象 | **行**（分组前） | **组**（分组后） |
| 能写聚合函数吗 | ❌ 不能 | ✅ 可以 |
| 位置 | `GROUP BY` 之前 | `GROUP BY` 之后 |

实测对比：

```sql
-- WHERE：先把 age>25 的行挑出来，再分组
SELECT gender, COUNT(*) FROM emp WHERE age > 25 GROUP BY gender;
```

| gender | 人数 |
| --- | --- |
| 女 | 2 |
| 男 | 4 |

那条 NULL 的记录（age=25）被 WHERE 挡掉了，所以没有 NULL 组了。

**口诀**：**WHERE 筛行，HAVING 筛组。** 能写成 WHERE 的，别写成 HAVING（快）。

### 2.5 排序查询：ORDER BY

```sql
SELECT name, age FROM emp ORDER BY age DESC LIMIT 3;
```

| name | age |
| --- | --- |
| 张三丰 | 40 |
| 刘德华 | 39 |
| 韦小宝 | 38 |

`ASC` 升序（默认），`DESC` 降序。

**多字段排序**（第一个字段相同了就看第二个）：

```sql
SELECT name, age, entrydate FROM emp
ORDER BY age ASC, entrydate DESC LIMIT 4;
```

| name | age | entrydate |
| --- | --- | --- |
| 张无忌 | 18 | 2005-09-01 |
| 宋青书 | 18 | 2005-01-01 |
| 李靖 | 18 | 2005-01-01 |
| 柳岩 | 20 | 2000-01-01 |

三个 18 岁的，entrydate 降序先出 2005-09-01 的张无忌，后面两个同样是 2005-01-01 就并列了。

> 多个字段排序时，**每个字段都可以单独写 ASC/DESC**，顺序决定优先级。

### 2.6 分页查询：LIMIT

```sql
SELECT id, name FROM emp ORDER BY id LIMIT 0, 3;   -- 第 1 页：跳过 0 条，取 3 条
```

| id | name |
| --- | --- |
| 1 | 柳岩 |
| 2 | 张无忌 |
| 3 | 韦小宝 |

```sql
LIMIT 3, 3;                                        -- 第 2 页：跳过 3 条，取 3 条
```

| id | name |
| --- | --- |
| 4 | 赵敏 |
| 5 | 宋青书 |
| 6 | 刘德华 |

语法两种写法，意思一样：

| 写法 | 含义 |
| --- | --- |
| `LIMIT 偏移量, 每页条数` | MySQL 常见写法 |
| `LIMIT 每页条数 OFFSET 偏移量` | 标准 SQL 写法 |
| `LIMIT 3` | 只写一个数 = 取前 3 条（偏移量默认 0） |

**分页公式**：页码从 1 开始

```
偏移量 = (页码 - 1) × 每页条数
```

第 1 页 `LIMIT 0, 5`，第 2 页 `LIMIT 5, 5`，第 3 页 `LIMIT 10, 5`。

> ⚠️ **分页必须配 ORDER BY**，否则每次查出来的顺序可能不一样，翻页会看到重复或漏掉的行。

### 2.7 执行顺序（22 集重点）

SQL 不是从左到右执行的，真实顺序是：

```
1. FROM        —— 先知道查哪张表
2. WHERE       —— 筛行
3. GROUP BY    —— 分组
4. HAVING      —— 筛组
5. SELECT      —— 选列、起别名、算聚合
6. ORDER BY    —— 排序
7. LIMIT       —— 分页取条数
```

**这个顺序解释了三条反直觉现象：**

**① WHERE 里不能用 SELECT 的别名**——因为 WHERE 在第 2 步跑，SELECT 的别名第 5 步才产生，那时还没有。

```sql
SELECT name AS 姓名, age AS 年龄 FROM emp WHERE 年龄 > 30;
-- ERROR 1054 (42S22): Unknown column '年龄' in 'WHERE'
```

**② ORDER BY 里可以用别名**——因为 ORDER BY 在第 6 步，在 SELECT 后面。

```sql
SELECT name AS 姓名, age AS 年龄 FROM emp ORDER BY 年龄 DESC LIMIT 3;
-- ✅ 正常运行
```

**③ 没有 GROUP BY 也能写 HAVING**，但**结果不可靠**。

实测：

```sql
SELECT gender, COUNT(*) FROM emp HAVING COUNT(*) > 5;
```

返回值是 `女  11`——不报错，但 gender 取了哪一行完全是碰运气（它随便拿了一行的值来凑数）。

原因：没有 GROUP BY 时整张表被当成"一个组"，SELECT 里那个非聚合的 `gender` 就没有明确归属。**这种写法要靠数据库的宽容度，而不是靠规则。** 换成开了 `ONLY_FULL_GROUP_BY` 的环境直接报错。

> 规矩：**SELECT 里出现非聚合列，就必须跟 GROUP BY**。没分组就别写聚合以外的列。

### 2.8 综合案例（21 集）

把前面几个合起来，这是实际开发最常写的形态：

```sql
-- 统计每个工作地点的 30 岁以下人数、平均年龄，按人数降序，只看前 2 名
SELECT workaddress,
       COUNT(*) AS 人数,
       ROUND(AVG(age),1) AS 平均年龄
FROM emp
WHERE age < 30
GROUP BY workaddress
HAVING COUNT(*) >= 2
ORDER BY 人数 DESC
LIMIT 2;
```

一条语句覆盖了 WHERE → GROUP BY → HAVING → 别名 → ORDER BY → LIMIT 六个环节。

---

## 3. 常见坑汇总

| # | 坑 | 现象 | 正确做法 |
| --- | --- | --- | --- |
| 1 | `WHERE 列 = NULL` | 永远 0 行 | 用 `IS NULL` |
| 2 | `COUNT(*)` 拿来做"某列有值数" | 多算了 NULL 行 | 用 `COUNT(列名)` |
| 3 | WHERE 里用 SELECT 别名 | `ERROR 1054 Unknown column` | 别名只在 ORDER BY 能用 |
| 4 | SELECT 有非聚合列但没 GROUP BY | 结果靠运气/报错 | 加上 GROUP BY |
| 5 | 分页不排序 | 翻页重复或漏行 | `ORDER BY` + `LIMIT` |
| 6 | `LIKE '张_'` 以为能匹配三个字 | 0 行 | `_` 只匹配一个字符 |
| 7 | AND / OR 混写不加括号 | 语义变了 | 加括号 |
| 8 | `BETWEEN` 以为不含边界 | 其实闭区间 | 知道就行 |
| 9 | 模糊查询用 `=` | 查不出 | 用 `LIKE` |

---

## 4. 什么时候用 / 不用

| 该用 | 不该用 |
| --- | --- |
| 列表页、详情页、统计报表 | 用 `SELECT *` 在生产代码里（列多了浪费带宽，加字段还可能出 bug） |
| 条件筛选、分页 | 在数据库里做复杂业务逻辑（该放程序里） |
| 分组统计 | 大表上不加索引的模糊查询（会全表扫描） |

---

## 5. 自测

- [ ] 查出所有 30 岁以上、北京的人，写出完整语句
- [ ] 用 `LIKE` 查"张"姓三字名的人
- [ ] 查 `name` 为 NULL 的记录，再用 `= NULL` 试一次，解释差异
- [ ] 统计男女各多少人、平均年龄
- [ ] 统计每个工作地点人数，只要 ≥5 人的
- [ ] 按年龄降序取前 3 名
- [ ] 实现第 2 页、每页 3 条的分页查询
- [ ] 写出 `SELECT name AS 姓名 ... WHERE 姓名 > ...` 看是否报错，解释为什么
