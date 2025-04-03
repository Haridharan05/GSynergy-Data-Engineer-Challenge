# Architecture Overview

This document outlines the design and architecture of the ETL pipeline developed for the GSynergy Data Engineer Interview Challenge. The solution leverages Databricks, PySpark, and Delta Lake to process raw data files, validate and clean data, and ultimately produce a refined aggregated table for business reporting. The entire process is designed to be scalable, maintainable, and production-ready.

---

## 1. Overview

The ETL pipeline is organized into three main layers:

- **Raw Layer:** Raw data files (in `.dlm` format) are stored in cloud storage (DBFS, Azure Blob, or S3). This layer is the landing zone for all incoming data.
  
- **Staging Layer (Bronze/Silver):** In this layer, raw data is ingested into Databricks and cleaned/validated. Data quality checks, schema enforcement, and transformation steps are applied here. Clean data is written into Delta tables within the `staging_db`.
  
- **Refined Layer (Gold):** Aggregated and business-ready data is produced in this layer. Key metrics are calculated (e.g., sales totals, discount amounts), and the final output table (`mview_weekly_sales`) is stored in the `refined_db`. Incremental update logic is applied to keep the refined table up-to-date.

---

## 2. Data Ingestion and Validation

### Raw Data Ingestion

- **Source Files:** The pipeline ingests raw, pipe-delimited `.dlm` files (both fact and dimension files) from a specified directory in DBFS.
- **Schema Enforcement:** Each file is read using an explicit schema defined in PySpark. This ensures that data types (e.g., string, double, timestamp) are correctly interpreted.
- **Initial Validation:** Basic validations are performed during ingestion, such as:
  - Dropping rows with missing primary key values.
  - Trimming string fields.
  - Converting date and time strings to proper timestamp/date types.

### Data Quality Checks

- **Numeric Validations:** Filters are applied to ensure that numeric columns (e.g., sales units, sales dollars, discount dollars) are non-negative.
- **Date Boundary Checks:** Only data with transaction dates beyond a specific threshold (e.g., `2015-01-01`) is processed.
- **Referential Integrity:** Checks are performed to ensure that foreign keys in fact tables exist in the corresponding dimension tables (e.g., `fscldt_id` in fact tables must be present in the calendar dimension `hier_clnd`).

---

## 3. Staging Layer

### Purpose

- **Centralized Clean Data:** The staging layer serves as a centralized repository for cleaned and validated data. This data is stored as Delta tables in the `staging_db`.
- **Separation of Concerns:** By isolating raw ingestion from business transformation, the pipeline becomes modular and easier to maintain.
  
### Components

- **Dimension Tables:** Each hierarchy (dimension) file is stored as a separate Delta table (e.g., `dim_possite`, `dim_prod`, `dim_clnd`).
- **Fact Tables:** Raw fact data is also stored in Delta format (e.g., `fact_transactions`, `fact_averagecosts`).

---

## 4. Refined Layer

### Purpose

- **Business-Ready Data:** The refined layer consolidates and aggregates data from the staging layer into business-friendly, query-optimized tables.
- **Aggregation:** For this challenge, an aggregated table called `mview_weekly_sales` is created. This table aggregates metrics such as `sales_units`, `sales_dollars`, and `discount_dollars` by key dimensions (e.g., POS site, SKU, fiscal date, price state, transaction type).

### Process

- **Transformation:** SQL or PySpark is used to aggregate data from the fact table in the staging layer.
- **Delta Table:** The aggregated data is written into a Delta table in the `refined_db`, ensuring ACID transactions, schema enforcement, and efficient incremental updates.
- **Incremental Updates:** Rather than reprocessing the entire dataset with each load, Delta Lake’s merge (upsert) functionality is used to incrementally update `mview_weekly_sales` as new fact data arrives.

---

## 5. Incremental Load Logic

### Approach

- **New Data Processing:** When new incremental files (or new data partitions) arrive, they are processed using the same validation and cleaning logic as the initial load.
- **Aggregation:** The new data is aggregated at the same granularity as the refined table.
- **Delta Merge:** The aggregated incremental data is merged into the existing `mview_weekly_sales` Delta table:
  - **When Matched:** Existing rows are updated by adding the new values to the existing totals.
  - **When Not Matched:** New rows are inserted for new key combinations.
  
### Benefits

- **Efficiency:** Only new or changed data is processed, reducing overall computation.
- **Consistency:** Delta Lake ensures ACID compliance, making the incremental merge robust.
- **Scalability:** This design scales as data volumes grow.

---

## 6. Technology and Best Practices

- **Databricks and Delta Lake:** The solution leverages the performance and scalability of Databricks and Delta Lake, enabling efficient handling of big data with ACID transactions.
- **Explicit Schemas:** Defining explicit schemas helps avoid runtime errors and ensures data consistency.
- **Modular Notebooks:** The solution is implemented in a single notebook for this challenge, but the architecture is modular—allowing future separation of ingestion, transformation, and incremental updates into separate notebooks or jobs.
- **Data Validation:** Rigorous validation and quality checks are performed at each stage to ensure the integrity of the final output.
- **Incremental Processing:** Using Delta Lake’s merge functionality provides a robust solution for handling incremental data loads without reprocessing the entire dataset.

---

## 7. Diagrams

Refer to the following diagrams for a visual representation of the architecture:

- **ER Diagram:** Located in `diagrams/ER_Diagram.png`. This diagram details the schema for each fact and dimension table, including columns, primary keys, and foreign key relationships.
- **Dataflow Diagram:** Located in `diagrams/Dataflow_Diagram.png`. This diagram shows the overall flow of data from raw ingestion, through staging, to the refined layer, and finally to consumption by BI tools.

---

## Conclusion

This architecture is designed to be robust, scalable, and efficient, fulfilling the requirements of the GSynergy challenge. The combination of raw data ingestion, thorough validation, a clean staging layer, and a refined, aggregated output with incremental updates ensures that the data pipeline is production-ready and capable of supporting business analytics.

For any further details or questions regarding the design and implementation, please refer to the accompanying documentation and diagrams in this repository.
```