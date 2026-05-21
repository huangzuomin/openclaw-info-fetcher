# 📘 Info Fetcher 小白使用教程

> 从零开始：部署 OpenClaw 信息抓取子智能体
> 适合：完全没有 OpenClaw 使用经验的初学者

---

## 目录

1. [这个东西是什么？](#1-这个东西是什么)
2. [你需要准备什么](#2-你需要准备什么)
3. [第一步：安装 OpenClaw](#3-第一步安装-openclaw)
4. [第二步：部署 Info Fetcher](#4-第二步部署-info-fetcher)
5. [第三步：申请 API Key](#5-第三步申请-api-key)
6. [第四步：配置环境变量](#6-第四步配置环境变量)
7. [第五步：测试使用](#7-第五步测试使用)
8. [常见问题](#8-常见问题)

---

## 1. 这个东西是什么？

**Info Fetcher** 是一个 AI 信息抓取助手，能帮你：

- 📄 读取网页内容，转成 Markdown 格式
- 🔍 用关键词搜索发现线索
- 🕷 批量抓取整个网站
- 📱 抓取社交媒体（X/微博/B站等）
- 🎬 获取 YouTube/B站视频字幕
- 📡 监控 RSS 订阅源
- 🐙 调研 GitHub 仓库

它不是独立软件，而是 **OpenClaw**（一个开源 AI 助手平台）的子智能体。

**简单理解**：
```
你（说话）→ OpenClaw（理解你的意思）→ Info Fetcher（去网上抓东西）→ 返回结果给你
```

---

## 2. 你需要准备什么

| 项目 | 要求 | 说明 |
|------|------|------|
| **电脑** | Linux / macOS / Windows WSL | 推荐 Ubuntu 22.04+ |
| **Node.js** | v20+ | [下载地址](https://nodejs.org/) |
| **Git** | 任意版本 | 用于克隆代码 |
| **API Key** | 至少1个 | 免费的就行，见第5步 |
| **OpenClaw** | 最新版 | 见第3步 |

---

## 3. 第一步：安装 OpenClaw

### 3.1 安装 Node.js（如果还没有）

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS
brew install node

# 验证
node --version   # 应该显示 v20+ 
npm --version
```

### 3.2 安装 OpenClaw

```bash
npm install -g openclaw

# 验证
openclaw --version
```

### 3.3 初始化 OpenClaw

```bash
# 创建工作目录
mkdir -p ~/.openclaw/workspace

# 启动 OpenClaw
openclaw start
```

第一次启动会要求你配置 AI 模型的 API Key。推荐先用免费的：
- **GLM**（智谱AI）：注册送免费额度 → https://open.bigmodel.cn/
- **小米 MiMo**：免费 → 在 OpenClaw 配置中添加 XIAOMI_API_KEY

> 💡 OpenClaw 支持连接 Telegram / Discord / 微信等聊天平台。配置好后，你直接在聊天软件里跟 AI 对话就行。详细配置见 [OpenClaw 文档](https://docs.openclaw.ai)

---

## 4. 第二步：部署 Info Fetcher

### 4.1 克隆仓库

```bash
cd ~/.openclaw/

# 克隆 info-fetcher 到指定位置
git clone https://github.com/huangzuomin/openclaw-info-fetcher.git workspace-info-fetcher
```

### 4.2 配置环境变量

```bash
cd workspace-info-fetcher

# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，填入你的 API Key
nano .env
```

### 4.3 在主工作空间中配置路由

打开 `~/.openclaw/workspace/AGENTS.md`（没有就创建），添加以下内容：

```markdown
## Info Fetcher 子智能体

当收到以下请求时，dispatch 到 info-fetcher：
- 提供 URL 要求抓取/读取网页内容
- "抓取"、"爬取"、"采集"、"fetch"、"crawl"
- 搜索发现线索（"找一下"、"有没有关于"）
- 社媒内容采集（X/微博/小红书/抖音/B站等）
- RSS feed 读取
- GitHub 仓库调研
- 视频字幕采集
- 来源可靠性评估

调用方式：
\`\`\`bash
sessions_spawn(
  task="<具体任务描述>",
  cwd="/home/你的用户名/.openclaw/workspace-info-fetcher",
  context="isolated"
)
\`\`\`
```

### 4.4 重启 OpenClaw

```bash
openclaw restart
```

---

## 5. 第三步：申请 API Key

Info Fetcher 有10个 Skill，每个 Skill 可能需要不同的 API Key。**你不需要全部申请**——按需配置即可。

### 🟢 免费必配（建议全部申请）

#### 5.1 智谱 GLM（AI 模型）
- **用途**：Info Fetcher 运行时需要 AI 模型
- **申请地址**：https://open.bigmodel.cn/
- **费用**：免费额度充足
- **步骤**：
  1. 注册账号（手机号）
  2. 进入「API Keys」页面
  3. 点击「创建 API Key」
  4. 复制 Key，保存好

#### 5.2 Brave Search（搜索）
- **用途**：关键词搜索、线索发现
- **申请地址**：https://brave.com/search/api/
- **费用**：免费版每月 2000 次查询
- **步骤**：
  1. 注册 Brave 账号
  2. 选择 "Free" 计划
  3. 获取 API Key

### 🟡 按需配置（有免费额度）

#### 5.3 Tavily（搜索，替代 Brave）
- **申请地址**：https://tavily.com/
- **费用**：免费 1000 次/月
- **和 Brave 二选一即可**

#### 5.4 GitHub Token（GitHub 调研）
- **申请地址**：https://github.com/settings/tokens
- **费用**：免费
- **步骤**：
  1. GitHub → Settings → Developer settings → Personal access tokens
  2. Generate new token (classic)
  3. 勾选 `public_repo` 权限
  4. 复制 Token

#### 5.5 Firecrawl（站点抓取）
- **申请地址**：https://www.firecrawl.dev/
- **费用**：免费 500 次/月
- **用途**：批量抓取整个网站时使用

### 🔴 高级功能（按需付费）

| 服务 | 用途 | 费用 | 申请地址 |
|------|------|------|---------|
| Browserbase | 浏览器自动化 | 付费 | https://www.browserbase.com/ |
| Apify | 托管抓取 | 免费额度 | https://apify.com/ |
| YouTube Data API | 视频元数据 | 免费 | https://console.cloud.google.com/ |

### 📋 API Key 申请优先级

```
如果你只想试试（0成本）：
  ✅ GLM API Key（AI模型）
  ✅ Brave Search API Key（搜索）
  → 可以用 web-read + search-discovery 两个 Skill

如果你想完整体验（仍然0成本）：
  ✅ 加上 GitHub Token
  ✅ 加上 Tavily API Key
  ✅ 加上 Firecrawl API Key
  → 可以用 6/10 个 Skill

如果你是专业用户：
  🔧 全部配置
```

---

## 6. 第四步：配置环境变量

编辑 `~/.openclaw/workspace-info-fetcher/.env`：

```bash
# 用你喜欢的编辑器打开
nano ~/.openclaw/workspace-info-fetcher/.env
```

填入你申请到的 Key：

```env
# ── 搜索层 ──────────────────────────────
BRAVE_API_KEY=BSA_xxxxx你的真实key
TAVILY_API_KEY=tvly-xxxxx你的真实key
EXA_API_KEY=

# ── 托管抓取层 ──────────────────────────
FIRECRAWL_API_KEY=fc-xxxxx你的真实key

# ── 平台专项层 ──────────────────────────
GITHUB_TOKEN=ghp_xxxxx你的真实token
YOUTUBE_API_KEY=
```

**只填你有 Key 的项，其他留空即可。**

然后重启 OpenClaw：

```bash
openclaw restart
```

---

## 7. 第五步：测试使用

现在你可以直接在聊天中测试了！

### 测试 1：读取网页

在 Telegram / Discord / 命令行中发消息：

> 帮我读取这个网页的内容：https://example.com

AI 会自动识别这是 URL 抓取任务，调用 info-fetcher 处理。

### 测试 2：搜索

> 搜一下最近关于 AI 视频生成的最新动态

### 测试 3：GitHub 调研

> 帮我看看 https://github.com/openclaw/openclaw 这个项目是干什么的

### 测试 4：RSS 读取

> 读取这个 RSS feed：https://hnrss.org/frontpage

---

## 8. 常见问题

### Q: 启动后没有反应？
检查 OpenClaw 是否在运行：
```bash
openclaw status
```

### Q: 抓取失败？
1. 检查网络连接
2. 检查 .env 中的 API Key 是否正确
3. 查看 OpenClaw 日志：
```bash
openclaw logs
```

### Q: 搜索返回空结果？
- Brave API 免费版每月 2000 次限制，可能用完了
- 换 Tavily 试试

### Q: 网页抓取不到内容？
- 有些网站禁止抓取，这是正常的
- 尝试在 URL 前加 jina.ai 代理：`https://r.jina.ai/https://目标网址`

### Q: Camofox（浏览器）报错？
Camofox 是浏览器自动化工具，用于 JS 动态页面。如果不需要抓取动态页面，可以忽略。

### Q: 我想修改 Info Fetcher 的行为？
编辑 `~/.openclaw/workspace-info-fetcher/AGENTS.md`，调整路由规则和任务分类逻辑。

### Q: 如何更新到最新版？
```bash
cd ~/.openclaw/workspace-info-fetcher
git pull
openclaw restart
```

---

## 📚 延伸阅读

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Info Fetcher 仓库](https://github.com/huangzuomin/openclaw-info-fetcher)

---

*有问题？在 GitHub 仓库开 Issue：https://github.com/huangzuomin/openclaw-info-fetcher/issues*
