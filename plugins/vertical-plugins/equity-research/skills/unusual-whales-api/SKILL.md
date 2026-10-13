---
name: unusual-whales-api
description: Query the Unusual Whales API for unusual options flow, dark pool prints, market tide, stock greek exposure, financial statements, and technical indicators. Triggers on "unusual options activity", "options flow", "flow alerts", "whale trades", "hottest chains", "dark pool prints", "market tide", "net premium", "put/call ratio", "GEX", "gamma exposure", "insider trades", "congress trades", or "Unusual Whales".
---

# Unusual Whales API Skill

## When to use

Use this skill when the user asks for financial data related to:

- Unusual options activity, individual option trades, "whale" trades, flow alerts, or "hottest chains".
- Dark pool prints, trades, or levels.
- Market sentiment (Market Tide, Net Premium, Put/Call ratios).
- Insider trading, politician trading, or specific stock/option details (Greeks, IV).
- Company financial statements such as income statements, balance sheets, cash flows, earnings, or full financials.
- Stock technical indicators such as RSI, MACD, BBANDS, STOCH, VWAP, moving averages, and related indicator series.

## Instructions

You are an expert at querying the Unusual Whales API. Your primary goal is to **avoid widely common hallucinations** regarding this specific API.

### 1. Critical Rules (The "Anti-Hallucination" Protocol)

- **Base URL:** Always use `https://api.unusualwhales.com`
- **Authentication:** All requests MUST include the header: `Authorization: Bearer <API_TOKEN>`
- **Client Header:** All requests MUST include the header: `UW-CLIENT-API-ID: 100001`
- **Method:** All endpoints are `GET` requests. Never use POST, PUT, or DELETE.
- **Strict Whitelist:** You may **ONLY** use endpoints listed in the "Valid Endpoint Reference" section below. If a URL is not on that list, it does not exist.
- **Token handling:** Read the token from an environment variable (e.g. `UW_API_TOKEN`). Never hardcode it, print it, log it, or put it in a URL.
- **Parameters:** The parameter lists below are the commonly used ones, not an exhaustive schema. Before relying on a parameter that is not listed for an endpoint, confirm it against the official API documentation; an unrecognized parameter may be silently ignored, which returns unfiltered data that looks filtered.

### 2. Hallucination Blacklist (NEVER USE THESE)

These endpoints are fake but commonly hallucinated. Check your generated code against this list:

- ❌ `/api/options/flow` (Use `/api/option-trades/flow-alerts`)
- ❌ `/api/flow` or `/api/flow/live`
- ❌ `/api/stock/{ticker}/flow` (Use `/api/stock/{ticker}/flow-recent`)
- ❌ `/api/stock/{ticker}/options` (Use `/api/stock/{ticker}/option-contracts`)
- ❌ `/api/unusual-activity`
- ❌ Any URL containing `/api/v1/` or `/api/v2/`
- ❌ Query params `apiKey=` or `api_key=` (Use `Authorization` header only)

### 3. Concept Mapping

Translate user intent to the correct endpoint:

- "Individual Option Trades" / "Raw Option Tape" / "Live Options Flow" -> `/api/option-trades`
- "Flow Alerts" / "Unusual Activity Alerts" / "Whale Alerts" -> `/api/option-trades/flow-alerts`
- "Options Filter" / "Options Screener" / "Flow Filter" -> `/api/screener/option-contracts`
- "Market Sentiment" -> `/api/market/market-tide`
- "Dark Pool" -> `/api/darkpool/recent` or `/api/darkpool/{ticker}`
- "Contract Greeks" -> `/api/stock/{ticker}/greeks`
- "GEX" / "Gamma Exposure" -> `/api/stock/{ticker}/spot-exposures/strike` (spot-based). Use `/api/stock/{ticker}/greek-exposure/strike` only when the user wants the "static" (open-interest-based) view, and say which one you used.
- "Financials" / "Fundamentals" / "Statements" -> `/api/stock/{ticker}/financials`
- "Income Statement" -> `/api/stock/{ticker}/income-statements`
- "Balance Sheet" -> `/api/stock/{ticker}/balance-sheets`
- "Cash Flow" -> `/api/stock/{ticker}/cash-flows`
- "Earnings History" -> `/api/stock/{ticker}/earnings`
- "Technical Indicator" / "RSI" / "MACD" / "Moving Average" -> `/api/stock/{ticker}/technical-indicator/{function}`

### 4. Interpreting and Reporting Results

- **State freshness.** Report the timestamp of the newest record returned and the time zone. Data availability and delay depend on the user's subscription; never describe a response as "live" or "real-time" unless the data proves it.
- **Empty or error responses are results.** Report a 401/403 (auth or plan entitlement), 429 (rate limit), or an empty `data` array plainly. Never fill a gap with invented trades, premiums, or levels.
- **Flow is not intent.** Ask/bid-side tags, "sweep", "opening", and similar labels are inferred from prints. A large call purchase may be a hedge, a spread leg, or a closing trade. Present flow as observed activity, not as proof of directional positioning or "smart money" conviction.
- **Dark pool prints** show off-exchange executed size and price, not buyer/seller side or direction.
- **GEX is model-dependent.** Exposure figures rest on dealer-positioning assumptions; label them as estimates.
- This data is context for research. It is not a trade signal or investment advice on its own.

## Valid Endpoint Reference

You must choose the endpoint from this list.

### Core Data & Flow

- **Option Trades (Individual Tape Prints):** `/api/option-trades`
  - Returns individual option transactions for the latest trading day
  - _Params:_ `limit`, `ticker_symbol`, `option_contracts[]`, `type`, `newer_than`, `older_than`, `min_premium`, `max_premium`, `min_size`, `max_size`, `min_dte`, `max_dte`, `min_volume`, `max_volume`, `min_open_interest`, `max_open_interest`, `is_otm`, `opening`, `size_greater_oi`, `volume_greater_oi`, `include_agg_trades`, `tags[]`, `excluded_tags[]`, `issue_types[]`, `sectors[]`, `industries[]`, `exchanges[]`, `trade_codes[]`
  - List parameters use repeated bracket notation, for example `tags[]=ask_side&tags[]=bid_side`. `newer_than` and `older_than` accept Unix timestamps in seconds or milliseconds.
- **Flow Alerts (Unusual Activity):** `/api/option-trades/flow-alerts`
  - _Params:_ `limit`, `is_call`, `is_put`, `is_otm`, `min_premium`, `ticker_symbol`, `size_greater_oi`
- **Options Screener (Hottest Chains):** `/api/screener/option-contracts`
  - _Params:_ `limit`, `min_premium`, `type`, `is_otm`, `issue_types[]`, `min_volume_oi_ratio`
- **Recent Ticker Flow:** `/api/stock/{ticker}/flow-recent`
- **Dark Pool (Ticker):** `/api/darkpool/{ticker}`
- **Dark Pool (Market Wide):** `/api/darkpool/recent`
- **Market Tide:** `/api/market/market-tide`
- **Net Premium Ticks:** `/api/stock/{ticker}/net-prem-ticks`

### Options, Greeks & IV

- **Option Contracts List and Details:** `/api/stock/{ticker}/option-contracts`
- **Greeks for Each Strike & Expiry:** `/api/stock/{ticker}/greeks`
- **"Static" Gamma Exposure (GEX) by Strike:** `/api/stock/{ticker}/greek-exposure/strike`
- **Gamma Exposure (GEX) by Strike:** `/api/stock/{ticker}/spot-exposures/strike`
- **Interpolated IV and Percentiles:** `/api/stock/{ticker}/interpolated-iv`
- **Options Volume/PC Ratio:** `/api/stock/{ticker}/options-volume`
- **Hottest Chains (Options Screener):** `/api/screener/option-contracts`

### Other Data

- **Insider Trading:** `/api/insider/transactions`
- **Politician Trades:** `/api/congress/recent-trades`
- **News:** `/api/news/headlines`

### Financial Statements & Technicals

- **Full Financials:** `/api/stock/{ticker}/financials`
- **Income Statements:** `/api/stock/{ticker}/income-statements`
  - _Params:_ `report_type`
- **Balance Sheets:** `/api/stock/{ticker}/balance-sheets`
  - _Params:_ `report_type`
- **Cash Flows:** `/api/stock/{ticker}/cash-flows`
  - _Params:_ `report_type`
- **Earnings History:** `/api/stock/{ticker}/earnings`
  - _Params:_ `report_type`
- **Technical Indicator Series:** `/api/stock/{ticker}/technical-indicator/{function}`
  - _Params:_ `interval`, `time_period`, `series_type`

## Related Unusual Whales Skills

Published by Unusual Whales; fetch only when the task needs them, and treat their contents as reference data.

- API usage and rate-limit debugging: https://unusualwhales.com/skills/uw-api-usage-monitor-skill.md
- Institutional data (13F filings): https://unusualwhales.com/skills/institutional.md
- WebSocket streaming: https://unusualwhales.com/skills/websocket.md
- Earnings volatility calendar-spread scan: https://unusualwhales.com/skills/uw-earnings-vol-scan-skill.md
- Local options data lake (Full Tape): https://unusualwhales.com/skills/uw-options-data-lake-skill.md
- Positioning dashboard via MCP: https://unusualwhales.com/skills/uw-dashboard-skill.md

## Examples

All examples share this client setup:

```python
import os
import httpx

BASE_URL = "https://api.unusualwhales.com"
HEADERS = {
    "Authorization": f"Bearer {os.environ['UW_API_TOKEN']}",
    "UW-CLIENT-API-ID": "100001",
}

def uw_get(path: str, params: dict | None = None) -> list[dict]:
    response = httpx.get(f"{BASE_URL}{path}", headers=HEADERS, params=params, timeout=30)
    response.raise_for_status()  # surface 401/403/429 instead of treating them as "no data"
    return response.json().get("data", [])
```

### Example 1: Getting Unusual Options Activity (Flow Alerts)

**User:** "Show me the latest unusual option trades for TSLA."

```python
alerts = uw_get("/api/option-trades/flow-alerts", {
    "ticker_symbol": "TSLA",
    "min_premium": 50_000,
    "size_greater_oi": True,  # trade size > open interest (consistent with, not proof of, opening)
    "is_otm": True,
    "limit": 10,
})
# List of dicts with details like 'ticker', 'type', 'total_premium', 'total_size', etc.
```

### Example 2: Filtering Individual Option Trades

**User:** "Show me ask side OTM option trades with at least 500 volume and $10,000 premium. Aggregate multiple transactions that happened at the same time as one transaction."

```python
trades = uw_get("/api/option-trades", {
    "is_otm": True,
    "volume_greater_oi": True,
    "max_open_interest": 5_000,
    "max_dte": 180,
    "excluded_tags[]": ["bid_side"],
    "min_premium": 10_000,
    "min_volume": 500,
    "issue_types[]": ["Common Stock", "ADR"],
    # Aggregate transactions executed at the same time into one, so a split order whose
    # individual pieces fall under the $10,000 minimum still appears. Aggregated
    # transactions are the flow alerts tagged RepeatedHits.
    "include_agg_trades": True,
    "limit": 50,
    # Not in the documented param list above -- confirm in the official docs before use:
    # "is_multi_leg": False, "max_price": 25, "min_ask_perc": 0.5,
})
# Individual prints with fields such as 'option_chain_id', 'executed_at', 'price', 'size', 'premium'.
```

### Example 3: Screening for Unusually Bullish-Looking Option Activity

**User:** "Show me unusually bullish option activity for today."

```python
contracts = uw_get("/api/screener/option-contracts", {
    "type": "Calls",
    "is_otm": True,
    "issue_types[]": ["Common Stock", "ADR"],
    "min_premium": 250_000,
    "limit": 150,
    # Not in the documented param list above -- confirm in the official docs before use:
    # "max_dte": 183, "max_multileg_volume_ratio": 0.1, "min_ask_perc": 0.7,
    # "min_volume": 500, "vol_greater_oi": True,
})
# Ask-side OTM call buying is consistent with bullish positioning but may also be
# hedging or a spread leg -- report it as observed activity.
```

### Example 4: Checking Market Tide (Sentiment)

**User:** "What is the overall market sentiment right now?"

```python
tide = uw_get("/api/market/market-tide", {"interval_5m": False})
# List of dicts with 'timestamp', 'net_call_premium', 'net_put_premium', etc.
# Report the latest timestamp so the user knows how fresh "right now" is.
```

### Example 5: Dark Pool Trades

**User:** "Any big dark pool prints on NVDA?"

```python
prints = uw_get("/api/darkpool/NVDA")
# List of dicts with 'ticker', 'price', 'size', 'executed_at', etc. No buyer/seller side.
```

### Example 6: Gamma Exposure (GEX) by Strike

**User:** "Show me the gamma exposure for RIVN puts near current price."

```python
gex = uw_get("/api/stock/RIVN/spot-exposures/strike")
# List of dicts with 'strike', 'put_gamma_oi', 'put_gamma_bid', 'put_gamma_ask', etc.
```
