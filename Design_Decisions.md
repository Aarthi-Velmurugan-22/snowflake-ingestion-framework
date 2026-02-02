# Design Decisions – Snowflake Incremental Data Pipeline

This document explains the key technical and architectural decisions made while building this Snowflake data pipeline.

1. Why S3 → RAW → CURATED layering?

We followed a layered architecture to separate concerns:

* **RAW layer** stores ingested data exactly as received from S3, including CDC metadata (`OP`, `OP_TS`).
* **CURATED layer** represents the latest business-ready state, optimized for analytics and consumption.

This design improves:

* Debuggability
* Reprocessing capability
* Long-term maintainability


2. Why use Snowflake Tasks for ingestion?

Snowflake Tasks were chosen to:

* Automate ingestion from S3
* Avoid external schedulers
* Keep orchestration fully inside Snowflake

Each table has its own task to allow:

* Independent retries
* Better failure isolation
* Easier scalability


3.  Why Include OP and OP_TS in RAW Only?

Operational metadata is required to:
- Identify inserts, updates, and deletes
- Apply correct CDC logic during merge

Once data is merged into CURATED, these columns are no longer required and are intentionally excluded to keep tables clean.


4. Why MERGE into CURATED tables?

MERGE statements allow handling:

* Inserts (`OP = 'I'`)
* Updates (`OP = 'U'`)
* Deletes (`OP = 'D'`)

in a single, atomic operation.

This ensures:

* Idempotent processing
* Correct final state in CURATED tables
* Clean handling of deletes from source systems


5. Why exclude OP and OP_TS from CURATED tables?

`OP` and `OP_TS` are technical CDC columns required only for change processing.

CURATED tables intentionally exclude them to:

* Match source (Postgres-like) schemas
* Keep analytics-friendly table design
* Separate technical metadata from business data

Auditability is preserved in RAW tables.


6. Why one pipeline per table?

Customer, Product, and Order pipelines are implemented independently to:

* Avoid cross-table dependencies
* Enable parallel ingestion
* Simplify troubleshooting

This approach mirrors real-world enterprise Snowflake implementations.


7. Why Task Chaining Instead of Scheduling Everything?

Chained tasks:
- Run only when upstream tasks complete successfully
- Reduce unnecessary warehouse usage
- Enable event-driven pipelines

This design is more efficient and production-friendly.


8. Why Place Tasks in the RAW Schema?

Snowflake enforces task dependency rules at the schema level.  
Placing ingestion and merge tasks in the same schema allows proper task chaining while still writing data to CURATED tables.


9. Future Improvements

* Data quality checks
* Centralized error logging
* Email / Slack alerting
* Metrics and monitoring dashboards


Summary

This project prioritizes:

* Simplicity
* Correctness
* Production-aligned Snowflake patterns

The architecture is intentionally extensible for real-world enhancements.
