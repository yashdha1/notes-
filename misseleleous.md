#### 1. Bloom Filters
- A **Bloom filter** is a compact probabilistic data structure used to quickly test whether an element is **definitely not present** or **possibly present** in a set. 
- Databases use Bloom filters to avoid expensive disk reads, network lookups, or joins when they already know a value cannot match. 
- eg. very basic is the **Trie datastructure**. 
>_ Could this row/key/value exist?

--- 

#### 2. SS Tables -> in cassandra, scylla
- A Sorted Strings Table (SSTable) is ==an immutable, ordered, and persistent file format used by LSM-tree based databases like Apache Cassandra and ScyllaDB to store data on disk== 
- Once written to disk, an SSTable file is never modified. Updates create new SSTables, while old ones are cleaned up later by compaction. you first write into the logs and then fanout in the db later. 
- Build for High volume ingestion workflows.  https://www.scylladb.com/glossary/sstable/ 

---
