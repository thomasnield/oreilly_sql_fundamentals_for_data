# Nulls, CASE Expressions, and Coalesce

We will cover `NULL` values and `CASE` expressions in this section. A `NULL` value is no value, indicating a blank value. The `CASE` expression allows us to pair conditions to resulting values. 

We will cover both these operations in SQL. 

## NULL Values

Let's take a look at the `WEATHER_MONITOR` table. Sample these four records. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE REPORT_CODE IN ('LJVE08D', 'EP4AKZR', '1FC27OH', 'F4DEAK3');
```

Note how some columns have values that indicate a `NULL` value. A null value is blank, meaning no value has been provided (not to be confused with `0` which is a value or an empty string `''`). 

Note that SQL databases will have `NULL` for blank values. 

If we have null values for rain, it might indicate that rain recordings were not possible because the instruments were broken. The same goes for the `SNOW` and other fields that are nullable. 

To qualify a null value, use `IS NULL`. Below we find records without a recorded `RAIN` measurement. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE RAIN IS NULL;
```

To qualify records that are not null, qualify with `IS NOT NULL`. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE RAIN IS NOT NULL;
```

Note that if you do not handle `NULL` values explicitly in your `WHERE` condition on a given column, then `NULL` values will always be omitted. For example, if we qualify for records where `RAIN > 0` then the `NULL` values will be omitted. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE RAIN > 0;
```

If you want to include `NULL` values in your condition, explicitly allow for `NULL`. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE RAIN IS NULL OR RAIN > 0;
```

A helpful function to know by heart is `COALESCE()`. It will take a possibly `NULL` value and convert it to a different value if it is indeed `NULL`. Otherwise it will leave the value alone. 

The first argument for `COALESCE()` is the value that might be `NULL`. The second argument is the value to translate it into if it is indeed `NULL`. We can treat all `RAIN` values that are `NULL` as `0` in the `COALESCE()` below. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE COALESCE(RAIN,0) > 0;
```

As another example, to turn missing `RAIN` values into `-1`, we can use the `COALESCE` like this. 

```sql
SELECT REPORT_CODE, 
RAIN, 
COALESCE(RAIN,-1) AS COALESCED_RAIN 
FROM WEATHER_MONITOR 
WHERE REPORT_CODE IN ('G0UINBG', 'PJVNOSP');
```

## CASE Expression

Take a look at the `TEMPERATURE` field in the table. 

```sql
SELECT REPORT_CODE, TEMPERATURE
FROM WEATHER_MONITOR;
```

Let's say we wanted to categorize each temperature as `HOT`, `MILD` or `COLD`. To do this we would have to use a `CASE` expression and attach a condition to each label. Let's demonstrate: 

```sql
SELECT REPORT_CODE, 
TEMPERATURE,
CASE 
  WHEN TEMPERATURE >= 78 THEN 'HOT'
  WHEN TEMPERATURE >= 60 THEN 'MILD'
  ELSE 'COLD'
END AS TEMPERATURE_LABEL
FROM WEATHER_MONITOR;
```

Note how we use a `CASE` to open up the `CASE` expression. Each `WHEN` specifies a condition and `THEN` specifies the resulting value if that condition is true. Each condition is evaluated from top-to-bottom, and the first one found to be true is the one that will be chosen. An `ELSE` can optionally be appended to specify a default value if all the other conditions fail to be met. In this case, we establish any other record as `COLD` since we already deducted it is not `HOT` or `MILD`. 

However, you have to be careful about `NULL` values if they are present in a column. If you use an `ELSE` on the `TEMPERATURE` field, and that field happens to have `NULL` values (there are three), then they will be labelled as `NULL`. A better way to handle the `NULL` values might be to have an explicit condition for `COLD`, and then make the `ELSE` the catch-all for anomalies like `NULL` and label them `N/A`. 

```sql
SELECT REPORT_CODE, 
TEMPERATURE,
CASE 
  WHEN TEMPERATURE >= 78 THEN 'HOT'
  WHEN TEMPERATURE >= 60 THEN 'MILD'
  WHEN TEMPERATURE < 60 THEN 'COLD'
  ELSE 'N/A'
END AS TEMPERATURE_LABEL
FROM WEATHER_MONITOR;
```

With a `CASE` expression, you can now do more interesting aggregations on fields that were not available before. For example, we can get a `COUNT` of the number records broken up by `TEMPERATURE_LABEL`. 

```sql
SELECT 
CASE 
  WHEN TEMPERATURE >= 78 THEN 'HOT'
  WHEN TEMPERATURE >= 60 THEN 'MILD'
  WHEN TEMPERATURE < 60 THEN 'COLD'
  ELSE 'N/A'
END AS TEMPERATURE_LABEL,
COUNT(*) AS RECORD_COUNT
FROM WEATHER_MONITOR
GROUP BY TEMPERATURE_LABEL;
```

As a sidenote, you might have figured out that the `COALESCE` is a shorthand `CASE` expression to convert `NULL` values. Take our previous example showing the coalesced `RAIN` values. 

```sql
SELECT REPORT_CODE, 
RAIN, 
COALESCE(RAIN,-1) AS COALESCED_RAIN 
FROM WEATHER_MONITOR 
WHERE REPORT_CODE IN ('G0UINBG', 'PJVNOSP');
```

We can express this using a `CASE` expression. 

```sql
SELECT REPORT_CODE, 
RAIN, 
CASE WHEN RAIN IS NULL THEN -1 ELSE RAIN END AS COALESCED_RAIN 
FROM WEATHER_MONITOR 
WHERE REPORT_CODE IN ('G0UINBG', 'PJVNOSP');
```

## The NULL Case Trick 

Let's say you calculate the total rain broken up by `YEAR` and `MONTH`, only for the `YEAR` 2021. 

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(RAIN) AS TOTAL_RAIN
FROM WEATHER_MONITOR 
GROUP BY YEAR, MONTH;
```

Now you want to break up that `TOTAL_RAIN` column into two columns, one for when a `TORNADO` was present and one for when there was not. What's the problem here? 

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(RAIN) AS TOTAL_TORNADO_RAIN,
SUM(RAIN) AS TOTAL_NON_TORNADO_RAIN
FROM WEATHER_MONITOR 
WHERE TORNADO = 1 
AND YEAR = 2021
GROUP BY YEAR, MONTH;
```

That `WHERE` condition inconveniently commits you to `TORNADO` being 1 or 0, but not both for each column. But you can get around this using a `CASE` expression and putting the respective conditions there. Observe below how we intercept the values going into each `SUM()` by checking for the `TORNADO` condition, and if it fails then we add a `0` to the `SUM` instead. Clever, right? 

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(CASE WHEN TORNADO = 1 THEN RAIN ELSE 0 END) AS TOTAL_TORNADO_RAIN,
SUM(CASE WHEN TORNADO = 0 THEN RAIN ELSE 0 END) AS TOTAL_NON_TORNADO_RAIN
FROM WEATHER_MONITOR 
WHERE YEAR = 2021 
GROUP BY YEAR, MONTH;
```

However, a `0` for the false condition can be problematic for other aggregation operations like `MIN`, `MAX`, `AVG` and `COUNT` as it will affect those calculations unlike `SUM`. You can instead use `NULL` as it will get ignored by all the aggregation operators, including `SUM`. 

```sql
SELECT 
CAST(strftime('%Y', REPORT_DATE) AS INTEGER) AS YEAR, 
CAST(strftime('%m', REPORT_DATE) AS INTEGER) AS MONTH, 
SUM(CASE WHEN TORNADO = 1 THEN RAIN ELSE NULL END) AS AVG_TORNADO_RAIN,
SUM(CASE WHEN TORNADO = 0 THEN RAIN ELSE NULL END) AS AVG_NON_TORNADO_RAIN
FROM WEATHER_MONITOR 
WHERE YEAR = 2021 
GROUP BY YEAR, MONTH;
```

Few people who are using `SQL` know this trick, and it can save many messy queries and derived tables. Use it liberally! 

## EXERCISE

For each `LOCATION_ID`, calculate the previous year total rain `PY_RAIN` and current year total rain `CY_RAIN`. Replace the question marks `?` and assume 2021 is the current year. 

```sql
SELECT 
LOCATION_ID,
SUM(
  CASE WHEN CAST(strftime('%Y', REPORT_DATE) AS INTEGER) = ? THEN ? ELSE ? END
) AS CY_RAIN,
SUM(
  CASE WHEN CAST(strftime('%Y', REPORT_DATE) AS INTEGER) = ? THEN ? ELSE ? END
) AS PY_RAIN
FROM WEATHER_MONITOR 
WHERE CAST(strftime('%Y', REPORT_DATE) AS INTEGER) IN (2020, 2021)
GROUP BY LOCATION_ID;
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
LOCATION_ID,
SUM(
  CASE WHEN CAST(strftime('%Y', REPORT_DATE) AS INTEGER) = 2021 THEN RAIN ELSE 0 END
) AS CY_RAIN,
SUM(
  CASE WHEN CAST(strftime('%Y', REPORT_DATE) AS INTEGER) = 2020 THEN RAIN ELSE 0 END
) AS PY_RAIN
FROM WEATHER_MONITOR 
WHERE CAST(strftime('%Y', REPORT_DATE) AS INTEGER) IN (2020, 2021)
GROUP BY LOCATION_ID;
```
