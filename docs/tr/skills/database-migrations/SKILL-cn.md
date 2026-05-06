---
name: database-migrations
description: 数据库迁移最佳实践，适用于 schema 更改、数据迁移、回滚以及 PostgreSQL、MySQL 和常见 ORM（Prisma、Drizzle、Django、TypeORM、golang-migrate）之间的零停机部署。
origin: ECC
---

# 数据库迁移模式

针对生产系统的安全、可回滚的数据库 schema 更改。

## 何时激活

- 创建或修改数据库表时
- 添加/删除列或索引时
- 运行数据迁移时（backfill、转换）
- 规划零停机 schema 更改时
- 为新项目设置迁移工具时

## 核心原则

1. **每次更改都是一次迁移** — 永远不要手动修改生产数据库
2. **迁移在生产中仅向前** — 回滚使用新的 forward 迁移
3. **Schema 和数据迁移分离** — 永远不要在单个迁移中混合 DDL 和 DML
4. **针对生产规模数据测试迁移** — 在 100 行上运行的迁移在 10M 行时可能会锁定
5. **迁移在生产运行后不可变** — 永远不要编辑已在生产中运行的迁移

## 迁移安全检查清单

在应用任何迁移之前：

- [ ] 迁移具有 UP 和 DOWN（或明确标记为不可回滚）
- [ ] 在大型表上没有全表锁（使用 concurrent 操作）
- [ ] 新列具有默认值或可为 nullable（切勿添加无默认值的 NOT NULL）
- [ ] 索引正在 concurrent 创建（对于现有表，不是在 CREATE TABLE 中内联）
- [ ] 数据 backfill 是与 schema 更改分离的单独迁移
- [ ] 已针对生产数据副本进行测试
- [ ] 回滚计划已记录

## PostgreSQL 模式

### 安全添加列

```sql
-- 好：Nullable 列，无锁
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- 好：带默认值的列（Postgres 11+ 即时，无需重写）
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- 坏：在现有表上添加无默认值的 NOT NULL（需要完全重写）
ALTER TABLE users ADD COLUMN role TEXT NOT NULL;
-- 这会锁定表并重写每一行
```

### 零停机添加索引

```sql
-- 坏：在大型表上会阻止写入
CREATE INDEX idx_users_email ON users (email);

-- 好：不会阻塞，允许并发写入
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);

-- 注意：CONCURRENTLY 不能在事务块内运行
-- 大多数迁移工具需要对此进行特殊处理
```

### 列重命名（零停机）

永远不要在生产中直接重命名。使用 expand-contract 模式：

```sql
-- 步骤 1：添加新列（迁移 001）
ALTER TABLE users ADD COLUMN display_name TEXT;

-- 步骤 2：Backfill 数据（迁移 002，数据迁移）
UPDATE users SET display_name = username WHERE display_name IS NULL;

-- 步骤 3：更新应用代码以读取/写入两列
-- 部署应用更改

-- 步骤 4：停止写入旧列，删除（迁移 003）
ALTER TABLE users DROP COLUMN username;
```

### 安全删除列

```sql
-- 步骤 1：从代码中删除所有应用对该列的引用
-- 步骤 2：部署无列引用的应用
-- 步骤 3：在后续迁移中删除列
ALTER TABLE orders DROP COLUMN legacy_status;

-- 对于 Django：使用 SeparateDatabaseAndState 从模型中移除
-- 不生成 DROP COLUMN（然后在下一次迁移中删除）
```

### 大数据迁移

```sql
-- 坏：在单个事务中更新所有行（锁定表）
UPDATE users SET normalized_email = LOWER(email);

-- 好：带进度的批量更新
DO $$
DECLARE
  batch_size INT := 10000;
  rows_updated INT;
BEGIN
  LOOP
    UPDATE users
    SET normalized_email = LOWER(email)
    WHERE id IN (
      SELECT id FROM users
      WHERE normalized_email IS NULL
      LIMIT batch_size
      FOR UPDATE SKIP LOCKED
    );
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    RAISE NOTICE 'Updated % rows', rows_updated;
    EXIT WHEN rows_updated = 0;
    COMMIT;
  END LOOP;
END $$;
```

## Prisma (TypeScript/Node.js)

### 工作流程

```bash
# 从 schema 更改创建迁移
npx prisma migrate dev --name add_user_avatar

# 在生产中应用待处理的迁移
npx prisma migrate deploy

# 重置数据库（仅开发）
npx prisma migrate reset

# 在 schema 更改后生成 client
npx prisma generate
```

### Schema 示例

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  avatarUrl String?  @map("avatar_url")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  orders    Order[]

  @@map("users")
  @@index([email])
}
```

### 自定义 SQL 迁移

对于 Prisma 无法表达的操作（concurrent 索引、数据 backfill）：

```bash
# 创建空迁移，然后手动编辑 SQL
npx prisma migrate dev --create-only --name add_email_index
```

```sql
-- migrations/20240115_add_email_index/migration.sql
-- Prisma 无法生成 CONCURRENTLY，所以我们手动编写
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email ON users (email);
```

## Drizzle (TypeScript/Node.js)

### 工作流程

```bash
# 从 schema 更改生成迁移
npx drizzle-kit generate

# 应用迁移
npx drizzle-kit migrate

# 直接推送 schema（仅开发，无迁移文件）
npx drizzle-kit push
```

### Schema 示例

```typescript
import { pgTable, text, timestamp, uuid, boolean } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: uuid("id").primaryKey().defaultRandom(),
  email: text("email").notNull().unique(),
  name: text("name"),
  isActive: boolean("is_active").notNull().default(true),
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
});
```

## Django (Python)

### 工作流程

```bash
# 从模型更改创建迁移
python manage.py makemigrations

# 应用迁移
python manage.py migrate

# 显示迁移状态
python manage.py showmigrations

# 为自定义 SQL 创建空迁移
python manage.py makemigrations --empty app_name -n description
```

### 数据迁移

```python
from django.db import migrations

def backfill_display_names(apps, schema_editor):
    User = apps.get_model("accounts", "User")
    batch_size = 5000
    users = User.objects.filter(display_name="")
    while users.exists():
        batch = list(users[:batch_size])
        for user in batch:
            user.display_name = user.username
        User.objects.bulk_update(batch, ["display_name"], batch_size=batch_size)

def reverse_backfill(apps, schema_editor):
    pass  # 数据迁移，无需回滚

class Migration(migrations.Migration):
    dependencies = [("accounts", "0015_add_display_name")]

    operations = [
        migrations.RunPython(backfill_display_names, reverse_backfill),
    ]
```

## golang-migrate (Go)

### 工作流程

```bash
# 创建迁移对
migrate create -ext sql -dir migrations -seq add_user_avatar

# 应用所有待处理的迁移
migrate -path migrations -database "$DATABASE_URL" up

# 回滚最后一次迁移
migrate -path migrations -database "$DATABASE_URL" down 1

# 强制版本（修复 dirty 状态）
migrate -path migrations -database "$DATABASE_URL" force VERSION
```

### 迁移文件

```sql
-- migrations/000003_add_user_avatar.up.sql
ALTER TABLE users ADD COLUMN avatar_url TEXT;
CREATE INDEX CONCURRENTLY idx_users_avatar ON users (avatar_url) WHERE avatar_url IS NOT NULL;

-- migrations/000003_add_user_avatar.down.sql
DROP INDEX IF EXISTS idx_users_avatar;
ALTER TABLE users DROP COLUMN IF EXISTS avatar_url;
```

## 零停机迁移策略

对于关键生产更改，遵循 expand-contract 模式：

```
阶段 1：EXPAND
  - 添加新列/表（nullable 或带默认值）
  - 部署：应用同时写入 旧 和 新
  - Backfill 现有数据

阶段 2：MIGRATE
  - 部署：应用从 新 读取，写入两者
  - 验证数据一致性

阶段 3：CONTRACT
  - 部署：应用仅使用 新
  - 在单独的迁移中删除旧列/表
```

### 时间表示例

```
第 1 天：迁移添加 new_status 列（nullable）
第 1 天：部署 App v2 — 写入 status 和 new_status
第 2 天：为现有行运行 backfill 迁移
第 3 天：部署 App v3 — 仅从 new_status 读取
第 7 天：迁移删除旧的 status 列
```

## 反模式

| 反模式 | 为什么会失败 | 更好的方法 |
|---------|-------------|------------|
| 生产中的手动 SQL | 无审计跟踪，不可重复 | 始终使用迁移文件 |
| 编辑已部署的迁移 | 在环境之间造成漂移 | 改为创建新迁移 |
| 无默认值的 NOT NULL | 锁定表，重写所有行 | 添加 nullable，backfill，然后添加约束 |
| 在大表中内联索引 | 在构建期间阻止写入 | CREATE INDEX CONCURRENTLY |
| 单个迁移中的 schema + 数据 | 回滚困难，长事务 | 分离迁移 |
| 在删除代码之前删除列 | 应用在缺少列时出错 | 先删除代码，然后在后续部署中删除列 |
