Snowflake Incremental Data Pipeline (S3 → RAW → CURATED)

📌 Project Overview

This project demonstrates an end-to-end **incremental data pipeline** built using **Snowflake**, following real-world data engineering best practices.

The pipeline ingests CSV files from **Amazon S3**, loads them into **RAW staging tables** using **Snowflake Tasks**, captures changes using **Streams**, and applies **CDC logic (Insert / Update / Delete)** into **CURATED tables** using **MERGE statements**.

This project is designed to closely resemble a **production-grade analytics pipeline**.


🏗️ Architecture

The pipeline follows a layered Snowflake architecture:

- Source files land in Amazon S3
- Snowflake Tasks load data into RAW tables
- Streams capture incremental changes
- MERGE logic applies inserts, updates, and deletes
- CURATED tables represent the latest state

🔧 Technologies Used

* Snowflake (Tasks, Streams, MERGE)
* Amazon S3 (external data source)
* SQL
* CDC (Change Data Capture) design

📂 Data Flow

1️⃣ S3 → RAW Layer

* Source CSV files are placed in S3 folders:

  * customer/
  * product/
  * order/

* Snowflake "external stage" points to the S3 bucket
* **Scheduled Snowflake Tasks (every 2 minutes)** load data into RAW tables using `COPY INTO`

RAW tables include metadata columns:

* `OP` → operation type (`I`, `U`, `D`)
* `OP_TS` → operation timestamp


2️⃣ RAW → CURATED Layer

* Streams track incremental changes in RAW tables
* MERGE logic applies:

  * Inserts for new records
  * Updates for changed records
  * Deletes when `OP = 'D'`
* CURATED tables exclude operational metadata (`OP`, `OP_TS`)
* CURATED tables represent the latest business state

---

🗃️ Schemas & Tables

RAW Schema

* CUSTOMER_STG
* PRODUCT_STG
* ORDER_STG

Includes:

* OP
* OP_TS
* Source columns

CURATED Schema

* CUSTOMER
* PRODUCT
* ORDER

Includes:

* Business columns only
* `CREATED_AT`
* `UPDATED_AT`


🔄 CDC Logic (MERGE)

CDC is handled using Snowflake `MERGE` statements driven by Streams:

* INSERT → when record does not exist
* UPDATE → when record exists and OP = 'U'
* DELETE → when OP = 'D'

This ensures Snowflake stays in sync with upstream source behavior.


📁 Repository Structure


snowflake-cdc-pipeline/
│
├── README.md
├── decisions.md
│
├── sql/
│   ├── raw/
│   │   ├── create_raw_tables.sql
│   │   ├── create_file_format.sql
│   │   ├── create_stage.sql
│   │   └── create_raw_tasks.sql
│   │
│   ├── curated/
│   │   ├── create_curated_tables.sql
│   │   ├── create_streams.sql
│   │   └── merge_tasks.sql
│
├── sample-data/
│   ├── customer.csv
│   ├── product.csv
│   └── order.csv
│
└── architecture/
    └── architecture.txt



## 🧠 Design Decisions

Key architectural and design choices are documented in **decisions.md**, including:

* Why Tasks were used instead of Snowpipe
* Why Streams + MERGE were chosen
* CDC strategy for deletes
* Separation of RAW and CURATED layers


✅ Key Learnings

* Designing production-style CDC pipelines
* Handling deletes without hard dependencies on source systems
* Using Snowflake Streams effectively
* Task orchestration and scheduling


🚀 Future Enhancements

* Add error handling and alerting
* Introduce Snowpipe for near real-time ingestion
* Add data quality checks
* Visual architecture diagram


👩‍💻 Author

**Aarthi Velmurugan**
Data Engineering Portfolio Project

⭐ If you like this project, feel free to star the repo!
