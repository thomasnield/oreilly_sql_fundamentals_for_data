# Pulling Data with SELECT

In this section, we are going to learn the most common SQL command. `SELECT` is used to retrieve data from one or more tables. It can also transform data before it is returned. However, it is a read-only operation so it does not change the underlying tables. 

## Selecting Columns 

Let's first select all columns from the `CUSTOMER` table. 

```sql
SELECT * FROM CUSTOMER;
```

Note that the asterisk `*` indicates to select all columns, and the `FROM` is preceded by the table you are selecting the columns from which is `CUSTOMER`. We can see that there are 10 customers in this table. 

If you want to limit your query to just the first 5 results, add a `LIMIT 5` so it cuts off returning data after 5 records. This is helpful if there are a lot of records and you want just a sample of records to see what the data looks like. 

```sql
SELECT * FROM CUSTOMER LIMIT 5;
```

Note you can also select specific columns separated by commas. This is helpful to only grab columns you are interested in as well as reduce the amount of data that has to be retrieved. Below we only retrieve the `CUSTOMER_NAME` and `ADDRESS` columns. 

```sql
SELECT CUSTOMER_NAME, ADDRESS FROM CUSTOMER;
```

If you want to see what tables are available in a database, you can ask for documentation from the database administrator or use a graphical user interface tool which displays the tables. 

In SQLite, there is a hidden administrative table called `sqlite_master` that allows you to list all the objects in a database. We will learn more about the `WHERE` keyword, but note it allows us to filter to only `table` objects. 

```sql
SELECT NAME FROM sqlite_master WHERE type='table';
```

## Expressions and Functions

Let's take a look at the `PRODUCT` table. 

```sql
SELECT * FROM PRODUCT;
```

Let's say we want to drop each price by 10%. We can multiply each price by `0.9` by creating a new field as an expression. We will call it `REDUCED_PRICE`. This does not modify the table, but rather transforms the data before it is returned. It is calculating that `REDUCED_PRICE` only within this query, much like a formula in Excel. This is what's great about SQL. It allows the stored data to be simple and minimal, but we can layer calculations and manipulations on top of it within a query. 

```sql
SELECT PRODUCT_NAME,
PRICE,
PRICE * 0.9 AS REDUCED_PRICE
FROM PRODUCT;
```

Note how SQL queries can be written across multiple lines for legibility. 

The mathematical operators you can expect in every SQL platform are as follows: 

| Symbol | Operation                       |
|--------|----------------------------------|
| `+`    | Adds two numbers                |
| `-`    | Subtracts two numbers            |
| `*`    | Multiplies two numbers           |
| `/`    | Divides two numbers              |
| `%`    | Divides, but returns remainder   |

Note that these mathematical operators only work between numeric values or fields. These symbols may be used in other contexts, such as the `*` can mean "select all columns" but between two numbers it is a multiplication.

Now let's say we want to calculate a `PROCESS_FEE` for each price, which is `.00047` multiplied on the `PRICE`. 

```sql
SELECT PRODUCT_NAME,
PRICE,
PRICE * .00047 AS PROCESS_FEE
FROM PRODUCT;
```

If we want to round these values to two decimal places, we have to use a function. Functions have a name, open with parentheses, accept arguments, and return a result. Here is the `ROUND()` function to two decimal places on the `REDUCED_PRICE` field. 

```sql
SELECT PRODUCT_NAME,
PRICE,
ROUND(PRICE * .00047, 2) AS PROCESS_FEE
FROM PRODUCT;
```

When you are working with text, an operator `||` can be used to concatenate text together (although some database platforms use a `CONCAT()` function instead). If we wanted to merge several fields in the `CUSTOMER` table to create a `SHIP_ADDRESS`, we can do so like this. Note how spaces `' '` and commas `', '` are padded in between each field.

```sql
SELECT CUSTOMER_NAME,
ADDRESS || ' ' || CITY || ', ' || STATE || ' ' || ZIP AS SHIP_ADDRESS
FROM CUSTOMER;
```

## Commenting Code and Syntax Rules

You can comment code out in SQL using a double dash `--` or multiline syntax `/* */`. These will be ignored by the SQL engine and can be a helpful way to provide context and explanations to your SQL code. 

```sql
-- this is a comment

/*
This is a
multiline comment
*/
```

SQL is not case sensitive so keywords, fields, and table names can be uppercase or lowercase regardless how they are named in storage. You will see queries often end with a semicolon `;` but this is only necessary when running multiple SQL commands at once. Usually running multiple SQL commands happen in writing data, not selecting data. 

# Exercise

Complete the SQL query below by replacing the question marks `?`. Retrieve all records from the `CUSTOMER` table, but grab the `CUSTOMER_NAME` and `CATEGORY` fields. Also concatenate the `CITY` and `STATE` with a comma in-between and name that expression `LOCATION`. 

```sql
SELECT ? FROM ?;
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
SELECT CUSTOMER_NAME, CATEGORY, CITY || ', ' || STATE AS LOCATION FROM CUSTOMER;
```

# Filtering Data with WHERE 

In this section, we will learn how to filter records based on a condition. This is achieved with the `WHERE` clause of a SQL query. 

Let's take a look at the table `WEATHER_MONITOR` and sample 10 records out of it. Note this is weather data capturing several measurements including `RAIN` and `LIGHTNING`, as well as TRUE/FALSE indicators like `LIGHTNING`, `HAIL`, `TORNADO` which will be 1 and 0 respectively (1 for TRUE, 0 for FALSE). 

## Filtering Numeric Expressions

We are first going to cover filtering data with numeric operations, some of which will extend into other data types like text. 

```sql
SELECT * FROM WEATHER_MONITOR LIMIT 10;
```

Let's say we want to find all records that have a temperature of exactly 64 degrees Fahrenheit. We can simply use an `=` operator in a `WHERE` condition like this:

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE TEMPERATURE = 64;
```

To get all records that are not 64 degrees, you can use the `!=` or `<>` operator which expresses "not equals." 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE TEMPERATURE != 64;
```

To get all records within a value range, you can use the `BETWEEN` operator. To get all records with a temperature between 10 and 20 degrees, target a `BETWEEN` on the `TEMPERATURE` field. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE TEMPERATURE BETWEEN 10 AND 20;
```

The `BETWEEN` is inclusive so it will include 10 and 20 degrees. If you want to exclude the bounds, and strictly only return records exclusively between 10 and 20 degrees, use comparative operators `>` and `<` with an `AND` to qualify both conditions. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE TEMPERATURE > 10 AND TEMPERATURE < 20;
```

The inclusive `BETWEEN` could also be accomplished using `>=` and `<=`. 

Let's say we want to get records where the `LOCATION_ID` is 5, 20, or 35. We can achieve this using an `OR` which specifies at least one condition must be true. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE LOCATION_ID = 5 
OR LOCATION_ID = 20 
OR LOCATION_ID = 35;
```

This demonstrates the `OR` allows a condition to be composed of multiple conditions, where at least one of them must be true. But for this particular problem we can use the `IN` operator to qualify a set of values in a set. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE LOCATION_ID IN (5, 20, 35);
```

You can also negate a condition by preceding it with the `NOT` keyword. To get all records where the `LOCATION_ID` is not 5, 20, or 35 run this query: 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE LOCATION_ID NOT IN (5, 20, 35);
```

## Filtering Boolean Values  

When you encounter fields that are binary (1 = TRUE, 0 = FALSE) which are also called booleans, you simply qualify the same way you would with other numbers. Here we find records where a tornado was sighted (1).  

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE TORNADO = 1;
```

You can also qualify records where a tornado was not sighted (0). 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE TORNADO = 0;
```

Be careful mixing `AND` and `OR` operations as this can mangle conditions, confusing both people and machines. For instance, suppose we wanted to find records where there was snow or sleet. For sleet to happen, there must be rain and the temperature must be less than or equal to 32 degrees. Now study the query below, and ask yourself which conditions belong to the `AND` versus the `OR`? 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE SNOW > 0 OR RAIN > 0 AND TEMPERATURE <= 32;
```

This technically works, although mixing `AND` and `OR` like this can create confusion and even errors for more complicated queries. This is why it is a good idea to force an order of operations with parentheses, so the conditions are grouped appropriately and evaluated in the intended order. This should be done even if it is just for clarity. Below we organize the query so the sleet condition is grouped into a single condition. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE SNOW > 0 OR (RAIN > 0 AND TEMPERATURE <= 32);
```

## Filtering Text Expressions

Let's say you want to look up a record with a given `REPORT_CODE`. Since that field is text and not a number, you need to specify that report code `'YJA6G3I'` in single quotes. This is because numeric values are not allowed to be column or table names, so we do not need quotes around literal numeric values. But we do need quotes around text values so the SQL engine does not get confused looking for that value as a column or table name. 

```sql
SELECT * FROM WEATHER_MONITOR
WHERE REPORT_CODE = 'YJA6G3I';
```

This rule applies to other operators we learned earlier, including using the `IN` operator. Below we look up three report codes. 

```sql
SELECT * FROM WEATHER_MONITOR
WHERE REPORT_CODE IN ('YJA6G3I', 'M511XRH', 'S4ED81Y');
```

Some operators are specific to text, such as concatenation `||` or `LIKE` which allows us to match text with wildcards. Here is a `LIKE` operation that searches for report codes that have a `Y` in the first position and a `D` in the third. The `_` in the pattern string is a wildcard for one character, and the `%` is a wildcard for any number of characters. 

```sql
SELECT * FROM WEATHER_MONITOR
WHERE REPORT_CODE LIKE 'Y_D%';
```

There are also functions that specifically are for working with strings like `length()` and `substr()`. Here we use a substring operation to extract out the middle 5 characters of the 7-character report code. The first argument is the string, the second is the starting character, and the third is the number of characters to grab starting from that position.

```sql
SELECT REPORT_CODE, substr(REPORT_CODE, 2, 5) FROM WEATHER_MONITOR;
```

You can view all the functions SQLite offers [in its documentation](https://www.sqlite.org/lang_corefunc.html). 

## Filtering Dates and Time

Dates and time can be a little awkward in SQL as each platform will treat them differently. You typically want to establish time zone awareness in your date and time data, storing dates as [Greenwich Mean Time (GMT)](https://en.wikipedia.org/wiki/Greenwich_Mean_Time) or [Coordinated Universal Time (UTC)](https://en.wikipedia.org/wiki/Coordinated_Universal_Time). Then you can track which timezone the data was recorded in and adjust to local time accordingly. 

To keep things simple, let's just work with the `REPORT_DATE` column. If we want to get all records where `REPORT_DATE` is after `2021-05-15`, I can provide that date in a string of `yyyy-MM-dd` format. This is the [ISO 8601 standard](https://en.wikipedia.org/wiki/ISO_8601) for formatting dates. SQLite will then recognize this as a date instead of a plain string. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE REPORT_DATE > '2021-05-15';
```

Each SQL platform will likely have a different way of extracting the month, day, or other components of a date or time. SQLite has a particular way of working with dates and times as well. If we want to filter for 2021 records, we can use `strftime()` to extract out the year using a [special formatting syntax](https://www.sqlite.org/lang_datefunc.html) where `%Y` will extract the year component. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE strftime('%Y', REPORT_DATE) = '2021';
```

You can convert the year from a string to an integer using the `CAST` operator. 

```sql
SELECT * FROM WEATHER_MONITOR 
WHERE CAST(strftime('%Y', REPORT_DATE) AS INTEGER) = 2021;
```

You can get today's date using `DATE('now')` and use this to qualify queries for today's date. 

```sql
SELECT DATE('now');
```

You can also get the current UTC time using the `TIME()` function. Note the format which is compliant to ISO 8601 format. 

```sql
SELECT TIME('now');
```

You can also work with a full date and time, as well as add and subtract different calendar operations. This grabs yesterday's date. 

```sql
SELECT DATETIME('now', '-1 day');
```

By following the ISO 8601 format, you can turn any properly formatted string into a `DATE`, `TIME` or `DATETIME` and perform any comparative, or calendar logic you want. 

```sql
SELECT DATETIME('2022-10-19 18:58:12') AS MY_DATE_TIME;
```

# EXERCISE

Complete the query below to find all records where there was a tornado and hail, OR the rain was greater than 5 inches and temperature was at least 70. 

```sql
SELECT * FROM WEATHER_MONITOR
WHERE ?;
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
SELECT * FROM WEATHER_MONITOR
WHERE (TORNADO = 1 AND HAIL = 1) OR (RAIN > 5 AND TEMPERATURE >= 70);
```
