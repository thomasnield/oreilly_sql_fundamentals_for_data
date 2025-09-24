# Exercises

Using station_data....
1) SELECT all records where TEMPERATURE is between 30 and 50 degrees.
2) SELECT all records where station_pressure is greater than 1000 and a tornado was present
3) SELECT all records with report codes E6AED7, B950A1, 98DDAD
4) SELECT all records WHERE station_pressure is null
5) 

## Answers

1)
```sql
   SELECT * FROM station_data
WHERE temperature BETWEEN 30 AND 50;
-- OR
SELECT * FROM station_data
WHERE temperature >= 30 and temperature <= 50;
```

2)
```sql
   SELECT * FROM STATION_DATA
WHERE station_pressure > 1000 AND tornado = 1;
-- OR
SELECT * FROM STATION_DATA
WHERE station_pressure > 1000 AND tornado = 1;
```

3)
```sql
   SELECT * FROM STATION_DATA
WHERE station_pressure > 1000 AND tornado = 1;
-- OR
SELECT * FROM STATION_DATA
WHERE station_pressure > 1000 AND tornado = 1;
```
