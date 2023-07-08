# Lecture 2: CRUD

## What is CRUD?

Hello Everyone  
Today we are going to start the journey of learning MySQL queries by learning about CRUD Operations. Okay tell me one thing. Let's say there is a table in which we are storing information about students. What all can we do in that table or its entries?

Correct. Primarily, on any entity stored in a table, there are 4 operations possible:

1. Create (or inserting a new entry)
2. Read (fetching some entries)
3. Update (updating information about an entry already stored)
4. Delete (deleting an entry)

Today we are going to discuss about these operations in detail. Understand that read queries can get a lot more complex, involving aggregate functions, subqueries etc, which we shall talk about in detail in later classes. So don't worry about that. Take today's class as an introduction to the world of MySQL queries.

We will be starting with learning about Create, then go to Read, then Update and finally Delete. So let's get started. For today's class as well as most of the classes ahead, we will be using Sakila database, which is an official sample database provided by MySQL. I hope you have downloaded and set that up on your machine already, following instructions shared in the earlier classes.

## Sakila Database Walkthrough

Let me give you all a brief idea about what Sakila database represents so that it is easy to relate to the conversations that we shall have around this over the coming weeks. Sakila database represents a digital video rental store, assume an old movie rental store before Netflix etc came. It's designed with functionality that would allow for all the operations of such a business, including transactions like renting films, managing inventory, and storing customer and staff information. Example: it has tables regarding films, actors, customers, staff, stores, payments etc. You shall get more familiar with this in the coming classes, don't worry!

## Create

### CREATE Query

First of all, what is SQL? SQL stands for Structured Query Language. It is a language used to interact with relational databases. It allows you to create tables, fetch data from them, update data, manage user permissions etc. Today we will just focus on creation of data. Remaining things will be covered over the coming classes. Why "Structured Query" bceause it allows to query over data arranged in a structured way. Eg: In Relational databases, data is structured into tables. 

A simple query to create a table in MySQL has:
  - column names
  - data type of column (integer, varchar, boolean, date, timestamp)
  - properties of column (unique, not null, default)

Similarily, the table could also have properties (primary key, key)

```sql
CREATE TABLE students (
    id INT AUTO_INCREMENT,
    firstName VARCHAR(50) NOT NULL,
    lastName VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    dateOfBirth DATE NOT NULL,
    enrollmentDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    psp DECIMAL(3, 2) CHECK (psp BETWEEN 0.00 AND 100.00),
    batchId INT,
    isActive BOOLEAN DEFAULT TRUE,
    PRIMARY KEY (id),
);
```

Here we are creating a table called students. Inside brackets, we mention the different columns that this table has. Alongwith each columns, we mention the data type of that column. Eg: firstName is of type VARCHAR(50). Please do watch the video on SQL Data Types attached to today's class to understand what VARCHAR, TIMESTAMP etc means. For our today's discussion, it suffices to know that these are different data types supported by MySQL. After the data type, we mention any constraints on that column. Eg: NOT NULL means that this column cannot be null. In tomorrow's class when we will learn how to insert data, if we try to not put a value of this column, we will get an error. UNIQUE means that this column cannot have duplicate values. If we insert a new row in a table, or update an existing row that leads to 2 rows having same value of this column, the query will fail and we will get an error. DEFAULT specifies that if no value is provided for this column, it will take the given value. Example, for enrollmentDate, it will take the value of current_timestamp, which the time when you are inserting the row. CHECK (psp BETWEEN 0.00 AND 100.00) means that the value of this column should be between 0.00 and 100.00. If some other value is put, the query will fail.


### INSERT Query

Now let's start with the first set of operation for the day: The Create Operation. As the name suggests, this operation is used to create new entries in a table. Let's say we want to add a new film to the database. How do we do that?

`INSERT` statement in MySQL is used to insert new entries in a table. Let's see how we can use it to insert a new film in the `film` table of Sakila database.

```sql
INSERT INTO film (title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features) 
VALUES ('The Dark Knight', 'Batman fights the Joker', 2008, 1, 3, 4.99, 152, 19.99, 'PG-13', 'Trailers'),
       ('The Dark Knight Rises', 'Batman fights Bane', 2012, 1, 3, 4.99, 165, 19.99, 'PG-13', 'Trailers'),
       ('The Dark Knight Returns', 'Batman fights Superman', 2016, 1, 3, 4.99, 152, 19.99, 'PG-13', 'Trailers');
```

Let's dive through the syntax of the query. First we have the `INSERT INTO` clause, which is used to specify the table in which we want to insert the new entry. Then we have the column names in the brackets, which are the columns in which we want to insert the values. Then we have the `VALUES` clause, which is used to specify the values that we want to insert in the columns. The values are specified in the same order as the columns are specified in the `INSERT INTO` clause. So the first value in the `VALUES` clause will be inserted in the first column specified in the `INSERT INTO` clause, and so on.

A few things to note here:

1. The column names is optional. If you don't specify the column names, then the values will be inserted in the columns in the order in which they were defined at the time of creating the table. Example: in the above query, if we don't specify the column names, then the values will be inserted in the order `film_id`, `title`, `description`, `release_year`, `language_id`, `original_language_id`, `rental_duration`, `rental_rate`, `length`, `replacement_cost`, `rating`, `special_features`, `last_update`. So the value `The Dark Knight` will be inserted in the `film_id` column, `Batman fights the Joker` will be inserted in the `title` column and so on.  
   - This is not a good practice, as it makes the query prone to errors. So always specify the column names. 
   - This makes writing queries tedious as while writing query you have to keep a track of what column was where. And even a small miss can lead to a big error. 
   - Also, if you don't specify the column names, then you have to specify values for all the columns. If you don't want to specify values for all the columns, then you have to specify the column names. Example: if you don't specify column names, then you have to specify values for all the columns, including `film_id`, `original_language_id` and `last_update`, which we may want to keep `NULL`.

Anyways, an example of a query without column names is as follows:

```sql
INSERT INTO film
VALUES (default, 'The Dark Knight', 'Batman fights the Joker', 2008, 1, NULL, 3, 4.99, 152, 19.99, 'PG-13', 'Trailers', default);
```

NULL is used to specify that the value of that column should be `NULL`, and `default` is used to specify that the value of that column should be the default value specified for that column. Example: `film_id` is an auto-increment column, so we don't need to specify its value. So we can specify `default` for that column, which will insert the next auto-increment value in that column.

So that's pretty much all that's there about Create operations. There is 1 more thing about insert, which is how to insert data from one table to another, but we will talk about that after talking about read. Any doubts till now? Via thumbs up/ down can you all let me know how many of you are 100% clear till here?

So seems like all doubts are clear. Before I start with read operations, let me have 2 small Quiz questions for you.

## Read

Now let's get to the most interest, and also maybe most important part of today's session: Read operations. . `SELECT` statement is used to read data from a table. Let's see how we can use it to read data via different queries on the `film` table of Sakila database. A basic select query is as follows:

```sql
SELECT * FROM film;
```

Here we are selecting all the columns from the `film` table. The `*` is used to select all the columns. This query will give you the value of each column in each row of the film table. If we want to select only specific columns, then we can specify the column names instead of `*`. Example:

```sql
SELECT title, description, release_year FROM film;
```

Here we are selecting only the `title`, `description` and `release_year` columns from the `film` table. Note that the column names are separated by commas. Also, the column names are case-insensitive, so `title` and `TITLE` are the same. Example following query would have also given the same result:

```sql
SELECT TITLE, DESCRIPTION, RELEASE_YEAR FROM film;
```

Now, let's learn some nuances around the `SELECT` statement.

### Selecting Distinct Values

Let's say we want to select all the distinct values of the `rating` column from the `film` table. How do we do that? We can use the `DISTINCT` keyword to select distinct values. Example:

```sql
SELECT DISTINCT rating FROM film;
```

This query will give you all the distinct values of the `rating` column from the `film` table. Note that the `DISTINCT` keyword, as all other keywords in MySQL, is case-insensitive, so `DISTINCT` and `distinct` are the same.

We can also use the `DISTINCT` keyword with multiple columns. Example:

```sql
SELECT DISTINCT rating, release_year FROM film;
```

This query will give you all the distinct values of the `rating` and `release_year` columns from the `film` table. Let's talk about how this works. A lot of SQL queries can be easily understood by relating them to basic for loops etc. Over this class, and the coming classes, I will relate every complex query with a corresponding pseudo code has you tried to do the same in a programming language. As all of you have already solved many DSA problems, this shall be much more easy and fun for you to learn. BTW: At the end, I will also share a final diagram that relates an SQL query to a corresponding pseudo code, with select, aggregate, group by, having, order by, limit, join, subquery, etc.

So, let's try to understand the above query with a pseudo code. The pseudo code for the above query would be as follows:

```python
answer = []

for each row in film:
    answer.append(row)

filtered_answer = []

for each row in answer:
    filtered_answer.append(row['rating'], row['release_year'])

unique_answer = set(filtered_answer)

return unique_answer
```

So what you see is that DISTINCT keyword on multiple column gives you for all of the rows in the table, the distinct value of pair of these columns.

### Select statement to print a constant value

Let's say we want to print a constant value in the output. Eg: The first program that almost every programmer writes: "Hello World". How do we do that? We can use the `SELECT` statement to print a constant value. Example:

```sql
SELECT 'Hello World';
```

That's it. No from, nothing. Just the value. You can also combine it with other columns. Example:

```sql
SELECT title, 'Hello World' FROM film;
```

### Operations on Columns

Let's say we want to select the `title` and `length` columns from the `film` table. If you see, the value of length is currently in minutes, but we want to select the length in hours instead of minutes. How do we do that? We can use the `SELECT` statement to perform operations on columns. Example:

```sql
SELECT title, length/60 FROM film;
```

Later in the course we will learn about Built In functions in SQL as well. You can use those functions as well to perform operations on columns. Example:

```sql
SELECT title, ROUND(length/60) FROM film;
```

ROUND function is used to round off a number to the nearest integer. So the above query will give you the title of the film, and the length of the film in hours, rounded off to the nearest integer.

### Inserting Data from Another Table

BTW, select can also be used to insert data in a table. Let's say we want to insert all the films from the `film` table into the `film_copy` table. We can combine the `SELECT` and `INSERT INTO` statements to do that. Example:

```sql
INSERT INTO film_copy (title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features)
SELECT title, description, release_year, language_id, rental_duration, rental_rate, length, replacement_cost, rating, special_features
FROM film;
```

Here we are using the `SELECT` statement to select all the columns from the `film` table, and then using the `INSERT INTO` statement to insert the selected data into the `film_copy` table. Note that the column names in the `INSERT INTO` clause and the `SELECT` clause are the same, and the values are inserted in the same order as the columns are specified in the `INSERT INTO` clause. So the first value in the `SELECT` clause will be inserted in the first column specified in the `INSERT INTO` clause, and so on.

Okay, let's take a pause to answer any doubts anyone may be having till now. For those who are absolutely clear can you please do a thumbs up in the chat. If any doubt, can you please do a thumbs down and post the doubt in chat.

Okay, let me also verify how well you have learnt till now with a few quiz questions.

Till now, we have been doing basic read operations. SELECT query with only FROM clause is rarely sufficient. Rarely do we want to return all rows. Often we need to have some kind of filtering logic etc. for the rows that should be returned. Let's learn how to do that.

### WHERE Clause

Let's say we want to select all the films from the `film` table which have a rating of `PG-13`. How do we do that? We can use the `WHERE` clause to filter rows based on a condition. Example:

```sql
SELECT * FROM film WHERE rating = 'PG-13';
```

Here we are using the `WHERE` clause to filter rows based on the condition that the value of the `rating` column should be `PG-13`. Note that the `WHERE` clause is always used after the `FROM` clause. In terms of pseudocode, you can think of where clause to work as follows:

```python
answer = []

for each row in film:
    if row.matches(conditions in where clause) # new line from above
        answer.append(row)

filtered_answer = []

for each row in answer:
    filtered_answer.append(row['rating'], row['release_year'])

unique_answer = set(filtered_answer) # assuming we also had DISTINCT

return unique_answer
```

If you seem where clause can be considered analgous to `if` in a programming language. With if as well there are many other operators that are used, right. Can you name which operators do we often use in programming languages with `if`?

> NOTE: Wait for students to give answer. Give hints to get AND, OR, NOT from them.

### AND, OR, NOT

Correct. We use things like `and` , `or`, `!` in programming languages to combine multiple conditions. Similarly, we can use `AND`, `OR`, `NOT` operators in SQL as well. Example: We want to get all the films from the `film` table which have a rating of `PG-13` and a release year of `2006`. We can use the `AND` operator to combine multiple conditions.

```sql
SELECT * FROM film WHERE rating = 'PG-13' AND release_year = 2006;
```

Similarly, we can use the `OR` operator to combine multiple conditions. Example: We want to get all the films from the `film` table which have a rating of `PG-13` or a release year of `2006`. We can use the `OR` operator to combine multiple conditions.

```sql
SELECT * FROM film WHERE rating = 'PG-13' OR release_year = 2006;
```

Similarly, we can use the `NOT` operator to negate a condition. Example: We want to get all the films from the `film` table which do not have a rating of `PG-13`. We can use the `NOT` operator to negate the condition.

```sql
SELECT * FROM film WHERE NOT rating = 'PG-13';
```

An advice on using these operators. If you are using multiple operators, it is always a good idea to use parentheses to make your query more readable. Else, it can be difficult to understand the order in which the operators will be evaluated. Example:

```sql
SELECT * FROM film WHERE rating = 'PG-13' OR release_year = 2006 AND rental_rate = 0.99;
```

Here, it is not clear whether the `AND` operator will be evaluated first or the `OR` operator. To make it clear, we can use parentheses. Example:

```sql
SELECT * FROM film WHERE rating = 'PG-13' OR (release_year = 2006 AND rental_rate = 0.99);
```

Till now, we have used only `=` for doing comparisons. Like traditional programming languages, MySQL also supports other comparison operators like `>`, `<`, `>=`, `<=`, `!=` etc. Just one special case, `!=` can also be written as `<>` in MySQL. Example:

```sql
SELECT * FROM film WHERE rating <> 'PG-13';
```

### IN Operator

With comparison operators, we can only compare a column with a single value. What if we want to compare a column with multiple values? For example, we want to get all the films from the `film` table which have a rating of `PG-13` or `R`. One way to do that can be to combine multiple consitions using `OR`.  A better way will be to use the `IN` operator to compare a column with multiple values. Example:

```sql
SELECT * FROM film WHERE rating IN ('PG-13', 'R');
```

Okay, now let's say we want to get those films that have ratings anything oter than the above 2. Any guesses how we may do that?

Correct! We had earlier discussed about `NOT`. You can also use `NOT` before `IN` to negate the condition. Example:

```sql
SELECT * FROM film WHERE rating NOT IN ('PG-13', 'R');
```

Think of IN to be like any other operator. Just that it allows comparison with multiple values.

Hope you had a good break. Let's continue with the session. In this second part of the session, we are going to start the discussion by discussing about another important keyword in SQL, `BETWEEN`.

### IS NULL Operator

Now we are almost at the end of the discussion about different operators. Do you all remember how we store emptiess, that is, no value for a particular column for a particular row? We store it as `NULL`. Interestingly working with NULLs is a bit tricky. We cannot use the `=` operator to compare a column with `NULL`. Example:

```sql
SELECT * FROM film WHERE description = NULL;
```

The above query will not return any rows. Why? Because `NULL` is not equal to `NULL`. Infact, `NULL` is not equal to anything. Nor is it not equal to anything. It is just `NULL`. 

Example:

```sql
SELECT NULL = NULL;
```

The above query will return `NULL`. Similarly, `3 = NULL` , `3 <> NULL` , `NULL <> NULL` will also return `NULL`. So, how do we compare a column with `NULL`? We use the `IS NULL` operator. Example:

```sql
SELECT * FROM film WHERE description IS NULL;
```

Similarly, we can use the `IS NOT NULL` operator to find all the rows where a particular column is not `NULL`. Example:

```sql
SELECT * FROM film WHERE description IS NOT NULL;
```

In many assignments, you will find that you will have to use the `IS NULL` and `IS NOT NULL` operators. Without them you will miss out on rows that had NULL values in them and get the wrong answer. Example:
find customers with id other than 2. If you use `=` operator, you will miss out on the customer with id `NULL`. Example:

```sql
SELECT * FROM customers WHERE id != 2;
```

The above query will not return the customer with id `NULL`. So, you will get the wrong answer. Instead, you should use the `IS NOT NULL` operator. Example:

```sql
SELECT * FROM customers WHERE id IS NOT NULL AND id != 2;
```

## Update

Now let's move to learn U of CRUD. Update and Delete are thankfully much simple, so don't worry, we will be able to breeze through it over the coming 20 mins. As the name suggests, this is used to update rows in a table. The general syntax is as follows:

```sql
UPDATE table_name SET column_name = value WHERE conditions;
```

Example:

```sql
UPDATE film SET release_year = 2006 WHERE id = 1;
```

The above query will update the `release_year` column of the row with `id` 1 in the `film` table to 2006. You can also update multiple columns at once. Example:

```sql
UPDATE film SET release_year = 2006, rating = 'PG' WHERE id = 1;
```

Let's talk about how update works. It works as follows:

```python
for each row in film:
    if row.matches(conditions in where clause)
        row['release_year'] = 2006
        row['rating'] = 'PG'
```

So basically update query iterates through all the rows in the table and updates the rows that match the conditions in the where clause. So, if you have a table with 1000 rows and you run an update query without a where clause, then all the 1000 rows will be updated. So, be careful while running update queries. Example:

```sql
UPDATE film SET release_year = 2006;
```

## Delete

Finally, we are at the end of CRUD. Let's talk about Delete operations. The general syntax is as follows:

```sql
DELETE FROM table_name WHERE conditions;
```

Example:

```sql
DELETE FROM film WHERE id = 1;
```

The above query will delete the row with `id` 1 from the `film` table. If you don't specify a where clause, then all the rows from the table will be deleted. Example:

```sql
DELETE FROM film;
```

Let's talk about how delete works as well in terms of code.

```python
for each row in film:
    if row.matches(conditions in where clause)
        delete row
```

