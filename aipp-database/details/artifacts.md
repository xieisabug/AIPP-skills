# artifacts.db - 产物数据库

存储可预览的代码产物和模板。

## 表结构详情

### 1. artifacts_collection - 产物集合表

存储各类代码产物和模板。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 产物 ID |
| name | TEXT | NOT NULL | 产物名称 |
| icon | TEXT | NOT NULL | 产物图标 |
| description | TEXT | NOT NULL DEFAULT '' | 产物描述 |
| artifact_type | TEXT | NOT NULL | 产物类型（vue/react/html/svg/xml/markdown/mermaid） |
| code | TEXT | NOT NULL | 产物代码 |
| tags | TEXT | - | 标签（JSON 格式） |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| last_used_time | DATETIME | - | 最后使用时间 |
| use_count | INTEGER | NOT NULL DEFAULT 0 | 使用次数 |

**约束：**
- CHECK (artifact_type IN ('vue', 'react', 'html', 'svg', 'xml', 'markdown', 'mermaid'))

**索引：**
- `idx_artifacts_collection_type` - artifact_type
- `idx_artifacts_collection_name` - name

```sql
CREATE TABLE artifacts_collection (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    icon TEXT NOT NULL,
    description TEXT NOT NULL DEFAULT '',
    artifact_type TEXT NOT NULL CHECK (artifact_type IN ('vue', 'react', 'html', 'svg', 'xml', 'markdown', 'mermaid')),
    code TEXT NOT NULL,
    tags TEXT,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_used_time DATETIME,
    use_count INTEGER NOT NULL DEFAULT 0
);

CREATE INDEX idx_artifacts_collection_type ON artifacts_collection(artifact_type);
CREATE INDEX idx_artifacts_collection_name ON artifacts_collection(name);
```

## 常用查询示例

### 查询所有产物类型

```sql
SELECT DISTINCT artifact_type
FROM artifacts_collection
ORDER BY artifact_type;
```

### 查询特定类型的产物

```sql
SELECT
    id,
    name,
    description,
    use_count,
    created_time
FROM artifacts_collection
WHERE artifact_type = ?
ORDER BY use_count DESC, created_time DESC;
```

### 查询最常使用的产物

```sql
SELECT
    id,
    name,
    artifact_type,
    use_count,
    last_used_time
FROM artifacts_collection
ORDER BY use_count DESC
LIMIT 20;
```

### 按标签搜索产物

```sql
SELECT
    id,
    name,
    artifact_type,
    description,
    tags
FROM artifacts_collection
WHERE tags LIKE ?
ORDER BY use_count DESC;
```

### 查询最近使用的产物

```sql
SELECT
    id,
    name,
    artifact_type,
    last_used_time,
    use_count
FROM artifacts_collection
WHERE last_used_time IS NOT NULL
ORDER BY last_used_time DESC
LIMIT 10;
```

### 搜索产物（按名称或描述）

```sql
SELECT
    id,
    name,
    artifact_type,
    description
FROM artifacts_collection
WHERE name LIKE ? OR description LIKE ?
ORDER BY use_count DESC;
```

### 更新产物使用统计

```sql
UPDATE artifacts_collection
SET use_count = use_count + 1,
    last_used_time = CURRENT_TIMESTAMP
WHERE id = ?;
```

### 获取产物统计信息

```sql
SELECT
    artifact_type,
    COUNT(*) as count,
    SUM(use_count) as total_uses,
    AVG(use_count) as avg_uses
FROM artifacts_collection
GROUP BY artifact_type
ORDER BY total_uses DESC;
```
