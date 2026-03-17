# Uber-Project
## Context: 
Ride-sharing platforms generate data from multiple systems at high speed, including trip events from applications, operational data from transactional systems, and external batch datasets used for reporting and analysis. The challenge is that this data often arrives in different formats, at different speeds, and from different sources, making it difficult to create a unified, analytics-ready foundation for business reporting, operational monitoring, and decision-making.
This project solves that problem by building an end-to-end Azure-based data platform that ingests both real-time streaming data and batch data, processes it through a structured medallion architecture, and transforms it into business-ready datasets for analytics. Using Azure Data Factory, Event Hubs, Azure Storage, and Databricks, the solution standardizes raw inputs, creates curated silver-layer datasets, and further models the data into a star schema with fact and dimension tables for downstream BI and reporting use cases.
The result is a scalable modern data pipeline that demonstrates how transportation or mobility data can be integrated, processed, and modeled for near real-time analytics, operational insights, and dashboard-ready consumption.

**What I Solved**
- Built a unified pipeline to handle both batch and streaming data
- Automated ingestion from source systems into Azure storage
- Processed raw data into clean, structured silver tables
- Designed a business-friendly star schema for analytics use cases
- Created a foundation for reporting on trips, customers, locations, and operational performance
- Demonstrated how modern Azure services can be combined to support scalable, analytics-ready data engineering workflows


### Uber Project in Azure
End-to-end Azure + Databricks project that captures **real-time ride events** from a web app via **Azure Event Hubs** and combines them with **batch reference / bulk ride data** ingested from **GitHub APIs** using **Azure Data Factory (ADF)**. The pipeline outputs both:
1) a **business-ready Silver “One Big Table” (OBT)** for direct analytics, and  
2) a **Gold Star Schema** (dimensions + fact) for scalable BI and semantic modeling.

## Project Architecture
<img width="1161" height="625" alt="image" src="https://github.com/user-attachments/assets/0afd5906-3426-4f9e-a80b-78faa5a22eec" />

### High-level Flow
- **Streaming path:** Web app → Event Hubs → Databricks (Kafka stream) → `rides_raw` → `stg_rides` → `silver_obt`
- **Batch path:** GitHub API → ADF pipeline (`API_to_ADLS`) → ADLS → Databricks → `stg_rides` → `silver_obt`
- **Modeling path:** `silver_obt` → Gold star schema (dims + fact) with DLT CDC/SCD

## ADF pipeline for data extraction
<img width="1904" height="902" alt="image" src="https://github.com/user-attachments/assets/1bd15c5b-a20f-4da9-9b6f-64faa04483b6" />

1) Batch ingestion (GitHub/API → ADLS via ADF)
- ADF pipeline API_to_ADLS reads a list of JSON files (Lookup), loops through each item (ForEach), and copies each GitHub JSON into ADLS as raw landing data.


## Databricks pipeline for Processing and Modeling layer
<img width="2242" height="1550" alt="image" src="https://github.com/user-attachments/assets/76a4ba7a-d1e9-45a2-925b-c9ddf8151d28" />


<img width="2288" height="660" alt="image" src="https://github.com/user-attachments/assets/fe92bfa5-363e-4315-bc75-d01703be44ae" />

2) Real-time ingestion (Event Hubs → Databricks)
- Databricks reads Event Hubs through Kafka options and creates a streaming table rides_raw (DLT).

3) Converged Silver staging table (Batch + Stream)
A unified streaming staging table stg_rides appends:
- Bulk/batch rides data
- Real-time rides events (JSON payload parsed using schema)

4) Silver OBT (Business-ready analytics table)
silver_obt is built as a streaming table with:
- Enrichment joins to mapping tables (vehicle types, cities, statuses, payment methods, cancellation reasons)
- A watermark on booking time with late-arrival handling

5) Gold Layer (Star Schema + CDC/SCD)
From silver_obt, the project builds:
- Dimensions: passenger, driver, vehicle, payments, booking, location
- Fact: ride-level fact table
CDC/SCD is handled using DLT create_auto_cdc_flow:
- SCD Type 1 for most dims + fact
- SCD Type 2 for location changes over time
