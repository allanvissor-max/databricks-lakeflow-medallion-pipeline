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

## How to Run

### 1. Create a Databricks Volume

Create a Databricks Volume for storing the source transaction files.

Example location:

```text
/Volumes/workspace/default/kafka_demo_files/
```

Inside the Volume, create a folder for transaction files:

```text
/Volumes/workspace/default/kafka_demo_files/transactions/
```

### 2. Upload the Sample Data

Upload the sample JSON or JSONL transaction file to:

```text
/Volumes/workspace/default/kafka_demo_files/transactions/
```

The pipeline uses Databricks Auto Loader to detect and process files added to this folder.

### 3. Create the Target Schema

Run the following SQL command in Databricks:

```sql
CREATE SCHEMA IF NOT EXISTS workspace.transactions_demo;
```

The pipeline tables will be created in this schema.

### 4. Create a Lakeflow Declarative Pipeline

In the Databricks workspace:

1. Open **Lakeflow Declarative Pipelines**.
2. Select **Create pipeline**.
3. Configure the pipeline target:

```text
Catalog: workspace
Schema: transactions_demo
```

### 5. Add the Pipeline Source Code

Add the following SQL file to the pipeline:

```text
pipeline/transactions_pipeline.sql
```

The pipeline processes transaction data through the following Medallion Architecture layers:

```text
Source JSON files
        ↓
Bronze table
        ↓
Silver table
        ↓
Data quality validation
        ↓
Gold aggregation tables
```

### 6. Run the Pipeline

Start the Lakeflow pipeline.

After a successful run, the following tables should be available in:

```text
workspace.transactions_demo
```

Example tables:

```text
transactions_bronze
transactions_silver
transactions_quarantine
transactions_gold
```

New files added to the source folder can be processed during subsequent pipeline runs.


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
