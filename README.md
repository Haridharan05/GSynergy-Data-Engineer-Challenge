# GSynergy Data Engineer Interview Challenge

This repository contains my solution for the GSynergy Databricks Data Engineer Interview Challenge. The solution demonstrates a complete ETL pipeline built on Databricks using PySpark and Delta Lake. It covers data ingestion, cleaning, validation, staging, transformation, aggregation, and incremental loading—all consolidated in a single notebook file.

## Table of Contents

- [Project Overview](#project-overview)
- [Repository Structure](#repository-structure)
- [Setup Instructions](#setup-instructions)
- [Execution Instructions](#execution-instructions)
- [Diagrams](#diagrams)
- [Table-wise Definitions (Data Dictionary)](#table-wise-definitions-data-dictionary)
- [Architecture Overview](#architecture-overview)
- [Video Demonstration](#video-demonstration)
- [Contact](#contact)

## Project Overview

This solution processes raw pipe-delimited `.dlm` files stored in DBFS (or a similar cloud storage) and transforms them into clean, standardized Delta tables in a staging database. Aggregated business metrics are then calculated and stored in a refined database as the final output table (`mview_weekly_sales`). Incremental updates are performed using Delta Lake's merge capabilities to ensure that the final table remains current. The entire solution is implemented in one notebook: **GS_FACT_&_DIM_ETL.ipynb**.

## Repository Structure

```
GSynergy-Data-Engineer-Challenge/
│
├── README.md
│
├── notebooks/
│   └── GS_FACT_&_DIM_ETL.ipynb
│
├── diagrams/
│   ├── ER_Diagram.puml         
│   ├── ER_Diagram.png          
│   ├── GS_FACT_&_DIM_ETL.drawio 
│
└── docs/
    ├── Data_Dictionary.md      
    └── Architecture_Overview.md 
```

## Setup Instructions

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/Haridharan05/GSynergy-Data-Engineer-Challenge.git
   cd GSynergy-Data-Engineer-Challenge
   ```

2. **Databricks Environment:**
   - Upload the contents of the `notebooks/` folder to your Databricks workspace.
   - Ensure that your raw data files are located at `dbfs:/FileStore/tables/GS_Sample_data/` (or update the BASE_PATH in the notebook accordingly).
   - Create or verify the existence of the staging and refined databases (`staging_db` and `refined_db`).

3. **Dependencies:**
   - The solution uses PySpark and Delta Lake which are pre-installed in Databricks.
   - If running locally, install the packages listed in `requirements.txt`.

## Execution Instructions

1. **Run the Notebook:**
   - Open `notebooks/GS_FACT_&_DIM_ETL.ipynb` in your Databricks workspace.
   - Execute all cells sequentially to:
     - Ingest raw data,
     - Validate and clean data,
     - Write cleaned data to staging Delta tables,
     - Create the refined aggregated table (`mview_weekly_sales`), and
     - Apply incremental update logic.

2. **Verify Results:**
   - You can use a SQL cell or the Databricks SQL Editor to query the final refined table:
     ```sql
     SELECT * FROM refined_db.mview_weekly_sales LIMIT 20;
     ```

## Diagrams

- **ER Diagram:**
  - **Editable Source:** `diagrams/GS_FACT_&_DIM_ETL.drawio`
  - **Exported Image:** `diagrams/ER_Diagram.png`
- **Dataflow Diagram:**
  - **Editable Source:** `diagrams/GS_FACT_&_DIM_ETL.drawio`
  - **Exported Image:** `diagrams/Dataflow_Diagram.png`

These diagrams illustrate:
- The full schema of all fact and dimension tables including columns and primary keys.
- The flow of data from raw file ingestion through staging to the refined output, including incremental updates.

## Table-wise Definitions (Data Dictionary)

Below is a summary of the key tables and their definitions. For full details, see the file `docs/Data_Dictionary.md`.

### fact_transactions

| **Column**         | **Data Type** | **Definition**                                                   | **PII Column** | **Sample Value**         |
|--------------------|---------------|------------------------------------------------------------------|----------------|--------------------------|
| order_id           | string        | Unique identifier for each order                                 | N              | 164087401                |
| line_id            | string        | Unique line item ID within an order                              | N              | 2                        |
| type               | string        | Transaction type (Sale, Return, etc.)                            | N              | Sale                     |
| dt                 | timestamp     | Date/time of the transaction                                     | N              | 2016-01-31T06:17:01      |
| pos_site_id        | string        | Foreign key referencing `hier_possite.site_id`                   | N              | CATMAIN                  |
| sku_id             | string        | Foreign key referencing `hier_prod.sku_id`                       | N              | 2668940801               |
| fscldt_id          | string        | Foreign key referencing `hier_clnd.fscldt_id`                    | N              | 20160131                 |
| price_substate_id  | string        | Foreign key referencing `hier_pricestate.substate_id`            | N              | FP                       |
| sales_units        | double        | Number of units sold                                               | N              | 1                        |
| sales_dollars      | double        | Revenue from the sale (in dollars)                               | N              | 58.95                    |
| discount_dollars   | double        | Discount applied on the sale                                       | N              | 0                        |
| original_order_id  | string        | Order ID for returns/exchanges                                     | N              | 164074437                |
| original_line_id   | string        | Line ID for returns/exchanges                                      | N              | 0                        |

### fact_averagecosts

| **Column**                  | **Data Type** | **Definition**                                                       | **PII Column** | **Sample Value** |
|-----------------------------|---------------|----------------------------------------------------------------------|----------------|------------------|
| fscldt_id                   | string        | Foreign key referencing `hier_clnd.fscldt_id`                         | N              | 20160131         |
| sku_id                      | string        | Foreign key referencing `hier_prod.sku_id`                            | N              | 0174410000       |
| average_unit_standardcost   | double        | Average standard cost per unit for the SKU on the given date         | N              | 10.06            |
| average_unit_landedcost     | double        | Average landed cost (including freight, etc.) per unit on the given date| N             | 10.06            |

### hier_possite

| **Column**    | **Data Type** | **Definition**                                  | **PII Column** | **Sample Value**   |
|---------------|---------------|-------------------------------------------------|----------------|--------------------|
| site_id       | string        | Primary key: Unique POS site identifier         | N              | CATMAIN            |
| site_label    | string        | Descriptive label for the POS site              | N              | Catalog Main       |
| subchnl_id    | string        | Sub-channel ID                                  | N              | CATMAIN            |
| subchnl_label | string        | Sub-channel descriptive label                   | N              | Catalog Main       |
| chnl_id       | string        | Channel ID                                      | N              | CAT                |
| chnl_label    | string        | Channel descriptive label                       | N              | Catalog            |

### hier_pricestate

| **Column**     | **Data Type** | **Definition**                                       | **PII Column** | **Sample Value** |
|----------------|---------------|------------------------------------------------------|----------------|------------------|
| substate_id    | string        | Primary key: Price substate identifier               | N              | FP               |
| substate_label | string        | Descriptive label for the price substate             | N              | Full Price       |
| state_id       | string        | Grouping identifier for multiple substates           | N              | MD               |
| state_label    | string        | Label for the grouped price state                    | N              | Markdown         |

### hier_prod

| **Column**     | **Data Type** | **Definition**                                              | **PII Column** | **Sample Value**                     |
|----------------|---------------|-------------------------------------------------------------|----------------|--------------------------------------|
| sku_id         | string        | Primary key: Unique SKU identifier                           | N              | 2668940801                           |
| sku_label      | string        | Human-readable description of the SKU                        | N              | COTTON UNDERWIRE CAMI WHITE 36C      |
| stylclr_id     | string        | Style & color identifier                                     | N              | 2230940                              |
| stylclr_label  | string        | Descriptive label for style & color                          | N              | COTTON UNDERWIRE CAMI WHITE          |
| styl_id        | string        | Style identifier                                             | N              | 22309                                |
| styl_label     | string        | Descriptive label for style                                  | N              | COTTON UNDERWIRE CAMI                |
| subcat_id      | string        | Subcategory identifier                                       | N              | 405                                  |
| subcat_label   | string        | Subcategory label                                            | N              | TOPS                                 |
| cat_id         | string        | Category identifier                                          | N              | 1000                                 |
| cat_label      | string        | Category label                                               | N              | TOPS                                 |
| dept_id        | string        | Department identifier                                        | N              | 2000                                 |
| dept_label     | string        | Department label                                             | N              | APPAREL                              |
| issvc          | string        | Flag indicating if the item is a service                     | N              | 0                                    |
| isasmbly       | string        | Flag indicating if the item is an assembly                   | N              | 0                                    |
| isnfs          | string        | Flag indicating if the item is non-fashion (derived)          | N              | 0                                    |

### hier_clnd

| **Column**     | **Data Type** | **Definition**                                                   | **PII Column** | **Sample Value** |
|----------------|---------------|------------------------------------------------------------------|----------------|------------------|
| fscldt_id      | string        | Primary key: Unique fiscal date identifier                       | N              | 20160131         |
| fscldt_label   | string        | Label for the fiscal date (e.g., "Jan 31, 2016")                   | N              | Jan 31, 2016     |
| fsclwk_id      | string        | Fiscal week identifier                                             | N              | 201601           |
| fsclwk_label   | string        | Fiscal week label                                                  | N              | WK 01, 2016      |
| fsclmth_id     | string        | Fiscal month identifier                                            | N              | 201601           |
| fsclmth_label  | string        | Fiscal month label                                                 | N              | Jan, 2016        |
| fsclqrtr_id    | string        | Fiscal quarter identifier                                          | N              | 20161            |
| fsclqrtr_label | string        | Fiscal quarter label                                               | N              | Q1, 2016         |
| fsclyr_id      | string        | Fiscal year identifier                                             | N              | 2016             |
| fsclyr_label   | string        | Fiscal year label                                                  | N              | 2016             |
| ssn_id         | string        | Season identifier                                                  | N              | SPRG2016         |
| ssn_label      | string        | Season label                                                       | N              | Spring 2016      |
| ly_fscldt_id   | string        | Corresponding date identifier from last year                       | N              | 20150131         |
| lly_fscldt_id  | string        | Corresponding date identifier from two years ago                    | N              | 20140131         |
| fscldow        | string        | Fiscal day of week                                                 | N              | 7                |
| fscldom        | string        | Fiscal day of month                                                | N              | 31               |
| fscldoq        | string        | Fiscal day of quarter                                              | N              | 31               |
| fscldoy        | string        | Fiscal day of year                                                 | N              | 31               |
| fsclwoy        | string        | Fiscal week of year                                                | N              | 5                |
| fsclmoy        | string        | Fiscal month of year                                               | N              | 2                |
| fsclqoy        | string        | Fiscal quarter of year                                             | N              | 1                |
| date           | date          | Actual calendar date in YYYY-MM-DD format                          | N              | 2016-01-31       |

### hier_rtlloc

| **Column**   | **Data Type** | **Definition**                                             | **PII Column** | **Sample Value**    |
|--------------|---------------|------------------------------------------------------------|----------------|---------------------|
| str          | string        | Primary key: Unique store identifier                       | N              | 118                 |
| str_label    | string        | Descriptive name of the store                              | N              | Evergreen Walk      |
| dstr         | string        | District identifier                                        | N              | 11                  |
| dstr_label   | string        | District label                                             | N              | Northeast           |
| rgn          | string        | Region identifier                                          | N              | 11                  |
| rgn_label    | string        | Region label                                               | N              | East                |

### hier_invstatus

| **Column**     | **Data Type** | **Definition**                                      | **PII Column** | **Sample Value** |
|----------------|---------------|-----------------------------------------------------|----------------|------------------|
| code_id        | string        | Primary key: Inventory status code                | N              | INV-900          |
| code_label     | string        | Descriptive label for the status                  | N              | Available        |
| bckt_id        | string        | Bucket identifier (e.g., IBIT, IBOO)                | N              | OHAV             |
| bckt_label     | string        | Descriptive label for the bucket                  | N              | Available        |
| ownrshp_id     | string        | Ownership identifier (e.g., IB, OH)                 | N              | OH               |
| ownrshp_label  | string        | Descriptive label for ownership                   | N              | On Hand          |

### hier_hldy

| **Column**  | **Data Type** | **Definition**                                  | **PII Column** | **Sample Value**   |
|-------------|---------------|-------------------------------------------------|----------------|--------------------|
| hldy_id     | string        | Primary key: Holiday identifier                 | N              | Valentines_Day     |
| hldy_label  | string        | Descriptive label for the holiday               | N              | Valentine's Day    |

### hier_invloc

| **Column**    | **Data Type** | **Definition**                                    | **PII Column** | **Sample Value**  |
|---------------|---------------|---------------------------------------------------|----------------|-------------------|
| loc           | string        | Primary key: Inventory location identifier        | N              | 103               |
| loc_label     | string        | Label for the inventory location                  | N              | Catalog/Internet  |
| loctype       | string        | Type of location (e.g., DC, Store)                | N              | DC                |
| loctype_label | string        | Descriptive label for the location type           | N              | DC                |

## Architecture Overview

Please refer to `docs/Architecture_Overview.md` for a detailed explanation of the ETL pipeline architecture, key transformation steps, incremental load logic, and design decisions.


## Contact

For any questions or clarifications, please contact:

- **Email:** [Haridharan M](mailto:harimicro05@gmail.com)
- **LinkedIn:** [Haridharan M](https://www.linkedin.com/in/haridharanm-bajajfinserv/)
- **Portfolio:** [MyCreativeSpace]( https://haridharan05.github.io/MyCreativeSpace/)

---

© 2025 GSynergy - Haridharan M