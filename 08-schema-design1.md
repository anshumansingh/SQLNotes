## Agenda

- What is Schema Design
- What can go wrong
- Normalisation
- How to approach Schema Design
- Cardinality
    - How to find cardinality in relations
    - How to represent different cardinalities
- Nuances when representing relations

## What is Schema Design

Let's understand what Schema is. Schema refers to the structure of the database. Broadly speaking, schema gives information about the following:
- Structure of a database
- Tables in a database
- Columns in a table
- Primary Key
- Foreign Key
- Index
- Pictorial representation of how the DB is structured.

In general, 'Design' refers to the pictorial reference for solving how should something be formed considering the constraints. Prototyping, blueprinting or a plan, or structuring how something should exist is called Design.

Before any table or database is created for a software, a design document is formed consisting:
- Schema
- Class Diagram
- Architectural Diagram

## What can go wrong


#### Example 1:
Let's say Flipkart has a table for t-shirts. T-shirt has a column named color. 
Some t-shirts could have multiple colors. What do you put in the color column then? Maybe I put all the colors comma separated. 

So, something like, 

| tshirt_id | collar_type | size | color |
|-----------|-------------|------|-------|
| 1 | Round | M | red |
| 2 | Round | L | red, green | 
| 3 | Round | L | blue, red | 

How do you find all t-shirts of color red here. 

```sql
SELECT * FROM tshirt WHERE color LIKE "%red%"
```

The above query is going to do full-text search on color. You'll not be able to make it fast as it cannot leverage the power of indexing. 
And for that reason, some of your queries will always be slow. 


#### Example 2: 

Let's say we want to store classes and their instructor. Instead of creating 2 separate tables, I choose to put all information in one single table. 


| class_id | topic | instructor_id | instructor_name | Instructor_email |
|----------|-------|---------------|-----------------|----------------|
| 1 | Transactions | 4 | Anshuman | abcd@abcd.com |
| 2 | Indexing  | 4 | Anshuman | abcd@abcd.com |
| 3 | Schema Design | 4 | Anshuman | abcd@abcd.com |
| 4 | SQL-1 | 6 | Ayush | ayush@abcd.com | 

This has the following problems:
 - Update problem: If name for Anshuman needs to be updated, it has to be updated in all 3 rows containing Anshuman. Missing even a single row causes inconsistency. 
 - Delete problem: If you delete the class #4, you end up loosing all infomation about the instructor Ayush. 
 - Insert problem: If a new instructor has been onboarded, there is no way to record their information. I cannot create a row with dummy entries. The only way to save their information is when they have a class assigned. 

Bad design. 
As you can see, if you start with bad design, it causes tons of issues around performance, data integrity in the future. If you design your schema well, 50% of the battle is won. Let's see principles used for good schema design. 

## Normalisation

Normalization is the process to eliminate data redundancy and enhance data integrity in the table. It is a systematic technique of decomposing tables to eliminate data redundancy (repetition) and undesirable characteristics like Insertion, Update, and Deletion anomalies.

To understand, if we are using the technique properly, various normalized forms are defined. Let's look at them one by one. 

### 1-NF

A table is referred to as being in its First Normal Form if atomicity of the table is 1.
Here, atomicity states that a single cell cannot hold multiple values. It must hold only a single-valued attribute.
The First normal form disallows the multi-valued attribute, composite attribute, and their combinations.

So, example 1 above is not in 1-NF form. 
However, if all your table columns contain atomic values, then your schema satisfies 1-NF form.

How do you solve example 1 to make it 1-NF? 
Create another table called tshirt_color and have a unique row for every tshirt-id, color combination. 

### 2-NF

A table is said to be in the second normal form if and only if:
 - The table is already in 1-NF form. 
 - If the proper subset of candidate key determines non-prime attribute, it is called partial dependency. A table should not have partial dependencies.

Let's see with an example (Example 2). 


| class_id | topic | instructor_id | instructor_name | Instructor_email |
|----------|-------|---------------|-----------------|----------------|
| 1 | Transactions | 4 | Anshuman | abcd@abcd.com |
| 2 | Indexing  | 4 | Anshuman | abcd@abcd.com |
| 3 | Schema Design | 4 | Anshuman | abcd@abcd.com |
| 4 | SQL-1 | 6 | Ayush | ayush@abcd.com | 

Here, instructor_name cannot alone decide the class_id or the topic or instructor_id. Various instructors could have the same name with different instructor ID. Hence, instructor_name is a non prime attribute. 
instructor_name can be derived from instructor_id which is a proper subset of the key (instructor_id alone cannot be the key). Hence, the above table violates 2-NF form. 

How do you solve to make it 2-NF? 
Only keep instructor_id in the table. Move all other instructor relalted parameters like instructor_name, instructor_email to another table where you have one entry for every unique instructor. 


## How to approach Schema Design

Let's learn about this using a familiar example. You are asked to build a software for Scaler which can handle some base requirements.

The requirements are as follows:
1. Scaler will have multiple batches.
2. For each batch, we need to store the name, start month and current instructor.
3. Each batch of Scaler will have multiple students.
4. Each batch has multiple classes.
5. For each class, store the name, date and time, instructor of the class.
6. For every student, we store their name, graduation year, University name, email, phone number. 
7. Every student has a buddy, who is also a student.
8. A student may move from one batch to another.
9. For each batch a student moves to, the date of starting is stored.
10. Every student has a mentor.
11. For every mentor, we store their name and current company name. 
12. Store information about all mentor sessions (time, duration, student, mentor, student rating, mentor rating).
13. For every batch, store if it is an Academy-batch or a DSML-batch.

Representation of schema doesn't matter. What matters is that you have all the tables needed to satisfy the requirements. Considering above requirements, how will you design a schema? Let's see the steps involved in creating the schema design.

Steps:
1. **Create the tables:** For this we need to identify the tables needed. To identify the tables,
    - Find all the nouns that are present in requirements.
    - For each noun, ask if you need to store data about that entity in your DB.
    - If yes, create the table; otherwise, move ahead.

    Here, such nouns are batches, instructors (if we just need to store instructor name then it will be a column in batches table. But if we need to store information about instructor then we need to make a separate table), students, classes, mentor, mentor session.
    
    Note that, a good convention about names:
Name of a table should be plural, because it is storing multiple values. Eg. 'mentor_sessions'. Name of a column is singular and in snake-case.

2. **Add primary key (id) and all the attributes** about that entity in all the tables created above.
    
    Expectation with the primary key is that:
    - It should rarely change. Because indexing is done on PK (primary key) and the data on disk is sorted according to PK. Hence, these are updated with every change in primary key.
    - It should ideally be a datatype which is easy to sort and has smaller size. Have a separate integer/big integer column called 'id' as a primary key. For eg. twitter's algorithm ([Snowflake](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake)) to generate the key (id) for every tweet.
    - A good convention to name keys is \<tablename>\<underscore>id. For example, 'batch_id'.

    Now, for writing attributes of each table, just see which attributes are of that entity itself. For `batches`, coulmns will be `name`, `start_month`. `current_instructor` will not be a column as we don't just want to store the name of current instructor but their details as well. So, it is not just one attribute, there will be a relation between `batches` and `instructors` table for this. So we will get these tables:

`batches`
| batch_id | name | start_month |
|----------|------|-------------|

`instructors`
| instructor_id | name | email | avg_rating |
|---------------|------|-------|------------|

`students`
| student_id | name | email | phone_number | grad_year | univ_name |
|------------|------|-------|--------------|-----------|-----------|

`classes`
| class_id | name | schedule_time |
|----------|------|---------------|

`mentors`
| mentor_id | name | company_name |
|-----------|------|--------------|

`mentor_sessions`
| mentor_session_id | time | duration | student_rating | mentor_rating |
|-------------------|------|----------|----------------|---------------|

3. **Representing relations:** For understanding this step, we need to look into cardinality.

## Cardinality

When two entities are related to each other, there is a questions: how many of one are related to how many of the other.

For example, for two tables students and batches, cardinality represents how many students are related to how many batches and vice versa.

- 1:1 cardinality means 1 student belongs to only 1 batch and 1 batch has only 1 students.
- 1:m cardinality means 1 student can belong to multiple batches and 1 batch has only 1 student.
- m:1 cardinality means 1 student belongs to only 1 batch and 1 batch can have multiple students.
- m:m cardinality means multiple students can belong to multiple batches, and vice versa.

In cardinality, `1` means an entity can be associated to 1 instance at max, [0, 1]. `m` means an entity can be associated with zero or more instances, [0, 1, 2, ... inf]

### Steps to calculate cardinality

If you want to calculate relationship between `noun1` and `noun2`, then you can do the following:
 - *Step 1:* If you take one example of `noun2`, how many noun1 are related to this example object. Output : Either `1` or `many`
 - *Step 2:* If you take one example of `noun1`, how many noun2 are related to this example object. Output : Either `1` or `many`
 
Take output from step1 (o1) and output from step2 (o2). o1:o2 is your relationship.

Let's take an example. 
What is the cardinality between employee and department. Assume that an employee can be part of only one department. 

 - Step 1: Example of department: Finance. How many employees can be part of Finance. Answer: **many**
 - Step 2: Example of employee: Sudhanshu. How many department can Sudhanshu be part of? Answer: **one**

So, answer = **many-to-one**

**Example 2:** What is the cardinality between ticket and seat in apps like bookMyShow? 

In one ticket, we can book multiple seats.
One seat can be booked in only 1 ticket.

So, the final cardinality between ticket and seat is **one-to-many**

**Example 3:**  Consider a monogamous community. What is the cardinality between husband and wife?

| husband | --- married to --- | wife |
| ------- | ------------------ | ---- |
| 1       | -->                | 1    |
| 1       | <--                | 1    |


In a monogamous community, 1 man is married to 1 woman and vice versa. Hence, the cardinality is **one-to-one**

**Example 4:** What is the cardinality between class and current instructor at Scaler?
Answer: many-to-one

## How to represent different cardinalities

When we have a 1:1 cardinality, the `id` column of any one relation can be used as an attribute in another relation. It is not suggested to include the both the respective `id` column of the two relations in each other because it may cause update anomaly in future transactions.

For 1:m and m:1 cardinalities, the `id` column of `1` side relation is included as an attribute in `m` side relation.

For m:m cardinalities, create a new table called a **mapping table** or **lookup table** which stores the ids of both tables according to their associations.

For example, for tables `orders` and `products` in previous quiz have m:m cardinality. So, we will create a new table `orders_products` to accomodate the relation between order ids and products ids.

`orders_products`
| order_id | product_id |
| -------- | ---------- |
| 1        | 1          |
| 1        | 2          |
| 1        | 3          |
| 2        | 2          |
| 2        | 4          |
| 3        | 1          |
| 3        | 5          |
| 4        | 5          |


We will cover case studies for the next class - applying the principles learnt. 
