# Introduction to Relational Databases in SQL

## Creating and Modifying tables

### Create

```sql
CREATE TABLE table_name (
    col1 dtype,
    col2 dtype,
    ...
)
```

### Adding Constraints

#### During table creation

```sql
CREATE TABLE table_name (
    col1 dtype NOT NULL,
    col2 dtype UNIQUE
);
```

#### After table creation

```sql
ALTER TABLE table_name
ALTER COLUMN col
SET NOT NULL;

-- For unique
ALTER TABLE table_name
ADD CONSTRAINT constraint_name UNIQUE(col);
```

### Removing Constraints

```sql
ALTER TABLE table_name
ALTER COLUMN col
DROP NOT NULL;
```

### Add Column

```sql
ALTER TABLE table_name
ADD COLUMN new_column_name dtype;
```

### Rename Column

```sql
ALTER TABLE table_name
RENAME COLUMN old_name TO new_name
```

### Alter Column Datatype

Simple:

```sql
ALTER TABLE table_name
ALTER COLUMN col_name
TYPE new_dtype;
```

Requiring transformation (eg float to int)

```sql
ALTER TABLE table_name
ALTER COLUMN col_name
TYPE integer
USING ROUND(col_name)
```

### Drop (DELETE) Column

```sql
ALTER TABLE table_name
DROP COLUMN col
```

### Drop (DELETE) Table

```sql
DROP TABLE table_name;
```

## Inserting Data

**Row-by-row**

```sql
INSERT INTO table_name (col1, col2, ...)
VALUES (val1, val2, ...);
```

**Migrating data from table to table**

```sql
INSERT INTO destination
SELECT DISTINCT col1, col2  -- We use DISTINCT to avoid copying duplicates
FROM source
```

### Updating Column Data

```sql
UPDATE table_a
SET column_to_update = table_b.column_to_update_from
FROM table_b
WHERE condition1 AND condition2 AND ...;
```

## Dealing with Data Types

### Casting
```sql
SELECT CAST(col AS dtype) AS alias
FROM table
```

## Keys

- keys uniquely identify rows
- superkeys are a set of attributes that also uniquely identify rows
- a minimal superkey, is a superkey containing the least number of attributes required to still be considered a key.

### Identifying candidate keys

The following expression helps identify the number of unique rows for the given columns:

```sql
SELECT COUNT(DISTINCT(column_a, column_b, ...))
FROM table;
```

The following example shows which columns can serve as a key and a superkey, suppose our table has 11 rows.

```sql
SELECT COUNT(*)
FROM universities;
-- OUTPUT: 11 rows

SELECT COUNT(DISTINCT(university_city)) -- checking if this candidate works as a key
FROM universities;
-- OUTPUT: 9 rows
-- Since the number of rows do not match total rows in our table (9 rows vs 11 rows)
-- then we can not use that column as a key.

SELECT COUNT(DISTINCT(university_city, another_col)) -- Suppose `another_col` had less than 11 unique rows too
FROM universities;
-- OUTPUT: 11 rows
-- Since the number of rows match (11 rows vs 11 rows)
-- then we can safely use this combination of columns as a superkey.
-- Also note that since both columns on their own cannot be used as a key,
-- this combination of rows would be considered a minimal superkey.

SELECT COUNT(DISTINCT(university_shortname))
FROM universities;
-- OUTPUT: 11 rows
-- This column can be used as a key since it has the same number of unique rows as our table.
```

### Adding a primary key on new table

```sql
CREATE TABLE table_name (
    col1 integer PRIMARY KEY,
    col2 dtype <constraints>,
    ...
);
```

Using multiple columns for the primary key:

```sql
CREATE TABLE table_name (
    col1 dtype,
    col2 dtype,
    col3 dtype,
    ...,
    PRIMARY KEY (col1, col2)
)
```

### Adding primary key to existing table

```sql
ALTER TABLE table_name
ADD CONSTRAINT constraint_name PRIMARY KEY (col_name);
```

### Surrogate keys

An artifically generated key, like an auto-incrementing uuid, used when there is no suitable candidates for a key.
For example, when a car table only has the columns [make, model, color], in which no combination is a suitable candidate key.

We would have to use a surrogate key and add it like so:

```sql
ALTER TABLE cars
ADD COLUMN id serial PRIMARY KEY;
```

If you'd try to insert a row using a key that already exists you'd get an error.

You could also concatenate two columns into a another column to create a surrogate key:

```sql
ALTER TABLE table_name
ADD COLUMN id varchar(256);

UPDATE table_name
SET id = CONCAT(col1, col2);

ALTER TABLE table_name
ADD CONSTRAINT pk PRIMARY KEY (id);
```

### Foreign Keys

Foreign keys are used to "link" one table to another. For example, there may be a `manufacturers` table with a `name` column that is the primary key.

- In a 1:1 relationship only 1 foreign key is used
- In an n:n relationship (many to many) at least 2 foreign keys are used

We can create another table, `cars`, that has a `manufacturer` as a foreign key to the `manufacturers` table:

```sql
CREATE TABLE manufacturers (
    name varchar(255) PRIMARY KEY
);

CREATE TABLE cars (
    model varchar(255) PRIMARY KEY,
    manufacturer_name varchar(255) REFERENCES manufacturers (name));  -- Foreign key to manufacturers table
);
```

To add more than one foreign key:

```sql
CREATE TABLE table_a (
    col1_id integer REFERENCES table_b (id),
    col2_id varchar(256) REFERENCES table_c (id),
    col3 dtype
);
```

#### Adding a foreign key to an existing table

```sql
ALTER TABLE table_a
ADD CONSTRAINT name_fkey FOREIGN KEY (col_id) REFERENCES table_b (id)  -- Naming convention: foreign key is named `name_id` and reference a col named `id`
```

## Referential Integrity

When using foreign keys, you are not allowed to insert a record with a foreign key that doesn't exist.

Using other integrity constraints you can ensure that if a record in the foreign table gets deleted, it will also be deleted in the table that references it.

By default, sql doesn't do this, the explicit constraint is `ON DELETE NO ACTION`:

```sql
CREATE TABLE table_a (
    id integer PRIMARY KEY,
    col1 dtype,
    ...
    col_id integer REFERENCES table_b (id) ON DELETE NO ACTION
)
```

Other ingerity constraints are:
- `ON DELETE`:
    - `CASCADE`: delete rows that reference this
    - `RESTRICT`: throw an error
    - `SET NULL`: set the referencing column to NULL
    - `SET DEFAULT`: set the referencing column to its default value (works only if you ahve specified a default value)

### Changing constraints

You can't change a constraint through an `ALTER` statement, it needs to be dropped and added again:

```sql
-- 1. Identify the correct constraint name
SELECT constraint_name, table_name, constraint_type
FROM information_schema.table_constraints
WHERE constraint_type = 'FOREIGN KEY';

-- --------OUTPUT---------
-- constraint_name                      table_name      constraint_type
-- affiliations_organization_id_fkey    affiliations    FOREIGN KEY
-- affiliations_professor_id_fkey	    affiliations	FOREIGN KEY
-- professors_fkey	                    professors      FOREIGN KEY

-- 2. Drop the right foreign key constraint
ALTER TABLE affiliations
DROP CONSTRAINT affiliations_organization_id_fkey;

-- 3. Add a new foreign key constraint from affiliations to organizations which cascades deletion
ALTER TABLE affiliations
ADD CONSTRAINT affiliations_organization_id_fkey FOREIGN KEY (organization_id) REFERENCES organizations (id) ON DELETE CASCADE;
```
