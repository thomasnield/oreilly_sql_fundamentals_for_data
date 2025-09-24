# Exercises

Using station_data....
1) SELECT all records where TEMPERATURE is between 30 and 50 degrees.
2) SELECT all records where station_pressure is greater than 1000 and a tornado was present
3) SELECT all records with report codes E6AED7, B950A1, 98DDAD
4) SELECT all records WHERE station_pressure is null
5) Find the SUM of precipitation by year when a tornado was present, and sort by year descending.
6) SELECT the year and max snow depth, but only years where the max snow depth is at least 50.
7) SELECT the report_code, year, quarter, and temperature, where a “quarter” is “Q1”, “Q2”, “Q3”, or “Q4” reflecting months 1-3, 4-6, 7-9, and 10-12 respectively.
8) Get the average temperature by quarter and year, where a “quarter” is “Q1”, “Q2”, “Q3”, or “Q4” reflecting months 1-3, 4-6, 7-9, and 10-12 respectively.

Using rexon_metals.db
9) SELECT the ORDER_ID, ORDER_DATE, and DESCRIPTION (from PRODUCT) (hint, you will need to INNER JOIN CUSTOMER_ORDER and PRODUCT)
10) Find the total revenue by product. Include the fields PRODUCT_ID, DESCRIPTION, and then the TOTAL_REVENUE.


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
-- SELECT all records with report codes E6AED7, B950A1, 98DDAD
SELECT * FROM STATION_DATA
WHERE report_code IN ('E6AED7','B950A1','98DDAD')
-- OR
SELECT * FROM STATION_DATA
WHERE report_code = 'E6AED7'
OR report_code = 'B950A1'
OR report_code = '98DDAD'
```

4)
```sql
-- SELECT all records WHERE station_pressure is null

SELECT * FROM STATION_DATA
WHERE station_pressure IS NULL;
```

5)
```
-- Find the SUM of precipitation by year when a tornado was present, and sort by year descending.

SELECT year, 
SUM(precipitation) as tornado_precipitation
FROM station_data
WHERE tornado = 1
GROUP BY year
ORDER BY year DESC
```

6)
```
-- SELECT the year and max snow depth, but only years where the max snow depth is at least 50.

SELECT year, 
max(snow_depth) AS max_snow_depth
FROM STATION_DATA
GROUP BY year
HAVING max(snow_depth) >= 50
```

7)
```sql
SELECT

report_code,
year,

CASE
    WHEN month BETWEEN 1 and 3 THEN 'Q1'
    WHEN month BETWEEN 4 and 6 THEN 'Q2'
    WHEN month BETWEEN 7 and 9 THEN 'Q3'
    WHEN month BETWEEN 10 and 12 THEN 'Q4'
END as quarter,

temperature

FROM STATION_DATA
```

8) 
```sql
SELECT
year,

CASE
    WHEN month BETWEEN 1 and 3 THEN 'Q1'
    WHEN month BETWEEN 4 and 6 THEN 'Q2'
    WHEN month BETWEEN 7 and 9 THEN 'Q3'
    WHEN month BETWEEN 10 and 12 THEN 'Q4'
END as quarter,

AVG(temperature) as avg_temp

FROM STATION_DATA
GROUP BY year, quarter
```

9) 
```sql
/*
SELECT the ORDER_ID, ORDER_DATE, and DESCRIPTION (from PRODUCT)
(hint, you will need to INNER JOIN CUSTOMER_ORDER and PRODUCT)
*/
SELECT ORDER_ID, ORDER_DATE, DESCRIPTION

FROM CUSTOMER_ORDER INNER JOIN PRODUCT
ON CUSTOMER_ORDER.PRODUCT_ID = PRODUCT.PRODUCT_ID
```

10) 
```sql
/*
Find the total revenue by product. Include the fields PRODUCT_ID, DESCRIPTION, and then the TOTAL_REVENUE.
(Hint: you will need to join CUSTOMER_ORDER and PRODUCT. Then do a GROUP BY)
*/
SELECT PRODUCT.PRODUCT_ID,
DESCRIPTION,
NVL(SUM (ORDER_QTY * PRICE), 0) AS TOTAL_REVENUE

FROM PRODUCT LEFT JOIN CUSTOMER_ORDER
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID
GROUP BY PRODUCT.PRODUCT_ID, DESCRIPTION
```
