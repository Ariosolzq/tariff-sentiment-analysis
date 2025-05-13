# 📘 Tariff Sentiment Intelligence via Reddit & NLP

## Project Overview

This project implements an end-to-end data pipeline and analytics framework to monitor, quantify, and visualize public sentiment around the United States’ 2025 tariff policies under President Trump. By combining large-scale social media ingestion, distributed messaging, NLP sentiment scoring, and advanced topic/time-series modeling, the system delivers real-time insights and historical analyses for business risk sensing and strategic decision-making.

---

## Motivation

In 2025, sweeping tariff increases on steel, aluminum, Chinese imports, and nearly all foreign goods triggered unprecedented market volatility, supply-chain disruptions, and consumer price pressures. Companies and policy analysts needed timely, data-driven indicators of public reaction—well ahead of traditional surveys or sales data—to adjust strategy, communications, and risk management. Reddit, with its high-volume, opinion-rich discussions, serves as a live laboratory for sentiment analysis and topic discovery.

---

## Pipeline & Architecture Overview

Below is a concise description of each layer in the distributed, fault-tolerant pipeline. A high-level diagram is provided in the repo’s documentation.

### 1. Data Ingestion  
A scheduled Python script (using PRAW) runs every 15 minutes to fetch new Reddit posts and full comment threads matching tariff-related keywords. Batches are written as timestamped JSON files. A checkpoint mechanism records the last processed post ID to avoid duplicates and minimize API calls.

### 2. Message Queue  
After each batch file is saved locally, a lightweight Python **Kafka Producer** reads the file, serializes each post/comment as JSON, and publishes it to the `reddit-tariff-topic` on Confluent Cloud. The topic has six partitions and a one-week retention policy, ensuring scalable, durable buffering and enabling multiple downstream consumers to replay or parallelize processing.

### 3. Data Processing (ETL)  
A Python **Kafka Consumer** ingests messages, deserializes JSON, and loads them into a Pandas DataFrame. The ETL routines perform:
- Field extraction (post/comment IDs, text, score, timestamps, depth)
- Cleaning (remove deleted/bot content, filter by language/length)
- Enrichment (convert UTC timestamps to ISO format, tag known policy events)

Optional Spark Structured Streaming code is available for cluster-scale transformation.

### 4. Sentiment Analysis  
The cleaned DataFrame is fed into a hybrid NLP layer:
- **VADER** computes a continuous compound score (–1 to +1) and basic positive/negative/neutral labels.
- **RoBERTa** (transformer model) assigns nuanced categorical labels (e.g. Anger, Joy, Fear) to capture context and sarcasm.
Results for each message—numeric score and categorical label—are appended back to the DataFrame and simultaneously published to a new Kafka topic (`tariff-sentiment-out`) for real-time dashboards.

### 5. Storage & Distribution  
Enriched data is written hourly to **Google Cloud Storage** in Parquet format, partitioned by date and data type (raw vs. processed). This GCS data lake supports historical replay, large-scale batch analysis, and integration with BI tools. Concurrently, Kafka retains recent seven days of messages for quick reprocessing or real-time subscription.

### 6. Analysis & Insights  
On the historical data in GCS:
- **Topic Modeling (LDA)** reveals five dominant themes:  
  1. Voting & Political Judgment  
  2. Institutional & National Identity  
  3. Executive Power & Trade Conflict  
  4. Abstract Ideals & American Values  
  5. Tariffs & Consumer Impact  
- **Time-Series Sentiment** tracks daily average VADER scores and RoBERTa label distributions, overlaid with key tariff announcement dates to measure public reaction spikes and recovery patterns.
- **Correlation & Drill-Down** merges sentiment indices with market data (e.g., retail stock returns) and segments by subreddit to identify early warning communities (investor vs. consumer forums) and topic-specific sentiment nuances.

---

## Data Description

**Sample Size**  
- **Posts:** ~245 unique submissions  
- **Comments:** 12,495 individual comments  

**Variables**  
- `post_id`, `title`, `selftext`, `score`, `created_utc`  
- `comment_id`, `body`, `parent_id`, `depth`, `subreddit`  

**Example JSON Record**  
```json
{
  "post_id": "1ifnoig",
  "title": "Trudeau announcing retaliatory tariffs on the United States",
  "selftext": "",
  "created_utc": 1738465409.0,
  "score": 137589,
  "comments": [
    {
      "comment_id": "mahs1qz",
      "parent_id": "t1_mahq5kh",
      "body": "Cause\n\n![gif](giphy|fstB58aVozghCu70FO|downsized)",
      "score": 14550,
      "created_utc": 1738466877.0,
      "depth": 1
    },
    {
      "comment_id": "mahuf30",
      "parent_id": "t3_1ifnoig",
      "body": "Trudeau's tariffs are only on a handful of specific products[...]",
      "score": 14511,
      "created_utc": 1738467690.0,
      "depth": 0
    }
  ]
}
