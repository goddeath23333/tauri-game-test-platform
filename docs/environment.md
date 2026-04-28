# 环境检测与支持说明

## 1. 环境检测

### 1.1 自动检测

应用启动时自动检测以下环境：

| 检测项 | 说明 | 失败处理 |
|--------|------|----------|
| 操作系统 | Windows 10+/Linux/macOS | 提示不支持的系统 |
| WebView2 | Windows浏览器组件 | 提示安装 |
| Node.js | 前端开发依赖 | 可选，仅开发模式需要 |
| Rust | Tauri构建依赖 | 提示安装 |
| 数据库驱动 | SQLite/PostgreSQL | 自动下载SQLite |

### 1.2 环境检测命令

```bash
# 检测所有依赖
gametester-cli doctor

# 输出示例:
# ✓ Windows 10 (build 19041)
# ✓ WebView2 Runtime
# ✓ Rust 1.94.0
# ✓ PostgreSQL Client 14.0
# ✓ SQLite 3.36.0
```

### 1.3 Rust环境检测

```bash
# 检查Rust安装
rustc --version
cargo --version

# 检查Tauri CLI
tauri --version
```

## 2. 支持的操作系统

### 2.1 Windows

| 版本 | 支持状态 |
|------|----------|
| Windows 10 | ✅ 支持 |
| Windows 11 | ✅ 支持 |

**依赖:**
- WebView2 Runtime (Windows 10/11默认已安装)

### 2.2 Linux

**依赖:**
- WebKitGTK 4.1+
- libsoup 3.0+
- openssl

**安装命令:**
```bash
# Ubuntu/Debian
sudo apt install libwebkit2gtk-4.1-dev libssl-dev

# Fedora
sudo dnf install webkit2gtk4.1-devel openssl-devel
```

### 2.3 macOS

**依赖:**
- Cocoa framework
- Security framework

## 3. 数据库要求

### 3.1 本地模式 (SQLite)

无需额外安装，应用内置SQLite驱动。

### 3.2 服务器模式

#### PostgreSQL

| 版本 | 推荐 |
|------|------|
| PostgreSQL 14+ | ✅ 推荐 |

#### MySQL

| 版本 | 推荐 |
|------|------|
| MySQL 8.0+ | ✅ 推荐 |

## 4. 客户端环境要求

### 4.1 最低配置

| 资源 | 最低要求 |
|------|----------|
| 内存 | 4 GB |
| 磁盘 | 200 MB |
| 处理器 | 双核 1.5GHz |

### 4.2 推荐配置

| 资源 | 推荐 |
|------|------|
| 内存 | 8 GB |
| 磁盘 | 1 GB |
| 处理器 | 四核 2.5GHz |

## 5. 网络要求

### 5.1 独立模式

- 无需网络连接
- 所有数据存储在本地

### 5.2 联网模式

| 需求 | 说明 |
|------|------|
| 端口 | 8080 (可自定义) |
| 协议 | HTTP/HTTPS |
| 防火墙 | 允许入站连接(服务器模式) |

## 6. 错误代码

| 代码 | 说明 | 解决方案 |
|------|------|----------|
| E001 | WebView2未安装 | 下载安装WebView2 Runtime |
| E002 | 数据库连接失败 | 检查数据库服务是否运行 |
| E003 | 端口被占用 | 更换端口或关闭占用程序 |
| E004 | Rust未安装 | 运行 `rustup install stable` |
| E005 | 网络超时 | 检查网络连接和服务器状态 |

## 7. 获取帮助

### 7.1 常用命令

```bash
# 查看帮助
gametester-cli --help

# 查看版本
gametester-cli --version

# 环境诊断
gametester-cli doctor
```

### 7.2 日志位置

| 平台 | 日志路径 |
|------|----------|
| Windows | `%APPDATA%\GameTester\logs\` |
| Linux | `~/.config/gametester/logs/` |
| macOS | `~/Library/Logs/GameTester/` |

### 7.3 技术支持

- GitHub Issues: https://github.com/gametester/app/issues
- 文档: https://gametester.app/docs
