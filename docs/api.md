# API接口文档

## 1. API概述

### 1.1 基础信息

| 项目 | 说明 |
|------|------|
| 基础URL | `http://localhost:8080/api` |
| 协议 | HTTP/HTTPS |
| 数据格式 | JSON |
| 认证方式 | JWT Token |

### 1.2 请求头

```
Content-Type: application/json
Authorization: Bearer <token>
```

### 1.3 响应格式

```json
{
  "success": true,
  "data": {},
  "error": null
}
```

---

## 2. 认证接口

### 2.1 用户登录

```
POST /auth/login
```

**请求体:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**响应:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "uuid",
      "name": "张三",
      "role": "developer"
    }
  }
}
```

### 2.2 用户注册

```
POST /auth/register
```

**请求体:**
```json
{
  "name": "张三",
  "email": "user@example.com",
  "password": "password123",
  "role": "tester"
}
```

### 2.3 获取当前用户

```
GET /auth/me
```

---

## 3. Issue接口

### 3.1 获取Issue列表

```
GET /issues
```

**查询参数:**
| 参数 | 类型 | 说明 |
|------|------|------|
| status | string | 状态筛选 |
| priority | string | 优先级筛选 |
| tag | string | 标签筛选 |
| requester_id | string | 请求人筛选 |
| assignee_id | string | 负责人筛选 |
| page | int | 页码 |
| limit | int | 每页数量 |

**示例:**
```
GET /issues?status=pending&priority=P1&page=1&limit=20
```

**响应:**
```json
{
  "success": true,
  "data": {
    "issues": [
      {
        "id": "uuid",
        "title": "游戏崩溃无法启动",
        "priority": "P1",
        "status": "pending",
        "requester": {"id": "uuid", "name": "李四"},
        "assignee": {"id": "uuid", "name": "张三"},
        "tags": [{"id": "uuid", "name": "崩溃", "color": "#FF0000"}],
        "created_at": "2024-01-15T14:30:00Z"
      }
    ],
    "total": 100,
    "page": 1,
    "limit": 20
  }
}
```

### 3.2 创建Issue

```
POST /issues
```

**请求体:**
```json
{
  "title": "游戏崩溃无法启动",
  "description": "游戏在启动画面后立即崩溃",
  "priority": "P1",
  "severity": "critical",
  "requester_id": "uuid",
  "assignee_id": "uuid",
  "tag_ids": ["uuid1", "uuid2"],
  "game_version": "1.0.0",
  "screenshot_urls": ["url1", "url2"]
}
```

### 3.3 获取Issue详情

```
GET /issues/:id
```

**响应:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "游戏崩溃无法启动",
    "description": "...",
    "priority": "P1",
    "status": "pending",
    "severity": "critical",
    "requester": {"id": "uuid", "name": "李四", "role": "tester"},
    "assignee": {"id": "uuid", "name": "张三", "role": "developer"},
    "tags": [...],
    "comments": [...],
    "status_history": [...],
    "attachments": [...],
    "created_at": "...",
    "updated_at": "..."
  }
}
```

### 3.4 更新Issue状态

```
PATCH /issues/:id/status
```

**请求体:**
```json
{
  "status": "in_progress",
  "comment": "开始修复此问题"
}
```

### 3.5 更新Issue信息

```
PATCH /issues/:id
```

**请求体:**
```json
{
  "title": "新标题",
  "priority": "P2",
  "assignee_id": "new_uuid",
  "tag_ids": ["uuid1", "uuid3"]
}
```

### 3.6 删除Issue

```
DELETE /issues/:id
```

---

## 4. 评论接口

### 4.1 获取评论列表

```
GET /issues/:id/comments
```

### 4.2 添加评论

```
POST /issues/:id/comments
```

**请求体:**
```json
{
  "content": "已确认问题，正在排查中。"
}
```

### 4.3 删除评论

```
DELETE /issues/:id/comments/:comment_id
```

---

## 5. 标签接口

### 5.1 获取标签列表

```
GET /tags
```

**响应:**
```json
{
  "success": true,
  "data": {
    "tags": [
      {
        "id": "uuid",
        "name": "崩溃",
        "color": "#FF0000",
        "usage_count": 5
      }
    ]
  }
}
```

### 5.2 创建标签

```
POST /tags
```

**请求体:**
```json
{
  "name": "新标签",
  "color": "#FF5733",
  "description": "标签描述"
}
```

### 5.3 更新标签

```
PATCH /tags/:id
```

### 5.4 删除标签

```
DELETE /tags/:id
```

---

## 6. 用户接口

### 6.1 获取用户列表

```
GET /users
```

**查询参数:**
| 参数 | 类型 | 说明 |
|------|------|------|
| role | string | 角色筛选 |

### 6.2 获取用户详情

```
GET /users/:id
```

### 6.3 更新用户信息

```
PATCH /users/:id
```

### 6.4 删除用户

```
DELETE /users/:id
```

---

## 7. 统计接口

### 7.1 获取统计数据

```
GET /stats
```

**响应:**
```json
{
  "success": true,
  "data": {
    "total_issues": 100,
    "by_status": {
      "pending": 20,
      "in_progress": 15,
      "fixed": 30,
      "verified": 35
    },
    "by_priority": {
      "P1": 5,
      "P2": 15,
      "P3": 50,
      "P4": 30
    },
    "by_tag": {...}
  }
}
```

### 7.2 获取用户统计

```
GET /stats/users/:id
```

---

## 8. 同步接口

### 8.1 推送本地数据

```
POST /sync/push
```

**请求体:**
```json
{
  "issues": [...],
  "comments": [...],
  "last_sync_at": "2024-01-15T10:00:00Z"
}
```

### 8.2 拉取远程数据

```
GET /sync/pull?since=2024-01-15T10:00:00Z
```

---

## 9. 错误码

| 错误码 | 说明 |
|--------|------|
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 409 | 资源冲突 |
| 500 | 服务器错误 |

**错误响应格式:**
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": 400,
    "message": "参数错误",
    "details": "title不能为空"
  }
}
```

---

## 10. 权限控制

| 接口 | admin | developer | tester |
|------|-------|-----------|--------|
| 创建Issue | ✅ | ✅ | ✅ |
| 修改Issue状态 | ✅ | ✅ | ❌ |
| 验证Issue | ✅ | ❌ | ✅ |
| 删除Issue | ✅ | ❌ | ❌ |
| 管理标签 | ✅ | ❌ | ❌ |
| 管理用户 | ✅ | ❌ | ❌ |