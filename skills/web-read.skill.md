---
name: web_read
description: 使用此 Skill 当用户提供明确的 URL，需要读取该网页正文并生成结构化输出时。
---

# web-read — 单 URL 网页读取

## 何时使用

满足以下条件时使用此 Skill：

- 用户提供了明确的 URL（一个或多个）
- 需要读取网页正文、标题、发布时间等信息
- 需要生成可入库的结构化结果
- 不需要登录态，不需要浏览器渲染（如需要，升级到 browser-fetch Skill）

## 不使用场景

- 用户只提供关键词，无明确 URL → 使用 search-discovery
- 目标页面需要 JS 渲染或登录 → 使用 browser-fetch
- 需要批量抓取整个站点 → 使用 site-crawl
- 已知平台专项内容（RSS / GitHub 仓库）→ 使用对应专项 Skill

## 输入

```yaml
url: string                  # 必填，目标网页 URL
preferred_tool: string       # 可选，指定优先工具（jina_reader / xcrawl / firecrawl / crawl4ai）
need_screenshot: boolean     # 可选，是否需要截图（默认 false）
need_raw: boolean            # 可选，是否保存 raw.html（默认 false）
task_id: string              # 可选，自定义任务 ID；不填则自动生成
```

## 工具路由

按以下顺序尝试，失败时自动降级：

```
1. Jina Reader（https://r.jina.ai/<url>）
   → 优先，无需 API Key，适合大多数公开网页
   
2. Firecrawl scrape
   → Jina 失败时，需要 FIRECRAWL_API_KEY
   
3. Crawl4AI
   → 前两者失败，需要本地安装 crawl4ai
   
4. OpenCLI / Playwright
   → 最终兜底，支持 JS 渲染，证据等级升级为 strong_evidence
```

每次切换工具必须记录到 fetch_log.json 中。

## 工作流

1. 接收 URL，生成 task_id（格式：`<date>-<hash前6位>`）
2. 判断是否满足使用条件（合规检查，参考 risk_policy）
3. 按路由顺序尝试读取
4. 提取：标题、正文（Markdown）、作者、发布时间、来源域名
5. 生成摘要（100-200 字）
6. 判断证据等级（见下方规则）
7. 按 `data/<date>/<task_id>/` 目录保存输出文件
8. 输出结构化报告

## 证据等级判定

```
evidence（标准）：
  - 有完整 URL
  - 有可读正文（> 100 字）
  - 有抓取时间
  - 有工具日志

strong_evidence（仅当以下全部满足）：
  - evidence 条件 + 有 screenshot.png + 有 raw.html

weak_evidence（正文不完整时）：
  - 有 URL，但正文过短（< 50 字）或疑似被截断
  - 无法确认发布时间
```

## 输出

```yaml
task_id: string
title: string
url: string
content_markdown: string    # 正文 Markdown
summary: string             # 自动生成摘要
metadata:
  source: string
  platform: web
  author: string | null
  published_at: string | null
  fetched_at: string
  fetch_tool: string
  language: string
  tags: list[string]
evidence_level: evidence | strong_evidence | weak_evidence
files:
  content: data/<date>/<task_id>/content.md
  metadata: data/<date>/<task_id>/metadata.json
  fetch_log: data/<date>/<task_id>/fetch_log.json
  raw: data/<date>/<task_id>/raw.html      # 如 need_raw=true
  screenshot: data/<date>/<task_id>/screenshot.png  # 如 need_screenshot=true
```

## 输出文件规范

### content.md

```markdown
---
id: <task_id>
title: <标题>
url: <url>
source: <域名>
platform: web
fetched_at: <ISO8601时间>
fetch_tool: <工具名>
evidence_level: <等级>
---

# <标题>

## 摘要

<100-200字摘要>

## 正文

<抓取后的正文>

## 来源信息

- URL: <url>
- 抓取时间: <时间>
- 工具: <工具>
- 证据等级: <等级>
```

### metadata.json

参见 `docs/07_data_schema.md` 第 3 节。

### fetch_log.json

参见 `docs/07_data_schema.md` 第 4 节。

## 失败处理

- 记录失败工具名称、错误信息、耗时
- 尝试下一个工具
- 如所有工具失败，输出 `status: failed`，列出失败原因
- 如目标站点 robots.txt 明确禁止，标记 `risk_level: blocked`，停止抓取

## 合规检查

执行前检查：

1. URL 格式是否合法
2. 目标站点是否为公开可访问资源
3. 是否有付费墙或验证码迹象（如有，降级为 blocked）
4. 频率是否合理（避免对同一域名高频请求）

## 对下游的说明

输出的 `content.md` 和 `metadata.json` 可直接传入 source-evaluation Skill 进行可靠性评估，也可直接保存到知识库。
