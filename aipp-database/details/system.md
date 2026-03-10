# system.db - 系统配置数据库

存储系统级和功能级配置。

## 表结构详情

### 1. system_config - 系统配置表

存储系统级的键值对配置。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| key | TEXT | NOT NULL UNIQUE | 配置键 |
| value | TEXT | NOT NULL | 配置值 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

```sql
CREATE TABLE system_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    key TEXT NOT NULL UNIQUE,
    value TEXT NOT NULL,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

### 2. feature_config - 功能配置表

存储按功能代码组织的配置项。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| feature_code | TEXT | NOT NULL | 功能代码 |
| key | TEXT | NOT NULL | 配置键 |
| value | TEXT | - | 配置值 |
| data_type | TEXT | - | 数据类型 |
| description | TEXT | - | 配置描述 |

**唯一约束：**
- UNIQUE(feature_code, key)

**唯一索引：**
- `idx_feature_config_unique` - (feature_code, key)

```sql
CREATE TABLE feature_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    feature_code TEXT NOT NULL,
    key TEXT NOT NULL,
    value TEXT,
    data_type TEXT,
    description TEXT,
    UNIQUE(feature_code, key)
);

CREATE UNIQUE INDEX idx_feature_config_unique ON feature_config(feature_code, key);
```

## 常用查询示例

### 查询所有系统配置

```sql
SELECT
    key,
    value
FROM system_config
ORDER BY key;
```

### 查询特定功能的配置

```sql
SELECT
    key,
    value,
    data_type,
    description
FROM feature_config
WHERE feature_code = ?
ORDER BY key;
```

### 查询特定配置项

```sql
SELECT value
FROM system_config
WHERE key = ?;
```

### 查询所有功能代码

```sql
SELECT DISTINCT feature_code
FROM feature_config
ORDER BY feature_code;
```

### 批量查询功能的配置

```sql
SELECT
    feature_code,
    key,
    value,
    data_type
FROM feature_config
WHERE feature_code IN ('feature1', 'feature2', 'feature3')
ORDER BY feature_code, key;
```
