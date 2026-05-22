---
name: social_fetch
description: 使用此 Skill 对特定社交媒体平台（X/Twitter、微博、小红书、抖音、B站、知乎、微信公众号等）执行专项内容采集。此 Skill 风险等级较高，执行前必须评估合规性。
---

# social-fetch - 社媒平台专项采集

## 何时使用

满足以下条件时使用此 Skill：

- 用户明确指定需要采集特定社媒平台的内容
- 目标内容在社媒平台上，无法通过普通 web-read 获取
- 任务属于研究、舆情分析等合法使用场景

## 不使用场景

- RSS Feed → 使用 rss-monitor
- GitHub → 使用 github-research
- YouTube 字幕 → 使用 video-transcript
- 普通公开网页 → 使用 web-read
- 需要批量采集个人隐私信息 → 拒绝执行（blocked）

## 重要风险说明

**此 Skill 默认 `risk_level: medium` 或 `high`（视平台）。**

国内平台（小红书、抖音、微博等）反爬措施强，需要：
1. 严格控制频率
2. 使用专用账号（非个人主账号）
3. 使用隔离 profile
4. 任务量大时必须人工确认

## 支持平台

| 平台 | 风险级别 | 推荐工具 |
|------|---------|---------|
| X / Twitter | medium | TweetClaw / Agent Reach / 6551 OpenTwitter |
| GitHub | low | github-research Skill（推荐直接使用该 Skill） |
| RSS | low | rss-monitor Skill（推荐直接使用该 Skill） |
| YouTube | low | video-transcript Skill |
| B站 | medium | Agent Reach / MediaCrawler |
| 微博 | medium | Agent Reach / MediaCrawler |
| 小红书 | high | MediaCrawler / 人工辅助 |
| 抖音 | high | MediaCrawler / 人工辅助 |
| 知乎 | medium | Agent Reach / 直接抓取公开页面 |
| 微信公众号 | high | 搜狗微信搜索 + Jina / 人工辅助 |

## 输入

```yaml
platform: string              # 必填，平台名称（见上表）
query: string                 # 可选，搜索关键词或话题
url: string                   # 可选，特定内容链接（帖子 / 用户主页 / 频道）
max_items: integer            # 可选，最大条目数（默认 20，高风险平台建议 ≤ 10）
time_range: string            # 可选，时间范围（如 "last_7_days"）
account_profile: string       # 可选，需要登录态时指定专用账号 profile
task_id: string               # 可选，自定义任务 ID
```

## 工具路由

```
1. TweetClaw（X/Twitter 首选，如已安装）
   → OpenClaw plugin；适合 search tweets、search tweet replies、follower export、user lookup、media download、monitor tweets、webhooks 和 giveaway draws
   → 如需 post tweets、post tweet replies、direct messages 或 media upload，先人工确认账号、内容和范围

2. Agent Reach（多平台备选，如已安装）
   → 支持多平台，有官方封装

3. MediaCrawler（国内平台备选）
   → 适合小红书、B站、微博、知乎、抖音
   → 需要本地安装和账号 Cookie

4. Apify（托管 Actors）
   → 国际平台备选，需要 APIFY_TOKEN

5. 6551 OpenTwitter / OpenNews
   → X/Twitter 专项备选

6. 直接 web-read（公开页面兜底）
   → 对于有公开 URL 的帖子，可降级到 web-read
```

## 证据等级

```
evidence：有帖子 URL + 文本 + 抓取时间
weak_evidence：有帖子文本但无法确认 URL 或发布时间
clue_only：只有摘要或搜索结果，无原始帖子 URL
```

## 工作流

1. 接收平台和查询信息
2. 风险评估：是否为 blocked 级别（批量隐私数据等）
3. 检查工具可用性
4. 如需登录态，验证是否使用专用账号 + 隔离 profile
5. 执行采集
6. 清洗内容，提取结构化字段
7. 标注 evidence_level 和 risk_level
8. 保存并返回结果

## 输出

```yaml
task_id: string
platform: string
query: string | null
item_count: integer
evidence_level: clue_only | weak_evidence | evidence
risk_level: medium | high
items:
  - platform: string
    item_url: string | null
    author: string
    author_id: string | null
    published_at: string | null
    text: string
    metrics:
      likes: integer | null
      reposts: integer | null
      comments: integer | null
    media: list[string]
    fetched_at: string
    fetch_tool: string
    evidence_level: string
files:
  items: data/<date>/<task_id>/social_items.json
  fetch_log: data/<date>/<task_id>/fetch_log.json
```

## 失败处理

- 工具不可用：返回错误，说明安装要求
- 平台封禁/验证码：标记 `blocked`，停止执行，建议降频或人工采集
- 账号风险：停止执行，不切换账号继续
- 单条内容失败：记录，继续其他内容

## 合规边界

- 不批量采集个人隐私信息（手机号、真实姓名、私信）
- 不使用个人主账号自动化操作
- 不规避平台封禁机制
- 高风险平台（小红书、抖音）默认需要人工确认才执行
- X/Twitter 写入、direct messages、media upload、giveaway draws 等动作必须先确认任务、账号、内容和范围
- 采集内容只用于研究、舆情分析等合法目的
