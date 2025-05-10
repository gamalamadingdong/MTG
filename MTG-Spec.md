# MTG Stock Influence Analysis - Architecture Spec

## Overview

The goal of this application is to analyze the influence of public statements—especially from political figures like Marjorie Taylor Greene—on stock market movements. By collecting news, social media, and official statements, applying NLP to extract key information, and correlating these events with market data, the system aims to identify and visualize potential links between statements and stock price changes.


## Tech Stack Decision

- **Backend:** Python (FastAPI)
- **Data Processing & NLP:** spaCy, HuggingFace Transformers, pandas
- **Data Storage:** PostgreSQL (for structured data), MongoDB (for unstructured/news data)
- **Market Data:** yfinance, Alpha Vantage API
- **Frontend:** Streamlit (for rapid dashboarding)
- **Deployment:** Docker, Azure (preferred cloud)
- **Other:** GitHub Actions (CI/CD), pytest (testing)

---

## High-Level Architecture

```
+-------------------+      +-------------------+      +-------------------+
|                   |      |                   |      |                   |
| News & Statement  +----->+   NLP Processing  +----->+   Event Linking   |
|   Scraper         |      |  (Entity, Sentiment)     | (Map to Stocks)   |
|                   |      |                   |      |                   |
+-------------------+      +-------------------+      +-------------------+
        |                          |                          |
        v                          v                          v
+-------------------+      +-------------------+      +-------------------+
|                   |      |                   |      |                   |
| Stock Market Data +----->+   Data Storage    +<-----+ Insider Trading   |
|   Collector       |      | (Postgres/Mongo)  |      |   Data (optional) |
|                   |      |                   |      |                   |
+-------------------+      +-------------------+      +-------------------+
                                    |
                                    v
                        +-----------------------+
                        |   Analysis & Modeling |
                        | (Correlation, ML)     |
                        +-----------------------+
                                    |
                                    v
                        +-----------------------+
                        |   Dashboard & Alerts  |
                        |    (Streamlit)        |
                        +-----------------------+
```

---

## Component Descriptions & Internal Architectures

### 1. News & Statement Scraper

**Purpose:** Collect statements from news, social media, and official sources.

**Functions:**
- `fetch_news_articles()`
- `fetch_social_media_posts()`
- `parse_statements()`
- `deduplicate_statements()`
- `store_raw_statements()`

**Dataflow:**
```
[External APIs] ---> [fetch_news_articles] ---> [parse_statements] ---> [deduplicate_statements] ---> [store_raw_statements]
```

**Example:**
- Input: News API, Twitter API
- Output: 
  ```json
  {
    "source": "Twitter",
    "author": "Marjorie Taylor Greene",
    "timestamp": "2025-05-10T12:00:00Z",
    "text": "We must support American energy companies!"
  }
  ```

---

### 2. NLP Processing

**Purpose:** Extract entities, sentiment, and topics from statements.

**Functions:**
- `extract_entities(text)`
- `analyze_sentiment(text)`
- `classify_topic(text)`
- `store_enriched_statements()`

**Dataflow:**
```
[store_raw_statements] ---> [extract_entities] ---> [analyze_sentiment] ---> [classify_topic] ---> [store_enriched_statements]
```

**Example:**
- Input: Statement text
- Output:
  ```json
  {
    "entities": ["ExxonMobil", "energy"],
    "sentiment": "positive",
    "topic": "energy policy"
  }
  ```

---

### 3. Event Linking

**Purpose:** Map statements to relevant stocks/sectors.

**Functions:**
- `match_entities_to_tickers(entities)`
- `link_statement_to_market_data(statement_id, ticker)`
- `store_event_links()`

**Dataflow:**
```
[store_enriched_statements] ---> [match_entities_to_tickers] ---> [link_statement_to_market_data] ---> [store_event_links]
```

**Example:**
- Input: Entities: ["ExxonMobil"]
- Output: Ticker: "XOM"

---

### 4. Stock Market Data Collector

**Purpose:** Gather historical and real-time stock data.

**Functions:**
- `fetch_historical_prices(ticker, date_range)`
- `fetch_realtime_prices(ticker)`
- `store_market_data()`

**Dataflow:**
```
[match_entities_to_tickers] ---> [fetch_historical_prices] ---> [store_market_data]
```

**Example:**
- Input: Ticker: "XOM", Date: "2025-05-10"
- Output: 
  ```json
  {
    "ticker": "XOM",
    "date": "2025-05-10",
    "close": 112.45
  }
  ```

---

### 5. Insider Trading Data (Optional)

**Purpose:** Collect public disclosures of trades by politicians.

**Functions:**
- `fetch_insider_trades()`
- `parse_trade_disclosures()`
- `store_insider_trades()`

**Dataflow:**
```
[Public Disclosure APIs] ---> [fetch_insider_trades] ---> [parse_trade_disclosures] ---> [store_insider_trades]
```

---

### 6. Data Storage

**Purpose:** Store all structured and unstructured data.

**Functions:**
- `insert_statement()`
- `insert_enriched_statement()`
- `insert_market_data()`
- `insert_event_link()`
- `insert_insider_trade()`

**Dataflow:**
```
[All previous components] ---> [Data Storage (Postgres/Mongo)]
```

---

### 7. Analysis & Modeling

**Purpose:** Correlate statements with market movements and build predictive models.

**Functions:**
- `correlate_statements_with_prices()`
- `train_predictive_model()`
- `backtest_strategy()`
- `store_analysis_results()`

**Dataflow:**
```
[Data Storage] ---> [correlate_statements_with_prices] ---> [train_predictive_model] ---> [backtest_strategy] ---> [store_analysis_results]
```

**Example:**
- Input: Statement about "XOM", price data
- Output: "Statement correlated with +2% price movement in 3 days"

---

### 8. Dashboard & Alerts

**Purpose:** Visualize insights and send notifications.

**Functions:**
- `display_statements_and_impacts()`
- `show_model_predictions()`
- `send_alerts(new_event)`

**Dataflow:**
```
[store_analysis_results] ---> [display_statements_and_impacts] ---> [send_alerts]
```

**Example:**
- Dashboard shows: "Trump statement on Boeing → +1.5% in 2 days"
- Alert: "New statement detected: Possible impact on XOM"

---

## Next Steps

1. Scaffold project structure and initialize repositories.
2. Prototype data collection and NLP pipeline.
3. Set up database schemas.
4. Build initial dashboard for data exploration.
