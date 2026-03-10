# plugin.db - 插件数据库

存储插件相关数据和配置。

## 表结构详情

### 1. Plugins - 插件表

存储插件的基础信息。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| plugin_id | INTEGER | PRIMARY KEY AUTOINCREMENT | 插件 ID |
| name | TEXT | NOT NULL | 插件名称 |
| version | TEXT | NOT NULL | 插件版本 |
| folder_name | TEXT | NOT NULL | 插件文件夹名称 |
| description | TEXT | - | 插件描述 |
| author | TEXT | - | 插件作者 |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 更新时间 |

```sql
CREATE TABLE Plugins (
    plugin_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    folder_name TEXT NOT NULL,
    description TEXT,
    author TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### 2. PluginStatus - 插件状态表

存储插件的运行状态。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| status_id | INTEGER | PRIMARY KEY AUTOINCREMENT | 状态 ID |
| plugin_id | INTEGER | - | 插件 ID |
| is_active | INTEGER | DEFAULT 1 | 是否激活（0/1） |
| last_run | TIMESTAMP | - | 最后运行时间 |

**外键：**
- FOREIGN KEY (plugin_id) REFERENCES Plugins(plugin_id)

```sql
CREATE TABLE PluginStatus (
    status_id INTEGER PRIMARY KEY AUTOINCREMENT,
    plugin_id INTEGER,
    is_active INTEGER DEFAULT 1,
    last_run TIMESTAMP,
    FOREIGN KEY (plugin_id) REFERENCES Plugins(plugin_id)
);
```

---

### 3. PluginConfigurations - 插件配置表

存储插件的配置参数。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| config_id | INTEGER | PRIMARY KEY AUTOINCREMENT | 配置 ID |
| plugin_id | INTEGER | - | 插件 ID |
| config_key | TEXT | NOT NULL | 配置键 |
| config_value | TEXT | - | 配置值 |

**外键：**
- FOREIGN KEY (plugin_id) REFERENCES Plugins(plugin_id)

```sql
CREATE TABLE PluginConfigurations (
    config_id INTEGER PRIMARY KEY AUTOINCREMENT,
    plugin_id INTEGER,
    config_key TEXT NOT NULL,
    config_value TEXT,
    FOREIGN KEY (plugin_id) REFERENCES Plugins(plugin_id)
);
```

---

### 4. PluginData - 插件数据表

存储插件运行时产生的数据（按会话隔离）。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| data_id | INTEGER | PRIMARY KEY AUTOINCREMENT | 数据 ID |
| plugin_id | INTEGER | - | 插件 ID |
| session_id | TEXT | NOT NULL | 会话 ID |
| data_key | TEXT | NOT NULL | 数据键 |
| data_value | TEXT | - | 数据值 |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 更新时间 |

**外键：**
- FOREIGN KEY (plugin_id) REFERENCES Plugins(plugin_id)

```sql
CREATE TABLE PluginData (
    data_id INTEGER PRIMARY KEY AUTOINCREMENT,
    plugin_id INTEGER,
    session_id TEXT NOT NULL,
    data_key TEXT NOT NULL,
    data_value TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (plugin_id) REFERENCES Plugins(plugin_id)
);
```

## 表关系图

```
Plugins (1) ----< (1) PluginStatus
Plugins (1) ----< (N) PluginConfigurations
Plugins (1) ----< (N) PluginData
```

## 常用查询示例

### 查询所有激活的插件

```sql
SELECT
    p.plugin_id,
    p.name,
    p.version,
    p.author,
    p.description,
    ps.is_active,
    ps.last_run
FROM Plugins p
LEFT JOIN PluginStatus ps ON p.plugin_id = ps.plugin_id
WHERE ps.is_active = 1
ORDER BY p.name;
```

### 查询特定插件的配置

```sql
SELECT
    config_key,
    config_value
FROM PluginConfigurations
WHERE plugin_id = ?
ORDER BY config_key;
```

### 查询插件的会话数据

```sql
SELECT
    session_id,
    data_key,
    data_value,
    updated_at
FROM PluginData
WHERE plugin_id = ? AND session_id = ?
ORDER BY updated_at DESC;
```

### 查询所有插件及其状态统计

```sql
SELECT
    p.name,
    p.version,
    COUNT(DISTINCT pd.session_id) as session_count,
    COUNT(pd.data_id) as data_count
FROM Plugins p
LEFT JOIN PluginData pd ON p.plugin_id = pd.plugin_id
GROUP BY p.plugin_id
ORDER BY p.name;
```
