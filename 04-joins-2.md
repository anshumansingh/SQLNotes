
## Agenda

 - Self Join
 - More problems on Joins
 - Inner vs Outer joins
 - WHERE vs ON
 - Union and Union All 


## Self Join

Let's say at Scaler, for every student we assign a Buddy. For this we have a `students` table, which looks as follows:

`id | name | buddy_id`

This `buddy_id` will be an id of what?

Correct. Now, let's say we have to print for every student, their name and their buddy's name. How will we do that? Here 2 rows of which tables would we want to stitch together to get this data?

Correct, an SQL query for the same shall look like:

```sql
SELECT s1.name, s2.name
FROM students s1
JOIN students s2
ON s1.buddy_id = s2.id;
```

This is an example of SELF join. A self join is a join where we are joining a table with itself. In the above query, we are joining the `students` table with itself. In a self joining, aliasing tables is very important. If we don't alias the tables, then SQL will not know which row of the table to match with which row of the same table (because both of them have same names as they are the same table only).

### SQL query as pseudocode

As we have been doing since the CRUD class, let's also see how Joins can be represented in terms of pseudocode.

Let's take this query:

```sql
SELECT s1.name, s2.name
FROM students s1
JOIN students s2
ON s1.buddy_id = s2.id;
```

In pseudocode, it shall look like:

```python3
ans = []

for row1 in students:
    for row2 in students:
        if row1.buddy_id == row2.id:
            ans.add(row1 + row2)

for row in ans:
    print(row.name, row.name)
```

## More problems on JOIN

### Joining multiple tables

Till now, we had only joined 2 tables. But what if we want to join more than 2 tables? Let's say we want to print the name of every film, along with the name of the language and the name of the original language. How can we do that? If you have to add 3 numbers, how do you do that?

To get the name of the language, we would first want to combine film and language table over the `language_id` column. Then, we would want to combine the result of that with the language table again over the `original_language_id` column. This is how we can do that:

```sql
SELECT f.title, l1.name, l2.name
FROM film f
JOIN language l1
ON f.language_id = l1.language_id
JOIN language l2
ON f.original_language_id = l2.language_id;
```

Let's see how this might work in terms of pseudocode:

```python3
ans = []

for row1 in film:
    for row2 in language:
        if row1.language_id == row2.id:
            ans.add(row1 + row2)

for row in ans:
    for row3 in language:
        if row.language_id == row3.language_id:
            ans.add(row + row3)

for row in ans:
    print(row.name, row.language_name, row.original_language_name)
```

### Joins with multiple conditions in ON clause

Till now, whenever we did a join, we joined based on only 1 condition. Like in where clause we can combine multiple conditions, in Joins as well, we can have multiple conditions.

Let's see an example. For every film, name all the films that were released in the range of 2 years before or after that film and there rental rate was more than the rate of the movie.

```sql
SELECT f1.name, f2.name
FROM film f1
JOIN film f2
ON (f2.year BETWEEN f1.year - 2 AND f1.year + 2) AND f2.rental > f1.rental;
```

> Note:
> 1. Join does not need to happen on equality of columns always.
> 2. Join can also have multiple conditions.

A Compound Join is one where Join has multiple conditions on different columns.


## Inner vs Outer Joins

While we have pretty much discussed everything that is mostly important to know about joins, there are a few nitty gritties that we should know about.

Let's take the join query we had written a bit earlier:

```sql
SELECT s1.name, s2.name
FROM students s1
JOIN students s2
ON s1.buddy_id = s2.id;
```

Let's say there is a student that does not have a buddy, i.e., their `buddy_id` is null. What will happen in this case? Will the student be printed?

If you remember what we discussed about CRUD , is NULL equal to anything? Nope. Thus, the row will never match with anything and not get printed. The join that we discussed earlier is also called inner join. You could have also written that as:

```sql
SELECT s1.name, s2.name
FROM students s1
INNER JOIN students s2
ON s1.buddy_id = s2.id
```

The keyword INNER is optional. By default a join is INNER join.

As you see, an INNER JOIN doesn't include a row that didn't match the condition for any combination.

Opposite of INNER JOIN is OUTER JOIN. Outer Join will include all rows, even if they don't match the condition. There are 3 types of outer joins:
- Left Join
- Right Join
- Full Join

As the names convey, left join will include all rows from the left table, right join will include all rows from the right table and full join will include all rows from both the tables.

Let's take an example to understand these well:

Assume we have 2 tables: students and batches with following data:


`batches`

| batch_id | batch_name |
|----------|------------|
| 1        | Batch A    |
| 2        | Batch B    |
| 3        | Batch C    |

`students`

| student_id | name       | batch_id |
|------------|------------|----------|
| 1          | John       | 1        |
| 2          | Jane       | 1        |
| 3          | Jim        | null     |
| 4          | Ram        | null     |
| 5          | Sita       | 2        |

Now let's write queries to do each of these joins:

```sql
SELECT s.name, b.batch_name
FROM students s
LEFT JOIN batches b
ON s.batch_id = b.batch_id;
```

```sql
SELECT s.name, b.batch_name
FROM students s
RIGHT JOIN batches b
ON s.batch_id = b.batch_id;
```

```sql
SELECT s.name, b.batch_name
FROM students s
FULL OUTER JOIN batches b
ON s.batch_id = b.batch_id;
```


Now let's use different types of joins and tell me which row do you think will not be a part of the join.

Output of LEFT JOIN (Go row by row in left table - which is students and then look for match/matches):

```
John batchA
Jane batchA
Jim NULL
Ram NULL
Sita batchB
```

Output of RIGHT JOIN (Go row by row in right table - which is batches table and then look for match/matches):
batchA has 2 matches - John and Jane
batchB has 1 match - Sita
batchC has 0 match - NULL

```
John batchA
Jane batchA
Sita batchB
NULL batchC
```

Output of FULL JOIN (Do the left join. Then look at every row of right table which is `batches` and figure out rows which were not printed yet - print them with null match)

```
John batchA
Jane batchA
Jim NULL
Ram NULL
Sita batchB
NULL batchC
```

## Join with WHERE v/s ON

Let's take an example to discuss this. If we consider a simple query:
```sql
SELECT *
FROM A
JOIN B
ON A.id = B.id;
```
In pseudocode, it will look like:

```python3
ans = []

for row1 in A:
    for row2 in B:
        if (ON condition matches):
            ans.add(row1 + row2)

for row in ans:
    print(row.id, row.id)
```
Here, the size of intermediary table (`ans`) will be less than `n*m` because some rows are filtered.

We can also write the above query in this way:

```sql
SELECT *
FROM A, B
WHERE A.id = B.id;
```
The above query is nothing but a CROSS JOIN behind the scenes which can be written as:

```sql
SELECT *
FROM A
CROSS JOIN B
WHERE A.id = B.id;
```
Here, the intermediary table `A CROSS JOIN B` is formed before going to WHERE condition.

In pseudocode, it will look like:

```python3
ans = []

for row1 in A:
    for row2 in B:
        ans.add(row1 + row2)

for row in ans:
    if (WHERE condition matches):
        print(row.id, row.id)
```

The size of `ans` is always `n*m` because table has cross join of A and B. The filtering (WHERE condition) happens after the table is formed.

From this example, we can see that:
1. The size of the intermediary table (`ans`) is always greater or equal when using WHERE compared to using the ON condition. Therefore, joining with ON uses less internal space.
2. The number of iterations on `ans` is higher when using WHERE compared to using ON. Therefore, joining with ON is more time efficient.

In conclusion,
1. The ON condition is applied during the creation of the intermediary table, resulting in lower memory usage and better performance.
2. The WHERE condition is applied during the final printing stage, requiring additional memory and resulting in slower performance.
3. Unless you want to create all possible pairs, avoid using CROSS JOINS.

## UNION and UNION ALL

Sometimes, we want to print the combination of results of multiple queries. Let's take an example of the following tables:

`students`
| id | name |
|----|------|

`employees`
| id | name |
|----|------|

`investors`
| id | name |
|----|------|


You are asked to print the names of everyone associated with Scaler. So, in the result we will have one column with all the names.

We can't have 3 SELECT name queries because it will not produce this singular column. We basically need SUM of such 3 queries. Join is used to stitch or combine rows, here we need to add the rows of one query after the other to create final result.

UNION allows you to combine the output of multiple queries one after the other.

```sql
SELECT name FROM students
UNION
SELECT name FROM employees
UNION
SELECT name FROM investors;
```
Now, as the output is added one after the other, there is a constraint: Each of these individual queries should output the same number of columns.

Note that, you can't use ORDER BY for the combined result because each of these queries are executed independently.

UNION outputs distinct values of the combined result. **It stores the output of individual queries in a set and then outputs those values in final result. Hence, we get distinct values. But if we want to keep all the values, we can use UNION ALL. It stores the output of individual queries in a list and gives the output, so we get all the duplicate values.**

If you want to perform any operation on the combined result, you put them in braces and give it an alias. 
For example, 

```sql
SELECT
  first_name, last_name
FROM
(SELECT first_name, last_name
FROM customer

UNION

SELECT first_name, last_name
FROM actor) AS some_alias


ORDER BY first_name, last_name
LIMIT 10
```

