# AI-Powered News Digest Template

An n8n workflow that aggregates RSS feeds across multiple categories, filters articles by keywords, scores them using AI (Claude Haiku), and delivers a formatted HTML digest to your notification channel.

## What It Does

```
RSS Feeds (3 categories)
    ├── Sports ──→ Keyword Filter ──→ AI Score ──→ Filter ≥70
    ├── Tech ────→ Keyword Filter ──→ AI Score ──→ Filter ≥70
    └── World ──→ Keyword Filter ──→ AI Score ──→ Filter ≥70
                                                      │
                                         Merge + Deduplicate
                                                      │
                                         24h Filter + Limit
                                                      │
                                          Build HTML Digest
                                                      │
                                       Send to Notification
```

## Key Features

- **3 independent content branches** — Sports, Tech, World News (add/remove as needed)
- **Keyword filtering** — blacklist noise patterns, whitelist important topics (zero API cost)
- **AI relevance scoring** — Claude Haiku scores articles 0-100, only ≥70 pass through
- **Deduplication** — prevents the same story appearing from multiple sources
- **24-hour window** — only recent articles are included
- **HTML formatting** — clean, sectioned digest with emoji headers
- **Scheduled delivery** — default 3x daily (6am, 3pm, 6pm)

## Prerequisites

| Requirement | Purpose |
|------------|---------|
| **Anthropic API key** | Claude Haiku for AI scoring |
| **Matrix account** (or alternative) | Notification delivery |
| **n8n instance** | v1.0+ recommended |

## Setup Instructions

1. **Import** — Go to n8n → Workflows → Import from File → select `AI_News_Digest_Template.json`
2. **Create Anthropic credential** — Settings → Credentials → Add → Anthropic → paste your API key
3. **Create Matrix credential** (or your preferred notification service) — add access token and homeserver URL
4. **Update Matrix room IDs** — edit each Matrix node and set your room ID
5. **Customize RSS feeds** — edit the "Sports/Tech/World RSS Feeds" Code nodes
6. **Customize keywords** — edit the "Keyword Filter" Code nodes for each branch
7. **Set schedule** — adjust the Schedule Trigger nodes to your preferred times
8. **Activate** — toggle the workflow active

## Node Breakdown

### Trigger Nodes
| Node | Purpose |
|------|---------|
| Trigger Daily 6am / 3pm / 6pm | Scheduled execution times |
| Manual Trigger | For testing |

### Per-Category Branch (Sports / Tech / World)
| Node | Purpose |
|------|---------|
| RSS Feeds (Code) | Returns array of RSS feed URLs — **edit these for your interests** |
| Split RSS | Splits array into individual items |
| RSS Read | Fetches articles from each feed |
| Normalize Articles | Standardizes fields (title, link, date, source, category) |
| Keyword Filter (Code) | Blacklist/whitelist pattern matching — **edit patterns for your interests** |
| Filter 24h | Only articles from last 24 hours |
| Prepare for AI Scoring | Batches articles into compact format for AI |
| AI Score (Agent) | Claude Haiku scores each article 0-100 |
| Parse Scores & Filter ≥70 | Extracts scores, keeps only high-relevance articles |

### Final Output
| Node | Purpose |
|------|---------|
| Merge branches | Combines all categories |
| Dedupe + 24h Filter + Limit | Global deduplication and per-section limits |
| Build HTML | Creates formatted HTML with conditional sections |
| Has Content? | Skips sending if no articles passed filters |
| Send to Matrix | Delivers the digest |

## Customization Guide

### Change Topics
Edit the Code nodes named `Sports/Tech/World RSS Feeds`. Each returns an array of `{ name, rss_url, source_category }` objects.

### Adjust AI Scoring
- Edit the AI Agent node prompts to match your interests
- Change the threshold from 70 to higher (stricter) or lower (more articles)
- Switch the LLM model in the Claude Haiku sub-nodes

### Add a New Category
1. Duplicate one complete branch (RSS → Filter → Score → Parse)
2. Rename nodes to your new category
3. Connect the output to the Merge node
4. Update the Build HTML node to include the new section

### Change Notification Service
Replace the Matrix nodes with Slack, Discord, Telegram, Email, or any other n8n node. Map the HTML content to the message body field.

## Estimated Token Usage

Per execution (assuming ~50 articles per branch after keyword filtering):
- ~2,500 tokens per AI scoring call × 3 branches = ~7,500 tokens
- Using Claude Haiku: approximately $0.002 per execution
- At 3x daily: ~$0.18/month
