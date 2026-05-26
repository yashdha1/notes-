*key words* 

- Partioning keys 
- Consistent Hashing
- Microservices 
- Load balancing 

### Consistent Hashing  IMP
DEFINATION -> Quoted from Wikipedia: *"Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of total slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be remapped.*  

TLDR ~When the number of buckets/nodes changes, only a small fraction of keys need to move. 

this makes the consistent hashing different than the traditional hashing algorithm.  

---

### KV stores  IMP

https://bytebytego.com/courses/system-design-interview/design-a-key-value-store 

- Single Server KV store. Can be implemented in simple HashTable structures. this is easier.

#### Distributed KV stores. 
When designing a distributed system, it is important to understand CAP (**C**onsistency, **A**vailability, **P**artition Tolerance) theorem. 

Since we building a **Partition Tolerance** system. We need to think what we want **highly consistent** or **highly available.**  

##### CAP Brush over: 
**Consistency**: consistency means all clients see the same data at the same time no matter which node they connect to.
**Availability:** availability means any client which requests data gets a response even if some of the nodes are down.
**Partition Tolerance:** a partition indicates a communication break between two nodes. Partition tolerance means the system continues to operate despite network partitions.

---

**CP systems** 
If we choose consistency over availability (CP system), we must block all write operations to _n1_ and _n2_ to avoid data inconsistency among these three servers, which makes the system unavailable. Bank systems usually have extremely high consistent requirements. For example, it is crucial for a bank system to display the most up-to-date balance info. If inconsistency occurs due to a network partition, the bank system returns an error before the inconsistency is resolved. 

**CA systems** 
if we choose availability over consistency (AP system), the system keeps accepting reads, even though it might return stale data. For writes, _n1_ and _n2_ will keep accepting writes, and data will be synced to _n3_ when the network partition is resolved.  

--- 
**Core Components** 

https://medium.com/nerd-for-tech/quorum-consensus-56bc1bacb0d2  


- Data partition    -> dividing the data 
- Data replication -> duplicating the data
- Consistency        -> crash of one node does not results in the failure of requests.
- Inconsistency resolution ->  
- Handling failures
- System architecture diagram
- Write path
- Read path

Good to know : 

 1. **Consistency Models**  
Consistency model is other important factor to consider when designing a key-value store. A consistency model defines the degree of data consistency, and a wide spectrum of possible consistency models exist:

- ***Strong consistency***: any read operation returns a value corresponding to the result of the most updated write data item. A client never sees out-of-date data. Postgres MVCC. and Qorum conseses based systems are needed here. Raft and Paxos Conseses algorithms to agree upon the state of the entire system before commit. 
- Weak consistency: subsequent read operations may not see the most updated value.
- ***Eventual consistency***: this is a specific form of weak consistency. Given enough time, all updates are propagated, and all replicas are consistent. Cassandra, Dynamo DB are eventually consisistent. Event based. 

--- 

  **Paradigms in system design**![[0026-10-system-design-trade-offs-you-cannot-ignore.png]]