a service  that helps you juggel all the services and deploy the application together. You dont need to self configure shit. You can do this via the beanstalk . It has access to the S3, IAM, ec2 and all the things that it can possible need for the deployment. 

#### Types of deployment : Beanstalk Deployment Options for Updates 

• All at once (deploy all in one go) **Big Bang** -> fastest, but instances aren’t available to serve traffic for a bit (downtime) 
• **Rolling**: update a few instances at a time (bucket), and then move onto the next bucket once the first bucket is healthy 
• **Rolling with additional batches:** like rolling, but spins up new instances to move the batch (so that the old application is still available) 
• **Immutable**: spins up new instances in a new ASG, deploys version to these instances, and then swaps all the instances when everything is healthy. 
• **Blue Green**: create a new environment and switch over when ready. *prolly the best one.* 
• **Traffic Splitting:** canary testing – send a small % of traffic to new deployment. *prolly the best one.* 

