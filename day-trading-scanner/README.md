# Day Trading Pre-Market Scanner Template

An n8n workflow that scans for pre-market stock gainers, filters them using Ross Cameron's trading criteria, detects bull flag pullback patterns on 1-minute candles, and sends actionable alerts with news catalysts to your notification channel.

> **Disclaimer:** This is an educational automation template. It is not financial advice. Always do your own research and consult a financial advisor before making trading decisions.

## What It Does

```
Schedule (every 5 min, pre-market hours)
        │
  Fetch Day Gainers (mboum API)
        │
  Filter 5 Criteria
  (Price $2-20, Gap 10%+, Float <10M, RelVol 5x)
        │
  Stocks Found? ──No──→ [end]
        │ Yes
  Enrich with Volume Data
        │
  Apply Volume Filter
        │
  Fetch 1-Min Candles
        │
  Detect Pullback Pattern (bull flag)
        │
  Alert Needed? ──No──→ Compile Watchlist → Send Summary
        │ Yes
  Fetch News Catalyst
        │
  Format Alert Message → Send Alert
```

## Key Features

- **5-criteria stock screener** — price range, gap %, float size, relative volume, pre-market activity
- **Bull flag pattern detection** — analyzes 1-minute candles for pullback setups with squeeze detection
- **Risk:reward calculation** — computes entry, stop loss, and target levels
- **News catalyst lookup** — fetches recent news for qualifying stocks
- **Watchlist compilation** — sends a summary of all qualifying stocks even without pullback patterns
- **Pre-market focused** — runs during optimal scanning hours (configurable)

## Prerequisites

| Requirement | Purpose |
|------------|---------|
| **mboum.com API key** | Market data (gainers, candles, news, volume) |
| **Matrix account** (or alternative) | Alert delivery |
| **n8n instance** | v1.0+ recommended |

## Setup Instructions

1. **Import** — Go to n8n → Workflows → Import from File → select `Day_Trading_Scanner_Template.json`
2. **Get mboum API key** — Sign up at [mboum.com](https://mboum.com) and get your API key
3. **Update API key** — In each HTTP Request node, replace `YOUR_MBOUM_API_KEY` in the query parameters
4. **Create Matrix credential** (or your preferred notification service) — add access token and homeserver URL
5. **Update Matrix room IDs** — edit each Matrix node and set your room ID
6. **Adjust schedule** — edit the Schedule Trigger cron expression for your timezone
7. **Activate** — toggle the workflow active

## Node Breakdown

| Node | Purpose |
|------|---------|
| Schedule (6-8:30 AM PT) | Cron trigger: `*/5 6-8 * * 1-5` (every 5 min, Mon-Fri) — **adjust for your timezone** |
| Fetch Day Gainers | GET mboum `/v2/markets/movers` — retrieves pre-market gainers |
| Filter 5 Criteria | Code node applying Ross Cameron's screening rules |
| Stocks Found? | If node — skips if no stocks qualify |
| Enrich with Volume Data | GET mboum `/v2/markets/stock/ticker-summary` — adds volume metrics |
| Apply Volume Filter | Filters for relative volume ≥ threshold |
| Fetch 1-Min Candles | GET mboum `/v1/markets/stock/history` — 1-minute price data |
| Detect Pullback Pattern | Analyzes candles for bull flag setups, squeeze, and R:R |
| Alert Needed? | If node — routes to detailed alert or watchlist summary |
| Fetch News | GET mboum `/v2/markets/news` — recent news for the stock |
| Format Alert Message | Builds formatted message with entry/stop/target and catalyst |
| Compile Watchlist | Summarizes all qualifying stocks into one message |
| Send Watchlist / Alert | Matrix nodes for notification delivery |

## Customization Guide

### Adjust Screening Criteria
Edit the `Filter 5 Criteria` Code node:
```javascript
const criteria = {
  minPrice: 2,      // Minimum stock price
  maxPrice: 20,     // Maximum stock price
  minGapPercent: 10, // Minimum pre-market gap %
  maxFloat: 10000,   // Max float in thousands (10000 = 10M shares)
  minRelativeVolume: 5  // Minimum relative volume multiplier
};
```

### Change Scanning Hours
Edit the Schedule Trigger node's cron expression:
- Default: `*/5 6-8 * * 1-5` (every 5 min, 6-8:30 AM Pacific, weekdays)
- For Eastern Time: `*/5 9-11 * * 1-5`
- For different intervals: change `*/5` to `*/10` for every 10 minutes

### Change Notification Service
Replace the Matrix nodes with Slack, Discord, Telegram, or Email. Map the alert message content to the message body field.

## API Usage

- mboum API calls per scan cycle: ~4 (gainers + volume + candles + news per qualifying stock)
- At every 5 minutes for 2.5 hours: ~30 cycles × 4 calls = ~120 API calls per trading day
- Check mboum.com pricing for your plan's rate limits
