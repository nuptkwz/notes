@[toc]

> 本手册介绍了MySQL的基本概念及使用，覆盖90%日常场景，建议在测试库实操练习。**重要原则**：生产环境变更前先备份、权限最小化、事务保一致。遇到复杂问题及时联系DBA！

## 一、核心概念速览
1. **数据库（Database）**  
   数据的逻辑容器（如`订单库`、`用户库`）。
2. **表（Table）**  
   结构化数据的存储单元，由行（记录）和列（字段）组成。
    - **字段约束**：`PRIMARY KEY`（主键）、`AUTO_INCREMENT`（自增）、`NOT NULL`（非空）
3. **常用数据类型**  
   | 类型          | 示例                  | 用途                     |  
   |---------------|-----------------------|--------------------------|  
   | `INT`         | `age INT`             | 整数                     |  
   | `VARCHAR(50)` | `name VARCHAR(50)`    | 变长字符串（最长50字符） |  
   | `DATETIME`    | `created_at DATETIME` | 日期和时间               |  
   | `DECIMAL(10,2)`| `price DECIMAL(10,2)` | 精确小数（如金额）       |

---

## 二、基础操作命令
### 1. 数据库操作
```sql
CREATE DATABASE mydb;                 -- 创建数据库  
USE mydb;                             -- 切换到mydb  
SHOW DATABASES;                       -- 查看所有数据库  
DROP DATABASE mydb;                   -- 删除数据库（慎用！）
```

### 2. 表操作
```sql
-- 创建表（含自增主键和默认值）
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP  -- 自动记录创建时间
) DEFAULT CHARSET=utf8mb4;            -- 支持中文和Emoji

-- 修改表结构
ALTER TABLE users ADD COLUMN age INT;      -- 添加列  
ALTER TABLE users DROP COLUMN age;          -- 删除列  
RENAME TABLE users TO members;              -- 重命名表
```

### 3. 数据操作（CRUD）
```sql
-- 增：插入数据
INSERT INTO users (name) VALUES ('张三');  

-- 查：进阶查询
SELECT * FROM users WHERE name LIKE '%张%';   -- 模糊查询  
SELECT name, COUNT(*) FROM users GROUP BY name HAVING COUNT(*) > 1; -- 分组统计

-- 改：更新数据
UPDATE users SET name='李四' WHERE id=1;  

-- 删：删除数据
DELETE FROM users WHERE id=2;               -- 必加WHERE条件！
```

> **避坑提示**：
> - 生产环境执行`DELETE`/`UPDATE`前先用`SELECT`验证条件，避免误操作全表
> - 删除表前先备份：`CREATE TABLE backup AS SELECT * FROM users;`

---

## 三、运维与管理实战
### 1. 用户与权限控制
```sql
-- 创建用户（允许远程连接）
CREATE USER 'dev'@'%' IDENTIFIED BY 'Pass123!'; 
-- 最小化授权（只给必要权限）
GRANT SELECT, INSERT ON mydb.* TO 'dev'@'%';  
FLUSH PRIVILEGES;                          -- 刷新权限

-- 权限回收
REVOKE INSERT ON mydb.* FROM 'dev'@'%';    -- 撤销插入权限
```

### 2. 数据备份与恢复
```bash
# 备份整个数据库（终端执行）
mysqldump -u root -p mydb > mydb_backup.sql    
# 仅备份表结构
mysqldump -u root -p -d mydb > schema.sql     

# 恢复数据
mysql -u root -p mydb < mydb_backup.sql      
```

### 3. 事务与锁机制
```sql
BEGIN;                      -- 开启事务
UPDATE account SET balance=balance-100 WHERE id=1;
UPDATE account SET balance=balance+100 WHERE id=2;
COMMIT;                     -- 提交事务（或ROLLBACK回滚）
```
> **注意**：事务用于保证数据一致性（如转账操作），避免中间状态被其他查询读到。

---

## 四、效率优化技巧
1. **索引优化**
   ```sql
   CREATE INDEX idx_name ON users(name);    -- 为name字段创建索引
   ```
    - 索引规则：频繁查询的字段（如`WHERE`条件列）建议加索引，但避免过度索引影响写入速度。

2. **查询优化**
    - 用`LIMIT`分页：`SELECT * FROM users LIMIT 10 OFFSET 20;`（避免全表扫描）
    - 用`EXPLAIN`分析慢查询：
      ```sql
      EXPLAIN SELECT * FROM users WHERE age>30;  -- 查看执行计划
      ```

---

## 五、安全与避坑指南
1. **字符集统一**  
   建表时强制设置`DEFAULT CHARSET=utf8mb4`，避免中文乱码。
2. **密码安全**
    - 修改密码：`ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';`
    - 避免明文密码：命令行用`-p`不接密码，交互式输入更安全。
3. **防误删保护**
    - 启用`--safe-updates`模式：禁止无`WHERE`的`DELETE`/`UPDATE`。
4. **连接问题排查**
   ```bash
   mysql -h 192.168.1.100 -P 3306 -u dev -p  # 远程连接
   netstat -na | findstr 3306                 # 检查端口占用（Windows）
   ```

---

**附：学习资源**
- 官方文档：https://dev.mysql.com/doc/
- 交互练习：https://sqlbolt.com/（实时练习SQL语法）
- 备份脚本生成器：https://www.mysqltuner.com/（性能优化建议）

更多技术干货欢迎关注微信公众号“**风雨同舟的AI笔记**”~

【转载须知】：<font color=red>**转载请注明原文出处及作者信息**</font>