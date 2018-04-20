---
title: "Xac 2 - Serializability of Transactions"
date: 2018-04-18
categories:
  - transactions
tags:
  - DBMS
  - Transactions
#thumbnailImagePosition: left
#thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/highlighted-code-showcase/peak-140.jpg
---

In the [previous blog](https://avinash0161.github.io/2018/04/xac-1---acidity-of-transactions/ "previous blog"), we talked about ACID properties of transactions and how it allows us to maintain a consistent state even though transactions execute concurrently. It was said that if the concurrent execution of transactions are equivalent to some serial execution, then the isolation property of transactions is satisfied and consistency is ensured. HOWEVER, HOW DOES A DBMS ENSURE THAT A CONCURRENT EXECUTION OF TRANSACTIONS IS EQUIVALENT TO SOME SERIAL SCHEDULE OF TRANSACTIONS?

So, there are some theoretical concepts regarding this. Before that, we define the term schedule as some order (may be interleaved) of execution of transactions. Transactions which actually are DBMS statements can be viewed at micro-level as just reads and writes. So, if `T1 ( r1(x), w1(y))` and `T2 (r2(z), w2(y))` are two transactions, then `r1(x), r2(z), w2(y), w1(y)` is one schedule and `r1(x), w1(y), r2(z), w2(y)` is another possible schedule (the latter schedule is also serial). There are many more schedules possible.

### Equivalence detection
There exists different types of equivalence definitions which lead to different types of serializability. Some equivalence definitions are flawed and thus even though according to the definition, the schedule is equivalent to a serial schedule, in reality it is not, and following schedules allowed by such equivalence definitions may lead to inconsistent database. We discuss three types of equivalences: -
- **Final State Equivalence (FSE) or result equivalence**: Two schedules are equivalent if at the end of both the state of database is same and the path to arrive at that state is also the same. However, this only means that the final write and the way to get there is checked. If the intermediate reads who have no role in final writes return inconsistent results, FSE will still approve the schedule as final state serializable. For eg: consider the schedule `r2(x) w2(x) r1(x) r1(y) r2(y) w2(y) c1 c2`. This is said to be FSE to both T1 T2 and T2 T1. However, the Select statements (i.e. r1(x) r1(y)) return different results in all the three cases (two serialized and the original schedule). The graphical method to find FSE is descibed in [this lecture slide](http://www.inf.uni-konstanz.de/dbis/teaching/ss05/txs/ch3.pdf "this lecture slide").
- **View Equivalence (VSE)**: Two schedules are view equivalent if 
	1. The first reads of data items in both schedules are done by same transactions. i.e. if Ti reads data item x in schedule 1, the same should happen in the second schedule.
	2. The final writes of all data items should be done by the same transactions.
	3. The producer-consumer pattern should be followed i.e. if a Ti reads value written by Tj in first schedule, the same should happen in the second schedule too.

	So, at every step, view equivalence ensures that the view to all transactions are same in both the schedules. More details are given in [this youtube video](https://www.youtube.com/watch?v=NWmhKcjAoNI "this youtube video") by Techtud. View equivalence guarantees consistency. However, finding all the serial order of executions and checking them for serializability, is time taking. Therefore, we have a stricter version of equivalence called conflict equivalence.
	
- **Conflict Equivalence**: Two operations are conflicting if they are on the same data item and at leat one of them is a write. We lay out all the transactions in the given schedules and form a dependency graph for both. If the graphs are the same, then they are conflict equivalent. For a schedule to be conflict serializable, its dependency graph shouldn't have any cycles. This is because a serial schedule can't have cycles in its dependency graph. More details are given in [this youtube video](https://www.youtube.com/watch?v=UJw0ElBBNiU "this youtube video") by Techtud.
So, in order to check if a schedule is conflict serializable, the DBMS has to just make dependence graph and check that there are no cycles.

An important thing to note is that if two schedules are view equivalent but not conflict equivalent then it must be due to the presence of blind writes. An example of blind write is given in this [Stack Exchange post](https://dba.stackexchange.com/questions/115435/what-is-meant-by-a-blind-write-in-a-schedule "Stack Exchange post"). 

### Similarity to Parallel Computing
If one notes, the attempt to use only serializable schedules to maintain consistency is akin to usage of locks in parallel computing. In parallel computing, the low level details are exposed to the developers, and they themselves have to use locks to maintain consistency and avoid race conditions. A benefit there is that given that kind of low level access, developers can obtain high concurrency. In contrast, the concurrency control manager of DBMS abstracts the low level lock usage (which can get murky in complex cases) and attempts to implement locking mechanism by itself which fits all purposes and doesn't challenge consistency. Mechanisms such as 2 phase locking schemes are methods to do just that. 2 phase locking and other concurrency control schemes are formulated in a way that they are stricter than even conflict serializability i.e. even a conflict serializable schedule wouldn't be approved by 2-phase locking schemes. Obviously, this attempt to build a  general solution which fits all cases, has its toll on the concurrency that the DBMS can provide.

### Final Thoughts
What we have seen in this blog is how to check for serializability. However, the DBMS system just gets a stream of reads and writes from different transactions. How does it implement serializability is discussed in [another post](https://avinash0161.github.io/2018/04/xac-4---ensuring-only-serializable-schedules-pass-through/ "another post").