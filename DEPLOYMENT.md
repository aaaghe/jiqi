# GitHub Actions CI/CD 部署指南

## 📋 概览

本项目使用 GitHub Actions 自动化以下流程：
- ✅ **代码检查** (Lint, Format, Vet)
- ✅ **安全扫描** (Gosec, 依赖检查)
- ✅ **构建** (Go Binary 和 Docker 镜像)
- ✅ **自动部署** (Staging 和 Production)

---

## 🔧 必需的 GitHub Secrets 配置

### 1. Docker 凭证（可选，用于 Docker Hub 推送）
```
DOCKER_USERNAME  → 你的 Docker Hub 用户名
DOCKER_PASSWORD  → 你的 Docker Hub 访问令牌
```

### 2. Staging 环境部署凭证
```
STAGING_DEPLOY_KEY   → SSH 私钥（用于连接 Staging 服务器）
STAGING_DEPLOY_HOST  → Staging 服务器地址（如：staging.example.com）
STAGING_DEPLOY_USER  → SSH 用户名（如：deploy）
STAGING_DEPLOY_PATH  → 应用部署路径（如：/home/deploy/bot）
```

### 3. Production 环境部署凭证
```
PROD_DEPLOY_KEY   → SSH 私钥（用于连接生产服务器）
PROD_DEPLOY_HOST  → 生产服务器地址
PROD_DEPLOY_USER  → SSH 用户名
PROD_DEPLOY_PATH  → 应用部署路径
```

### 4. Telegram Bot 环境变量
```
TG_BOT_TOKEN  → Telegram Bot API Token
API_ID        → Telegram App API ID
API_HASH      → Telegram App API Hash
LOG_LEVEL     → 日志级别（debug/info/warn/error）
```

### 5. 通知集成（可选）
```
SLACK_WEBHOOK  → Slack Webhook URL（用于部署通知）
```

---

## 🚀 工作流程

### A. 自动流程 (Push to Kai)

```
Push to Kai
    ↓
1️⃣ Lint & Quality Check (lint.yml)
   - golangci-lint
   - gofmt
   - go vet
    ↓
2️⃣ Security Scan (security.yml)
   - Gosec（安全漏洞检查）
   - go mod verify（依赖完整性）
    ↓
3️⃣ Build (deploy.yml - build job)
   - go build
   - go test
   - 上传二进制文件
    ↓
4️⃣ Docker Build & Push (deploy.yml - docker-build job)
   - Docker 镜像构建
   - 推送到 Docker Hub
    ↓
5️⃣ Deploy to Staging (deploy.yml - deploy-staging job)
   - SSH 连接到 Staging 服务器
   - git pull origin Kai
   - docker-compose up -d
   - 发送 Slack 通知
```

### B. 手动部署到生产 (Workflow Dispatch)

进入 GitHub → Actions → Deploy TG-FileStreamBot → "Run workflow" → 选择 "production"

```
Manual Trigger
    ↓
1️⃣ Build & Docker
    ↓
2️⃣ Deploy to Production
   - SSH 连接到生产服务器
   - git pull origin Kai
   - docker-compose up -d
   - 发送 Slack 通知
```

---

## 📝 如何配置

### Step 1: 生成 SSH 密钥对

```bash
# 生成用于 GitHub Actions 的 SSH 密钥
ssh-keygen -t ed25519 -f github-actions-key -N ""

# github-actions-key.pub → 添加到服务器的 ~/.ssh/authorized_keys
# github-actions-key → 作为 GitHub Secret 存储（STAGING_DEPLOY_KEY 等）
```

### Step 2: 配置服务器（Staging/Production）

在你的服务器上：

```bash
# 1. 克隆仓库
git clone https://github.com/aaaghe/jiqi.git /home/deploy/bot
cd /home/deploy/bot

# 2. 创建 .env 文件
cat > .env << 'EOF'
TG_BOT_TOKEN=your_bot_token_here
API_ID=your_api_id
API_HASH=your_api_hash
LOG_LEVEL=info
HTTP_PORT=8080
EOF

# 3. 验证 docker-compose 配置
docker-compose config

# 4. 首次启动（可选）
docker-compose up -d
```

### Step 3: 在 GitHub 中配置 Secrets

1. 进入 GitHub 仓库 → Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 添加以下 secrets：

| Secret Name | Value |
|------------|-------|
| `DOCKER_USERNAME` | 你的 Docker Hub 用户名 |
| `DOCKER_PASSWORD` | Docker Hub 访问令牌 |
| `STAGING_DEPLOY_KEY` | 粘贴 `github-actions-key` 的内容 |
| `STAGING_DEPLOY_HOST` | staging.example.com |
| `STAGING_DEPLOY_USER` | deploy |
| `STAGING_DEPLOY_PATH` | /home/deploy/bot |
| `PROD_DEPLOY_KEY` | 生产服务器的 SSH 密钥 |
| `PROD_DEPLOY_HOST` | production.example.com |
| `PROD_DEPLOY_USER` | deploy |
| `PROD_DEPLOY_PATH` | /home/deploy/bot |
| `SLACK_WEBHOOK` | https://hooks.slack.com/services/YOUR/WEBHOOK/URL |

---

## 📊 查看工作流运行状态

1. 进入 GitHub 仓库 → Actions 标签
2. 选择工作流查看详细信息
3. 点击具体的 run 查看日志

---

## 🔍 常见问题

### Q: 为什么构建失败？
**A:** 检查：
- Go 版本是否为 1.22+
- `TG-FileStreamBot-main` 目录是否存在
- go.mod 和 go.sum 文件是否完整

### Q: 为什么无法连接到服务器？
**A:** 检查：
- SSH 密钥是否正确上传到服务器的 `~/.ssh/authorized_keys`
- 防火墙是否允许 SSH 连接（端口 22）
- 部署用户是否有权限访问指定的目录

### Q: 如何禁用某个工作流？
**A:** 
- 进入 Actions → 点击要禁用的工作流 → "Disable workflow"
- 或者删除对应的 .yml 文件并提交

### Q: 如何手动触发部署？
**A:**
- 进入 Actions → 选择 "Deploy TG-FileStreamBot"
- 点击 "Run workflow"
- 选择环境（staging/production）和分支
- 点击 "Run"

### Q: 如何查看 Docker 构建日志？
**A:**
- 在 Actions 中点击对应的 run
- 查看 "docker-build" job 的详细日志

---

## 📚 相关文档

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Docker Compose 文档](https://docs.docker.com/compose/)
- [Go 官方文档](https://golang.org/doc/)
- [SSH 密钥配置指南](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

---

## 🆘 需要帮助？

如有问题，请：
1. 检查 Actions 日志获取详细错误信息
2. 验证所有 Secrets 是否正确配置
3. 确认服务器网络连接和权限设置
4. 查看 Slack 通知了解部署状态
