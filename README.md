# AWS_datapipeline_project

# Automated Serverless JSON Flattening ETL Pipeline

A robust, serverless event-driven ETL (Extract, Transform, Load) pipeline built on AWS to automatically ingest, flatten, and transform complex nested JSON order data into highly optimized Apache Parquet format for analytical data lakes.

## 📌 Architecture Overview

The pipeline follows a completely decoupled, event-driven serverless architecture across three core execution phases:

```
[ User Uploads JSON ] ──> [ Amazon S3: Incoming Folder ]
                                    │
                         (S3 Event Notification)
                                    ▼
                         [ AWS Lambda (Python 3.13) ] ──> (Flattens Nested JSON)
                                    │
                           (S3 Write Operation)
                                    ▼
                        [ Amazon S3: Parquet Datalake ]
                                    │
                         (AWS Glue Crawler Scan)
                                    ▼
                        [ AWS Glue Data Catalog ] ──> Ready for Athena/BI Querying
```

### 1. Ingestion & Trigger Phase
* **Source Storage:** Users drop nested transaction or order data (`orders_etl.json`) into the designated input directory: `s3://etl-project-viit-22310646/orders_json_incoming/`.
* **Event Notification:** S3 detects the write event and instantly publishes a trigger notification to invoke the downstream transformation handler.

### 2. Processing & Transformation Phase
* **Compute Layer:** An asynchronous **AWS Lambda** function running on the **Python 3.13** runtime is activated.
* **Data Transformation:** The function reads the source nested JSON objects, processes complex relational arrays or hierarchies natively, and flattens them into granular tabular structures using standard Python libraries (e.g., `pandas`, `pyarrow`, or `awswrangler`).
* **Storage Optimization:** The transformed row-column matrix is serialized into highly compressed column-oriented **Apache Parquet** format.

### 3. Cataloging & Analytics Phase
* **Target Storage:** Output Parquet datasets are automatically versioned and named by timestamp (e.g., `orders_ETL_20251022_073323.parquet`) and landed inside `s3://etl-project-viit-22310646/orders_parquet_datalake/`.
* **Schema Discovery:** An **AWS Glue Crawler** (`etl_pipeline_crawler`) automatically scans the target data lake partition on a recurring trigger or run-on-demand basis, dynamically deducing the schema and updating metadata properties inside the **AWS Glue Data Catalog Database** (`etl_pipeline`).

---

## 📊 Transformed Data Schema & Preview

Upon flattening, highly nested array elements (such as multiple items within a single order payload) are exploded into continuous relational records, facilitating rapid SQL analytical execution. 

### Output Attributes

| Attribute Name | Data Type | Description |
| :--- | :--- | :--- |
| `order_id` | Integer | Unique identifier of the transaction payload |
| `order_date` | Date (YYYY-MM-DD) | Timestamp of the processed transaction |
| `total_amount` | Decimal / Float | Aggregated cost metric for the purchase |
| `customer_id` | Integer | Unique identifier for the customer entity |
| `customer_name`| String | First and last name of the buyer |
| `email` | String | Contact email address used for registration |
| `address` | String | Physical shipping destination |
| `product_id` | String | SKU or alphanumeric identifier for the item |
| `product_name` | String | Categorical product name descriptive label |
| `category` | String | High-level market vertical taxonomy |

---

## 🔒 Security & Identity Access Management (IAM)

Adhering strictly to the principle of least privilege access, operational security relies on a custom managed execution role assigned to the pipeline components:

* **Role Name:** `AWSGlueServiceRole-etl-pipeline-iam-role`
* **Attached Policies:**
  1. `AWSGlueServiceRole` (AWS Managed): Grants capabilities to instantiate metadata discoveries, modify Data Catalog definitions, and store log streams.
  2. `Customer-Managed S3 Access Policy`: Configures granular `GetObject` actions on incoming buckets, paired with explicit `PutObject` permissions on the targeted outbound datalake path.

---

## 🛠️ Infrastructure Monitoring & Performance Metrics

Full observability and operational visibility are engineered through native **Amazon CloudWatch** integrations:
* **Log Ingestion:** Lambda invocations report lifecycle events directly to standard log groups (`/aws/lambda/...`).
* **Performance Benchmark Profile:**
  * **Runtime Engine:** Python 3.13
  * **Memory Configuration:** Allocated 128 MB / Max Used 128 MB
  * **Transformation Efficiency:** Average execution duration ranges between ~6.8 seconds (`6841.84 ms`) to 9.4 seconds (`9427 ms`) billing segments, ensuring cost-efficient data operations.
