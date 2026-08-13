# Grouping and Sorting 

In this section, we will learn how to summarize records using SQL's `GROUP BY` and `ORDER BY` operators. Along the way we will learn aggregating functions like `SUM`, `COUNT`, `MIN`, `MAX`, and `AVG`. 

We will continue working with the `WEATHER_MONITOR` table and summarize records using aggregate functions. 

## Aggregate Functions and GROUP BY 

Let's take a look at three fields in the `WEATHER_MONITOR` table. 

```sql
SELECT REPORT_CODE, REPORT_DATE, RAIN
FROM WEATHER_MONITOR;

```

Let's say we wanted to find the total `RAIN` across the entire table. If we remove the `REPORT_CODE` and `REPORT_DATE` fields, and put the `SUM()` around `RAIN`, observe what happens.

```sql
SELECT SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR;

```

So we have 1720.78 inches of rain total across the whole table. Let's break up that `TOTAL_RAIN` by `LOCATION_ID`. We can achieve this by selecting the `LOCATION_ID` and performing a `GROUP BY` on it.

```sql
SELECT LOCATION_ID, SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY LOCATION_ID;

```

Note how we have sums broken out by `LOCATION_ID` now, or in other words we rolled up that `TOTAL_RAIN` by `LOCATION_ID`. If we wanted to get the total by `LOCATION_ID` and `YEAR`, we can break it up by those two fields/expressions.

```sql
SELECT 
LOCATION_ID, 
strftime('%Y', REPORT_DATE) AS YEAR, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY LOCATION_ID, YEAR;

```

Note also we can use `GROUP BY` with an ordinal index for each selected column/expression rather than the column name. Note this uses 1-based indexing.

```sql
SELECT 
LOCATION_ID, 
strftime('%Y', REPORT_DATE) AS YEAR, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY 1, 2;

```

There are other aggregation functions besides `SUM()`. `MIN()` will find the minimum value for a given column while `MAX()` will find the maximum. `AVG()` will calculate the average column while `COUNT()` will count the number of non-null values for that column. Here are all five of these aggregate functions to create a report summarizing descriptive rain statistics by `LOCATION_ID` and `YEAR`.

```sql
SELECT 
LOCATION_ID, 
strftime('%Y', REPORT_DATE) AS YEAR, 
SUM(RAIN) AS TOTAL_RAIN, 
MIN(RAIN) AS MIN_RAIN,
MAX(RAIN) AS MAX_RAIN,
AVG(RAIN) AS AVG_RAIN, 
COUNT(RAIN) AS COUNT_RAIN
FROM WEATHER_MONITOR 
GROUP BY LOCATION_ID, YEAR;

```

We can also use a `WHERE` filter to only allow certain records to qualify in our aggregations. Below we calculate the total `RAIN` by `YEAR` and `LOCATION_ID`, but only where a `TORNADO` was present.

```sql
SELECT 
LOCATION_ID, 
strftime('%Y', REPORT_DATE) AS YEAR, 
SUM(RAIN) AS TOTAL_TORNADO_RAIN
FROM WEATHER_MONITOR 
WHERE TORNADO = 1
GROUP BY LOCATION_ID, YEAR;

```

## Counting Records

If you want to count the number of records in a table, pass the whole record to the `COUNT()` function rather than a specific field. This can be achieved with using an asterisk `*`.

```sql
SELECT COUNT(*) AS RECORD_COUNT
FROM WEATHER_MONITOR;

```

All the other operations we used previously to slice and filter records can also be used with the `COUNT(*)`. Below we break up the record count by `YEAR`, but only count records where `RAIN` was at least 2 inches.

```sql
SELECT 
strftime('%Y', REPORT_DATE) AS YEAR, 
COUNT(*) AS RECORD_COUNT
FROM WEATHER_MONITOR 
WHERE RAIN >= 2
GROUP BY YEAR; 

```

## Sorting

Let's take a look at the query below showing the `TOTAL_RAIN` by `YEAR` and `MONTH`.

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY YEAR, MONTH;

```

Notice that the records coincidentally are ordered by `YEAR` ascending and `MONTH` ascending. You should never expect records to come back in any order without an `ORDER BY`, even if the SQL engine has an implementation that gives this impression. This can happen especially if the data is physically stored sorted (e.g. chronologically).

To enforce an ascending order by `YEAR` and `MONTH`, add an `ORDER BY` operator.

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY YEAR, MONTH
ORDER BY YEAR, MONTH;

```

You can also reference the selected expressions using ordinal index.

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY 1, 2
ORDER BY 1, 2;

```

If we wanted to have the most recent years displayed first, add the `DESC` keyword to make a given field sort in descending order.

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY YEAR, MONTH
ORDER BY YEAR DESC, MONTH;

```

## EXERCISE

Complete the query below to find the total, minimum, and maximum snowfall by year. Order on the year descending so the latest year is on the top.

```sql
SELECT 
strftime('%Y', REPORT_DATE) AS YEAR, 
? AS TOTAL_SNOW, 
? AS MIN_SNOW,
? AS MAX_SNOW
FROM WEATHER_MONITOR 
? BY ?
? BY ? DESC;

```

### SCROLL DOWN FOR ANSWER
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
|<br>
v 

```sql
SELECT 
strftime('%Y', REPORT_DATE) AS YEAR, 
SUM(SNOW) AS TOTAL_SNOW, 
MIN(SNOW) AS MIN_SNOW,
MAX(SNOW) AS MAX_SNOW
FROM WEATHER_MONITOR 
GROUP BY YEAR
ORDER BY YEAR DESC;
```
