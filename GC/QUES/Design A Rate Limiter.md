
mother link -> https://bytebytego.com/courses/system-design-interview/design-a-rate-limiter 

Why? 
1. prevent DOS attacks -> malicious attempt to make a server, network, or website unavailable to users by overloading it with excessive traffic or requests. RATE LIMITERS PREVENT THIS. 
2. Reducing Cost By reducing the number of requests from the same origin. 
3. too many requests -> *ERROR 429*


## TOPICS 
### **Client side Rate Limiting** 

**TOPICS**
	1. Debouncing -> waiting for users to finish 
	2. throtlling -> time based 
	3. Token Buckets (refillable) -> specifinc token of requests
	4. Leaeky Bucket -> Queue based throtlling. 
		* Libraries in this Client side Rate limiting .
			* `lodash.debounce`
			- `lodash.throttle`
			- `Bottleneck`
			- `p-queue`
		
### **Server Side Rate limiting.**  

**TOPICS** 
	1. API GATEWAY -> (nginx, kong, apisix)
	2. server-side rate limiting. ALB and ELB and shits
	3. Distributed event based rate limiting. Redis based.
	4. Application level Rate Limiting. (using inbuilds libs and framework provided resources).

**REMEMBER**
	1. Throtlling vs Limiting  -> https://www.geeksforgeeks.org/system-design/api-throttling-vs-api-rate-limiting-system-design/ 
	2. SSL termination. 

**ALGORITHMS**
	1. Token bucket. *fixed request tokens* and the *variable request tokens amt*. 
	2. Leaking bucket. Queue based. eg.(Allow only 10000 request per sec on servers.) 
	3. Fixed window counter.
	4. Sliding window log
	5. Sliding window counter


this can be a good reference image.
	option 1 -> rate limited     -> *dropped*
	option 2 -> rate throttlled -> *queued*


![[figure-4-13-G2VF2RCQ.webp]]



Q. Rate limiting in distributed enviorment? 
-> issues . *race condition* and *syncronization* issues. 

Q. Ip based rate limiting? 
-> https://blog.programster.org/rate-limit-requests-with-iptables 


