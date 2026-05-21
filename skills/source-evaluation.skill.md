---
name: source_evaluation
description: 使用此 Skill 评估抓取结果的来源可靠性和证据等级，生成可靠性报告，供报告生成和知识库入库前的质量把关。
---

# source-evaluation — 来源可靠性评估

## 何时使用

满足以下条件时使用此 Skill：

- 抓取完成后，在入库前需要评估内容可靠性
- 需要明确区分线索和证据
- 准备生成正式报告，需要标注来源质量

## 不使用场景

- 只是快速查询，无需正式报告
- 来源明确为高可靠性（如政府官网），可跳过（但建议保留调用）

## 输入

```yaml
source_url: string              # 必填，来源 URL
source_type: string             # 必填，来源类型
  # 可选值：
  # web_article / news / government / academic
  # search_result / ai_search / rss / github
  # social_media / video / forum / official_api
content: string                 # 必填，已抓取的内容（正文 Markdown）
metadata: object                # 必填，metadata.json 内容
has_screenshot: boolean         # 可选，是否有截图（默认 false）
has_raw: boolean                # 可选，是否有原始 HTML（默认 false）
```

## 评估维度

### 1. 来源可信度（source_reliability）

```
high（高可信）：
  - 政府官网（.gov / .gov.cn）
  - 官方机构、权威媒体官网
  - 学术机构、期刊
  - 官方 API 数据（GitHub API、Twitter API）

medium（中等可信）：
  - 主流媒体网站
  - 知名博客、专业媒体
  - 公开 RSS Feed（已知来源）
  - 论坛内容（可引用，需注明）

low（低可信）：
  - 未知个人网站
  - 社媒帖子（无法验证发布者）
  - 匿名来源
  - AI 搜索聚合结果
```

### 2. 证据等级（evidence_level）

```
strong_evidence：有 URL + 完整正文 + 截图 + raw HTML + 元数据
evidence：有 URL + 完整正文 + 抓取时间 + 元数据
weak_evidence：有 URL，但正文不完整或无法确认发布时间
clue_only：无原始 URL，或为 AI 搜索摘要，或内容来源不可确认
```

### 3. 风险标志（risk_flags）

```
possible_paywall：内容疑似付费墙截断
dynamic_content_only：内容依赖 JS 渲染，可能不完整
login_required：内容需要登录才能完整查看
ai_generated：内容疑似由 AI 生成
outdated_content：内容日期超过 6 个月
no_author：无作者信息
no_publish_date：无发布时间
unverifiable：来源无法独立核验
```

## 工作流

1. 接收来源 URL、类型、内容和元数据
2. 评估 `source_reliability`（基于域名和来源类型）
3. 评估 `evidence_level`（基于内容完整性和留存材料）
4. 检测 `risk_flags`
5. 生成评估说明
6. 返回评估结果

## 输出

```yaml
source_url: string
source_type: string
source_reliability: low | medium | high
evidence_level: clue_only | weak_evidence | evidence | strong_evidence
risk_flags: list[string]
notes: string                   # 评估说明，说明判断依据
recommended_action: string      # 建议操作（直接入库 / 人工复核 / 二次核验 / 不建议使用）
```

## 评估原则

- 不对内容真实性作主观判断，只评估来源可验证性
- 搜索结果摘要无论质量多高，必须标注 `clue_only`
- 官方政府网站内容不等于内容一定正确，只是来源可靠性高
- 有截图的内容可作为"曾经存在"的证据，但内容本身仍需复核

## 对下游的说明

评估结果可直接追加到 `metadata.json` 中的 `source_reliability` 和 `evidence_level` 字段，然后调用 save-to-kb 入库。
