# conversation.db - 对话数据库

存储用户对话、消息、附件等核心数据。

## 表结构详情

### 1. conversation - 对话表

存储对话的基础信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 对话 ID |
| name | TEXT | NOT NULL | 对话名称 |
| assistant_id | INTEGER | - | 关联的助手 ID |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

```sql
CREATE TABLE conversation (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    assistant_id INTEGER,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

### 2. message - 消息表

存储对话中的所有消息记录。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 消息 ID |
| conversation_id | INTEGER | NOT NULL | 所属对话 ID |
| message_type | TEXT | NOT NULL | 消息类型 |
| content | TEXT | NOT NULL | 消息内容 |
| llm_model_id | INTEGER | - | 使用的模型 ID |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| token_count | INTEGER | - | Token 总数 |
| parent_id | INTEGER | - | 父消息 ID（用于消息层级） |
| start_time | DATETIME | - | 开始时间 |
| finish_time | DATETIME | - | 完成时间 |
| llm_model_name | TEXT | - | 模型名称 |
| generation_group_id | TEXT | - | 生成组 ID |
| parent_group_id | TEXT | - | 父组 ID |
| tool_calls_json | TEXT | - | 工具调用 JSON |
| input_token_count | INTEGER | DEFAULT 0 | 输入 Token 数 |
| output_token_count | INTEGER | DEFAULT 0 | 输出 Token 数 |
| first_token_time | DATETIME | - | 首个 Token 时间 |
| ttft_ms | INTEGER | - | 首个 Token 耗时（毫秒） |

**索引：**
- `idx_message_conversation_id` - conversation_id
- `idx_message_conversation_created` - (conversation_id, created_time)
- `idx_message_parent_id` - parent_id

```sql
CREATE TABLE message (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    conversation_id INTEGER NOT NULL,
    message_type TEXT NOT NULL,
    content TEXT NOT NULL,
    llm_model_id INTEGER,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    token_count INTEGER,
    parent_id INTEGER,
    start_time DATETIME,
    finish_time DATETIME,
    llm_model_name TEXT,
    generation_group_id TEXT,
    parent_group_id TEXT,
    tool_calls_json TEXT,
    input_token_count INTEGER DEFAULT 0,
    output_token_count INTEGER DEFAULT 0,
    first_token_time DATETIME,
    ttft_ms INTEGER
);

CREATE INDEX idx_message_conversation_id ON message(conversation_id);
CREATE INDEX idx_message_conversation_created ON message(conversation_id, created_time);
CREATE INDEX idx_message_parent_id ON message(parent_id);
```

---

### 3. message_attachment - 消息附件表

存储消息的附件信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 附件 ID |
| message_id | INTEGER | - | 关联消息 ID |
| attachment_type | INTEGER | NOT NULL | 附件类型 |
| attachment_url | TEXT | - | 附件 URL |
| attachment_content | TEXT | - | 附件内容 |
| use_vector | BOOLEAN | DEFAULT 0 NOT NULL | 是否使用向量 |
| token_count | INTEGER | - | Token 数 |
| attachment_hash | TEXT | - | 附件哈希 |

**索引：**
- `idx_message_attachment_message_id` - message_id

```sql
CREATE TABLE message_attachment (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    message_id INTEGER,
    attachment_type INTEGER NOT NULL,
    attachment_url TEXT,
    attachment_content TEXT,
    use_vector BOOLEAN DEFAULT 0 NOT NULL,
    token_count INTEGER,
    attachment_hash TEXT
);

CREATE INDEX idx_message_attachment_message_id ON message_attachment(message_id);
```

---

### 4. acp_session - ACP 会话表

存储 ACP（Agent Control Protocol）会话关联信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| conversation_id | INTEGER | PRIMARY KEY | 对话 ID |
| session_id | TEXT | NOT NULL | ACP 会话 ID |
| updated_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 更新时间 |

```sql
CREATE TABLE acp_session (
    conversation_id INTEGER PRIMARY KEY,
    session_id TEXT NOT NULL,
    updated_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

### 5. sub_task_definition - 子任务定义表

存储子任务的配置和定义。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 任务定义 ID |
| name | TEXT | NOT NULL | 任务名称 |
| code | TEXT | NOT NULL UNIQUE | 任务代码 |
| description | TEXT | NOT NULL | 任务描述 |
| system_prompt | TEXT | NOT NULL | 系统提示词 |
| plugin_source | TEXT | NOT NULL | 插件来源 |
| source_id | INTEGER | NOT NULL | 来源 ID |
| is_enabled | BOOLEAN | DEFAULT 1 NOT NULL | 是否启用 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 更新时间 |

**索引：**
- `idx_sub_task_definition_code` - code
- `idx_sub_task_definition_source` - (plugin_source, source_id)

```sql
CREATE TABLE sub_task_definition (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    code TEXT NOT NULL UNIQUE,
    description TEXT NOT NULL,
    system_prompt TEXT NOT NULL,
    plugin_source TEXT NOT NULL,
    source_id INTEGER NOT NULL,
    is_enabled BOOLEAN NOT NULL DEFAULT 1,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_time DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_sub_task_definition_code ON sub_task_definition(code);
CREATE INDEX idx_sub_task_definition_source ON sub_task_definition(plugin_source, source_id);
```

---

### 6. sub_task_execution - 子任务执行表

存储子任务的执行记录和状态。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 执行记录 ID |
| task_definition_id | INTEGER | NOT NULL | 任务定义 ID |
| task_code | TEXT | NOT NULL | 任务代码 |
| task_name | TEXT | NOT NULL | 任务名称 |
| task_prompt | TEXT | NOT NULL | 任务提示词 |
| parent_conversation_id | INTEGER | NOT NULL | 父对话 ID |
| parent_message_id | INTEGER | - | 父消息 ID |
| status | TEXT | NOT NULL DEFAULT 'pending' | 执行状态（pending/running/success/failed/cancelled） |
| result_content | TEXT | - | 结果内容 |
| error_message | TEXT | - | 错误信息 |
| llm_model_id | INTEGER | - | 使用的模型 ID |
| llm_model_name | TEXT | - | 模型名称 |
| token_count | INTEGER | DEFAULT 0 | Token 总数 |
| input_token_count | INTEGER | DEFAULT 0 | 输入 Token 数 |
| output_token_count | INTEGER | DEFAULT 0 | 输出 Token 数 |
| started_time | DATETIME | - | 开始时间 |
| finished_time | DATETIME | - | 完成时间 |
| created_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| mcp_result_json | TEXT | - | MCP 结果 JSON |

**外键：**
- FOREIGN KEY (task_definition_id) REFERENCES sub_task_definition(id) ON DELETE CASCADE

**索引：**
- `idx_sub_task_execution_conversation` - parent_conversation_id
- `idx_sub_task_execution_message` - parent_message_id
- `idx_sub_task_execution_status` - status

```sql
CREATE TABLE sub_task_execution (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_definition_id INTEGER NOT NULL,
    task_code TEXT NOT NULL,
    task_name TEXT NOT NULL,
    task_prompt TEXT NOT NULL,
    parent_conversation_id INTEGER NOT NULL,
    parent_message_id INTEGER,
    status TEXT NOT NULL DEFAULT 'pending' CHECK(status IN ('pending', 'running', 'success', 'failed', 'cancelled')),
    result_content TEXT,
    error_message TEXT,
    llm_model_id INTEGER,
    llm_model_name TEXT,
    token_count INTEGER DEFAULT 0,
    input_token_count INTEGER DEFAULT 0,
    output_token_count INTEGER DEFAULT 0,
    started_time DATETIME,
    finished_time DATETIME,
    created_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    mcp_result_json TEXT,
    FOREIGN KEY (task_definition_id) REFERENCES sub_task_definition(id) ON DELETE CASCADE
);

CREATE INDEX idx_sub_task_execution_conversation ON sub_task_execution(parent_conversation_id);
CREATE INDEX idx_sub_task_execution_message ON sub_task_execution(parent_message_id);
CREATE INDEX idx_sub_task_execution_status ON sub_task_execution(status);
```

## 表关系图

```
conversation (1) ----< (N) message
                               |
                               v
                        message_attachment

conversation (1) ---- (1) acp_session

sub_task_definition (1) ----< (N) sub_task_execution
                                                    |
                                                    v
                                              conversation
```
