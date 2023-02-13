# Week 3 Homework

# Question 1: What is the count for fhv vehicle records for year 2019?

```sql
SELECT count(*) FROM `igneous-ethos-375619.fhv_tripdata.fhv_2019`;
```

# Question 2: Write a query to count the distinct number of affiliated_base_number for the entire dataset on both the tables.

```sql
-- BigQuery Table count Distinct Affiliated_base_number
SELECT COUNT (DISTINCT Affiliated_base_number) FROM `igneous-ethos-375619.fhv_tripdata.fhv_2019`;

-- External Table count Distinct Affiliated_base_number
SELECT COUNT (DISTINCT Affiliated_base_number) FROM `igneous-ethos-375619.fhv_tripdata.fhv_2019_external`;
```

# Question 3: How many records have both a blank (null) PUlocationID and DOlocationID in the entire dataset?

```sql
SELECT COUNT(*) FROM `igneous-ethos-375619.fhv_tripdata.fhv_2019`
WHERE PUlocationID IS NULL AND DOlocationID IS NULL;
```

# Question 4: What is the best strategy to make an optimized table in Big Query if your query will always filter by pickup_datetime and order by affiliated_base_number?

It would make more sense use pickup_datetime for partitioning because:
1. Number of partition limit is 4000, affiliated_base_number could be more than 4000 but we can adjust time column to Daily, Hourly, etc.
2. Partitioning can be done on Integer but Affiliated_base_number is a STRING type

# Question 5: Write a query to retrieve the distinct affiliated_base_number between pickup_datetime 03/01/2019 and 03/31/2019 (inclusive)

```sql
-- Non-Partition Table
SELECT COUNT (DISTINCT Affiliated_base_number) FROM `igneous-ethos-375619.fhv_tripdata.fhv_2019`
WHERE DATE(pickup_datetime) BETWEEN '2019-03-01' AND '2019-03-31';

--Partitioned and Clustered Table
SELECT COUNT (DISTINCT Affiliated_base_number) FROM `igneous-ethos-375619.fhv_tripdata.fhv_2019_clustered_partitioned`
WHERE DATE(pickup_datetime) BETWEEN '2019-03-01' AND '2019-03-31';
```
