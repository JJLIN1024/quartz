---
title: Chapter 7 Transactions
date: 2024-12-08
lastmod: 2024-12-08
author:
  - Jimmy Lin
tags: 
draft: false
---

## What is a Transaction

A transaction is a way for an application to group several reads and writes together into a logical unit.

## ACID

Atomicity, Consistency, Isolation, Durability
>  The high-level idea is sound, but the devil is in the details.

Single-Object Transations OS 的同步章節裡面都有教
## Multi-Object Transactions

why do we need multi-object transactions?
1.  In a relational data model, a row in one table often has a foreign key reference to a row in another table. (Similarly, in a graph-like data model, a vertex has edges to other vertices.) 
2. In databases with secondary indexes (almost everything except pure key-value stores), the indexes also need to be updated every time you change a value

## Weak (Non-Serializable) Isolation

A popular comment on revelations of such problems is “Use an ACID database if you’re handling financial data!”—but that misses the point. Even many popular relational data‐ base systems (which are usually considered “ACID”) use weak isolation, so they wouldn’t necessarily have prevented these bugs from occurring.
## Read Committed

1. When reading from the database, you will only see data that has been committed (no dirty reads).
2. When writing to the database, you will only overwrite data that has been com‐ mitted (no dirty writes).

Imagine a transaction has written some data to the database, but the transaction has not yet committed or aborted. Can another transaction see that uncommitted data? If yes, that is called a dirty read 

![[截圖 2024-12-08 下午2.51.39.png]]

If a transaction needs to update several objects, a dirty read means that another transaction may see some of the updates but not others.

what happens if the earlier write is part of a transaction that has not yet committed, so the later write overwrites an uncommitted value? This is called a dirty write 

![[截圖 2024-12-08 下午2.54.40.png]]
> buyer is Bob but recipient is sent to the wrong person - Alice

read committed does not prevent the race condition between two counter increments

![[截圖 2024-12-08 下午2.57.28.png]]

databases prevent dirty writes by using row-level locks:

How do we prevent dirty reads? One option would be to use the same lock,
requiring read locks does not work well in practice, because one long-running write transaction can force many read-only transactions to wait until the long-running transaction has completed

most databases prevent dirty reads using the approach illustrated in
Figure 7-4: for every object that is written, the database remembers both the old committed value and the new value set by the transaction that currently holds the write lock. While the transaction is ongoing, any other transactions that read the object are simply given the old value. Only when the new value is committed do transactions switch over to reading the new value.

![[截圖 2024-12-08 下午3.10.06.png]]
> nonrepeatable read or read skew: 100$ vanished in the air

read skew is not a lasting problem, but some situations cannot tolerate such temporary inconsistency, for example: Backups, Analytic queries and integrity checks

## Snapshot isolation

each transaction reads from a consistent snapshot of the database—that is, the trans‐ action sees all the data that was committed in the database at the start of the transac‐ tion.

implementations of snapshot isolation typically use write locks to prevent dirty writes 
readers never block writers, and writers never block readers.

 The database must potentially keep several different committed versions of an object, because various in-progress trans‐ actions may need to see the state of the database at different points in time
 
multi- version concurrency control (MVCC).

A typical approach is that read committed uses a separate snapshot for each query, while snapshot isolation uses the same snapshot for an entire transaction

![[截圖 2024-12-08 下午3.18.59.png]]

If a transaction deletes a row, the row isn’t actually deleted from the database, but it is marked for deletion by setting the deleted_by field to the ID of the transaction that requested the deletion. At some later time, when it is certain that no transaction can any longer access the deleted data, a garbage collection process in the database removes any rows marked for deletion and frees their space.

### Visibility rules for observing a consistent snapshot

1. At the start of each transaction, the database makes a list of all the other transac‐ tions that are in progress (not yet committed or aborted) at that time. Any writes that those transactions have made are ignored, even if the transactions subse‐ quently commit.
2. Any writes made by aborted transactions are ignored.
3. Any writes made by transactions with a later transaction ID (i.e., which started after the current transaction started) are ignored, regardless of whether those transactions have committed.
4. All other writes are visible to the application’s queries.

Put another way, an object is visible if both of the following conditions are true:
• At the time when the reader’s transaction started, the transaction that created the object had already committed.
• The object is not marked for deletion, or if it is, the transaction that requested deletion had not yet committed at the time when the reader’s transaction started.

Indexes and snapshot isolation
How do indexes work in a multi-version database? One option is to have the index simply point to all versions of an object and require an index query to filter out any object versions that are not visible to the current transaction. When garbage collec‐ tion removes old object versions that are no longer visible to any transaction, the cor‐ responding index entries can also be removed.

Or, 提供 snapshot 的方式還有：append-only B-trees, 這我們都很熟了

## Preventing Lost Updates

There are several other interesting kinds of conflicts that can occur between concur‐ rently writing transactions. The best known of these is the lost update problem, illus‐ trated in Figure 7-1 with the example of two concurrent counter increments.

Solution: Atomic write operations(provided by DB), or application to explicitly lock objects that are going to be updated.

explicitly lock objects that are going to be updated:
```sql
-- Start a new transaction
BEGIN TRANSACTION;

-- Lock the row(s) for the specified figure to ensure no conflicting changes
SELECT * 
FROM figures 
WHERE name = 'robot' AND game_id = 222 
FOR UPDATE;

-- Business logic to check if the move is valid
-- This check happens outside SQL, in the application logic.

-- If the move is valid, update the position of the selected figure
UPDATE figures 
SET position = 'c4' 
WHERE id = 1234;

-- Commit the transaction to make the changes permanent
COMMIT;

```
> The FOR UPDATE clause places a row-level lock on all rows matching the condition (name = 'robot' AND game_id = 222).

Atomic operations and locks are ways of preventing lost updates by forcing the read- modify-write cycles to happen sequentially. An alternative is to allow them to execute in parallel and, if the transaction manager detects a lost update, abort the transaction and force it to retry its read-modify-write cycle.
An advantage of this approach is that databases can perform this check efficiently in conjunction with snapshot isolation





