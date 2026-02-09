# DTC_HW3

### Overview
The main goal was to explore data ingestion patterns, external tables, partitioning, clustering strategies, and query optimization techniques using NYC Taxi trip data.
The workflow demonstrates a common Data Lake → Data Warehouse architecture, where raw Parquet files are stored in GCS and queried through BigQuery.

I used a Jupyter Notebook with a DLT pipeline to ingest Parquet files containing NYC Yellow Taxi trip data into a Google Cloud Storage bucket. From there, I created an external BigQuery table referencing the files stored in GCS.

```sql
CREATE OR REPLACE EXTERNAL TABLE rides_dataset.yellow_tripdata_external
OPTIONS (
  FORMAT='PARQUET',
  uris=['gs://warehouse-zoomcamp/rides_dataset/rides/yellow_tripdata_2024_*_parquet.parquet']
);

-- Q1: Count rows in external table
SELECT COUNT(*)
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_external`;


-- Q2: Compare distinct pickup locations between external and materialized tables
SELECT COUNT(DISTINCT pu_location_id)
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_external`;

SELECT COUNT(DISTINCT pu_location_id)
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_nonpartitioned`;


-- Q3: Compare column scan behavior
SELECT pu_location_id
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_nonpartitioned`;

SELECT pu_location_id, do_location_id
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_nonpartitioned`;


-- Q4: Count trips with zero fare
SELECT COUNT(*)
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_external`
WHERE fare_amount = 0;


-- Q5: Create partitioned and clustered table
CREATE OR REPLACE TABLE `rides_dataset.yellow_tripdata_partitioned`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY vendor_id
AS
SELECT *
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_external`;


-- Q6: Compare vendor distribution between partitioned and non-partitioned tables
SELECT DISTINCT vendor_id
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_nonpartitioned`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';

SELECT DISTINCT vendor_id
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_partitioned`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';


-- Q9: Demonstrate metadata optimization behavior
SELECT COUNT(*)
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_nonpartitioned`;

-- Comparison query forcing column scan
SELECT COUNT(*)
FROM `data-warehouse-dtc.rides_dataset.yellow_tripdata_nonpartitioned`
WHERE vendor_id = 6;
