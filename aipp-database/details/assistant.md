# assistant.db - 助手数据库

存储助手配置、模型绑定、提示词、技能关联等。

## 表结构详情

### 1. assistant - 助手表

存储助手的基础信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 助手 ID |
| name | TEXT | NOT NULL | 助手名称 |
| description | TEXT | - | 助手描述 |
| assistant_type | INTEGER | NOT NULL DEFAULT 0 | 助手类型 |
| is_addition | BOOLEAN | NOT NULL DEFAULT 0 | 是否为附加助手 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

```sql
CREATE TABLE assistant (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    assistant_type INTEGER NOT NULL DEFAULT 0,
    is_addition BOOLEAN NOT NULL DEFAULT 0,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

### 2. assistant_model - 助手模型表

存储助手绑定的模型信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 关联 ID |
| assistant_id | INTEGER | NOT NULL | 助手 ID |
| alias | TEXT | - | 模型别名 |
| provider_id | INTEGER | NOT NULL DEFAULT 0 | 提供商 ID |
| model_code | TEXT | NOT NULL DEFAULT '' | 模型代码 |

```sql
CREATE TABLE assistant_model (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER NOT NULL,
    alias TEXT,
    provider_id INTEGER NOT NULL DEFAULT 0,
    model_code TEXT NOT NULL DEFAULT ''
);
```

---

### 3. assistant_prompt - 助手提示词表

存储助手的系统提示词。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 提示词 ID |
| assistant_id | INTEGER | - | 助手 ID |
| prompt | TEXT | NOT NULL | 提示词内容 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (assistant_id) REFERENCES assistant(id)

```sql
CREATE TABLE assistant_prompt (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER,
    prompt TEXT NOT NULL,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (assistant_id) REFERENCES assistant(id)
);
```

---

### 4. assistant_model_config - 助手模型配置表

存储助手的模型参数配置。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| assistant_id | INTEGER | - | 助手 ID |
| assistant_model_id | INTEGER | - | 助手模型 ID |
| name | TEXT | NOT NULL | 配置名称 |
| value | TEXT | - | 配置值 |
| value_type | TEXT | NOT NULL DEFAULT 'float' | 值类型 |

**外键：**
- FOREIGN KEY (assistant_id) REFERENCES assistant(id)

**唯一索引：**
- `idx_assistant_model_config_unique` - (assistant_id, name)

```sql
CREATE TABLE assistant_model_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER,
    assistant_model_id INTEGER,
    name TEXT NOT NULL,
    value TEXT,
    value_type TEXT NOT NULL DEFAULT 'float',
    FOREIGN KEY (assistant_id) REFERENCES assistant(id)
);

CREATE UNIQUE INDEX idx_assistant_model_config_unique ON assistant_model_config(assistant_id, name);
```

---

### 5. assistant_prompt_param - 助手提示词参数表

存储助手提示词的参数配置。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 参数 ID |
| assistant_id | INTEGER | - | 助手 ID |
| assistant_prompt_id | INTEGER | - | 助手提示词 ID |
| param_name | TEXT | NOT NULL | 参数名称 |
| param_type | TEXT | - | 参数类型 |
| param_value | TEXT | - | 参数值 |

**外键：**
- FOREIGN KEY (assistant_id) REFERENCES assistant(id)
- FOREIGN KEY (assistant_prompt_id) REFERENCES assistant_prompt(id)

```sql
CREATE TABLE assistant_prompt_param (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER,
    assistant_prompt_id INTEGER,
    param_name TEXT NOT NULL,
    param_type TEXT,
    param_value TEXT,
    FOREIGN KEY (assistant_id) REFERENCES assistant(id),
    FOREIGN KEY (assistant_prompt_id) REFERENCES assistant_prompt(id)
);
```

---

### 6. assistant_mcp_config - 助手 MCP 配置表

存储助手关联的 MCP 服务器配置。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| assistant_id | INTEGER | NOT NULL | 助手 ID |
| mcp_server_id | INTEGER | NOT NULL | MCP 服务器 ID |
| is_enabled | BOOLEAN | NOT NULL DEFAULT 1 | 是否启用 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (assistant_id) REFERENCES assistant(id) ON DELETE CASCADE

**唯一约束：**
- UNIQUE(assistant_id, mcp_server_id)

**唯一索引：**
- `idx_assistant_mcp_cfg_unique` - (assistant_id, mcp_server_id)

```sql
CREATE TABLE assistant_mcp_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER NOT NULL,
    mcp_server_id INTEGER NOT NULL,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (assistant_id) REFERENCES assistant(id) ON DELETE CASCADE,
    UNIQUE(assistant_id, mcp_server_id)
);

CREATE UNIQUE INDEX idx_assistant_mcp_cfg_unique ON assistant_mcp_config(assistant_id, mcp_server_id);
```

---

### 7. assistant_mcp_tool_config - 助手 MCP 工具配置表

存储助手关联的 MCP 工具配置。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| assistant_id | INTEGER | NOT NULL | 助手 ID |
| mcp_tool_id | INTEGER | NOT NULL | MCP 工具 ID |
| is_enabled | BOOLEAN | NOT NULL DEFAULT 1 | 是否启用 |
| is_auto_run | BOOLEAN | NOT NULL DEFAULT 0 | 是否自动运行 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (assistant_id) REFERENCES assistant(id) ON DELETE CASCADE

**唯一约束：**
- UNIQUE(assistant_id, mcp_tool_id)

**唯一索引：**
- `idx_assistant_mcp_tool_cfg_unique` - (assistant_id, mcp_tool_id)

```sql
CREATE TABLE assistant_mcp_tool_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER NOT NULL,
    mcp_tool_id INTEGER NOT NULL,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    is_auto_run BOOLEAN NOT NULL DEFAULT 0,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (assistant_id) REFERENCES assistant(id) ON DELETE CASCADE,
    UNIQUE(assistant_id, mcp_tool_id)
);

CREATE UNIQUE INDEX idx_assistant_mcp_tool_cfg_unique ON assistant_mcp_tool_config(assistant_id, mcp_tool_id);
```

---

### 8. assistant_skill_config - 助手技能配置表

存储助手的技能配置。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| assistant_id | INTEGER | NOT NULL | 助手 ID |
| skill_identifier | TEXT | NOT NULL | 技能标识符 |
| is_enabled | BOOLEAN | NOT NULL DEFAULT 1 | 是否启用 |
| priority | INTEGER | NOT NULL DEFAULT 0 | 优先级 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (assistant_id) REFERENCES assistant(id) ON DELETE CASCADE

**唯一约束：**
- UNIQUE(assistant_id, skill_identifier)

**索引：**
- `idx_assistant_skill_config_assistant` - assistant_id

```sql
CREATE TABLE assistant_skill_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    assistant_id INTEGER NOT NULL,
    skill_identifier TEXT NOT NULL,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    priority INTEGER NOT NULL DEFAULT 0,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (assistant_id) REFERENCES assistant(id) ON DELETE CASCADE,
    UNIQUE(assistant_id, skill_identifier)
);

CREATE INDEX idx_assistant_skill_config_assistant ON assistant_skill_config(assistant_id);
```

## 表关系图

```
assistant (1) ----< (N) assistant_model
                       |
                       v
              assistant_model_config

assistant (1) ----< (N) assistant_prompt
                       |
                       v
              assistant_prompt_param

assistant (1) ----< (N) assistant_mcp_config
assistant (1) ----< (N) assistant_mcp_tool_config
assistant (1) ----< (N) assistant_skill_config
```
