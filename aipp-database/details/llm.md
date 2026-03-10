# llm.db - 大模型数据库

存储 LLM 提供商配置和可用模型。

## 表结构详情

### 1. llm_provider - LLM 提供商表

存储 LLM 服务提供商的基础信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 提供商 ID |
| name | TEXT | NOT NULL | 提供商名称 |
| api_type | TEXT | NOT NULL | API 类型 |
| description | TEXT | - | 提供商描述 |
| is_official | BOOLEAN | DEFAULT 0 NOT NULL | 是否为官方提供商 |
| is_enabled | BOOLEAN | DEFAULT 0 NOT NULL | 是否启用 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

```sql
CREATE TABLE llm_provider (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    api_type TEXT NOT NULL,
    description TEXT,
    is_official BOOLEAN NOT NULL DEFAULT 0,
    is_enabled BOOLEAN NOT NULL DEFAULT 0,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

### 2. llm_provider_config - LLM 提供商配置表

存储 LLM 提供商的配置参数（如 API Key、Base URL 等）。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| name | TEXT | NOT NULL | 配置名称 |
| llm_provider_id | INTEGER | NOT NULL | 提供商 ID |
| value | TEXT | - | 配置值 |
| append_location | TEXT | DEFAULT 'header' | 追加位置（header/body/query） |
| is_addition | BOOLEAN | DEFAULT 0 NOT NULL | 是否为附加配置 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

```sql
CREATE TABLE llm_provider_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    llm_provider_id INTEGER NOT NULL,
    value TEXT,
    append_location TEXT DEFAULT 'header',
    is_addition BOOLEAN NOT NULL DEFAULT 0,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

### 3. llm_model - LLM 模型表

存储可用的 LLM 模型信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 模型 ID |
| name | TEXT | NOT NULL | 模型名称 |
| llm_provider_id | INTEGER | NOT NULL | 提供商 ID |
| code | TEXT | NOT NULL | 模型代码 |
| description | TEXT | - | 模型描述 |
| vision_support | BOOLEAN | DEFAULT 0 NOT NULL | 是否支持视觉 |
| audio_support | BOOLEAN | DEFAULT 0 NOT NULL | 是否支持音频 |
| video_support | BOOLEAN | DEFAULT 0 NOT NULL | 是否支持视频 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (llm_provider_id) REFERENCES llm_provider(id)

```sql
CREATE TABLE llm_model (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    llm_provider_id INTEGER NOT NULL REFERENCES llm_provider,
    code TEXT NOT NULL,
    description TEXT,
    vision_support BOOLEAN DEFAULT 0 NOT NULL,
    audio_support BOOLEAN DEFAULT 0 NOT NULL,
    video_support BOOLEAN DEFAULT 0 NOT NULL,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

## 表关系图

```
llm_provider (1) ----< (N) llm_model
                         |
                         v
              llm_provider_config
```

## 常用查询示例

### 查询所有启用的提供商及其模型

```sql
SELECT
    p.id as provider_id,
    p.name as provider_name,
    p.api_type,
    m.id as model_id,
    m.name as model_name,
    m.code as model_code
FROM llm_provider p
LEFT JOIN llm_model m ON p.id = m.llm_provider_id
WHERE p.is_enabled = 1
ORDER BY p.id, m.id;
```

### 查询支持视觉功能的模型

```sql
SELECT
    m.id,
    m.name,
    m.code,
    p.name as provider_name
FROM llm_model m
JOIN llm_provider p ON m.llm_provider_id = p.id
WHERE m.vision_support = 1 AND p.is_enabled = 1;
```

### 查询提供商的配置

```sql
SELECT
    p.name as provider_name,
    c.name as config_name,
    c.value,
    c.append_location
FROM llm_provider p
JOIN llm_provider_config c ON p.id = c.llm_provider_id
WHERE p.is_enabled = 1
ORDER BY p.id, c.id;
```
