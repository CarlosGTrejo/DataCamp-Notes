# Introduction to Snowflake SQL

## SNOWFLAKE SQL AND KEY CONCEPTS

### Introduction to Snowflake SQL

Options to connect to Snowflake:

- web interface (snowsight)
- drivers (odbc, jdbc, and connectors)
- snowflake CLI

Snowflake specific types:

- `NUMBER(precision, scale)`
  - precision is the number of digits
  - scale is the number of decimals
- `DATETIME_LTZ`
  - adds local time zone information
  - format: `YYY-MM-DD HH:MI:SS`

Data type conversion:

- `CAST(source AS dtype)`
- `source::dtype`

Conversion functions

- `TO_VARCHAR`
- `TO_DATE`

#### Checking data types:

`DESC TABLE table_name`

Example:

```SQL
DESC TABLE orders
```

Output:

| name       | type         | kind   | null? | default | primary key |
| :--------- | :----------- | :----- | :---- | :------ | :---------- |
| ORDER_ID   | NUMBER(38,0) | COLUMN | N     | null    | Y           |
| ORDER_DATE | DATE         | COLUMN | Y     | null    | N           |
| ORDER_TIME | TIME(9)      | COLUMN | Y     | null    | N           |

### Functions, sorting and grouping

#### String functions

`INITCAP( <expr> )`

- capitalize each word in a string (`foo bar` -> `Foo Bar`)

`CONCAT( <expr1> [ , <exprN> ...] )`

- concatenates the expressions

#### Date & Time

- `CURRENT_DATE()` OR `CURRENT_DATE`
- `CURRENT_TIME()` OR `CURRENT_TIME`

#### Other useful features

- `EXTRACT( <date_or_time_part> FROM <date_or_time_expr> )`
- `GROUP BY ALL`
  - group by all of the selected columns instead of re-writting them
  - ex:
    - `SELECT col1, col2, AVG(col3) ... GROUP BY col1, col2`
    - `SELECT col1, col2, AVG(COL3) ... GROUP BY ALL`

---

## ADVANCED SNOWFLAKE SQL CONCEPTS

### Joining in Snowflake

#### Natural Joins

- automatically match columns using the same name and eliminates duplicated ones

Syntax:

```sql
SELECT ...
FROM table_name
NATURAL [ { LEFT | RIGHT | FULL } [ OUTER ] ] JOIN table_two
...
```

Example:

```sql
SELECT
	-- Get the pizza category
    pt.category,
    SUM(p.price * od.quantity) AS total_revenue
FROM order_details AS od
NATURAL JOIN pizzas AS p
-- NATURAL JOIN the pizza_type table
NATURAL JOIN pizza_type AS pt
-- GROUP the records by category
GROUP BY category
-- ORDER by total_revenue and limit the records
ORDER BY total_revenue DESC
LIMIT 1;
```

#### Lateral Joins

- lets a subquery in `FROM` reference columns from preceding tables or views
- Subquery in the `FROM` clause _must_ be aliased

Syntax:

```sql
SELECT ...
FROM LHE, ...
LATERAL RHE
```

- `LHE`: table, view, or subquery
- `RHE`: Inline view or subquery

Example:

```sql
SELECT
  p.pizza_id,
  lat.name,
  lat.category
FROM pizzas AS p,
LATERAL -- Keyword LATERAL
  ( SELECT *
    FROM pizza_type AS t
    -- Referencing outer query column: p.pizza_type_id
    WHERE p.pizza_type_id = t.pizza_type_id
  ) AS lat
```

Example 2:

```sql
SELECT
  *
FROM orders AS o,
LATERAL (
  -- Subquery calculating total_spent
  SELECT
    SUM(p.price * od.quantity) AS total_spent
  FROM order_details AS od
  JOIN pizzas AS p
    ON od.pizza_id = p.pizza_id
  WHERE o.order_id = od.order_id
) AS t
ORDER BY o.order_id
```

### Subquerying and CTEs

```sql
SELECT pt.name,
    pt.category,
    SUM(od.quantity) AS total_orders
FROM pizza_type pt
JOIN pizzas p
    ON pt.pizza_type_id = p.pizza_type_id
JOIN order_details od
    ON p.pizza_id = od.pizza_id
GROUP BY ALL
HAVING SUM(od.quantity) < (
  -- Calculate AVG of total_quantity
  SELECT AVG(total_quantity)
  FROM (
    -- Calculate total_quantity
    SELECT SUM(od.quantity) AS total_quantity
    FROM pizzas p
    JOIN order_details od 
    	ON p.pizza_id = od.pizza_id
    GROUP BY p.pizza_id
    -- Alias as subquery
  ) AS subquery
)
```

### Snowflake Query Optimization

- Remember to specify which col to join on, or it can lead to "exploding joins" using up time and resources
- Use `UNION ALL` if you know your data has no duplicates as it is faster.
- Use filters to narrow down data
- Apply limits for quicker results
- Avoid `SELECT *` if you already know the cols you need
- Use `WHERE` early on, applying filters before `JOIN` will process fewer rows
- Use the query history view to analyze various metrics on previously executed queries to observe performance metrics
  - ```sql
    SELECT query_text, start_time, end_time, execution_time
    FROM snowflake.account_usage.query_history
    WHERE query_text ILIKE '%YOUR QUERY TEXT%'
      AND execution_time > 1000 -- or another number (measured in ms)
    ```

### Handling semi-structured data

- Snowflake has native support for storing, processing, and querying JSON data

#### Comparisons:

- Postgres: Uses `JSONB`
- Snowflake: Uses `VARIANT`
  - supports objects (key-value pairs) and arrays
  - ```sql
    CREATE TABLE table_name (
      col1 dtype,
      col2 VARIANT -- VARIANT dtype
    )
    ```

#### Semi-structured data functions

##### PARSE_JSON

- `PARSE_JSON( expr )`
  - `expr`: JSON data string
  - Returns: `VARIANT` type, valid JSON obj

Example:

```sql
SELECT PARSE_JSON(
-- Enclosed in strings
'{
"cust_id": 1,
"cust_name": "cust1",
"cust_age": 40,
"cust_email":"cust1***@gmail.com"
}
' -- Enclosed in strings
) AS customer_info_json
```

##### OBJECT_CONSTRUCT

- `OBJECT_CONSTRUCT( [key1, value1 [, keyN, valueN ...]])`
  - Returns: JSON object

Example:

```sql
SELECT OBJECT_CONSTRUCT(
-- Comma separated values rather than : notation
'cust_id', 1,
'cust_name', 'cust1',
'cust_age', 40,
'cust_email', 'cust1***@gmail.com'
)
```

#### Querying JSON data in Snowflake

- use `:` to retrieve the value of a given key

Example:

```sql
SELECT
  customer_info:cust_age, -- Use colon to access cust_age from column
  customer_info:cust_name,
  customer_info:cust_email
FROM
  cust_info_json_data;
```

#### Querying nested JSON data in Snowflake

- use `.` or `:` to access nested elements

Example:

```sql
SELECT
  customer_info:address:street AS street_name
FROM
  cust_info_json_data
```

Using dot notation (use `:` for the first level, then dots for subsequent levels):

```sql
SELECT
  customer_info:address.street AS street_name
FROM
  cust_info_json_data
```
