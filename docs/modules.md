# 模块设计文档

## 1. 模块概述

游戏测试平台包含以下核心模块：

| 模块 | 职责 | 位置 |
|------|------|------|
| Issue模块 | Issue CRUD、状态管理 | 前端+后端 |
| 评论模块 | 评论添加、历史查看 | 前端+后端 |
| 标签模块 | 标签管理、关联Issue | 前端+后端 |
| 用户模块 | 用户管理、权限控制 | 前端+后端 |
| 同步模块 | 数据同步、离线处理 | 后端 |
| 设置模块 | 用户偏好、连接配置 | 前端+后端 |
| 统计模块 | 数据统计、图表展示 | 前端+后端 |
| 通知模块 | 状态变更通知、提醒 | 前端+后端 |
| 搜索模块 | 高级搜索、全文搜索 | 前端+后端 |
| 导出模块 | 数据导出、报告生成 | 前端+后端 |
| 附件模块 | 截图上传、文件管理 | 前端+后端 |
| 版本模块 | 游戏版本追踪 | 前端+后端 |
| 批量操作模块 | 批量指派、状态变更 | 前端+后端 |
| 历史记录模块 | 操作日志、审计 | 后端 |
| 仪表盘模块 | 数据概览、趋势图 | 前端 |

---

## 2. Issue模块

### 2.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 创建Issue | 提交新Issue | IssueForm.vue | create_issue |
| 查看列表 | Issue列表展示 | IssueList.vue | get_issue_list |
| 查看详情 | Issue详情页 | IssueDetail.vue | get_issue_detail |
| 更新状态 | 状态流转 | StatusBadge.vue | update_issue_status |
| 指派负责人 | 指派/转交 | UserSelect.vue | assign_issue |
| 删除Issue | 删除Issue | - | delete_issue |
| 筛选搜索 | 状态/优先级/标签筛选 | IssueList.vue | get_issue_list |

### 2.2 前端组件

```
IssueForm.vue
├── 标题输入
├── 优先级选择 (P1-P4)
├── 请求人选择 (默认当前用户)
├── 负责人选择 (可选)
├── 标签选择 (多选)
├── 描述输入
├── 截图上传
└── 提交按钮

IssueList.vue
├── 筛选栏
│   ├── 状态筛选
│   ├── 优先级筛选
│   ├── 标签筛选
│   └── 搜索框
├── Issue卡片列表
│   ├── 优先级标识
│   ├── 标签列表
│   ├── 标题
│   ├── 状态
│   ├── 负责人(岗位)
│   ├── 请求人(岗位)
│   └── 创建时间
└── 新建按钮、同步按钮

IssueDetail.vue
├── 返回按钮
├── Issue信息区
│   ├── 优先级+标签+标题
│   ├── 状态选择
│   ├── 优先级选择
│   ├── 请求人(岗位)
│   ├── 负责人选择(岗位)
│   ├── 标签编辑
│   └── 时间信息
├── 描述区
├── 截图区
├── 评论历史区
├── 评论输入区
└── 操作按钮区
```

### 2.3 后端命令

```rust
// src-tauri/src/commands/issue_commands.rs

#[tauri::command]
pub async fn create_issue(
    title: String,
    description: String,
    priority: String,
    requester_id: String,
    assignee_id: Option<String>,
    tag_ids: Vec<String>,
) -> Result<Issue, String>;

#[tauri::command]
pub async fn get_issue_list(
    status: Option<String>,
    priority: Option<String>,
    tag_id: Option<String>,
    requester_id: Option<String>,
    assignee_id: Option<String>,
    page: i32,
    limit: i32,
) -> Result<Vec<IssueSummary>, String>;

#[tauri::command]
pub async fn get_issue_detail(id: String) -> Result<IssueDetail, String>;

#[tauri::command]
pub async fn update_issue_status(
    id: String,
    new_status: String,
    comment: Option<String>,
) -> Result<(), String>;

#[tauri::command]
pub async fn assign_issue(
    id: String,
    assignee_id: String,
) -> Result<(), String>;

#[tauri::command]
pub async fn delete_issue(id: String) -> Result<(), String>;
```

### 2.4 数据流

```
创建Issue:
用户填写表单 → IssueForm.vue → issueStore.createIssue()
→ invoke('create_issue') → Rust处理 → 本地缓存 + 离线队列
→ 联网时同步到服务器

查看列表:
IssueList.vue加载 → issueStore.fetchIssues()
→ invoke('get_issue_list') → Rust读取本地缓存
→ 联网时从服务器更新缓存

状态变更:
点击状态按钮 → StatusBadge.vue → issueStore.updateStatus()
→ invoke('update_issue_status') → Rust验证权限 → 更新状态
→ 记录状态历史 → 联网时同步
```

---

## 3. 评论模块

### 3.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 添加评论 | 评论Issue | CommentInput.vue | add_comment |
| 查看历史 | 评论列表 | CommentList.vue | get_comments |
| 删除评论 | 删除自己的评论 | - | delete_comment |

### 3.2 前端组件

```
CommentList.vue
├── 评论卡片列表
│   ├── 作者(岗位)
│   ├── 评论内容
│   ├── 创建时间
│   └── 删除按钮(仅自己的评论)
└── 加载更多按钮

CommentInput.vue
├── 评论输入框
└── 发送按钮
```

### 3.3 后端命令

```rust
// src-tauri/src/commands/comment_commands.rs

#[tauri::command]
pub async fn add_comment(
    issue_id: String,
    content: String,
) -> Result<Comment, String>;

#[tauri::command]
pub async fn get_comments(
    issue_id: String,
    page: i32,
    limit: i32,
) -> Result<Vec<Comment>, String>;

#[tauri::command]
pub async fn delete_comment(id: String) -> Result<(), String>;
```

---

## 4. 标签模块

### 4.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 查看标签列表 | 全部标签 | TagManage.vue | get_tags |
| 创建标签 | 新建标签 | TagForm.vue | create_tag |
| 编辑标签 | 修改名称/颜色 | TagForm.vue | update_tag |
| 删除标签 | 删除标签 | - | delete_tag |
| 关联Issue | Issue添加标签 | IssueForm.vue | add_issue_tag |

### 4.2 前端组件

```
TagManage.vue
├── 标签列表
│   ├── 标签名称
│   ├── 颜色显示
│   ├── 使用次数
│   ├── 编辑按钮
│   └── 删除按钮
└── 新建标签按钮

TagForm.vue
├── 名称输入
├── 颜色选择器
├── 描述输入
└── 保存按钮

TagBadge.vue
├── 标签名称
├── 背景颜色
└── 点击筛选功能
```

### 4.3 后端命令

```rust
// src-tauri/src/commands/tag_commands.rs

#[tauri::command]
pub async fn get_tags() -> Result<Vec<Tag>, String>;

#[tauri::command]
pub async fn create_tag(
    name: String,
    color: String,
    description: Option<String>,
) -> Result<Tag, String>;

#[tauri::command]
pub async fn update_tag(
    id: String,
    name: String,
    color: String,
) -> Result<(), String>;

#[tauri::command]
pub async fn delete_tag(id: String) -> Result<(), String>;
```

---

## 5. 用户模块

### 5.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 用户登录 | JWT认证 | LoginForm.vue | login |
| 用户注册 | 创建账户 | RegisterForm.vue | register |
| 查看用户列表 | 成员管理 | UserManage.vue | get_users |
| 编辑用户 | 修改角色 | UserForm.vue | update_user |
| 删除用户 | 删除账户 | - | delete_user |
| 获取当前用户 | 获取自己信息 | - | get_current_user |

### 5.2 前端组件

```
UserManage.vue
├── 用户列表
│   ├── 用户名称
│   ├── 岗位显示
│   ├── 统计数据
│   ├── 编辑按钮(管理员)
│   └── 删除按钮(管理员)
└── 添加成员按钮(管理员)

UserForm.vue
├── 名称输入
├── 邮箱输入
├── 角色选择
└── 保存按钮

UserSelect.vue
├── 用户下拉列表
│   ├── 用户名称(岗位)
│   └── 搜索筛选
└── 选择确认

LoginForm.vue
├── 邮箱输入
├── 密码输入
└── 登录按钮

RegisterForm.vue
├── 名称输入
├── 邮箱输入
├── 密码输入
├── 角色选择
└── 注册按钮
```

### 5.3 后端命令

```rust
// src-tauri/src/commands/user_commands.rs

#[tauri::command]
pub async fn login(
    email: String,
    password: String,
) -> Result<LoginResponse, String>;

#[tauri::command]
pub async fn register(
    name: String,
    email: String,
    password: String,
    role: String,
) -> Result<User, String>;

#[tauri::command]
pub async fn get_users(role: Option<String>) -> Result<Vec<UserSummary>, String>;

#[tauri::command]
pub async fn get_current_user() -> Result<User, String>;

#[tauri::command]
pub async fn update_user(
    id: String,
    name: String,
    role: String,
) -> Result<(), String>;

#[tauri::command]
pub async fn delete_user(id: String) -> Result<(), String>;
```

---

## 6. 同步模块

### 6.1 功能列表

| 功能 | 说明 | 触发条件 |
|------|------|----------|
| 自动同步 | 启动时同步 | 应用启动 |
| 定时同步 | 定期同步 | 可配置间隔 |
| 手动同步 | 点击同步 | 用户点击 |
| 状态同步 | 状态变更同步 | Issue状态变更 |
| 离线队列上传 | 上传离线操作 | 联网时 |
| 缓存更新 | 更新本地缓存 | 同步成功后 |

### 6.2 同步流程

```
sync_service.rs
├── check_connection()      // 检查网络连接
├── upload_pending_queue()  // 上传离线队列
├── download_updates()      // 下载服务器更新
├── merge_local_cache()     // 合并本地缓存
├── clear_synced_queue()    // 清理已同步队列
└── update_sync_status()    // 更新同步状态
```

### 6.3 后端命令

```rust
// src-tauri/src/commands/sync_commands.rs

#[tauri::command]
pub async fn sync_now() -> Result<SyncResult, String>;

#[tauri::command]
pub async fn get_sync_status() -> Result<SyncStatus, String>;

#[tauri::command]
pub async fn get_pending_count() -> Result<i32, String>;

#[tauri::command]
pub async fn set_sync_interval(minutes: i32) -> Result<(), String>;
```

---

## 7. 设置模块

### 7.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 主题设置 | 深色/浅色 | Settings.vue | set_theme |
| 语言设置 | 中文/英文 | Settings.vue | set_language |
| 服务器配置 | URL设置 | Settings.vue | set_server_url |
| 同步间隔 | 同步频率 | Settings.vue | set_sync_interval |
| 离线模式 | 离线开关 | Settings.vue | set_offline_mode |
| 环境检测 | 检测依赖 | Settings.vue | check_environment |

### 7.2 前端组件

```
Settings.vue
├── 连接设置区
│   ├── 服务器地址输入
│   ├── 测试连接按钮
│   ├── 自动同步开关
│   ├── 同步间隔设置
│   └── 离线模式开关
├── 界面设置区
│   ├── 主题选择
│   └── 语言选择
├── 环境检测区
│   ├── 系统版本
│   ├── WebView2状态
│   ├── Rust版本
│   └── SQLite状态
└── 保存按钮
```

### 7.3 后端命令

```rust
// src-tauri/src/commands/settings_commands.rs

#[tauri::command]
pub async fn get_settings() -> Result<Settings, String>;

#[tauri::command]
pub async fn set_setting(key: String, value: String) -> Result<(), String>;

#[tauri::command]
pub async fn test_connection(url: String) -> Result<bool, String>;

#[tauri::command]
pub async fn check_environment() -> Result<EnvironmentStatus, String>;
```

---

## 8. 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                      前端模块依赖                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  IssueForm.vue                                              │
│      ├── UserSelect.vue (选择请求人/负责人)                  │
│      ├── TagBadge.vue (选择标签)                            │
│      └── CommentInput.vue (添加评论)                        │
│                                                             │
│  IssueList.vue                                              │
│      ├── StatusBadge.vue (状态显示)                         │
│      ├── PriorityBadge.vue (优先级显示)                     │
│      └── TagBadge.vue (标签显示)                            │
│                                                             │
│  IssueDetail.vue                                            │
│      ├── StatusBadge.vue (状态显示)                         │
│      ├── PriorityBadge.vue (优先级显示)                     │
│      ├── TagBadge.vue (标签显示)                            │
│      ├── CommentList.vue (评论历史)                         │
│      ├── CommentInput.vue (添加评论)                        │
│      └── UserSelect.vue (选择负责人)                        │
│                                                             │
│  Settings.vue                                               │
│      └── EnvironmentCheck.vue (环境检测)                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      后端模块依赖                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  issue_commands.rs                                          │
│      ├── user_commands.rs (获取用户信息)                    │
│      ├── tag_commands.rs (获取标签信息)                     │
│      ├── comment_commands.rs (获取评论)                     │
│      └── sync_commands.rs (触发同步)                        │
│                                                             │
│  sync_commands.rs                                           │
│      ├── db/sqlite.rs (本地数据库)                          │
│      └── db/remote.rs (远程数据库)                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. 状态管理 (Pinia Stores)

```typescript
// stores/issue.ts
export const useIssueStore = defineStore('issue', {
  state: () => ({
    issues: [] as IssueSummary[],
    currentIssue: null as IssueDetail | null,
    filter: {
      status: null,
      priority: null,
      tagId: null,
    },
    loading: false,
  }),
  actions: {
    async fetchIssues() {},
    async fetchIssueDetail(id: string) {},
    async createIssue(data: CreateIssueDto) {},
    async updateStatus(id: string, status: string) {},
    async assignIssue(id: string, assigneeId: string) {},
  },
});

// stores/user.ts
export const useUserStore = defineStore('user', {
  state: () => ({
    currentUser: null as User | null,
    users: [] as UserSummary[],
    token: '',
  }),
  actions: {
    async login(email: string, password: string) {},
    async register(data: RegisterDto) {},
    async fetchUsers() {},
    async updateUser(id: string, data: UpdateUserDto) {},
  },
});

// stores/tag.ts
export const useTagStore = defineStore('tag', {
  state: () => ({
    tags: [] as Tag[],
  }),
  actions: {
    async fetchTags() {},
    async createTag(data: CreateTagDto) {},
    async updateTag(id: string, data: UpdateTagDto) {},
  },
});

// stores/sync.ts
export const useSyncStore = defineStore('sync', {
  state: () => ({
    lastSyncAt: null,
    pendingCount: 0,
    syncing: false,
    offlineMode: false,
  }),
  actions: {
    async syncNow() {},
    async getSyncStatus() {},
    async setOfflineMode(enabled: boolean) {},
  },
});

// stores/settings.ts
export const useSettingsStore = defineStore('settings', {
  state: () => ({
    theme: 'dark',
    language: 'zh-CN',
    serverUrl: '',
    syncInterval: 5,
  }),
  actions: {
    async loadSettings() {},
    async saveSettings() {},
    async testConnection() {},
  },
});
```

---

## 10. 统计模块

### 10.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| Issue统计 | 按状态/优先级/标签统计 | StatsView.vue | get_issue_stats |
| 用户统计 | 每人Issue数量统计 | StatsView.vue | get_user_stats |
| 趋势图 | Issue创建趋势 | TrendChart.vue | get_trend_data |
| 标签统计 | 标签使用频率 | TagStats.vue | get_tag_stats |
| 响应时间统计 | 平均处理时间 | StatsView.vue | get_response_time |

### 10.2 前端组件

```
StatsView.vue
├── 概览卡片
│   ├── 总Issue数
│   ├── 待处理数
│   ├── 处理中数
│   ├── 已修复数
│   └── 已验证数
├── 状态分布图 (饼图)
├── 优先级分布图 (柱状图)
├── 标签使用图 (柱状图)
├── 创建趋势图 (折线图)
└── 用户排行榜

TrendChart.vue
├── 时间范围选择 (日/周/月)
├── 折线图展示
└── 数据导出按钮

UserStats.vue
├── 用户列表
│   ├── 创建Issue数
│   ├── 处理Issue数
│   ├── 验证Issue数
│   └── 平均响应时间
└── 排序筛选
```

### 10.3 后端命令

```rust
// src-tauri/src/commands/stats_commands.rs

#[tauri::command]
pub async fn get_issue_stats() -> Result<IssueStats, String>;

#[tauri::command]
pub async fn get_user_stats() -> Result<Vec<UserStats>, String>;

#[tauri::command]
pub async fn get_trend_data(
    range: String,  // day/week/month
) -> Result<TrendData, String>;

#[tauri::command]
pub async fn get_tag_stats() -> Result<Vec<TagStats>, String>;

#[tauri::command]
pub async fn get_response_time() -> Result<ResponseTimeStats, String>;
```

---

## 11. 通知模块

### 11.1 功能列表

| 功能 | 说明 | 触发条件 |
|------|------|----------|
| Issue指派通知 | 被指派时通知 | 指派负责人 |
| 状态变更通知 | 状态改变时通知 | 状态流转 |
| 评论通知 | 新评论时通知 | 添加评论 |
| 修复完成通知 | 修复完成时通知 | 状态变为Fixed |
| 验证完成通知 | 验证完成时通知 | 状态变为Verified |
| 系统通知 | 系统消息 | 系统事件 |

### 11.2 前端组件

```
NotificationCenter.vue
├── 通知列表
│   ├── 通知类型图标
│   ├── 通知内容
│   ├── 时间显示
│   ├── 已读/未读状态
│   └── 点击跳转
├── 全部标记已读按钮
└── 清空通知按钮

NotificationBadge.vue
├── 未读数量显示
└── 点击打开通知中心

NotificationToast.vue
├── 弹出通知
├── 自动消失
└── 点击跳转
```

### 11.3 后端命令

```rust
// src-tauri/src/commands/notification_commands.rs

#[tauri::command]
pub async fn get_notifications(
    page: i32,
    limit: i32,
) -> Result<Vec<Notification>, String>;

#[tauri::command]
pub async fn mark_as_read(id: String) -> Result<(), String>;

#[tauri::command]
pub async fn mark_all_as_read() -> Result<(), String>;

#[tauri::command]
pub async fn get_unread_count() -> Result<i32, String>;

#[tauri::command]
pub async fn clear_notifications() -> Result<(), String>;
```

---

## 12. 搜索模块

### 12.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 快速搜索 | 标题关键词搜索 | SearchBar.vue | quick_search |
| 高级搜索 | 多条件组合搜索 | AdvancedSearch.vue | advanced_search |
| 全文搜索 | 描述内容搜索 | SearchBar.vue | fulltext_search |
| 搜索历史 | 保存搜索记录 | SearchHistory.vue | get_search_history |
| 搜索建议 | 自动补全建议 | SearchBar.vue | get_suggestions |

### 12.2 前端组件

```
SearchBar.vue
├── 搜索输入框
├── 搜索建议下拉
├── 搜索按钮
└── 高级搜索入口

AdvancedSearch.vue
├── 关键词输入
├── 状态选择 (多选)
├── 优先级选择 (多选)
├── 标签选择 (多选)
├── 请求人选择
├── 负责人选择
├── 时间范围选择
├── 搜索按钮
└── 保存搜索条件

SearchHistory.vue
├── 最近搜索列表
├── 点击快速搜索
└── 清除历史按钮

SearchResult.vue
├── 结果列表
├── 高亮关键词
└── 分页显示
```

### 12.3 后端命令

```rust
// src-tauri/src/commands/search_commands.rs

#[tauri::command]
pub async fn quick_search(keyword: String) -> Result<Vec<IssueSummary>, String>;

#[tauri::command]
pub async fn advanced_search(
    keyword: Option<String>,
    statuses: Option<Vec<String>>,
    priorities: Option<Vec<String>>,
    tag_ids: Option<Vec<String>>,
    requester_id: Option<String>,
    assignee_id: Option<String>,
    start_date: Option<String>,
    end_date: Option<String>,
) -> Result<Vec<IssueSummary>, String>;

#[tauri::command]
pub async fn fulltext_search(keyword: String) -> Result<Vec<IssueSummary>, String>;

#[tauri::command]
pub async fn get_search_history() -> Result<Vec<SearchHistory>, String>;

#[tauri::command]
pub async fn get_suggestions(keyword: String) -> Result<Vec<String>, String>;
```

---

## 13. 导出模块

### 13.1 功能列表

| 功能 | 说明 | 格式 |
|------|------|------|
| 导出Issue列表 | 导出全部Issue | CSV/Excel |
| 导出单个Issue | 导出Issue详情 | PDF/HTML |
| 导出统计报告 | 导出统计数据 | PDF/Excel |
| 导出用户报告 | 导出用户统计 | CSV/Excel |
| 导出备份 | 全量数据备份 | JSON/SQL |

### 13.2 前端组件

```
ExportDialog.vue
├── 导出类型选择
│   ├── Issue列表
│   ├── Issue详情
│   ├── 统计报告
│   ├── 用户报告
│   └── 全量备份
├── 格式选择
│   ├── CSV
│   ├── Excel
│   ├── PDF
│   ├── JSON
├── 范围选择 (时间/状态)
├── 导出按钮
└── 下载进度显示

ExportProgress.vue
├── 进度条
├── 导出状态
└── 取消按钮
```

### 13.3 后端命令

```rust
// src-tauri/src/commands/export_commands.rs

#[tauri::command]
pub async fn export_issues(
    format: String,
    filters: Option<ExportFilters>,
) -> Result<String, String>;  // 返回文件路径

#[tauri::command]
pub async fn export_issue_detail(
    issue_id: String,
    format: String,
) -> Result<String, String>;

#[tauri::command]
pub async fn export_stats_report(format: String) -> Result<String, String>;

#[tauri::command]
pub async fn export_user_report(format: String) -> Result<String, String>;

#[tauri::command]
pub async fn export_backup() -> Result<String, String>;
```

---

## 14. 附件模块

### 14.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 上传截图 | 上传图片附件 | FileUpload.vue | upload_attachment |
| 上传日志 | 上传日志文件 | FileUpload.vue | upload_attachment |
| 查看附件 | 查看附件列表 | AttachmentList.vue | get_attachments |
| 下载附件 | 下载附件文件 | AttachmentList.vue | download_attachment |
| 删除附件 | 删除附件 | AttachmentList.vue | delete_attachment |
| 预览图片 | 图片预览 | ImagePreview.vue | - |

### 14.2 前端组件

```
FileUpload.vue
├── 文件选择按钮
├── 拖拽上传区域
├── 文件类型限制 (图片/日志)
├── 文件大小限制显示
├── 上传进度条
└── 已上传文件列表

AttachmentList.vue
├── 附件卡片列表
│   ├── 文件名
│   ├── 文件大小
│   ├── 上传者(岗位)
│   ├── 上传时间
│   ├── 预览按钮
│   ├── 下载按钮
│   └── 删除按钮
└── 上传入口

ImagePreview.vue
├── 图片放大显示
├── 缩放控制
├── 旋转控制
└── 关闭按钮
```

### 14.3 后端命令

```rust
// src-tauri/src/commands/attachment_commands.rs

#[tauri::command]
pub async fn upload_attachment(
    issue_id: String,
    file_path: String,
) -> Result<Attachment, String>;

#[tauri::command]
pub async fn get_attachments(issue_id: String) -> Result<Vec<Attachment>, String>;

#[tauri::command]
pub async fn download_attachment(id: String) -> Result<String, String>;

#[tauri::command]
pub async fn delete_attachment(id: String) -> Result<(), String>;
```

---

## 15. 版本管理模块

### 15.1 功能列表

| 功能 | 说明 | 前端 | 后端 |
|------|------|------|------|
| 创建版本 | 新建游戏版本 | VersionForm.vue | create_version |
| 版本列表 | 查看版本列表 | VersionList.vue | get_versions |
| Issue关联版本 | Issue标记版本 | IssueForm.vue | - |
| 版本Issue统计 | 每版本Issue数 | VersionStats.vue | get_version_stats |
| 版本对比 | 版本间Issue对比 | VersionCompare.vue | compare_versions |

### 15.2 前端组件

```
VersionManage.vue
├── 版本列表
│   ├── 版本号
│   ├── 发布日期
│   ├── Issue数量
│   ├── 已修复数
│   └── 编辑按钮
└── 新建版本按钮

VersionForm.vue
├── 版本号输入
├── 发布日期选择
├── 版本说明
└── 保存按钮

VersionStats.vue
├── 版本选择
├── Issue统计
│   ├── 按状态分布
│   ├── 按优先级分布
│   └── 按标签分布
└── 导出按钮

VersionCompare.vue
├── 版本A选择
├── 版本B选择
├── 对比结果
│   ├── 新增Issue
│   ├── 已修复Issue
│   ├── 未修复Issue
└── 导出对比报告
```

### 15.3 后端命令

```rust
// src-tauri/src/commands/version_commands.rs

#[tauri::command]
pub async fn create_version(
    version: String,
    release_date: String,
    description: Option<String>,
) -> Result<Version, String>;

#[tauri::command]
pub async fn get_versions() -> Result<Vec<Version>, String>;

#[tauri::command]
pub async fn get_version_stats(version: String) -> Result<VersionStats, String>;

#[tauri::command]
pub async fn compare_versions(
    version_a: String,
    version_b: String,
) -> Result<CompareResult, String>;
```

---

## 16. 批量操作模块

### 16.1 功能列表

| 功能 | 说明 | 权限 |
|------|------|------|
| 批量指派 | 批量指派负责人 | 管理员 |
| 批量状态变更 | 批量修改状态 | 管理员 |
| 批量添加标签 | 批量添加标签 | 管理员/请求人 |
| 批量删除 | 批量删除Issue | 管理员 |
| 批量导出 | 批量导出Issue | 全部用户 |

### 16.2 前端组件

```
BatchOperation.vue
├── Issue选择列表
│   ├── 全选按钮
│   ├── 单选复选框
│   └── 已选数量显示
├── 操作选择
│   ├── 批量指派
│   ├── 批量状态变更
│   ├── 批量添加标签
│   ├── 批量删除
│   └── 批量导出
├── 操作参数输入
└── 执行按钮

BatchAssign.vue
├── 负责人选择
├── 确认按钮
└── 取消按钮

BatchStatus.vue
├── 新状态选择
├── 批量评论输入
└── 确认按钮
```

### 16.3 后端命令

```rust
// src-tauri/src/commands/batch_commands.rs

#[tauri::command]
pub async fn batch_assign(
    issue_ids: Vec<String>,
    assignee_id: String,
) -> Result<BatchResult, String>;

#[tauri::command]
pub async fn batch_update_status(
    issue_ids: Vec<String>,
    new_status: String,
    comment: Option<String>,
) -> Result<BatchResult, String>;

#[tauri::command]
pub async fn batch_add_tags(
    issue_ids: Vec<String>,
    tag_ids: Vec<String>,
) -> Result<BatchResult, String>;

#[tauri::command]
pub async fn batch_delete(issue_ids: Vec<String>) -> Result<BatchResult, String>;

#[tauri::command]
pub async fn batch_export(issue_ids: Vec<String>) -> Result<String, String>;
```

---

## 17. 历史记录模块

### 17.1 功能列表

| 功能 | 说明 | 权限 |
|------|------|------|
| 操作日志 | 记录所有操作 | 管理员查看 |
| 状态变更历史 | Issue状态流转记录 | 全部用户 |
| 登录日志 | 用户登录记录 | 管理员查看 |
| 数据变更日志 | 数据修改记录 | 管理员查看 |
| 审计报告 | 生成审计报告 | 管理员 |

### 17.2 前端组件

```
HistoryLog.vue
├── 日志筛选
│   ├── 操作类型选择
│   ├── 用户选择
│   ├── 时间范围
├── 日志列表
│   ├── 操作类型
│   ├── 操作者(岗位)
│   ├── 操作对象
│   ├── 操作详情
│   └── 操作时间
└── 导出日志按钮

StatusHistory.vue
├── 状态变更列表
│   ├── 旧状态
│   ├── 新状态
│   ├── 变更者(岗位)
│   ├── 变更时间
│   └── 变更备注
└── 时间线展示
```

### 17.3 后端命令

```rust
// src-tauri/src/commands/history_commands.rs

#[tauri::command]
pub async fn get_operation_logs(
    operation_type: Option<String>,
    user_id: Option<String>,
    start_date: Option<String>,
    end_date: Option<String>,
    page: i32,
    limit: i32,
) -> Result<Vec<OperationLog>, String>;

#[tauri::command]
pub async fn get_status_history(issue_id: String) -> Result<Vec<StatusHistory>, String>;

#[tauri::command]
pub async fn get_login_logs(
    user_id: Option<String>,
    page: i32,
    limit: i32,
) -> Result<Vec<LoginLog>, String>;

#[tauri::command]
pub async fn export_audit_report() -> Result<String, String>;
```

---

## 18. 仪表盘模块

### 18.1 功能列表

| 功能 | 说明 | 展示方式 |
|------|------|----------|
| 数据概览 | 关键指标汇总 | 卡片 |
| Issue趋势 | 创建/完成趋势 | 折线图 |
| 状态分布 | 状态占比 | 饼图 |
| 优先级分布 | 优先级占比 | 柱状图 |
| 用户活跃度 | 用户操作统计 | 排行榜 |
| 待处理提醒 | 高优先级提醒 | 列表 |

### 18.2 前端组件

```
Dashboard.vue
├── 概览卡片区
│   ├── 今日新增Issue
│   ├── 本周完成Issue
│   ├── 待处理总数
│   ├── P1紧急Issue数
│   └── 平均响应时间
├── 趋势图区
│   ├── Issue创建趋势 (折线图)
│   └── Issue完成趋势 (折线图)
├── 分布图区
│   ├── 状态分布 (饼图)
│   ├── 优先级分布 (柱状图)
│   └── 标签分布 (柱状图)
├── 用户活跃度区
│   ├── 创建排行
│   ├── 处理排行
│   └── 验证排行
├── 待处理提醒区
│   ├── P1紧急Issue列表
│   ├── 超时未处理Issue
│   └── 快速跳转
└── 快捷操作区
    ├── 快速创建Issue
    ├── 快速同步
    └── 快速导出

OverviewCard.vue
├── 指标名称
├── 指标数值
├── 变化趋势 (↑↓)
└── 点击查看详情

TrendChart.vue
├── 时间范围切换 (日/周/月)
├── 图表展示
└── 数据说明

DistributionChart.vue
├── 图表类型切换
├── 图表展示
└── 数据详情

ActivityRank.vue
├── 用户列表
│   ├── 用户名(岗位)
│   ├── 操作数量
│   └── 排名
└── 时间范围选择

AlertList.vue
├── 提醒列表
│   ├── Issue标题
│   ├── 优先级标识
│   ├── 超时时间
│   └── 快速处理按钮
└── 全部查看按钮
```

### 18.3 后端命令

```rust
// src-tauri/src/commands/dashboard_commands.rs

#[tauri::command]
pub async fn get_dashboard_overview() -> Result<DashboardOverview, String>;

#[tauri::command]
pub async fn get_trend_data(range: String) -> Result<TrendData, String>;

#[tauri::command]
pub async fn get_distribution_data() -> Result<DistributionData, String>;

#[tauri::command]
pub async fn get_activity_rank(range: String) -> Result<Vec<ActivityRank>, String>;

#[tauri::command]
pub async fn get_alert_list() -> Result<Vec<AlertItem>, String>;
```

---

## 19. 完整模块依赖图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           前端模块完整依赖                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Dashboard.vue (仪表盘)                                                  │
│      ├── OverviewCard.vue                                               │
│      ├── TrendChart.vue                                                 │
│      ├── DistributionChart.vue                                          │
│      ├── ActivityRank.vue                                               │
│      └── AlertList.vue                                                  │
│                                                                         │
│  IssueList.vue                                                          │
│      ├── SearchBar.vue → AdvancedSearch.vue                             │
│      ├── StatusBadge.vue                                                │
│      ├── PriorityBadge.vue                                              │
│      ├── TagBadge.vue                                                   │
│      └── BatchOperation.vue                                             │
│                                                                         │
│  IssueDetail.vue                                                        │
│      ├── StatusBadge.vue                                                │
│      ├── PriorityBadge.vue                                              │
│      ├── TagBadge.vue                                                   │
│      ├── CommentList.vue → CommentInput.vue                             │
│      ├── AttachmentList.vue → FileUpload.vue                            │
│      ├── UserSelect.vue                                                 │
│      ├── StatusHistory.vue                                              │
│      └── ImagePreview.vue                                               │
│                                                                         │
│  IssueForm.vue                                                          │
│      ├── UserSelect.vue                                                 │
│      ├── TagBadge.vue                                                   │
│      ├── FileUpload.vue                                                 │
│      └── VersionSelect.vue                                              │
│                                                                         │
│  StatsView.vue                                                          │
│      ├── TrendChart.vue                                                 │
│      ├── UserStats.vue                                                  │
│      └── TagStats.vue                                                   │
│                                                                         │
│  NotificationCenter.vue                                                 │
│      ├── NotificationBadge.vue                                          │
│      └── NotificationToast.vue                                          │
│                                                                         │
│  VersionManage.vue                                                      │
│      ├── VersionForm.vue                                                │
│      ├── VersionStats.vue                                               │
│      └── VersionCompare.vue                                             │
│                                                                         │
│  ExportDialog.vue                                                       │
│      └── ExportProgress.vue                                             │
│                                                                         │
│  Settings.vue                                                           │
│      └── EnvironmentCheck.vue                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                           后端模块完整依赖                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  issue_commands.rs                                                      │
│      ├── user_commands.rs                                               │
│      ├── tag_commands.rs                                                │
│      ├── comment_commands.rs                                            │
│      ├── attachment_commands.rs                                         │
│      ├── version_commands.rs                                            │
│      ├── history_commands.rs                                            │
│      ├── notification_commands.rs                                       │
│      └── sync_commands.rs                                               │
│                                                                         │
│  stats_commands.rs                                                      │
│      ├── issue_commands.rs                                              │
│      ├── user_commands.rs                                               │
│      └── tag_commands.rs                                                │
│                                                                         │
│  dashboard_commands.rs                                                  │
│      ├── stats_commands.rs                                              │
│      ├── issue_commands.rs                                              │
│      └── user_commands.rs                                               │
│                                                                         │
│  search_commands.rs                                                     │
│      ├── issue_commands.rs                                              │
│      └── tag_commands.rs                                                │
│                                                                         │
│  batch_commands.rs                                                      │
│      ├── issue_commands.rs                                              │
│      ├── notification_commands.rs                                       │
│      └── history_commands.rs                                            │
│                                                                         │
│  export_commands.rs                                                     │
│      ├── issue_commands.rs                                              │
│      ├── stats_commands.rs                                              │
│      └── user_commands.rs                                               │
│                                                                         │
│  sync_commands.rs                                                       │
│      ├── db/sqlite.rs                                                   │
│      ├── db/remote.rs                                                   │
│      └── 所有commands (同步各模块数据)                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```