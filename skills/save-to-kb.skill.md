---
name: save_to_kb
description: 使用此 Skill 将抓取结果统一保存到结构化知识库目录，生成 content.md、metadata.json 和 fetch_log.json。
---

# save-to-kb — 统一入库

## 何时使用

满足以下条件时使用此 Skill：

- 已完成一次抓取，需要将结果持久化保存
- 需要按标准目录结构（`data/<date>/<task_id>/`）归档
- 需要生成或更新索引文件

此 Skill 是所有抓取链路的终点，每次成功抓取后都应调用。

## 不使用场景

- 尚未抓取任何内容（先抓取再入库）
- 需要评估来源可靠性 → 先调用 source-evaluation，再入库

## 输入

```yaml
task_id: string              # 必填，任务 ID（格式：<date>-<hash>）
content_markdown: string     # 必填，正文 Markdown 内容
metadata: object             # 必填，符合 metadata.json 规范的对象
fetch_log: object            # 必填，符合 fetch_log.json 规范的对象
raw_file_path: string        # 可选，raw.html 或 raw.json 的临时路径
screenshot_file_path: string # 可选，screenshot.png 的临时路径
attachments: list[string]    # 可选，其他附件文件路径
overwrite: boolean           # 可选，是否覆盖同名任务目录（默认 false）
```

## 工作流

1. 根据 `task_id` 中的日期部分确定目录：`data/<date>/<task_id>/`
2. 检查目录是否已存在：
   - 已存在且 `overwrite=false`：返回错误，不覆盖
   - 已存在且 `overwrite=true`：备份后覆盖
   - 不存在：创建目录
3. 写入 `content.md`（插入 YAML frontmatter）
4. 写入 `metadata.json`（序列化 metadata 对象）
5. 写入 `fetch_log.json`（序列化 fetch_log 对象）
6. 如有 `raw_file_path`，复制到 `raw.html` 或 `raw.json`
7. 如有 `screenshot_file_path`，复制到 `screenshot.png`
8. 更新或创建 `data/<date>/index.json`（追加本次任务记录）
9. 返回保存结果

## 目录结构

```text
data/
  <YYYY-MM-DD>/
    index.json               ← 当天所有任务的索引
    <task_id>/
      content.md             ← 必须
      metadata.json          ← 必须
      fetch_log.json         ← 必须
      raw.html               ← 可选
      raw.json               ← 可选
      screenshot.png         ← 可选
      attachments/           ← 可选
```

## 输出

```yaml
saved: boolean
task_id: string
date: string
paths:
  root: data/<date>/<task_id>/
  content: data/<date>/<task_id>/content.md
  metadata: data/<date>/<task_id>/metadata.json
  fetch_log: data/<date>/<task_id>/fetch_log.json
  raw: data/<date>/<task_id>/raw.html        # 如有
  screenshot: data/<date>/<task_id>/screenshot.png  # 如有
index_updated: boolean
```

## index.json 格式

```json
[
  {
    "task_id": "20260506-abc123",
    "title": "文章标题",
    "url": "https://example.com/article",
    "evidence_level": "evidence",
    "fetch_tool": "jina_reader",
    "fetched_at": "2026-05-06T10:00:00+08:00",
    "saved_at": "2026-05-06T10:00:05+08:00",
    "directory": "data/2026-05-06/20260506-abc123/"
  }
]
```

## 安全规则

- 不在 content.md 或 metadata.json 中保存明文 Cookie、Token、API Key
- 不在日志中保存账号密码
- 保存路径必须在项目目录内，不允许写入绝对路径外的位置

## 对下游的说明

保存完成后，`data/<date>/index.json` 可用于后续检索、变更检测和报告生成。
