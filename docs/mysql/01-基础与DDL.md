# 01 — 基础与 DDL

## 1.1 数据库基础概念

### 什么是数据库

数据库（Database）是按照数据结构来组织、存储和管理数据的仓库。

### 关系型数据库

- **数据以表格形式存储**（行和列）
- **表与表之间可以建立关联**（通过外键）
- **使用 SQL**（Structured Query Language）操作数据
- 常见产品：MySQL、PostgreSQL、Oracle、SQL Server、SQLite

### 核心术语

| 术语 | 说明 |
|------|------|
| 数据库（Database） | 多个表的集合 |
| 表（Table） | 数据的二维结构，由行和列组成 |
| 行（Row/Record） | 一条数据记录 |
| 列（Column/Field） | 一个字段，有固定的数据类型 |
| 主键（Primary Key） | 唯一标识一行数据的字段 |
| 外键（Foreign Key） | 关联其他表主键的字段 |
| 索引（Index） | 加速数据查询的数据结构 |

### MySQL 架构简图

```
客户端 → 连接器 → 分析器 → 优化器 → 执行器 → 存储引擎
                                     ↓
                                查询缓存 (8.0 已移除)
```

## 1.2 数据类型

### 数值类型

```sql
-- 整数
TINYINT     -- 1字节, -128~127 或 0~255
SMALLINT    -- 2字节
MEDIUMINT   -- 3字节
INT         -- 4字节, 约 ±21 亿
BIGINT      -- 8字节

-- 小数
DECIMAL(M,D) -- 精确小数, M 总位数, D 小数位
FLOAT        -- 单精度浮点
DOUBLE       -- 双精度浮点
```

### 字符串类型

```sql
CHAR(N)       -- 定长字符串, 0-255
VARCHAR(N)    -- 变长字符串, 0-65535
TEXT          -- 长文本, 最大 65535 字符
MEDIUMTEXT    -- 中等文本, 最大 16MB
LONGTEXT      -- 超大文本, 最大 4GB
TINYTEXT      -- 小文本, 最大 255 字符
```

> CHAR 性能优于 VARCHAR，但浪费空间；VARCHAR 更省空间。定长数据用 CHAR，不定长用 VARCHAR。

### 日期时间类型

```sql
DATE      -- 日期 YYYY-MM-DD
TIME      -- 时间 HH:MM:SS
DATETIME  -- 日期时间 YYYY-MM-DD HH:MM:SS (1000~9999)
TIMESTAMP -- 时间戳 1970-01-01 00:00:01 ~ 2038-01-19 (自动时区转换)
YEAR      -- 年份
```

> 推荐用 DATETIME 存储业务时间，用 TIMESTAMP 存储系统时间（自动更新）。

### 其他类型

```sql
ENUM('v1','v2')   -- 枚举，单选
SET('v1','v2')    -- 集合，多选
JSON              -- JSON 数据 (5.7.8+)
BLOB              -- 二进制大对象
```

## 1.3 数据库操作（DDL — 库级别）

```sql
-- 创建数据库
CREATE DATABASE mydb;
CREATE DATABASE IF NOT EXISTS mydb;
-- 指定字符集和排序规则
CREATE DATABASE mydb
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- 查看所有数据库
SHOW DATABASES;

-- 查看建库语句
SHOW CREATE DATABASE mydb;

-- 切换数据库
USE mydb;

-- 删除数据库（危险操作）
DROP DATABASE mydb;
DROP DATABASE IF EXISTS mydb;

-- 修改数据库字符集
ALTER DATABASE mydb CHARACTER SET utf8mb4;
```

> **推荐使用 utf8mb4**，这是真正的 UTF-8，支持 emoji 和所有 Unicode 字符。MySQL 的 `utf8` 是阉割版，最多 3 字节。

## 1.4 表操作（DDL — 表级别）

### 创建表

```sql
CREATE TABLE student (
    id          INT PRIMARY KEY AUTO_INCREMENT COMMENT '学号',
    name        VARCHAR(50)  NOT NULL COMMENT '姓名',
    age         TINYINT      DEFAULT 0 COMMENT '年龄',
    gender      ENUM('男','女') COMMENT '性别',
    email       VARCHAR(100) UNIQUE COMMENT '邮箱',
    class_id    INT COMMENT '班级ID',
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    INDEX idx_name (name),
    INDEX idx_class (class_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='学生表';
```

### 约束

| 约束 | 关键字 | 说明 |
|------|--------|------|
| 主键 | PRIMARY KEY | 唯一且非空，一张表只能有一个 |
| 非空 | NOT NULL | 字段不能为空 |
| 唯一 | UNIQUE | 字段值不能重复 |
| 默认 | DEFAULT | 设置默认值 |
| 检查 | CHECK | 自定义条件约束（8.0.16+ 生效） |
| 外键 | FOREIGN KEY | 关联其他表的主键 |

```sql
-- 外键示例
CREATE TABLE score (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    student_id  INT NOT NULL,
    subject     VARCHAR(50) NOT NULL,
    score       DECIMAL(5,2),
    FOREIGN KEY (student_id) REFERENCES student(id)
        ON DELETE CASCADE ON UPDATE CASCADE
);

-- 外键行为
-- ON DELETE CASCADE    : 删除主表时级联删除子表
-- ON DELETE SET NULL   : 删除主表时子表设 NULL
-- ON DELETE RESTRICT   : 有子表记录时禁止删除主表
-- ON DELETE NO ACTION  : 同 RESTRICT
```

### 修改表

```sql
-- 添加列
ALTER TABLE student ADD COLUMN phone VARCHAR(20);
ALTER TABLE student ADD COLUMN addr VARCHAR(200) AFTER email;

-- 修改列
ALTER TABLE student MODIFY COLUMN name VARCHAR(100);
ALTER TABLE student CHANGE COLUMN name full_name VARCHAR(50);
-- MODIFY 只能改类型/约束，CHANGE 还可以重命名

-- 删除列
ALTER TABLE student DROP COLUMN phone;

-- 添加索引
ALTER TABLE student ADD INDEX idx_age (age);
ALTER TABLE student ADD UNIQUE idx_email (email);

-- 删除索引
ALTER TABLE student DROP INDEX idx_age;

-- 重命名表
RENAME TABLE student TO students;
ALTER TABLE student RENAME TO students;
```

### 查看表信息

```sql
-- 查看所有表
SHOW TABLES;

-- 查看表结构
DESC student;
DESCRIBE student;
SHOW COLUMNS FROM student;

-- 查看建表语句
SHOW CREATE TABLE student;

-- 查看索引
SHOW INDEX FROM student;
```

### 删除表

```sql
DROP TABLE student;
DROP TABLE IF EXISTS student;

-- 清空表数据（保留结构）
TRUNCATE TABLE student;
-- TRUNCATE vs DELETE:
-- TRUNCATE 是 DDL，不可回滚，重置自增，不触发触发器
-- DELETE   是 DML，可回滚，保留自增，触发触发器
```

## 1.5 字符集与排序规则

```sql
-- 查看支持的字符集
SHOW CHARACTER SET;

-- 查看排序规则
SHOW COLLATION WHERE Charset = 'utf8mb4';
-- utf8mb4_unicode_ci  : 按 Unicode 标准比较，不区分大小写
-- utf8mb4_general_ci  : 更快的比较，不区分大小写
-- utf8mb4_bin         : 二进制比较，区分大小写

-- 推荐设定
SET NAMES utf8mb4;
```
