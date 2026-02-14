# n8n Automation Portfolio

Production-ready n8n workflow templates for real-world automation tasks. Each template is fully documented with setup instructions, customization guides, and built-in workflow notes.

![n8n](https://img.shields.io/badge/n8n-workflow%20automation-orange)
![License](https://img.shields.io/badge/license-MIT-green)

## Workflows

### 1. AI-Powered News Digest
> Multi-category RSS aggregator with Claude AI relevance scoring

Fetches RSS feeds across Sports, Tech, and World News — filters by keywords, scores articles with Claude Haiku AI (0-100), and delivers a curated HTML digest 3x daily.

**Tech:** n8n, RSS, Claude AI (Anthropic), Matrix, HTML

[View Details](ai-news-digest/) · <a href="https://github.com/arm-ser/n8n-automation-portfolio/raw/main/ai-news-digest/AI-Powered%20News%20Digest%20Template.json" download>Download JSON</a>

---

### 2. Day Trading Pre-Market Scanner
> Real-time stock screener with bull flag pattern detection

Scans pre-market gainers every 5 minutes using 5 screening criteria (price, gap, float, volume), detects bull flag pullback patterns on 1-minute candles, and sends alerts with news catalysts.

**Tech:** n8n, REST APIs (mboum), Matrix, Technical Analysis

[View Details](day-trading-scanner/) · <a href="https://github.com/arm-ser/n8n-automation-portfolio/raw/main/day-trading-scanner/Day%20Trading%20Pre-Market%20Scanner%20Template.json" download>Download JSON</a>

> *Disclaimer: Educational template only. Not financial advice.*

---

### 3. Email 2FA OTP Extractor
> Smart email monitor that extracts and forwards OTP codes instantly

Watches IMAP mailboxes for 2FA emails, uses regex scoring to extract verification codes and sign-in links, deduplicates via PostgreSQL, and forwards to your notification channel.

**Tech:** n8n, IMAP, PostgreSQL, Regex, Matrix

[View Details](email-otp-extractor/) · <a href="https://github.com/arm-ser/n8n-automation-portfolio/raw/main/email-otp-extractor/Email%202FA%20OTP%20Extractor%20Template.json" download>Download JSON</a>

---

## How to Use

1. Click **Download JSON** for any workflow above
2. Open your n8n instance → **Workflows** → **Import from File**
3. Select the downloaded `.json` file
4. Follow the setup instructions in the workflow's sticky notes and the subfolder README
5. Configure your credentials and activate

## About

These templates are sanitized versions of production workflows running on a self-hosted n8n instance. All personal credentials and identifiers have been replaced with placeholders. Notification nodes use Matrix by default but can be swapped for Slack, Discord, Telegram, Email, or any other service.

Built with [n8n](https://n8n.io) — the workflow automation platform.
