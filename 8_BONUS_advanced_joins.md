#  Advanced Joins and Temporary Tables

## Creating a Volatile Table

Herei show to create a volatile/temporary table of discount rules. This table will dispose at the end of each session. It is no different than a standard `CREATE TABLE` statement other than the `TEMP` keyword.


```sql
CREATE TEMP TABLE DISCOUNT (
    CUSTOMER_ID_REGEX   VARCHAR (20) NOT NULL DEFAULT ('.*'),
    PRODUCT_ID_REGEX    VARCHAR (20) NOT NULL DEFAULT ('.*'),
    PRODUCT_GROUP_REGEX VARCHAR (30) NOT NULL DEFAULT ('.*'),
    STATE_REGEX         VARCHAR (30) NOT NULL DEFAULT ('.*'),
    DISCOUNT_RATE     DOUBLE       NOT NULL
);

INSERT INTO DISCOUNT (STATE_REGEX, DISCOUNT_RATE) VALUES ('LA|OK', 0.20);
INSERT INTO DISCOUNT (PRODUCT_GROUP_REGEX, STATE_REGEX, DISCOUNT_RATE) VALUES ('BETA|GAMMA','TX', 0.10);
INSERT INTO DISCOUNT (PRODUCT_ID_REGEX, CUSTOMER_ID_REGEX, DISCOUNT_RATE) VALUES ('^[379]$', '^(1|6|12)$', 0.30);
```


Note you can also create a temporary (or permanent) table from a SELECT query. This is helpful to persist expensive query results and reuse it multiple times during a session. SQLite is a bit more convoluted to do this [than other platforms](https://www.techonthenet.com/sql/tables/create_table2.php):

```sql
CREATE TEMP TABLE ORDER_TOTALS_BY_DATE AS
WITH ORDER_TOTALS_BY_DATE AS (
    SELECT ORDER_DATE,
    SUM(QUANTITY) AS TOTAL_QUANTITY
    FROM CUSTOMER_ORDER
    GROUP BY 1
)
SELECT * FROM ORDER_TOTALS_BY_DATE
```



## Joining with Regular Expressions

Left-joining to the temporary table and qualifying on the regular expressions for each respective field allows us to apply the discounts to each `CUSTOMER_ORDER` as specified.

```sql
SELECT CUSTOMER_ORDER.*,
PRICE,
DISCOUNT_RATE,
PRICE * (1 - DISCOUNT_RATE) AS DISCOUNTED_PRICE

FROM CUSTOMER_ORDER
INNER JOIN CUSTOMER
ON CUSTOMER_ORDER.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID

INNER JOIN PRODUCT
ON CUSTOMER_ORDER.PRODUCT_ID = PRODUCT.PRODUCT_ID

LEFT JOIN DISCOUNT
ON CUSTOMER_ORDER.CUSTOMER_ID REGEXP DISCOUNT.CUSTOMER_ID_REGEX
AND CUSTOMER_ORDER.PRODUCT_ID REGEXP DISCOUNT.PRODUCT_ID_REGEX
AND PRODUCT.PRODUCT_GROUP REGEXP DISCOUNT.PRODUCT_GROUP_REGEX
AND CUSTOMER.STATE REGEXP DISCOUNT.STATE_REGEX

WHERE ORDER_DATE BETWEEN '2021-03-26' AND '2021-03-31'
```

If you expect records to possibly get multiple discounts, then sum the discounts and `GROUP BY` everything else:

```sql
SELECT CUSTOMER_ORDER_ID,
CUSTOMER_NAME,
STATE,
ORDER_DATE,
CUSTOMER_ORDER.PRODUCT_ID,
PRODUCT_NAME,
PRODUCT_GROUP
QUANTITY,
PRICE,
SUM(DISCOUNT_RATE) as TOTAL_DISCOUNT_RATE,
PRICE * (1 - SUM(DISCOUNT_RATE)) AS DISCOUNTED_PRICE

FROM CUSTOMER_ORDER
INNER JOIN CUSTOMER
ON CUSTOMER_ORDER.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID

INNER JOIN PRODUCT
ON CUSTOMER_ORDER.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID

LEFT JOIN DISCOUNT
ON CUSTOMER_ORDER.CUSTOMER_ID REGEXP DISCOUNT.CUSTOMER_ID_REGEX
AND CUSTOMER_ORDER.PRODUCT_ID REGEXP DISCOUNT.PRODUCT_ID_REGEX
AND PRODUCT.PRODUCT_GROUP REGEXP DISCOUNT.PRODUCT_GROUP_REGEX
AND CUSTOMER.STATE REGEXP DISCOUNT.STATE_REGEX

WHERE ORDER_DATE BETWEEN '2021-03-26' AND '2021-03-31'

GROUP BY 1,2,3,4,5,6,7,8
```


## Self Joins

We can join a table to itself by invoking it twice with two aliases. This can be useful, for example, to look up the previous day's order quantity (if any) for a given `CUSTOMER_ID` and `PRODUCT_ID`:

```sql

SELECT o1.CUSTOMER_ORDER_ID,
o1.CUSTOMER_ID,
o1.PRODUCT_ID,
o1.ORDER_DATE,
o1.QUANTITY,
o2.QUANTITY AS PREV_DAY_QUANTITY

FROM CUSTOMER_ORDER o1
LEFT JOIN CUSTOMER_ORDER o2

ON o1.CUSTOMER_ID = o2.CUSTOMER_ID
AND o1.PRODUCT_ID = o2.PRODUCT_ID
AND o2.ORDER_DATE = date(o1.ORDER_DATE, '-1 day')

WHERE o1.ORDER_DATE BETWEEN '2021-03-05' AND '2021-03-11'
```

Note if you want to get the previous quantity ordered for that record's given `CUSTOMER_ID` and `PRODUCT_ID`, even if it wasn't strictly the day before, you can use a subquery instead that qualifies previous dates and orders them descending. Then you can use `LIMIT 1` to grab the most recent at the top.


```sql
SELECT ORDER_DATE,
PRODUCT_ID,
CUSTOMER_ID,
QUANTITY,
(
    SELECT QUANTITY
    FROM CUSTOMER_ORDER c2
    WHERE c1.ORDER_DATE > c2.ORDER_DATE
    AND c1.PRODUCT_ID = c2.PRODUCT_ID
    AND c1.CUSTOMER_ID = c2.CUSTOMER_ID
    ORDER BY ORDER_DATE DESC
    LIMIT 1
) as PREV_QTY
FROM CUSTOMER_ORDER c1
```


## Recursive Self Joins

At some point of your career, you may encounter a table that is inherently designed to be self-joined. For instance, run this query: 

```sql
SELECT * FROM EMPLOYEE
```

This is a table containing employee information, including their manager via a `MANAGER_ID` field. Here is a sample of the results below. 

| ID | FIRST_NAME | LAST_NAME  | TITLE               | DEPARTMENT  | MANAGER_ID | 
|----|------------|------------|---------------------|-------------|------------| 
| 13 | Pembroke   | Killgus    | Accountant I        | Accounting  | 10         | 
| 14 | Harper     | Argontt    | Director            | Operations  | 3          | 
| 15 | Fabio      | Treversh   | Manager             | Operations  | 14         | 
| 16 | Gerard     | Morforth   | Analyst             | Operations  | 15         | 
| 17 | Stephanus  | Palatino   | Senior Analyst      | Operations  | 15         | 
| 18 | Jennilee   | Withers    | Analyst             | Operations  | 15         | 
| 19 | Desdemona  | Farmar     | Business Consultant | Operations  | 15         | 
| 20 | Ashlin     | Creamen    | Manager             | Operations  | 14         | 
| 21 | Daniel     | Licquorish | Analyst             | Operations  | 20         | 

This `MANAGER_ID` points to another `EMPLOYEE` record. If you want to bring in Daniel and his superior's information, this isn't hard to do with a self join. 

```sql
SELECT e1.FIRST_NAME, 
e1.LAST_NAME, 
e1.TITLE,
e2.FIRST_NAME AS MANAGER_FIRST_NAME,
e2.LAST_NAME AS MANAGER_LAST_NAME

FROM EMPLOYEE e1 INNER JOIN EMPLOYEE e2
ON e1.MANAGER_ID = e2.ID

WHERE e1.FIRST_NAME = 'Daniel'
```

| FIRST_NAME | LAST_NAME  | TITLE   | MANAGER_FIRST_NAME | MANAGER_LAST_NAME | 
|------------|------------|---------|--------------------|-------------------| 
| Daniel     | Licquorish | Analyst | Ashlin             | Creamen           | 


But what if you wanted to display the entire hierarchy above Daniel? Well shoot, this is hard because now you have to do several self joins to daisy-chain your way to the top. What makes this even harder is you don't know how many self joins you will need to do. For cases like this, it can be helpful to leverage recursive queries. 

A recursion is a special type of common table expression (CTE). Typically, you "seed" a starting value and then use `UNION` or `UNION ALL` to append the results of a query that uses each "seed", and the result becomes the next seed. 

In this case, we will use a `RECURSIVE` common table expression to seed Daniel's ID, and then append each `MANAGER_ID` of each `EMPLOYEE_ID` that matches the seed. This will give a set of ID's for employees hierarchical to Daniel. We can then use these ID's to navigate Daniel's hierarchy via JOINS, IN, or other SQL operators. 

```sql
-- generates a list of employee ID's hierarchical to Daniel

WITH RECURSIVE hierarchy_of_daniel(x) AS (
 SELECT 21 -- start with Daniel's ID
 UNION ALL -- append each manager ID recursively
 SELECT MANAGER_ID 
 FROM hierarchy_of_daniel INNER JOIN EMPLOYEE
 ON EMPLOYEE.ID = hierarchy_of_daniel.x -- employee ID must equal previous recursion
)

SELECT * FROM EMPLOYEE
WHERE ID IN hierarchy_of_daniel;
```

Recursive queries are a bit tricky to get right, but practice them if you deal frequently with hierarchical records. You will likely use them with a specific part of the hierarchy in focus (e.g. Daniel's superiors). It's harder to show the hierarchy for everyone at once, but there are ways. For instance, you can put a RECURSIVE operation in a subquery and use `GROUP_CONCAT`. 

```sql
SELECT e1.* , 

(
    WITH RECURSIVE hierarchy_of(x) AS (
     SELECT e1.ID 
     UNION ALL -- append each manager ID recursively
     SELECT MANAGER_ID 
     FROM hierarchy_of INNER JOIN EMPLOYEE
     ON EMPLOYEE.ID = hierarchy_of.x -- employee ID must equal previous recursion
     )

    SELECT GROUP_CONCAT(ID) FROM EMPLOYEE e2
    WHERE ID IN hierarchy_of
) AS HIERARCHY_IDS

FROM EMPLOYEE e1
```

| ID | FIRST_NAME | LAST_NAME  | TITLE               | DEPARTMENT  | MANAGER_ID | HIERARCHY_IDS| 
|----|------------|------------|---------------------|-------------|------------|--------------| 
| 14 | Harper     | Argontt    | Director            | Operations  | 3          | 1,3,14       | 
| 15 | Fabio      | Treversh   | Manager             | Operations  | 14         | 1,3,14,15    | 
| 16 | Gerard     | Morforth   | Analyst             | Operations  | 15         | 1,3,14,15,16 | 
| 17 | Stephanus  | Palatino   | Senior Analyst      | Operations  | 15         | 1,3,14,15,17 | 
| 18 | Jennilee   | Withers    | Analyst             | Operations  | 15         | 1,3,14,15,18 | 
| 19 | Desdemona  | Farmar     | Business Consultant | Operations  | 15         | 1,3,14,15,19 | 
| 20 | Ashlin     | Creamen    | Manager             | Operations  | 14         | 1,3,14,20    | 
| 21 | Daniel     | Licquorish | Analyst             | Operations  | 20         | 1,3,14,20,21 | 
| 22 | Quill      | Pinder     | Senior Analyst      | Operations  | 20         | 1,3,14,20,22 | 
| 23 | Maybelle   | Freiburger | Business Consultant | Operations  | 20         | 1,3,14,20,23 | 
| 24 | Angelique  | Havis      | Business Consultant | Operations  | 20         | 1,3,14,20,24 | 
| 25 | Lyn        | Geale      | Director            | Technology  | 4          | 1,4,25       | 
| 26 | Tammy      | Eakly      | Manager             | Help Desk   | 25         | 1,4,25,26    | 
| 27 | Junie      | Blanque    | Technician I        | Help Desk   | 26         | 1,4,25,26,27 | 




Note recursive queries also can be used to improvise a set of consecutive values without creating a table. For instance, we can generate a set of consecutive integers. Here is how you create a set of integers from 1 to 1000. 

```sql
WITH RECURSIVE my_integers(x) AS (
    SELECT 1
        UNION ALL
    SELECT x + 1 
    FROM my_integers
    WHERE x < 1000
)
SELECT * FROM my_integers
```

Generating integers can also be be helpful to "repeat-and-modify" records in a given table. For example, say we have a table of air travel bookings where each booking can have "x" number of passengers (such as 3 passengers). 

| BOOKING_ID | BOOKED_EMPLOYEE_ID | DEPARTURE_DATE | ORIGIN | DESTINATION | FARE_PRICE | NUM_OF_PASSENGERS | RETURN_BOOKING_ID |
|------------|--------------------|----------------|--------|-------------|------------|-------------------|-------------------|
| 1          | 6                  | 2021-03-01     | DFW    | ORD         | 170        | 2                 | 2                 |
| 2          | 6                  | 2021-03-04     | ORD    | DFW         | 160        | 2                 |                   |
| 3          | 19                 | 2021-03-21     | DFW    | JFK         | 210        | 3                 | 4                 |
| 4          | 19                 | 2021-03-24     | JFK    | DFW         | 220        | 3                 |                   |
| 5          | 1                  | 2021-03-26     | DFW    | LAX         | 180        | 1                 | 6                 |
| 6          | 1                  | 2021-03-27     | LAX    | DFW         | 190        | 1                 |                   |
| 7          | 5                  | 2021-03-27     | DFW    | ORD         | 210        | 2                 | 8                 |
| 8          | 5                  | 2021-03-29     | ORD    | DFW         | 190        | 2                 |                   |
| 9          | 9                  | 2021-03-28     | DFW    | SFO         | 220        | 3                 | 10                |
| 10         | 9                  | 2021-03-28     | SFO    | DFW         | 230        | 3                 |                   |
| 11         | 31                 | 2021-04-01     | DFW    | LAX         | 190        | 1                 | 12                |
| 12         | 31                 | 2021-04-05     | LAX    | DFW         | 180        | 1                 |                   |

We can break up each booking into individual bookings for each passenger (e.g create 3 records off of a booking with 3 passengers). 

```sql 
WITH RECURSIVE repeat_helper(x) AS (
    SELECT 1
        UNION ALL
    SELECT x + 1 
    FROM repeat_helper
    WHERE x < 1000
)

SELECT BOOKING_ID, 
BOOKED_EMPLOYEE_ID,
DEPARTURE_DATE,
ORIGIN,
DESTINATION,
FARE_PRICE,
repeat_helper.x AS PASSENGER_NUMBER
FROM EMPLOYEE_AIR_TRAVEL CROSS JOIN repeat_helper
ON repeat_helper.x <= NUM_OF_PASSENGERS
```



| BOOKING_ID | BOOKED_EMPLOYEE_ID | DEPARTURE_DATE | ORIGIN | DESTINATION | FARE_PRICE | PASSENGER_NUMBER |
|------------|--------------------|----------------|--------|-------------|------------|------------------|
| 1          | 6                  | 2021-03-01     | DFW    | ORD         | 170        | 1                |
| 1          | 6                  | 2021-03-01     | DFW    | ORD         | 170        | 2                |
| 2          | 6                  | 2021-03-04     | ORD    | DFW         | 160        | 1                |
| 2          | 6                  | 2021-03-04     | ORD    | DFW         | 160        | 2                |
| 3          | 19                 | 2021-03-21     | DFW    | JFK         | 210        | 1                |
| 3          | 19                 | 2021-03-21     | DFW    | JFK         | 210        | 2                |
| 3          | 19                 | 2021-03-21     | DFW    | JFK         | 210        | 3                |
| 4          | 19                 | 2021-03-24     | JFK    | DFW         | 220        | 1                |
| 4          | 19                 | 2021-03-24     | JFK    | DFW         | 220        | 2                |
| 4          | 19                 | 2021-03-24     | JFK    | DFW         | 220        | 3                |
| 5          | 1                  | 2021-03-26     | DFW    | LAX         | 180        | 1                |
| 6          | 1                  | 2021-03-27     | LAX    | DFW         | 190        | 1                |
| 7          | 5                  | 2021-03-27     | DFW    | ORD         | 210        | 1                |

You can also use some clever `CASE` expression logic with an integer generator to find total costs of sending employees to each airport. 

```sql
WITH RECURSIVE repeat_helper(x) AS (
    SELECT 1
        UNION ALL
    SELECT x + 1 
    FROM repeat_helper
    WHERE x < 1000
)

SELECT
CASE WHEN repeat_helper.x == 1 THEN ORIGIN ELSE DESTINATION END AS AIRPORT,
SUM(FARE_PRICE * NUM_OF_PASSENGERS) AS AIRPORT_REVENUE
FROM EMPLOYEE_AIR_TRAVEL CROSS JOIN repeat_helper
ON repeat_helper.x <= 2
GROUP BY AIRPORT
```

| AIRPORT | AIRPORT_REVENUE |
|---------|-----------------|
| DFW     | 4840            |
| JFK     | 1290            |
| LAX     | 740             |
| ORD     | 1460            |
| SFO     | 1350            |


You can apply the same concept to generate a set of chronological dates. This recursive query will generate all dates from today to '2030-12-31':

```sql
WITH RECURSIVE my_dates(x) AS (
    SELECT date('now')
        UNION ALL
    SELECT date(x, '+1 day')
    FROM my_dates
    WHERE x < '2030-12-31'
)
SELECT * FROM my_dates
```

## Cross Joins

Sometimes it can be helpful to generate a "cartesian product", or every possible combination between two or more data sets using a CROSS JOIN. This is often done to generate a data set that fills in gaps for another query. Not every calendar date has orders, nor does every order date have an entry for every product, as shown in this query:

```sql
SELECT ORDER_DATE,
PRODUCT_ID,
SUM(QUANTITY) as TOTAL_QTY

FROM CUSTOMER_ORDER

GROUP BY 1, 2
```

We should use a cross join to resolve this problem. For instance, we can leverage a `CROSS JOIN` query to generate every possible combination of `PRODUCT_ID` and `CUSTOMER_ID`.

```sql
SELECT
CUSTOMER_ID,
PRODUCT_ID
FROM CUSTOMER
CROSS JOIN PRODUCT
```

In this case we should bring in `CALENDAR_DATE` and cross join it with `PRODUCT_ID` to get every possible combination of calendar date and product. Note the `CALENDAR_DATE` comes from the `CALENDAR` table, which acts as a simple list of consecutive calendar dates. Note we could also have used a recursive query, as shown in the previous example, to generate the dates. We'll stick with a simple table instead for now in case you are not comfortable with recursion yet. We should only filter the calendar to a date range of interest, like `2021-01-01` and `2021-03-31`.

```sql
SELECT
CALENDAR_DATE,
PRODUCT_ID
FROM PRODUCT
CROSS JOIN CALENDAR
WHERE CALENDAR_DATE BETWEEN '2021-01-01' and '2021-03-31'
```

Then we can `LEFT JOIN` to our previous query to get every product quantity sold by calendar date, even if there were no orders that day:

```sql
SELECT CALENDAR_DATE,
all_combos.PRODUCT_ID,
TOTAL_QTY

FROM
(
  SELECT
  CALENDAR_DATE,
  PRODUCT_ID
  FROM PRODUCT
  CROSS JOIN CALENDAR
  WHERE CALENDAR_DATE BETWEEN '2021-01-01' and '2021-03-31'
) all_combos

LEFT JOIN
(
  SELECT ORDER_DATE,
  PRODUCT_ID,
  SUM(QUANTITY) as TOTAL_QTY

  FROM CUSTOMER_ORDER

  GROUP BY 1, 2
) totals

ON all_combos.CALENDAR_DATE = totals.ORDER_DATE
AND all_combos.PRODUCT_ID = totals.PRODUCT_ID

ORDER BY CALENDAR_DATE, all_combos.PRODUCT_ID
```

Note you can also use common table expressions:

```sql
WITH all_combos AS (
  SELECT
  CALENDAR_DATE,
  PRODUCT_ID
  FROM PRODUCT
  CROSS JOIN CALENDAR
  WHERE CALENDAR_DATE BETWEEN '2021-01-01' and '2021-03-31'
),

totals AS (
  SELECT ORDER_DATE,
  PRODUCT_ID,
  SUM(QUANTITY) as TOTAL_QTY

  FROM CUSTOMER_ORDER

  GROUP BY 1, 2
)


SELECT CALENDAR_DATE,
all_combos.PRODUCT_ID,
TOTAL_QTY

FROM all_combos LEFT JOIN totals

ON all_combos.CALENDAR_DATE = totals.ORDER_DATE
AND all_combos.PRODUCT_ID = totals.PRODUCT_ID

ORDER BY CALENDAR_DATE, all_combos.PRODUCT_ID
```


## Comparative Joins

Note also you can use comparison operators in joins. For instance, we can self-join to create rolling quantity totals and generate a cartesian product on previous dates to the current order, and then sum those quantities. It is much easier to use windowing functions for this purpose though, which is covered in the next section.

```sql
SELECT c1.ORDER_DATE,
c1.PRODUCT_ID,
c1.CUSTOMER_ID,
c1.QUANTITY,
SUM(c2.QUANTITY) as ROLLING_QTY

FROM CUSTOMER_ORDER c1 INNER JOIN CUSTOMER_ORDER c2
ON c1.PRODUCT_ID = c2.PRODUCT_ID
AND c1.CUSTOMER_ID = c2.CUSTOMER_ID
AND c1.ORDER_DATE >= c2.ORDER_DATE

GROUP BY 1, 2, 3, 4
```


## Exercise 

For every `CALENDAR_DATE` and `CUSTOMER_ID`, show the total `QUANTITY` ordered for the date range of `2021-01-01` to `2021-03-31`:

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

**ANSWER:**

```sql
SELECT CALENDAR_DATE,
all_combos.CUSTOMER_ID,
coalesce(TOTAL_QTY, 0) AS TOTAL_QTY

FROM
(
  SELECT
  CALENDAR_DATE,
  CUSTOMER_ID
  FROM CUSTOMER
  CROSS JOIN CALENDAR
  WHERE CALENDAR_DATE BETWEEN '2021-01-01' and '2021-03-31'
) all_combos

LEFT JOIN
(
  SELECT ORDER_DATE,
  CUSTOMER_ID,
  SUM(QUANTITY) as TOTAL_QTY

  FROM CUSTOMER_ORDER

  GROUP BY 1, 2
) totals

ON all_combos.CALENDAR_DATE = totals.ORDER_DATE
AND all_combos.CUSTOMER_ID = totals.CUSTOMER_ID

ORDER BY CALENDAR_DATE, all_combos.CUSTOMER_ID
```

Using Common Table Expressions:

```sql

WITH all_combos AS (
  SELECT
  CALENDAR_DATE,
  CUSTOMER_ID
  FROM CUSTOMER
  CROSS JOIN CALENDAR
  WHERE CALENDAR_DATE BETWEEN '2021-01-01' and '2021-03-31'
),

totals AS (
  SELECT ORDER_DATE,
  CUSTOMER_ID,
  SUM(QUANTITY) as TOTAL_QTY

  FROM CUSTOMER_ORDER

  GROUP BY 1, 2
)

SELECT CALENDAR_DATE,
all_combos.CUSTOMER_ID,
coalesce(TOTAL_QTY, 0) AS TOTAL_QTY

from all_combos LEFT JOIN totals

ON all_combos.CALENDAR_DATE = totals.ORDER_DATE
AND all_combos.CUSTOMER_ID = totals.CUSTOMER_ID

ORDER BY CALENDAR_DATE, all_combos.CUSTOMER_ID
```
