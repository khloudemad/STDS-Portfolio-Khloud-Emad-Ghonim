# 🏗️ GreenStream Energy: Serverless ETL Pipeline Design

This project documents a **conceptual serverless ETL pipeline** for processing energy consumption data from 50,000 smart meters. The goal is to transform raw, inconsistent data into clean, analytics-ready datasets while ensuring high data quality for peak detection, anomaly identification, and future forecasting.

## 📍 Task A: System Design – ETL Architecture Diagram

![ETL Pipeline Architecture](Diagrams/ETL%20Flow.png)

> **Key Components**:
> - **Source**: Raw CSV uploads from smart meters
> - **Raw Storage**: Landing zone (immutable for audit)
> - **Event-Driven Trigger**: Automatic processing on new file arrival
> - **Transformation & Validation**: Cleaning + business rules application
> - **Destinations**:
>   - Structured relational database for operational queries
>   - Parquet archival for efficient long-term analytics
> - **Error Handling**: Automatic retries + Dead Letter Queue + alerts

*(Interactive diagram created in Obsidian Canvas – static export for GitHub)*

## 📜 Task B: Transformation Logic & Business Rules

| Rule | Category                  | Condition                                                                 | Action / Flag                              | Rationale                                      |
|------|---------------------------|---------------------------------------------------------------------------|--------------------------------------------|------------------------------------------------|
| 1    | Unit Standardization      | energy_unit == "W"                                                        | Divide by 1000 → "kW", flag `unit_converted` | Uniform scale for aggregation                  |
| 2    | Missing Values            | energy_value IS NULL                                                      | Flag `missing_value`, forward-fill <1h gaps | Preserve continuity without bias               |
| 3    | Data Validation           | value < 0 or > 10 kW or \|z-score\| > 3 (24h window)                       | Flag outlier/invalid → error queue         | Prevent skew in analytics/models               |
| 4    | Faulty Meter Detection    | Near-zero (<0.01 kW) >24h or daily total = 0 >3 days                      | Flag `potentially_faulty` + alert          | Early hardware/connectivity detection          |
| 5    | Timestamp & Deduplication | Non-UTC or duplicates                                                     | Normalize to UTC, keep latest, flag duplicate | Accurate time-based analysis                   |

All flags stored as metadata for audit and filtering.

## ⚙️ Task C: Single Record Lifecycle

1. **Ingestion** → Raw record lands in immutable storage bucket  
2. **Trigger** → Upload event starts serverless processing  
3. **Transformation & Validation** → Rules applied sequentially; failures → retries → DLQ + alert  
4. **Structured Load** → Clean data inserted into relational DB for querying  
5. **Analytical Archival** → Converted to partitioned Parquet for efficient historical queries  
6. **Outcome** → Success: available in both layers | Failure: isolated + alert

## 🚀 Next Steps & Enhancements

- Build real-time dashboards (e.g., Grafana) for peak monitoring
- Integrate ML-based anomaly detection on flagged records
- Automate fault alerts to ticketing system (e.g., Jira)
- Add data lineage tracking and quality metrics dashboard

## 📂 Repository Structure
GreenStream Energy ETL Design/

├── README.md                          ← Project overview (this file)

├── GreenStream ETL Pipeline Design.md ← Master Obsidian note (main document)

├── Business Rules.md                  ← Detailed transformation rules

├── Single Record Lifecycle.md         ← Record processing journey

├── Diagrams/

│   ├── ETL Pipeline Flow.canvas       ← Interactive Canvas diagram

│   └── ETL Pipeline Flow.png          ← Exported static image (shown above)

└── Attachments/                       ← Optional supporting files


**Tags**: `#ETL` `#DataPipeline` `#DataScience` `#DesignThinking` `#Serverless` `#EnergyAnalytics` `#SmartMeters`

---

*Designed with Obsidian • Conceptual Design for Data Scientists*
