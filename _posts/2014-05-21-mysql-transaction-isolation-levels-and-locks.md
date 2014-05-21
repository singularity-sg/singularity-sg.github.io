---
layout: post
title: "MySQL Transaction Isolation Levels and Locks"
description: ""
category: "database" 
tags: ["mysql"]
---
{% include JB/setup %}

Recently, an application that my team was working on encountered problems with a MySQL deadlock situation and it took us some time to figure out the reasons behind it. This application that we deployed was running on a 2-node cluster and they both are connected to an AWS MySQL database. The MySQL db tables are mostly based on InnoDB which supports transaction (meaning all the usual commit and rollback semantics) as well as row-level locking that MyISAM engine does not provide. So the problem arose when our users, due to some poorly designed user interface, was able to execute the same long running operation twice on the database.

As it turned out, due to the fact that we have a dual node cluster, each of the user operation originated from a different web application (which in turn meant 2 different transaction running the same queries). The deadlock query happened to be a "INSERT INTO T... SELECT FROM S WHERE" query that introduced shared locks on the records that were used in the SELECT query. It didn't help that both T and S in this case happened to be the same table. In effect, both the shared locks and exclusive locks were applied on the same table. An attempt to explain the possible cause of the deadlock on the queries could be explained by the following table. This is based on the assumption that we are using a default REPEATABLE_READ transaction isolation level (I will explain the concept of transaction isolation later)

<!-- more -->

Assuming that we have a table as such

| RowId              | Value                   | 
|:-----              |:----------              |
| 1                  | Collection 1            |
| 2                  | Collection 2            |
| ...                | Collection N            |
| 450000             | Collection 450000       |
{: #example}

The following is a sample sequence that could possibly cause a deadlock based on the 2 transactions running an SQL query like "INSERT INTO T SELECT FROM T WHERE ... " : 

|Time                | Transaction 1                                 | Transaction 2                              | Comment                                                                            | 
|:-------------------|:--------------                                |:--------------                             |:---------                                                                           |     
|T1                  | Statement executed                            |                                            | Statement executed. A shared lock is applied to records that are read by selection |
|T2                  | Read lock s1 on Row 10-20                     |                                            | The lock on the index across a range. InnoDB has a concept of gap locks.           |
|T3                  |                                               | Statement executed                         | Transaction 2 statement executed. Similar shared lock to s1 applied by selection   |
|T4                  |                                               | Read lock s2 on Row 10-20                  | Shared read locks allow both transaction to read the records only                  |
|T5                  | Insert lock x1 into Row 13 in index wanted    |                                            | Transaction 1 attempts to get exclusive lock on Row 13 for insertion but Transaction 2 is holding a shared lock |
|T6                  |                                               | Insert lock x2 into Row 13 in index wanted | Transaction 2 attempts to get exclusive lock on Row 13 for insertion but Transaction 1 is holding a shared lock |
|T7                  |                                               |                                            | Deadlock!                                                                          |
{: #scenario}

The above scenario occurs only when we use REPEATABLE_READ (which introduces shared read locks). If we were to lower the transation isolation level to READ_COMMITTED, we would reduce the chances of a deadlock happening. Of course, this would mean relaxing the consistency of the database records. In the case of our data requirements, we do not have such strict requirements for strong consistency. Thus, it is acceptable for one transaction to read records that are committed by other transactions. 

So, to delve deeper into the idea of Transaction Isolation, this concept has been defined by ANSI/ISO SQL as the following from highest isolation levels to lowest 

1. Serializable
: This is the highest isolation level and usually requires the use of shared read locks and exclusive write locks (as in the case of MySQL). 
: What this means in essence that any query made will require access to a shared read lock on the records which prevents another transaction's query to modify these records. Every update statement will require access to an exclusive write lock
: Also, range-locks must be acquired when a select statement with a WHERE condition is used. This is implemented as a gap lock in MySQL.

2. Repeatable Reads
: This is the default level used in MySQL. This is mainly similar to Serializable beside the fact that a range lock is not used. However, the way that MySQL implements this level seemed to me a little different. Based on Wikipedia's [article] (http://en.wikipedia.org/wiki/Isolation_(database_systems)#Isolation_levels) on Transaction Isolation, a range lock is not implemented and so phantom reads can still occur. Phantom reads refer to a possibility that select queries will have additional records when the same query is made within a transaction. However, what I understand from MySQL's [document] (http://dev.mysql.com/doc/refman/5.6/en/set-transaction.html#isolevel_serializable) is that range locks are still used and the same select queries made in the same transaction will always return the same records. Maybe I'm mistaken in my understanding and if there's any mistakes in my intepretations, I stand ready to be corrected.

3. Read Committed
: This is an isolation level that will maintain a write lock until the end of the transaction but read locks will be released at the end of the SELECT statement. It does not promise that a SELECT statement will find the same data if it is re-run again in the same transaction. It will, however, guarantee that the data that is read are not "dirty" and has been committed.

4. Read Uncommitted
: This is an isolation level that I doubt would be useful for most use cases. Basically, it allows a transaction to see all data that has been modified, including "dirty" or uncommitted data. This is the lowest isolation level  
 
Having gone through the different transaction isolation levels, we could see how the selection of the Transaction Isolation level determines the kind of database locking mechanism. From a practical standpoint, the default MySQL isolation level (REPEATABLE_READ) might not always be a good choice when you are dealing with a scenario like ours where there is really no need for such strong consistency in the data reads. I believe that by lowering the isolation level, it is likely to reduce chances that your database queries meet with a deadlock. Also, it might even allow a higher concurrent access to your database which improve the performance level of your queries. Of course, this comes with the caveat that you need to understand how important consistent reads are for your application. If you are dealing with data where precision is paramount (e.g. your bank accounts), then it is definitely necessary to impose as much isolation as possible so that you would not read inconsistent information within your transaction.
