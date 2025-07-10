# Render 部署指南

## 快速部署步骤

### 方案1: 通过GitHub仓库部署（推荐）

1. **访问 [render.com](https://render.com)**
2. **GitHub登录**
3. **点击 "New +" -> "Web Service"**
4. **选择 "Build and deploy from a Git repository"**
5. **连接GitHub并选择 `cyuanxv/new-api` 仓库**
6. **配置部署参数：**
   - **Name**: `new-api`
   - **Region**: `Oregon (US West)`
   - **Branch**: `main`
   - **Build Command**: (留空，使用Docker)
   - **Start Command**: (留空，使用Docker)

7. **高级设置：**
   - **Auto-Deploy**: `Yes`
   - **Environment**: `Docker`

8. **环境变量设置：**
```
TZ=Asia/Shanghai
PORT=3000
SQL_DSN=file:/data/new-api.db?cache=shared&mode=rwc
REDIS_CONN_STRING=
ERROR_LOG_ENABLED=true
```

9. **持久化存储（重要）：**
   - 在 Settings -> Disks 中添加
   - **Name**: `data`
   - **Mount Path**: `/data`
   - **Size**: `1 GB`

### 方案2: 使用 render.yaml 自动配置

如果仓库包含 `render.yaml` 文件，Render会自动读取配置。

## 部署特点

### 免费计划限制
- ✅ 750小时/月免费额度
- ✅ 自动休眠（15分钟无访问）
- ✅ 1GB持久化存储
- ⚠️ 冷启动延迟（30-60秒）
- ⚠️ 每月重置一次

### 付费计划优势
- 🚀 无休眠，24/7在线
- 🚀 更快启动速度
- 🚀 更多存储空间
- 💰 $7/月起

## 访问应用

部署完成后获得域名：
`https://your-app-name.onrender.com`

## 数据持久化

- SQLite数据库：`/data/new-api.db`
- 日志文件：`/data/logs/`
- 免费计划提供1GB持久化存储
- 数据在重启后保持

## 性能优化

### 减少冷启动影响
1. 设置健康检查URL：`/api/status`
2. 使用外部监控服务定期ping
3. 考虑升级到付费计划

### 监控建议
```bash
# 简单的外部监控脚本
curl -f https://your-app.onrender.com/api/status || echo "App is down"
```

## 故障排查

### 部署失败
1. 检查构建日志
2. 确认Dockerfile正确
3. 验证环境变量设置

### 数据丢失
- 确保持久化磁盘已正确挂载到 `/data`
- 检查磁盘大小是否足够

### 性能问题
- 免费计划有资源限制
- 考虑优化SQLite查询
- 必要时升级计划

## 与Railway对比

| 特性 | Render免费 | Railway | 
|------|------------|---------|
| 部署限制 | 无 | 仅数据库 |
| 休眠机制 | 15分钟 | 无 |
| 存储 | 1GB | 1GB |
| 域名 | onrender.com | railway.app |
| 冷启动 | 30-60秒 | 即时 |

## 迁移到其他平台

如果需要迁移到其他平台：

1. **导出数据**：
```bash
# 下载SQLite数据库
wget https://your-app.onrender.com/backup/database
```

2. **推荐平台**：
   - **Fly.io**: 更好的性能
   - **Koyeb**: 欧洲节点
   - **Vercel**: 静态部署（需改造）

## 成本计算

- **免费版**: $0/月（有限制）
- **Starter**: $7/月（无休眠）
- **Professional**: $25/月（更多资源）

建议先用免费版测试，稳定后再考虑升级。