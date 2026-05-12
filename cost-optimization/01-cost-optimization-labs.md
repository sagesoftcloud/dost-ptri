# Day 8 Labs — Cost Optimization Hands-On

## DOST PTRI — AWS Fundamentals Training Program

> ⚠️ **For training purposes only.**

| Lab | Activity | Time |
|-----|----------|:----:|
| 1 | S3 Lifecycle Policy | 15 min |
| 2 | Automated EC2 On/Off (Lambda + EventBridge) | 40 min |
| 3 | AWS Pricing Calculator — Compute 10 Services | 20 min |
| 4 | Cost Allocation Tagging on EC2 | 5 min |

**Total: ~80 minutes**

---

# Lab 1 — S3 Lifecycle Policy (15 min)

> 🎯 **Goal:** Create a lifecycle rule that automatically transitions objects to cheaper storage classes and deletes old files.

## Step 1: Create an S3 Bucket

1. Go to **S3 Console** → **Create bucket**
2. **Bucket name:** `day8-[yourname]-lifecycle-demo`
3. **Region:** `ap-southeast-1`
4. Leave everything else default → **Create bucket**

## Step 2: Upload Test Files

1. Click into your bucket → **Upload**
2. Upload 2-3 small files (any text/image files)
3. Click **Upload**

## Step 3: Create Lifecycle Rule

1. Go to your bucket → **Management** tab → **Create lifecycle rule**
2. Configure:
   - **Rule name:** `cost-optimization-rule`
   - **Apply to all objects in the bucket** ✅
3. **Lifecycle rule actions** — check all:
   - ✅ Transition current versions of objects between storage classes
   - ✅ Expire current versions of objects
4. **Transitions:**

| Transition | Days after creation |
|------------|:-------------------:|
| Standard → Standard-IA | 30 days |
| Standard-IA → Glacier Instant Retrieval | 60 days |
| Glacier Instant → Glacier Deep Archive | 180 days |

5. **Expiration:** Delete objects after `365` days
6. Click **Create rule**

## Step 4: Verify

1. Go to **Management** tab — see your rule listed
2. Check the **Timeline summary** — visual representation of transitions

> 💡 **Cost impact:** This single rule can reduce storage costs by 60-80% for aging data. Standard = $0.023/GB → Deep Archive = $0.00099/GB (23x cheaper!)

## Clean Up

- Delete the bucket: Select bucket → **Empty** → **Delete bucket**

---

# Lab 2 — Automated EC2 On/Off with Lambda + EventBridge (40 min)

> 🎯 **Goal:** Build a cost-saving automation that stops EC2 at 6 PM and starts at 8 AM (weekdays). Real FinOps!

**Savings math:** A `t3.micro` running 24/7 = $7.49/month. Running 8AM–6PM weekdays only = $2.29/month. **69% savings!**

---

## Step 1: Launch a Test EC2 Instance (5 min)

1. Go to **EC2 Console** → **Launch instance**
2. Configure:
   - **Name:** `day8-[yourname]-cost-test`
   - **AMI:** Amazon Linux 2023
   - **Instance type:** `t3.micro`
   - **Key pair:** Proceed without
3. Click **Launch instance**
4. **Copy your Instance ID** (e.g., `i-0abc123def456789`) — you'll paste this into the Lambda code

> ⚠️ Wait until Instance state = **Running** before proceeding.

---

## Step 2: Create IAM Role for Lambda (5 min)

1. Go to **IAM** → **Roles** → **Create role**
2. **Trusted entity:** AWS service → **Lambda**
3. Attach policies:
   - `AmazonEC2FullAccess`
   - `AWSLambdaBasicExecutionRole`
4. **Role name:** `day8-lambda-ec2-scheduler`
5. Click **Create role**

> ⚠️ Production: use a custom policy with only `ec2:StartInstances`, `ec2:StopInstances`, `ec2:DescribeInstances`.

---

## Step 3: Create STOP Lambda Function (8 min)

1. Go to **Lambda** → **Create function**
2. Configure:
   - **Name:** `day8-[yourname]-ec2-stop`
   - **Runtime:** Python 3.12
   - **Execution role:** Existing → `day8-lambda-ec2-scheduler`
3. Replace code with:

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name='ap-southeast-1')
    
    # ⚠️ CHANGE THIS to your Instance ID from Step 1
    instances = ['i-XXXXXXXXXXXXXXXXX']  # <-- Replace with your Instance ID
    
    ec2.stop_instances(InstanceIds=instances)
    print(f'Stopped: {instances}')
    return {'stopped': instances}
```

4. **Replace `i-XXXXXXXXXXXXXXXXX`** with your actual Instance ID from Step 1 (e.g., `i-0abc123def456789`)
5. Click **Deploy**
6. **Configuration** → **General** → Edit → Timeout: `30 sec` → Save

---

## Step 4: Create START Lambda Function (5 min)

1. **Create function** → same settings but:
   - **Name:** `day8-[yourname]-ec2-start`
2. Replace code with:

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name='ap-southeast-1')
    
    # ⚠️ CHANGE THIS to your Instance ID from Step 1
    instances = ['i-XXXXXXXXXXXXXXXXX']  # <-- Replace with your Instance ID
    
    ec2.start_instances(InstanceIds=instances)
    print(f'Started: {instances}')
    return {'started': instances}
```

3. **Replace `i-XXXXXXXXXXXXXXXXX`** with the same Instance ID you used in the STOP function
4. **Deploy** → Set timeout to 30 sec

---

## Step 5: Create EventBridge Schedules (7 min)

> 💡 **Instructor will announce the schedule times** based on today's training. Convert PHT to UTC (subtract 8 hours).
>
> Example: If instructor says "stop at 2:30 PM PHT" → 2:30 PM - 8 = 6:30 UTC → cron: `30 6 ? * * *`

### STOP Schedule:

1. Go to **EventBridge** → **Schedules** → **Create schedule**
2. **Name:** `day8-[yourname]-stop-ec2`
3. **Recurring** → **Cron-based schedule**
4. **Cron expression:** `<MM> <HH> ? * * *`

   > Replace `<HH>` and `<MM>` with the UTC time announced by instructor.
   > Example: Stop at 2:35 PM PHT = `35 6 ? * * *`

5. **Next** → Target: Lambda → `day8-[yourname]-ec2-stop`
6. **Create schedule**

### START Schedule:

1. **Create schedule**
2. **Name:** `day8-[yourname]-start-ec2`
3. **Cron expression:** `<MM> <HH> ? * * *`

   > Use the START time announced by instructor (5 minutes after STOP).
   > Example: Start at 2:40 PM PHT = `40 6 ? * * *`

4. Target: Lambda → `day8-[yourname]-ec2-start`
5. **Create schedule**

> ⚠️ **Cron format:** `minute hour day-of-month month day-of-week year`
>
> **PHT to UTC cheat sheet:**
> | PHT | UTC |
> |-----|-----|
> | 10:00 AM | 2:00 |
> | 11:00 AM | 3:00 |
> | 1:00 PM | 5:00 |
> | 2:00 PM | 6:00 |
> | 3:00 PM | 7:00 |
> | 4:00 PM | 8:00 |

---

## Step 6: Test (5 min)

1. Go to Lambda → `day8-[yourname]-ec2-stop` → **Test** → Event: `{}` → **Test**
2. Check EC2 Console — instance should be **Stopping** ✅
3. Go to Lambda → `day8-[yourname]-ec2-start` → **Test**
4. Check EC2 Console — instance should be **Running** ✅

---

## Step 7: Clean Up (5 min)

1. **EventBridge** → Schedules → delete both
2. **Lambda** → delete both functions
3. **IAM** → Roles → delete `day8-lambda-ec2-scheduler`
4. **EC2** → Terminate instance

---

# Lab 3 — AWS Pricing Calculator (20 min)

> 🎯 **Goal:** Estimate monthly cost for a multi-tier architecture using the AWS Pricing Calculator with 10 services.

## Instructions

1. Open **[AWS Pricing Calculator](https://calculator.aws/)**
2. Click **Create estimate**
3. Region: **Asia Pacific (Singapore) — ap-southeast-1**

## Your Task: Estimate These 10 Services

Create a cost estimate for the following TexSCAN-like architecture. Use the values provided:

| # | Service | Configuration |
|---|---------|---------------|
| 1 | **Amazon EC2** | 2× t3.medium, On-Demand, Linux, 24/7 (web servers) |
| 2 | **Amazon RDS** | 1× db.t3.medium, MySQL, Multi-AZ, 100 GB gp3 storage |
| 3 | **Amazon S3** | 500 GB Standard storage, 1 million GET requests/month, 100K PUT requests/month |
| 4 | **Amazon CloudFront** | 100 GB data transfer out/month, 1 million HTTPS requests |
| 5 | **AWS Lambda** | 5 million requests/month, 256 MB memory, 200ms avg duration |
| 6 | **Amazon DynamoDB** | On-Demand, 10 million reads/month, 2 million writes/month, 50 GB storage |
| 7 | **Amazon API Gateway** | REST API, 5 million requests/month |
| 8 | **Elastic Load Balancer** | 1× Application Load Balancer, 100 GB processed/month |
| 9 | **Amazon SQS** | Standard queue, 10 million requests/month |
| 10 | **Amazon Cognito** | 10,000 monthly active users (MAU) |

## Steps for Each Service

1. Click **Add service**
2. Search for the service name
3. Select region: `Asia Pacific (Singapore)`
4. Fill in the configuration from the table above
5. Click **Add to my estimate**
6. Repeat for all 10 services

## After Adding All 10 Services

1. Review the **total monthly estimate**
2. Identify the **top 3 most expensive** services
3. Click **Export** → download as CSV or PDF
4. **Discussion questions:**
   - Which service surprised you with its cost?
   - How would you reduce the total? (Hint: Reserved Instances, Serverless, lifecycle policies)
   - What happens if you switch EC2 to Savings Plan (1yr)?
   - What if you change RDS from Multi-AZ to Single-AZ?

> 💡 **Try it:** Go back and change EC2 to 1-year Reserved Instance (No Upfront). How much did the estimate drop?

---

# Lab 4 — Cost Allocation Tagging on EC2 (5 min)

> 🎯 **Goal:** Apply proper cost allocation tags to the EC2 instance you created in Lab 2. This is how organizations track spend per project, environment, and team.

## Step 1: Tag Your EC2 Instance

1. Go to **EC2 Console** → select your instance (`day8-[yourname]-cost-test`)
2. Click **Tags** tab → **Manage tags**
3. Add the following tags:

| Key | Value |
|-----|-------|
| `Name` | `day8-[yourname]-cost-test` |
| `Environment` | `dev` |
| `Project` | `DOST-PTRI` |
| `Team` | `[yourname]` |
| `CostCenter` | `training-2026` |
| `AutoSchedule` | `true` |

4. Click **Save**

## Step 2: Verify in Console

1. Go to **EC2 Console** → **Instances**
2. Click the **gear icon** (column settings) at the top right of the table
3. Enable columns: `Environment`, `Project`, `Team`
4. Now you can see tags directly in the instance list — easy to filter!

## Step 3: Filter by Tag

1. In the EC2 instance list, click the **search/filter bar**
2. Select **Tag: Environment** → type `dev`
3. Only instances tagged `dev` should appear

> 💡 **Why this matters:**
> - In Cost Explorer, you can filter costs by these tags
> - "How much is the `dev` environment costing us?" → instant answer
> - Without tags = unattributed costs = no accountability
> - Best practice: tag **everything** at creation time

---

## Key Concepts Reinforced

| Concept | What You Did |
|---------|-------------|
| **S3 Lifecycle** | Automated storage class transitions for cost savings |
| **Lambda** | Serverless compute for EC2 automation (zero cost when idle) |
| **EventBridge Scheduler** | Cron-based scheduling for recurring tasks |
| **Instance ID targeting** | Hardcoded instance ID in Lambda for direct control |
| **Cost-saving automation** | 69% savings by scheduling EC2 on/off |
| **Pricing Calculator** | Estimated real-world architecture costs |
| **Service cost comparison** | Identified expensive vs cheap services |
| **Reserved vs On-Demand** | Saw the savings from commitment |
| **Cost Allocation Tags** | Tagged EC2 with Environment=dev for cost tracking |
| **Tag-based filtering** | Filtered resources by environment in console |

---

## 🎉 Labs Complete!

You've successfully:
- ✅ Created S3 lifecycle rules (Standard → IA → Glacier → Deep Archive → Delete)
- ✅ Built Lambda functions to start/stop EC2 instances
- ✅ Created EventBridge cron schedules (instructor-timed STOP/START)
- ✅ Tested both functions and verified EC2 state changes
- ✅ Estimated costs for 10 AWS services using Pricing Calculator
- ✅ Identified top cost drivers in a multi-tier architecture
- ✅ Compared On-Demand vs Reserved pricing impact
- ✅ Tagged EC2 with cost allocation tags (Environment=dev, Project=DOST-PTRI)
- ✅ Filtered instances by tag in the console
