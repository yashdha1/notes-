System design -> 

1. **Back-of-the-envelope Estimation** -> https://highscalability.com/google-pro-tip-use-back-of-the-envelope-calculations-to-choo/ 
2. **High Level Design Doc** -> https://bytebytego.com/courses/system-design-interview/a-framework-for-system-design-interviews 
3. **How does the Stack overflow is Monolith** -> https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/#core-hardware 
4. OLTP vs OLAP 
5. framework for system design interviews : https://bytebytego.com/courses/system-design-interview/a-framework-for-system-design-interviews
6. edge computing :  https://www.cloudflare.com/learning/serverless/glossary/what-is-edge-computing/ 
7. **Qourom Conssenses** : https://medium.com/nerd-for-tech/quorum-consensus-56bc1bacb0d2 
8. Random Number generator in a distributed system : https://bytebytego.com/courses/system-design-interview/design-a-unique-id-generator-in-distributed-systems 
9. **SQS, SNS and event bridge** -> https://docs.aws.amazon.com/decision-guides/latest/sns-or-sqs-or-eventbridge/sns-or-sqs-or-eventbridge.html 
10. How do DB scales the READ/Write operations : 
11. some more resources here related to system design -> https://algomaster.io/ 
12. **Caching** and *types of caches* -> https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/   cache aside, cache read through, cache write throgh syncronous  (pehele cache mein likho then db mein, dynamoDB ), asyncronous write through cache.   
13. Load balancer -> https://www.f5.com/glossary/load-balancer -> sticky sessions, high throughput loadbalancing, algorithms round robin, consistent hashing, partition key, alsos the AWS provided family of load balancers. HAProxy, nginx etc. 
14. nginx by f5 -> event driven (async behavior at network layer),  
15. http vs http2 -> why is http2 better? multiplexing, binary transfer, 
16. Web pages ->  https://bytebytego.com/guides/how-does-the-browser-render-a-web-page/ 
17. server response protocols to clients  :- polling, short polling, SSE and sockets. 
18.  Apache Kafka vs Apache Pulsar. Usage with the event processors like **Flink** and **Kafka Streams** and **Spark**. 
19. kafka kraft -> https://developer.confluent.io/learn/kraft/ 
20. How to shard -> https://www.pingcap.com/blog/database-sharding-defined/  levels -> application layer, middleware level, build-in (MySQL)
21. Distributed SQL -> https://www.pingcap.com/blog/why-distributed-sql-databases-elevate-modern-app-dev/
22. **Cockroach DB** : distributed ACID compliant system (replication solution), uses *raft quorum conseses.*  
23. hybrid transactional and analytical processing **(HTAP)**.
24. Redis Cluster and distributed caching systems. -> https://medium.com/@rajatpachauri12345/what-are-redis-cluster-and-how-to-setup-redis-cluster-locally-69e87941d573
25. **Load Distribution Algorithms** : Deterministic Hashing, Consistent Hashing, range based distribution and ***Virtual Bucket Hashing***. 
26. **B-Tree vs LSM-Tree**-> Indexing types and do these topics **(Bloom filters, Skip lists, SSTables)**
		{read, write} -> {logn, logn} in **btrees** vs {logn, 1} in **lsm**  
27. **Event Sourcing Systems** ( maintaining the history of change) vs **Non Event Sourcing** (Not maintaining shit, normal systems) types of systems. 
28. **Development Patterns** -> **Clean Architecture** (layered architecture. **hexagonal architecture**) -> https://dev.to/alwil17/building-a-production-ready-fastapi-boilerplate-with-clean-architecture-5757 , **MVC** , **Onion**, **CQRS** . 
29. Types of Caching  -> **Cache aside** , **Read through** , **Write around**, **Write back**  and  **Write through**.
30. Designing for High Availablity vs High Consistency.  AP vs CP deliemma and solutions available. 
31. 
**project-3** | Discussion Forum | CLI Project

TOPICS LIST : 

KAFKA, Kraft, Reddis, FASTAPI, SQLALCHEMY, SQL, RabbitMQ, 
- since we can replay the event in kafka how can we ensure that the same event is not consumed multiple times ?
- 
