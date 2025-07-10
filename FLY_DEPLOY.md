# Fly.io 部署指南

## 为什么选择 Fly.io？

相比其他免费平台，Fly.io 提供：
- ✅ **持久化存储支持**（免费3GB）
- ✅ **3个免费应用**
- ✅ **全球节点**（包括日本、新加坡）
- ✅ **自动休眠/唤醒**
- ✅ **更好的性能**

## 部署步骤

### 1. 安装 Fly CLI

**macOS:**
```bash
brew install flyctl
```

**Windows/Linux:**
```bash
curl -L https://fly.io/install.sh | sh
```

### 2. 登录和初始化

```bash
# 登录Fly.io
fly auth login

# 进入项目目录
cd /path/to/new-api

# 初始化应用（会自动检测fly.toml）
fly apps create new-api
```

### 3. 创建持久化存储

```bash
# 创建3GB的持久化卷
fly volumes create new_api_data --region nrt --size 3
```

### 4. 部署应用

```bash
# 部署到Fly.io
fly deploy
```

### 5. 设置域名（可选）

```bash
# 分配域名
fly apps open
```

## 配置说明

### fly.toml 配置文件解释

```toml
# 应用名称
app = "new-api"
# 主要区域（nrt=日本东京，更近更快）
primary_region = "nrt"

# 构建配置
[build]
  dockerfile = "Dockerfile"

# 环境变量
[env]
  TZ = "Asia/Shanghai"
  PORT = "3000"
  SQL_DSN = "file:/data/new-api.db?cache=shared&mode=rwc"
  REDIS_CONN_STRING = ""
  ERROR_LOG_ENABLED = "true"

# HTTP服务配置
[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = true    # 自动休眠
  auto_start_machines = true   # 自动唤醒
  min_machines_running = 0     # 可以完全休眠

# 持久化存储挂载
[[mounts]]
  source = "new_api_data"      # 卷名称
  destination = "/data"        # 挂载路径

# 健康检查
[checks]
  [checks.status]
    grace_period = "10s"
    interval = "30s"
    method = "GET"
    path = "/api/status"
    port = 3000
    timeout = "5s"
```

## 常用命令

```bash
# 查看应用状态
fly status

# 查看日志
fly logs

# 进入容器
fly ssh console

# 重启应用
fly machine restart

# 查看存储卷
fly volumes list

# 删除应用
fly apps destroy new-api
```

## 成本分析

### 免费额度
- **3个应用**
- **每月3GB持久化存储**
- **每月160小时运行时间**
- **共享CPU资源**

### 超出免费额度后
- **CPU时间**: $0.000001/秒
- **存储**: $0.15/GB/月
- **网络**: $0.02/GB

## 数据备份

```bash
# 连接到容器
fly ssh console

# 备份SQLite数据库
sqlite3 /data/new-api.db .dump > backup.sql
cat backup.sql

# 下载备份（在本地运行）
fly ssh sftp get /data/new-api.db ./backup.db
```

## 监控和维护

### 查看应用状态
```bash
fly status --app new-api
```

### 监控资源使用
```bash
fly metrics --app new-api
```

### 自动监控脚本
```bash
#!/bin/bash
# 每5分钟检查一次应用状态
while true; do
  curl -f https://new-api.fly.dev/api/status || echo "App down at $(date)"
  sleep 300
done
```

## 故障排查

### 部署失败
```bash
# 查看构建日志
fly logs --app new-api

# 检查配置
fly config show
```

### 数据丢失
- 确认持久化卷已创建：`fly volumes list`
- 检查挂载配置：确认 fly.toml 中的 mounts 配置

### 性能问题
- 查看机器状态：`fly machine list`
- 监控资源使用：`fly metrics`

## 优势对比

| 特性 | Fly.io | Render免费 | Railway |
|------|--------|------------|---------|
| 持久化存储 | ✅ 3GB | ❌ 无 | ❌ 受限 |
| 应用数量 | 3个 | 无限 | 仅数据库 |
| 休眠时间 | 可配置 | 15分钟 | 无 |
| 冷启动 | 5-10秒 | 30-60秒 | 即时 |
| 全球节点 | ✅ | ❌ | ✅ |
| SSH访问 | ✅ | ❌ | ❌ |

Fly.io 是目前最适合部署此项目的免费平台！