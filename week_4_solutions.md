# Week 4 Homework

# Q1: What is the count of records in the model fact_trips after running all models with the test run variable disabled and filtering for 2019 and 2020 data only (pickup datetime)?

After doing deduplication of green and yellow trips, when we run the below script, it will get the accurate answer (Image attached for the output)

```sql
{{ config(materialized='table') }}

with green_data as (
    select *, 
        'Green' as service_type -- Adding another column called service type and all fields will be Green to identify the green taxi data
    from {{ ref('stg_green_tripdata') }}
), 

yellow_data as (
    select *, 
        'Yellow' as service_type -- Adding another column called service type and all fields will be Green to identify the green taxi data
    from {{ ref('stg_yellow_tripdata') }}
), 

trips_unioned as (
    select * from green_data
    union all
    select * from yellow_data
), 

dim_zones as (
    select * from {{ ref('dim_zones') }}
    where borough != 'Unknown'
)
select 
    trips_unioned.tripid, 
    trips_unioned.vendorid, 
    trips_unioned.service_type,
    trips_unioned.ratecodeid, 
    trips_unioned.pickup_locationid, 
    pickup_zone.borough as pickup_borough, 
    pickup_zone.zone as pickup_zone, 
    trips_unioned.dropoff_locationid,
    dropoff_zone.borough as dropoff_borough, 
    dropoff_zone.zone as dropoff_zone,  
    trips_unioned.pickup_datetime, 
    trips_unioned.dropoff_datetime, 
    trips_unioned.store_and_fwd_flag, 
    trips_unioned.passenger_count, 
    trips_unioned.trip_distance, 
    trips_unioned.trip_type, 
    trips_unioned.fare_amount, 
    trips_unioned.extra, 
    trips_unioned.mta_tax, 
    trips_unioned.tip_amount, 
    trips_unioned.tolls_amount, 
    trips_unioned.ehail_fee, 
    trips_unioned.improvement_surcharge, 
    trips_unioned.total_amount, 
    trips_unioned.payment_type, 
    trips_unioned.payment_type_description, 
    trips_unioned.congestion_surcharge
from trips_unioned
inner join dim_zones as pickup_zone
on trips_unioned.pickup_locationid = pickup_zone.locationid
inner join dim_zones as dropoff_zone
on trips_unioned.dropoff_locationid = dropoff_zone.locationid
```
![alt text](https://github.com/AbdulWahabAbrar/de-zoomcamp-2023/blob/main/q1.png "Fact Table Count")

# Q2: What is the distribution between service type filtering by years 2019 and 2020 data as done in the videos?

Yellow Service Type: 89.9; Green Service Type: 10.1

# Q3: What is the count of records in the model stg_fhv_tripdata after running all models with the test run variable disabled (:false)?

```sql
{{ config(materialized='view') }}

select

    dispatching_base_num,
   -- identifiers
    cast(pulocationid as integer) as  pickup_locationid,
    cast(dolocationid as integer) as dropoff_locationid,
    
    -- timestamps
    cast(pickup_datetime as timestamp) as pickup_datetime,
    cast(dropoff_datetime as timestamp) as dropoff_datetime,
    
    -- trip info
    cast(SR_Flag as integer) as share_ride_flag,
    Affiliated_base_number

from {{source('staging', 'fhv_tripdata')}}
-- dbt build --m <model.sql> --var 'is_test_run: false'
{% if var('is_test_run', default=true) %}

  limit 100

{% endif %}
```

If we compile the above code with ```bash dbt build --var 'is_test_run: false'```, It will get the accurate result (Screenshot attached below)

![alt text](https://github.com/AbdulWahabAbrar/de-zoomcamp-2023/blob/main/q3.png "Stg FHV Table Count")

# Q4: 
