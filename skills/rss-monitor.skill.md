---
name: rss_monitor
description: 使用此 Skill 读取和监控 RSS/Atom Feed，获取最新条目列表，支持增量更新和变更检测。
---

# rss-monitor — RSS Feed 读取与监控

## 何时使用

满足以下条件时使用此 Skill：

- 用户提供了公开的 RSS 或 Atom Feed URL
- 需要获取 Feed 的最新条目列表
- 需要追踪某个来源的持续更新

## 不使用场景

- 目标站点无 RSS，但需要抓取页面更新 → 使用 site-crawl + 变更检测
- 需要读取具体文章全文 → 使用 web-read（对 Feed 条目 URL 调用）

## 输入

```yaml
feed_url: string             # 必填，RSS/Atom Feed URL
max_items: integer           # 可选，最大条目数（默认 20）
since: string                # 可选，只获取此时间之后的条目（ISO8601）
fetch_full_content: boolean  # 可选，是否对每条抓取全文（默认 false）
task_id: string              # 可选，自定义任务 ID
```

## 工具路由

```
1. feedparser（Python 库，无需 API Key）
   → 首选，支持 RSS 2.0 / Atom 1.0 / RSS 1.0

2. Agent Reach（如已安装）
   → 备选，支持更复杂的 Feed 场景
```

## 工作流

1. 接收 Feed URL
2. 调用 feedparser 解析
3. 提取条目列表：标题、链接、发布时间、摘要、作者
4. 应用时间过滤（`since` 参数）
5. 如 `fetch_full_content=true`，对每条调用 web-read
6. 为每条标注 `evidence_level`：
   - 仅 Feed 摘要：`weak_evidence`（有 URL 但内容不完整）
   - 抓取全文后：`evidence`
7. 保存结果
8. 返回结构化报告

## 输出

```yaml
task_id: string
feed_url: string
feed_title: string
feed_description: string
item_count: integer
items:
  - title: string
    url: string
    published_at: string
    summary: string
    author: string | null
    evidence_level: weak_evidence | evidence
    full_content_fetched: boolean
files:
  items: data/<date>/<task_id>/feed_items.json
  fetch_log: data/<date>/<task_id>/fetch_log.json
```

## 失败处理

- Feed URL 无效或返回错误：记录失败，返回错误信息
- Feed 格式不支持：记录失败，建议检查 URL 是否为有效 Feed
- 全文抓取失败：标注具体条目的 `full_content_fetched: false`，继续处理其他条目

## 合规说明

- RSS 条目本身是公开发布的内容，`risk_level: low`
- 抓取频率建议不超过每 5-30 分钟一次（根据 Feed 更新频率调整）
- 不抓取需要订阅或付费才能访问的 RSS 内容
