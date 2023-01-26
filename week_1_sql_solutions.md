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