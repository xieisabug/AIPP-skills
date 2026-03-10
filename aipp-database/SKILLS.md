---
name: AIPP Database
description: AIPP（也就是当前程序）的数据库介绍，能够直接对程序的各种数据进行修改和维护。
version: 1.0.0
author: Xieisabug
---

# AIPP 数据库介绍

AIPP 使用 SQLite 数据库存储所有用户数据，数据按功能域分离到不同的数据库文件中。

## 数据库文件位置

所有数据库文件位于用户数据目录下的 `db/` 子目录：
- **macOS**: `~/Library/Application Support/com.xieisabug.aipp/db/`
- **Windows**: `%APPDATA%\com.xieisabug.aipp\db\`
- **Linux**: `~/.config/com.xieisabug.aipp/db/`

---

## 1. conversation.db - 对话数据库

存储用户对话、消息、附件等核心数据。

### 表列表

| 表名 | 说明 |
|------|------|
| `conversation` | 对话基础信息（名称、关联助手、创建时间） |
| `message` | 消息记录（内容、类型、token 统计、时间、工具调用） |
| `message_attachment` | 消息附件（文件 URL、内容、哈希、向量标记） |
| `acp_session` | ACP会话关联 |
| `sub_task_definition` | 子任务定义（任务配置、系统提示词、来源） |
| `sub_task_execution` | 子任务执行记录（状态、结果、token 统计） |

---

## 2. assistant.db - 助手数据库

存储助手配置、模型绑定、提示词、技能关联等。

### 表列表

| 表名 | 说明 |
|------|------|
| `assistant` | 助手基础信息（名称、描述、类型） |
| `assistant_model` | 助手绑定的模型（别名、提供商、模型代码） |
| `assistant_prompt` | 助手提示词 |
| `assistant_model_config` | 助手模型配置参数（温度、max_tokens 等） |
| `assistant_prompt_param` | 助手提示词参数 |
| `assistant_mcp_config` | 助手关联的 MCP 服务器 |
| `assistant_mcp_tool_config` | 助手关联的 MCP 工具配置 |
| `assistant_skill_config` | 助手技能配置（启用状态、优先级） |

---

## 3. llm.db - 大模型数据库

存储 LLM 提供商配置和可用模型。

### 表列表

| 表名 | 说明 |
|------|------|
| `llm_provider` | LLM 提供商（OpenAI、Ollama、Anthropic、DeepSeek、ACP 等） |
| `llm_model` | 模型信息（名称、代码、能力支持） |
| `llm_provider_config` | 提供商配置（API Key、Base URL 等） |

---

## 4. mcp.db - MCP 服务器数据库

存储 MCP 服务器配置和工具调用历史。

### 表列表

| 表名 | 说明 |
|------|------|
| `mcp_server` | MCP 服务器配置（传输类型、命令、URL、启用状态） |
| `mcp_server_tool` | MCP 工具列表（名称、描述、参数） |
| `mcp_server_resource` | MCP 资源列表 |
| `mcp_server_prompt` | MCP 提示词列表 |
| `mcp_tool_call` | MCP 工具调用记录（状态、结果、错误信息） |

---

## 5. plugin.db - 插件数据库

存储插件相关数据和配置。

### 表列表

| 表名 | 说明 |
|------|------|
| `Plugins` | 插件信息（名称、版本、作者） |
| `PluginStatus` | 插件状态（是否激活、最后运行时间） |
| `PluginConfigurations` | 插件配置参数 |
| `PluginData` | 插件存储的数据（按会话隔离） |

---

## 6. system.db - 系统配置数据库

存储系统级和功能级配置。

### 表列表

| 表名 | 说明 |
|------|------|
| `system_config` | 系统配置（键值对存储） |
| `feature_config` | 功能配置（按功能代码组织的配置项） |

---

## 7. artifacts.db - 产物数据库

存储可复用的代码产物和模板。

### 表列表

| 表名 | 说明 |
|------|------|
| `artifacts_collection` | 产物集合（HTML、React、Vue、SVG、Markdown、Mermaid 等） |

---

## 常用查询示例

### 查询所有对话及其消息数

```sql
SELECT
    c.id,
    c.name,
    c.created_time,
    COUNT(m.id) as message_count
FROM conversation c
LEFT JOIN message m ON c.id = m.conversation_id
GROUP BY c.id
ORDER BY c.created_time DESC;
```

### 查询指定对话的所有消息

```sql
SELECT
    id,
    message_type,
    content,
    created_time
FROM message
WHERE conversation_id = ?
ORDER BY created_time;
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

### 查询所有启用的 MCP 服务器

```sql
SELECT
    id,
    name,
    description,
    transport_type,
    is_enabled
FROM mcp_server
WHERE is_enabled = 1;
```

### 查询对话的 token 使用统计

```sql
SELECT
    c.id,
    c.name,
    SUM(m.input_token_count) as total_input,
    SUM(m.output_token_count) as total_output,
    SUM(m.token_count) as total_tokens
FROM conversation c
LEFT JOIN message m ON c.id = m.conversation_id
GROUP BY c.id;
```

---

## 使用建议

- 在 details 文件夹中，有各个数据库的详细介绍，可以直接了解表结构
- 可以使用 Python、Bun（在用户数据目录的bun目录）、Node.js 等编程语言和工具，结合 SQLite 库进行数据库操作，如果没有安装 SQLite 相关的库可以进行安装。
- 进行高质量的查询语句编写，谨慎的进行修改，如果需要修改记录，尝试先创建一些备份数据，可以创建备份数据库文件，然后插入对应的数据。
- 尽量进行逻辑删除