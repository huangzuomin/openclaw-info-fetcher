---
name: search_discovery
description: 使用此 Skill 当用户提供关键词、话题或模糊信息需求，需要从搜索引擎发现线索时。所有搜索结果默认标记为 clue_only，必须二次核验才能升级为 evidence。
---

# search-discovery — 搜索线索发现

## 何时使用

满足以下条件时使用此 Skill：

- 用户提供关键词或话题描述，而非明确 URL
- 用户需要"找一下"、"有没有关于"、"最近有什么"类型的信息
- 需要从多个来源发现候选线索

## 不使用场景

- 用户已提供明确 URL → 使用 web-read
- 用户需要特定平台内容（Twitter / 微博 / 小红书等）→ 使用 social-fetch
- 用户需要 RSS 监控 → 使用 rss-monitor
- 用户需要 GitHub 项目信息 → 使用 github-research

## 重要规则：线索不是证据

**此 Skill 的所有输出默认为 `clue_only`。**

- 搜索结果的标题和摘要 ≠ 证据
- AI 搜索聚合的回答 ≠ 证据
- 即使摘要内容看起来完整，也必须通过 web-read 二次抓取原文后才能标记为 evidence

## 输入

```yaml
query: string                # 必填，搜索关键词或自然语言描述
language: zh | en | auto    # 可选，语言偏好（默认 auto）
platform_hint: string        # 可选，提示优先搜索哪个平台或来源（如 "government"、"github"）
time_range: string           # 可选，时间范围（如 "last_7_days"、"last_month"）
max_results: integer         # 可选，最大结果数（默认 10）
auto_fetch_top: integer      # 可选，自动对前 N 条结果执行 web-read 二次抓取（默认 0，不自动抓取）
task_id: string              # 可选，自定义任务 ID
```

## 工具路由

按以下顺序尝试，至少需要一个可用：

```
1. Brave Search API（BRAVE_API_KEY 环境变量）
   → 首选，结果质量高，有原始链接
   
2. Tavily Search API（TAVILY_API_KEY 环境变量）
   → 备选，支持中文查询

3. Exa API（EXA_API_KEY 环境变量）
   → 适合技术内容和 Semantic 搜索

4. AI-Search-Hub（集成 AI 搜索聚合）
   → 最后手段，结果无原始链接，只能作为 clue
```

如果所有工具都不可用（无 API Key），返回提示并建议用户配置至少一个搜索 API Key。

## 工作流

1. 接收搜索请求，生成 task_id
2. 判断语言和平台偏好
3. 按路由顺序调用搜索工具
4. 去重（相同域名 + 相近标题）
5. 对每条结果标注：
   - `evidence_level: clue_only`
   - `verified: false`
   - `source_type: search_result`
6. 生成候选 URL 推荐列表（按相关性排序）
7. 如 `auto_fetch_top > 0`，对前 N 条调用 web-read
8. 保存线索文件
9. 返回结构化报告

## 证据等级处理

```
所有搜索结果初始 evidence_level = clue_only
经 web-read 成功抓取原文后 → evidence
经 web-read + 截图后 → strong_evidence
```

**不允许**跳过 web-read 直接将搜索结果标记为 evidence。

## 输出

```yaml
task_id: string
query: string
sources_used: list[string]       # 实际使用的搜索工具
result_count: integer
clue_count: integer
results:
  - title: string
    url: string
    snippet: string
    source: string
    rank: integer
    evidence_level: clue_only    # 固定为 clue_only
    verified: false              # 固定为 false
    fetched_at: string
    source_type: search_result
recommended_urls: list[string]   # 推荐进一步抓取的 URL
files:
  clues: data/<date>/<task_id>/clues.json
  fetch_log: data/<date>/<task_id>/fetch_log.json
```

## 失败处理

- 所有搜索工具失败：返回错误，说明需要配置 API Key
- 部分工具失败：使用成功的工具结果，在 fetch_log 中记录失败工具
- 结果数量为零：返回空列表，建议修改关键词或平台

## 合规说明

- 不使用搜索结果作为正式报告的事实依据
- AI 搜索聚合结果（无原始链接）必须标注 `source_type: ai_search_result`
- 搜索结果仅用于发现线索，不用于直接引用
