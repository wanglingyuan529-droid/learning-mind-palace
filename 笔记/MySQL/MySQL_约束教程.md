---
tags: [MySQL, 数据库, SQL, 约束]
---

# MySQL 约束教程

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"
> 覆盖：32. 基础-约束-概述 ~ 36. 基础-约束-小结（共 5 集）
> 所有示例和输出都是在本机 **MariaDB 13.0.2** 上实测跑出来的

---

## 0. 一句话认识

**约束是写在表结构上的"规矩"，用来拦住不合法的数据。它管的是"什么值能进这张表"。**

| 约束 | 关键字 | 拦住什么 |
| --- | --- | --- |
| 非空 | `NOT NULL` | 这列不能是 NULL |
| 默认值 | `DEFAULT` | 不给值时填什么 |
| 主键 | `PRIMARY KEY` | 一行都不能重复 |
| 唯一 | `UNIQUE` | 这列值不能重复 |
| 检查 | `CHECK` | 值必须满足条件 |
| 外键 | `FOREIGN KEY` | 不能引用不存在的行 |

前四个在 DDL 课里已经用过（建表时那些 `NOT NULL`、`PRIMARY KEY` 就是），本课把它们系统化，**重点放在外键**（34、35 两集都在讲它）。

---

## 1. 为什么需要它

程序会出 bug，人会手动改数据。**约束是数据库这一层的最后一道闸门。**

- 没有 `NOT NULL`：注册接口漏传用户名，库里就出现一条"没有名字"的用户
- 没有 `UNIQUE`：并发注册同一个邮箱，出现两条一样的账号
- 没有 `FOREIGN KEY`：把员工分到了不存在的部门，数据成了孤儿

**约束的原则是"能拦住就别指望程序自觉"。** 程序可以改，约束在库里是硬性的。

---

## 2. 核心用法

### 2.1 六种约束的写法

```sql
CREATE TABLE 表名 (
    字段 类型 [NOT NULL] [UNIQUE] [PRIMARY KEY] [DEFAULT 值] [CHECK(条件)],
    [CONSTRAINT 外键名 FOREIGN KEY (字段) REFERENCES 父表(父字段)]
);
```

对应到一张表上：

```sql
CREATE TABLE dept (
  id   INT PRIMARY KEY AUTO_INCREMENT COMMENT '部门id',
  name VARCHAR(50) NOT NULL COMMENT '部门名称'
) COMMENT '部门表';
```

这一行里三个约束：`PRIMARY KEY`（主键）、`AUTO_INCREMENT`（自增）、`NOT NULL`（非空）。

### 2.2 主键 PRIMARY KEY

- 唯一 + 非空，一张表只能有一个
- 通常配 `AUTO_INCREMENT`，让数据库自己发号
- **实测**：主键列就算不写 `NOT NULL`，数据库也会自动加上（`SHOW CREATE TABLE` 里能看到）

### 2.3 唯一 UNIQUE

同一列里值不能重复，但**允许多个 NULL**（NULL 不算"重复"）。

实测：

```sql
CREATE TABLE t_uni (a VARCHAR(10) UNIQUE);
INSERT INTO t_uni VALUES ('x');
INSERT INTO t_uni VALUES ('x');
```

第二条报：

```
ERROR 1062 (23000): Duplicate entry 'x' for key 'a'
```

> 注意：唯一约束受**排序规则**影响。`_ci` 规则下 `'x'` 和 `'X'` 会被视为重复（见 DDL 课 2.1）。

### 2.4 非空 NOT NULL 与默认值 DEFAULT

这两条在 [DDL 教程](MySQL_DDL教程.md) 里已经讲透，一句话回顾：

- `NOT NULL` 要求这列**最终有值**
- `DEFAULT` 管"你不给时拿什么顶上"

配合起来：`NOT NULL DEFAULT 0` 意思是"不能是 NULL，不写就填 0"。

### 2.5 检查 CHECK

限制值的范围。**MySQL 8.0.16 起才真正生效**（之前的版本只解析不执行），MariaDB 10.2.1 起支持。

```sql
CREATE TABLE t_check (age INT CHECK (age >= 0 AND age <= 150));
INSERT INTO t_check VALUES (-1);
```

实测报错（MariaDB）：

```
ERROR 4025 (23000): CONSTRAINT `t_check.age` failed for `demo_sql`.`t_check`
```

`200` 也被拦住了，因为超过 150 的上限。**这就是 CHECK 的价值**：能把"年龄不可能是负数"这种业务规则写进结构里。

### 2.6 ⭐ 外键约束 FOREIGN KEY（34、35 集重点）

#### 它解决什么问题

假设有 `dept`（部门）和 `emp`（员工）两张表。员工表里存 `dept_id` 表示"这个人在哪个部门"。

如果没有外键，你可以：

- 给员工填一个不存在的部门 id `99`
- 删除一个还有员工的部门

**外键就是用来堵这两个漏洞的。**

#### 建表语法

```sql
-- 父表：部门
CREATE TABLE dept (
  id   INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) NOT NULL
);

-- 子表：员工，dept_id 引用 dept.id
CREATE TABLE emp (
  id      INT PRIMARY KEY AUTO_INCREMENT,
  name    VARCHAR(10) NOT NULL,
  dept_id INT,
  CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id) REFERENCES dept(id)
);
```

| 部分 | 含义 |
| --- | --- |
| `dept` | 父表（被引用） |
| `emp` | 子表（引用别人） |
| `dept_id` | 外键列，存的值必须存在于 `dept.id` 里 |
| `fk_emp_dept` | 约束名（起个好认的，报错时能看出来是哪个） |

#### 实测：外键堵住的两个漏洞

**漏洞一：插入不存在的部门 id**

```sql
INSERT INTO emp (name, dept_id) VALUES ('王五', 99);
```

```
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails
(`demo_sql`.`emp_fk`, CONSTRAINT `fk_emp_dept` FOREIGN KEY (`dept_id`) REFERENCES `dept` (`id`))
```

**漏洞二：删除还被引用的部门**

```sql
DELETE FROM dept WHERE id = 1;
```

```
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails
(`demo_sql`.`emp_fk`, CONSTRAINT `fk_emp_dept` FOREIGN KEY (`dept_id`) REFERENCES `dept` (`id`))
```

这就是外键默认的 `RESTRICT` 行为：**不许删，先把子表数据处理掉。**

#### 五种删除/更新行为（35 集）

外键的"当父表被改时怎么办"，可以选五种：

| 行为 | 意思 | 实测 |
| --- | --- | --- |
| `RESTRICT`（默认） | 拒绝操作 | ✅ 报 ERROR 1451 / 1452 |
| `NO ACTION` | 同 RESTRICT | 同默认 |
| `CASCADE` | 级联：父表删/改，子表跟着删/改 | ✅ 实测通过 |
| `SET NULL` | 父表删了，子表外键列置 NULL | ✅ 实测通过 |
| `SET DEFAULT` | 置默认值 | InnoDB **不生效**，别用 |

#### 实测 CASCADE（级联删除 + 级联更新）

建表时写上：

```sql
CREATE TABLE emp_cascade (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(10),
  dept_id INT,
  FOREIGN KEY (dept_id) REFERENCES dept(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
);
```

先插两条：

| name | dept_id |
| --- | --- |
| 甲 | 1 |
| 乙 | 2 |

**删除父表 id=1 之后**：

```sql
DELETE FROM dept WHERE id = 1;
```

| name | dept_id |
| --- | --- |
| 乙 | 2 |

「甲」**跟着一起被删了**——这就是 CASCADE。

**更新父表 id=2 → 200 之后**：

```sql
UPDATE dept SET id = 200 WHERE id = 2;
```

「乙」的 dept_id 也从 2 变成了 **200**。父键改了，子键自动跟着改。

#### 实测 SET NULL

```sql
FOREIGN KEY (dept_id) REFERENCES dept(id) ON DELETE SET NULL
```

删掉父表 id=3 后，子表那条的 dept_id 变成 **NULL**（不是被删，是"失去归属"）。

> 注意：`SET NULL` 要求子表这一列**允许 NULL**。如果它是 `NOT NULL`，建表就会报错。

#### 怎么查已有的外键

```sql
SELECT TABLE_NAME, CONSTRAINT_NAME, COLUMN_NAME, REFERENCED_TABLE_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = '你的库名' AND REFERENCED_TABLE_NAME IS NOT NULL;
```

实测输出：

| TABLE_NAME | CONSTRAINT_NAME | COLUMN_NAME | REFERENCED_TABLE_NAME |
| --- | --- | --- | --- |
| emp_fk | fk_emp_dept | dept_id | dept |

#### 外键用不用在开发里？

| 场景 | 建议 |
| --- | --- |
| 学习 / 小型项目 | 用，能强制保证数据一致 |
| 互联网高并发业务 | **很多公司不用**，改在应用层校验 |

原因：外键会给插入、删除加锁检查，高并发下影响性能；而且分库分表后跨库根本建不了外键。**但知道原理是必须的**——出了问题（比如看到 `ERROR 1451`）得能看懂。

---

## 3. 常见坑汇总

| # | 坑 | 现象 | 正确做法 |
| --- | --- | --- | --- |
| 1 | 删父表数据报错 | `ERROR 1451` | 先删子表数据，或改外键行为 |
| 2 | 子表插不存在的父 id | `ERROR 1452` | 先确认父表有这个 id |
| 3 | 给唯一列插重复值 | `ERROR 1062 Duplicate entry` | 先查再插，或用 `INSERT IGNORE` |
| 4 | 以为 CHECK 是万能的 | 老版本（MySQL < 8.0.16）直接忽略 | 老版本靠程序校验 |
| 5 | 建外键时父表字段类型不一致 | 建表报错 | 两边类型必须一致 |
| 6 | 用 `SET NULL` 但列是 NOT NULL | 建表报错 | 列改成允许 NULL |
| 7 | 以为 `SET DEFAULT` 能用 | 建表可能成功但行为不对 | 别用，换成 SET NULL |
| 8 | 直接 `DROP TABLE 父表` | 报错（有子表引用） | 先删子表/外键 |
| 9 | 外键列没建索引 | 性能差 | InnoDB 会自动给外键建索引，通常不用管 |

---

## 4. 什么时候用 / 不用

| 该用 | 不该用 |
| --- | --- |
| `NOT NULL` / `UNIQUE`（几乎必加） | 给所有列都加 CHECK（维护成本高） |
| `PRIMARY KEY`（每张表都该有） | 分库分表场景建外键（跨库建不了） |
| 对一致性要求高的业务用外键 | 高频写入表上滥用外键 |

---

## 5. 自测

- [ ] 建父表和子表，给子表加外键，验证插入不存在的父 id 报 1452
- [ ] 验证删除被引用的父行报 1451
- [ ] 改成 `ON DELETE CASCADE`，删父行，观察子表数据
- [ ] 改成 `ON DELETE SET NULL`，删父行，观察子表外键列变 NULL
- [ ] 用 `ON UPDATE CASCADE`，改父表 id，观察子表跟着变
- [ ] 建一张 `UNIQUE` 表，插两条相同值，抄下报错
- [ ] 建一张 `CHECK (age BETWEEN 0 AND 150)` 的表，插 -1，观察报错
- [ ] 用 `information_schema.KEY_COLUMN_USAGE` 查出自己库里的所有外键
