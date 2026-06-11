# End-to-End Azure Data Engineering Platform
### Enterprise-Grade Data Platform Utilizing Medallion Architecture, Azure Databricks, Synapse Analytics, and Power BI

[![Azure Data Factory](https://img.shields.io/badge/Azure-Data%20Factory-Blue?logo=microsoftazure&style=flat-square)](https://azure.microsoft.com/en-us/products/data-factory/)
[![Azure Databricks](https://img.shields.io/badge/Azure-Databricks-Red?logo=databricks&style=flat-square)](https://azure.microsoft.com/en-us/products/databricks/)
[![Azure Synapse](https://img.shields.io/badge/Azure-Synapse%20Analytics-blueviolet?logo=microsoftazure&style=flat-square)](https://azure.microsoft.com/en-us/products/synapse-analytics/)
[![Power BI](https://img.shields.io/badge/Power%20BI-Data%20Visualization-yellow?logo=powerbi&style=flat-square)](https://powerbi.microsoft.com/)

---

## 📋 Executive Summary
This repository contains the production-ready infrastructure, control-flow orchestration, and transformation assets for an enterprise-scale **Cloud Data Platform**.

The system implements a robust **Medallion Architecture** pattern to ingest, standardize, and systematically model data sourced from legacy on-premises transactional databases and enterprise resource files. By migrating raw transactional entities into a structured, unified analytical warehouse tier, the platform enables executive stakeholder layers to perform sub-second, interactive reporting within optimized Power BI dashboards.

### Key Engineering Highlights
* **Dynamic, Metadata-Driven Orchestration:** Eradicated pipeline hardcoding by constructing an abstract, parameterized ingestion framework inside Azure Data Factory (ADF) utilizing `Lookup` and sequential `ForEach` loops to process multiple core entities concurrently.
* **Decoupled Lakehouse Transfomations:** Leveraged PySpark optimization engines within Azure Databricks to handle data cleaning, relational data type parsing, deduplication, and atomic schema evolution tracking.
* **Enterprise Security Guardrails:** Designed with a zero-trust mindset—centralizing all infrastructure connection strings, database tokens, and master passwords within **Azure Key Vault** utilizing implicit system-assigned **Managed Identities (MSI)**.

---

## 🏗️ Technical Architecture & Data Infrastructure

The end-to-end data platform decouples storage layers from elastic compute resource pools to maximize cost efficiency and scalability limits:

<p align="center">
  <img src="solution_architecture.png" alt="End-to-End Azure Data Platform Architecture" width="100%">
</p>

### Operational Component Breakdown

1. **Enterprise Data Sources:** Transactional databases (On-Premises SQL Server) and peripheral ERP file arrays tracking vital customer, product, and sales cycles.
2. **Orchestration Layer (Azure Data Factory):** Acts as the centralized scheduling and compute runtime coordinator, executing automated batch extraction windows seamlessly.
3. **Landing & Lake Storage (ADLS Gen2):** Hierarchical cloud object store configuring an append-only, immutable landing tier for data assets.
4. **Data Transformation Compute (Azure Databricks):** Orchestrates the phased transformation engine:
   * **Bronze Layer:** Retains raw, untouched transactional data history for point-in-time state reconstruction.
   * **Silver Layer:** Standardizes naming syntax, casts schema structures, resolves structural null states, and deduplicates historical deltas.
   * **Gold Layer:** Builds aggregated KPI tracking, calculates analytical margins, and maps data assets onto optimized analytical Star Schemas.
5. **Enterprise Data Warehouse (Azure Synapse Analytics):** Hosts a Dedicated SQL Pool running Massively Parallel Processing (MPP) architectures to serve curated golden semantic data products.
6. **Business Intelligence (Power BI Enterprise):** Direct semantic structures pulling analytical views to serve cross-department analytical dashboards.

---

## 🗄️ Relational Database Entity Model (Source Layer)

The source system relies on a normalization pattern spanning multi-table dependencies across sales, address matrixing, and detailed inventory hierarchies:

<p align="center">
  <img src="dataModel/data_model.png" alt="Source Relational Entity Diagram" width="85%">
</p>

### Key Relational Components Transformed:
* **Core Transaction Tables:** Transaction loops track through `SalesOrderHeader` and nested `SalesOrderDetail` tables referencing core billing lines.
* **Customer Dimensions:** Normalizes enterprise identifiers across `Customer`, `Address`, and intermediary `CustomerAddress` relationship models.
* **Product Catalog Graphing:** Relates discrete storage parameters through recursive hierarchies spanning `Product`, `ProductCategory`, `ProductModel`, and localized `ProductDescription` mappings.

---

## 🚀 Ingestion & Transformation Pipeline Implementation

### 1. Control-Flow Parameterization
The core ingestion pipeline (`copy-all-data`) queries an isolated catalog mapping table to compute target path distributions and handle system data drift dynamically:

<p align="center">
  <img src="Architecture/data_flow.png" alt="Azure Data Factory Ingestion Logic" width="100%">
</p>

* **`Lookup1` Activity:** Queries the dynamic configuration register containing valid target data boundaries, active table primary keys, and storage paths.
* **`ForEach1` Activity:** Spawns parallel batch worker processes to stream incoming parameters concurrently without linear thread locking.
* **`Copy data1` Activity:** An internal parameterized streaming block mapping relational database sources straight into partitioned raw storage folders.

### 2. Multi-Tier Medallion Pipeline Automation
Following a successful landing trigger, ADF initiates multi-stage notebook processes over active Spark clusters to advance the data's maturity state:

<p align="center">
  <img src="Architecture/medallion_architecture.png" alt="Azure Databricks Automated Notebook Pipeline" width="100%">
</p>

* **`Bronze to Silver` Notebook:** Executes automated structural column validations, handles broken character sequences, converts datatypes to proper unified formats, and generates clean delta storage records.
* **`Silver to Gold` Notebook:** Merges complex transactional entity boundaries (e.g., matching tracking orders directly to customer models) to establish high-performance analytical star schemas.
