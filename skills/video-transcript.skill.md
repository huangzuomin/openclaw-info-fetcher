---
name: video_transcript
description: 使用此 Skill 获取视频的字幕、文字稿和元数据，支持 YouTube、B站等平台的公开视频。不涉及视频下载，只获取文字和元数据。
---

# video-transcript — 视频字幕与元数据采集

## 何时使用

满足以下条件时使用此 Skill：

- 用户提供了 YouTube、B站等平台的视频链接
- 需要获取视频字幕、文字稿用于研究或知识提取
- 需要获取视频标题、作者、发布时间、简介等元数据

## 不使用场景

- 需要下载视频文件 → 不在范围内（且通常违反平台 ToS）
- 需要截图或截帧 → 使用 browser-fetch
- 平台需要登录才能查看视频 → 需先评估合规性

## 输入

```yaml
video_url: string              # 必填，视频 URL（YouTube、B站等）
platform: youtube | bilibili | other  # 可选，平台（自动识别）
language: string               # 可选，字幕语言偏好（如 "zh-Hans"、"en"）
auto_generated_ok: boolean     # 可选，是否接受 AI 自动生成字幕（默认 true）
task_id: string                # 可选，自定义任务 ID
```

## 支持平台

| 平台 | 字幕方式 | 工具 |
|------|---------|------|
| YouTube | 官方字幕 / 自动生成 | yt-dlp（只抓字幕，不下载视频）/ YouTube API |
| B站 | 官方字幕 / CC 字幕 | B站 API / MediaCrawler |
| 其他平台 | 无标准字幕 | browser-fetch + 手动提取（兜底） |

## 工具路由

```
YouTube：
  1. yt-dlp --write-sub --skip-download（只下载字幕文件）
  2. YouTube Data API v3（YOUTUBE_API_KEY）
  3. Jina Reader / web-read（抓取自动生成字幕页面）

B站：
  1. B站公开 API（无需 Key，速率限制严格）
  2. MediaCrawler（如已安装）
  3. web-read + 字幕解析（兜底）
```

## 工作流

1. 接收视频 URL，解析平台
2. 获取视频元数据（标题、作者、发布时间、简介）
3. 获取可用字幕列表
4. 选择目标语言字幕（优先用户指定语言）
5. 下载字幕文件（.vtt / .srt 格式）
6. 转换为纯文本 Markdown
7. 生成内容摘要
8. 标注 evidence_level
9. 保存并返回结果

## 证据等级

```
evidence：有视频 URL + 字幕文本 + 元数据 + 抓取时间
weak_evidence：有视频 URL + 部分字幕（自动生成，可能有误）+ 元数据
clue_only：只有元数据，无字幕
```

**注意**：自动生成字幕的准确率不稳定，建议在报告中注明字幕类型（官方 vs 自动生成）。

## 输出

```yaml
task_id: string
video_url: string
platform: string
title: string
channel: string
channel_url: string
published_at: string | null
duration_seconds: integer | null
description: string
view_count: integer | null
language_detected: string
subtitle_type: official | auto_generated | none
transcript_markdown: string    # 字幕转 Markdown
summary: string                # 自动生成摘要
evidence_level: clue_only | weak_evidence | evidence
files:
  content: data/<date>/<task_id>/content.md
  metadata: data/<date>/<task_id>/metadata.json
  transcript_raw: data/<date>/<task_id>/transcript.vtt   # 如有
  fetch_log: data/<date>/<task_id>/fetch_log.json
```

## 失败处理

- 视频不存在或已删除：返回错误
- 无可用字幕：返回元数据，标注 `subtitle_type: none`，`evidence_level: clue_only`
- 仅有自动生成字幕：标注 `subtitle_type: auto_generated`，`evidence_level: weak_evidence`
- 平台封禁访问：标记 `risk_level: blocked`

## 合规说明

- 只获取字幕文字和元数据，不下载视频文件
- 遵守 YouTube ToS（不超速率限制，不绕过区域封锁）
- `risk_level: low`（公开视频的字幕信息）
- 不获取需要登录才能访问的视频（除非合法授权）
