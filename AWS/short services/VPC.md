A [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-your-vpc.html) is a virtual network that closely resembles a traditional network that you'd operate in your own data center. After you create a VPC, you can add subnets. Logically isolated private network. 

**Subnets** -> A subnet s a range of IP addresses in your VPC. A subnet must reside in a single Availability Zone. After you add subnets, you can deploy AWS resources in your VPC.

* ***IP addressin**g -> You can assign [IP addresses](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html), both IPv4 and IPv6, to your VPCs and subnets. You can also bring your public IPv4 addresses and IPv6 GUA addresses to AWS and allocate them to resources in your VPC, such as EC2 instances, NAT gateways, and Network Load Balancers.


*  **Route Tables** 
Each VPC contains an implicit virtual router that relies on Route Tables to direct traffic. Every subnet must be associated with exactly one route table, and each route defines.  Route tables determine whether traffic remains internal to the VPC or is sent to external networks. They are the digital roadmap of the cloud network. 


* **NACL -> network access control layer :** 
Firewall at the subnet level.  NACLs are stateless firewalls that control inbound and outbound traffic at the subnet level, evaluating each packet independently. Every VPC has a default NACL that can be modified but not deleted, and rules must be explicitly defined for both inbound and outbound traffic.

--- 

# **Route53:** Domain resolver 

A **global, distributed, cached, hierarchical database** optimized for speed and resilience. 
##### The 8 steps in a DNS lookup: 

1. A user types ‘example.com’ into a web browser and the query travels into the Internet and is received by a DNS recursive resolver.
2. The resolver then queries a DNS root nameserver (.). https://www.netnod.se/i-root/what-are-root-name-servers 
3. The root server then responds to the resolver with the address of a Top Level Domain (TLD) DNS server (such as .com or .net), which stores the information for its domains. When searching for example.com, our request is pointed toward the .com TLD.
4. The resolver then makes a request to the .com TLD.
5. The TLD server then responds with the IP address of the domain’s nameserver, example.com.
6. Lastly, the recursive resolver sends a query to the domain’s nameserver.
7. The IP address for example.com is then returned to the resolver from the nameserver.
8. The DNS resolver then responds to the web browser with the IP address of the domain requested initially.

Once the 8 steps of the DNS lookup have returned the IP address for example.com, the browser is able to make the request for the web page:

https://www.geeksforgeeks.org/computer-networks/working-of-domain-name-system-dns-server/ 

10. The browser makes a [HTTP](https://www.cloudflare.com/learning/ddos/glossary/hypertext-transfer-protocol-http/) request to the IP address.
11. The server at that IP returns the webpage to be rendered in the browser (step 10).



## ELB vs ALB 


[What is an Application Load Balancer? - Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) 

ex: ALB can be used in microservices settings. 

just put the ec2 instance with a security group and the inbound trafic and the reverse proxy outbound traffic would be sorted. 


sticky load balancing : cookie based : or can Also be done via the authorisation headers. 



Auto scalling groups : free way to scale in or scale out via sc2 instances. 


CIDR : In AWS, **CIDR (Classless Inter-Domain Routing)** defines the IP address range for a **Virtual Private Cloud (VPC)** and its subnets. Every VPC must have a **primary IPv4 CIDR block** and can optionally have **additional IPv4** and **IPv6 CIDR blocks**.   