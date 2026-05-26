%%  %%


Remember on the localstack everything is mocked 
security groups and the ami images and everything. 


Here's a practical AWS CLI reference for EC2:
## Instances

```bash
# List all instances
aws ec2 describe-instances

# List with a readable summary
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,InstanceType]' --output table

# Start / Stop / Reboot / Terminate
aws ec2 start-instances --instance-ids i-0abc123
aws ec2 stop-instances --instance-ids i-0abc123
aws ec2 reboot-instances --instance-ids i-0abc123
aws ec2 terminate-instances --instance-ids i-0abc123

# Get instance status
aws ec2 describe-instance-status --instance-ids i-0abc123
```

---

##  Launch an Instance

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-0abc123 \
  --subnet-id subnet-0abc123 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'
```

---

#### Key Pairs

generating unique keys to access the instances without password login everytime. rather using the public keys to login in the services. Example are RSA keys .pem keys that are created in the process of all of this. 

```bash
# List key pairs
aws ec2 describe-key-pairs

# Create a key pair (saves private key)
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
chmod 400 MyKeyPair.pem

# Delete a key pair
aws ec2 delete-key-pair --key-name MyKeyPair
```

---

## Security Groups

#### Security Groups

You can enforce rules over either. You attach a security group to an EC2 instance. It checks all incoming and outgoing traffic, Only traffic matching the rules is allowed. 

1. **Inbound traffic :** Control traffic coming _into_ your instance. example limits IP access to your ports and traffic on the ports. eg - Allow HTTP and SSH. 
2. **Outbound Rules** : outgoing traffic control. Allow all outbound traffic (default). Restrict access to specific services if needed. 

```bash
# List security groups
aws ec2 describe-security-groups

# Create a security group
aws ec2 create-security-group --group-name MySG --description "My SG" --vpc-id vpc-0abc123

# Allow SSH (port 22) inbound
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp --port 22 --cidr 0.0.0.0/0

# Revoke a rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp --port 22 --cidr 0.0.0.0/0
```

---

## AMIs & Snapshots

An **AMI [Amazon machine images]** is a **pre-configured template** used to launch EC2 instances. 

Snapshots are just volumes for the EC2 instances in the S3 buckets when they are closed. Stored in Amazon S3 (managed by AWS internally), making them: (1) Highly durable, (2) Automatically replicated. 

#### What it contains:
- Operating System (Linux, Windows, etc.), Installed software/applications, Configuration settings and Block device mappings (storage info). 

#### Purpose:

- Quickly launch new instances with the same setup
- Create reusable environments

```bash
# List your AMIs
aws ec2 describe-images --owners self

# Create an AMI from an instance
aws ec2 create-image --instance-id i-0abc123 --name "MyBackup" --no-reboot

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Create a snapshot
aws ec2 create-snapshot --volume-id vol-0abc123 --description "My Snapshot"

# Delete a snapshot
aws ec2 delete-snapshot --snapshot-id snap-0abc123
```

---

## 📦 Volumes (EBS)


high performance block storage for the AWS.  Amazon EBS allows you to create storage volumes and attach them to Amazon EC2 instances. Once attached, you can create a file system on top of these volumes, run a database, or use them just like you would use block storage. Amazon EBS volumes are placed in a specific Availability Zone, where they are automatically replicated to protect from the failure of a single component. All EBS volume types offer durable snap.

```bash
# List volumes
aws ec2 describe-volumes

# Create a volume
aws ec2 create-volume --size 20 --volume-type gp3 --availability-zone ap-south-1a

# Attach a volume
aws ec2 attach-volume --volume-id vol-0abc123 --instance-id i-0abc123 --device /dev/xvdf

# Detach a volume
aws ec2 detach-volume --volume-id vol-0abc123
```

---

## 🌐 Elastic IPs

```bash
# Allocate an Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with an instance
aws ec2 associate-address --instance-id i-0abc123 --allocation-id eipalloc-0abc123

# Release an Elastic IP
aws ec2 release-address --allocation-id eipalloc-0abc123
```

---

### Elastic Load Balancer : 

Load balancing functionalities in the gateway and stuff. Elastic Load Balancing (ELB) automatically distributes incoming application traffic across multiple targets and virtual appliances in one or more Availability Zones (AZs). 
## 💡 Handy Tips

|Tip|Command|
|---|---|
|Filter by state|`--filters "Name=instance-state-name,Values=running"`|
|Set default region|`aws configure set region ap-south-1`|
|Use profiles|`--profile myprofile`|
|Dry run (test perms)|`--dry-run`|
|Output as JSON/table/text|`--output json`|

---

AMAZON EBS (elastic block storage) and EFS(elastic file storage) and ELB (elastic load balancer) 

putty for the sercure remote connection to the remote machine over the same network. 


---



































