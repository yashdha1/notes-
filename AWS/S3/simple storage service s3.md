You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere on the web. Use Amazon S3 to store, manage, analyze, and protect any amount of data for virtually any use case, such as data lakes, cloud-native applications, and mobile apps. 

Amazon S3 offers a range of storage classes designed for different use cases. You can store 

1. mission-critical production data in S3 Standard or S3 Express One Zone for frequent access. single digit latency. 
2. save costs by storing infrequently accessed data in S3 Standard-IA or S3 One Zone-IA. 
3. archive data at the lowest costs in S3 Glacier Instant Retrieval, S3 Glacier Flexible Retrieval, and S3 Glacier Deep Archive.
https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html#S3Features 


**# Storage Management in S3**

- [S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html) – Configure a lifecycle configuration to manage your objects and store them cost effectively throughout their lifecycle. You can transition objects to other S3 storage classes or expire objects that reach the end of their lifetimes.
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) – Prevent Amazon S3 objects from being deleted or overwritten for a fixed amount of time or indefinitely. You can use Object Lock to help meet regulatory requirements that require _write-once-read-many_ _(WORM)_ storage or to simply add another layer of protection against object changes and deletions.
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html) – Replicate objects and their respective metadata and object tags to one or more destination buckets in the same or different AWS Regions for reduced latency, compliance, security, and other use cases.
- [S3 Batch Operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/batch-ops.html) – Manage billions of objects at scale with a single S3 API request or a few clicks in the Amazon S3 console. You can use Batch Operations to perform operations such as **Copy**, **Invoke AWS Lambda function**, and **Restore** on millions or billions of objects


**RBAC with Amazon RBAC**  : 
IAM - Identity and access management, IAM is a web service that helps you securely control access to AWS resources, including your Amazon S3 resources. 

**Event based notification service** : 
Event based notification service, Amazon Simple Notification Service (Amazon SNS), Amazon Simple Queue Service (Amazon SQS), and AWS Lambda when a change is made to your S3 resources. 

**Monitoring Tools** : 
**Cloud watch ->** Track the operational health of your S3 resources and configure billing alerts when estimated charges reach a user-defined threshold.
**CloudTrail ->**  Record actions taken by a user, a role, or an AWS service in Amazon S3. CloudTrail logs provide you with detailed API tracking for S3 bucket-level and object-level operations. 

**Analytics and Insights** : 
*Amazon S3 Storage Lens*  Understand, analyze, and optimize your storage. S3 Storage Lens provides 60+ usage and activity metrics and interactive dashboards to aggregate data for your entire organization, specific accounts, AWS Regions, buckets, or prefixes.


How does the S3 Works?  layman...

Amazon S3 is an object storage service that stores data as objects, hierarchical data, or tabular data within buckets. An _object_ is a file and any metadata that describes the file. A _bucket_ is a container for objects.
To store your data in Amazon S3, you first create a bucket and specify a bucket name and AWS Region. Then, you upload your data to that bucket as objects in Amazon S3. Each object has a _key_ (or _key name_), which is the unique identifier for the object within the bucket.
	S3 provides features that you can configure to support your specific use case. For example, you can use S3 Versioning to keep multiple versions of an object in the same bucket, which allows you to restore objects that are accidentally deleted or overwritten.
Buckets and the objects in them are private and can be accessed only if you explicitly grant access permissions. You can use bucket policies, AWS Identity and Access Management (IAM) policies, access control lists (ACLs), and S3 Access Points to manage access.


### Amazon ECS - (Elastic Container Service)

containerisation service in the AWS. 


*IaC -> infrastrcuture as a code.*
Writing really simple config files to achive the state of the architecture needed. Faster. realiable and simpler. Idempotent. Immutable vs mutable infrastrucure. 

Declarative vs Imperative approaches. <- imp
integrations with the CICD pipelines. 

* ***IaC Tools**  Terraform, AWS cloud formation. 
* Configuratieon mangement tools. -> Ansible, salt Stack  etc. 


ALL THE CLOUD SERVICES IN THE AMAZON : 

1. ec2 
2. s3
3. lambda
4. amazon rds
5. dynamoDB
6. amazon ECR -> contaierisation tool
7. amazon EFS -> file storage service
8. amazon glacier 

