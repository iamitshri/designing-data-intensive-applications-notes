# Chapter 6: Partitioning

- When data can not fit in single disk and/or query throughput grows, we need to break the data up into partitions, also knows as sharding
- Main reason for partition is scalability, large dataset can be distributed across many disks and the query load can be distributed across many processors

### Partition and Replication
- Partition is usually combined with replication. 
- Node stores many partitions. A partition belongs one to node but its copies are replicated to other nodes for fault tolerance

### Partition of key-value data

- Ideally keys & queries are evenly distributed, however partitioning can be unfair, data and queries can be disproportionately spread out, meaning skewed
- A partition with disproportionately high load is called hot-spot.
- Simplest approach to avoid hot spots is divide data randomly among nodes 
#### Partitioning by Key-Range

- Range boundaries needs to be set either manually or dynamically 
- Pros: 
  - Range scan queries are easier
  - within partition keys can be kept in sorted order

- Cons:
  - Hot spots are possible

#### Partitioning by Hash & key

- To avoid the risk of skew and hotspot many distributed datastores use hash function to determine the partition for a given key.
- A good function takes a skewed data and makes it uniformly distributed
- Partition are assigned range of hashes
- Hot spots and skew is still possible for example when a celebrity posts an update causing millions of follower take action on that single user id
  - Today most data systems can not automatically compensate for such highly skewed workload, so its the responsibility of application to reduce the skew
  - For example, if one key is known to be very hot, a simple technique is to add a random number to the beginning or end of the key
    - This technique also requires additional bookkeeping, so apply this technique to small number of hot keys

- Pros: 
- data is evenly distributed

Cons:
- Range queries are not efficient

#### Partitioning by approach 
- Hybrid approaches are also possible, for example with a compound key: using one part of the key to identify the partition and another part for the sort order.


#### Partition and Secondary indexes
We also discussed the interaction between partitioning and secondary indexes. A secondary index also needs to be partitioned, and there are two methods:

Document-partitioned indexes (local indexes), where the secondary indexes are stored in the same partition as the primary key and value. This means that only a single partition needs to be updated on write, but a read of the secondary index requires a scatter/gather across all partitions.

Term-partitioned indexes (global indexes), where the secondary indexes are partitioned separately, using the indexed values. An entry in the secondary index may include records from all partitions of the primary key. When a document is written, several partitions of the secondary index need to be updated; however, a read can be served from a single partition.

Finally, we discussed techniques for routing queries to the appropriate partition, which range from simple partition-aware load balancing to sophisticated parallel query execution engines.

By design, every partition operates mostly independently—that’s what allows a partitioned database to scale to multiple machines. However, operations that need to write to several partitions can be difficult to reason about: for example, what happens if the write to one partition succeeds, but another fails? We will address that question in the following chapters.

