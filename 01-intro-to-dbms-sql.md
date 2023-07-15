# Lecture 1: Introduction to DBMS and SQL

## Agenda

- What is a Database
- What, What Not, Why, How of Scaler SQL Curriculum
- Types of Databases
- Intro to Relational Databases
- Intro to Keys

## What is a Database

Okay, tell me, In your day to day life whenever you have a need to save some information, where do you save it? Especially when you may need to refer to it later, maybe something like your expenses for the month, or your todo or shopping list?

Correct! Many of us use softwares like Excel, Google Sheets, Notion, Notes app etc to keep a track of things that are important for us and we may need to refer to it in future. Everyone, be it humans or organizations, have need to store a lot of data that me useful for them later. Example, let's think about Scaler. At Scaler, we would want to keep track of all of your's attendance, assignments solved, codes written, coins, mentor session etc! We would also need to store details about instructors, mentors, TAs, batches, etc. And not to forget all of your's email, phone number, password. Now, where will we do this?

For now, forget that you know anything about databases. Imagine yourself to be a new programmer who just knows how to write code in a programming language. Where will you store data so that you are able to retrieve it later and process that?

Correct! You will store it in files. You will write code to read data from files, and write data to files. And you will write code to process that data. For example you may create separate CSV (comma separated values, you will understand as we proceed) files to store information about let's say students, instructors, batches. Eg:

```
students.csv
name, batch, psp, attendance, coins, rank
Naman, 1, 94, 100, 0, 1
Amit, 2, 81, 70, 400, 1
Aditya, 1, 31, 100, 100, 2
```

```instructors.csv
name, subjects, average_rating
Rachit, C++, 4.5
Rishabh, Java, 4.8
Aayush, C++, 4.9
```

```batches.csv
id, name, start_date, end_date
1, AUG 22 Intermediate, 2022-08-01, 2023-08-01
2, AUG 22 Beginner, 2022-08-01, 2023-08-01
```

Now, let's say you want to find out the average attendance of students in each batch. How will you do that? You will have to write code to read data from students.csv, and batches.csv, and then process it to find out the average attendance of students in each batch. Right? Do you think this will be very cumbersome?

### Issues with Files as a Database

1. Inefficient

While the above set of data is very small in size, let's think of actual Scaler scale. We have 2M+ users in our system. Imagine going through a file with 2M lines, reading each line, processing it to find your relevant information. Even a very simple task like finding the psp of a student named Naman will require you to open the file, read each line, check if the name is Naman, and then return the psp. Time complexity wise, this is O(N) and very slow.

2. Integrity

Is there anyone stopping you from putting a new line in `students.csv` as ```Naman, 1, Hello, 100, 0, 1``` . If you see that `Hello` that is unexpected. The psp can't be a string. But there is no one to validate and this can lead to very bad situations. This is known as data integrity issue, where the data is not as expected.

3. Concurrency

Later in the course, you will learn about multi-threading and multi-processing. It is possible for more than 1 people to query about the same data at the same time. Similarly, 2 people may update the same data at the same time. On save, whose version should you save? Imagine you give same Google Doc to 2 people and both make changes on the same line and send to you. Whose version will you consider to be correct? This is known as concurrency issue.

4. Security

Earlier we talked about storing password of users. Imagine them being stored on files. Anyone who has access to the file can see the password of all users. Also anyone who has access to the file can update it as well. THere is no authorization at user level. Eg: a particular person may be only allowed to read, not write. '

### What's a Database

Now let's get back to our main topic. What is a database? A database is nothing but a collection of related data. Example, Scaler will have a Database that stores information about our students, users, batches, classes, instructors, and everything else. Similarly, Facebook will have a database that stores information about all of it's users, their posts, comments, likes, etc. The above way of storing data into files was also nothing but a database, though not the easiest one to use and with a lot of issues.

### What's a Database Management System (DBMS)

A DBMS as the name suggests is a software system that allows to efficiently manage a database. A DBMS allows us to create, retrieve, update, and delete data (often also called CRUD operations). It also  allows to define rules to ensure data integrity, security, and concurrency. It also provides ways to query the data in the database efficiently. Eg: find all students with psp > 50, find all students in batch 1, find all students with rank 1 in their batch, etc. There are many database management systems, each with their tradeoffs. We will talk about the types of databases later today.

Now let me talk about how the curriculum is structured. We will be having 11 live classes, followed by a contest. In the live classes we are going to cover everything that is important for you for interviews as well as day to day job. Having said that, as I mentioned that Databases are  avery vast field, attached with every live class you will be able to see a set of recorded videos curated for you for extra learning you can gain. Those videos will mostly be around theoretical concepts behind databases that are not much asked in interviews much nor used at day to day job, but good to know. Sometimes these videos will be on solving extra problems for a particular SQL Topic. You can go through those videos if you want to gain deeper understanding but aren't mandatory to solve assignments or clear contest.

| S.No | Lecture Title |
| --- | --- |
| 1 | Introduction to Databases and SQL |
| 2 | CRUD |
| 3 | CRUD-2 and Joins |
| 4 | Joins - 2 |
| 5 | Aggregate and Subqueries |
| 6 | Indexes |
| 7 | Transactions |
| 8 | Schema Design - 1 |
| 9 | Schema Design - 2 |
| 10 | Views and Window Functions |
| CONTEST | - |

I hope this excites you for your learning journey in this module and I wish you are able to make the most out of it. 

## Types of Databases

Welcome back after the break. Hope you had a good rest and had some water, etc. Now let's start with the next topic for the day and discuss different types of databases that exist. Okay, tell me one thing, when you have to store some data, eg let's say you are an instructor at Scaler and want to keep a track of attendance and psp of every student of you, in what form will you store that?


Correct! Often one of the easiest and most intuitive way to store data can be in forms of tables. Example for the mentioned use case, I may create a table with 3 columns: name, attendance, psp and fill values for each of my students there. This is very intuitive and simple and is also how relational databases work.

### Relational Databases

Relational Databases allow you to represent a database as a collection of multiple related tables. Each table has a set of columns and rows. Each row represents a record and each column represents a field. Example, in the above case, I may have a table with 3 columns: name, attendance, psp and fill values for each of my students there. Let's learn some properties of relational databases.

1. Relational Databases represent a database as a collection of tables with each table storing information about something. This something can be an entity or a relationship between entities. Example: I may have a table called students to store information about students of my batch (an entity). Similarly I may have a table called student_batches to store information about which student is in which batch (a relationship betwen entities).
2. Every row is unique. This means that in a table, no 2 rows can have same values for all columns. Example: In the students table, no 2 students can have same name, attendance and psp. There will be something different for example we might also want to store their roll number to distingusih 2 students having the same name.
3. All of the values present in a column hold the same data type. Example: In the students table, the name column will have string values, attendance column will have integer values and psp column will have float values. It cannot happen that for some students psp is a String.
4. Values are atomic. What does atomic mean? What does the word `atom` mean to you?

Correct. Similarly, atomic means indivisible. So, in a relational database, every value in a column is indivisible. Example: If we have to store multiple phone numbers for a student, we cannot store them in a single column as a list. How to store those, we will learn in the end of the course when we do Schema Design. Having said that, there are some SQL databases that allow you to store list of values in a column. But that is not a part of SQL standard and is not supported by all databases. Even those that support, aren't most optimal with queries on such columns.

5. The columns sequence is not guaranteed. This is very important. SQL standard doesn't guarantee that the columns will be stored in the same sequence as you define them. So, if you have a table with 3 columns: name, attendance, psp, it is not guaranteed that the data will be stored in the same sequence. So it is recommended to not rely on the sequence of columns and always use column names while writing queries. While MySQL guaranteees that the order of columns shall be same as defined at time of creating table, it is not a part of SQL standard and hence not guaranteed by all databases and relying on order can cause issues if in future a new column is added in between.

6. The rows sequence is not guaranteed. Similar to columns, SQL doesn't guarantee the order in which rows shall be returned after any query. So, if you want to get rows in a particular order, you should always use `ORDER BY` clause in your query which we will learn about in the next class. So when you write an SQL query, don't assume that the first row will always be the same. The order of rows may change across multiple runs of same query. Having said that, MySQL does return rows in order of their primary key (we will learn about this later today), but again, don't rely on that as not guaranteed by SQL standard.
  
7. The name of every column is unique. This means that in a table, no 2 columns can have same name. Example: In the students table, I cannot have 2 columns with name `name`. This is because if I have to write a query to get the name of a student, I will have to write `SELECT name FROM students`. Now if there are 2 columns with name `name`, how will the database know which one to return? Hence, the name of every column is unique.


### Non-Relational Databases

Now that we have learnt about relational databases, let's talk about non-relational databases. Non-relational databases are those databases that don't follow the relational model. They don't store data in form of tables. Instead, they store data in form of documents, key-value pairs, graphs, etc. In the DBMS module, we will not be talking about them. We will talk about them in the HLD Module. 

In the DBMS module, our goal is to cover the working of relational databases and how to work with them, that is via SQL queries. 

## Keys in Relational Databases

Now we are moving to probably the most important foundational concept of Relational Databases: Keys. let's say you are working at Scaler and are maintaining a table of every students' details. Someone tells you to update the psp of Naman to 100. How will you do that? What can go wrong?

What if there are 2 Namans?
 
Correct. If there are 2 Namans, how will you know which one to update? This is where keys come into picture. Keys are used to uniquely identify a row in a table. There are 2 important types of keys: Primary Key and Foreign Key. There are also other types of keys like Super Key, Candidate Key etc. Let's learn about them one by one.

### Super Keys

To understand this, let's take an example of a students table at scaler with following columns.

| name | psp | email | batch | phone number |
| - | - |- | - | - |
|Naman | 1 | 94 | 100 | 0 | 1 |
|Amit |  2 | 81 |  70 | 400 | 1 |
|Aditya| 1| 31| 100| 100| 2 |

Which are the columns that can be used to uniquely identify a row in this table?

Let's start with name. How many of you think name can be used to uniquely identify a row in this table? 

Correct. Name is not a good idea to recognize a row. Why? Because there can be multiple students with same name. So, if we have to update the psp of a student, we cannot use name to uniquely identify the student. Email, phone number on the other hand are a great idea, assuming no 2 students have same email, or same phone number. 

Do you think the value of combination of columns (name, email) can uniquely identify a student? Do you think there will be only 1 student with a particular combination of name and email. Eg: will there be only 1 student like (Naman, naman@scaler.com)?

Correct, similarly do you think (name, phone number) can uniquely identify a student? What about (name, email, phone number)? What about (name, email, psp)? What about (email, psp)?

The answer to each of the above is Yes. Each of these can be considered a `Super Key`. A super key is a combination of columns whose values can uniquely identify a row in a table. What do you think are other such super keys in the students table?

In the above keys, did you ever feel something like "but this column was useless to uniquely identify a row.." ? Let's take example of (name, email, psp). Do you think psp is required to uniquely identify a row? Similarly, do you think name is required as you anyways have email right? 

Now let's remove the columns that weren't necessary. 

Also, let's say we were an offline school, and students don't have email or phone number. In that case what do you think schools use to uniquely identify a student? Eg: If we remove redundant columns from (name, email, psp), we will be left with (email). Similarly, if we remove redundant columns from (name, email, phone number), we will be left with (phone number) or (email). These are known as `candidate keys`. A candidate key is a super key from which no column can be removed and still have the property of uniquely identifying a row. If any more column is removed from a candidate key, it will no longer be able to uniquely identify a row. Let's take another example. Consider a table Scaler has for storing students's attendance for every class

| student_id | class_id | attendance |

What do you think are the candidate keys for this table? Do you think (student_id) is a candidate key? Will there be only 1 row with a particular student_id?

Is (class_id) a candidate key? Will there be only 1 row with a particular class_id?

Is (student_id, class_id) a candidate key? Will there be only 1 row with a particular combination of student_id and class_id?

Yes! (student_id, class_id) is a candidate key. If we remove any of the columns of this, the remanining part is not a candidate key. Eg: If we remove student_id, we will be left with (class_id). But there can be multiple rows with same class_id. Similarly, if we remove class_id, we will be left with (student_id). But there can be multiple rows with same student_id. Hence, (student_id, class_id) is a candidate key.

Is (student_id, class_id, attendance) a candidate key? Will there be only 1 row with a particular combination of student_id, class_id and attendance?

But can we remove any column from this and still have a candidate key? Eg: If we remove attendance, we will be left with (student_id, class_id). This is a candidate key. Hence, (student_id, class_id, attendance) is not a candidate key.

### Primary Key

We just learnt about super keys and candidate keys. Can 1 table have mulitiple candidate keys? Yes. The table earlier had both (email), (phone number) as candidate keys. A key in MySQL plays a very important role. Example, MySQL orders the data in disk by the key. Similarly, by default, it returns answers to queries ordered by key. Thus, it is important that there is only 1 key. And that is called `primary key`. A primary key is a candidate key that is chosen to be the key for the table. In the students table, we can choose (email) or (phone number) as the primary key. Let's choose (email) as the primary key.

Sometimes, we may have to or want to create a new column to be the primary key. Eg: If we have a students table with columns (name, email, phone number), we may have to create a new column called roll number or studentId to be the primary key. This may be because let's say a user can change their email or phone number. Something that is used to uniquely identify a row should ideally never change. Hence, we create a new column called roll number or studentId to be the primary key.

We will see later today on how MySQL allows to create primary keys etc. Before we go to foreign keys and composite keys, let's actually get our hands dirt with SQL. 

### Foreign Keys

Now let's get to the last topic of the day. Which is foreign keys. Let's say we have a table called batches which stores information about batchesat Scaler. It has columns (id, name, startDate, endDate). We would want to know for every student, which batch do they belong to. How can we do that?

Correct, We can add batchId column in students table. But how do we know which batch a student belongs to? How do we ensure that the batchId we are storing in the students table is a valid batchId? What if someone puts the value in batchID column as 4 but there is no batch with id 4 in batches table. We can set such kind of constraints using foreign keys. A foreign key is a column in a table that references a column in another table. It has nothing to do with primary, candidate, super keys. It can be any column in 1 table that refers to any column in other table. In our case, batchId is a foreign key in the students table that references the id column in the batches table. This ensures that the batchId we are storing in the students table is a valid batchId. If we try to insert any value in the batchID column of students table that isn't present in id column of batches table, it will fail. ANother example:

Let's say we have `years` table as:
`| id | year | number_of_days |`

and we have a table students as:
`| id | name | year |`

Is `year` column in students table a foreign key? 

The correct answer is yes. It is a foreign key that references the id column in years table. Again, foreign key has nothing to do with primary key, candidate key etc. It is just any column on one side that references another column on other side. Though often it doesn't make sense to have that and you just keep primary key of the other table as the foreign key. If not a primary key, it should be a column with unique constraint. Else, there will be ambiguities.

Okay, now let's think of what can go wrong with foreign keys?

Correct, let's say we have students and batches tables as follows:

| batch_id | batch_name |
|----------|------------|
| 1        | Batch A    |
| 2        | Batch B    |
| 3        | Batch C    |


| student_id | first_name | last_name | batch_id |
|------------|------------|-----------|----------|
| 1          | John       | Doe       | 1        |
| 2          | Jane       | Doe       | 1        |
| 3          | Jim        | Brown     | 2        |
| 4          | Jenny      | Smith     | 3        |
| 5          | Jack       | Johnson   | 2        |

Now let's say we delete the row with batch_id 2 from batches table. What will happen? Yes, the students Jim and Jack will be orphaned. They will be in the students table but there will be no batch with id 2. This is called orphaning. This is one of the problems with foreign keys. Another problem is that if we update the batch_id of a batch in batches table, it will not be updated in students table. Eg: If we update the batch_id of Batch A from 1 to 4, the students John and Jane will still have batch_id as 1. This is called inconsistency.

To fix for these, MySQL allows you to set ON DELETE constraints so that cascading deletes happen when such a delete happens. Implementation details will be discussed in the next set of classes.  

