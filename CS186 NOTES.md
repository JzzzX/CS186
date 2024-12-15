# CS186 NOTES

## SQL

### Basic Queies

#### 基本概念

- **表（Table）**：关系型数据库中的数据结构，例如 `Person` 表。
- **行（Row / Tuple）**：表中的记录，例如 `Ace 20 4`。
- **列（Column / Attribute）**：表的字段，例如 `name, age, num_dogs`。

------

#### 基础查询语法

```sql
SELECT <列名>
FROM <表名>;
```

示例：

```sql
SELECT name, num_dogs
FROM Person;
```

输出为：

| name | num_dogs |
| ---- | -------- |
| Ace  | 4        |
| Ada  | 3        |
| Ben  | 2        |
| Cho  | 3        |

⚠️ 查询结果的行顺序是**不确定的**，除非使用 `ORDER BY`。

------

#### 条件查询（WHERE）

```sql
SELECT <列名>
FROM <表名>
WHERE <条件>;
```

示例：

```sql
SELECT name, num_dogs
FROM Person
WHERE age >= 18;
```

执行步骤：

1. 按 `FROM` 确定数据来源。
2. 根据 `WHERE` 条件过滤记录。

------

#### 布尔操作符

- **AND**：所有条件为真时通过。
- **OR**：任一条件为真时通过。
- **NOT**：条件为假时通过。

示例：

```sql
SELECT name, num_dogs
FROM Person
WHERE age >= 18 AND num_dogs > 3;
```

------

#### 处理 NULL 值

- `x IS NULL` 检查 `x` 是否为 NULL。
- `x IS NOT NULL` 检查 `x` 是否非 NULL。

------

#### 分组与聚合（GROUP BY）

**语法：**

```sql
SELECT <列名>, 聚合函数
FROM <表名>
GROUP BY <列名>
HAVING <条件>;
```

- **聚合函数**：SUM, AVG, MAX, MIN, COUNT。

- **HAVING**：对分组后的数据进行过滤。

- 与 WHERE 的区别：

	- `WHERE`：分组之前过滤记录。
- `HAVING`：分组之后过滤分组。

示例：

```sql
SELECT age, AVG(num_dogs)
FROM Person
WHERE age >= 18
GROUP BY age
HAVING COUNT(*) > 1;
```

------

#### 排序（ORDER BY）

```sql
SELECT <列名>
FROM <表名>
ORDER BY <列名> [ASC|DESC];
```

默认情况下，排序顺序是升序，但如果我们想按降序排序，可以在列名后添加 DESC 关键字。如果我们想按 "狗数 "升序排序，并按 "名称 "降序打破并列关系，我们可以使用下面的查询：

```sql
SELECT name, num_dogs
FROM Person
ORDER BY num_dogs, name DESC;
```

------

#### 限制返回行数（LIMIT）

```sql
SELECT <列名>
FROM <表名>
LIMIT <行数>;
```

------

#### 练习

**问题 1**：查找 `ownerid=3` 的狗的名字。

```sql
SELECT name
FROM dogs
WHERE ownerid = 3;
```

**问题 2**：列出年龄最大的 5 只狗的名字和年龄（按名字升序打破平局）。

```sql
SELECT name, age
FROM dogs
ORDER BY age DESC, name ASC
LIMIT 5;
```

**问题 3**：统计每个品种的狗的数量，仅显示多于一只的品种。

```sql
SELECT breed, COUNT(*)
FROM dogs
GROUP BY breed
HAVING COUNT(*) > 1;
```

------

#### 总结

SQL 查询步骤顺序（逻辑顺序）：

1. **FROM**：确定数据来源。
2. **WHERE**：过滤记录。
3. **GROUP BY**：分组。
4. **HAVING**：过滤分组。
5. **SELECT**：选择需要的列。
6. **ORDER BY**：排序。
7. **LIMIT**：限制返回行数。

<img src="截图/截屏2024-12-12 19.51.13.png" alt="截屏2024-12-12 19.51.13" style="zoom:30%;" />

------



### Joins and Subqueries

**连接与子查询**

------

#### **SQL 中的连接 (Joins)**

![Untitled](https://notes.bencuan.me/cs186/SQL%20Basics/Untitled%201.png)

#### **交叉连接 (Cross Join)**

- **定义**：

	- 交叉连接会生成**笛卡尔积**，即左表中的每一行与右表中的每一行组合。

	- 语法：

		```sql
		SELECT *
		FROM table1, table2;
		```

- **特点**：

	- 不需要任何条件，结果可能非常庞大，包含所有可能的行组合。
	- 通常需要结合 `WHERE` 子句限制结果集。

- **示例**： 假设表 `courses` 和 `enrollment` 数据如下：

	`courses` 表：

	| num   | name |
	| ----- | ---- |
	| CS186 | DB   |
	| CS188 | AI   |
	| CS189 | ML   |

	`enrollment` 表：

	| c_num | students |
	| ----- | -------- |
	| CS186 | 700      |
	| CS188 | 800      |

	执行以下查询：

	```sql
	SELECT *
	FROM courses, enrollment;
	```

	返回结果：

	| num   | name | c_num | students |
	| ----- | ---- | ----- | -------- |
	| CS186 | DB   | CS186 | 700      |
	| CS186 | DB   | CS188 | 800      |
	| CS188 | AI   | CS186 | 700      |
	| CS188 | AI   | CS188 | 800      |
	| CS189 | ML   | CS186 | 700      |
	| CS189 | ML   | CS188 | 800      |

	如果想要匹配课程编号 (`num` 和 `c_num`)，需要添加 `WHERE` 子句：

	```sql
	SELECT *
	FROM courses, enrollment
	WHERE courses.num = enrollment.c_num;
	```

	返回结果：

	| num   | name | c_num | students |
	| ----- | ---- | ----- | -------- |
	| CS186 | DB   | CS186 | 700      |
	| CS188 | AI   | CS188 | 800      |

------

#### **内连接 (Inner Join)**

- **定义**：

	- 内连接是最常用的连接类型，仅返回**两表中匹配条件的记录**。
	- 与交叉连接 + `WHERE` 的逻辑相同，但语法更清晰。

- **语法**：

	```sql
	SELECT column_names
	FROM table1
	INNER JOIN table2
	ON table1.column_name = table2.column_name;
	```

- **示例**： 上述课程与注册表的例子，用内连接表示为：

	```sql
	SELECT *
	FROM courses
	INNER JOIN enrollment
	ON courses.num = enrollment.c_num;
	```

	返回结果与交叉连接 + 条件一致：

	| num   | name | c_num | students |
	| ----- | ---- | ----- | -------- |
	| CS186 | DB   | CS186 | 700      |
	| CS188 | AI   | CS188 | 800      |

------

#### **外连接 (Outer Join)**

- **定义**：

	- 外连接允许保留**未匹配的记录**，并使用 `NULL` 填充另一侧的列值。
	- 包括：
		- **左连接 (LEFT JOIN)**：保留左表所有记录。
		- **右连接 (RIGHT JOIN)**：保留右表所有记录。
		- **完全外连接 (FULL OUTER JOIN)**：保留两表中所有记录。

- **语法**：

	```sql
	SELECT column_names
	FROM table1
	LEFT JOIN table2
	ON table1.column_name = table2.column_name;
	```

- **示例：左连接**：

	```sql
	SELECT *
	FROM courses
	LEFT JOIN enrollment
	ON courses.num = enrollment.c_num;
	```

	返回结果：

	| num   | name | c_num | students |
	| ----- | ---- | ----- | -------- |
	| CS186 | DB   | CS186 | 700      |
	| CS188 | AI   | CS188 | 800      |
	| CS189 | ML   | NULL  | NULL     |

- **示例：完全外连接**：

	```sql
	SELECT *
	FROM courses
	FULL OUTER JOIN enrollment
	ON courses.num = enrollment.c_num;
	```

	返回结果：

	| num   | name | c_num | students |
	| ----- | ---- | ----- | -------- |
	| CS186 | DB   | CS186 | 700      |
	| CS188 | AI   | CS188 | 800      |
	| CS189 | ML   | NULL  | NULL     |
	| NULL  | NULL | CS160 | 400      |

------

#### **自然连接 (Natural Join)**

- **定义**：

	- 自动匹配两表中**列名相同**的列，基于这些列进行连接。
	- 语法简单，但易引发歧义。

- **语法**：

	```sql
	SELECT *
	FROM table1
	NATURAL JOIN table2;
	```

- **注意**：

	- 由于连接条件是隐式的，可能因列名变化或多余列导致问题。
	- 不建议在复杂场景中使用。

------

#### **子查询 (Subqueries)**

##### **子查询的基本概念**

- 子查询是嵌套在另一个查询中的查询，用于解决更复杂的问题。
- 可用作：
	- **WHERE 子句**的过滤条件。
	- **FROM 子句**中的临时表。
	- **SELECT 子句**中的派生值。

------

##### **WHERE 子句中的子查询**

- 示例：查找学生人数超过平均值的课程：

	```sql
	SELECT num
	FROM enrollment
	WHERE students >= (
	    SELECT AVG(students)
	    FROM enrollment
	);
	```

	- 逻辑：
		- 内层子查询计算平均学生人数。
		- 外层查询筛选出人数大于或等于平均值的课程。

------

##### **相关子查询 (Correlated Subqueries)**

- 子查询依赖于外层查询的值，每一行都会调用子查询。

- 示例：查找在两个表中都存在的课程：

	```sql
	SELECT *
	FROM classes
	WHERE EXISTS (
	    SELECT *
	    FROM enrollment
	    WHERE classes.num = enrollment.num
	);
	```

	- 逻辑：
		- 对于每个 `classes` 的行，检查是否存在匹配的 `enrollment` 行。
		- 如果存在，返回 `TRUE`；否则返回 `FALSE`。

------

##### **FROM 子句中的子查询**

- 定义：

	- 子查询可以作为临时表使用，供外层查询调用。

- 示例：

	```sql
	SELECT *
	FROM (
	    SELECT num
	    FROM classes
	) AS a
	WHERE num = 'CS186';
	```

	- 内层子查询返回课程编号，外层查询筛选出编号为 `CS186` 的记录。

------

#### 练习

我们有两张表：dogs 和 users。

dogs **表**（存储狗的信息）：

```sql
CREATE TABLE dogs (
    dogid integer,
    ownerid integer,
    name varchar,
    breed varchar,
    age integer,
    PRIMARY KEY (dogid),
    FOREIGN KEY (ownerid) REFERENCES users(userid)
);
```

users **表**（存储用户信息）：

```sql
CREATE TABLE users (
    userid integer,
    name varchar,
    age integer,
    PRIMARY KEY (userid)
);
```

**练习问题**

1. 写一条查询语句，列出所有由“Josh Hug”拥有的狗的名字

```sql
SELECT dogs.name
FROM dogs
INNER JOIN users ON dogs.ownerid = users.userid
WHERE users.name = "Josh Hug";
```

2. 使用不同的连接方式重写上一条查询（如使用交叉连接和WHERE 子句）

```sql
SELECT dog.names
FROM dogs, users
WHERE dogs.ownerid = users.userid AND users.name = "Josh Hug"
```

3. 写一条查询语句，找出拥有最多狗的用户的名字以及他们拥有狗的数量。假设没有并列情况（只有一个用户拥有最多狗）

```sql
SELECT users.name, COUNT(*)
FROM users INNER JOIN dogs ON users.userid = dogs.ownerid
GROUP BY users.userid, users.name
ORDER BY COUNT(*) DESC
LIMIT 1;
```

4. 重写上一条查询，考虑存在并列的情况（即可能有多个用户拥有最多狗）

```sql
SELECT users.name, COUNT(*)
FROM users INNER JOIN dogs ON users.userid = dogs.ownerid
GROUP BY users.userid, users.name
HAVING COUNT(*) >= ALL (
	SELECT COUNT(*)
    FROM dogs
    GROUP BY ownerid
);
```























