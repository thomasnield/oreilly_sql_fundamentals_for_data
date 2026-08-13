# Subqueries and Common Table Expressions

Hopefully by now you are somewhat comfortable with joins, particularly the `INNER JOIN` and `LEFT JOIN`. While joins are a core skill, SQL becomes incredibly flexible and powerful when you learn subqueries, derived tables, and common table expressions. By using these operations you may discover SQL can declaratively express logic and tasks you might previously have thought not possible.

## Scalar Subqueries

Let's find the maximum `ORDER_DATE` that exists in the `CUSTOMER_ORDER` table.

```sql
SELECT MAX(ORDER_DATE) FROM CUSTOMER_ORDER
```

Now let's say we want to get all the `CUSTOMER_ORDER` records for that latest `ORDER_DATE`. Instead of hard-coding that value as a literal, we can embed that first query as a **subquery** which is a query within a query. In this case, it is a **scalar subquery** because it returns a single value.

```sql
SELECT * FROM CUSTOMER_ORDER
WHERE ORDER_DATE = (SELECT MAX(ORDER_DATE) FROM CUSTOMER_ORDER)
```

## Array Subqueries

Let's say you wanted to get all `CUSTOMER_ORDER` records for customers that are in the `STATE` of "TX". We could achieve this using an `INNER JOIN` but let's try doing this in a slightly simpler (and possibly more efficient) way.

First let's get a single column of `CUSTOMER_ID` for customers in the `STATE` of "TX".

```sql
SELECT CUSTOMER_ID FROM CUSTOMER WHERE STATE = 'TX'
```

We can then embed this as a subquery into an `IN` operation. This is known as an **array subquery** because it returns a list of values.

```sql
SELECT * FROM CUSTOMER_ORDER

WHERE CUSTOMER_ID IN (SELECT CUSTOMER_ID FROM CUSTOMER WHERE STATE = 'TX')
```

Another common task for array subqueries is getting parent records without any children, such as `CUSTOMER` records without any `CUSTOMER_ORDER` records. We can qualify a `DISTINCT` set of `CUSTOMER_ID` values from the `CUSTOMER_ORDER` table (removing any duplicates) and check for `CUSTOMER` records whose `CUSTOMER_ID` is not present.

In this case, we should only get one `CUSTOMER` that does not have any `CUSTOMER_ORDER` records.

```sql
SELECT * FROM CUSTOMER

WHERE CUSTOMER_ID NOT IN (SELECT DISTINCT CUSTOMER_ID FROM CUSTOMER_ORDER)
```

## Correlated Subqueries

We can use **correlated subqueries** to have a subquery reference the outer query's fields. For example, we can show the `CUSTOMER_ORDER` records but also calculate the average quantity ordered for all records sharing that record's `CUSTOMER_ID` and `PRODUCT_ID`. Note that `CUSTOMER_ORDER` is being used in two contexts: the inner query aliased as `co2` and the outer query aliased as `co1`.

```sql
SELECT CUSTOMER_ORDER_ID, 
CUSTOMER_ID,
ORDER_DATE,
PRODUCT_ID,
QUANTITY,

(
  SELECT AVG(QUANTITY)
  FROM CUSTOMER_ORDER co2 
  WHERE co1.CUSTOMER_ID = co2.CUSTOMER_ID 
  AND co1.PRODUCT_ID = co2.PRODUCT_ID
) AS AVG_QTY

FROM CUSTOMER_ORDER co1
```

This is not the most efficient way to do this task by any means, and we will learn some better ways to do this particular task of grabbing an aggregate value sharing each given records' attributes. But correlated subqueries can be a powerful tool to flexibly calculate other queries dependent on each record. Just note this is computationally expensive as every record will kick off this subquery.

## Derived Tables

When a subquery contains multiple columns, we call it a **derived table**. This is often used to declare a `SELECT` query and join it as if it were a table. Observe below how we can show the average quantity ordered alongside each `CUSTOMER_ORDER` record, for all records sharing each record's `CUSTOMER_ID` and `PRODUCT_ID`.

```sql
SELECT CUSTOMER_ORDER_ID, 
CUSTOMER_ORDER.CUSTOMER_ID,
ORDER_DATE,
CUSTOMER_ORDER.PRODUCT_ID,
QUANTITY,
AVG_QTY

FROM CUSTOMER_ORDER LEFT JOIN 

(
  SELECT CUSTOMER_ID, 
  PRODUCT_ID,
  AVG(QUANTITY) AS AVG_QTY

  FROM CUSTOMER_ORDER
  GROUP BY CUSTOMER_ID, PRODUCT_ID
) avg_quantity

ON CUSTOMER_ORDER.CUSTOMER_ID = avg_quantity.CUSTOMER_ID
AND CUSTOMER_ORDER.PRODUCT_ID = avg_quantity.PRODUCT_ID
```

This is much more efficient as we calculate the average `QUANTITY` for each `PRODUCT_ID` and `CUSTOMER_ID` all at once and join to it.

## Common Table Expressions (CTEs)

A much better way to declare derived tables is to instead declare a **common table expression**, which allows you to declare a named subquery in advance prior to using it in a `SELECT` query. Let's take the previous example and observe it in a common table expression.

```sql
WITH avg_quantity AS (
  SELECT CUSTOMER_ID, 
  PRODUCT_ID,
  AVG(QUANTITY) AS AVG_QTY

  FROM CUSTOMER_ORDER
  GROUP BY CUSTOMER_ID, PRODUCT_ID
)

SELECT CUSTOMER_ORDER_ID, 
CUSTOMER_ORDER.CUSTOMER_ID,
ORDER_DATE,
CUSTOMER_ORDER.PRODUCT_ID,
QUANTITY,
AVG_QTY

FROM CUSTOMER_ORDER LEFT JOIN avg_quantity

ON CUSTOMER_ORDER.CUSTOMER_ID = avg_quantity.CUSTOMER_ID
AND CUSTOMER_ORDER.PRODUCT_ID = avg_quantity.PRODUCT_ID
```

The benefits are primarily reusability and code organization. We can use a common table expression multiple times without having to re-declare its `SELECT` query redundantly. We can also avoid messy nesting of `SELECT` queries inside `SELECT` queries, and break up the query into digestible steps.

Another benefit is you can procedurally have multiple common table expressions, where each one can point to the previous. Below we declare the `tx_customer_ids` to get customer IDs in the state of `TX`, and then use that to get orders for only customers in the state of `TX`. Finally we whittle down those orders for only `PRODUCT_ID` of 7.

```sql
WITH tx_customer_ids AS (
  SELECT CUSTOMER_ID 
  FROM CUSTOMER
  WHERE STATE = 'TX'
), 

tx_customer_orders AS (
  SELECT * FROM CUSTOMER_ORDER 
  WHERE CUSTOMER_ID IN tx_customer_ids
)

SELECT * FROM tx_customer_orders
WHERE PRODUCT_ID = 7
```

While this example might unnecessarily break up these steps, this is to show you can break up more complex queries into simple steps.

## Exercise

For each `CUSTOMER_ORDER` in the month of March, retrieve all the fields. Also bring in the minimum and maximum product quantities ordered across all records sharing each record's `PRODUCT_ID` and `CUSTOMER_ID`.

```sql
? min_max_quantity AS (
  ?
)

SELECT CUSTOMER_ORDER_ID, 
CUSTOMER_ORDER.CUSTOMER_ID,
ORDER_DATE,
CUSTOMER_ORDER.PRODUCT_ID,
QUANTITY,
MIN_QTY,
MAX_QTY

FROM CUSTOMER_ORDER LEFT JOIN ?

ON CUSTOMER_ORDER.CUSTOMER_ID = min_max_quantity.CUSTOMER_ID
AND CUSTOMER_ORDER.PRODUCT_ID = min_max_quantity.PRODUCT_ID
```

### SCROLL DOWN FOR ANSWER
|<br>
|<br>
|<br>
|<br>
|<br>
v 


### Answer

```sql
WITH min_max_quantity AS (
  SELECT CUSTOMER_ID, 
  PRODUCT_ID,
  MIN(QUANTITY) AS MIN_QTY,
  MAX(QUANTITY) AS MAX_QTY

  FROM CUSTOMER_ORDER
  GROUP BY CUSTOMER_ID, PRODUCT_ID
)

SELECT CUSTOMER_ORDER_ID, 
CUSTOMER_ORDER.CUSTOMER_ID,
ORDER_DATE,
CUSTOMER_ORDER.PRODUCT_ID,
QUANTITY,
MIN_QTY,
MAX_QTY

FROM CUSTOMER_ORDER LEFT JOIN min_max_quantity

ON CUSTOMER_ORDER.CUSTOMER_ID = min_max_quantity.CUSTOMER_ID
AND CUSTOMER_ORDER.PRODUCT_ID = min_max_quantity.PRODUCT_ID
```