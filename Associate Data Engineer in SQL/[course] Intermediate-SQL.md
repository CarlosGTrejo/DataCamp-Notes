# Intermediate SQL

> DataCamp's AI native courses do not properly export user's notes.
> When I tried exporting my notes multiple times it only exported
> the "instructor's notes". The folder where my notes should've
> been was empty. For this reason I recommend taking notes outside
> of DataCamp, as it grants you more flexibility and control.

## Summary Values

**SQL Query Basics**

- A **SQL query** retrieves data from a database using the `SELECT` and
  `FROM` keywords.

```bigquery
SELECT column_1, column_2
FROM table_name;
```

- `SELECT` specifies which columns to retrieve (`*` for all columns).
- `FROM` specifies which table to query.
- `;` is the statement terminator that marks the end of a query.


**Data Aggregation**

- **Data aggregation** is the operation of summarizing data by applying
  functions (like SUM, AVG, COUNT) to columns to produce single values.
- Data aggregation reveals insights and patterns that are difficult to
  recognize by examining raw data alone.


**Summarizing Data in SQL**

- To apply summary operations in SQL, use aggregate functions within a
  SELECT statement:

```bigquery
SELECT COUNT(column_1) AS metric_1,
       SUM(column_2)   AS metric_2
FROM table_name;
```

- The `AS` keyword assigns a temporary name (alias) to a column, making
  output more readable.
- Calculate multiple summary values in one query by listing aggregate
  functions separated by commas.
- You can also calculate just one summary value by using a single aggregate
  function.


**Summary Operations**

- `SUM()`: Calculates the total of all values in a column
- `AVG()`: Calculates the average value in a column
- `MIN()`: Finds the minimum value in a column
- `MAX()`: Finds the maximum value in a column
- `COUNT()`: Counts the number of rows or values
- `COUNT(DISTINCT column)`: Counts the number of unique values


**Design Framework**

- Thinking through your query before writing code is one of the most
  effective habits of successful data analysts.
- To effectively design a data aggregation operation, answer two key
  questions:
    1. **Which column do we wish to summarize?** Select the column
       containing values relevant to your question.
    2. **Which summary operation do we wish to apply?** Choose the
       appropriate operation based on the insight you need.

---

## One Grouping Column

**Grouped Aggregation**

- **Grouped aggregation** calculates summary statistics for each distinct
  value in a grouping column, producing a summary table.
- Individual summary values tell us about the overall dataset, but grouped
  aggregation reveals patterns and variations across categories.
- The summary table has one row per group and one column per metric.


**Design Framework**

- To design an effective grouped aggregation, answer three questions:
    1. **By which column do we group?** The grouping column determines how
       rows are split into groups.
    2. **Which column(s) do we summarize?** The input columns on which
       metrics are calculated.
    3. **Which summary operation(s) do we apply?** Common operations
       include count, sum, mean, and unique count.
- Thinking through your query before writing code is one of the most
  effective habits of successful data analysts.


**Split-Apply-Combine**

- Grouped aggregation works through a three-step process:
    1. **Split**: Data is divided into groups based on unique values in the
       grouping column.
    2. **Apply**: Summary operations are performed on each group
       separately.
    3. **Combine**: Results from each group are assembled into a single
       summary table.
- This happens behind the scenes when we use the `GROUP BY` clause.


**Uniqueness**

- `SELECT DISTINCT` returns the unique values in a column.

```bigquery
SELECT DISTINCT column_name
FROM table_name;
```

- Before grouping, use `SELECT DISTINCT` to explore what groups exist in
  your data. This reveals unexpected categories and helps you anticipate
  issues before aggregation.


**GROUP BY**

- The `GROUP BY` clause, placed after `FROM`, groups rows by a specified
  column.

```bigquery
SELECT grouping_column,
       COUNT(column_1) AS total_count,
       SUM(column_2)   AS total_amount
FROM table_name
GROUP BY grouping_column;
```

- The `SELECT` clause should include the grouping column and aggregate
  functions applied to other columns.
- Alias each aggregate expression with `AS` for clarity.


**Sorting**

- The `ORDER BY` clause sorts results by a specified column.
- By default, sorting is ascending (lowest to highest). Add `DESC` for
  descending order.

```bigquery
SELECT grouping_column, SUM(amount) AS total
FROM table_name
GROUP BY grouping_column
ORDER BY total DESC;
```


**SQL Clause Order**

- SQL requires clauses to be written in a specific order: `SELECT` →
  `FROM` → `GROUP BY` → `ORDER BY`.
- This is a syntax convention designed for readability.
- Placing clauses out of order produces a syntax error.

---

## Multiple Grouping Columns

**Grouping by Multiple Columns**

- Multi-column aggregation enables cross-dimensional analysis, answering
  questions like "How does X vary across combinations of A and B?"
- To group by multiple columns, list the grouping columns in both the
  `SELECT` clause (to identify groups) and the `GROUP BY` clause (to define
  groups), separated by commas.

```sql
SELECT grouping_col1,
       grouping_col2,
       COUNT(column_1)          AS metric_1,
       COUNT(DISTINCT column_2) AS metric_2,
       SUM(column_3)            AS metric_3,
       AVG(column_3)            AS metric_4
FROM table_name
GROUP BY grouping_col1, grouping_col2;
```

- All columns in the `SELECT` clause that are not used in aggregate
  functions must be included in the `GROUP BY` clause.


**Metric Reliability**

- Summaries based on small groups are often unreliable. Exceptional values
  in small groups are frequently anomalies that disappear as group size
  increases.
- Always include the group size (typically a count) in summary tables to
  evaluate which summary values can be trusted.


**Trade-offs of Many Grouping Columns**

- You can group by as many columns as needed for the desired level of
  detail.
- However, each additional grouping column increases the number of
  resulting groups.
- Too many grouping columns can lead to:
    1. An overwhelming number of summary values that are challenging to
       interpret
    2. Smaller group sizes that produce less reliable summary statistics


**Handling Missing Groups**

- Groups with no data will not appear in the summary table.
- Missing groups can occur for three reasons:
    1. **Data Loss or Query Error**: Data was lost, incompletely collected,
       or a bug in the query failed to capture existing combinations—something
       went wrong that needs investigation.
    2. **No Instances Occurred**: The combination is valid but had zero
       activity in the data—the data is correct.
    3. **Impossible Combination**: The combination cannot exist by definition.


**Multi-Column Sorting**

- We can sort by multiple columns with individual directions for each.

```sql
SELECT grouping_col1,
       grouping_col2,
       COUNT(column_1) AS metric_1,
       SUM(column_2)   AS metric_2
FROM table_name
GROUP BY grouping_col1, grouping_col2
ORDER BY grouping_col1 ASC, metric_2 DESC;
```

- SQL clauses must appear in this order: `SELECT` → `FROM` → `GROUP BY` →
  `ORDER BY`.

---

## Basic Transformations

**Data Transformation**

- Data Transformation is a fundamental data manipulation operation that
  allows us to derive new columns from existing columns.
- With data transformation, the output table is a copy of the original
  input table with the new columns attached at the end. This is in contrast
  to data aggregation, which summarizes the data, providing a higher level,
  less granular view.


**Data Transformation in SQL**

- We can create new columns in SQL by adding expressions in the `SELECT`
  statement.

```sql
SELECT *,
       col_usd * 0.92 AS col_eur
FROM table_name;

```

- We can use any standard arithmetic operator to combine columns, as
  well as any SQL function.
- Use parentheses to group operations for clarity and to control operator
  precedence. When in doubt, add parentheses.
- Instead of `SELECT *` to select all columns, we can specify specific
  columns to include.


**Calculating Ratios**

- Ratios measure relationships between different quantities and are
  calculated by dividing one column value by another.
- Division by zero is a common error in calculations.
- Use `NULLIF` to handle division by zero gracefully:

```sql
SELECT *,
       col_a / NULLIF(col_b, 0) AS ratio
FROM table_name;
```

- Or use `SAFE_DIVIDE` to handle division by zero in BigQuery:

```sql
SELECT *,
       SAFE_DIVIDE(col_a, col_b) AS ratio
FROM table_name;
```

---

## Complex Transformations

**WITH Clause for Multi-Step Calculations**

- SQL cannot reference a column alias in the same `SELECT` where it's
  defined
- Use the `WITH` clause to break complex transformations into named steps
- Each step can reference columns created in previous steps

```sql
WITH step_1 AS (SELECT *,
                       (col_a * col_b) - col_c AS intermediate
                FROM table_name)
SELECT *,
       SAFE_DIVIDE(100 * intermediate, col_a * col_b) AS ratio
FROM step_1;

```


**Scalar Subqueries for Aggregate Values**

- Aggregate functions like `SUM()` collapse rows, so they can't be used
  directly in row-level calculations
- A scalar subquery calculates the aggregate separately and returns a
  single value that each row can use

```sql
SELECT *,
       col_a - (SELECT AGG_FUNC(col_a) FROM table_name) AS result
FROM table_name;

```


**Calculating Percent of Total**

- Percent of total normalizes values to a common scale, making relative
  size immediately clear
- Divide each value by the sum (via scalar subquery) and multiply by 100

```sql
SELECT *,
       100 * col_a / (SELECT SUM(col_a) FROM table_name) AS col_a_pot
FROM table_name;

```


**Rounding and Transformation Functions**

- Use the `ROUND()` function to control decimal places for readability
- Rounding is one example of many transformation functions; AI assistants
  can help you find the right function for your needs

```sql
SELECT *,
       ROUND(100 * col_a / col_b, 2) AS percentage
FROM table_name;

```

---

## Basic Filtering

**Filtering in SQL**

- We apply filtering to our data to extract only those rows that are
  relevant for the purpose at hand.
- To filter rows in SQL, we use the `WHERE` clause with a condition.

```sql
SELECT col_2, col_3
FROM table_1
WHERE col_1 = 'value';
```


**Strings in SQL**

- String values are text data, as opposed to numeric data.
- In SQL, string values must be enclosed in quotes—either single quotes (
  `'`) or double quotes (`"`).
- Single quotes work in all SQL databases, while double quotes only work in
  some.


**Logical Expressions**

- Filtering uses logical expressions that evaluate to `TRUE` or `FALSE`.
- A logical expression has three parts: column reference, comparison
  operator, and value.
- The most commonly used comparison operators:
    - `=` (equal to), `!=` or `<>` (not equal to)
    - `<` (less than), `>` (greater than)
    - `<=` (less than or equal), `>=` (greater than or equal)


**Partial String Matching**

- To filter rows where a column contains a pattern, use `LIKE` with `%`
  wildcards.

```sql
SELECT *
FROM table_1
WHERE col_1 LIKE '%value%';
```

- The `%` is a wildcard that matches any sequence of characters:
    - `'%value%'`: contains 'value' anywhere
    - `'value%'`: starts with 'value'
    - `'%value'`: ends with 'value'


**Missing Values**

- Missing values are represented as `NULL` in SQL.
- To filter rows where a column is null (missing), use `IS NULL`.

```sql
SELECT *
FROM table_1
WHERE col_1 IS NULL
```

- To filter rows where a column has a value, use `IS NOT NULL`.

```sql
SELECT *
FROM table_1
WHERE col_1 IS NOT NULL
```

---

## Multiple Conditions

**AND and OR Operations**

- We can combine multiple filtering conditions using `AND` and `OR`
  operations.
- The `AND` operation returns rows that satisfy both conditions.
- The `OR` operation returns rows that satisfy either condition.

| Condition 1 | Condition 2 | AND Result | OR Result |
|-------------|-------------|------------|-----------|
| <green>True</green>  | <green>True</green>  | <green>True</green>  | <green>True</green>  |
| <green>True</green>  | <red>False</red>     | <red>False</red>     | <green>True</green>  |
| <red>False</red>     | <green>True</green>  | <red>False</red>     | <green>True</green>  |
| <red>False</red>     | <red>False</red>     | <red>False</red>     | <red>False</red>     |


**Logical Operator Precedence**

- By default, `AND` operations are executed before `OR` operations.
- We can use parentheses to enforce a particular order of execution.


**Design Framework**

We developed the following framework for designing filtering operations:

1. **What filtering conditions do we need to apply?**
2. **If applicable, how do we combine multiple conditions?**


**AND and OR Keywords**

- We can combine multiple filtering conditions using `AND` and `OR`
  keywords.
- The `AND` keyword returns rows that satisfy both conditions.

```bigquery
SELECT col_1, col_2
FROM table_1
WHERE col_1 = 'value_a'
  AND col_2 = 'value_b';
```

- The `OR` keyword returns rows that satisfy either condition.

```bigquery
SELECT col_1, col_2
FROM table_1
WHERE col_1 = 'value_a'
   OR col_2 = 'value_b';
```


**IN Operator**

- To check if a column value is in a list of values, we use the `IN`
  operator, which is equivalent to having multiple equality (`=`)
  conditions combined with `OR`.

```bigquery
SELECT col_1, col_2
FROM table_1
WHERE col_1 IN ('value_a', 'value_b', 'value_c');
```


**Parentheses for Evaluation Order**

- Parentheses control the order in which conditions are evaluated.
- Use parentheses to group conditions that should be evaluated together.

```bigquery
SELECT col_1, col_2, col_3
FROM table_1
WHERE condition_1
  AND (condition_2 OR condition_3);
```

---

## Complex Filtering

**Top N Filtering**

- Select the top N or bottom N rows based on a column value
- In SQL, use `ORDER BY` with `LIMIT` to sort and then keep only the first
  N rows:

```sql
SELECT *
FROM table_1
ORDER BY column_1 DESC LIMIT n;
```


**Intermediate Columns**

- Break complex filtering conditions into named boolean columns for clarity
  and verification
- Use `WITH` to create intermediate columns first, then filter—needed
  because `WHERE` is evaluated before `SELECT`:

```sql
WITH enriched_table AS (SELECT *,
                               col_1 <= a    AS condition1,
                               col_2 > b     AS condition2,
                               col_1 < col_3 AS condition3
                        FROM table_1)
SELECT *
FROM enriched_table
WHERE condition1
  AND (condition2 OR condition3);
```


**Direct Calculation in Filtering**

- Use expressions directly as conditions in filtering operations:

```sql
SELECT *
FROM table_1
WHERE col_1 <= a
   OR col_2 / col_3 > b;
```

- For simple cases this works well, but for complex conditions, use
  intermediate columns with the `WITH` clause to keep code clear and reduce
  mistakes.


**Condition Inversion**

- Use the `NOT` keyword to negate a condition and get the complement:

```sql
SELECT *
FROM table_1
WHERE NOT (col_1 <= a OR col_2 / col_3 > b);
```

- Always enclose compound conditions in parentheses when using `NOT`.


**Result Verification**

- Complex filtering conditions are prone to errors—systematic verification
  catches mistakes before they lead to incorrect conclusions
- Verification happens at two stages:
    1. **While building** — Verify each intermediate column works correctly
       on its own
    2. **After building** — Verify the final filtered result includes the
       right rows and excludes the right rows
- Pick a few rows and check them by hand to build confidence that your
  filter is working correctly

---

## Conditional Transformation

**Conditional Transformation**

- Creates new columns by evaluating conditions for each row and applying
  different logic based on which conditions are met
- For each row, the system evaluates conditions in order and applies the
  first matching outcome
- Condition order matters — earlier conditions take precedence


**CASE WHEN with Single Condition**

- Use for single-condition transformations with two outcomes

```sql
CASE WHEN condition THEN true_value ELSE false_value
END
AS column_name
```

- Returns `true_value` when `condition` is TRUE and `false_value` when
  `condition` is FALSE.


**CASE WHEN with Multiple Conditions**

- Use CASE WHEN multiple-condition transformations with multiple outcomes

```sql
CASE
    WHEN condition_1 THEN outcome_1
    WHEN condition_2 THEN outcome_2
    ELSE default_outcome
END
AS column_name
```

- Evaluates conditions in order; first match wins
- `ELSE` provides the default (returns NULL if omitted)


**Binning**

- Groups numerical values into discrete categories based on thresholds
- Unlocks data summaries and insights that wouldn't otherwise be possible
- Often carried out using `CASE WHEN` in a `SELECT` statement.


**Chaining CTEs**

- Use `WITH` only once, separate each CTE with a comma
- Each CTE can reference any CTE defined above it
- The final `SELECT` queries from whichever CTE has what you need

---

## Conditional Aggregation

**SQL Execution Order**

- Understanding execution order is key to knowing which filtering technique
  to use.
- Complete order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` →
  `ORDER BY`
- `WHERE` runs before `GROUP BY` (filters rows), `HAVING` runs after
  `GROUP BY` but before `SELECT` (filters groups based on aggregates).


**Pre-Aggregation Filtering**

- Filters rows before they enter groups—excluded rows never participate in
  aggregation.
- Use the `WHERE` clause before `GROUP BY`.

```sql
SELECT group_col, agg(value_col)
FROM table
WHERE condition
GROUP BY group_col
```


**Post-Aggregation Filtering**

- Filters entire groups after aggregation based on aggregate values.
- Use the `HAVING` clause after `GROUP BY`.
- Cannot use `WHERE` because it runs before aggregates exist.

```sql
SELECT group_col, agg(value_col) AS agg_value
FROM table
GROUP BY group_col
HAVING agg(value_col) condition
```


**Conditional Aggregation**

- Selectively includes or excludes values within the aggregation itself.
- Use `CASE WHEN` inside aggregate functions.

```sql
SELECT group_col,
       agg(CASE WHEN condition THEN value_col END)
FROM table
GROUP BY group_col
```


**NULL Handling in Aggregates**

- Aggregate functions (`AVG`, `MIN`, `MAX`, `SUM`) ignore NULL values in
  calculations.
- `COUNT(column)` excludes NULLs; `COUNT(*)` counts all rows including
  NULLs.
- This is why `CASE WHEN` works for conditional aggregation—unmatched rows
  return NULL and are automatically excluded from the calculation.

