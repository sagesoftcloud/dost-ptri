# Day 9 Labs — Security Fundamentals

**Program:** DOST PTRI — AWS Fundamentals Training Program

> ⚠️ **FOR TRAINING PURPOSES ONLY.** This lab guide is intended for educational use within the DOST PTRI AWS Fundamentals Training Program. Do not use in production environments.

---

## Lab Structure

| Lab | Activity | Time |
|-----|----------|------|
| 1 | Walkthrough: AWS Security Services (WAF, GuardDuty, Config, Security Hub) | 15 min |
| 2 | Create a Secret in AWS Secrets Manager | 10 min |
| 3 | NACL + Security Group Configuration (HTTP, HTTPS, SSH) | 35 min |
| 4 | AWS Policy Generator — Create IAM Policies (Beginner → Advanced) | 30 min |

**Total Time:** ~90 minutes

### Prerequisites

- AWS Console access
- Region: `ap-southeast-1` (Singapore)

---

## Lab 1 — Walkthrough: AWS Security Services (15 min)

**Objective:** Explore AWS security services through the console. This is a guided tour — no configuration, just observe and learn.

---

### Step 1: AWS WAF (3 min)

1. Go to the **WAF & Shield** console
2. Observe: **Web ACLs**, **Rules**, **IP sets**
3. Note: WAF protects API Gateway / ALB / CloudFront from SQL injection, XSS, and DDoS
4. Check if any Web ACLs exist in the account

> 📝 **What WAF does:** Acts as a web application firewall that filters malicious HTTP/HTTPS requests before they reach your application.

---

### Step 2: Amazon GuardDuty (3 min)

1. Go to the **GuardDuty** console
2. If not enabled, just view the landing page and features
3. Observe: Findings types — **Recon**, **Trojan**, **UnauthorizedAccess**
4. Note: GuardDuty = intelligent threat detection using ML, monitors CloudTrail / VPC Flow Logs / DNS

> 📝 **What GuardDuty does:** Continuously monitors for malicious activity and unauthorized behavior using machine learning and threat intelligence feeds.

---

### Step 3: AWS Config (4 min)

1. Go to the **AWS Config** console
2. Observe: **Rules**, **Resources**, **Compliance dashboard**
3. Look for any non-compliant resources (red indicators)
4. Note: Config = continuous compliance monitoring, tracks resource configuration changes over time

> 📝 **What Config does:** Records and evaluates resource configurations against desired settings. Think of it as a "configuration audit trail."

---

### Step 4: AWS Security Hub (5 min)

1. Go to the **Security Hub** console
2. Observe: **Security score**, **Findings**, **Standards** (CIS, AWS Foundational)
3. Check the security score percentage
4. Note: Security Hub = single pane of glass for all security findings

> 📝 **What Security Hub does:** Aggregates findings from GuardDuty, Config, Inspector, and third-party tools into one unified dashboard.

---

### 💡 Key Insight

> These services work together as a security ecosystem:
> - **GuardDuty** → detects threats
> - **Config** → checks compliance
> - **Security Hub** → aggregates everything
> - **WAF** → blocks attacks at the edge

---

## Lab 2 — Create a Secret in AWS Secrets Manager (10 min)

**Objective:** Store a database credential securely instead of hardcoding it in application code.

---

### Step 1: Create a Secret

1. Go to **Secrets Manager** → click **Store a new secret**
2. Secret type: **Other type of secret**
3. Add the following key/value pairs:

   | Key | Value |
   |-----|-------|
   | `db_username` | `texscan_admin` |
   | `db_password` | `MyS3cur3P@ss2026!` |
   | `db_host` | `texscan-db.cluster-abc123.ap-southeast-1.rds.amazonaws.com` |

4. Click **Next**
5. Secret name: `day9-[yourname]-db-credentials`
6. Description: `TexSCAN database credentials for training`
7. Click **Next** → **Next** → **Store**

---

### Step 2: Retrieve the Secret

1. Click on your newly created secret
2. Scroll down and click **Retrieve secret value**
3. Observe the key/value pairs displayed in plaintext
4. Click on the **Sample code** tab — see Python/Java/Node.js code examples

---

### Step 3: View the Python Code Snippet

This is how your application would retrieve the secret programmatically:

```python
import boto3
import json

client = boto3.client('secretsmanager', region_name='ap-southeast-1')
response = client.get_secret_value(SecretId='day9-[yourname]-db-credentials')
secret = json.loads(response['SecretString'])
print(secret['db_username'])  # texscan_admin
```

---

### 💡 Key Insight

> **Never hardcode credentials in code.** Use Secrets Manager to:
> - 🔄 Rotate secrets automatically
> - 🔐 Control access via IAM policies
> - 📋 Audit access via CloudTrail

---

### 🧹 Clean Up

- Go to your secret → **Actions** → **Delete secret**
- Choose: Schedule deletion (7 days) or force delete

---

## Lab 3 — NACL + Security Group Configuration (35 min)

**Objective:** Create your own VPC, then launch an EC2 instance with a basic website to experiment with Security Groups and NACLs to control HTTP (80), HTTPS (443), and SSH (22) access.

---

### Step 1: Create Your Own VPC (5 min)

1. Go to **VPC Console** → click **Create VPC**
2. Select: **VPC and more** (this creates subnets, route tables, and IGW automatically)
3. Configure:

   | Setting | Value |
   |---------|-------|
   | Name tag auto-generation | `day9-[yourname]` |
   | IPv4 CIDR | `10.0.0.0/16` |
   | Number of Availability Zones | 1 |
   | Number of public subnets | 1 |
   | Number of private subnets | 0 |
   | NAT gateways | None |

4. Click **Create VPC**
5. Wait for all resources to show ✅ Created

> 💡 This gives you your own isolated VPC with a public subnet, internet gateway, and route table — no shared NACL conflicts with other students.

---

### Step 2: Launch EC2 with Web Server (10 min)

1. Go to **EC2 Console** → click **Launch instance**
2. Configure the following:

   | Setting | Value |
   |---------|-------|
   | Name | `day9-[yourname]-security-lab` |
   | AMI | Amazon Linux 2023 |
   | Instance type | `t3.micro` |
   | Key pair | Proceed without a key pair (we'll use Instance Connect) |

3. **Network settings** → click **Edit**:
   - VPC: Select **`day9-[yourname]-vpc`** (the VPC you just created)
   - Subnet: Select **`day9-[yourname]-subnet-public1`**
   - Auto-assign public IP: **Enable**
   - Security group: **Create new security group**
   - Security group name: `day9-[yourname]-web-sg`
   - **Remove all inbound rules** (we'll add them manually later)

4. **Advanced details** → paste this into **User data**:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
cat <<'EOF' > /var/www/html/index.html
<html>
<head><title>Day 9 Security Lab</title></head>
<body style="margin:0; background:#000; text-align:center; font-family:Arial;">
<h1 style="color:#fff; padding:20px;">Hello [yourname], let's dance! 🕺</h1>
<img src="https://media.giphy.com/media/Vuw9m5wXviFIQ/giphy.gif" alt="Rick Roll" style="max-width:480px; width:100%;">
</body>
</html>
EOF
```

5. Click **Launch instance**
6. Wait for instance state: **Running** ✅
7. Copy the **Public IPv4 address**

---

### Step 2: Test with NO Inbound Rules (2 min)

1. Open browser → `http://[PUBLIC-IP]`
2. Result: ❌ **Connection timeout** (nothing allowed in!)
3. Try **EC2 Instance Connect** → ❌ Also fails (port 22 blocked)

> 💡 **Key Insight:** Security Groups are **DENY ALL by default** for inbound traffic. You must explicitly allow traffic.

---

### Step 3: Add SSH (Port 22) to Security Group (3 min)

1. Go to **EC2** → **Security Groups** → select `day9-[yourname]-web-sg`
2. **Inbound rules** → **Edit inbound rules** → **Add rule**:

   | Type | Port | Source |
   |------|------|--------|
   | SSH | 22 | `3.0.5.32/29` (EC2 Instance Connect IP for `ap-southeast-1`) |

   > ⚠️ **Important:** EC2 Instance Connect works by connecting FROM AWS's IP range, not your personal IP. Each region has a specific IP range. For Singapore (`ap-southeast-1`), the range is `3.0.5.32/29`.

3. Click **Save rules**
4. Try **EC2 Instance Connect** again → ✅ **Works now!**
5. Run: `curl localhost` → should show your HTML page

---

### Step 4: Add HTTP (Port 80) to Security Group (3 min)

1. **Edit inbound rules** → **Add rule**:

   | Type | Port | Source |
   |------|------|--------|
   | HTTP | 80 | 0.0.0.0/0 (Anywhere) |

2. Click **Save rules**
3. Open browser → `http://[PUBLIC-IP]` → ✅ **Website loads!**

---

### Step 5: Add HTTPS (Port 443) to Security Group (2 min)

1. **Edit inbound rules** → **Add rule**:

   | Type | Port | Source |
   |------|------|--------|
   | HTTPS | 443 | 0.0.0.0/0 (Anywhere) |

2. Click **Save rules**
3. Note: `https://[PUBLIC-IP]` won't work yet (no SSL cert installed) but the port is open

---

### Step 6: Experiment with NACL (15 min)

> **NACLs** = subnet-level firewall. **Stateless** (must allow both inbound AND outbound).

#### 6a: Find Your NACL

1. Go to **VPC Console** → **Network ACLs**
2. Find the NACL associated with your VPC (`day9-[yourname]`) — it's the one auto-created with your VPC
3. Check current rules — default NACL allows **ALL** traffic

#### 6b: Block HTTP at NACL Level

1. **Edit Inbound rules** → **Add rule**:

   | Rule # | Type | Source | Allow/Deny |
   |--------|------|--------|------------|
   | 50 | HTTP (80) | 0.0.0.0/0 | **DENY** |

2. Click **Save**
3. Test browser → `http://[PUBLIC-IP]` → ❌ **Blocked!** (even though SG allows it)

> 💡 **Key Insight:** NACL DENY overrides Security Group ALLOW. NACL rules are evaluated by rule number (lowest first).

#### 6c: Remove the DENY Rule

1. **Edit Inbound rules** → Remove rule #50
2. Click **Save**
3. Test browser → `http://[PUBLIC-IP]` → ✅ **Works again!**

#### 6d: Block Outbound at NACL (Experiment)

1. **Edit Outbound rules** → **Add rule**:

   | Rule # | Type | Destination | Allow/Deny |
   |--------|------|-------------|------------|
   | 50 | HTTP (80) | 0.0.0.0/0 | **DENY** |

2. Click **Save**
3. Test browser → ❌ **Blocked!** (NACL is stateless — needs both directions)
4. Remove rule #50 from outbound → ✅ **Works again**

---

### Security Group vs. NACL Comparison

| | Security Group | NACL |
|--|----------------|------|
| **Level** | Instance (ENI) | Subnet |
| **Stateful?** | Yes (return traffic auto-allowed) | No (must allow both directions) |
| **Default inbound** | Deny all | Allow all |
| **Default outbound** | Allow all | Allow all |
| **Rules** | Allow only | Allow AND Deny |
| **Evaluation** | All rules evaluated | Rules evaluated by number (lowest first) |
| **Pinoy analogy** | Guard sa door ng unit mo 🚪 | Guard sa gate ng subdivision 🏘️ |

---

### Step 7: Clean Up (5 min)

1. ☐ **Terminate** EC2 instance (`day9-[yourname]-security-lab`)
2. ☐ **Delete** security group (`day9-[yourname]-web-sg`) — wait for instance to terminate first
3. ☐ **Delete VPC** → Go to **VPC Console** → select `day9-[yourname]-vpc` → **Actions** → **Delete VPC** (this deletes subnets, route tables, IGW automatically)
4. ☐ **Delete** secret from Secrets Manager (`day9-[yourname]-db-credentials`)

---

## Lab 4 — AWS Policy Generator: Create IAM Policies (30 min)

**Objective:** Use the AWS Policy Generator tool to create 3 IAM policies — from basic (beginner) to complex (advanced) — and attach them in the IAM console.

**Tool:** [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html)

---

### Introduction: What is the AWS Policy Generator?

> The AWS Policy Generator is a free web tool that helps you build IAM, S3 Bucket, SNS, and SQS policies using a visual interface — no need to memorize JSON syntax!

> 💡 **Use case:** Your team needs to grant a new developer S3 read access. Instead of writing JSON from scratch (and risking syntax errors), you use the Policy Generator to select the service, actions, and resource ARN — it outputs valid JSON ready to paste into IAM.

---

### Policy 1: Beginner — S3 Read-Only Access (5 min)

**Scenario:** A developer needs to view and download files from a specific S3 bucket but cannot upload, delete, or modify anything.

#### Step 1: Open the Policy Generator

1. Go to: https://awspolicygen.s3.amazonaws.com/policygen.html
2. Select Policy Type: **IAM Policy**

#### Step 2: Configure the Statement

| Field | Value |
|-------|-------|
| Effect | Allow |
| AWS Service | Amazon S3 |
| Actions | `GetObject`, `ListBucket` |
| ARN | `arn:aws:s3:::dost-ptri-day9-[yourname]/*` |

> ⚠️ For `ListBucket`, the ARN should be the bucket itself: `arn:aws:s3:::dost-ptri-day9-[yourname]`
> For `GetObject`, use the objects path: `arn:aws:s3:::dost-ptri-day9-[yourname]/*`

3. Click **Add Statement** (add one for each ARN pattern)
4. Click **Generate Policy**

#### Step 3: Expected Output

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::dost-ptri-day9-[yourname]/*"
    },
    {
      "Sid": "Stmt2",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::dost-ptri-day9-[yourname]"
    }
  ]
}
```

#### Step 4: Review Your Policy

1. Review the generated JSON — understand each field: `Effect`, `Action`, `Resource`
2. *(Optional)* Copy the policy to a text editor or notepad for your reference

> 💡 **Lesson:** Least privilege — give only `Get` and `List`, not full S3 access. Specific bucket ARN, not `*`.

---

### Policy 2: Intermediate — EC2 + CloudWatch with Conditions (10 min)

**Scenario:** A DevOps engineer needs to start/stop EC2 instances and view CloudWatch metrics, but ONLY in the `ap-southeast-1` region and only for instances tagged with `Environment=Training`.

#### Step 1: Add Statement 1 — EC2 Actions

| Field | Value |
|-------|-------|
| Effect | Allow |
| AWS Service | Amazon EC2 |
| Actions | `StartInstances`, `StopInstances`, `DescribeInstances` |
| ARN | `arn:aws:ec2:ap-southeast-1:*:instance/*` |

Click **Add Conditions (Optional)**:

| Condition | Key | Value |
|-----------|-----|-------|
| StringEquals | `aws:RequestedRegion` | `ap-southeast-1` |

Click **Add Condition** → then **Add Statement**

#### Step 2: Add Statement 2 — CloudWatch Read

| Field | Value |
|-------|-------|
| Effect | Allow |
| AWS Service | Amazon CloudWatch |
| Actions | `GetMetricData`, `ListMetrics`, `DescribeAlarms` |
| ARN | `*` |

Click **Add Statement**

#### Step 3: Add Statement 3 — Restrict by Tag

> ⚠️ The Policy Generator doesn't support tag-based conditions directly in the UI. We'll add this manually after generating.

Click **Generate Policy**, then manually add this condition to the EC2 statement:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "ap-southeast-1",
    "ec2:ResourceTag/Environment": "Training"
  }
}
```

#### Step 4: Final Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "arn:aws:ec2:ap-southeast-1:*:instance/*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-southeast-1",
          "ec2:ResourceTag/Environment": "Training"
        }
      }
    },
    {
      "Sid": "Stmt2",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricData",
        "cloudwatch:ListMetrics",
        "cloudwatch:DescribeAlarms"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Step 5: Review Your Policy

1. Review the final JSON — note how `Condition` restricts access beyond just Action + Resource
2. *(Optional)* Copy the policy for your reference

> 💡 **Lesson:** Conditions add powerful guardrails — region lock + tag-based access = defense in depth for IAM. The Policy Generator gives you a starting point, but real-world policies often need manual refinement.

---

### Policy 3: Advanced — Multi-Service Deny + Allow with MFA Enforcement (15 min)

**Scenario:** A security admin needs a policy that:
- Allows full access to DynamoDB and Lambda
- Denies deletion of CloudTrail logs (prevent evidence tampering)
- Requires MFA for any IAM actions

#### Step 1: Add Statement 1 — Allow DynamoDB Full Access

| Field | Value |
|-------|-------|
| Effect | Allow |
| AWS Service | Amazon DynamoDB |
| Actions | `All Actions (*)` |
| ARN | `arn:aws:dynamodb:ap-southeast-1:*:table/day9-*` |

Click **Add Statement**

#### Step 2: Add Statement 2 — Allow Lambda Full Access

| Field | Value |
|-------|-------|
| Effect | Allow |
| AWS Service | AWS Lambda |
| Actions | `All Actions (*)` |
| ARN | `arn:aws:lambda:ap-southeast-1:*:function:day9-*` |

Click **Add Statement**

#### Step 3: Add Statement 3 — Deny CloudTrail Deletion

| Field | Value |
|-------|-------|
| Effect | **Deny** |
| AWS Service | AWS CloudTrail |
| Actions | `DeleteTrail`, `StopLogging` |
| ARN | `*` |

Click **Add Statement**

#### Step 4: Generate and Add MFA Condition

Click **Generate Policy**, then manually add an MFA enforcement statement:

```json
{
  "Sid": "DenyIAMWithoutMFA",
  "Effect": "Deny",
  "Action": "iam:*",
  "Resource": "*",
  "Condition": {
    "BoolIfExists": {
      "aws:MultiFactorAuthPresent": "false"
    }
  }
}
```

#### Step 5: Final Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDynamoDB",
      "Effect": "Allow",
      "Action": "dynamodb:*",
      "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/day9-*"
    },
    {
      "Sid": "AllowLambda",
      "Effect": "Allow",
      "Action": "lambda:*",
      "Resource": "arn:aws:lambda:ap-southeast-1:*:function:day9-*"
    },
    {
      "Sid": "DenyCloudTrailDeletion",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:DeleteTrail",
        "cloudtrail:StopLogging"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyIAMWithoutMFA",
      "Effect": "Deny",
      "Action": "iam:*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

#### Step 6: Review Your Policy

1. Review the final JSON — identify which statements are Allow vs. Deny
2. Note how `BoolIfExists` condition enforces MFA
3. *(Optional)* Copy the policy for your reference

> 💡 **Lesson:** Real-world policies combine Allow + Deny. Explicit Deny ALWAYS wins over Allow. MFA conditions protect sensitive actions. Scoped ARNs (`day9-*`) prevent accidental access to production resources.

---

### Policy Comparison Summary

| | Policy 1 (Beginner) | Policy 2 (Intermediate) | Policy 3 (Advanced) |
|--|---------------------|------------------------|---------------------|
| **Services** | S3 only | EC2 + CloudWatch | DynamoDB + Lambda + CloudTrail + IAM |
| **Effect** | Allow only | Allow only | Allow + Deny |
| **Conditions** | None | Region + Tag | MFA enforcement |
| **ARN Scope** | Specific bucket | Regional + wildcard | Service-scoped prefix |
| **Complexity** | Single service, 2 actions | Multi-service, conditions | Multi-service, deny rules, MFA |
| **Real-world use** | Junior dev / read-only user | DevOps engineer | Security admin / compliance |

---

### 🧹 Clean Up

> No cleanup needed — policies were generated in the browser only, not applied to the AWS account.

---

## Key Concepts Reinforced

| Concept | What You Learned |
|---------|-----------------|
| Defense in Depth | Multiple layers: WAF → NACL → Security Group → OS firewall |
| Least Privilege | Start with no access, add only what's needed |
| Secrets Management | Never hardcode credentials; use Secrets Manager |
| Stateful vs. Stateless | SG remembers connections; NACL does not |
| Rule Evaluation | NACL = by number (lowest first); SG = all rules together |
| Shared Responsibility | AWS secures the cloud; you secure what's IN the cloud |
| IAM Policy Structure | Effect + Action + Resource + Condition = complete policy |
| Explicit Deny | Deny ALWAYS overrides Allow in IAM evaluation |
| MFA Enforcement | Conditions can require MFA for sensitive operations |
| Scoped ARNs | Use specific ARNs and prefixes instead of `*` |

---

## ✅ Lab Complete Checklist

- [x] Explored WAF, GuardDuty, Config, and Security Hub consoles
- [x] Created and retrieved a secret in Secrets Manager
- [x] Launched EC2 with a web server
- [x] Tested Security Group deny-all default behavior
- [x] Added SSH, HTTP, and HTTPS rules to Security Group
- [x] Blocked traffic using NACL DENY rules
- [x] Understood stateful (SG) vs. stateless (NACL) behavior
- [x] Used AWS Policy Generator to create 3 IAM policies
- [x] Understood Allow vs. Deny and IAM policy evaluation logic
- [x] Applied conditions (region, tag, MFA) to policies
- [x] Cleaned up all resources

---

**Congratulations!** 🎉 You've completed the Day 9 Security Fundamentals labs.

You now understand how AWS security services work together to protect your infrastructure at multiple layers — from edge (WAF) to subnet (NACL) to instance (Security Group) to application (Secrets Manager).

---

*DOST PTRI — AWS Fundamentals Training Program | Day 9 | Security Fundamentals*
