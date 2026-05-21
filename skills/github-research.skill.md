---
name: github_research
description: 使用此 Skill 读取 GitHub 仓库信息，包括 README、基本统计、最近动态、Issue 和 Release，用于项目调研和技术跟踪。
---

# github-research — GitHub 仓库调研

## 何时使用

满足以下条件时使用此 Skill：

- 用户提供了 GitHub 仓库 URL 或 `owner/repo` 格式
- 需要了解项目基本信息、活跃度、技术栈
- 需要追踪特定项目的更新动态

## 不使用场景

- 需要克隆或修改代码 → 不在范围内（超出信息抓取边界）
- 需要私有仓库信息 → 需要 GitHub Token，且必须是合法授权访问
- 需要搜索 GitHub 上的项目 → 使用 search-discovery（`platform_hint: github`）

## 输入

```yaml
repo_url: string             # 必填，GitHub 仓库 URL 或 "owner/repo" 格式
include_readme: boolean      # 可选，是否读取 README（默认 true）
include_releases: boolean    # 可选，是否读取最新 Releases（默认 true）
include_issues: boolean      # 可选，是否读取最近 Issues（默认 false）
max_releases: integer        # 可选，最大 Release 数（默认 5）
max_issues: integer          # 可选，最大 Issue 数（默认 10）
task_id: string              # 可选，自定义任务 ID
```

## 工具路由

```
1. GitHub API（公开端点，无需 Token）
   → 首选，适合公开仓库，速率限制 60 次/小时
   → API: https://api.github.com/repos/{owner}/{repo}

2. GitHub API + Token（GITHUB_TOKEN 环境变量）
   → 如有 Token，速率限制提升到 5000 次/小时
   → 适合私有仓库（需要合法授权）

3. gh CLI（如本地已安装）
   → 备选，本地工具

4. Jina Reader / web-read
   → 兜底，直接读取 GitHub 网页版
```

## 工作流

1. 解析仓库 URL，提取 `owner/repo`
2. 调用 GitHub API 获取基本信息
3. 如 `include_readme=true`，获取 README 内容并转为 Markdown
4. 如 `include_releases=true`，获取最新 Releases
5. 如 `include_issues=true`，获取最近 Issues
6. 生成项目摘要
7. 标注 `evidence_level: evidence`（有原始 URL + API 数据 + 时间戳）
8. 保存并返回结果

## 输出

```yaml
task_id: string
repo: string                 # owner/repo 格式
html_url: string
description: string
stars: integer
forks: integer
open_issues: integer
language: string
topics: list[string]
created_at: string
updated_at: string
last_pushed_at: string
license: string | null
homepage: string | null
readme_markdown: string      # README 内容（如已抓取）
releases:
  - tag: string
    name: string
    published_at: string
    url: string
    body_summary: string
issues:
  - number: integer
    title: string
    state: string
    created_at: string
    url: string
summary: string              # 自动生成项目摘要
evidence_level: evidence
files:
  content: data/<date>/<task_id>/content.md
  metadata: data/<date>/<task_id>/metadata.json
  fetch_log: data/<date>/<task_id>/fetch_log.json
```

## 失败处理

- 仓库不存在或无权访问（404 / 403）：返回错误，说明原因
- API 速率限制（429）：等待后重试，或提示用户配置 `GITHUB_TOKEN`
- README 过长：只保留前 3000 字，标注已截断

## 合规说明

- 只抓取公开仓库信息（默认）
- 私有仓库需要合法 Token，且操作范围仅限于已授权的读取权限
- `risk_level: low`（公开信息）
- 不保存 GitHub Token 到日志或文档
