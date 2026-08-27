# SQL DDL & DML: Defining and Manipulating Data

Notes from Data With Baraa SQL course, Module: DDL & DML

## The Big Picture: DDL vs DML

SQL commands are grouped into categories based on what they do:

- **DDL (Data Definition Language)**: defines or changes the
  *structure* of the database, such as creating tables, adding or
  removing columns, or deleting tables entirely. DDL affects the
  shape of the data, not the data itself.
- **DML (Data Manipulation Language)**: works with the data inside
  tables, such as adding rows, changing values, or removing rows.
  DML doesn't touch the structure; it just changes what's stored in
  it.

A simple way to remember it: DDL builds the container. DML fills or
empties it.

---

## DDL: Data Definition

### CREATE TABLE

Builds a new table, defining its columns and their data types.

```sql
CREATE TABLE persons (
    id INT NOT NULL,
    person_name VARCHAR(50) NOT NULL,
    birth_date DATE,
    phone VARCHAR(15) NOT NULL,
    CONSTRAINT pk_persons PRIMARY KEY (id)
)
```

Key pieces:
- `INT`, `VARCHAR(n)`, `DATE`: data types that define what kind of
  value each column can hold. `VARCHAR(50)` means text up to 50
  characters.
- `NOT NULL`: a constraint meaning this column can't be left empty;
  every row must have a value here.
- `CONSTRAINT pk_persons PRIMARY KEY (id)`: sets `id` as the
  **primary key**, meaning every value in that column must be unique
  and identifies each row. A table can only have one primary key.

Columns without `NOT NULL` (like `birth_date` here) are allowed to
be empty (`NULL`).

### ALTER TABLE

Changes the structure of a table that already exists, without having
to drop and recreate it.

**Adding a column:**
```sql
ALTER TABLE persons
ADD email VARCHAR(50) NOT NULL
```

**Removing a column:**
```sql
ALTER TABLE persons
DROP COLUMN phone
```

`ALTER` is used any time the table's shape needs to change after it's
already in use, for example if a new piece of information needs
tracking that the original design didn't include.

### DROP TABLE

Deletes an entire table, structure and all data inside it,
permanently.

```sql
DROP TABLE persons
```

This is different from deleting rows (that's DML, see `DELETE`
below). `DROP TABLE` removes the table itself; querying it afterward
would return an error, because it no longer exists.

---

## DML: Data Manipulation

### INSERT INTO

Adds new rows into an existing table.

```sql
INSERT INTO customers (id, first_name, country, score)
VALUES
    (6, 'Anna', 'USA', NULL),
    (7, 'Sam', NULL, 100)
```

- Column names in `INSERT INTO (...)` tell SQL which columns you're
  providing values for, and in what order.
- Multiple rows can be inserted in one statement by separating value
  sets with commas.
- `NULL` can be used for a column when you don't have a value for it
  yet, as long as the column allows `NULL` (isn't `NOT NULL`).

You don't have to provide every column. If you leave one out, and it
allows `NULL`, it's just left empty:

```sql
INSERT INTO customers (id, first_name)
VALUES
    (10, 'Sahra')
```

**Inserting data copied from another table:**

`INSERT INTO ... SELECT` lets you populate a table using data
already in another table, instead of typing values manually.

```sql
INSERT INTO persons (id, person_name, birth_date, phone)
SELECT
    id,
    first_name,
    NULL,
    'Unknown'
FROM customers
```

This reads `id` and `first_name` from `customers`, and fills
`birth_date` with `NULL` and `phone` with the literal text
`'Unknown'` for every row copied over. Useful for migrating or
seeding data from one table into another with a different structure.

### UPDATE

Changes existing values in rows that are already in a table.

```sql
UPDATE customers
SET score = 0
WHERE id = 6
```

- `SET` specifies which column(s) to change and what to change them
  to.
- `WHERE` specifies which rows to apply the change to.

**This is critical: if you forget the `WHERE` clause, the update
applies to every single row in the table.** Always double check the
`WHERE` condition before running an `UPDATE`.

Multiple columns can be updated in one statement:

```sql
UPDATE customers
SET
    score = 0,
    country = 'UK'
WHERE id = 10
```

`WHERE` conditions aren't limited to exact matches. They can also
target rows based on a condition like being empty:

```sql
UPDATE customers
SET score = 0
WHERE score IS NULL
```

(Note: `IS NULL` is used instead of `= NULL`, because `NULL`
represents "unknown/absent," not a comparable value. SQL requires the
special `IS NULL` / `IS NOT NULL` syntax to check for it.)

### DELETE FROM

Removes specific rows from a table, based on a condition.

```sql
DELETE FROM customers
WHERE id > 5
```

Like `UPDATE`, the `WHERE` clause is what limits the damage. Leaving
it out deletes every row in the table, though the table itself still
exists afterward (empty, but present).

### DELETE vs TRUNCATE vs DROP: the key distinction

These three all "remove" something, but they're very different:

| Command | Removes | Table structure | Can use WHERE? |
|---|---|---|---|
| `DELETE FROM table WHERE ...` | specific rows | stays | yes |
| `TRUNCATE TABLE table` | all rows | stays | no |
| `DROP TABLE table` | all rows + structure | gone entirely | no |

```sql
-- Removes all data from persons, but the persons table still exists
TRUNCATE TABLE persons
```

`TRUNCATE` is a fast way to empty a table completely when you don't
need to filter which rows go. It sits between `DELETE` (row level,
DML) and `DROP` (structural, DDL) since it clears everything without
removing the table itself.

---

## Key Takeaways

- DDL changes structure (`CREATE`, `ALTER`, `DROP`). DML changes data
  (`INSERT`, `UPDATE`, `DELETE`).
- `NOT NULL` and `PRIMARY KEY` are constraints set at table creation
  to enforce data integrity.
- `INSERT INTO ... SELECT` copies data from one table into another
  without manual retyping.
- **Always check the `WHERE` clause before running `UPDATE` or
  `DELETE`.** Missing it means the change applies to the entire
  table.
- `DELETE` (row level, filterable) vs `TRUNCATE` (clears all rows,
  keeps table) vs `DROP` (removes the table entirely) are three
  different levels of removal. Know which one a task actually calls
  for.