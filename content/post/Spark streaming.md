---
title: "Spark Streaming - Advantages and Limitations"
date: 2018-04-18
categories:
  - systems
tags:
  - Systems
  - Data processing
---

### References
- https://databricks.com/blog/2015/07/30/diving-into-apache-spark-streamings-execution-model.html
- http://cs.brown.edu/~ugur/8rulesSigRec.pdf


### Let's start
**The 8 requirements of real-time stream processing**. (1) Keep data moving - There should be no storage in the critical processing path. The processing should be active (event-based) rather than passive (polling-based). (2) Support for SQL on streams - Apart from general SQL queries, streams need special queries like 'Merge' (to merge two streams in ascending time order), that are only there for streams. Also, there needs to be a concept of window, over which aggregates can be computed. There needs to be a way to handle delayed, missing and out-of-order data when using windows and that is also the third requirement. (3) Handle stream imperfections (missing, delayed and out of order data) - Mostly time-outs are used to handle delayed data. Some systems like Apache Samza [re-emit the output of the window/s that are impacted by the late arrival](https://engineering.linkedin.com/blog/2016/06/stream-processing-hard-problems-part-1-killing-lambda). (4) Generate predictable outcomes - This means ensuring time ordered and deterministic processing throughout the pipeline. Generating predictable outcomes is also necessary for being able to replay and reprocess input stream for fault tolerance and recovery. (5) 

**Traditional Stream processing engines** use continuous operators that  process the streaming data one record at a time and forwards the records to other operators in the pipeline. They do this to minimize the latency (between input of a record and output of results). This causes a few issues as mentioned in [Models and Issues in Data Stream Systems](https://infolab.usc.edu/csci599/Fall2002/paper/DML2_streams-issues.pdf "Models and Issues in Data Stream Systems").

- (How does it not face the issue that we faces which forced us to do batching)

In the traditional continuous operator model, the continuous operator is statically allocated to a node. A continuous operator has a particular logic and operates on entire or a subset (based on some key) of the output of the previous operator. In contrast, Spark tasks are assigned dynamically to the workers based on the locality of the data and available resources. Spark streaming batches the incoming records into tiny sub-second micro batches. Then, the latency optimized spark engine launches tasks on appropriate computing nodes to process the batches. (`TODO:` Doesn't this make the Spark engine or driver a bottleneck because it has to decide where to allocate every batch).  This leads to two advantages:
1. Fast failure and straggler recovery - The static allocation of continuous operators to worker nodes in traditional model makes it challenging for traditional systems to recover quickly from faults and stragglers (computing nodes that have slowed down). [Picture from Databricks site](https://databricks.com/wp-content/uploads/2015/07/image41-1024x602.png)
> In case of node failures, traditional systems have to restart the failed continuous operator on another node and replay some part of the data stream to recompute the lost information. Note that only one node is handling the recomputation, and the pipeline cannot proceed until the new node has caught up after the replay. In Spark, the computation is already discretized into small, deterministic tasks that can run anywhere without affecting correctness. So failed tasks can be relaunched in parallel on all the other nodes in the cluster, thus evenly distributing all the recomputations across many nodes, and recovering from the failure faster than the traditional approach.


2. Dynamic load balancing - [Picture from Databricks site](https://databricks.com/wp-content/uploads/2015/07/image31-1024x581.png)

> In the traditional record-at-a-time approach taken by most other systems, if one of the partitions is more computationally intensive than the others, the node statically assigned to process that partition will become a bottleneck and slow down the pipeline. In Spark Streaming, the job’s tasks will be naturally load balanced across the workers — some workers will process a few longer tasks, others will process more of the shorter tasks.

3. Since every batch is an RDD (distributed dataset), the Spark streaming batch is interoperable with pre-computed static dataset. It can be joined and all spark functions that are possible on a static RDD can be done on it.

**Limitation:** Spark streaming admits to not handling out-of-order data.  They promise to add support for this later.
> In practice, users sometimes have records that are delivered out of order, or with a timestamp that differs from the time of ingestion. Spark streaming will support "event time" by allowing user-defined time extraction function. This will include a slack duration for late or out-of-order data.




