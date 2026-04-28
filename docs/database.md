# 数据库设计文档

## 1. 数据库选型

| 环境 | 数据库 | 说明 |
|------|--------|------|
| 本地 | SQLite | 轻量级，无需安装 |
| 服务器 | PostgreSQL | 稳定可靠，开源免费 |
| 服务器(可选) | MySQL | 广泛使用 |

## 2. 数据表结构

### 2.1 用户表 (users)

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'tester',
    avatar_url VARCHAR(500),
    issues_created INT DEFAULT 0,
    issues_assigned INT DEFAULT 0,
    issues_completed INT DEFAULT 0,
    issues_verified INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 角色枚举: admin, developer, tester
```

### 2.2 标签表 (tags)

```sql
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) NOT NULL UNIQUE,
    color VARCHAR(7) NOT NULL DEFAULT '#808080',
    description TEXT,
    usage_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 预置标签数据
INSERT INTO tags (name, color, description) VALUES
    ('崩溃', '#FF0000', '游戏崩溃类问题'),
    ('启动', '#FF6B00', '启动相关问题'),
    ('音频', '#FFD700', '音效、音乐问题'),
    ('UI', '#00FF00', '界面显示问题'),
    ('性能', '#0088FF', '性能优化问题'),
    ('网络', '#9C27B0', '网络连接问题'),
    ('存储', '#795548', '数据存储问题'),
    ('兼容性', '#607D8B', '设备兼容问题');
```

### 2.3 Issue表 (issues)

```sql
CREATE TABLE issues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    priority VARCHAR(10) NOT NULL DEFAULT 'P3',
    status VARCHAR(50) NOT NULL DEFAULT 'pending',
    severity VARCHAR(50) NOT NULL DEFAULT 'medium',
    requester_id UUID NOT NULL REFERENCES users(id),
    assignee_id UUID REFERENCES users(id),
    game_version VARCHAR(100),
    screenshot_urls JSONB DEFAULT '[]',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 优先级枚举: P1(紧急), P2(高), P3(中), P4(低)
-- 状态枚举: pending, in_progress, fixed, verified
-- 严重程度枚举: low, medium, high, critical
```

### 2.4 Issue标签关联表 (issue_tags)

```sql
CREATE TABLE issue_tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    issue_id UUID NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
    tag_id UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(issue_id, tag_id)
);
```

### 2.5 评论表 (comments)

```sql
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    issue_id UUID NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
    author_id UUID NOT NULL REFERENCES users(id),
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2.6 状态历史表 (status_history)

```sql
CREATE TABLE status_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    issue_id UUID NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
    old_status VARCHAR(50),
    new_status VARCHAR(50) NOT NULL,
    changed_by UUID NOT NULL REFERENCES users(id),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2.7 附件表 (attachments)

```sql
CREATE TABLE attachments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    issue_id UUID NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size BIGINT,
    mime_type VARCHAR(100),
    uploaded_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 3. 本地SQLite结构（轻量化设计）

### 3.1 设计原则

```
本地数据库轻量化原则:
- 仅存储必要数据，不存储完整Issue详情
- 缓存最近50条Issue摘要
- 用户列表仅存储姓名+岗位
- 离线操作队列同步后自动清理
- 预估大小: < 5MB
```

### 3.2 本地表结构

```sql
-- 用户偏好设置（永久保留）
CREATE TABLE local_settings (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL
);

-- 默认设置
INSERT INTO local_settings (key, value) VALUES
    ('theme', 'dark'),
    ('language', 'zh-CN'),
    ('sync_interval', '5'),
    ('server_url', ''),
    ('offline_mode', 'false');

-- 最近Issue缓存（定期清理，保留50条）
CREATE TABLE local_issue_cache (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    priority TEXT NOT NULL,
    status TEXT NOT NULL,
    requester_name TEXT NOT NULL,
    requester_role TEXT NOT NULL,
    assignee_name TEXT,
    assignee_role TEXT,
    tag_names TEXT,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    synced_at TEXT
);

-- 用户列表缓存（姓名+岗位，轻量）
CREATE TABLE local_users (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    role TEXT NOT NULL,
    role_display TEXT NOT NULL
);

-- 标签列表缓存（轻量）
CREATE TABLE local_tags (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    color TEXT NOT NULL
);

-- 离线操作队列（同步后删除）
CREATE TABLE pending_sync (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    operation TEXT NOT NULL,
    entity_type TEXT NOT NULL,
    entity_id TEXT NOT NULL,
    payload TEXT NOT NULL,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

-- 同步状态记录
CREATE TABLE sync_status (
    id INTEGER PRIMARY KEY CHECK (id = 1),
    last_sync_at TEXT,
    pending_count INTEGER DEFAULT 0
);
```

### 3.3 本地数据清理策略

```sql
-- 清理超过50条的Issue缓存
DELETE FROM local_issue_cache 
WHERE id NOT IN (
    SELECT id FROM local_issue_cache 
    ORDER BY updated_at DESC 
    LIMIT 50
);

-- 清理已同步的离线操作
DELETE FROM pending_sync 
WHERE synced = 1;

-- 更新同步状态
UPDATE sync_status SET 
    last_sync_at = CURRENT_TIMESTAMP,
    pending_count = (SELECT COUNT(*) FROM pending_sync);
```

### 3.4 本地与服务器数据对比

| 数据 | 本地存储 | 服务器存储 | 同步策略 |
|------|----------|-----------|----------|
| Issue详情 | 仅摘要 | 完整数据 | 服务器为主 |
| Issue描述 | 不存储 | 完整存储 | 服务器为主 |
| Issue截图 | 不存储 | 文件服务器 | 服务器为主 |
| 评论历史 | 不存储 | 完整存储 | 服务器为主 |
| 用户信息 | 姓名+岗位 | 完整信息 | 定期同步 |
| 标签信息 | 全部标签 | 全部标签 | 定期同步 |
| 状态历史 | 不存储 | 完整存储 | 服务器为主 |
| 用户偏好 | 完整存储 | 不存储 | 仅本地 |

## 4. 索引设计

```sql
-- 性能优化索引
CREATE INDEX idx_issues_status ON issues(status);
CREATE INDEX idx_issues_priority ON issues(priority);
CREATE INDEX idx_issues_requester ON issues(requester_id);
CREATE INDEX idx_issues_assignee ON issues(assignee_id);
CREATE INDEX idx_issues_created_at ON issues(created_at DESC);
CREATE INDEX idx_comments_issue ON comments(issue_id);
CREATE INDEX idx_status_history_issue ON status_history(issue_id);
CREATE INDEX idx_issue_tags_issue ON issue_tags(issue_id);
CREATE INDEX idx_issue_tags_tag ON issue_tags(tag_id);
CREATE INDEX idx_tags_name ON tags(name);
```

## 5. 触发器设计

### 5.1 标签使用计数更新

```sql
CREATE OR REPLACE FUNCTION update_tag_usage_count()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE tags SET usage_count = usage_count + 1 WHERE id = NEW.tag_id;
    END IF;
    IF TG_OP = 'DELETE' THEN
        UPDATE tags SET usage_count = usage_count - 1 WHERE id = OLD.tag_id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_issue_tags_count
AFTER INSERT OR DELETE ON issue_tags
FOR EACH ROW EXECUTE FUNCTION update_tag_usage_count();
```

### 5.2 用户统计更新

```sql
CREATE OR REPLACE FUNCTION update_user_stats()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE users SET issues_created = issues_created + 1 
        WHERE id = NEW.requester_id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_issue_user_stats
AFTER INSERT ON issues
FOR EACH ROW EXECUTE FUNCTION update_user_stats();
```

## 6. 部署脚本

### 6.1 PostgreSQL一键部署

```bash
# 下载数据库脚本
curl -O https://raw.githubusercontent.com/gametester/db/main/init-postgres.sql

# 执行初始化
psql -U postgres -d gametester -f init-postgres.sql
```

### 6.2 MySQL一键部署

```bash
# 下载数据库脚本
curl -O https://raw.githubusercontent.com/gametester/db/main/init-mysql.sql

# 执行初始化
mysql -u root -p gametester < init-mysql.sql
```

## 7. 一键部署CLI

```bash
# 初始化数据库
gametester-cli db init --type postgres --host localhost --port 5432

# 一键启动服务
gametester-cli deploy --db postgres --port 8080 --host 0.0.0.0

# 导出数据库
gametester-cli db export --output ./backup

# 导入数据
gametester-cli db import --input ./backup
```

## 8. 数据关系图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   users     │     │   issues    │     │   tags      │
├─────────────┤     ├─────────────┤     ├─────────────┤
│ id          │◄────│ requester_id│     │ id          │
│ name        │     │ assignee_id │◄────│ name        │
│ email       │     │ title       │     │ color       │
│ role        │     │ priority    │     │ usage_count │
│ stats...    │     │ status      │     └─────────────┘
└─────────────┘     │ tag_ids[]   │           │
      │             └─────────────┘           │
      │                   │                   │
      │                   ▼                   │
      │             ┌─────────────┐           │
      │             │ issue_tags  │◄──────────┘
      │             ├─────────────┤
      │             │ issue_id    │
      │             │ tag_id      │
      │             └─────────────┘
      │
      ▼
┌─────────────┐     ┌─────────────┐
│  comments   │     │ attachments │
├─────────────┤     ├─────────────┤
│ issue_id    │     │ issue_id    │
│ author_id   │     │ uploaded_by │
│ content     │     │ file_path   │
└─────────────┘     └─────────────┘
```