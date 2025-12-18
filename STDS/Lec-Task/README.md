# 🏗️ GreenStream Energy: Serverless ETL Pipeline Design

This project documents a **conceptual serverless ETL pipeline** for processing energy consumption data from 50,000 smart meters. The goal is to transform raw, inconsistent data into clean, analytics-ready datasets while ensuring high data quality for peak detection, anomaly identification, and future forecasting.

## 📍 Task A: System Design – ETL Architecture Diagram

The following diagram illustrates the complete end-to-end data flow:

![ETL Pipeline Architecture](Diagrams/ETL%20Pipeline%20Flow.png)

> **Key Components**:
> - **Source**: Raw CSV uploads from smart meters
> - **Raw Storage**: Landing zone (immutable for audit)
> - **Event-Driven Trigger**: Automatic processing on new file arrival
> - **Transformation & Validation**: Cleaning + business rules application
> - **Destinations**:
>   - Structured relational database (e.g., RDS) for operational queries
>   - Parquet archival for efficient long-term analytics
> - **Error Handling**: Automatic retries + Dead Letter Queue + alerts

*(Diagram created in Obsidian Canvas – exported as PNG for GitHub display)*

## 📜 Task B: Transformation Logic & Business Rules

Rigorous, data-science-driven rules applied sequentially during the Transform phase to ensure consistency and quality.

| Rule | Category                  | Condition                                                                 | Action / Flag                              | Rationale                                      |
|------|---------------------------|---------------------------------------------------------------------------|--------------------------------------------|------------------------------------------------|
| 1    | Unit Standardization      | energy_unit == "W"                                                        | Divide by 1000 → "kW", add `unit_converted: true` | Uniform scale for aggregation                  |
| 2    | Missing Values            | energy_value IS NULL                                                      | Flag `missing_value: true`, forward-fill <1h gaps | Avoid bias while preserving continuity         |
| 3    | Data Validation           | value < 0 or > 10 kW (household) or \|z-score\| > 3 (24h window)           | Flag outlier/invalid → route to error queue | Prevent skew in models                         |
| 4    | Faulty Meter Detection    | Near-zero (<0.01 kW) for >24h or daily total = 0 for >3 days              | Flag `potentially_faulty` + alert          | Early detection of hardware/connectivity issues |
| 5    | Timestamp & Deduplication | Duplicates or non-UTC                                                      | Normalize to UTC, keep latest, flag duplicate | Accurate time-based analytics                  |

All flags are stored as metadata columns for auditability and filtering.

## ⚙️ Task C: Single Record Lifecycle

Step-by-step journey of one smart meter reading:

1. **Ingestion**  
   Raw record lands in raw storage bucket (preserved for reproducibility).

2. **Trigger**  
   Upload event automatically starts serverless processing.

3. **Transformation & Validation**  
   Sequential rule application (Rules 1–5). Failures → up to 3 retries → DLQ + alert.

4. **Structured Load**  
   Clean record inserted into relational database for fast querying and validation.

5. **Analytical Archival**  
   Records batched and converted to partitioned Parquet for efficient historical analysis.

6. **Outcome**  
   - Success: Fully available in both operational and analytical layers  
   - Failure: Isolated in DLQ, original raw preserved, alert sent

## 🚀 Next Steps & Enhancements

- Build real-time dashboards (e.g., Grafana) for peak monitoring
- Integrate ML-based anomaly detection on top of flagged data
- Automate fault alerts to ticketing system (e.g., Jira)
- Add data lineage tracking and quality metrics dashboard

## 📂 Repository Structure
GreenStream Energy ETL Design/
├── README.md                      ← This file
├── GreenStream ETL Pipeline Design.md   ← Master Obsidian note
├── Business Rules.md              ← Detailed transformation logic
├── Single Record Lifecycle.md     ← Record journey explanation
├── Diagrams/
│   └── ETL Pipeline Flow.canvas   ← Interactive architecture diagram
│   └── ETL Pipeline Flow.png      ← Exported static image
└── Attachments/                   ← Optional supporting files
text---

**Tags**: `#ETL` `#DataPipeline` `#DataScience` `#DesignThinking` `#Serverless` `#EnergyAnalytics` `#SmartMeters`

---
*Designed with Obsidian • Conceptual Design for Data Scientists*
