---
name: browser_fetch
description: 使用此 Skill 对需要 JavaScript 渲染、点击、滚动、截图或登录态的动态网页执行浏览器自动化抓取。这是最重的工具，仅在轻量读取无法满足时使用。
---

# browser-fetch — 浏览器自动化抓取

## 何时使用

满足以下条件时使用此 Skill：

- 目标页面依赖 JavaScript 渲染，轻量工具（Jina / Firecrawl）无法获取正文
- 需要截图作为证据留存
- 需要执行点击、滚动、展开等浏览器操作
- 需要使用登录态（授权访问）

**仅在 web-read 失败后使用此 Skill。**

## 不使用场景

- 普通静态网页 → 先尝试 web-read
- 只需要文字内容，无需截图或交互 → 先用 web-read
- 目标为付费墙或验证码页面 → 拒绝执行（blocked）

## 输入

```yaml
url: string                  # 必填，目标页面 URL
actions: list[object]        # 可选，需要执行的浏览器操作序列
  - type: click | scroll | wait | type | hover
    selector: string         # CSS 选择器
    value: string            # 输入内容（type 操作）
    direction: down | up     # 滚动方向
    pixels: integer          # 滚动距离
    seconds: float           # 等待秒数
need_login: boolean          # 可选，是否需要登录态（默认 false）
profile: string              # 可选，浏览器 Profile 名称（need_login=true 时必填）
need_screenshot: boolean     # 可选，是否截图（默认 true）
need_raw_html: boolean       # 可选，是否保存 raw.html（默认 false）
wait_for_selector: string    # 可选，等待某个元素出现再抓取
task_id: string              # 可选，自定义任务 ID
```

## 工具路由

```
1. OpenCLI（本地浏览器，首选）
   → 轻量、本地可控、适合大多数场景

2. Playwright（更细控制，备选）
   → 支持复杂交互、多步操作、网络拦截

3. web-access（可视化观察）
   → 需要人工观察时使用

4. Browserbase（云端浏览器）
   → 需要生产化、并发或持久会话时使用
   → 需要 BROWSERBASE_API_KEY
```

## 证据等级

```
evidence（标准浏览器抓取）：
  - 有 URL + 页面文本 + 截图 + 时间戳

strong_evidence（完整留存）：
  - 有 URL + 页面文本 + 截图 + raw.html + 操作日志
```

## 登录态规则

如 `need_login=true`：

1. 必须使用专用账号（非个人主账号）
2. 必须使用隔离 profile（`profiles/<profile>/`）
3. 不保存明文 Cookie 到日志
4. 必须事先确认该访问是合法授权的

如未提供合法 `profile`，拒绝执行登录态任务。

## 工作流

1. 接收 URL，生成 task_id
2. 风险评估：是否付费墙、验证码、明确禁止自动化？
3. 如有风险，标记并请求确认
4. 选择浏览器工具
5. 打开页面，等待加载
6. 执行 `actions` 序列（如有）
7. 等待 `wait_for_selector`（如有）
8. 抓取页面文本
9. 截图（如 `need_screenshot=true`）
10. 保存 raw.html（如 `need_raw_html=true`）
11. 标注 evidence_level
12. 保存并返回结果

## 输出

```yaml
task_id: string
final_url: string            # 最终 URL（可能经过重定向）
page_title: string
page_text: string            # 页面正文（纯文本或 Markdown）
evidence_level: evidence | strong_evidence
files:
  content: data/<date>/<task_id>/content.md
  metadata: data/<date>/<task_id>/metadata.json
  screenshot: data/<date>/<task_id>/screenshot.png    # 如有
  raw_html: data/<date>/<task_id>/raw.html            # 如有
  fetch_log: data/<date>/<task_id>/fetch_log.json
action_log:
  - action: string
    status: success | failed
    timestamp: string
```

## 失败处理

- 页面加载超时：记录失败，返回错误
- 验证码拦截：标记 `risk_level: blocked`，停止执行
- 付费墙检测：标记 `risk_level: blocked`，拒绝继续
- 操作步骤失败：记录失败步骤，保存已有截图，继续后续步骤

## 合规说明

- 不绕过付费墙和验证码
- 登录态必须使用授权账号
- `risk_level` 默认为 `medium`（浏览器自动化），登录态为 `high`
- 不自动化访问 robots.txt 明确禁止的页面
