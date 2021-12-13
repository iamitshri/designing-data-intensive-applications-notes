# Chapter 5: Replication
- 

# Main Points
- Replication is copying data to another server to achieve benefits such as 
  - Read Scalability: Read requests can be off loaded to read-replicas
  - Improve availability: Keeping the system running when one or more machine goes down 
  - Reducing the latency by keeping the data closer to its end users

- Types of Replication
- Single Leader
  - Leader accepts all the writes, it asynchronously copies the changes to followers aka replicas
  - Leader sends CDC events to follower replicas.  Mysql uses row based replication and use log sequence number to mark each change in Write ahead log.
- Multi Leader
  - All replicas accepts write requests from clients, leaders asynchronously resolve write conflicts
- Leaderless 
  - Read and write requests are sent to all the replicas
  - Read & write quorum is used to decide valid read and write requests
  - Client sends read requests to all replicas, waits for response only from the 'r' replicas
  - Similarly write requests are sent to all replicas, client only waits for successful response from 'w' replicas
  - If n is number of servers (typically an odd number), then n > r + w for better guarantee of correct read or write response

- Replication can be done 
  - Synchronous
    - If there are more than one replicas, all replicas are up-to-date with leader, but replication failure to one node causes client to see write failure
  - Asynchronous
    - Clients can do faster writes, followers replicas are eventually consistent.
    - Replication lag may cause read requests to see stale data 

Replication lag and multiple follower replica create consistency issues such as 
- Monotonic read Consistency:  (Strongly Consistent  > Monotonic read > Eventual consistency)
  - After users have seen the data at one point in time, they should not later see data in some earlier point in time
  - e.g. User visits a webpage sees a comment to its post, later  refreshing the page comment should not disappear unless its deleted 
- Read-After-Write Consistency
  - Users should always see data that they submitted themselves. 
  - e.g. If a users updates profile on social media, changes should still be visible after page refresh by the same user. 
- Consistent Prefix Reads (Causal dependency)
  - Users should see the data in a state theat makes causal sense. e.g. seeing a question and its reply in the correct order