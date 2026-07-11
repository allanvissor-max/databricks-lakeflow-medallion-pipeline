# Databricks Lakeflow Medallion Pipeline

This project demonstrates an end-to-end data pipeline built with Databricks Lakeflow Spark Declarative Pipelines.

The pipeline ingests raw JSON transaction files, applies data quality validation, standardises valid records and creates reporting-ready customer-level metrics.

## Architecture

```text
JSON source files
        ↓
Bronze raw layer
        ├──→ Data quality issues
        └──→ Silver validated layer
                         ↓
                  Gold reporting layer
```

## Pipeline layers

### Bronze layer

The Bronze streaming table incrementally ingests raw JSON files from a Databricks Volume.

The original transaction fields are retained, together with:

* ingestion timestamp
* source file path

### Data quality layer

Invalid records are stored separately for investigation instead of being permanently discarded.

The pipeline detects issues such as:

* missing transaction ID
* missing customer ID
* missing or non-positive amount
* invalid transaction date
* missing currency
* missing transaction status

### Silver layer

The Silver streaming table contains only validated records.

It also applies:

* data type conversion
* whitespace removal
* currency standardisation
* status standardisation

### Gold layer

The Gold materialized view creates reporting-ready metrics by customer:

* transaction count
* total transaction amount
* average transaction amount
* first transaction date
* latest transaction date

Only completed transactions are included in the Gold output.

## Technologies

* Databricks
* Lakeflow Spark Declarative Pipelines
* Delta Lake
* SQL
* Streaming tables
* Materialized views
* Databricks Volumes
* Medallion Architecture
* Data quality validation
* Git and GitHub

## Project structure

```text
databricks-lakeflow-medallion-pipeline/
├── pipeline/
│   └── transactions_pipeline.sql
├── sample_data/
│   └── transactions_001.json
├── screenshots/
├── README.md
└── .gitignore
```

## How to run

1. Go to Databricks Volume Folder for the source files.
```
Databricks -> Catalog -> workspace
└── default
    └── Volumes
        └── kafka_demo_files
            └── transactions
                └── transactions_001.json
```

3. Upload the JSON files to a folder such as:

```text
/Volumes/workspace/default/kafka_demo_files/transactions/
```

3. Create a target schema:

```sql
CREATE SCHEMA IF NOT EXISTS workspace.transactions_demo;
```

4. Create a Databricks Lakeflow ETL pipeline.

5. Configure the pipeline target:

```text
Catalog: workspace
Schema: transactions_demo
```

6. Add the SQL from:

```text
pipeline/transactions_pipeline.sql
```

7. Run the pipeline.

## Output tables

The pipeline creates the following datasets:

```text
workspace.transactions_demo.bronze_transactions
workspace.transactions_demo.dq_transaction_issues
workspace.transactions_demo.silver_transactions
workspace.transactions_demo.gold_customer_summary
```

## Expected sample results

| Dataset               |   Expected result |
| --------------------- | ----------------: |
| Bronze transactions   |         5 records |
| Data quality issues   | 2 invalid records |
| Silver transactions   |   3 valid records |
| Gold customer summary |       2 customers |

## Data quality examples

The sample data includes:

* one transaction with a negative amount
* one transaction with a missing transaction ID

These records are stored in the data quality issues table and excluded from the Silver layer.

## Key learning outcomes

This project demonstrates practical knowledge of:

* Databricks pipeline development
* Bronze–Silver–Gold architecture
* incremental file ingestion
* SQL data transformations
* data quality rule implementation
* invalid-record quarantine patterns
* reporting-ready data modelling
* pipeline dependency management
* GitHub version control
