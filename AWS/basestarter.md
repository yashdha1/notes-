### Types of cloud computing services : 

**1. Infrastructure as a Service (IaaS):**

-> IaaS providers, such as Amazon Web Services (AWS), azure and GCP. Supply a virtual server instance and storage, as well as application programming interfaces (APIs) that let users migrate workloads to a virtual machine (VM). 
-> Users have an allocated storage capacity and can start, stop, access and configure the VM and storage as desired. IaaS providers offer small, medium, large, extra-large, and memory- or compute-optimized instances, in addition to enabling customization of instances, for various workload needs. 
-> The IaaS cloud model is closest to a remote data center for business users.

**2. Platform as as Service (PaaS):**

-> In the PaaS model, cloud providers host development tools on their infrastructures. 
-> Users access these tools over the internet using APIs, web portals or gateway software. PaaS is used for general software development, and many PaaS providers host the software after it's developed. 
-> Common PaaS products include Salesforce's Lightning Platform, AWS Elastic Beanstalk and Google App Engine

**3. Software as a Service (SaaS):**

-> SaaS is a distribution model that delivers software applications over the internet; these applications are often called web services. 
-> Users can access SaaS applications and services from any location using a computer or mobile device that has internet access. 
-> In the SaaS model, users gain access to application software and databases. One common example of a SaaS application is Microsoft 365 for productivity and email services.

--- 
**Key Differences and Use Cases**

- **Control vs. Convenience:** IaaS offers the most control, whereas SaaS provides the most convenience. Say you use a IaaS of the 
- **Usage:** IaaS is ideal for building infrastructure, PaaS for application development, and SaaS for end-user applications.
- **Flexibility:** These models (IaaS, PaaS, SaaS) are often referred to as the "cloud stack" because they build upon one another.
--- 
### Cloud Deployment Models and types of clouds : 

**1. Public Cloud**
Public cloud is **open to all** to store and access information via the Internet using the pay-per-usage method.In public cloud, computing resources are managed and operated by the Cloud Service Provider (CSP).**Example:** Amazon elastic compute cloud (EC2), IBM SmartCloud Enterprise, Microsoft, Google App Engine, Windows Azure Services Platform.

**2. Private Cloud**
Private cloud is also known as an **internal cloud** or **corporate cloud**. It is used by organizations to build and manage their own data centers internally or by the third party. It can be deployed using Opensource tools such as Openstack and Eucalyptus.

**3. Hybrid Cloud**
Hybrid Cloud is a combination of the public cloud and the private cloud.
**we can say: Hybrid Cloud = Public Cloud + Private Cloud.**
Hybrid cloud is partially secure because the services which are running on the public cloud can be accessed by anyone, while the services which are running on a private cloud can be accessed only by the organization's users.

**4. Community Cloud:**
Community cloud allows systems and services to be accessible by a group of several organizations to share the information between the organization and a specific community. It is owned, managed, and operated by one or more organizations in the community, a third party, or a combination of them. ![[iaas_focus-paas-saas-diagram-1200x1046.png.webp]]

--- 

## Terminologies 

 **1. Auto Scaling** 
A cloud computing feature that automatically scales up or down the number of computing resources being allocated to your application based on its computing requirements at any given moment. Autoscaling ensures that new instances are continuously increased during demand spikes are reduced during demand drops, guaranteeing lowered costs and consistent performance. Services like the AWS Auto Scaling service allows end-users to configure one unified scaling policy per application source, like a set of resource tags or an AWS CloudFormation stack.


**2. Cloudburst**
A quality of service metric used to gauge the scalability and performance of cloud applications within hosted cloud platforms. A positive cloudburst indicates that the cloud-based application is efficient and capable of managing application scalability. A negative cloudburst indicates an inability to handle a spike in demand. 

- **How it Works:** When on-premises servers reach a predefined capacity threshold (e.g., 80% CPU usage), the workload automatically scales out to a public cloud provider like AWS, Azure, or Google Cloud, note HPE and Atlassian.
- **Use Cases:** Ideal for unpredictable or seasonal demand, such as e-commerce, big data analytics, and CI/CD pipelines during software releases.
- **Benefits:** 
	1. **Cost Efficiency:** Pay-as-you-go pricing for public cloud resources only when needed.
	2. **Scalability & Flexibility:** Instantly access vast computing resources, according to HPE Europe and HPE US.
	3. **Performance:** Maintains consistent application performance during high-load periods.


 **3. Container Registry**
A central place for your team to manage container images, perform vulnerability analysis, and decide on who has access to what with fine-tuned access controls. During the CI/CD process, developers should have access to all the container images required for an application. Hosting all the container images in a single instance enables users to identify, commit and pull images when they need to. 

**4. CDNs**
 is a system of distributed servers that used geographical proximity to provide alternative server nodes for users to download resources. Each node in a CDN caches static content like JavaScript/CSS files, images and other structural components. CDNs account for a vast amount of content available to end-users online. Companies such as Amazon and Microsoft operate their own CDNs to complement their various cloud offerings.

**5. Federated Databases**
is a system in which multiple databases seemingly function as one entity; however, each component database in the system exists independently of the others and is completely functional and self-sustained. When the federated database receives an application query, the system figures out which of its component databases contains that data being requested and passes the request to it. A federated database is a viable solution to database search issues.

**6. Grid Computing**
Is a distributed architecture of multiple computers connected to solve a complex problem. Unlike traditional networks that primarily focus on communication between devices, grid computing harnesses the unused processing cycles of connected computers to solve a problem that is too complex for any stand-alone machine. The computers are either directly connected or through scheduling systems.








CronJobs are usually serverless. Cause those are just easy to complete via them.. 






























IAM
EC2 
Lambdas
VPC
Fargate 
ECR 
ECS
EKS
Terraform 
Cloudfront
route 53 
DYNAMODB
RDS


SNS
SQS 
EVENTBRIDGE 
ELB
ALB
API GATEWAY
COGNITO
BEANSTALK 

TOOLS : 
TERRAFORM 
























## Core Concepts

### 1. Core Python Fundamentals

- **Data Structures & Algorithms**
    - When and why to use list, tuple, dict, set?
    - What’s the cost of searching in each built-in type?
    - What is list comprehension and why/when use it?
    - Difference between shallow and deep copy
    - How do you avoid bugs with default parameters?
- **Intermediate OOP**
    - When is inheritance appropriate and when is composition better?
    - How does method resolution order (MRO) work?
    - What are class/static methods vs instance methods?
    - Why and when would you use @property?
    - Common pitfalls with mutable class variables
- **Modules & Package Management**
    - How do you structure a multi-module project?
    - What’s the role of **init**.py?
    - Common module import issues (circular deps, relative vs absolute imports)

### 2. Basic Web Concepts

- **Framework Usage (Flask/Django/FastAPI)**
    - What triggers a new request cycle in a web app?
    - How does the framework know what function to call for a URL?
    - Difference between synchronous and asynchronous endpoint handling
    - When should you use blueprints (Flask) or apps (Django)?
    - Why static file handling is treated separately
    - What is the difference between rate limiting and throttling?
- **URL Routing & View Design**
    - What are path, query, and body parameters?
    - What makes an API RESTful?
    - How would you ensure a route only accepts certain HTTP methods?
    - Handling missing/wrong parameters and bespoke error responses
    - Use-case for pagination, filtering, sorting
- **Session & State Handling**
    - Difference between stateless and stateful APIs
    - Why and when use cookies, sessions, or JWT?
    - Risks and limitations of cookie-based sessions

### 3. Data Management

- **Relational Database Integration**
    - Advantages of using ORM
    - Common edge cases (N+1 query problem, connection pooling)
    - Why use transactions?
- **Schema Evolution**
    - Risks in managing schema migrations
    - Methods for rolling back failures
    - Strategy for handling backward-incompatible changes

### 4. Testing & Quality Assurance

- **Unit and Code Quality Testing**
    - Purpose of unit vs integration tests
    - Common mistakes (not mocking dependencies)
    - Testing error scenarios
    - Static vs Dynamic analysis

### 5. Deployment & Operations

- **Basic Docker & Container Concepts**
    - Use-case for containers in development/deployment
    - Potential pitfalls (large images, environment management)
    - Difference between containers and virtual machines
- **Simple CI/CD Pipelines**
    - Stages in a basic pipeline (test, build, deploy)
    - Rollback strategies in deployment failures
    - Environment segregation (dev, staging, prod)

### 6. Cloud Concepts (AWS/Azure)

- **Deploying on IaaS/PaaS**
    - Instance/VM vs Platform/App Service — what changes for the developer?
    - Risks with networking, ports, firewalls, environment variables
- **Serverless Technologies**
    - What is serverless? Example use-case (event-driven, short-running)
    - Cold starts: what are they and why do they matter?
    - Challenges with state persistence in serverless
    - Resource/time limits in Lambda/Azure Functions
    - When would you use serverless vs a container/on-demand instance?
- **Storage Services**
    - Object storage use-cases vs block storage (e.g., S3, Azure Blob)
    - How do you secure access (IAM, SAS tokens)?
    - Common challenges: file consistency, access permissions, lifecycle policies
    - Edge cases: eventual consistency, reading/writing large files
- **On-Demand Instances**
    - Benefits (elasticity) and tradeoffs (cost/cache/data persistence issues)
    - Handling instance interruptions (statelessness, recovery)
    - Challenges: unpredictable availability, cost management
- **Container Services**
    - What are container orchestration services (ECS/EKS, AKS)?
    - Benefits vs drawbacks vs self-hosted Docker
    - Challenges: networking, autoscaling, persistent storage integration
    - How do container services differ from running Docker directly?
    - When would you use ECS/EKS or AKS vs self-managed cluster?

---

## Advanced Concepts

### 7. Advanced Python

- **Architectural Design & Patterns**
    - When would you use dependency injection?
    - Comparing Repository, Factory, Singleton: their usage in a backend app
    - How do SOLID principles help in refactoring and scaling code?
    - Tradeoffs when applying patterns (e.g., overengineering, complexity)
- **Decorators, Context Managers, Meta-programming**
    - Typical use-cases for decorators (timing, access control, memoization)
    - What can go wrong with complex decorators?
    - Why/when would you create a custom context manager?
    - What are metaclasses and when do they make sense?
    - Risks: readability, maintainability in advanced Python
- **Python Internals & Performance**
    - How does the GIL affect backend server concurrency?
    - Memory leaks in Python web apps—causes and mitigations
    - Profiling: what tools/techniques can identify CPU/memory bottlenecks?
    - When would you use async IO vs multiprocessing vs threading?
    - What is thread safe code?
    - What are Coroutines and Event loop?

### 8. Advanced Web/API Concepts

- **Middleware Design & Advanced Routing**
    - Custom middleware for auth/logging—common mistakes
    - Routing strategies for large-scale applications
    - Error handling and propagation patterns
- **Application Modularization**
    - Use-case for blueprints/apps/plugins
    - Pitfalls in module boundaries (tight coupling, circular imports)
- **API Versioning, Documentation & Schema Evolution**
    - Strategies for backward-compatibility
    - Common mistakes in API docs/autogen tools
    - Schema evolution and breaking change strategies
- **Enterprise Auth & Security**
    - OAuth2 vs JWT vs SSO scenarios
    - Edge cases (token revocation, refresh, third-party federated auth)
    - Rate-limiting, auditing, logging for compliance
- **Microservices & Service Communication**
    - Why use microservices; common pitfalls
    - Choosing between HTTP, gRPC, WebSockets
    - Handling distributed transactions and eventual consistency

### 9. Advanced Data Management

- **Complex Queries, Indexing, and Performance**
    - Identifying and fixing slow queries
    - Indexing strategies/pitfalls
    - Impact of joins and subqueries on performance
- **NoSQL & Distributed Databases**
    - When to use Redis/MongoDB over relational DBs
    - Edge cases with distributed consistency
    - Optimistic vs pessimistic locking strategies

### 10. Deployment, Scalability & Reliability

- **Kubernetes, Service Discovery, and Orchestration**
    - Benefits/edge cases of orchestration for microservices
    - Risks in rollout/release strategies
    - Load balancing, health checks, service discovery patterns
- **Async Programming & Event Processing**
    - When async is essential; common pitfalls with concurrency
    - Queue/task errors and retry patterns
    - Event-driven architectures (SQS, SNS, Event Hub)
- **Advanced Caching & CDN Integration**
    - Strategies for cache invalidation
    - Edge cases in cache consistency and data freshness
    - Using CDNs for API and static resource distribution

### 11. Advanced Cloud & DevOps (AWS/Azure)

- **Infrastructure as Code & Automation**
    - Difference between CloudFormation, Terraform, and ARM templates
    - IaC edge cases: dependencies, race conditions, drift detection
    - Automating deployments and disaster recovery
- **Cloud-Native Networking & Security**
    - VPCs, subnets, security groups—how do they interact?
    - Zero trust architectures in AWS/Azure
    - Network bottlenecks and troubleshooting techniques
- **Serverless & Event-Driven Architectures**
    - Orchestrating workflows with Lambda/Functions and SQS/Event Hub
    - Limitations and scaling bottlenecks in event-driven design
    - Edge-case: event ordering, at-least-once vs exactly-once processing
- **Container Orchestration & Advanced Services**
    - Designing for high-availability (multi-AZ/region deployments)
    - Managing persistent storage in Kubernetes/ECS/AKS
    - Common pitfalls: rolling updates, blue/green deployments
- **Monitoring, Logging & Cost Optimization**
    - What tools/services would you use (CloudWatch, Azure Monitor, custom stacks)?
    - How to debug a slow/stuck cloud-native app?
    - Edge cases: cost overrun alarms, log storage management, security for logs/metrics APIs