# SQL Data Warehouse Project

## Overview

This project demonstrates the end-to-end development of a modern data warehouse using **SQL Server** and **T-SQL**. It integrates data from CRM and ERP source systems, applies data cleansing and transformation rules, and delivers a business-ready star schema for reporting and analytics.

The project follows the **Medallion Architecture**, organizing data into Bronze, Silver, and Gold layers.

## Project Objectives

- Consolidate CRM and ERP data into a centralized data warehouse.
- Build repeatable data pipelines using SQL stored procedures.
- Clean, standardize, and enrich raw source data.
- Resolve data quality issues such as duplicates, missing values, invalid dates, and inconsistent codes.
- Create a star schema optimized for analytical queries.
- Document the architecture, data flow, and final analytical model.

## Data Architecture

The architecture moves CRM and ERP data from CSV source files through three processing layers before making it available for reporting and analytics.

![Data Warehouse Architecture](docs/data_warehouse_archi.drawio.png)

### Architecture Layers

| Layer | Purpose | Main Operations |
|---|---|---|
| **Bronze** | Stores raw CRM and ERP data in its original form. | CSV ingestion, full load, truncate and insert |
| **Silver** | Produces clean and standardized datasets. | Deduplication, normalization, null handling, date validation, derived columns |
| **Gold** | Delivers business-ready analytical views. | Data integration, business rules, dimensional modeling |

## How to Run the Project

### Prerequisites

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)
- Permission to create databases, schemas, tables, views, and stored procedures

### Execution Order

1. Run `scripts/init_database.sql` to create the database and the Bronze, Silver, and Gold schemas.
2. Run the Bronze DDL script to create the raw-data tables.
3. Update the CSV file paths inside the Bronze loading procedure for your local environment.
4. Execute the Bronze loading procedure:

```sql
EXEC bronze.load_bronze;
```

5. Run the Silver DDL script to create the cleaned-data tables.
6. Execute the Silver transformation procedure:

```sql
EXEC silver.load_silver;
```

7. Run `gold/ddl_gold.sql` to create the analytical views.
8. Query the Gold layer:

```sql
SELECT TOP 100 *
FROM gold.fact_sales;
```

> **Important:** The `BULK INSERT` file paths are environment-specific and must be updated before executing the Bronze loading procedure.

## ETL Process

### 1. Bronze Layer — Raw Data Ingestion

The Bronze layer receives raw CSV files from the CRM and ERP source systems. Its purpose is to preserve the source data without applying business transformations.

The `bronze.load_bronze` stored procedure:

- Truncates the existing Bronze tables before each full load.
- Loads CRM and ERP CSV files with `BULK INSERT`.
- Separates the source data into six raw tables.
- Records the duration of each table load and the complete batch.
- Uses `TRY...CATCH` to capture loading errors.

**CRM source tables:**

- `bronze.crm_cust_info`
- `bronze.crm_prd_info`
- `bronze.crm_sales_details`

**ERP source tables:**

- `bronze.erp_cust_az12`
- `bronze.erp_loc_a101`
- `bronze.erp_px_cat_g1v2`

### 2. Silver Layer — Cleaning and Transformation

The Silver layer transforms the raw Bronze data into clean, standardized, and analysis-ready tables.

The `silver.load_silver` stored procedure applies transformations such as:

- Keeping the most recent customer record with `ROW_NUMBER()`.
- Standardizing gender, marital status, country, and product-line values.
- Removing unwanted prefixes and characters from business keys.
- Replacing missing product costs.
- Validating and converting date fields.
- Deriving product end dates with `LEAD()`.
- Recalculating missing or invalid sales and price values.
- Recording table-level and batch-level loading durations.

Each Bronze table has a corresponding Silver table, which preserves the source subject areas while improving data quality and consistency.

### 3. Gold Layer — Business Views and Data Model

The Gold layer integrates the cleaned Silver datasets into business-ready views optimized for reporting and analysis.

It uses a **star schema** consisting of two dimensions and one fact view:

- `gold.dim_customers`: combines CRM customer information with ERP demographics and location data.
- `gold.dim_products`: enriches CRM product records with ERP category and subcategory information.
- `gold.fact_sales`: stores sales transactions connected to customers and products through surrogate keys.

The Gold transformation logic also:

- Filters products to retain only their current active records.
- Creates surrogate keys for the customer and product dimensions.
- Applies source-priority rules when the same attribute exists in CRM and ERP.
- Connects sales transactions to the appropriate customer and product records.

![Gold Layer Star Schema](docs/gold_layer_star_schema.drawio.png)

### Gold-Layer Relationships

| Dimension | Primary Key | Fact View Foreign Key | Relationship |
|---|---|---|---|
| `gold.dim_customers` | `customer_key` | `gold.fact_sales.customer_key` | One-to-many |
| `gold.dim_products` | `product_key` | `gold.fact_sales.product_key` | One-to-many |

For detailed column definitions, data types, and business descriptions, see the [Data Catalog](docs/data_catalog.md).

## Data Flow and Lineage

The following diagram provides a table-level view of the complete pipeline. It shows how CRM and ERP files are loaded into Bronze, transformed into matching Silver tables, and integrated into the Gold analytical views.

![Data Warehouse Flow](docs/data_warehouse_flow.png)

The main lineage paths are:

- CRM sales data flows into `gold.fact_sales`.
- CRM customer data is combined with ERP demographics and location data to create `gold.dim_customers`.
- CRM product data is enriched with ERP category data to create `gold.dim_products`.
- The three Gold views form the final star schema used for reporting and analysis.

## Repository Structure

```text
SQL-data-Warehouse-Project/
├── datasets/
│   ├── source_crm/
│   └── source_erp/
├── docs/
│   ├── data_catalog.md
│   ├── data_warehouse_archi.drawio.png
│   ├── data_warehouse_flow.png
│   └── gold_layer_star_schema.drawio.png
├── gold/
│   └── ddl_gold.sql
├── scripts/
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   └── init_database.sql
└── README.md
```

## Key SQL Concepts Demonstrated

- Medallion architecture
- ETL pipeline development
- Stored procedures
- `BULK INSERT`
- Window functions: `ROW_NUMBER()` and `LEAD()`
- Data cleansing and standardization
- Derived columns and business rules
- `LEFT JOIN` data integration
- Star-schema dimensional modeling
- Dimension and fact views
- Surrogate and business keys
- Error handling with `TRY...CATCH`

## Technologies

- **Database:** Microsoft SQL Server
- **Language:** T-SQL
- **Data sources:** CSV files
- **Modeling approach:** Star schema
- **Architecture:** Bronze, Silver, and Gold layers
- **Documentation:** Markdown and diagrams.net

## Author

**Alberto Franco**

This project was created as part of my data analytics and data engineering portfolio to demonstrate practical SQL, ETL, data modeling, and technical documentation skills.
