**Azure Data Engineering Projects - Deep Dive Explanation**

---

## Project 1: Scalable Data Pipeline for High-Volume Log Processing

### Step 1: High-Level Summary

**Objective:** Process millions of server logs (structured CSV & semi-structured JSON) in real-time and batch using Azure services.

**Architecture Overview:**

* **Source Data:** Application logs from multiple servers
* **Ingestion:** Kafka (Apache Kafka for real-time ingestion)
* **Processing:** Spark Structured Streaming (real-time), Azure Data Factory (batch)
* **Storage:** Raw data in Azure Data Lake (ADLS Gen2)
* **Transformations:** Apache Spark jobs (PySpark)
* **Loading:** Transformed data to Azure Synapse Analytics and Power BI
* **Monitoring:** Azure Monitor + Log Analytics

---

### Step 2: Data Details

**Data Sources:**

1. **Structured Data (CSV):** Server logs

   * Columns: `timestamp`, `log_level`, `server_id`, `user_id`, `message`, `response_time`, `status_code`
2. **Semi-Structured Data (JSON):** Application event metadata

   * JSON Fields: `event_type`, `device`, `os`, `location`, `event_properties` (nested field with keys like `latency`, `error_code`, `retry_count`)

**Volume:**

* \~5GB/day (CSV), \~2GB/day (JSON)

---

### Step 3: Key Spark & Azure Tasks

**Data Handling:**

* Used `spark.read.csv()` and `spark.read.json()` to read files from ADLS.
* Auto infer schema for CSV; defined schema manually for nested JSON.
* Flattened JSON using explode() and selectExpr().

**Transformations & Aggregations:**

* Parsed timestamp fields, filtered out NULL/error logs.
* Aggregated by `log_level`, `server_id`, and `status_code`.
* Computed KPIs like:

  * Avg response time per server/hour
  * Error rate (count of status\_code >= 500)
  * Retry pattern analysis using JSON `retry_count`

**Issues Faced & Solutions:**

* **Large file handling:**

  * Used partitioning on `date` and `server_id`.
  * Tuned Spark config: executor memory, shuffle partitions.
* **JSON parsing errors:**

  * Created a custom schema and used `permissive` mode to avoid crashes.
* **Latency in streaming:**

  * Reduced batch interval, optimized sink writing using foreachBatch.

**End-to-End Flow (Story Style):**

> "Logs were being pushed by applications in real-time to Kafka. A PySpark structured streaming job running on Azure Databricks consumed the logs, processed them (flattened JSON, aggregated errors), and wrote the output to ADLS Gen2. Simultaneously, batch logs were picked up by Azure Data Factory every 1 hour and run through another Spark job to compute historical trends, which were stored in Synapse for dashboarding. This pipeline helped reduce alert response time by 40%."

---

## Project 2: Automated ETL for E-Commerce Analytics

### Step 1: High-Level Summary

**Objective:** Automate ETL pipeline to process user transactions and events for downstream ML models and dashboarding.

**Architecture:**

* **Source:** CSV from e-commerce DB export + JSON from user interaction events (web/app)
* **Ingestion:** ADF (scheduled pipeline)
* **Storage:** ADLS Gen2 raw zone
* **Processing:** Azure Databricks with PySpark
* **Loading:** Azure Synapse (for dashboards), Azure SQL DB (for APIs)

---

### Step 2: Data Details

**CSV Columns:** `order_id`, `user_id`, `product_id`, `timestamp`, `price`, `quantity`, `location`, `device_type`

**JSON Fields:** `event_type`, `timestamp`, `session_id`, `user_id`, `action`, `properties` (nested: `scroll_depth`, `clicks`, `time_spent_sec`)

---

### Step 3: ETL Process in Detail

**1. Ingestion:**

* Created ADF pipeline with two triggers: daily CSV pull from SFTP & JSON from API.
* Linked services: Azure Blob + REST API connector.

**2. Processing:**

* Used Databricks notebooks:

  * Cleansed data: removed NULLs, deduplicated.
  * Flattened JSON interactions.
  * Joined CSV + JSON on `user_id` and timestamp proximity (within 1-minute window).

**3. Transformations:**

* Feature engineering:

  * Avg session time
  * Scroll vs click ratio
  * Spend per user per session
* Aggregated by `user_id`, `device_type`, `location`

**4. Load:**

* Wrote cleaned tables to Synapse and Azure SQL DB using JDBC.

**Issues & Solutions:**

* **Late data arrival:** Used watermarking and window functions in Spark.
* **High memory consumption:** Used broadcast joins for smaller lookup tables.

**End-to-End Flow (Story Style):**

> "A customer visits the site, interacts with the product pages, and places an order. Their clickstream JSON gets captured in real-time and is pulled by ADF. At the same time, daily orders are exported via CSV. Our PySpark job joins both to get complete session analytics. Features like session duration, action intensity, and device impact are computed and stored in Synapse for business teams to analyze purchase patterns."

---

## Interview Scenario-Based Questions

**1. What types of data did you handle?**

* CSV (structured): Logs, transactions
* JSON (semi-structured): Event metadata, user interaction

**2. How did you handle nested JSON?**

* Defined manual schema, used explode and flattening techniques.

**3. What performance issues did you face?**

* Spark memory pressure: Resolved using partitioning, tuning shuffle settings
* Streaming latency: Used foreachBatch for efficient sinks

**4. How did you join streaming + batch data?**

* By matching timestamp proximity and applying watermark in Spark.

**5. How did you ensure data quality?**

* Validated schema, null checks, deduplication
* Logs & alerts via Azure Monitor + Databricks job metrics

**6. Business KPIs you generated?**

* Response time, error rate (logs)
* Session duration, click ratio, spend/user (e-commerce)

**7. Why did you choose Azure services?**

* Native integration, ease of scaling, low-code options (ADF), strong Spark support (Databricks), enterprise-grade security (Azure Monitor, IAM)

---

Let me know when you're ready to prepare for behavioral or technical deep-dive questions next.
