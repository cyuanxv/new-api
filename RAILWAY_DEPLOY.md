# Railway 部署指南

## 快速部署步骤

### 1. 准备代码仓库
1. 将项目代码推送到GitHub仓库
2. 确保包含 `railway.yml` 和 `docker-compose.railway.yml` 文件

### 2. Railway 部署
1. 访问 [Railway.app](https://railway.app)
2. 使用GitHub账号登录
3. 点击 "New Project" -> "Deploy from GitHub repo"
4. 选择你的new-api仓库
5. Railway会自动识别Docker配置并开始部署

### 3. 环境变量配置（可选）
在Railway控制台的Variables选项卡中，可以设置：

```
TZ=Asia/Shanghai
PORT=3000
ERROR_LOG_ENABLED=true
SESSION_SECRET=your_random_secret_string_here
```

### 4. 数据库配置
此配置使用SQLite数据库，数据存储在容器的 `/data` 目录中。Railway会自动提供持久化存储。

### 5. 访问应用
部署完成后，Railway会提供一个公网域名，格式如：
`https://your-app-name.railway.app`

## 重要说明

### 数据持久化
- SQLite数据库文件存储在 `/data/new-api.db`
- 日志文件存储在 `/app/logs`
- Railway提供持久化存储，重启不会丢失数据

### 性能优化
- 使用内存缓存替代Redis，降低资源消耗
- SQLite适合中小规模使用
- 如需更高性能，可升级到Railway的PostgreSQL addon

### 成本控制
- Railway提供$5免费额度
- 按使用量计费，Sleep模式可节省成本
- 监控使用情况避免超出免费额度

### 域名绑定（可选）
1. 在Railway控制台Settings页面
2. 添加自定义域名
3. 配置DNS解析

## 本地测试Railway配置

```bash
# 使用Railway配置本地测试
docker-compose -f docker-compose.railway.yml up -d

# 查看日志
docker-compose -f docker-compose.railway.yml logs -f

# 停止服务
docker-compose -f docker-compose.railway.yml down
```

## 故障排查

### 部署失败
1. 检查Dockerfile是否存在
2. 确认railway.yml语法正确
3. 查看Railway部署日志

### 应用无法访问
1. 确认端口配置正确（默认3000）
2. 检查健康检查是否通过
3. 查看应用日志排查错误

### 数据丢失
- Railway提供持久化存储，重启不会丢失数据
- 建议定期备份SQLite数据库文件

## 进阶配置

### 多机部署
如需多机部署，需要：
1. 使用外部数据库（PostgreSQL addon）
2. 配置SESSION_SECRET环境变量
3. 设置FRONTEND_BASE_URL

### 监控告警
1. Railway控制台提供基础监控
2. 可配置Webhook通知
3. 集成第三方监控服务

### 备份策略
```bash
# 导出SQLite数据库
sqlite3 /data/new-api.db .dump > backup.sql

# 恢复数据库
sqlite3 /data/new-api.db < backup.sql
```