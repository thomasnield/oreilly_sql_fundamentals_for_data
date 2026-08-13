# Creating Tables and Writing Data

In this last module we will cover writing tasks in SQL, including the creation of tables and data as well as editing existing data. Most SQL usage is reading data, especially if you work in an analytical or data science role. But when you work with moving data from one source to another, you will inevitably need to take on the creation of data in relational databases.

## Planning a Database

If we wanted to create a new database from scratch, we could declare a nonexistent database file in our path and SQLite will create that file. However, we are going to add a new table to an existing database.

When you plan a database, or additions to a database, consider carefully what the table relationships will be and what fields need to be captured. Make sure that the data is simple and normalized, and a table is not trying to be more than one thing. For example, we do not want a table to contain both `CUSTOMER` and `CUSTOMER_ORDER` information as those are two separate entities, so we store them in separate tables.

Note that it is easy to add fields and tables to a database. However it is much harder to modify and remove fields and tables when dependencies between parents and children are involved.

In our use case, we want to create a table called `CUSTOMER_PAYMENT` that tracks whether customers paid for part or all of a `CUSTOMER_ORDER`.

The fields we need to track include:

*   `CUSTOMER_PAYMENT_ID` - Primary key identifier for each record
*   `CUSTOMER_ORDER_ID` - Foreign key tying each payment to an order
*   `RECEIVE_DATE` - Date payment was received
*   `RECEIVE_AMOUNT` - Payment amount received
*   `MEMO` - Any memo text from customer

## Field Attributes in SQL

When creating a table, it is important to be aware of the field attributes of each column. SQLite is fast and loose with data types because it is dynamically typed, and does not enforce a given column to consistently be numbers or text as specified. Other data platforms will.

In SQLite, you can look up the data types for an existing table using a `PRAGMA` command with the `table_info()` function. Below we can see what each datatype is for each column of the `CUSTOMER_ORDER` table. Note the `type` column, the `notnull` indicator, the default value `dflt_value`, and primary key `pk` indicator. These are all attributes we will want to define when we create our own tables.

```sql
PRAGMA table_info(CUSTOMER_ORDER);
```

Here is a quick breakdown of each of these attributes about a table field.

*   **Data Type** (type) - The type of data (e.g. numeric, text, date) stored in a given column

*   **Not Null** (notnull) - Indicates whether the column bans null values.

*   **Default Value** (dflt_value) - The default value to provide a column if no value is provided

*   **Primary Key** (pk) - Indicates whether the column is the primary key.

The data type of a column can be the most involved to thoroughly understand, as each SQL platform will have different flavors of data types to choose from. SQLite streamlines to only store 5 types of data: [NULL, Integer, Real, Text, and Blob](https://www.sqlite.org/datatype3.html). Unlike other SQL platforms, SQLite does not enforce type consistency on a column because it is dynamically typed.

However, SQLite does [alias the 5 types with type affinities](https://www.sqlite.org/datatype3.html) so they compatibly resemble more specific types on other database platforms. Some of these affinity types include:

|TYPE|DESCRIPTION|
|---|---|
|`INTEGER` |A discrete integer that can be negative or positive, with size variants like `TINYINT`, `SMALLINT`, `MEDIUMINT`, etc.|
|`CHAR(X)`|A fixed-width text with a length of `X` characters. |
|`VARCHAR(X)`|A variable-width text with a maximum of `X` characters.|
|`FLOAT`|A floating-point number|
|`DOUBLE`|A double-precision floating-point number|
|`DATE`|A date value type holding a year, month, and day.|
|`TIME`|A time value holding an hour, minute, seconds, fraction of a second.|
|`DATETIME`|Merges a date and time together|

Please note this is just a sampling of data types we will work with in this notebook. Whatever database platform you use, take time to go through its documentation and get familiar with its datatypes so you know which ones to choose.

## Creating and Dropping Tables

To create this table, we use the `CREATE TABLE` command. Note that we follow the command with the name of the table, and then inside parentheses declare each column, its datatype, and its behaviors separated by commas.

The foreign key declaration is done last, pointing the `CUSTOMER_ORDER_ID` to its primary key counterpart in the parent `CUSTOMER_ORDER` table.

```sql
CREATE TABLE CUSTOMER_PAYMENT (
  CUSTOMER_PAYMENT_ID INTEGER PRIMARY KEY NOT NULL,
  CUSTOMER_ORDER_ID INTEGER NOT NULL,
  RECEIVE_DATE DATE NOT NULL DEFAULT (date('now')), 
  RECEIVE_AMOUNT DOUBLE NOT NULL, 
  MEMO VARCHAR(100), 

  FOREIGN KEY (CUSTOMER_ORDER_ID) REFERENCES CUSTOMER_ORDER(CUSTOMER_ORDER_ID)
)
```

While there are no records in this table, we can now run a `SELECT` query against it and see the table is now there.

```sql
SELECT * FROM CUSTOMER_PAYMENT
```

If you ever need to delete a table, use the `DROP TABLE` command. This will only be allowed if no child records are pointed to this table.

If you run this command, be sure to run the `CREATE TABLE` operation above again before proceeding.

```sql
DROP TABLE CUSTOMER_PAYMENT
```

## Writing Records with INSERT

To insert a new record into a table, use the `INSERT` command. At minimum, you only need to provide the fields that have no default or null values. In this case, we only need to provide the `CUSTOMER_ORDER_ID` and the `RECEIVE_AMOUNT`. Note we signal those are the fields that will be provided in parentheses and then those `VALUES` are then provided in a second set of parentheses.

```sql
INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_AMOUNT)
VALUES (1, 550)
```

Now let's select from the `CUSTOMER_PAYMENT` table.

```sql
SELECT * FROM CUSTOMER_PAYMENT
```

The `CUSTOMER_PAYMENT_ID` is an `INTEGER` and a `PRIMARY KEY` so SQLite will automatically assign an incremental integer if one is not provided, starting at the number `1`. As we specified in the `CREATE TABLE` command earlier, the `RECEIVE_DATE` will default to today's date, and the `MEMO` does not have a `NOT NULL` constraint so it defaults to `NULL`.

You can also batch insert several records by providing multiple rows after `VALUES`, separated by commas.

```sql
INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_DATE, RECEIVE_AMOUNT, MEMO) VALUES 
(2, '2020-05-01', 560, 'Thank you again!'), 
(4, '2020-05-05', 430, 'Payment 1 of 2'),
(4, '2020-05-10', 270, 'Payment 2 of 2')
```

Run a `SELECT` query on `CUSTOMER_PAYMENT` and you will see the three additional records added.

```sql
SELECT * FROM CUSTOMER_PAYMENT
```

> Never manually concatenate a SQL string together. If you need to construct a SQL string make sure to use parameterized placeholders and inject values safely rather than building SQL through string concatenation.

## UPDATE and DELETE

To update a record, use the `UPDATE` command followed by the targeted table for the changes. Then use the `SET` keyword to assign one or more fields (separated by commas) to a new value.

Be sure to use a `WHERE` command if you are only targeting specific records. Otherwise it will update every record with those assignment changes. Below we update the `CUSTOMER_PAYMENT` record with `CUSTOMER_PAYMENT_ID` of `2` to have the `RECEIVE_AMOUNT` and `RECEIVE_DATE` values changed.

```sql
UPDATE CUSTOMER_PAYMENT SET RECEIVE_AMOUNT = 580, RECEIVE_DATE = '2020-05-05'
WHERE CUSTOMER_PAYMENT_ID = 2
```

To delete one or more records, use a `DELETE` command with a `WHERE` condition targeting those records. Let's say we want to delete records where no `MEMO` was provided. A good practice is to preview records you want to delete with a `SELECT`.

```sql
SELECT * FROM CUSTOMER_PAYMENT
WHERE MEMO IS NULL 
```

We can then use that `WHERE` condition with a `DELETE` command as demonstrated below.

```sql
DELETE FROM CUSTOMER_PAYMENT
WHERE MEMO IS NULL 
```

View the changes by running this query.

```sql
SELECT * FROM CUSTOMER_PAYMENT
```

## Transactions

When you are making edits to a database, strongly consider doing so within a **transaction** which acts as a rewind button from the point a transaction is started. As a matter of fact, when you start making write operations like we did above, it already opened a transaction that has never been finalized. This means we need to `COMMIT` the changes we just made or else they will be lost the moment the database connection is closed. Let's commit those changes now.

```sql
COMMIT
```

The reason for this is if anything goes wrong — whether an error, a power failure, a network error, or other mishaps occur — we need to restore the database at its last point of integrity.

To manually start a transaction, you call it with SQL like this:

```sql
BEGIN TRANSACTION
```

We are now in transaction mode. If anything goes wrong we `ROLLBACK`, otherwise we `COMMIT`. Below is a successful transaction where two records are created successfully.

```sql
INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_AMOUNT) VALUES (11, 720);
INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_AMOUNT) VALUES (12, 540);

COMMIT
```

Now here is an example that fails. Note that the second `INSERT` is missing a value for the `RECEIVE_AMOUNT`.

```sql
BEGIN TRANSACTION;

INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_AMOUNT) VALUES (15, 1020);
INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_AMOUNT) VALUES (17);

-- this second INSERT fails, so:
ROLLBACK
```

The transaction will fail and roll back, meaning that first `INSERT` is rolled back and if you check the `CUSTOMER_PAYMENT` table, you should not see it there.

```sql
SELECT * FROM CUSTOMER_PAYMENT
```

## Exercise

Complete the SQL below to insert the two records within a transaction into `CUSTOMER_PAYMENT` that commits on success, and rolls back on failure. Provide the `CUSTOMER_ORDER_ID`, `RECEIVE_DATE`, and `RECEIVE_AMOUNT`.

```sql
?

? INTO ? (?, ?, ?) VALUES (25, '2020-05-11', 1090);
? INTO ? (?, ?, ?) VALUES (27, '2020-05-12', 2070);

?
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
BEGIN TRANSACTION;

INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_DATE, RECEIVE_AMOUNT) VALUES (25, '2020-05-11', 1090);
INSERT INTO CUSTOMER_PAYMENT (CUSTOMER_ORDER_ID, RECEIVE_DATE, RECEIVE_AMOUNT) VALUES (27, '2020-05-12', 2070);

COMMIT;

SELECT * FROM CUSTOMER_PAYMENT
```