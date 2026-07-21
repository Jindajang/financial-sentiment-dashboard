# Financial Sentiment Dashboard

An automated pipeline that pulls live stock price data and financial news, scores market sentiment with an LLM, stores the results, and surfaces both real-time alerts and a Power BI dashboard for tracking trends over time.

## What it does

On a schedule, the pipeline works through a watchlist of tickers and for each one:

1. **Pulls price data** — 1-minute, 15-minute, and 1-hour candlesticks from Twelve Data, merged and aggregated into a single price history.
2. **Gathers news** — recent articles from NewsAPI and sector-level news sentiment from Alpha Vantage, filtered and deduplicated (with a safety check for high-impact biotech events like FDA approvals).
3. **Scores sentiment with an LLM** — a Mistral-powered agent analyses the gathered articles against a tiered evidence framework (hard financial data outweighs analyst opinion, forward guidance outweighs past performance) and returns a structured `{category, score, rationale}` result.
4. **Combines and stores results** — sentiment scores are merged with the price data and written to a Supabase (Postgres) table.
5. **Alerts on results** — a summary and chart are sent via Telegram; failures are caught by a dedicated error handler and reported the same way.
6. **Visualises trends** — a Power BI dashboard connects to the Supabase table to track sentiment scores and price movement across the watchlist over time.

## Architecture

```
Schedule Trigger
      │
      ▼
Load Watchlist ──► Loop Over Items
      │                    │
      ▼                    ▼
Price Data (1m/15m/1h)   News + Sector Sentiment
   Twelve Data              NewsAPI + Alpha Vantage
      │                    │
      ▼                    ▼
Merge & Aggregate     Filter & Organise Articles
      │                    │
      │                    ▼
      │              Mistral Sentiment Agent
      │                    │
      └────────► Combine Price + Sentiment
                           │
                           ▼
                  Write to Supabase
                           │
                           ▼
                  Telegram Alert (+ error handling)
                           │
                           ▼
                  Power BI Dashboard
```

*(Replace this diagram with an annotated screenshot of the actual n8n canvas — see `screenshots/` below.)*

## Tech stack

- **Orchestration:** n8n (JavaScript Code nodes for data shaping/filtering)
- **LLM:** Mistral Cloud, via n8n's LangChain agent node
- **Data sources:** Twelve Data (price candlesticks), NewsAPI, Alpha Vantage (news sentiment)
- **Storage:** Supabase (Postgres)
- **Alerts:** Telegram Bot API
- **Visualisation:** Power BI (connected directly to the Supabase table)

> Note: this export shows the pipeline logic as n8n Code nodes (JavaScript), not standalone Python scripts. If you have a separate Python component (e.g. Power BI data prep or a modelling notebook), add it to this repo under a `python/` folder and update this section.

## Repo structure

```
n8n/
  financial-sentiment-workflow.json   # sanitised n8n workflow export
screenshots/
  architecture.png                    # annotated diagram/screenshot of the n8n canvas
  dashboard.png                       # Power BI dashboard screenshot
.env.example
README.md
```

## Setup

1. **Clone the repo** and open your n8n instance.
2. **Import the workflow:** n8n → Workflows → three-dot menu → *Import from File* → select `n8n/financial-sentiment-workflow.json`.
3. **Set the required environment variables** (see `.env.example`):
   - `TWELVEDATA_API_KEY`
   - `NEWSAPI_API_KEY`
   - `ALPHAVANTAGE_API_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`
4. **Configure n8n credentials** for the nodes that use n8n's built-in credential store: Telegram API, Supabase API, and Mistral Cloud API.
5. **Set your watchlist** — update the ticker list the workflow loops over.
6. **Activate the workflow** so the Schedule Trigger starts running.
7. **Connect Power BI** to the Supabase Postgres table to visualise results.

## Security note

API keys and a Supabase service-role key were removed from this export and replaced with environment-variable references before publishing. If you fork this project, generate your own keys — do not reuse any credentials that may have appeared in earlier versions of this repo.
