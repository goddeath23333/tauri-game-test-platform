# Tauri游戏测试平台 - 代码规范

## 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| Rust模块/变量 | snake_case | `issue_commands`, `issue_id` |
| Rust结构体/枚举 | PascalCase | `Issue`, `IssueStatus` |
| 前端组件/类型 | PascalCase | `IssueList.vue`, `IssueStatus` |

## Issue状态

```rust
pub enum IssueStatus {
    Pending,     // 待处理
    InProgress,  // 处理中
    Fixed,       // 已修复
    Verified,    // 已验证
}
```

## 状态流转

```
Pending → InProgress → Fixed → Verified
   ↑                            ↑
提交Issue               测试人员确认
```

## Tauri命令

```rust
#[tauri::command]
async fn create_issue(issue: Issue) -> Result<Issue, String>;

#[tauri::command]
async fn get_issue_list(status: Option<IssueStatus>) -> Result<Vec<Issue>, String>;

#[tauri::command]
async fn update_issue_status(issue_id: String, status: IssueStatus) -> Result<(), String>;
```

## 提交规范

```
<type>: <message>
feat/fix/refactor/docs/test/chore
```
