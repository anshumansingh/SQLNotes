### Agenda
 - Delete
 - Delete vs truncate vs drop
 - Limit
 - Count
 - Order By
 - Join 


### Delete

```sql
DELETE FROM table_name WHERE conditions;
```

Example:

```sql
DELETE FROM film WHERE id = 1;
```

The above query will delete the row with `id` 1 from the `film` table. 

Beware, If you don't specify a where clause, then all the rows from the table will be deleted. Example:

```sql
DELETE FROM film;
```

Let's talk about how delete works as well in terms of code.

```python
for each row in film:
    if row.matches(conditions in where clause)
        delete row
```

There is a minor advance thing about DELETE which we shall talk about along with Joins in the next class. So, don't worry about it for now.

### Delete vs Truncate vs Drop

There are two more commands which are used to delete rows from a table. They are `TRUNCATE` and `DROP`. Let's discuss them one by one.

#### Truncate

The command looks as follows:

```sql
TRUNCATE film;
```

The above query will delete all the rows from the `film` table. TRUNCATE command internally works by removing the complete table and then recreating it. So, it is much faster than DELETE. But it has a disadvantage. It cannot be rolled back. We will learn more about rollbacks in the class on Transactions (In short, rollbacks can only happen for incomplete transactions - not committed yet - to be discussed in future classes). But at a high level, this is because as the complete table is deleted as an intermediate step, no log is maintained as to what all rows were deleted, and thus is not easy to revert. So, if you run a TRUNCATE query, then you cannot undo it. 

>Note: It also resets the primary key ID. For example, if the highest ID in the table before truncating was 10, then the next row inserted after truncating will have an ID of 1.

#### Drop

The command looks as follows:

Example:

```sql
DROP TABLE film;
```

The above query will delete the `film` table. The difference between `DELETE` and `DROP` is that `DELETE` is used to delete rows from a table and `DROP` is used to delete the entire table. So, if you run a `DROP` query, then the entire table will be deleted. All the rows and the table structure will be deleted. So, be careful while running a `DROP` query. Nothing will be left of the table after running a `DROP` query. You will have to recreate the table from scratch.

Note that,
DELETE:
1. Removes specified rows one-by-one from table (may delete all rows if no condition is present in query but keeps table structure intact). 
2. It is slower than TRUNCATE.
3. Doesn't reset the key. 
4. It can be rolled back.

TRUNCATE:
1. Removes the complete table and then recreats it.
2. Faster than DELETE.
3. Resets the key.
4. It can not be rolled back because the complete table is deleted as an intermediate step.

DROP:
1. Removes complete table and the table structre as well.
2. It can not be rolled back.

Rollback to be discussed in transactions class. Note that there is no undo in SQL queries once the query is completely committed. 


### LIKE Operator

LIKE operator is one of the most important and frequently used operator in SQL. Whenever there is a column storing strings, there comes a requirement to do some kind of pattern matching. Example, assume Scaler's database where we have a `batches` table with a column called `name`. Let's say we want to get the list of `Academy` batches and the rule is that an Academy batch shall have `Academy` somewhere within the name. How do we find those? We can use the `LIKE` operator for this purpose. Example:

```sql
SELECT * FROM batches WHERE name LIKE '%Academy%';
```

Similarly, let's say in our Sakila database, we want to get all the films which have `LOVE` in their title. We can use the `LIKE` operator. Example:

```sql
SELECT * FROM film WHERE title LIKE '%LOVE%';
```

Let's talk about how the `LIKE` operator works. The `LIKE` operator works with the help of 2 wildcards in our queries, `%` and `_`. The `%` wildcard matches any number of characters (>= 0 occurrences of any set of characters). The `_` wildcard matches exactly one character (any character). Example:

1. LIKE 'cat%' will match "cat", "caterpillar", "category", etc. but not "wildcat" or "dog".
2. LIKE '%cat' will match "cat", "wildcat", "domesticcat", etc. but not "cattle" or "dog".
3. LIKE '%cat%' will match "cat", "wildcat", "cattle", "domesticcat", "caterpillar", "category", etc. but not "dog" or "bat".
4. LIKE '_at' will match "cat", "bat", "hat", etc. but not "wildcat" or "domesticcat".
5. LIKE 'c_t' will match "cat", "cot", "cut", etc. but not "chat" or "domesticcat".
6. LIKE 'c%t' will match "cat", "chart", "connect", "cult", etc. but not "wildcat", "domesticcat", "caterpillar", "category".


### COUNT

Count function takes the values from a particular column and returns the number of values in that set. Umm, but don't you think it will be exactly same as the number of rows in the table? Nope. Not true. Aggregate functions only take not null values into account. So, if there are any null values in the column, they will not be counted.

Example: Let's take a students table with data like follows:

| id | name | age | batch_id |
|----|------|-----|----------|
| 1  | A    | 20  | 1        |
| 2  | B    | 21  | 1        |
| 3  | C    | 22  | null     |
| 4  | D    | 23  | 2        |

If you will try to run COUNT and give it the values in batch_id column, it will return 3. Because there are 3 not null values in the column. This is different from number of rows in the students table.

Let's see how do you use this operation in SQL.

```sql
SELECT COUNT(batch_id) FROM students;
```

To understand how aggregate functions work via a pseudocode, let's see how SQL query optimizer may execute them.

```python
table = []

count = 0

for row in table:
    if row[batch_id] is not null:
        count += 1

print(count)
```

Few things to note here:
While printing, do we have access to the values of row? Nope. We only have access to the count variable. So, we can only print the count. Extrapolating this point, when you use aggregate functions, you can only print the result of the aggregate function. You cannot print the values of the rows.

Eg:
    
```sql
SELECT COUNT(batch_id), batch_id FROM students;
```

This will be an invalid query. Because, you are trying to print the values of `batch_id` column as well as the count of `batch_id` column. But, you can only print the count of `batch_id` column.

### LIMIT Clause

And now let's discuss the last clause for the day. LIMIT clause allows us to limit the number of rows returned by a query. Example:

```sql
SELECT * FROM film LIMIT 10;
```

The above query will return only 10 rows from the `film` table. If you want to return 10 rows starting from the 11th row, you can use the `OFFSET` keyword. Example:

```sql
SELECT * FROM film LIMIT 10 OFFSET 10;
```

The above query will return 10 rows starting from the 11th row from the `film` table. 
Note that in MySQL, you cannot use the `OFFSET` keyword without the `LIMIT` keyword. Example:

```sql
SELECT * FROM film OFFSET 10;
```

throws an error. 

LIMIT clause is applied at the end. Just before printing the results. Taking the example of pseudocode, it works as follows:

```python
answer = []

for each row in film:
    if row.matches(conditions in where clause) # new line from above
        answer.append(row)

answer.sort(column_names in order by clause)

filtered_answer = []

for each row in answer:
    filtered_answer.append(row['rating'], row['release_year'])

return filtered_answer[start_of_limit: end_of_limit]
```

Thus, if your query contains ORDER BY clause, then LIMIT clause will be applied after the ORDER BY clause. Example:

```sql
SELECT * FROM film ORDER BY title LIMIT 10;
```

The above query will return 10 rows from the `film` table in ascending order of the `title` column.


### ORDER BY Clause

Now let's discuss another important clause. ORDER BY clause allows to return values in a sorted order. Example:

```sql
SELECT * FROM film ORDER BY title;
```

The above query will return all the rows from the `film` table in ascending order of the `title` column. If you want to return the rows in descending order, you can use the `DESC` keyword. Example:

```sql
SELECT * FROM film ORDER BY title DESC;
```

You can also sort by multiple columns. Example:

```sql
SELECT * FROM film ORDER BY title, release_year;
```

The above query will return all the rows from the `film` table in ascending order of the `title` column and then in ascending order of the `release_year` column. Consider the second column as tie breaker. If 2 rows have same value of title, release year will be used to break tie between them. Example:

```sql
SELECT * FROM film ORDER BY title DESC, release_year DESC;
```

Above query will return all the rows from the `film` table in descending order of the `title` column and if tie on `title`, in descending order of the `release_year` column.

By the way, you can ORDER BY on a column which is not present in the SELECT clause. Example:

```sql
SELECT title FROM film ORDER BY release_year;
```

Let's also build the analogy of this with a pseudocode.

```python
answer = []

for each row in film:
    if row.matches(conditions in where clause) # new line from above
        answer.append(row)

answer.sort(column_names in order by clause)

filtered_answer = []

for each row in answer:
    filtered_answer.append(row['rating'], row['release_year'])

return filtered_answer
```

If you see, the `ORDER BY` clause is applied after the `WHERE` clause. So, first the rows are filtered based on the `WHERE` clause and then they are sorted based on the `ORDER BY` clause. And only after that are the columns that have to be printed taken out. And that's why you can sort based on columns not even in the `SELECT` clause.

#### ORDER BY Clause with DISTINCT keyword

If you also have DISTINCT in the SELECT clause, then you can only sort by columns that are present in the SELECT clause. Example:

```sql
SELECT DISTINCT title FROM film ORDER BY release_year;
```

The above query will give an error. You can only sort by `title` column. Example:

```sql
SELECT DISTINCT title FROM film ORDER BY title;
```

Why this? Because without this the results can be ambiguous. Example:

```sql
SELECT DISTINCT title FROM film ORDER BY release_year;
```

The above query will return all the distinct titles from the `film` table. But which `release_year` should be used to sort them? There can be multiple `release_year` for a particular `title`. So, the results will be ambiguous.

### Joins

Every SQL query we had written till now was only finding data from 1 table. Most of the queries we had written in the previous classes were on the `film` table where we applied multiple filters etc. But do you think being able to query data from a single table is enough? Let's take a scenario of Scaler. Let's say we have 2 tables as follows in the Scaler's database:

`batches`

| batch_id | batch_name |
|----------|------------|
| 1        | Batch A    |
| 2        | Batch B    |
| 3        | Batch C    |

`students`

| student_id | first_name | last_name | batch_id |
|------------|------------|-----------|----------|
| 1          | John       | Doe       | 1        |
| 2          | Jane       | Doe       | 1        |
| 3          | Jim        | Brown     | 2        |
| 4          | Jenny      | Smith     | 3        |
| 5          | Jack       | Johnson   | 2        |

Suppose, someone asks you to print the name of every student, along with the name of their batch. The output should be something like:

| student_name | batch_name |
|--------------|------------|
| John     | Batch A    |
| Jane     | Batch A    |
| Jim    | Batch B    |
| Jenny  | Batch C    |
| Jack | Batch B    |

Will you be able to get all of this data by querying over a single table? No. The `student_name` is there in the students table, while the `batch_name` is in the batches table! We somehow need a way to combine the data from both the tables. This is where joins come in. What does the word `join` mean to you? 

Joins, as the name suggests, are a way to combine data from multiple tables. For example, if I want to combine the data from the `students` and `batches` table, I can use joins for that. Think of joins as a way to stitch rows of 2 tables together, based on the condition you specify. Example: In our case, we would want to stitch a row of students table with a row of batches table based on what? Imagine that every row of `students` I try to match with every row of `batches`. Based on what condition to be true between those will I stitch them?

We would want to stitch a row of students table with a row of batches table based on the `batch_id` column. This is what we call a `join condition`. A join condition is a condition that must be true between the rows of 2 tables for them to be stitched together. Let's see how we can write a join query for our example. 

```sql
SELECT students.first_name, batches.batch_name
FROM students
JOIN batches
ON students.batch_id = batches.batch_id;
```

Let's break down this query. The first line is the same as what we have been writing till now. We are selecting the `first_name` column from the `students` table and the `batch_name` column from the `batches` table. The next line is where the magic happens. We are using the `JOIN` keyword to tell SQL that we want to join the `students` table with the `batches` table. The next line is the join condition. We are saying that we want to join the rows of `students` table with the rows of `batches` table where the `batch_id` column of `students` table is equal to the `batch_id` column of `batches` table. This is how we write a join query. 

Let's take an example of this on the Sakila database. Let's say for every film, we want to print its name and the language. How can we do that?

```sql
SELECT film.title, language.name
FROM film
JOIN language
ON film.language_id = language.language_id;
```

Now, sometimes typing name of tables in the query can become difficult. For example, in the above query, we have to type `film` and `language` multiple times. To make this easier, we can give aliases to the tables. For example, we can give the alias `f` to the `film` table and `l` to the `language` table. We can then use these aliases in our query. Let's see how we can do that:

```sql
SELECT f.title, l.name
FROM film AS f
JOIN language AS l
ON f.language_id = l.language_id;
```

