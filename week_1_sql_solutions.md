### Question 1: Knowing Docker tags

I went to docker documentation to get this information that --iidfile string has the text "Write the image ID to the file" and read the purpose of this tag

### Question 2: Understanding Docker first run: Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed ( use pip list). How many python packages/modules are installed?

After setting up my GCP VM Instance I opened Python 3.9 image on docker in Interactive mode and the entrypoint of bash as following

```bash
docker run -it --entrypoint=bash python:3.9
```

And when I was able to get in I typed following command to get the Python packages in pip and their versions

```bash
python -m pip list
```


### Question 3: Count Records: How many taxi trips were totally made on January 15?

```sql
SELECT COUNT(*) FROM green_trip_data
WHERE DATE(lpep_pickup_datetime) = '2019-01-15';
```

### Question 4: Largest Trip for Each Day: Which was the day with the largest trip distance Use the pick up time for your calculations.

```sql
SELECT lpep_pickup_datetime, trip_distance FROM green_trip_data
WHERE trip_distance = (SELECT MAX(trip_distance) FROM green_trip_data);
```

### Question 5: The number of Passengers: In 2019-01-01 how many trips had 2 and 3 passengers?

```sql
SELECT DATE(lpep_pickup_datetime) as date, passenger_count, COUNT(*) as trip_count
FROM green_trip_data
WHERE passenger_count IN (2,3) and DATE(lpep_pickup_datetime) = '2019-01-01'
GROUP BY DATE(lpep_pickup_datetime), passenger_count;
```

### Question 6: Largest Tip: For the passengers picked up in the Astoria Zone which was the drop off zone that had the largest tip? We want the name of the zone, not the id.

```sql
SELECT dropoff_loc, MAX(tip_amount) as total_tip
FROM joined_trip_data
WHERE pickup_loc LIKE '%Astoria%'
GROUP BY dropoff_loc
ORDER BY total_tip DESC
LIMIT 1;
```
