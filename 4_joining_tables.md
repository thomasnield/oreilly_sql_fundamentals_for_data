# INNER JOIN and LEFT JOIN  

The most defining feature of SQL is arguably the join, as this is what relational databases are really designed to do. While there are several types of join, none are so commonly used as the `INNER JOIN` and `LEFT JOIN`. We will cover these two operators in this section. 

## Primary and Foreign Keys

Let's take a look at two tables: `CUSTOMER` and `CUSTOMER_ORDER`. What do they have in common? 

```sql
SELECT * FROM CUSTOMER;
```

```sql
SELECT * FROM CUSTOMER_ORDER;
```

The two tables have a `CUSTOMER_ID` column, and you can probably infer what it does for each table. The `CUSTOMER` table has a unique `CUSTOMER_ID` assigned to each customer record. But in the `CUSTOMER_ORDER` table it is used to assign an order to a given `CUSTOMER`, using that `CUSTOMER_ID`. 

This makes the `CUSTOMER` table the **parent table** with the `CUSTOMER_ID` being the **primary key**. The `CUSTOMER_ORDER` table is the **child table** with the `CUSTOMER_ID` being the **foreign key**. You can think of it as the parent table *supplies* data to the child table, via the primary key to the foreign key. 

The primary key cannot have duplicate values, and this makes sense as no two customers should have the same `CUSTOMER_ID`. However, there can be multiple instances of a value in a foreign key column, as a given customer can have multiple orders. This is a classic **one-to-many relationship**. 

These relationships are designed to be joined on, and are a fundamental use case for `INNER JOIN` and `LEFT JOIN`. 

## INNER JOIN

The `INNER JOIN` is the most common type of join in SQL. It stitches together two or more tables on one or more fields. In our example, it would be helpful to make our `CUSTOMER_ORDER` records more descriptive, by bringing in `CUSTOMER` information alongside each `CUSTOMER_ORDER` record. An `INNER JOIN` can achieve this as shown below.

```sql
SELECT 
CUSTOMER_ORDER_ID,
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
ADDRESS,
CITY,
STATE,
ZIP,
ORDER_DATE,
PRODUCT_ID,
QUANTITY
FROM CUSTOMER INNER JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID;
```

Above we have pulled fields from both `CUSTOMER` and `CUSTOMER_ORDER`, and since the `CUSTOMER_ID` exists in both tables we choose one by using the syntax `CUSTOMER.CUSTOMER_ID` to select the one from the `CUSTOMER` table. 

> For ambiguous fields like `CUSTOMER_ID`, it is a good rule of thumb to prefer the one in the parent table in case you switch your `INNER JOIN` to a `LEFT JOIN` so it never becomes `NULL`. We will learn about this later. 

The `INNER JOIN` allows us to stitch both tables together and define the commonality using the `ON` keyword, which is where our join condition goes. You can actually specify any condition you want for a `JOIN`, but 99.9% of the time you will likely use a simple equals `=` to line up records between the two tables. 

Another way to think of this is we are copying each `CUSTOMER` record across each respective `CUSTOMER_ORDER` sharing that respective `CUSTOMER_ID`. 

> Occasionally, you might come across colleagues who simply write `JOIN` rather than `INNER JOIN`. This is a shorthand or alias for an `INNER JOIN`, but it is better practice to explicitly express `INNER JOIN` so you make it clear that was the type of join you intended. 

You also should avoid using an old convention of inner joining by selecting tables in a comma-separated way, and using the `WHERE` condition for your join condition as shown below. This is an [inflexible and messy syntax that is less legible](https://stackoverflow.com/questions/1018822/inner-join-on-vs-where-clause), and I encourage avoiding it. 

```sql
SELECT 
CUSTOMER_ORDER_ID,
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
ADDRESS,
CITY,
STATE,
ZIP,
ORDER_DATE,
PRODUCT_ID,
QUANTITY
FROM CUSTOMER, CUSTOMER_ORDER
WHERE CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID;
```

## LEFT JOIN

What happens if there are `CUSTOMER` records that do not have any `CUSTOMER_ORDER` records? Do they show up in an `INNER JOIN`? For example, "Alpha Medical" with a `CUSTOMER_ID` of 1 does not have any orders. Does it show up in our `INNER JOIN` query? Let's add a `WHERE` condition to find out. 

```sql
SELECT 
CUSTOMER_ORDER_ID,
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
ADDRESS,
CITY,
STATE,
ZIP,
ORDER_DATE,
PRODUCT_ID,
QUANTITY
FROM CUSTOMER INNER JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
WHERE CUSTOMER.CUSTOMER_ID = 1;
```

Sure enough we get an empty result. But look what happens if we change our `INNER JOIN` to a `LEFT JOIN` (or `LEFT OUTER JOIN` which are both aliases for the same operation). 

```sql
SELECT 
CUSTOMER_ORDER_ID,
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
ADDRESS,
CITY,
STATE,
ZIP,
ORDER_DATE,
PRODUCT_ID,
QUANTITY
FROM CUSTOMER LEFT JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
WHERE CUSTOMER.CUSTOMER_ID = 1;
```

Note how "Alpha Medical" now shows up with a placeholder record even though it did not have any `CUSTOMER_ORDER` records. All of its `CUSTOMER_ORDER` fields are `NULL` because there were no `CUSTOMER_ORDER` records to join to and populate this information. But the `LEFT JOIN` did append this one placeholder record for "Alpha Medical".

In other words, the `LEFT JOIN` includes all records in the "left" table even if there are no records to join to in the "right" table. By "left" I mean the table literally specified to the left of the `LEFT JOIN` operator. This means the order you declare the tables in your `FROM` matters with a `LEFT JOIN`. 

> There is also a `RIGHT JOIN` or `RIGHT OUTER JOIN` operator, which flips the direction and includes all records in the `RIGHT` table even if there are none to join to in the `LEFT` table. However it is seldom used as what can be done with a `RIGHT JOIN` can also be achieved with a `LEFT JOIN`. There is also a `FULL OUTER JOIN` which includes all records in both directions, but it also is rarely used. As a matter of fact, SQLite does not support the `RIGHT JOIN` or `FULL OUTER JOIN` for this reason. 

As we will see, this can be useful for creating reports later as we likely want to include customers that do not have any orders. Another common use case for `LEFT JOIN` is finding parent records that do not have any children, such as `CUSTOMER` records that do not have any `CUSTOMER_ORDER` records. We can do this by qualifying any `CUSTOMER_ORDER` fields to be null, which normally are not null but consequently become null as a result of the `LEFT JOIN`. 

```sql
SELECT 
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME
FROM CUSTOMER LEFT JOIN CUSTOMER_ORDER
ON CUSTOMER_ORDER.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
WHERE CUSTOMER_ORDER.CUSTOMER_ID IS NULL;
```

## Joining Multiple Tables

What if we wanted to bring `PRODUCT` information to our `CUSTOMER_ORDER` records as well as `CUSTOMER` information? This is possible by executing a second join. Let's take a look at the `PRODUCT` table and note it uses a `PRODUCT_ID`, which also exists in the `CUSTOMER_ORDER` table as a foreign key.

```sql
SELECT * FROM PRODUCT;
```

Let's bring in the `PRODUCT_NAME` and `PRICE` to show alongside each `CUSTOMER_ORDER`. We can execute a second join on the `PRODUCT_ID` and stitch the `PRODUCT` information table to our existing stitchwork between `CUSTOMER_ORDER` and `CUSTOMER`. 

```sql
SELECT 
CUSTOMER_ORDER_ID,
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
ADDRESS,
CITY,
STATE,
ZIP,
ORDER_DATE,
PRODUCT.PRODUCT_ID,
QUANTITY, 
PRICE
FROM CUSTOMER INNER JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
INNER JOIN PRODUCT
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID;
```

You can mix `INNER JOIN` and `LEFT JOIN` in a query, and you have to reason carefully through these scenarios as this becomes use-case specific. But in this scenario, if we wanted to include all `CUSTOMER` records we need to use a `LEFT JOIN` for both joins including for `PRODUCT`, because the null values from the first `LEFT JOIN` will propagate to the next join. The second `LEFT JOIN` will tolerate these null values but the `INNER JOIN` will not and simply omit them. 

```sql
SELECT 
CUSTOMER_ORDER_ID,
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
ADDRESS,
CITY,
STATE,
ZIP,
ORDER_DATE,
PRODUCT.PRODUCT_ID,
QUANTITY, 
PRICE
FROM CUSTOMER LEFT JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
LEFT JOIN PRODUCT
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID;
```

## Aggregating Joins

If you think of the above queries we ran earlier as "new" tables that were produced by the joins, logically we can apply a `GROUP BY` on them as well as aggregation functions like `SUM()`. If we wanted to find the total revenue by customer, let's add a `PRICE * QUANTITY` expression and name it `REVENUE`. 

```sql
SELECT
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
PRICE * QUANTITY AS REVENUE
FROM CUSTOMER LEFT JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
LEFT JOIN PRODUCT
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID;
```

Then we can `SUM()` that expression and add a `GROUP BY` to roll up the `CUSTOMER` attributes. 

```sql
SELECT
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
SUM(PRICE * QUANTITY) AS TOTAL_REVENUE
FROM CUSTOMER LEFT JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
LEFT JOIN PRODUCT
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID
GROUP BY CUSTOMER.CUSTOMER_ID, CUSTOMER_NAME;
```

Finally, we can `COALESCE()` the `TOTAL_REVENUE` to turn any null values to `0`. And that is it!

```sql
SELECT
CUSTOMER.CUSTOMER_ID, 
CUSTOMER_NAME,
COALESCE(SUM(PRICE * QUANTITY), 0) AS TOTAL_REVENUE
FROM CUSTOMER LEFT JOIN CUSTOMER_ORDER
ON CUSTOMER.CUSTOMER_ID = CUSTOMER_ORDER.CUSTOMER_ID
LEFT JOIN PRODUCT
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID
GROUP BY CUSTOMER.CUSTOMER_ID, CUSTOMER_NAME;
```

And that is it! We have learned the fundamentals of SQL joins. If you get comfortable with this operation, you can call yourself a SQL developer. 

## EXERCISE

Find the total revenue by product by completing the query below, replacing the question marks "?" with the proper SQL.

```sql
SELECT
PRODUCT.PRODUCT_ID, 
PRODUCT_NAME,
COALESCE(SUM(PRICE * QUANTITY), 0) AS TOTAL_REVENUE
FROM ?
GROUP BY PRODUCT.PRODUCT_ID, PRODUCT_NAME;
```

### SCROLL DOWN FOR ANSWER
|<br>
|<br>
|<br>
|<br>
|<br>
v 

```sql
SELECT
PRODUCT.PRODUCT_ID, 
PRODUCT_NAME,
COALESCE(SUM(PRICE * QUANTITY), 0) AS TOTAL_REVENUE
FROM PRODUCT LEFT JOIN CUSTOMER_ORDER
ON PRODUCT.PRODUCT_ID = CUSTOMER_ORDER.PRODUCT_ID
GROUP BY PRODUCT.PRODUCT_ID, PRODUCT_NAME;
```