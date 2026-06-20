# 02 — DML 与查询

## 2.1 插入数据（INSERT）

```sql
-- 插入单行（完整字段）
INSERT INTO student VALUES (1, '张三', 20, '男', 'zhangsan@mail.com', 1, NOW(), NOW());

-- 插入单行（指定字段）
INSERT INTO student (name, age, gender, email)
VALUES ('李四', 22, '女', 'lisi@mail.com');

-- 插入多行
INSERT INTO student (name, age, gender)
VALUES
    ('王五', 21, '男'),
    ('赵六', 23, '女'),
    ('孙七', 19, '男');

-- 从查询结果插入
INSERT INTO student_backup SELECT * FROM student WHERE age > 20;

-- 存在则更新（ON DUPLICATE KEY UPDATE）
INSERT INTO student (id, name, age)
VALUES (1, '张三改', 25)
ON DUPLICATE KEY UPDATE name = VALUES(name), age = VALUES(age);

-- 替换插入（存在则删除再插入）
REPLACE INTO student (id, name, age) VALUES (1, '新张三', 26);
```

## 2.2 更新数据（UPDATE）

```sql
-- 更新单列
UPDATE student SET age = 21 WHERE id = 1;

-- 更新多列
UPDATE student SET age = age + 1, updated_at = NOW() WHERE id = 1;

-- 无条件更新（危险！全表更新）
-- UPDATE student SET status = 1;  -- 慎用！

-- 关联更新
UPDATE student s
JOIN class c ON s.class_id = c.id
SET s.class_name = c.name
WHERE c.grade = 3;
```

> **UPDATE 不加 WHERE 是全表更新，极其危险。建议先用 SELECT 确认条件，再改写成 UPDATE。**

## 2.3 删除数据（DELETE）

```sql
-- 条件删除
DELETE FROM student WHERE id = 100;

-- 批量删除
DELETE FROM student WHERE age < 18;

-- 截断表（DDL，不可回滚，重置自增）
TRUNCATE TABLE student;

-- 关联删除
DELETE s FROM student s
JOIN class c ON s.class_id = c.id
WHERE c.status = 'closed';
```

> DELETE 是 DML 可回滚；TRUNCATE 是 DDL 不可回滚且更快。

## 2.4 查询数据（SELECT）

### 基本查询

```sql
-- 查询所有列
SELECT * FROM student;

-- 查询指定列
SELECT id, name, age FROM student;

-- 去重
SELECT DISTINCT gender FROM student;

-- 别名
SELECT id AS 学号, name AS 姓名 FROM student;

-- 限制行数
SELECT * FROM student LIMIT 10;        -- 前 10 行
SELECT * FROM student LIMIT 10, 20;    -- 跳过 10 行，取 20 行（分页）
```

### WHERE 条件

```sql
-- 比较运算符：=  !=  <>  >  >=  <  <=
SELECT * FROM student WHERE age > 20;

-- 逻辑运算符：AND  OR  NOT
SELECT * FROM student WHERE age > 20 AND gender = '男';

-- IN / NOT IN
SELECT * FROM student WHERE class_id IN (1, 2, 3);

-- BETWEEN ... AND ...
SELECT * FROM student WHERE age BETWEEN 18 AND 25;

-- LIKE 模糊匹配
SELECT * FROM student WHERE name LIKE '张%';   -- 以张开头
SELECT * FROM student WHERE name LIKE '%三';    -- 以三结尾
SELECT * FROM student WHERE name LIKE '%小%';   -- 包含小
SELECT * FROM student WHERE name LIKE '王_';    -- 王开头，两字

-- IS NULL / IS NOT NULL
SELECT * FROM student WHERE email IS NULL;
```

### 排序与分组

```sql
-- 排序 ORDER BY
SELECT * FROM student ORDER BY age DESC;            -- 降序
SELECT * FROM student ORDER BY age ASC, id DESC;    -- 多字段

-- 聚合函数
SELECT COUNT(*) FROM student;              -- 计数
SELECT AVG(age) FROM student;              -- 平均值
SELECT SUM(score) FROM score;              -- 求和
SELECT MAX(age), MIN(age) FROM student;    -- 最大最小

-- 分组 GROUP BY
SELECT gender, COUNT(*) AS cnt
FROM student
GROUP BY gender;

-- 带条件的分组 HAVING
SELECT class_id, COUNT(*) AS cnt
FROM student
GROUP BY class_id
HAVING cnt > 10;

-- WHERE vs HAVING
-- WHERE   : 分组前过滤行
-- HAVING  : 分组后过滤分组结果
```

### 综合查询示例

```sql
SELECT
    class_id,
    gender,
    COUNT(*) AS cnt,
    AVG(age) AS avg_age
FROM student
WHERE age >= 18
GROUP BY class_id, gender
HAVING cnt > 3
ORDER BY class_id ASC, cnt DESC
LIMIT 10;
```

## 2.5 连接查询（JOIN）

### JOIN 类型

```
INNER JOIN      返回两表匹配的行
LEFT JOIN       返回左表全部 + 右表匹配的行（无匹配则 NULL）
RIGHT JOIN      返回右表全部 + 左表匹配的行（无匹配则 NULL）
FULL JOIN       MySQL 不支持，用 UNION 模拟
CROSS JOIN      笛卡尔积（慎用）
```

### 示例

```sql
-- 内连接：只显示有分数的学生
SELECT s.name, sc.subject, sc.score
FROM student s
INNER JOIN score sc ON s.id = sc.student_id;

-- 左连接：显示所有学生（含没分数的）
SELECT s.name, sc.subject, sc.score
FROM student s
LEFT JOIN score sc ON s.id = sc.student_id;

-- 自连接：查询同班同学
SELECT s1.name, s2.name AS classmate
FROM student s1
JOIN student s2 ON s1.class_id = s2.class_id
WHERE s1.id != s2.id;

-- 多表连接
SELECT s.name, sc.subject, sc.score, c.name AS class_name
FROM student s
JOIN score sc ON s.id = sc.student_id
JOIN class c ON s.class_id = c.id;
```

## 2.6 子查询

```sql
-- WHERE 子查询（标量）
SELECT * FROM student
WHERE age > (SELECT AVG(age) FROM student);

-- WHERE 子查询（多值）
SELECT * FROM student
WHERE class_id IN (
    SELECT id FROM class WHERE grade = 3
);

-- 作为派生表
SELECT t.class_id, AVG(t.age)
FROM (
    SELECT * FROM student WHERE age > 18
) t
GROUP BY t.class_id;

-- EXISTS 子查询
SELECT * FROM student s
WHERE EXISTS (
    SELECT 1 FROM score sc
    WHERE sc.student_id = s.id AND sc.score > 90
);

-- SELECT 中的子查询（标量子查询）
SELECT name, (
    SELECT AVG(score) FROM score WHERE student_id = s.id
) AS avg_score
FROM student s;
```

## 2.7 联合查询（UNION）

```sql
-- UNION 去重合并
SELECT name FROM student WHERE gender = '男'
UNION
SELECT name FROM student WHERE age > 20;

-- UNION ALL 不去重（更快）
SELECT name FROM student WHERE gender = '男'
UNION ALL
SELECT name FROM student WHERE age > 20;
```

## 2.8 常用查询模板

```sql
-- 分页查询
SELECT * FROM student ORDER BY id LIMIT #{offset}, #{pageSize};

-- 随机取 N 条
SELECT * FROM student ORDER BY RAND() LIMIT 5;

-- 查询重复数据
SELECT email, COUNT(*) FROM student
GROUP BY email HAVING COUNT(*) > 1;

-- CASE WHEN 条件判断
SELECT name, score,
    CASE
        WHEN score >= 90 THEN '优秀'
        WHEN score >= 80 THEN '良好'
        WHEN score >= 60 THEN '及格'
        ELSE '不及格'
    END AS grade
FROM score;
```
