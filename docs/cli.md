# CLI部署文档

## 1. CLI工具概述

`gametester-cli` 是游戏测试平台的一键部署工具，支持：
- 数据库一键部署
- 服务器启动/停止
- 数据导入/导出
- 环境检测

## 2. 安装CLI

### Windows

```bash
# 下载最新版本
curl -O https://github.com/gametester/cli/releases/latest/gametester-cli.exe

# 添加到PATH
setx PATH "%PATH%;C:\path\to\gametester-cli.exe"

# 验证安装
gametester-cli --version
```

### Linux/macOS

```bash
curl -fsSL https://github.com/gametester/cli/releases/latest/install.sh | sh
```

## 3. 核心命令

### 3.1 deploy - 一键部署

```bash
gametester-cli deploy [OPTIONS]

OPTIONS:
  --db <type>          数据库类型: postgres, mysql, sqlite
  --port <port>        服务器端口 (默认: 8080)
  --host <host>        服务器地址 (默认: 127.0.0.1)
  --password <pwd>     数据库密码
  --init-db            初始化数据库
```

**示例:**

```bash
# PostgreSQL一键部署
gametester-cli deploy --db postgres --port 8080 --host 0.0.0.0 --password secret --init-db

# MySQL一键部署
gametester-cli deploy --db mysql --port 8080 --password secret --init-db

# SQLite本地模式
gametester-cli deploy --db sqlite --port 8080
```

### 3.2 start - 启动服务器

```bash
gametester-cli start [OPTIONS]

OPTIONS:
  --port <port>        端口 (默认: 8080)
  --host <host>        地址 (默认: 127.0.0.1)
  --config <path>      配置文件路径
```

**示例:**

```bash
gametester-cli start --port 8080 --host 0.0.0.0
```

### 3.3 stop - 停止服务器

```bash
gametester-cli stop
```

### 3.4 db - 数据库管理

```bash
# 初始化数据库
gametester-cli db init --type <postgres|mysql|sqlite>

# 导入数据
gametester-cli db import --input <path> --type <type>

# 导出数据
gametester-cli db export --output <path> --type <type>

# 数据库状态
gametester-cli db status
```

**示例:**

```bash
gametester-cli db init --type postgres --host localhost --port 5432 --password secret
gametester-cli db export --output ./backup
gametester-cli db status
```

### 3.5 config - 配置管理

```bash
# 查看当前配置
gametester-cli config show

# 设置配置项
gametester-cli config set <key> <value>

# 示例
gametester-cli config set server.port 8080
gametester-cli config set database.type postgres
```

## 4. GitHub数据库下载

### 4.1 下载预构建数据库

```bash
gametester-cli db download --release latest --output ./db
gametester-cli db download --release v1.0.0 --db postgres --output ./db
```

### 4.2 可用仓库

| 仓库 | 说明 |
|------|------|
| gametester/db-postgres | PostgreSQL数据库镜像 |
| gametester/db-mysql | MySQL数据库镜像 |
| gametester/db-sqlite | SQLite数据库文件 |

## 5. 出口设置

### 5.1 配置出口IP

```bash
gametester-cli config set server.host 0.0.0.0
gametester-cli config set server.public_ip 203.0.113.1
```

### 5.2 配置CORS

```bash
gametester-cli config set server.allow_origins "https://example.com"
gametester-cli config set server.allow_methods "GET,POST,PUT,DELETE"
```

### 5.3 配置SSL

```bash
gametester-cli config set server.ssl_cert ./certs/cert.pem
gametester-cli config set server.ssl_key ./certs/key.pem
```

## 6. 完整部署流程

```bash
# 1. 安装CLI
curl -O https://github.com/gametester/cli/releases/latest/gametester-cli.exe

# 2. 一键部署服务器 + 数据库
gametester-cli deploy --db postgres --port 8080 --host 0.0.0.0 --init-db

# 3. 验证服务
curl http://localhost:8080/api/health

# 4. 启动桌面客户端
./游戏测试平台.exe
```

## 7. 常见问题

### Q: 端口被占用?

```bash
gametester-cli start --port 8081
```

### Q: 数据库连接失败?

```bash
gametester-cli db status
gametester-cli config set database.host localhost
```

### Q: 需要重启服务?

```bash
gametester-cli stop
gametester-cli start
```
