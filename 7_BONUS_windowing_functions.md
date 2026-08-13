
# Section V - Windowing

Windowing functions allow you to greater contextual aggregations in ways much more flexible than GROUP BY. Many major database platforms support windowing functions.


## PARTITION BY


Sometimes it can be helpful to create a contextual aggregation for each record in a query. Windowing functions can make this much easier and save us a lot of subquery work.

For instance, it may be helpful to not only get each CUSTOMER_ORDER for the month of MARCH, but also the average quantity that customer purchased for that `PRODUCT_ID`. We can do that with an ` OVER PARTITION BY` combined with the `AVG()` function. We have done this operation with derived tables and common table expressions in the past, but now we can do it in a one-line window function!

```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
AVG(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID) as AVG_PRODUCT_QTY_ORDERED

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```

Here is the maximum quantity for shared customers and products:

```sql

SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
MAX(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID) as MAX_PRODUCT_QTY_ORDERED

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```

Each `MAX_PRODUCT_QTY_ORDERED` will only be the minimum `QUANTITY` of that given record's `PRDOUCT_ID` and `CUSTOMER_ID`. The `WHERE` will also filter that scope to only within `MARCH`.


You can have multiple windowed fields in a query. Below, we get a MIN, MAX, and AVG for that given window.

```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
MIN(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID) as MIN_PRODUCT_QTY_ORDERED,
MAX(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID) as MAX_PRODUCT_QTY_ORDERED,
AVG(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID) as AVG_PRODUCT_QTY_ORDERED

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```

You can also mix and match scopes which is difficult to do with derived tables.

```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
MIN(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID) as MIN_PRODUCT_CUSTOMER_QTY_ORDERED,
MIN(QUANTITY) OVER(PARTITION BY PRODUCT_ID) as MIN_PRODUCT_QTY_ORDERED,
MIN(QUANTITY) OVER(PARTITION BY CUSTOMER_ID) as MIN_CUSTOMER_QTY_ORDERED


FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'
```

When you are declaring your window redundantly, you can reuse it using a `WINDOW` declaration, which goes between the `WHERE` and the `ORDER BY`.

```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
MIN(QUANTITY) OVER(w) as MIN_PRODUCT_QTY_ORDERED,
MAX(QUANTITY) OVER(w) as MAX_PRODUCT_QTY_ORDERED,
AVG(QUANTITY) OVER(w) as AVG_PRODUCT_QTY_ORDERED

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

WINDOW w AS (PARTITION BY PRODUCT_ID, CUSTOMER_ID)

ORDER BY CUSTOMER_ORDER_ID
```

##  ORDER BY

You can also use an `ORDER BY` in your window to only consider values that comparatively come before that record.

####  USING ORDER BY

For instance, you can get a ROLLING_TOTAL of the QUANTITY by ordering by the ORDER_DATE.


```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
SUM(QUANTITY) OVER (ORDER BY ORDER_DATE) AS ROLLING_QUANTITY

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```

Note you can precede the `ORDER BY` clause with a `DESC` keyword to window in the opposite direction.

####  Ordering and Bounds


Above, notice our example output has the same rolling total for all records on a given date. This is because the ORDER BY in a window function by default does a logical boundary, which in this case is the `ORDER_DATE`. This means it is rolling up everything on that `ORDER_DATE` and previous to it. A side effect is all records with the same `ORDER_DATE` are going to get the same rolling total.

This is the default behavior our query did previously:

```sql

SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
SUM(QUANTITY) OVER (ORDER BY ORDER_DATE RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ROLLING_QUANTITY

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```



If you want to incrementally roll the quantity by each row's physical order (not logical order by the entire `ORDER_DATE`), you can use `ROWS BETWEEN` instead of `RANGE BETWEEN`.


```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
SUM(QUANTITY) OVER (ORDER BY ORDER_DATE ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ROLLING_QUANTITY

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```

Note the `AND CURRENT ROW` is a default, so you can shorthand it like this:

```sql
SUM(QUANTITY) OVER (ORDER BY ORDER_DATE ROWS UNBOUNDED PRECEDING) AS ROLLING_QUANTITY
```

>In this particular example, you could have avoided using a physical boundary by specifying your window with an `ORDER BY CUSTOMER_ORDER_ID`. But we covered the previous strategy anyway to see how to execute physical boundaries. Here is an excellent overview of windowing functions and bounds:
http://mysqlserverteam.com/mysql-8-0-2-introducing-window-functions/

##  Mixing PARTITION BY / ORDER BY

We can combine the PARTITION BY / ORDER BY to create rolling aggregations partitioned on certain fields.

####  Simple MAX over PARITION and ORDER BY

For example, for each record we can get the max quantity ordered up to that date


```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
MAX(QUANTITY) OVER(PARTITION BY PRODUCT_ID, CUSTOMER_ID ORDER BY ORDER_DATE) as MAX_TO_DATE_PRODUCT_QTY

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```



####  Rolling Total Quantity by PRODUCT_ID, with physical boundary

```sql
SELECT CUSTOMER_ORDER_ID,
ORDER_DATE,
CUSTOMER_ID,
PRODUCT_ID,
QUANTITY,
SUM(QUANTITY) OVER(PARTITION BY PRODUCT_ID ORDER BY ORDER_DATE ROWS UNBOUNDED PRECEDING) as total_qty_for_customer_and_product

FROM CUSTOMER_ORDER
WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'
```

You need to be very careful mixing PARITITION BY with an ORDER BY that uses a physical boundary! If you sort the results, it can get confusing very quickly because you lose that physical ordered context.

```sql
SELECT CUSTOMER_ORDER_ID,
ORDER_DATE,
CUSTOMER_ID,
PRODUCT_ID,
QUANTITY,
SUM(QUANTITY) OVER(PARTITION BY PRODUCT_ID ORDER BY ORDER_DATE) as total_qty_for_product

FROM CUSTOMER_ORDER
WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY ORDER_DATE
```

##  Rolling Windows

You can also use movable windows to create moving aggregations. For instance, you can create a six-row rolling average (3 rows before, 3 rows after).


```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,
AVG(QUANTITY) OVER (ORDER BY ORDER_DATE ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING)  AS ROLLING_AVG

FROM CUSTOMER_ORDER

WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```

##  LEAD and LAG

Two highly useful windowing functions are `LEAD()` and `LAG()`, which retrieves the previous or next value within a defined window. 

For example, to simply retrive the previous quantity ordered in a table by `ORDER_DATE`, we can use the `LAG()` function like this. 

```sql 
SELECT *,
LAG (QUANTITY, 1, 0) OVER (ORDER BY ORDER_DATE) AS PREV_QTY
FROM CUSTOMER_ORDER 
```

`LAG()` will retrieve the quantity of the next record on that ordering. 

```sql 
SELECT *,
LEAD (QUANTITY, 1, 0) OVER (ORDER BY ORDER_DATE) AS NEXT_QTY

FROM CUSTOMER_ORDER 
```

You can also use `LEAD()` and `LAG()` with a `PARTIION BY`, making it possible to silo the previous quantity only within records that share each given record's `PRODUCT_ID` and `CUSTOMER_ID`. 

```sql 
SELECT *,
LAG (QUANTITY, 1, 0) OVER (PARTITION BY PRODUCT_ID, CUSTOMER_ID ORDER BY ORDER_DATE) AS PREV_QTY

FROM CUSTOMER_ORDER 

ORDER BY CUSTOMER_ORDER_ID
``` 

#  Ranking with ROW_NUMBER()

The `ROW_NUMBER()` function can be highly helpful with windowing functions to rank items. 

For example, say I wanted to get the top 3 selling PRODUCTs by CUSTOMER. I can use a `ROW_NUMBER()` function to assign a ranking number to each sorted quantity by `CUSTOMER_ID` and `PRODUCT_ID`. Then I can filter for only the first three items. 


```sql 
WITH TOTAL_QTYS AS (
  SELECT CUSTOMER_ID, PRODUCT_ID, SUM(QUANTITY) AS TOTAL_QTY 
  FROM CUSTOMER_ORDER 
  GROUP BY 1,2
),

PRODUCT_SALES_BY_CUSTOMER AS (
   SELECT CUSTOMER_ID, PRODUCT_ID, TOTAL_QTY,
   ROW_NUMBER() OVER (PARTITION BY CUSTOMER_ID ORDER BY TOTAL_QTY DESC) AS RANKING
   FROM TOTAL_QTYS
) 
SELECT * FROM PRODUCT_SALES_BY_CUSTOMER 
WHERE RANKING <= 3
```

Note if you want identical values to receive the same ranking, use the `RANK()` function instead of `row_number()`. Use `DENSE_RANK()` if you want to force the values to be consecutive rather than dupes causing ranks to be skipped. 


# EXERCISE

For the month of March, bring in the rolling sum of quantity ordered (up to each `ORDER_DATE`) by `CUSTOMER_ID` and `PRODUCT_ID`.

```sql
SELECT CUSTOMER_ORDER_ID,
ORDER_DATE,
CUSTOMER_ID,
PRODUCT_ID,
QUANTITY,
SUM(QUANTITY) OVER(PARTITION BY CUSTOMER_ID, PRODUCT_ID ORDER BY ORDER_DATE) as total_qty_for_customer_and_product

FROM CUSTOMER_ORDER
WHERE ORDER_DATE BETWEEN '2017-03-01' AND '2017-03-31'

ORDER BY CUSTOMER_ORDER_ID
```