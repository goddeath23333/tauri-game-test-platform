# 游戏测试平台

一个基于 Tauri + Vue 3 的桌面应用，用于游戏测试 Issue 管理与工作流程跟踪。

## 项目简介

游戏测试平台是一个专为游戏测试团队设计的 Issue 管理系统，支持：

- **Issue 管理**：创建、查看、修改、状态流转
- **工作流程跟踪**：从提交到验证的完整流程
- **评论系统**：开发人员与测试人员的沟通记录
- **离线支持**：本地 SQLite 存储，支持离线操作
- **团队协作**：可选联网模式，支持多用户同步

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端框架 | Vue 3 + TypeScript |
| 后端框架 | Tauri 2.x (Rust) |
| 本地数据库 | SQLite |
| 构建工具 | Vite |
| 远程数据库 | PostgreSQL / MySQL (可选) |

## 用户角色

| 角色 | 职责 |
|------|------|
| 测试人员 (tester) | 测试游戏、提交 Issue、验证修复 |
| 开发人员 (developer) | 回复工单、修复 Issue |
| 管理员 (admin) | 管理用户、标签、指派负责人 |

## Issue 状态流转

```
Pending → InProgress → Fixed → Verified
   ↑                            ↓
提交Issue               测试人员确认
```

| 状态 | 说明 |
|------|------|
| Pending | 待处理，开发人员可开始修复 |
| InProgress | 处理中，开发人员正在修复 |
| Fixed | 已修复，等待测试人员验证 |
| Verified | 已验证，流程结束 |

## 项目结构

```
wanyaotest/
├── src/                    # Vue 前端源码
│   ├── App.vue             # 主组件
│   └── main.ts             # 入口文件
├── src-tauri/              # Tauri 后端源码
│   ├── src/
│   │   ├── main.rs         # Rust 入口
│   │   └── lib.rs          # Tauri 配置
│   ├── Cargo.toml          # Rust 依赖
│   └── tauri.conf.json     # Tauri 配置
├── docs/                   # 项目文档
│   ├── requirement.md      # 需求文档
│   ├── architecture.md     # 架构设计
│   ├── progress.md         # 开发进度
│   └── api.md              # API 文档
├── dist/                   # 前端构建产物
├── package.json            # 前端依赖配置
└── vite.config.ts          # Vite 配置
```

## 开发指南

### 环境要求

- Node.js 18+
- Rust 1.77+
- Windows 操作系统

### 安装依赖

```bash
npm install
```

### 开发模式

```bash
npm run tauri dev
```

### 构建发布

```bash
npm run tauri build
```

## 核心功能

### Issue 管理

- 创建 Issue（标题、描述、严重程度、截图）
- 查看 Issue 列表（支持状态筛选）
- 查看 Issue 详情（包含评论历史）
- 修改 Issue 状态

### 工作流程

```
测试人员测试 → 提交Issue → 开发人员回复 → 开始修复 → 修复完成 → 测试人员确认
```

### 部署模式

**独立模式（本地优先）**
- 仅使用本地 SQLite 数据库
- 无需服务器，适合个人使用

**联网模式（服务器同步）**
- 本地 SQLite + 远程 PostgreSQL/MySQL
- 数据可与团队共享
- 支持多用户协作

## 文档

详细文档请参阅 [docs](./docs/) 目录：

- [需求文档](./docs/requirement.md)
- [架构设计](./docs/architecture.md)
- [开发进度](./docs/progress.md)
- [API 文档](./docs/api.md)

## 开发进度

项目当前处于 **初始化阶段**，基础框架已搭建完成。

详细进度请参阅 [progress.md](./docs/progress.md)。

## 许可证

MIT License