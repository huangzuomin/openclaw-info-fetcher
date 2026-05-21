---
name: site_crawl
description: 使用此 Skill 当用户需要批量抓取一个网站的栏目、专题页或整个站点时，生成 URL 清单和结构化索引。
---

# site-crawl — 站点批量抓取

## 何时使用

满足以下条件时使用此 Skill：

- 用户需要抓取某个网站的多个页面（栏目、专题、分类等）
- 需要生成 URL 清单或批量读取正文
- 单 URL 读取（web-read）无法满足需求

## 不使用场景

- 单个 URL 读取 → 使用 web-read
- 搜索发现线索 → 使用 search-discovery
- 站点明确禁止批量访问（robots.txt）→ 拒绝执行

## 风险说明

站点批量抓取属于 `risk_level: medium`。执行前必须：

1. 检查 robots.txt
2. 控制抓取频率（默认 1-3 秒间隔）
3. 不超过 max_pages 限制
4. 涉及国内平台时需额外谨慎

## 输入

```yaml
start_url: string               # 必填，起始 URL（栏目或站点首页）
scope: same_domain | path_only | custom  # 可选，抓取范围（默认 same_domain）
max_pages: integer              # 可选，最大页面数（默认 20，上限 200）
depth: integer                  # 可选，最大深度（默认 2）
include_patterns: list[string]  # 可选，只包含匹配此正则的 URL
exclude_patterns: list[string]  # 可选，排除匹配此正则的 URL
fetch_content: boolean          # 可选，是否抓取正文（默认 false，只生成 URL 清单）
delay_seconds: float            # 可选，请求间隔秒数（默认 1.5）
task_id: string                 # 可选，自定义任务 ID
```

## 工具路由

```
阶段一：URL 发现（map）
  1. Firecrawl map（FIRECRAWL_API_KEY）
  2. XCrawl map（XCRAWL_API_KEY）
  3. Crawl4AI crawl（本地安装）
  4. 自定义 sitemap.xml 解析（兜底）

阶段二：正文抓取（crawl，如 fetch_content=true）
  1. Firecrawl crawl
  2. XCrawl crawl
  3. Crawl4AI
  4. 逐 URL 调用 web-read（最慢但最稳定）
```

## 工作流

1. 接收起始 URL，生成 task_id
2. 检查 robots.txt，如有明确禁止则停止
3. 执行 map 阶段：发现所有候选 URL
4. 应用 scope、include/exclude 过滤
5. 限制到 max_pages
6. 如 `fetch_content=true`，对每个 URL 抓取正文
7. 记录每个 URL 的抓取状态（success / failed / skipped）
8. 生成 index.json
9. 如失败率超过 30%，在日志中标注并建议切换工具
10. 返回统计报告

## 输出

```yaml
task_id: string
site: string
start_url: string
url_count: integer
success_count: integer
failed_count: integer
skipped_count: integer
urls:
  - url: string
    status: success | failed | skipped
    title: string | null
    content_file: string | null
    error: string | null
index_file: data/<date>/<task_id>/index.json
fetch_log: data/<date>/<task_id>/fetch_log.json
```

## index.json 格式

```json
{
  "site": "example.com",
  "start_url": "https://example.com/news/",
  "generated_at": "2026-05-06T10:00:00+08:00",
  "url_count": 50,
  "items": [
    {
      "url": "https://example.com/news/article-1",
      "title": "文章标题",
      "status": "success",
      "content_file": "data/2026-05-06/task-001/content/article-1.md"
    }
  ]
}
```

## 失败处理

- robots.txt 禁止：标记 `risk_level: blocked`，拒绝执行
- 单个 URL 失败：记录，继续其他 URL
- 总失败率 > 30%：记录警告，建议换工具或降低频率
- 工具无 API Key：降级到下一级工具

## 合规说明

- 抓取前检查 robots.txt
- 遵守 Crawl-delay 指令
- 不抓取登录后才可见的内容
- 不高频并发请求（默认 1.5 秒间隔）
- 默认 `risk_level: medium`
