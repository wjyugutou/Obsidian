# SQL 常用关键字

> 实际开发中最常用的关键字，按用途分类整理。

## 查询（DQL）

| 关键字 | 用途 | 示例 |
| --- | --- | --- |
| `SELECT` | 查询数据 | `SELECT 姓名 FROM Students` |
| `FROM` | 指定表 | `FROM Students` |
| `AS` | 别名 | `SELECT 姓名 AS name` |
| `DISTINCT` | 去重 | `SELECT DISTINCT 班級` |
| `WHERE` | 行级过滤 | `WHERE 成績 > 60` |
| `ORDER BY` | 排序（默认升序） | `ORDER BY 班級, 成績 DESC` |
| `ASC` / `DESC` | 升序 / 降序 | `ORDER BY 成績 DESC` |
| `GROUP BY` | 分组 | `GROUP BY 班級` |
| `HAVING` | 分组后过滤（用于聚合条件） | `HAVING COUNT(*) > 10` |
| `LIMIT` | 限制行数（MySQL/PostgreSQL/SQLite） | `LIMIT 10` |
| `OFFSET` | 跳过行数（分页） | `LIMIT 10 OFFSET 20` |
| `TOP` | 限制行数（SQL Server） | `SELECT TOP 10 *` |

## 条件 / 逻辑

| 关键字 | 用途 | 示例 |
| --- | --- | --- |
| `AND` / `OR` / `NOT` | 逻辑组合 | `WHERE 成績 >= 60 AND 班級 = '1年1班'` |
| `IN` | 属于集合 | `WHERE 班級 IN ('1年1班','1年2班')` |
| `BETWEEN ... AND` | 范围（含边界） | `WHERE 成績 BETWEEN 60 AND 90` |
| `LIKE` | 模糊匹配（`%` 任意多个，`_` 单个） | `WHERE 姓名 LIKE '张%'` |
| `IS NULL` / `IS NOT NULL` | 空值判断（不能用 `= NULL`） | `WHERE 成绩 IS NOT NULL` |
| `EXISTS` | 是否存在子查询结果 | `WHERE EXISTS (SELECT 1 ...)` |
| `ANY` / `ALL` | 与子查询比较 | `WHERE 成績 > ALL (SELECT ...)` |
| `CASE ... WHEN ... THEN ... ELSE ... END` | 条件分支 | `CASE WHEN 成績 >= 90 THEN '优' ELSE '良' END` |

## 连接（JOIN）

| 关键字 | 用途 |
| --- | --- |
| `JOIN` / `INNER JOIN` | 内连接，只取两表匹配的行 |
| `LEFT JOIN` | 左连接，左表全部 + 右表匹配（无匹配补 NULL） |
| `RIGHT JOIN` | 右连接，右表全部 + 左表匹配 |
| `FULL OUTER JOIN` | 全连接，两表全部（MySQL 不支持） |
| `CROSS JOIN` | 笛卡尔积 |
| `ON` | 连接条件（推荐用 ON 而非 WHERE 做连接条件） |
| `USING(col)` | 等价于 `ON 两表.col = col` 的简写 |
| `UNION` / `UNION ALL` | 纵向合并结果集（UNION 去重，UNION ALL 不去重、更快） |

## 聚合函数

| 关键字 | 用途 |
| --- | --- |
| `COUNT(*)` | 计数 |
| `SUM(col)` | 求和 |
| `AVG(col)` | 平均值 |
| `MAX(col)` / `MIN(col)` | 最大值 / 最小值 |

> 聚合函数遇到 `NULL` 会忽略该行；`COUNT(*)` 不计 NULL。

## 数据操作（DML）

| 关键字 | 用途 | 示例 |
| --- | --- | --- |
| `INSERT INTO ... VALUES` | 插入 | `INSERT INTO Students (姓名, 成績) VALUES ('张三', 85)` |
| `UPDATE ... SET` | 更新 | `UPDATE Students SET 成績 = 90 WHERE 姓名 = '张三'` |
| `DELETE FROM` | 删除行 | `DELETE FROM Students WHERE 成績 < 60` |
| `TRUNCATE TABLE` | 清空表（速度比 DELETE 快，不可回滚） | `TRUNCATE TABLE Students` |

## 建表 / 表结构（DDL）

| 关键字 | 用途 |
| --- | --- |
| `CREATE TABLE` / `DROP TABLE` / `ALTER TABLE` | 建表 / 删表 / 改表 |
| `ADD` / `DROP COLUMN` / `MODIFY` | 加列 / 删列 / 改列类型 |
| `PRIMARY KEY` | 主键（唯一 + 非空） |
| `FOREIGN KEY ... REFERENCES` | 外键（关联其他表） |
| `UNIQUE` | 唯一约束 |
| `NOT NULL` / `NULL` | 非空 / 允许空 |
| `DEFAULT` | 默认值 |
| `AUTO_INCREMENT` | 自增（MySQL；PostgreSQL 用 `SERIAL`） |
| `CONSTRAINT` | 给约束命名 |
| `CREATE INDEX` / `DROP INDEX` | 建索引 / 删索引 |
| `CREATE VIEW` / `DROP VIEW` | 视图 |
| `CREATE DATABASE` / `USE` | 建库 / 切换库 |

## 事务（TCL）

| 关键字 | 用途 |
| --- | --- |
| `BEGIN TRANSACTION` | 开启事务 |
| `COMMIT` | 提交（永久生效） |
| `ROLLBACK` | 回滚（撤销未提交的修改） |
| `SAVEPOINT` | 事务内设保存点，可部分回滚 |

## 权限（DCL）

| 关键字 | 用途 |
| --- | --- |
| `GRANT` | 授权 |
| `REVOKE` | 收回权限 |

## 常用但不常写的关键字

- `WHERE 1=1`：恒真条件，常用于拼接动态 SQL 简化逻辑
- `NULLIF(a, b)`：a = b 时返回 NULL（避免除零）
- `COALESCE(a, b, c)`：返回第一个非 NULL 值
- `IFNULL(a, b)` / `ISNULL(a, b)`：MySQL / SQL Server 的 COALESCE 简写

## 备注

- `CONTAINS`：非标准关键字，SQL Server / Oracle 做全文搜索用（需先建全文索引），MySQL 对应 `MATCH ... AGAINST`，PostgreSQL 对应 `@@`。日常模糊查询用 `LIKE` 即可
- `<>` 与 `!=` 等价，都是"不等于"；`<>` 是 ANSI 标准写法
- 中文排序取决于 collation（如 utf8mb4 按 Unicode 码点，非拼音）
- 各数据库关键字略有差异（如 `LIMIT` vs `TOP`），迁移数据库时注意
