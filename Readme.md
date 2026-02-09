# Zoomcamp Data Engineering Module 3 Homework: Data Warehousing & BigQuery

In this project, I worked with NYC Yellow Taxi trip data (Jan–Jun 2024) to explore how BigQuery storage design decisions directly affect query performance and cost.

The focus wasn’t just on writing SQL; it was on understanding how BigQuery thinks: columnar storage, external vs native tables, partitioning, clustering, and metadata optimizations

<img width="465" height="424" alt="image" src="https://github.com/user-attachments/assets/2ce5a532-05c9-469e-aa0b-65ef8b319cd0" />

## BigQuery Setup

#### Create an external table using the Yellow Taxi Trip Records.
```
CREATE OR REPLACE EXTERNAL TABLE `solar-router-483810-s0.homework_3.yellow_taxi_external`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://solar-router-483810-s0-ny-taxi-data/yellow_tripdata_2024-*.parquet']
);

SELECT *
FROM `solar-router-483810-s0.homework_3.yellow_taxi_external`
LIMIT 5;
```

#### Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table).

```
CREATE OR REPLACE TABLE `solar-router-483810-s0.homework_3.yellow_taxi_native`
AS
SELECT *
FROM `solar-router-483810-s0.homework_3.yellow_taxi_external`;
```

#### question 1 - What is count of records for the 2024 Yellow Taxi Data?
````
SELECT COUNT(*) FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`;
````

#### question 2 - Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables. What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?
````
SELECT DISTINCT PULocationID FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`;
````
**This query will process 155.12 MB when run.**

````
SELECT DISTINCT PULocationID FROM `solar-router-483810-s0.homework_3.yellow_taxi_external`;
````
**This query will process 0 B when run.**

#### question 3 - Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
````
SELECT PULocationID
FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`;
````
````
SELECT PULocationID, DOLocationID
FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`;
````
#### question 4: How many records have a fare_amount of 0?
````
SELECT COUNT(*) FROM `solar-router-483810-s0.homework_3.yellow_taxi_native` WHERE fare_amount = 0;
````
#### question 5: What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)
````
CREATE OR REPLACE TABLE `solar-router-483810-s0.homework_3.yellow_taxi_optimized`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID
AS
SELECT *
FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`;
````
#### question 6: Question 6: Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive).Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?
````
SELECT DISTINCT VendorID
FROM `solar-router-483810-s0.homework_3.yellow_taxi_optimized`
WHERE tpep_dropoff_datetime >= '2024-03-01' and tpep_dropoff_datetime <= '2024-03-15';
````
**this is partitioned giving 26.84 MB**

````
SELECT DISTINCT VendorID
FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`
WHERE tpep_dropoff_datetime >= '2024-03-01' and tpep_dropoff_datetime <= '2024-03-15';
````
**this is non-partitioned giving 310.24 MB**


SELECT COUNT(*)
FROM `solar-router-483810-s0.homework_3.yellow_taxi_native`;
