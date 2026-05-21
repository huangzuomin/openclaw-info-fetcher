# OpenClaw Info Fetcher

> An OpenClaw sub-agent for structured information gathering — from web pages, social media, search, RSS, GitHub, and video transcripts.

## What It Does

Info-fetcher is an **information retrieval dispatcher** that runs as a sub-agent inside [OpenClaw](https://github.com/openclaw/openclaw). It doesn't try to be a universal scraper — instead, it:

1. **Understands your intent** — What do you need? A full article? A search clue? A site map? Social media posts?
2. **Routes to the right tool** — Lightweight fetch → Crawl → Browser automation, based on task complexity
3. **Structures the output** — Every result comes with source URL, fetch timestamp, evidence level, and metadata
4. **Distinguishes clues from evidence** — AI search results are clues; fetched original content with provenance is evidence

## Architecture

```
User Request
    ↓
Task Classification (URL? Search? Platform? Batch?)
    ↓
Tool Routing
    ├── web-read (single URL, lightweight)
    ├── search-discovery (keyword → clue discovery)
    ├── site-crawl (batch URL map + crawl)
    ├── browser-fetch (JS rendering, login, screenshots)
    ├── social-fetch (X, Weibo, Bilibili, etc.)
    ├── video-transcript (YouTube, Bilibili subtitles)
    ├── rss-monitor (feed parsing)
    ├── github-research (repo analysis)
    └── source-evaluation (reliability scoring)
    ↓
Structured Output (content.md + metadata.json + fetch_log.json)
```

## Skills

| Skill | Description | Dependencies |
|-------|-------------|-------------|
| **web-read** | Single URL → Markdown | None (uses OpenClaw web_fetch) |
| **search-discovery** | Keyword → clue discovery | Brave/Tavily/Exa API key |
| **site-crawl** | Site map + batch crawl | Firecrawl/XCrawl API key |
| **browser-fetch** | JS rendering, login, screenshots | Camofox/Browserbase |
| **social-fetch** | X, Weibo, Bilibili, etc. | Platform-specific tools |
| **video-transcript** | YouTube/Bilibili subtitles | yt-dlp, YouTube API |
| **rss-monitor** | RSS/Atom feed parsing | None |
| **github-research** | Repo info + activity | GitHub token |
| **source-evaluation** | Reliability scoring | None (model-based) |
| **save-to-kb** | Structured knowledge base output | None |

## Setup

### Prerequisites

- [OpenClaw](https://github.com/openclaw/openclaw) installed and running
- API keys for the services you want to use (see `.env.example`)

### Installation

```bash
# 1. Clone into your OpenClaw workspace
cd ~/.openclaw/
git clone https://github.com/YOUR_USERNAME/openclaw-info-fetcher.git workspace-info-fetcher

# 2. Configure environment
cp workspace-info-fetcher/.env.example workspace-info-fetcher/.env
# Edit .env with your API keys

# 3. That's it — OpenClaw will detect the sub-agent automatically
```

### Usage from OpenClaw

In your main workspace's AGENTS.md, add routing rules:

```markdown
## Info Fetcher

When receiving URL fetch, web scraping, or information gathering requests, 
dispatch to the info-fetcher sub-agent:

\`\`\`
sessions_spawn(
  task="<specific task description>",
  cwd="/home/ai/.openclaw/workspace-info-fetcher",
  context="isolated"
)
\`\`\`
```

Then in conversation, just describe what you need:
- "Fetch this URL and summarize: https://..."
- "Search for recent papers on..."
- "Crawl this site's blog section: https://..."
- "Get the transcript from this YouTube video: https://..."

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and fill in the keys you need:

| Variable | Purpose | Required For |
|----------|---------|-------------|
| `BRAVE_API_KEY` | Brave Search API | search-discovery |
| `TAVILY_API_KEY` | Tavily Search API | search-discovery (alternative) |
| `EXA_API_KEY` | Exa Search API | search-discovery (alternative) |
| `FIRECRAWL_API_KEY` | Firecrawl | site-crawl |
| `GITHUB_TOKEN` | GitHub API | github-research |
| `YOUTUBE_API_KEY` | YouTube Data API | video-transcript |

### Tool Routing Priority

1. User wants clues or original content?
2. Is there a specific URL?
3. Does it need login/auth?
4. Is it a JS-heavy dynamic page?
5. Is it a platform-specific task (RSS/GitHub/YouTube/X)?
6. Batch or single?
7. Screenshots needed?
8. Any compliance or account risk?

## Output Format

Every fetch task produces:

```
output/
├── content.md          # Main content in Markdown
├── metadata.json       # Source, timestamp, tool, evidence level
├── fetch_log.json      # Tool calls, timing, errors
├── raw/                # Original HTML/text if preserved
└── screenshots/        # If applicable
```

### Evidence Levels

| Level | Meaning |
|-------|---------|
| `clue` | From AI search/aggregation — needs verification |
| `evidence` | Fetched from original source with URL + timestamp |
| `strong_evidence` | Multiple corroborating sources |

## Principles

- **Lightweight first** — Use the simplest tool that works
- **Clue ≠ Evidence** — Always distinguish between search results and fetched originals
- **Traceable** — Every result has source URL, fetch time, and tool used
- **Fail gracefully** — If primary tool fails, try fallback; if all fail, say so
- **Respect boundaries** — No paywall bypassing, no mass personal data scraping, no platform ToS violations

## License

MIT
