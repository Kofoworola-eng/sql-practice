# SQL Filtering: WHERE Operators

Notes from Data With Baraa SQL course, Module: Filtering Data

## What is Data Filtering?

Filtering means narrowing down a result set so that only the rows
matching a certain condition are returned, instead of every row in
the table. In SQL, filtering happens through the `WHERE` clause. The
condition you write after `WHERE` is evaluated for every row, and
only rows where the condition is true make it into the result.

There are several categories of operators used to build these
conditions: comparison operators, logical operators, range operators,
membership operators, and search (pattern matching) operators. Each
one solves a different kind of filtering problem.

---

## Comparison Operators

Comparison operators compare a column's value to a specific value.

- `=` equal to
- `!=` or `<>` not equal to
- `>` greater than
- `>=` greater than or equal to
- `<` less than
- `<=` less than or equal to

```sql
-- Retrieve all customers from Germany
SELECT *
FROM customers
WHERE country = 'Germany';

-- Retrieve all customers who are not from Germany
SELECT *
FROM customers
WHERE country != 'Germany';

-- Retrieve all customers with a score greater than 500
SELECT *
FROM customers
WHERE score > 500;

-- Retrieve all customers with a score of 500 or more
SELECT *
FROM customers
WHERE score >= 500;

-- Retrieve all customers with a score less than 500
SELECT *
FROM customers
WHERE score < 500;

-- Retrieve all customers with a score of 500 or less
SELECT *
FROM customers
WHERE score <= 500;
```

Note the difference between `>` and `>=`, and between `<` and `<=`.
The strict versions (`>`, `<`) exclude the boundary value itself,
while the "or equal to" versions include it. For example, `score > 500`
would not include a customer with a score of exactly 500, but
`score >= 500` would.

---

## Logical Operators

Logical operators combine multiple conditions together, or reverse
a condition entirely.

### AND

Returns rows only when **all** conditions are true at the same time.

```sql
-- Retrieve all customers who are from the USA and have a score greater than 500
SELECT *
FROM customers
WHERE country = 'USA' AND score > 500;
```

A row only appears in the result if both `country = 'USA'` and
`score > 500` are true for that row. If either condition fails, the
row is excluded.

### OR

Returns rows when **at least one** of the conditions is true.

```sql
-- Retrieve all customers who are either from the USA or have a score greater than 500
SELECT *
FROM customers
WHERE country = 'USA' OR score > 500;
```

This is broader than `AND`. A customer from Germany with a score of
600 would appear here (because the score condition is true), even
though they are not from the USA.

### NOT

Reverses a condition, returning rows where the condition is false.

```sql
-- Retrieve all customers with a score NOT less than 500
SELECT *
FROM customers
WHERE NOT score < 500;
```

This returns everyone whose score is 500 or greater, since it
excludes anyone for whom "score < 500" is true. `NOT` is useful when
it's easier to describe what you don't want than what you do want.

---

## Range Operator: BETWEEN

`BETWEEN` checks whether a value falls within a specified range,
inclusive of both endpoints.

```sql
-- Retrieve all customers whose scores fall in the range between 100 and 500
SELECT *
FROM customers
WHERE score >= 100 AND score <= 500;

-- Same result using BETWEEN
SELECT *
FROM customers
WHERE score BETWEEN 100 AND 500;
```

Both queries above return the same result. `BETWEEN 100 AND 500` is
shorthand for `>= 100 AND <= 500`. It is inclusive, meaning a score
of exactly 100 or exactly 500 is included in the result, not excluded.

`BETWEEN` is purely a readability improvement over writing out the
two comparisons with `AND`. It does not do anything that `AND`
couldn't already do, but it is easier to read and less error prone
for range conditions.

---

## Membership Operator: IN

`IN` checks whether a value matches any value in a given list. It is
a cleaner alternative to writing multiple `OR` conditions for the
same column.

```sql
-- Retrieve all customers from either Germany or USA
SELECT *
FROM customers
WHERE country IN ('Germany', 'USA');
```

Without `IN`, this would have to be written as:

```sql
WHERE country = 'Germany' OR country = 'USA'
```

Both produce the same result, but `IN` is shorter and scales much
better when checking against many values. If you needed to check
against ten countries, writing ten `OR` conditions would be tedious
and harder to read, while `IN` handles it cleanly in one list.

---

## Search Operator: LIKE (Pattern Matching)

`LIKE` is used to search for a pattern within text, rather than an
exact match. It works together with two wildcard characters:

- `%` represents any number of characters (including zero characters)
- `_` represents exactly one character

### Starts with a pattern

```sql
-- Find all customers whose first name starts with M
SELECT *
FROM customers
WHERE first_name LIKE 'M%';
```

The `%` after `M` means "M, followed by anything." This matches
names like "Max," "Maria," or just "M" on its own.

### Ends with a pattern

```sql
-- Find all customers whose first name ends with 'n'
SELECT *
FROM customers
WHERE first_name LIKE '%n';
```

The `%` before `n` means "anything, followed by n." This matches
names like "Susan" or "John."

### Contains a pattern anywhere

```sql
-- Find all customers whose first name contains 'r'
SELECT *
FROM customers
WHERE first_name LIKE '%r%';
```

Placing `%` on both sides means the letter `r` can appear anywhere in
the name, at the start, middle, or end, as long as it appears
somewhere.

### Pattern in a specific position

```sql
-- Find all customers whose first name has 'r' in the third position
SELECT *
FROM customers
WHERE first_name LIKE '__r%';
```

Each underscore `_` stands for exactly one character. `__r%` means
"any character, then any character, then the letter r, then anything
after that." This effectively checks that `r` is the third character
in the name.

---

## Key Takeaways

- Comparison operators (`=`, `!=`, `>`, `>=`, `<`, `<=`) compare a
  column to a specific value.
- Logical operators (`AND`, `OR`, `NOT`) combine or reverse
  conditions. `AND` requires all conditions to be true. `OR` requires
  at least one to be true. `NOT` reverses a condition.
- `BETWEEN` is shorthand for a range check using `>=` and `<=`
  together, and it is inclusive of both endpoints.
- `IN` checks membership in a list of values, replacing multiple `OR`
  conditions on the same column.
- `LIKE` matches text patterns using `%` (any number of characters)
  and `_` (exactly one character) as wildcards. The position of the
  wildcard determines whether you are matching the start, end,
  middle, or a specific character position within the text.
- Choosing the right operator is often about readability as much as
  correctness. `BETWEEN` and `IN` in particular exist mainly to make
  conditions that are already possible with `AND`/`OR` easier to read
  and less error prone.