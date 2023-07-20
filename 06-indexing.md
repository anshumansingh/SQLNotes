## Agenda

- Introduction to Indexing
- How Indexes Work
- Indexes and Range Queries
    - Data structures used for indexing
- Cons of Indexes
- Indexes on Multiple Columns
- Indexing on Strings
- How to create index


## Introduction to Indexing

Hello Everyone

Till now, in the course, we had been discussing majorly about how to write SQL queries to fetch data we want to fetch. While discussing those queries, I also often wrote pseudocode talking about how at a higher level that query might work behind the scenes. 

Let us go back to that pseudocode. What do you think are some of the problems you see a user of DB will face if the DB really worked exactly how the pseudocode mentioned it worked?

Correct! In the pseudocode we had, for loops iterated over each row of the database to retrieve the desired rows. This resulted in a minimum time complexity of O(N) for every query. When joins or other operations are involved, the complexity further increases.

Adding to this, in which hardware medium is the data stored in a database?

Yes. A database stores its data in disk. Now, one of the biggest problems with disk is that accessing data from disk is very slow. Much slower than accessing data from RAM. For reference, read https://gist.github.com/jboner/2841832 Reading data from disk is 20x slower than reading from RAM! Let's talk about how data is fetched from the disk. We all know that on disk DB stores data of each row one after other. When data is fetched from disk, OS fetches data in forms of blocks. That means, it reads not just the location that you wnat to read, but also locations nearby.

First OS fetches data from disk to memory, then CPU reads from memory. Now imagine a table with 100 M rows and you have to first get the data for each row from disk into RAM, then read it. It will be very slow. Imagine you have a query like:

```sql
select * from students where id = 100;
```

To execute above, you will have to go through literally each row on the disk, and access even the blocks where this row doesn't exist. Don't you think this is a massive issue and can lead to performance problems?

To understand this better, let's take an example of a book. Imagine a big book covering a lot of topics. Now, if you want to find a particular topic in the book, what will you do? Will you start reading the book from the first page? No, right? You will go to the index of the book, find the page number of the topic you want to read, and then go to that page. This is exactly what indexing is. As index of a book helps go to the correct page of the book fast, the index of a database helps go to the correct block of the disk fast.

Now this is a very important line. Many people say that an index sorts a table. Nope. It has nothing to do with sorting. We will go over this a bit later in today's class. The major problem statement that indexes solve is to reduce the number of disk block accesses to be done. By preventing wastefull disk block accesses, indexes are able to increase performance of queries.

## How Indexes Work

While we have talked about the problem statement that indexes help solve, let's talk about how indexes work behind the scenes to optimize the queries. Let's try to build indexes ourselves. Let's imagine a huge table with 100s of millions of rows in table spread across 100s of disk blocks. We have a query like:

```sql
select * from students where id = 100;
```

We want to somehow avoid going to any of the disk block that is definitely not going to have the student with id 100. I need something that can help me directly know that hey, the row with id 100 is present in this block. Are you familiar with a data structure that can be stored in memory and can quickly provide the block information for each ID?

Correct. A map or a hashtable, whatever you call it can help us. If we maintain a hashmap where key is the id of the student and value is the disk block where the row containing that ID is present, is it going to solve our problem? Yes! That will help. Now, we can directly go to the block where the row is present and fetch the row. This is exactly how indexes work. They use some other data structure, which we will come to later.

Here we had queries on id. An important thing about `id` is that id is?

Yes. ID is unique. Will the same approach work if the column on which we are querying may have duplicates? Like multiple rows with same value of that column? Let's see. Let's imagine we have an SQL query as follows:

```sql
select * from students where name = 'Naman';
```

How will you modify your map to be able to accomodate multiple rows with name 'Naman'? 

Yes. What if we modify our map a bit. Now our keys will be String (name) and values will be a list of blocks that contain that name. Now, for a query, we will first go to the list of blocks for that name, and then go to each block and fetch the rows. This way as well, have we avoided fetching the blocks from the disk that were useless? Yes! Again, this will ensure our performance speeds up.

## Indexes and Range Queries

So, is this all that is there about indexes? Are they that simple? Well, no. Are the SQL queries you write always like `x = y`? What other kind of queries you often have to do in DB?

If a HashMap is how an index works, do you think it will be able to take care of range queries? Let's say we have a query like:

```sql
select * from students where psp between 40.1 and 90.1;
```

Will you be able to use hashmap to get the blocks that contain these students? Nope. A hashmap allows you to get a value in O(1) but if you have to check all the values in the range, you will have to check them 1 by 1 and potentially it will take O(N) time.

Hmm. So how will we solve it. Is there any other type of Map you know? Something that allow you to iterate over the values in a sorted way?

Correct. There is another type of Map called TreeMap. 

Let's in brief talk about the working of a TreeMap. For more detailed discussion, revise your DSA classes. A TreeMap uses a Balanced Binary Search Tree (often AVL Tree or Red Black Tree) to store data. Here, each node contains data and the pointers to left and right node.

Now, how will a TreeMap help us in our case? A TreeMap allows us to get the node we are trying to query in O(log N). From there, we can move to the next biggest value in O(log N). Thus, queries on range can also be solved.

## B and B+ Trees

Databases also use a Tree like data structure to store indexes. But they don't use a TreeMap. They use a B Tree or a B+ Tree. Here, each node can have multiple children. This helps further reduce the height of the tree, making queries faster. We will learn about these later.

## Cons of Indexes

While we have seen how indexes help make the read queries faster, like everything in engineering, they also have their cons. Let's try to think of those. What are the 4 types of operations we can do on a database? Out of these, which operations may require us to do extra work because of indexes?

Yes, whenever we update data, we may also have to update the corresponding nodes in the index. This will require us to do extra work and thus slow down those operations. 

Also, do you think we can store index only on memory? Well technically yes, but memory is volatile. If something goes wrong, we may have to recreate complete index again. Thus, often a copy of index is also stored on disk. This also requires extra space. There are two big problems that can arise with the use of index:
1. Writes will be slower
2. Extra storage


Thus, it is recommended to use index if and only if you see the need for it. Don't create indexes prematurely. 

## Indexes on Multiple Columns

How do you decide on which columns to create an index? Let's revisit how an index works. If I create an index on the `id` column, the tree map used for storing data will allow for faster retrieval based on that column. However, a query like: 

```sql
select * from students where psp = 90.1;
```

will not be faster with this index. The index on `id` has no relevance to the `psp` column, and the query will perform just as slowly as before. Therefore, we need to create an index on the column that we are querying.

We can also create index on 2 columns. Imagine a students table with columns like:

`id | name | email | batch_id | psp |`

We are writing a query like this:

```sql
select * from students where name = 'Naman';
```

Let's say I create an index on (id, name). Let's see how the index will look like:

When create index on these 2 columns, it is indexed according to id first and then if there is a tie it, will be indexed on name. So, there can be a name with different ids and we will not be able to filter it, as name is just a tie breaker here.

Thus, if we create an index on (id, name), it will actually not help us on the filter of name column. 

## Indexing on Strings

Now let's think of a scenario. How often do we need to use a query like this:

```sql
SELECT * FROM user WHERE email = 'abc@scaler.com';
```

But this query is very slow, so we will definitely create an index on the email column. So, the map that is created behind the scenes using indexing will have email mapped to the corresponding block in the memory.

Now, instead of creating index on whole email, we can create an index for the first part of the email (text before @) and have list of blocks (for more than one email having same first part) mapped to it. Hence, the space is saved.

Typically, with string columns, index is created on prefix of the column instead of the whole column. It gives enough increase in performance.

Consider the query:
```sql
SELECT * FROM user
WHERE address LIKE '%ambala%';
```

We can see that indexing will not help in such queries for pattern matching. In such cases, we use Full-Text Index about which we will discuss later.

## How to create index

Let's look at the syntax using `film` table:
```sql
CREATE INDEX idx_film_title_release
ON film(title, release_year);
```

Good practices for creating index:
1. Prefix the index name by 'idx'
2. Format for index name - idx\<underscore>\<table name>\<underscore>\<attribute name1>\<underscore>\<attribute name2>...


Now, let's use the index in a query:
```sql
EXPLAIN ANALYZE SELECT * FROM film
WHERE title = 'Shawshank Redemption';
```

If you look at the log of this query, "Index lookup on film using idx_film_title_release" is printed. If we remove the index and run the above query again, we can see that the time in executing the query is different. In case where indexing is not used, it takes more time to execute and more rows are searched to find the title.

