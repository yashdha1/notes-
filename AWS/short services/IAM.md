> | Free service | Global service |  

root user has all the rights. 


You can create users and create groups. Easily. 
and assign permissions. 

All the important commands and thingy you can do with amazon IAM. 
# AWS CLI IAM Commands

Here's a comprehensive reference for IAM in the AWS CLI:

---

## 👤 Users

```bash
# List all users
aws iam list-users

# Create a user
aws iam create-user --user-name alice

# Delete a user
aws iam delete-user --user-name alice

# Get user details
aws iam get-user --user-name alice
```

---

## 🔑 Access Keys

```bash
# Create access key for a user
aws iam create-access-key --user-name alice

# List access keys
aws iam list-access-keys --user-name alice

# Deactivate an access key
aws iam update-access-key --user-name alice --access-key-id AKIAIOSFODNN7EXAMPLE --status Inactive

# Delete an access key
aws iam delete-access-key --user-name alice --access-key-id AKIAIOSFODNN7EXAMPLE
```

---

## 👥 Groups

```bash
# Create a group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group --user-name alice --group-name Developers

# List groups for a user
aws iam list-groups-for-user --user-name alice

# Remove user from group
aws iam remove-user-from-group --user-name alice --group-name Developers

# List all groups
aws iam list-groups
```

---

## 📜 Policies

```bash
# List all managed policies
aws iam list-policies --scope Local        # custom policies
aws iam list-policies --scope AWS          # AWS managed policies

# Attach a managed policy to a user
aws iam attach-user-policy \
  --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Attach a policy to a group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Attach a policy to a role
aws iam attach-role-policy \
  --role-name MyRole \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# List attached policies for a user
aws iam list-attached-user-policies --user-name alice

# Detach policy from user
aws iam detach-user-policy \
  --user-name alice \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### Create a Custom Policy

```bash
# Create policy from a JSON file
aws iam create-policy \
  --policy-name MyS3Policy \
  --policy-document file://policy.json
```

**policy.json example:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"]
    }
  ]
}
```

```bash
# Get a policy's details
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/MyS3Policy

# Delete a custom policy
aws iam delete-policy --policy-arn arn:aws:iam::123456789012:policy/MyS3Policy
```

---

## 🎭 Roles

```bash
# Create a role (with a trust policy)
aws iam create-role \
  --role-name MyEC2Role \
  --assume-role-policy-document file://trust-policy.json

# List all roles
aws iam list-roles

# Get a role
aws iam get-role --role-name MyEC2Role

# Delete a role
aws iam delete-role --role-name MyEC2Role
```

**trust-policy.json (for EC2):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 🔐 Inline Policies

```bash
# Put inline policy on a user
aws iam put-user-policy \
  --user-name alice \
  --policy-name MyInlinePolicy \
  --policy-document file://policy.json

# List inline policies for a user
aws iam list-user-policies --user-name alice

# Delete an inline policy
aws iam delete-user-policy --user-name alice --policy-name MyInlinePolicy
```

---

## 🛡️ MFA  -> 

by the AWS UI wala you can do this via the UI easily add the MFA types and add the MFA here and add them in here. \

```bash
# List MFA devices for a user
aws iam list-mfa-devices --user-name alice 

# Deactivate MFA device
aws iam deactivate-mfa-device \
  --user-name alice \
  --serial-number arn:aws:iam::123456789012:mfa/alice
```

---

## 🔍 Useful Audit Commands

```bash
# Simulate policy (check if action is allowed)
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject

# Get account authorization details (full IAM snapshot)
aws iam get-account-authorization-details

# Generate a credential report
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 -d

# Get account summary (counts of users, groups, etc.)
aws iam get-account-summary
```

---

## Tips

|Flag|Purpose|
|---|---|
|`--output table`|Human-readable table output|
|`--output json`|Default JSON output|
|`--query`|Filter output with JMESPath|
|`--profile`|Use a named profile|
|`--region`|Override default region|

**Example with filtering:**

```bash
# List only usernames
aws iam list-users --query 'Users[*].UserName' --output table
```

---

IAM is global (not region-specific), so you don't need `--region` for most IAM commands.


 