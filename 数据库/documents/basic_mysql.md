> 使用 MySQL 的常用命令和部分知识点补充

## 命令行使用MySQL

```shell
# 使用密码在命令行登录 MySQL
> mysql -u root -p

# ;结尾

# 获得帮助
mysql> help 
mysql> \h

# 退出命令行
mysql> quit
mysql> exit
# control + d
```

   

## MySQL 命令

> 书写习惯：

### SHOW 查看/显示

```sql
-- 查看数据库列表
SHOW DATABASES;	

-- 使用数据库 database_name
USE database_name;	

-- 查看数据库中表的列表
SHOW TABLES;	

-- 查看表各列结构
SHOW COLUMNS FROM table_name;	

-- 显示授权用户
SHOW GRANTS;	
-- 显示服务器错误
SHOW ERRORS;	 
-- 显示警告信息
SHOW WARNINGS;	
-- 查看创建视图的语句
SHOW CREATE VIEW view_name;
```

   

### CREATE 创建表

```sql
-- 可以使用反引号``将表名括起来，防止与MySQL关键字冲突
CREATE TABLE tab_name (
   col1 	col1_type 	NOT NULL,
   col2 	INTEGER 	AUTOINCREMENT,
   col3 	col3_type 	NOT NULL	DEFAULT 0,
   .....
   colN 	colN_type,
   PRIMARY KEY (col1)	# 指定主键
) ENGINE=InnoDB;

/* 
常用类型：
TEXT 字符串, CHAR(100) 固长字符串
INTEGER 整型, BIGINT 长整型, REAL 实数, 
BOOL 布尔值
BLOB 二进制
DATETIME 时间 
*/
-- PRIMARY KEY 标记主键，NOT NULL标记非空
-- AUTOINCREMENT 自增，只能用于整型
-- DEFAULT 设置列默认值
-- ENGINE=InnoDB 指定表使用的引擎
```

   

### DROP/ALTER 删除/更新表

```sql
-- 删除表
DROP TABLE tab_name;

-- 新增列
ALTER TABLE ADD COLUMNS col_name col_type;

-- 重命名表
ALTER TABLE old_tab RENAME TO new_tab

-- 重命名列名
ALTER TABLE tab_name RENAME COLUMN old_col TO new_col
```

   

### INSERT 新增记录

```sql
-- 单条
INSERT INTO tab_name VALUES (xx, xx)

-- 指定列名（推荐）
INSERT INTO tab_name (col1, col3) VALUES (xx, xx)

-- 多条
INSERT INTO tab_name (col1, col2, col3) VALUES
    (xx, xx, xx),
    ...
    (xx, xx, xx);
```

   

### DELETE/UPDATE 删除/更新记录

```sql
-- 删除满足条件的记录
DELETE FROM tab_name WHERE [ condition ];

-- 更新记录
UPDATE tab_name SET col1=value1, col2=value2

-- 更新满足条件的记录（推荐加上 WHERE）
UPDATE tab_name
    SET col1=value1, col2=value2
    WHERE [ conditions ]
```

   

### SELECT 查询

```sql
-- 所有列，通配符*
SELECT * FROM tab_name;

-- DISTINCT 去除重复 
SELECT DISTINCT col1 FROM tab_name;

-- COUNT() 统计个数 
SELECT COUNT(*) FROM tab_name

-- 指定列，逗号分隔
SELECT col1, col2 FROM table_name;

-- 带查询条件 >，<，<>，!=，=，<=，>=，BETWEEN AND
-- 操作符 AND，OR，IN，NOT
SELECT * FROM table_name 
	WHERE col2 >= 18 AND col1 <> 3;
-- 空值判断 IS NULL
SELECT column_name FROM table_name
    WHERE column_name IS NULL;
-- LIKE + 通配符（% 匹配字符串，_ 匹配单个字符）
SELECT * FROM table_name
    WHERE col2 >= 18 AND col1 LIKE %stu%;
    
-- LIMIT 限制数量 
SELECT * FROM table_name 
	LIMIT 1;
SELECT * FROM table_name 
	LIMIT 4 OFFSET 3;	# 从下标为3行开始，取4行
	
-- GROUP BY 分组
SELECT col1, count(*) FROM tab_name
    WHERE [ conditions ]
    GROUP BY col1
    
-- Having 过滤
SELECT col1, count(*) FROM tab_name
    WHERE [ conditions ]
    GROUP BY col1
    HAVING [ conditions ]
    
-- ORDER BY 排序, DESC 降序，ASC 升序，MySQL中'A'和'a'相同等级
SELECT * FROM table_name 
	ORDER BY col2 DESC;
	
-- SELECT 子句顺序：SELECT - FROM - WHERE - GROUP BY - HAVING - ORDER BY - LIMIT
```

   

### 其他

```sql
-- AS 别名
SELECT col1 AS c1, col2 AS c2
	FROM table_name AS t;
	
-- 内联结，或使用 WHERE 方式内联结
SELECT col1, col2
	FROM table1 INNER JOIN tabel2;
	
-- 创建视图
CREATE VIEW view_name AS
	# 查询语句
-- 删除视图
DROP VIEW view_name;
```

   

### Transaction 事务

```sql
-- 提交
BEGIN;
INSERT INTO ...
...
COMMIT; 
-- 回滚
BEGIN;
...
ROLLBACK;
```

   

## 基本概念名词

`SQL` 结构化查询语言（Structured Query Language）

`DBMS` 数据库管理系统（数据库软件）

`schema` 模式，关于数据库和表的布局及特性的信息

`column` 列，表中的一个字段

`row` 行，数据按行存储

`primary key` 主键，表中的一列或几列，**唯一区分**表中的每行，不同行主键不同，不允许`NULL`

`*` 通配符

`table_name.column_name` 完全限定表名

`ACID` 事务具有原子性（Atomicity）、一致性（Consistency)、隔离性（Isolation）、持久性（Durability）四个标准属性

<br>   

## 其他补充

#### MySQL 优点

1. 成本：开发源代码，可以免费使用
2. 性能：执行快
3. 可信赖
4. 简单：容易安装和使用

   <br>

<br>

<br>

<br>

#### 默认端口：3306

- 查看端口：`SHOW GLOBAL VARIABLES LIKE 'port'`

   

#### 创建不同表可以指定不同的引擎

- `InnoDB` 提供事务处理，不支持全文本搜索
- `MEMORY `数据存储在内存（不是磁盘），适合临时表
- `MyISAM` 高性能，支持全文本搜索，不支持事务处理
- 注意：外键不能跨引擎 

   

### 临时记录

视图不包含任何数据，包含一个SQL查询

利用视图一次性编写基础SQL语句，可化简复杂的SQL查询语句

存储过程：一条或多条MySQL语句的集合

- 使用存储过程比使用单独的SQL语句快
- MySQL将编写存储过程的安全和访问，与执行存储过程的安全和访问区分开来





