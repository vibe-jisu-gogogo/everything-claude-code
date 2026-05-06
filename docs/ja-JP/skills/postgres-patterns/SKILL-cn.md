---
name: postgres-patterns
description: PostgreSQL database patterns for query optimization, schema design, indexing, and security. Based on Supabase best practices.
---

# PostgreSQL 模式

PostgreSQL最佳实践的快速参考。有关详细指导，请使用 `database-reviewer` 代理。

## 激活时机

- 编写 SQL 查询或 migration 时
- 设计数据库 schema 时
- 排查慢查询问题时
- 实现 Row Level Security 时
- 配置 connection pooling 时

## 快速参考

### 索引速查表

| 查询模式 | 索引类型 | 示例 |
|--------------|------------|---------|
| `WHERE col = value` | B-tree（默认） | `CREATE INDEX idx ON t (col)` |
| `WHERE col > value` | B-tree | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | 复合 | `CREATE INDEX idx ON t (a, b)` |
| `WHERE jsonb @> '{}'` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| `WHERE tsv @@ query` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| 时间序列范围 | BRIN | `CREATE INDEX idx ON t USING brin (col)` |

### 数据类型快速参考

| 用途 | 正确类型 | 应避免的类型 |
|----------|-------------|-------|
| ID | `bigint` | `int`、随机 UUID |
| 字符串 | `text` | `varchar(255)` |
| 时间戳 | `timestamptz` | `timestamp` |
| 金额 | `numeric(10,2)` | `float` |
| 标志 | `boolean` | `varchar`、`int` |

### 通用模式

**复合索引顺序:**
```sql
-- 等值列在前，范围列在后
CREATE INDEX idx ON orders (status, created_at);
-- 适用于: WHERE status = 'pending' AND created_at > '2024-01-01'
```

**覆盖索引:**
```sql
CREATE INDEX idx ON users (email) INCLUDE (name, created_at);
-- 避免 SELECT email, name, created_at 的表扫描
```

**部分索引:**
```sql
CREATE INDEX idx ON users (email) WHERE deleted_at IS NULL;
-- 索引更小，仅包含活跃用户
```

**RLS 策略（优化版）:**
```sql
CREATE POLICY policy ON orders
  USING ((SELECT auth.uid()) = user_id);  -- 用 SELECT 包裹！
```

**UPSERT:**
```sql
INSERT INTO settings (user_id, key, value)
VALUES (123, 'theme', 'dark')
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value;
```

**游标分页:**
```sql
SELECT * FROM products WHERE id > $last_id ORDER BY id LIMIT 20;
-- O(1) vs OFFSET 的 O(n)
```

**队列处理:**
```sql
UPDATE jobs SET status = 'processing'
WHERE id = (
  SELECT id FROM jobs WHERE status = 'pending'
  ORDER BY created_at LIMIT 1
  FOR UPDATE SKIP LOCKED
) RETURNING *;
```

### 反模式检测

```sql
-- 查找没有索引的外键
SELECT conrelid::regclass, a.attname
FROM pg_constraint c
JOIN pg_attribute a ON a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
WHERE c.contype = 'f'
  AND NOT EXISTS (
    SELECT 1 FROM pg_index i
    WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
  );

-- 查找慢查询
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY mean_exec_time DESC;

-- 检查表膨胀
SELECT relname, n_dead_tup, last_vacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

### 配置模板

```sql
-- 连接限制（根据 RAM 调整）
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET work_mem = '8MB';

-- 超时设置
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30s';
ALTER SYSTEM SET statement_timeout = '30s';

-- 监控
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 安全默认值
REVOKE ALL ON SCHEMA public FROM public;

SELECT pg_reload_conf();
```

## 相关资源

- Agent: `database-reviewer` - 完整的数据库审查工作流
- Skill: `clickhouse-io` - ClickHouse 分析模式
- Skill: `backend-patterns` - API 和后端模式

---

*基于 [Supabase Agent Skills](Supabase Agent Skills (credit: Supabase team))（MIT 许可证）*
