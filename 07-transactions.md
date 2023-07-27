## Agenda

 - Need for concurrency
 - Problems that arise with concurrency
 - Introduction to transactions
 - Commit / Rollback
 - ACID
 - Is ACID boolean
 - Durability levels
 - Isolation levels


## Need for concurrency

Till now, all our SQL queries were written with the assumption that there is no interference from any other operation on the machine. 
Basically, all operations are being run sequentially. 

That's however like there being only one queue in immigration. Leads to things being slow and large wait time for someone in the queue.
In such a case, what do you do? 
Correct. Open multiple counters so that there are multiple *parallel* lines. 

Very similarily, in a database, there could be multiple people trying to run their queries at the same time. If DB chooses to run them sequentially, it will become really slow. Machines have multi-core CPUs. So, Database can think of running multiple queries concurrently which will increase it's throughput and reduce the wait time for people trying to run their queries. 

## Problems that arise with concurrency 

However, concurrency is not all good. It can lead to some issues around data integrity and unexpected behavior. Let's explore one such case.

Imagine we have a database for a bank. 
What's one common transaction in a bank? Transfer money from person X to person Y. 

What would be steps of doing that money transaction (let's say transfer 500 INR from X to Y):

```
  1. Read balA = current balance of user X. 
  2. If balA >= 500:
  3.   update current balane of user X to be (balA - 500)
  4.   Read balB = current balance of user Y
  5.   update current balance of user Y to be (balB + 500)

```


Let's imagine there are 2 money transactions happening at the same time - Person A transferring 700 INR to Person B, and Person A transferring 800 INR to Person C. 
Assume current balance of A is 1000, B is 5000, C is 2000. 

It is possible that Step 1 (`Read balA = current balance of A`) gets exectuted for both money transaction at the same time. So, both money transaction find balA = 1000. And hence step 2 (balance being larger than the money being transferred) passes for both. Then step 3 gets executed for both money transactions (let's say). A's balance will be updated to 300 by money transaction 1 and then to 200 by money transaction 2. 
A's balance at the end will be 200, with both B and C getting the money and ending at 5700 and 2800 respectively. 

Total sum of money does not add up. Seems like the bank owes more money now. 
If you see, if both money transactions had happened one after another (after first had completely finished), then this issue would not have occurred. 
How do we avoid this? 

## Introduction to transactions. 

Let's understand the solution to the above in 2 parts. 

 1. What guidelines does the database management system give to the end user to be able to solve the above. Basically, what should I do as the end user, and the guarantees provided by my database. 
 2. How does the DB really solve it internally? What's happening behind the scene. 

Let's first look at 1. What tools does the DBMS give to me to be able to avoid situations like the above. 

In the above case, doing a money transaction involved 5 steps (multiple SQL queries). 
Database says why don't you group these tasks into a single unit called **Transactions**. 

Formally, Transactions group a set of tasks into a single execution unit. Each transaction begins with a specific task and ends when all the tasks in the group successfully complete. If any of the tasks fail, the transaction fails. Therefore, a transaction has only two results: success or failure.

For example, a transaction example could be:

```sql
BEGIN TRANSACTION

UPDATE film SET title = "TempTitle1" WHERE id = 10;
UPDATE film SET title = "TempTitle2" WHERE id = 100;

COMMIT
```

The above transaction has 2 SQL statements. These statements do not get finalized on the disk until commit command is executed. If any statement fails, all updates are rolled back (like undo operation). 
You can also explicitly write `ROLLBACK` which undoes all operations till the last commit. 

Database basically indicates that if you want a group of operations to happen together and you want database to handle all concurrency, put them in a single transaction and then send it to the database. 
If you do so, database promises the following. 

## ACID

If you send a transaction to a DBMS, it sets the following expectations to you:

 - **ATOMICITY:** All or none. Either all operations in the transactions will succeed or none will. 
 - **CONSISTENCY:** Correctness / Validity. All operations will be executed correctly and will leave the database in a consistent state before and after the transaction. For example, in the money transfer example, consistency was violated because the money amount sum should have stayed the same, but it was not. 
 - **ISOLATION:** Multiple transactions can be executed concurrently without interfering with each other. Isolation is one of the factors helping with consistency. 
 - **DURABILITY:** Updates/Writes to the database are permanent and will not be lost. They are persisted. 


However, there is one caveat to the above. Atomicity is boolean - all transactions are definitely atomic. 
But all the remaining parameters are not really boolean but have levels. That is so, because there is a tradeoff. To make my database support highest amount of isolation and consistency, I will have to compromise performance which we will see later. So, based on your application requirement, your DBMS lets you configure the level of isolation or durability. Let's discuss them one by one. 

### Durability levels

The most basic form of durability is writing updates to disk. However, someone can argue that what if my hard disk has a failure which causes me to loose information stored on the hard disk. 
I can choose then to have replica and all commits are forwarded to replicas as well. That has cost of latency and cost of additional machines. 
We will discuss more about master slave and replication during system design classes. 

## Isolation Levels

Before we explore isolation levels, let's understand how would database handle concurrency between multiple transactions happening at the same time. 
Typically, you would take locks to block other operations from interfering with your current transactions. [You'll study more about locks during concurrency class]. 
When you take a lock on a table row for example, any other transaction which also might be trying to access the same row would wait for you to complete. Which means with a lot of locks, overall transactions become slower, as they may be spending a lot of time waiting for locks to be released. 

Locks are of 2 kinds:
 - Shared Locks: Which means multiple transactions could be reading the same entity at the same time. However, a transaction that intends to write, will have to wait for ongoing reads to finish, and then would have to block all other reads and writes when it is writing/updating the entity. 
 - Exclusive Locks: Exclusive lock when taken blocks all reads and writes from other transaction. They have to wait till this transaction is complete. 
There are other kind of locks as well, but not relevant to this discussion for now.

A database can use a combination of the above to achieve isolation during multiple transactions happening at the same time. 
Note that the locks are acquired on rows of the table instead of the entire table. More granular the lock, better it is for performance. 

As we can see that locks interfere with performance, database lets you choose isolation levels. Lower the level, more the performance, but lower the isolation/consistency levels. 

The isolation levels are the following:

### Read uncommitted.

This is most relaxed isolation level. In READ UNCOMMITTED isolation level, there isn’t much isolation present between the transactions at all, ie ., No locks.
This is the lowest level of isolation, and does almost nothing. It means that transactions can read data being worked with by other transactions, even if the changes aren’t committed yet. This means the performance is going to be really fast. 

However, there are major challenges to consistency. 
Let's consider a case. 

| Time | Transaction 1 | Transaction 2 |
| ---  | ------------- | ------------- |
| 1 | Update row #1, balance updated from 500 to 1000 | | 
| 2 | | Select row #1, gets value as 1000 |
| 3 | Rollback (balance reverted to 500) | | 

T1 reverts the balance to 500. However, T2 is still using the balance as 1000 because it read a value which was not committed yet. This is also called as `dirty read`. 
`Read uncommitted` has the problem of dirty reads when concurrent transactions are happening. 

**Example usecase:** Imagine you wanted to maintain count of live users on hotstar live match. You want very high performance, and you don't really care about the exact count. If count is off by a little, you don't mind. So, you won't mind compromising consistency for performance. Hence, read uncommitted is the right isolation level for such a use case. 

### Read committed

The next level of isolation is READ_COMMITTED, which adds a little locking into the equation to avoid dirty reads. In READ_COMMITTED, transactions can only read data once writes have been committed. Let’s use our two transactions, but change up the order a bit: T2 is going to read data after T1 has written to it, but then T1 gets rolled back (for some reason).

| Time | Transaction 1 | Transaction 2 |
| ---  | ------------- | ------------- |
| 1 | Selects row #1 | |
| 2 | Updates row #1, acquires lock | |
| 3	| | Tries to select row #1, but blocked by T1’s lock |
| 4 | Rolls back transaction | |
| 5 | | Selects row #1 |

READ_COMMITTED helps avoid a dirty read here: if T2 was allowed to read row #1 at Time 3, that read would be invalid; T1 ended up getting rolled back, so the data that T2 read was actually wrong. Because of the lock acquired at Time 2 (thanks READ_COMMITTED!), everything works smoothly and T2 waits to execute its SELECT query.

**This is the default isolation levels in some DBMS like Postgres.**

However, this isolation level has a problem of **Non-repeatable reads.** Let's understand what that is.

Consider the following. 

| Time | Transaction 1 | Transaction 2 |
| ---  | ------------- | ------------- |
| 1 | Selects emails with low psp | |
| 2 | | update some low psp users with a better psp, acquires lock |
| 3 | | commit, lock released |
| 4	| Select emails with low psp again | |

At timestamp 4, I might want to read the emails again because I might want to update the status of having scheduled reminder emails to them. However, I will get a different set of emails in the same transaction (timestamp 1 vs timestamp 4). This issues is called non-repeatable reads and can happen in the current isolation level. 

### Repeatable reads

The third isolation level is repeatable reads. This is the default isolation levels in many DBMS including MySQL. 

The primary difference in repeable reads is the following:
 - Every transaction reads all the committed rows required for executing reads and writes before the start of the transaction and stores it locally in memory as a snapshot. That way, if you read the same information multiple times in the same transaction, you will get the same entries. 
 - Locking mechanism:
   - Writes acquire exclusive locks (same as read committed)
   - Reads with write intent (SELECT FOR UPDATE) acquire exclusive locks. 

Further reading: https://ssudan16.medium.com/database-isolation-levels-explained-61429c4b1e31 

| Time | Transaction 1 | Transaction 2 |
| ---- | ------------- | ------------- |
| 1 | Selects row #1 **for update**, acquires lock on row #1 | |
| 2	|  | Tries to update row #1, but is blocked by T1’s lock |
| 3 | updates row #1, commits transaction | |
| 4 | | Updates row #1 |

The first example we took of money transfer could work if select for update is properly used in transactions with this isolation level. 
However, this still has the following issue:
 - Normal reads do not take any lock. So, it is possible while I have a local copy in my snapshot, the real values in DB have changed. Reads are not strongly consistent. 
 - Phantom reads: A very corner case, but the table might change if there are new rows inserted while the transaction is ongoing. Since new rows were not part of the snapshot, it might cause inconsistency in new writes, reads or updates. 

### Serializable Isolation level

This is the strictest isolation level in a DB. It's the same as repeatable reads with the following differences:
 - All reads acquire a shared lock. So, they don't let updates happen till they are completed. 
 - No snapshots required anymore.
 - Range locking - To avoid phantom reads, this isolation level also locks a few entries in the range close to what is being read. 

**Example usecase:** This is the isolation levels that banks use because they want strong consistency with every single piece of their information. 
However, systems that do not require as strict isolation levels (like Scaler or Facebook) would then use Read Committed OR Repeatable Reads. 


