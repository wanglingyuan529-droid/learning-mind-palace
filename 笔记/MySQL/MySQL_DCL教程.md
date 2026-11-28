---
tags: [MySQL, 数据库, SQL, DCL]
---

# MySQL DCL 教程（用户与权限）

> 用途：学习笔记 · 定位：只要"技"（怎么用、怎么避坑），不讲"道"
> 覆盖：24. 基础-SQL-DCL-用户管理 ~ 26. 基础-SQL-DCL小结（共 3 集）
> 所有示例和输出都是在本机 **MariaDB 13.0.2** 上实测跑出来的

---

## 0. 一句话认识

**DCL（Data Control Language）管"**谁能干什么**"：建用户、给权限、收权限。**

| 操作 | 语句 | 一句话 |
| --- | --- | --- |
| 建用户 | `CREATE USER` | 添个人 |
| 改密码 | `ALTER USER` | 换口令 |
| 删用户 | `DROP USER` | 删账号 |
| 给权限 | `GRANT` | 发钥匙 |
| 收权限 | `REVOKE` | 收回钥匙 |
| 看权限 | `SHOW GRANTS` | 查有哪些钥匙 |

这三个集只讲"技"，重点是 **GRANT / REVOKE 怎么给权限**。

---

## 1. 为什么需要它

你平时用 root 账号连数据库，什么都能干——**这就是最大的风险**。

想象一个场景：程序里有个 bug 把 `DELETE FROM emp` 写进了查询接口，而程序连库用的是 root。**整张表当场清空。**

正确做法是给程序单独建一个账号，**只给 SELECT**。这样就算代码写错，数据库也会拒绝执行——**DCL 是最后一道防线**。

> ⚠️ 本笔记的操作需要管理员权限。生产环境建用户、给权限都要走审批，别随手 `GRANT ALL`。

---

## 2. 核心用法

### 2.1 用户管理

#### 查用户

```sql
SELECT User, Host FROM mysql.user;
```

用户信息存在系统库 `mysql` 的 `user` 表里。

#### 创建用户

```sql
CREATE USER 'test_z'@'localhost' IDENTIFIED BY 'Zz123456';
```

语法要点：

```
'用户名'@'主机地址'
```

| 部分 | 意思 |
| --- | --- |
| 用户名 | 账号名 |
| `@localhost` | **只能从本机连** |
| `@'%'` | 允许任意 IP 连（生产慎用） |
| `@'192.168.1.10'` | 只允许指定 IP |

> **用户名 + 主机地址**才是一个完整账号。`'test_z'@'localhost'` 和 `'test_z'@'%'` 是**两个不同的用户**，权限各算各的。这是 DCL 最容易误解的一点。

#### 改密码

```sql
ALTER USER 'test_z'@'localhost' IDENTIFIED BY 'NewPass456';
```

改完要用新密码重新登录。

#### 删用户

```sql
DROP USER 'test_z'@'localhost';
```

### 2.2 权限控制

#### 看某人的权限

```sql
SHOW GRANTS FOR 'test_z'@'localhost';
```

**刚建的用户，实测输出是：**

```
GRANT USAGE ON *.* TO `test_z`@`localhost` IDENTIFIED BY PASSWORD '...'
```

`USAGE` 的意思是"**能连接，但没有任何数据权限**"。所以新用户能登录，但查不了任何业务表。

#### 给权限：GRANT

语法：

```sql
GRANT 权限列表 ON 库名.表名 TO '用户'@'主机';
```

```sql
GRANT SELECT ON demo_sql.* TO 'test_z'@'localhost';
```

| 部分 | 含义 |
| --- | --- |
| `SELECT` | 权限类型，多个用逗号分隔 |
| `demo_sql.*` | 授权范围：整个库的所有表 |
| `demo_sql.emp` | 只授权这一张表 |

常用权限类型：

| 权限 | 能干什么 |
| --- | --- |
| `SELECT` | 查 |
| `INSERT` | 插 |
| `UPDATE` | 改 |
| `DELETE` | 删 |
| `CREATE` | 建表建库 |
| `DROP` | 删表删库 |
| `ALTER` | 改表结构 |
| `ALL PRIVILEGES` | 以上全给（危险） |

给完再看一次：

```sql
SHOW GRANTS FOR 'test_z'@'localhost';
```

```
GRANT USAGE ON *.* TO ...
GRANT SELECT ON `demo_sql`.* TO `test_z`@`localhost`
```

多了一条 `GRANT SELECT ON demo_sql.*`。

#### 实测：给权限前后的差别

给 `SELECT` 之后，用新用户登录查表：

```sql
-- 以 test_z 身份连接
SELECT COUNT(*) FROM emp;     -- ✅ 返回 11
```

但让他删数据：

```sql
DELETE FROM emp WHERE id = 1;
-- ERROR 1142 (42000): DELETE command denied to user 'test_z'@'localhost'
--                     for table `demo_sql`.`emp`
```

**这就是权限的意义**：程序代码写错了，数据库替你把关。

#### 收权限：REVOKE

```sql
REVOKE SELECT ON demo_sql.* FROM 'test_z'@'localhost';
```

再查：

```sql
SELECT COUNT(*) FROM emp;
-- ERROR 1142 (42000): SELECT command denied to user 'zz'@'localhost'
--                     for table `ptest`.`t`
```

**权限回收立刻生效**，不用重启、不用重连（前提是新开的连接）。

#### 一步到位：ALL PRIVILEGES

```sql
GRANT ALL PRIVILEGES ON demo_sql.* TO 'test_z'@'localhost';
```

> 生产环境**不要**这么写。按最小权限原则：程序只要 SELECT 就只给 SELECT。
> 确实需要更多时，按功能拆：读账号给 SELECT，写账号给 INSERT/UPDATE，绝不混用。

---

## 3. 常见坑汇总

| # | 坑 | 现象 | 正确做法 |
| --- | --- | --- | --- |
| 1 | 只写用户名忘了 `@host` | 建出来的账号跟预期对不上 | 用户名和 host 是一个整体 |
| 2 | 以为 `'u'@'localhost'` 能远程连 | 远程连不上 | 远程要 `'u'@'%'` 或指定 IP |
| 3 | 新用户能连上就连库 | `USAGE` 只是连接权限 | 必须显式 `GRANT` |
| 4 | 用 `GRANT ALL` 图省事 | 程序 bug 能清库 | 最小权限：只给需要的 |
| 5 | 改密码后旧连接还能用 | 改的是新连接的密码 | 主动断开旧连接 |
| 6 | 忘记 `DROP USER` 测试账号 | 留下后门账号 | 用完即删 |
| 7 | 密码明文写进代码/脚本 | 泄露 | 用环境变量或配置中心 |

---

## 4. 什么时候用 / 不用

| 该用 | 不该用 |
| --- | --- |
| 给每个程序单独建账号 | 让程序用 root 连库 |
| 按功能拆分只读/读写账号 | 给开发环境所有库 `ALL PRIVILEGES` |
| 离职/交接时收回权限 | 共享同一个账号给多人 |

---

## 5. 自测

- [ ] 建一个只能查、不能改的用户，实际验证 SELECT 成功、DELETE 报 1142
- [ ] 用 `SHOW GRANTS` 前后对比，看权限多了哪一行
- [ ] 把权限 `REVOKE` 掉，再查一次，确认立刻失效
- [ ] 试一下 `'u'@'localhost'` 和 `'u'@'%'` 建两个同名账号，观察 `mysql.user` 里的行数
- [ ] 删掉测试账号，用 `SELECT COUNT(*) FROM mysql.user WHERE User='...'` 确认
