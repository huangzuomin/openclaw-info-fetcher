# TOOLS.md

## Tool Notes

此文件记录 info-fetcher 在运行时环境中可用的工具使用说明。此文件不授予工具访问权限，仅记录如何使用已配置好的工具。

## Tool Categories

### 搜索发现
- Web Search API：用于关键词搜索、线索发现。
- AI 搜索聚合：辅助发现线索，但结果标记为 clue。

### 轻量读取
- Web Reader / WebFetch：用于普通网页的轻量级 Markdown 转换。

### 站点抓取
- Map/Crawl 工具：用于站点批量抓取，生成 URL 清单和正文。

### 浏览器自动化
- Playwright / Browserbase / web-access：处理 JS 动态页面、登录态、截图需求。

### 平台专项
- Agent Reach：社媒平台数据采集。
- MediaCrawler：视频平台字幕和元数据。
- Apify：托管抓取 Actors。
- RSS 工具：RSS feed 解析和监控。
- GitHub API / gh CLI：仓库信息和动态追踪。

### 入库与结构化
- 统一输出目录结构：正文 Markdown、metadata.json、raw 原始材料、screenshot、fetch_log.json。

## Rules

- 根据任务类型和路由规则选择最合适的工具。
- 不发明不存在的工具或命令参数。
- 优先使用轻量工具，浏览器工具仅作为兜底。
- 记录每次工具调用的来源、时间和结果。
