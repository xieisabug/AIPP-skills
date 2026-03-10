# mcp.db - MCP 服务器数据库

存储 MCP 服务器配置和工具调用历史。

## 表结构详情

### 1. mcp_server - MCP 服务器表

存储 MCP 服务器的配置信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 服务器 ID |
| name | TEXT | NOT NULL UNIQUE | 服务器名称 |
| description | TEXT | - | 服务器描述 |
| transport_type | TEXT | NOT NULL | 传输类型（stdio/sse/http等） |
| command | TEXT | - | 启动命令 |
| environment_variables | TEXT | - | 环境变量（JSON 格式） |
| url | TEXT | - | 服务 URL |
| timeout | INTEGER | DEFAULT 30000 | 超时时间（毫秒） |
| is_long_running | BOOLEAN | DEFAULT 0 NOT NULL | 是否为长连接 |
| is_enabled | BOOLEAN | DEFAULT 1 NOT NULL | 是否启用 |
| is_builtin | BOOLEAN | DEFAULT 0 NOT NULL | 是否为内置服务器 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| headers | TEXT | - | HTTP 头 |
| is_deletable | BOOLEAN | DEFAULT 1 NOT NULL | 是否可删除 |

```sql
CREATE TABLE mcp_server (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    transport_type TEXT NOT NULL,
    command TEXT,
    environment_variables TEXT,
    url TEXT,
    timeout INTEGER DEFAULT 30000,
    is_long_running BOOLEAN NOT NULL DEFAULT 0,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    is_builtin BOOLEAN NOT NULL DEFAULT 0,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    headers TEXT,
    is_deletable BOOLEAN NOT NULL DEFAULT 1
);
```

---

### 2. mcp_server_tool - MCP 工具表

存储 MCP 服务器提供的工具列表。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 工具 ID |
| server_id | INTEGER | NOT NULL | 服务器 ID |
| tool_name | TEXT | NOT NULL | 工具名称 |
| tool_description | TEXT | - | 工具描述 |
| is_enabled | BOOLEAN | DEFAULT 1 NOT NULL | 是否启用 |
| is_auto_run | BOOLEAN | DEFAULT 0 NOT NULL | 是否自动运行 |
| parameters | TEXT | - | 工具参数（JSON 格式） |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE

**唯一约束：**
- UNIQUE(server_id, tool_name)

**唯一索引：**
- `idx_mcp_server_tool_unique` - (server_id, tool_name)

```sql
CREATE TABLE mcp_server_tool (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id INTEGER NOT NULL,
    tool_name TEXT NOT NULL,
    tool_description TEXT,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    is_auto_run BOOLEAN NOT NULL DEFAULT 0,
    parameters TEXT,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE,
    UNIQUE(server_id, tool_name)
);

CREATE UNIQUE INDEX idx_mcp_server_tool_unique ON mcp_server_tool(server_id, tool_name);
```

---

### 3. mcp_server_resource - MCP 资源表

存储 MCP 服务器提供的资源列表。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 资源 ID |
| server_id | INTEGER | NOT NULL | 服务器 ID |
| resource_uri | TEXT | NOT NULL | 资源 URI |
| resource_name | TEXT | NOT NULL | 资源名称 |
| resource_type | TEXT | NOT NULL | 资源类型 |
| resource_description | TEXT | - | 资源描述 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE

**唯一约束：**
- UNIQUE(server_id, resource_uri)

**唯一索引：**
- `idx_mcp_server_resource_unique` - (server_id, resource_uri)

```sql
CREATE TABLE mcp_server_resource (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id INTEGER NOT NULL,
    resource_uri TEXT NOT NULL,
    resource_name TEXT NOT NULL,
    resource_type TEXT NOT NULL,
    resource_description TEXT,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE,
    UNIQUE(server_id, resource_uri)
);

CREATE UNIQUE INDEX idx_mcp_server_resource_unique ON mcp_server_resource(server_id, resource_uri);
```

---

### 4. mcp_server_prompt - MCP 提示词表

存储 MCP 服务器提供的提示词列表。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 提示词 ID |
| server_id | INTEGER | NOT NULL | 服务器 ID |
| prompt_name | TEXT | NOT NULL | 提示词名称 |
| prompt_description | TEXT | - | 提示词描述 |
| is_enabled | BOOLEAN | DEFAULT 1 NOT NULL | 是否启用 |
| arguments | TEXT | - | 参数（JSON 格式） |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**外键：**
- FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE

**唯一约束：**
- UNIQUE(server_id, prompt_name)

**唯一索引：**
- `idx_mcp_server_prompt_unique` - (server_id, prompt_name)

```sql
CREATE TABLE mcp_server_prompt (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id INTEGER NOT NULL,
    prompt_name TEXT NOT NULL,
    prompt_description TEXT,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    arguments TEXT,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE,
    UNIQUE(server_id, prompt_name)
);

CREATE UNIQUE INDEX idx_mcp_server_prompt_unique ON mcp_server_prompt(server_id, prompt_name);
```

---

### 5. mcp_tool_call - MCP 工具调用记录表

存储 MCP 工具的调用历史。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 调用记录 ID |
| conversation_id | INTEGER | NOT NULL | 对话 ID |
| message_id | INTEGER | - | 消息 ID |
| server_id | INTEGER | NOT NULL | 服务器 ID |
| server_name | TEXT | NOT NULL | 服务器名称 |
| tool_name | TEXT | NOT NULL | 工具名称 |
| parameters | TEXT | NOT NULL | 调用参数（JSON 格式） |
| status | TEXT | NOT NULL DEFAULT 'pending' | 调用状态（pending/executing/success/failed） |
| result | TEXT | - | 调用结果 |
| error | TEXT | - | 错误信息 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| started_time | DATETIME | - | 开始时间 |
| finished_time | DATETIME | - | 完成时间 |
| llm_call_id | TEXT | - | LLM 调用 ID |
| assistant_message_id | INTEGER | - | 助手消息 ID |
| subtask_id | INTEGER | - | 子任务 ID |

**外键：**
- FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE

```sql
CREATE TABLE mcp_tool_call (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    conversation_id INTEGER NOT NULL,
    message_id INTEGER,
    server_id INTEGER NOT NULL,
    server_name TEXT NOT NULL,
    tool_name TEXT NOT NULL,
    parameters TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'executing', 'success', 'failed')),
    result TEXT,
    error TEXT,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    started_time DATETIME,
    finished_time DATETIME,
    llm_call_id TEXT,
    assistant_message_id INTEGER,
    subtask_id INTEGER,
    FOREIGN KEY (server_id) REFERENCES mcp_server(id) ON DELETE CASCADE
);
```

## 表关系图

```
mcp_server (1) ----< (N) mcp_server_tool
mcp_server (1) ----< (N) mcp_server_resource
mcp_server (1) ----< (N) mcp_server_prompt
mcp_server (1) ----< (N) mcp_tool_call
```

## 常用查询示例

### 查询所有启用的 MCP 服务器及其工具数量

```sql
SELECT
    s.id,
    s.name,
    s.description,
    s.transport_type,
    s.is_enabled,
    COUNT(t.id) as tool_count
FROM mcp_server s
LEFT JOIN mcp_server_tool t ON s.id = t.server_id
WHERE s.is_enabled = 1
GROUP BY s.id
ORDER BY s.id;
```

### 查询失败的工具调用

```sql
SELECT
    id,
    conversation_id,
    server_name,
    tool_name,
    error,
    created_time
FROM mcp_tool_call
WHERE status = 'failed'
ORDER BY created_time DESC;
```

### 查询特定服务器的所有可用资源

```sql
SELECT
    resource_uri,
    resource_name,
    resource_type,
    resource_description
FROM mcp_server_resource
WHERE server_id = ?
ORDER BY resource_name;
```

### 查询工具调用的执行时间统计

```sql
SELECT
    server_name,
    tool_name,
    COUNT(*) as call_count,
    AVG(julianday(finished_time) - julianday(started_time)) * 86400000 as avg_duration_ms,
    SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) as success_count,
    SUM(CASE WHEN status = 'failed' THEN 1 ELSE 0 END) as failed_count
FROM mcp_tool_call
WHERE started_time IS NOT NULL AND finished_time IS NOT NULL
GROUP BY server_name, tool_name
ORDER BY call_count DESC;
```
