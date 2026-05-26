Indivisual concepts to know when approaching systems design. 
*[FAST NOTES]* -> need to do these topics in detail too.
1. Load balancing : shotload of all of this shit. 
2. Consistent Hashing : 
3. Horizontal and Vertical scalling
4. Task Queues and Messaging Queues. 
5. Monolith vs Microservices
	1. Monolith : Small team,  harder to scale Less Complicated, Faster, No RPC. 
		-> no agile, colapse can be fatality, Complicated deployments, single point of failures. harder to scale, engineer need to know the entire system to proceed everytime. 
	2. Microservices : Saclablity, centric knowloedge for the enginners. parallel devlopment is easier, streamlined approach and efficient. 
		-> harder to setup and design. 

6. Database sharding wali shit 
		-> memchache.
		-> fix the joins across the shards of the databases. 
		-> Flxiliblity. 
		-> shard indexing.
		-> master slave architecture. 
		-> Hadoop wali shit of mapReduce. 

7. Caching : LRU, LFU, Redis, 
		-> nuksaan - negative cache, Thrashing
		-> in meory cache, db cache, redis cache
		-> how to apply all of these services. 

8. Single Point of failure : [domino wali shit] [we dont want this] 
		-> Single point of failure means that your architecture is not resillient. 
		-> to mitigate a single point of failure in the scene. We need to find ways around it. 
		-> duplicate nodes , load balancers , and the Copies of databases.
		-> in disaster : regisnal wali system. if everything fails. connect to diffrent region. 
		-> *chaos monkey* -> wali shit. Do this. -> go to the server and randomly take down one server to check if the server is really fault tolerent. 

9. CDN : content delivery network 
		-> speed, regionality, regulations, effiiency. Caching in CDN.

https://roadmap.sh/system-design 
https://medium.com/@lazygeek78/system-design-of-netflix-8a31bf9ca53f 



**AWS Fargate**  vs Lambda -> both are serverless cloud compute in AWS. 

FARGATE -> container based lambda 
LAMBDA -> function based lambda 

If your project involves event-driven, short-duration tasks or unpredictable workloads, AWS Lambda might be the better fit. If you need to run containerized applications with specific resource needs or require persistent processes, AWS Fargate would be more appropriate. 


**AWS ECS and ECR** -> amazon elastic container registry. 


You can run containerised application on AWS as a serverless services or as a servered services. 
ie. via the EC2 or the just have a cluster of elastic containers running, ie. Elastic Fargate; you just have task definations and the fargate manages the clustere o images 

- **EventBridge**
    - Built for **event-driven architectures**
    - Routes events between services based on rules
    - Think: “When X happens, send it to Y and Z”
    - - Push-based (automatically delivers events to targets)
	- Uses rules to filter and route events
	- doesn’t store events long-term (short-lived)
	- Focus is on delivery, not persistence. 
	- MENTAL MODEL : like redis pubsub. fire and forget. very fast. 
- **SQS**
    - Built for **reliable message processing**
    - Stores messages until a consumer processes them
    - Think: “Hold this task until a worker is ready 
    - - Pull-based (consumers poll the queue)
	- No routing logic — just stores messages. 
	-  Stores messages up to **14 days**
    - Acts as a buffer between services
    - MENTAL MODEL : like redis streams. remember shit. 



Read the Desicision Guides on the official aws documents. 
