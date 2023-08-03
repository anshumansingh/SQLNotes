## Agenda

- Scaler Schema Design - continued
- Deciding Primary Keys of a mapping table
- Representing Foreign keys and indexes
- Case Study - Schema design of Netflix


## Steps to design schema 

Let's go over the steps to design schema one more time. 
1. Figure out entities. Nouns from the given requirements need to be split between entities and attributes. 
2. Figure out relationships. Based on cardinality, we decide the tables/columns needed. 
3. Do we see attributes that should be enum instead? [Described later in the notes]
4. Find out the primary keys in every table. Add foreign key wherever applicable. 
5. Find out common query patterns and build index on them. Keep iterating as you discover more frequent use cases. 


## Scaler Schema Design

For reference from previous class, Scaler Schema Design:

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

## Step 1: Find entities (Nouns)

Let's use the above description to list out entities (nouns)
1. batches (attributes: name, start month)
2. instructors (name)
3. students (name, graduationYear, universityName, email, phoneNumber)
4. classes (name, date, time)
5. mentor (name, currentCompanyName)
6. mentor_sessions (time, duration, student_rating, mentor_rating)

Quick note here:
 - Not all nouns become entities. 
 - For example, name is a noun. But it's an attribute of a class and not a separate entity in itself. 

## Step 2: Relationships

1. Point number 2 in the requirement tells us there is a relationship between batch and current instructor. 
    Cardinality: m:1 
    Hence, we add current_instructor_id column to batches table. 

2. Point number 3 tells us that each batch can have multiple students. And a student is in one batch at a time. They can move, but at a time, they are exactly in one batch. So, there is a relationship between batch and students. 
    Cardinality: 1:m 
    Hence, we can add batch_id as a column in students table.
However, here comes the tricky part. Imagine, I want to track all dates when the student moved. Which means the relationship becomes many:many (current + historical batches). I might still have the batch_id in students table to indicate current batch, but I will need a separate table to maintain all historical batches along with their move date. 
When a student is moved from one batch to another, this date is an attribute of the relation between `students` and `batches`. So, we will create a new table like this:

`student_batches`

| student_id | batch_id | move_date |
|------------|----------|-----------|

As we have included `batch_id` here, we can remove it from `students` table but that will decrease the performance because everytime we will have to query on this new table also. So, for ease, we will keep the `batch_id` in `students` also.

3. Point number 4 tells us that each batch has multiple classes. Whether a class only has one batch or multiple batches is not specified. Let's assume, that we want the ability to have multiple batches attend a class. 
    Cardinality: m:m
    We will need a separate batch_classes table. 

4. Point number 5 tells us there is a relationship between classes and instructor. What is the cardinality between `class` and `instructor`. As this is m:1 cardinality, instructor_id will be included in `classes`.

5. Point number 7 tells us that every student has a buddy. 
Here, the cardinality of the buddy relation between a student and another student is m:1. 

| student | --- buddy --- | student |
| ------- | ------------- | ------- |
| 1       | -->           | 1       |
| m       | <--           | 1       |  > 

So, the `students` table will have one more column called `buddy_id`.

6. Point number 10 tells us that there is a relationship between student and mentor. A student has one mentor, but a mentor can have many students. 
   Cardinality: m:1
So, we will include mentor_id as a column in the students table. 

7. Finally, point number 12 tells us mentor_sessions has a relationship with `mentors` and `students`. 
    Cardinality between `mentor_sessions` and `students` : m:1
    Cardinality between `mentor_sessions` and `mentors`  : m:1
    So, we add `student_id` and `mentor_id` columns to `mentor_sessions` table. 

**Hence, the final table structure after step 1 and 2:**

`batches`

| batch_id | name | start_month | curr_inst_id |
|----------|------|-------------|--------------|

`instructors`
| instructor_id | name |
| ------------- | ---- | 

`classes`

| class_id | name | schedule_time | instructor_id |
|----------|------|---------------| ------------- |

`batch_classes`

| batch_id | class_id |
| -------- | -------- | 

`student_batches`

| student_id | batch_id | move_date |
|------------|----------|-----------|

`students`

| student_id | name | email | phone_number | grad_year | univ_name | batch_id | buddy_id | mentor_id |
|------------|------|-------|--------------|-----------|-----------|----------| -------- | --------- |

`mentors`

| mentor_id | name | currentCompanyName |
|-----------|------|--------------------|

`mentor_sessions`

| mentor_session_id | time | duration | student_rating | mentor_rating | student_id | mentor_id |
|-------------------|------|----------|----------------|---------------| ---------- | --------- |

## Step 3: Identify Enums

Now, for the batch type, it can be DSML or Academy. Here, the batch type is enum (enum represents one of the given fixed set of values). 

Eg: 
```
enum Gender{
    male,
    female
};
```

So, we will have a `batch_types` table. 

`batch_types`

| id | value |
| -- | ----- |

Cardinality between `batches` and `batch_types` will be m:1. In `batches` table we will have `batch_type_id`.

`batches`

| batch_id | name | start_month | curr_inst_id | batch_type_id |
|----------|------|-------------|--------------| ------------- |

### How to represent enum

1. Using strings
    Cons: The problem in storing enums this way is that it will take a lot of space. It will have slow string comparison.

    Pros: Readability. No joins are required.

    `batches`

    | batch_id | name | type    |
    | -------- | ---- | ------- |
    | 1        | b1   | DSML    |
    | 2        | b2   | Academy |
    | 3        | b3   | Academy |
    | 4        | b4   | DSML    |

2. Using integers
    Here, 0 means DSML type batch and 1 means Academy type batch.
    
    Cons: No readability. We can not add or delete values (enums) in between as it will cause discrepencies. Also, what a particular value represents is not in the database.

    `batches`

    | batch_id | name | type_id |
    | -------- | ---- | ------- |
    | 1        | b1   | 0       |
    | 2        | b2   | 1       |
    | 3        | b3   | 1       |
    | 4        | b4   | 0       |

3. Lookup table
    It will have id and value columns where each type is stored as separate. The `type_id` of `batches` will refer to the `id` column of `batch_types`. All the above cons are solved with this method. 
    
    **batch_types**
    
    | id | value      |
    | -- | ---------- | 
    | 1  | Academy    |
    | 2  | DSML       | 
    | 3  | Neovarsity |
    | 4  | SST        |
    
So, the best way to represent enums is to use lookup table.


## Step 4: Deciding Primary Keys of a mapping table
    
### Example from previous discussion:

For `student_batches` the primary key will be (student_id, batch_id).

`student_batches`

| student_id | batch_id | move_date |
|------------|----------|-----------|

If in case we have our table like this, the primary key will be `id`. Size of index will be lesser here. 

`student_batches`

| id | student_id | batch_id | move_date |
| -- |------------|----------|-----------|

### Example 2
1. Scaler has exams.
2. For each batch a student joins, they will have to take exams of that batch.
3. Each exam is associated to a batch.

`exams`

| id | name | start_date | end_date   |
| -- | ---- | ---------- | ---------- |

Between batch and exam, each exam is associated to a batch, we will have to create a mapping table. One batch can have multiple exams, One exam can be present fo multiple batches. 

`exam_batches`

| exam_id | batch_id |
| ------- | -------- |

Similarly we also have a table called `student_batches`.

`student_batches`

| student_id | batch_id | date |
|------------|----------|------|

To figure out which student went through which exams, we will need to join `student_batches` with `exam_batches`. Basically, we are forming a relation between two mapping tables.

### Example 3

1. One student can belong to multiple batches.
2. Every batch has exams.
3. Same exam may happen on different batches on different dates.
4. If a students moves the batch, they may have to give some exams again.

`student_batches`

| student_id | batch_id | date |
|------------|----------|------|

Cardinality between batches ad exams is m:m. So, we will have a `batch_exams` table. Date is also an attribute of this relation.

`batch_exams`

| batch_id | exam_id | date |
| -------- | ------- | ---- |

Between students and exams also the cardinality is m:m. But if we have (student_id, exam_id) as primary key of the new `student_exams` table, it will not allow one student to take a particular exam twice. So, we will have to add `batch_id` also in PK. The below `student_batch_exams` will be our new table.

`student_batch_exams`

| student_id | batch_id | exam_id | marks |
| ---------- | -------- | ------- | ----- | 

Hence, we can see that sometimes a mapping may also have a relation with another entity. In these cases, not having a primary key can cause problems. 

**Advantages of a separate key:**
If a relation is being mapped to another entity or relation, it saves space.

**Advantages of NO separate key:**
Queries on first column will become faster because the table will be sorted by that column. A mapping table is often used for relationships and thus will require joins. Having no separate key makes things faster.

## Step 5: Representing Foreign keys and indexes

There are 2 steps here. First establishing foreign key relationships and then indexes for frequent use-cases. 

### Step 5.1: Foreign Keys

Typically, when you have a relationship table, which exists because there is a relationship between entity1 and entity2, then it's usually recommended to have a foreign key relationship between the relationship table and the entity it references. 

For example, consider the following table which exists due to m:m cardinality relationship between `batches` and `classes`. 

`batch_classes`

| batch_id | class_id |
| -------- | -------- | 

If we expect that whenever we query this table, we will need to get the associated batch and class details (which is almost always the case), then it makes sense to have a foreign key relationship with `batches` and `classes` table. 

```sql
ALTER TABLE batch_classes ADD CONSTRAINT fk_batch FOREIGN KEY (batch_id) REFERENCES batches(batch_id);
ALTER TABLE batch_classes ADD CONSTRAINT fk_class FOREIGN KEY (class_id) REFERENCES classes(class_id);
```

Very similarily, all other relationships we discussed in step 2, are eligible for a foreign key constraint. 

*Note that however, it is not mandatory to specify foreign key relationships. Foreign key relationships help with data consistency - so, incase you expect that the column I am referring to, must be unique - then specifying a foreign key constraint will enforce that (MySQL automatically then creates an index on that column - the foreign table column -  unless one already exists).*

### Step 5.2: Identify Indexes

The second step is to list down the frequent use-cases and explore if the queries I have to write for the frequent use-cases - are they fast or not? 
If they are not, then it warrants creating an index to avoid full table scans. 

Let's say that the learners often search mentor by a name. This is a use case. On which column of which table will you create an index for this? You have to create an index on `name` column of `mentors` table.

`mentors`
| mentor_id | name | company_name |
|-----------|------|--------------|

As a rule of thumb, given a query, look at the join conditions and where condition, followed by Order by with limit. If your query is slow, creating an index on those columns helps. **Please refer to the quizzes done during the class to understand this better.**

After drawing the complete Schema, mention the indexes.
This was all about Schema Design!

## Case Study - Netflix Schema Design

Let's go over the problem statement first. 

[Netflix Schema Design](https://docs.google.com/document/d/1xQbcv-smnV_JY6NUb4gz2owwPaQMWdoWty6PZyFEsq8/edit?usp=sharing)

**Problem Statement**
Design Database Schema for a system like Netflix with following Use Cases.
**Use Cases**
1. Netflix has users.
2. Every user has an email and a password.
3. Users can create profiles to have separate independent environments.
4. Each profile has a name and a type. Type can be KID or ADULT.
5. There are multiple videos on netflix.
6. For each video, there will be a title, description and a cast.
7. A cast is a list of actors who were a part of the video. For each actor we need to know their name and list of videos they were a part of.
8. For every video, for any profile who watched that video, we need to know the status (COMPLETED/ IN PROGRESS).
9. For every profile for whom a video is in progress, we want to know their last watch timestamp.

Let's approach this problem as one should in an interview.

1. Finding all the nouns to create tables.

- `users` 
- `profiles`
- `videos`
- `actors` (cast is nothing but a mapping between videos and actors)

1.2. Enums:

- `profile_type` (lookup table)
- `watch_status_type` (enum, it is an attribute of relation between profile and videos)

1.3. Finding attributes of particular entites.
    
    `users`
    | id | email | password |
    | -- | ----- | -------- |
    
    `profiles`
    | id | name | 
    | -- | ---- |
    
    `profile_type`
    | id | value | 
    | -- | ----- |
    
    `videos`
    | id | name | description |
    | -- | ---- | ----------- |
    
    `actors`
    | id | name | 
    | -- | ---- |
    
    `watch_status_type`
    | id | value | 
    | -- | ----- |
    
2. Representing relationships.

    Now, there are no relationships in the first and second use cases. Moving forward, what is the cardinality between `users` and `profiles`? One user can have multiple profiles but one profile is associated with one user. Therefore, it is 1:m, id of user will be in `profiles` table.
    
    `profiles`
    | id | name | user_id |
    | -- | ---- | ------- |
    
    What is the cardinality between `profiles` and `profile_type`? It is m:1, `profiles` will have another column `profile_type_id`.
    
    `profiles`
    | id | name | user_id | profile_type_id |
    | -- | ---- | ------- | --------------- |
    
    What is the cardinality between `videos` and `actors`? One video can have multiple actors and one actor could be in multiple videos. So, it is m:m. 
    
    `video_actors`
    | video_id | actor_id | 
    | -------- | -------- |
    
    Status is an information about relation between `videos` and `profiles`. Hence, a new table is created. Last watch timestamp is also an attribute on these two.
    
    `video_profiles`
    | video_id | profile_id | watch_status_type_id | last_watched_ts |
    | -------- | ---------- | -------------------- | --------------- |

3. Enum is already done in step 1.2.

4. Let's identify primary key for every table. 

 - users : new column `id`
 - profiles: new column `id`
 - profile_type: new column `id`
 - videos: new column `id`
 - actors: new column `id`
 - video_actors: (video_id, actor_id)
 - video_profiles: (profile_id, video_id)
 - watch_status: new column `id`

5. Indexes required. 

 - Use case 1: Log IN. 

```sql
SELECT password FROM users WHERE email = xyz
```

Index on `email` in `users`

 - Get all profiles for a given user

```sql
SELECT id, name, profile_type_id FROM profiles WHERE user_id = xyz 
```

Index on `user_id` in `profiles`

 - Recently played videos

```sql
SELECT video_id FROM video_profiles WHERE profile_id = xyz AND watch_status = 2 ORDER BY last_watched_ts DESC LIMIT 10
```

Index on (profile_id, watch_status)


And so on. 



