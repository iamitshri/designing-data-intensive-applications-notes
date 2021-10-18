# Chapter 3. Storage and Retrieval

- This chapter is about understanding how we can store data and how we can find it again
- App developers do need to select appropriate storage engine from the many that are available 
- We will examine two families of storage engines: log-structured storage engines and page-oriented storage engines such as B-trees
- Word Log is used in a general sense: an append only sequence of records. It's intended for machines to read
- index data structure typically used to increase the lookup speed. However it maintains some metadata which incurs overhead in maintaining it.
  - Index slows the write operation, because index needs to be udpated on each write operation
  - It is an important  trade-off in storage systems:
    - A well-chosen indexes speed up read queries but every index slows down writes
    - Use application knowledge or typical query pattern to add indexes

- Hash Indexes
  - Bitcask storage strategy
  - Has a good performance for read and writes provided all keys fit in memory
    - In-memory map where key is mapped to a byte offset in the data file
      - The location at which the value can be found 
  - Append only log 
    - how to avoid running out of space?
      - Break the log into segments of a certain size by closing a segment file when it reaches certain size
      - We can then do compaction on these segments
        - Compaction means throwing away duplicate keys in the log and keeping only the most recent update for each key
      - Segments can also be merged together at the same time 
      - Segments are immutable so the merged segments are written to a new file
      - Merging and compaction is done in the background thread
      - We can still continue to serve read requests using the old segment files and write requests to the latest segment file
      - After merging process is completed, we switch read requests to using the new merged segment instead of old segments
        - Old segments files can simply be deleted
  
### SSTables and LSM-Trees
- Sorted String Table: sequence of key value pair sorted by key
- During merging SSTable segments, when same key is present in different segments, we can keep the most recent segment value
- For finding a key we don't need a full hash index in memory, due to sorted nature we can use range to decide potential. Only few keys are needed in memory to maintain the offsets for some of the keys, but it can be sparse
- ![img.png](images/SSTable-with-in-memory-index.png)
#### Constructing and maintaining SSTables
- Maintaining sorted structure in memory: RedBlackTree, AVL Tree
- Write out the sorted structure (Trees mentioned above) called memtable to the disk when it reaches certain size
- Read requests can be served by looking up in the memtable first, then from the most recent segment of db and then in next older segment
- Run merging and compaction from time to time to combine segments and to discard overwritten or deleted values

#### Making an LSM-tree (Log structured Merge Tree) out of SSTables
- Storage engines that are based on this principle of merging and compacting sorted files are often called LSM storage engines
- Lucene indexing engine for full-text search used by Elasticsearch and solr
- Given a word in a search query, find all the documents (web pages, product descriptions, etc.) that mention the word. This is implemented with a key-value structure where the key is a word (a term) and the value is the list of IDs of all the documents that contain the word (the postings list). In Lucene, this mapping from term to postings list is kept in SSTable-like sorted files, which are merged in the background as needed.

#### Performance Optimization
- LSM-tree algorithm can be slow when looking up keys that do not exist in the database:
  - You have to check memtable, then the segments all the way back to the oldest
- In order to optimize this kind of access, storage engines often use additional Bloom filters 
- A Bloom filter is a memory-efficient data structure for approximating the contents of a set. It can tell you if a key does not appear in the database, and thus saves many unnecessary disk reads for non existent keys

### B-Trees
- B-Tree: Most commonly used indexing structure
- Introduced in 1970
- Standard index implementation in almost all the relational databases
- Stood the test of time
- Like SSTables, B-trees keep key-value pairs sorted by key, which allows efficient key value lookups and range queries.
- Log Structured indexex break down db in to variable size segments and always write a segment sequentially.
- However B-trees break the db down intpo fixed-size blocks or pages, (4KB typically) and read or write one page at a time.
- Design corresponds more closely to the underlying hardware as discks are also arranged in fixed-sized blocks
- Each page has address or location but on disk instead of in memory
- ![img.png](images/Looking%20up%20a%20key%20using%20a%20B-tree%20index.png)
- Root page contains several keys and references to child pages
- Each child is responsible for a continuous range of keys and the keys between the references indicate where the boundaries between those ranges lie
- The number of references to child pages in one page of the B-tree is called the branching factor
- Btree algorithm ensures that the tree remains balanced: a B-tree with n keys always has a depth of O(log n). Most databases can fit into a B-tree that is three or four levels deep, so you don’t need to follow many page references to find the page you are looking for.
  - (A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB.)
- To make B-trees resilient, a on-disk WAL (write ahead log) is used to write each modification before its applied to the pages of the tree. 
  - This log is used to restore db in consistent state when db comes back up after crash
- Protection concurrent threads is achieved by lightweight locks

#### Comparing B-Trees and LSM-Trees
- Even though B-tree implementations are generally more mature than LSM-tree implementations, LSM-trees are also interesting due to their performance characteristics. 
- As a rule of thumb, LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads.
- Reads are typically slower on LSM-trees because they have to check several different data structures and SSTables at different stages of compaction.

- Write Amplification
  - One write to the database resulting in multiple writes to the disk over the course of the database’s lifetime
#### Advantages of LSM-trees
- LSM-trees are typically able to sustain higher write throughput than B-trees, partly because they sometimes have lower write amplification (although this depends on the storage engine configuration and workload), and partly because they sequentially write compact SSTable files rather than having to overwrite several pages in the tree. This difference is particularly important on magnetic hard drives, where sequential writes are much faster than random writes.

#### Downsides of LSM-trees
- A downside of log-structured storage is that the compaction process can sometimes interfere with the performance of ongoing reads and writes.

