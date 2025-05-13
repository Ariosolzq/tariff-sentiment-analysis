# 📘 Tariff Sentiment Intelligence via Reddit & NLP

This repository contains the end-to-end data pipeline and analysis framework used to quantify public sentiment around U.S. trade tariffs introduced under President Trump in 2025. The system ingests Reddit discussions, processes and scores sentiment, and generates insights using NLP and time-series modeling.

---

## 🔍 Overview

This project leverages Reddit API, Confluent Kafka, Google Cloud Storage, and NLP techniques (VADER + RoBERTa) to monitor, classify, and analyze real-time social sentiment toward tariff policy. It is built for business intelligence teams and policy researchers interested in public reactions to economic events.

---

## 🧱 Architecture Summary

```mermaid
graph TD
  A[Reddit API (PRAW)] --> B[Local Batch JSON Files]
  B --> C[Kafka Producer]
  C --> D[Kafka Topic (raw-reddit-posts)]
  D --> E[Python Consumer / Pandas Processing]
  E --> F[Sentiment Layer (VADER & RoBERTa)]
  F --> G[Kafka Topic (sentiment-output)]
  F --> H[GCS (structured partitioned files)]
  H --> I[LDA + Time Series Analysis]
```

---

## 📂 Folder Structure

```
tariff-sentiment-pipeline/
│
├── ingestion/
│   └── reddit_harvester.py        # Pulls posts/comments via PRAW
│
├── kafka/
│   ├── producer.py                # Streams JSON to Kafka topic
│   └── consumer_processing.py     # Processes messages, applies ETL
│
├── sentiment/
│   └── sentiment_pipeline.py      # VADER + RoBERTa classification
│
├── storage/
│   └── gcs_writer.py              # Writes enriched data to GCS
│
├── analysis/
│   └── lda_time_series.ipynb      # Topic modeling + sentiment trend
│
├── notebooks/
│   └── exploratory/               # Sample data inspections
│
└── config/
    └── schema_registry.json       # Kafka JSON schema definition
```

---

## 🧪 Sample Output (Schema)

```json
{
  "post_id": "abc123",
  "title": "Trump raises tariffs again",
  "body": "This is going to hurt consumers...",
  "created_utc": 1738465409,
  "subreddit": "politics",
  "score": 524,
  "vader_score": -0.72,
  "roberta_label": "Negative",
  "policy_event_tag": true
}
```

---

## 📊 Key Technologies

- **Reddit API (PRAW)** – Data source
- **Apache Kafka (Confluent Cloud)** – Message queue
- **Pandas / PySpark (ETL)** – Data processing
- **VADER + RoBERTa** – Sentiment scoring
- **Google Cloud Storage (GCS)** – Scalable storage
- **LDA, Matplotlib, Seaborn** – Topic modeling & visualization

---

## 📈 Results Summary

- 245 Reddit threads (~12,500 comments) processed  
- 5 major topics discovered via LDA  
- Sentiment tracked over time against 4 key policy dates  
- Negative sentiment spike detected ~24 hours before market downturns

---

## 📌 How to Run

1. Set your Reddit API credentials in `config/reddit_credentials.json`
2. Run the batch harvester:
   ```bash
   python ingestion/reddit_harvester.py
   ```
3. Start the Kafka producer to stream to topic:
   ```bash
   python kafka/producer.py
   ```
4. Launch sentiment processor and writer:
   ```bash
   python sentiment/sentiment_pipeline.py
   ```
5. Perform analysis in `analysis/lda_time_series.ipynb`

---

## 🔒 License

This project is released under the MIT License. Attribution welcome.

---

## 🤝 Acknowledgements

Built as part of BUDT 737: Enterprise Big Data project, University of Maryland, Spring 2025.  
Special thanks to Prof. Tej Anand for guidance.

---
