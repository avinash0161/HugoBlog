---
title: "Xac 1 - ACIDity of transactions"
date: 2018-04-12
categories:
  - transactions
tags:
  - DBMS
  - Transactions
#thumbnailImagePosition: left
#thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/highlighted-code-showcase/peak-140.jpg
---

### Why do we need transactions in databases? 
Transactions are needed when a group of DBMS statements together need to do something and either all of them should execute or none should execute. Ex: Consider transfer of funds from one bank account to other. It should never happen that one account was debited but the other one was never credited. So, either the SQL statements of both credit and debit should pass or both should fail. This is done by making them a part of one transaction. Why? because transactions guarantee **atomicity** i.e. either entire transaction commits or aborts.

### So what's the problem

However, we can't have one transaction executing serially after another because one of them may be doing a disk IO while the CPU is free and can be used by other transaction. So, we allow the transactions to be concurrent. However, this causes a problem. If two transactions, which are concurrent, touch the same data, inconsistency might be exposed to the outer world. For ex, say there are two bank accounts with ₹2000 each and two transactions: 1. transfers ₹100 from one account to another, 2. Shows the total amount in the two accounts. Now, say the transactions are running concurrently. The first transaction debits the amount. Before, it can credit, the second transaction runs and shows the total amount as ₹3900. This is inconsistent. 

### Solution to inconsistency
This is taken care of by the next two properties of transactions - **consistency and isolation**.

**Consistency** property mandates that a transaction should be such that if the database was consistent at the beginning of the transaction, it will be consistent after the transaction. The definition of consistency varies within DBMS systems and is often user defined. However, right now, you can think of consistency as not violating any logical constraints. However, transactions overlap and are concurrent. So, a transaction which would have left the database in a consistent state after it was over may not do so  because the data it was accessing was meddled with, by some other transaction executing concurrently. Here, comes the importance of **isolation** (also known as serializability) which says that concurrent execution of transactions should leave the system in a state which is equivalent to some serial execution order of the transactions. If isolation is guaranteed, we know that there exists a serial order of execution of transactions which results in the current database state. This coupled with the consistency property assures that a database which starts in a consistent state will end in a consistent state, no matter how many transactions take place. In other words, isolation can also mean that intermediate state of database during a transaction isn't visible to other transactions (This visibility property doesn't hold for multi-level transactions, but is taken care of by the concurrency control mechanisms). Providing isolation is one of the goals of concurrency control in the databases.

Lastly, the property of **Durability** says that if a transaction commits, its changes become permanent and will be so even in the event of power loss, crashes etc. Together these are known as the ACID properties of the transactions.