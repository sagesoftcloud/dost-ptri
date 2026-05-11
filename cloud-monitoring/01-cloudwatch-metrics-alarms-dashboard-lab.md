# Day 7 Lab — EC2 CloudWatch Metrics & CPU Alarm

## DOST PTRI — AWS Fundamentals Training Program

> ⚠️ **For training purposes only.**

This lab walks you through launching an EC2 instance, generating CPU load, observing CloudWatch metrics, and creating an alarm that sends an email notification.

| Step | Activity | Time |
|------|----------|:----:|
| 1 | Launch an EC2 instance | 10 min |
| 2 | Generate CPU load & observe metrics | 15 min |
| 3 | Create SNS topic & CloudWatch Alarm | 15 min |
| 4 | Trigger the alarm & receive email | 10 min |
| 5 | Explore CloudTrail | 10 min |
| 6 | Explore Cost Explorer | 10 min |
| 7 | Explore Trusted Advisor | 5 min |
| 8 | Clean up | 5 min |

**Total: ~80 minutes**

---

## Prerequisites

- AWS Console access (your training account)
- A valid email address for alarm notifications

---

## Step 1: Launch an EC2 Instance (10 min)

1. Go to **EC2 Console** → **Launch instance**
2. Configure:
   - **Name:** `day7-[yourname]-monitoring`
   - **AMI:** Amazon Linux 2023 (default)
   - **Instance type:** `t3.micro`
   - **Key pair:** Proceed without a key pair (we'll use EC2 Instance Connect)
   - **Network settings:** Allow SSH from anywhere (for Instance Connect)
   - **Everything else:** Leave defaults
3. Click **Launch instance**
4. Wait until **Instance state** = `Running`

---

## Step 2: Install CloudWatch Agent & Generate Load (20 min)

### 2.1 Attach IAM Role (if not already attached)

Your EC2 instance needs permissions for SSM and CloudWatch:

1. Go to **EC2 Console** → select your instance
2. Click **Actions** → **Security** → **Modify IAM role**
3. Select a role with these policies (or create one):
   - `CloudWatchAgentServerPolicy`
   - `AmazonSSMManagedInstanceCore`
4. Click **Update IAM role**

> 💡 If no role exists, create one in IAM → Roles → Create role → EC2 → attach both policies above.

### 2.2 Install CloudWatch Agent via EC2 Console

No SSH needed — install directly from the EC2 Monitoring tab:

1. Go to **EC2 Console** → select your instance
2. Click the **Monitoring** tab
3. Click **Manage CloudWatch Agent** (or **Install CloudWatch Agent**)
4. Select your instance — confirm it shows **SSM Agent: Installed**
5. Click **Install**
6. Wait for status to change from **Installing** → **Installed**

> 💡 This may take 2–3 minutes. Click refresh if it seems stuck.

### 2.3 Configure the Agent

After installation, you'll go through 3 steps:

#### Step 1: Agent Status
- Verify your instance shows **CloudWatch Agent: Installed** ✅
- Click **Next**

#### Step 2: Edit Configuration

AWS auto-detects your instance and recommends metrics. Review the settings:

| Setting | Value | Description |
|---------|-------|-------------|
| **Collection interval** | `60` seconds | How often metrics are collected |
| **Metric namespace** | `CWAgent` | Where metrics appear in CloudWatch |
| **Region** | `ap-southeast-1` | Your AWS region |

**Global dimensions** (select these):
- ✅ `InstanceId`
- ✅ `InstanceType`

> These dimensions let you filter metrics per instance later.

**Aggregation dimensions:**
- Keep default: `InstanceId`

**Selected metrics:**
- AWS auto-selects ~23 essential metrics (CPU, memory, disk, network)
- Review the list — you should see `mem_used_percent`, `disk_used_percent`
- You can add/remove metrics or keep the recommendations

> 💡 **Compute Optimizer enabled** — this feeds data to AWS for right-sizing recommendations later in Cost Explorer.

Click **Next**

#### Step 3: Review and Deploy
- Review your configuration summary
- Click **Deploy**
- Wait for the configuration to be applied to your instance

> After deployment, metrics will appear under the **CWAgent** namespace in CloudWatch within 5 minutes.

### 2.4 Connect to Instance & Generate CPU + Memory Stress

1. Go to **EC2 Console** → select your instance → click **Connect**
2. Choose **EC2 Instance Connect** tab → click **Connect**
3. Run:

```bash
sudo yum install -y stress-ng
stress-ng --cpu 1 --vm 1 --vm-bytes 512M --timeout 300s
```

> `--vm 1 --vm-bytes 512M` allocates 512 MB of memory stress. This will show up in the CWAgent memory metric.

Leave this running — don't close the terminal.

### 2.5 View CPU Metrics in CloudWatch

1. Open a **new browser tab** → go to **CloudWatch** → **Metrics** → **All metrics**
2. Click **EC2** → **Per-Instance Metrics**
3. Find your instance (`day7-[yourname]-monitoring`) and select **CPUUtilization**
4. Set:
   - **Time range:** 1h
   - **Period:** 1 minute
   - **Statistic:** Average
5. Watch the graph — you should see CPU climbing to ~100%

> 💡 It takes 1–2 minutes for new data points to appear. Be patient.

### 2.6 View Memory Metrics (from CloudWatch Agent)

1. Go back to **All metrics**
2. Look for the **CWAgent** namespace → click it
3. Click **InstanceId** (or **ImageId, InstanceId, InstanceType**)
4. Find and select **mem_used_percent**
5. You should see memory usage climbing due to the stress test

**What to observe:**

| Metric | Where | What You'll See |
|--------|-------|-----------------|
| CPUUtilization | EC2 namespace | ~100% (from `--cpu 1`) |
| mem_used_percent | CWAgent namespace | Spike from stress `--vm-bytes 512M` |

> ⚠️ **This is why the CloudWatch Agent matters!** Without it, you'd see CPU at 100% but have NO visibility into memory. If your app crashes from out-of-memory, you'd never know from default metrics alone.

> 💡 CWAgent metrics may take 5 minutes to appear after installation. If not visible yet, continue with the lab and check back later.

---

## Step 3: Create SNS Topic & CloudWatch Alarm (15 min)

### 3.1 Create an SNS Topic

1. Go to **SNS Console** → **Topics** → **Create topic**
2. Configure:
   - **Type:** Standard
   - **Name:** `day7-[yourname]-cpu-alarm`
3. Click **Create topic**

### 3.2 Subscribe Your Email

1. Click **Create subscription**
2. Configure:
   - **Protocol:** Email
   - **Endpoint:** your email address
3. Click **Create subscription**
4. **Check your email** → click the **Confirm subscription** link

> ⚠️ You MUST confirm or you won't receive the alarm email.

### 3.3 Create the Alarm

1. Go to **CloudWatch** → **Alarms** → **Create alarm**
2. Click **Select metric** → **EC2** → **Per-Instance Metrics**
3. Find your instance → select **CPUUtilization** → click **Select metric**
4. Configure:
   - **Statistic:** Average
   - **Period:** 1 minute
   - **Threshold:** Greater than `80`
5. Click **Next**
6. **Notification:**
   - Alarm state trigger: **In alarm**
   - SNS topic: select `day7-[yourname]-cpu-alarm`
7. Click **Next**
8. **Name:** `day7-[yourname]-cpu-high`
9. Click **Next** → **Create alarm**

---

## Step 4: Trigger the Alarm & Receive Email (10 min)

### 4.1 Check Alarm State

1. Go to **CloudWatch** → **Alarms**
2. Your alarm should show **INSUFFICIENT_DATA** initially
3. If your stress test is still running, wait 1–2 minutes — it will transition to **ALARM** 🔴

### 4.2 If Stress Already Ended — Run It Again

Go back to your EC2 Instance Connect terminal:

```bash
stress-ng --cpu 1 --timeout 300s
```

### 4.3 Receive the Email

1. Once the alarm state changes to **ALARM**, check your email
2. You should receive a notification from AWS with:
   - Alarm name
   - Current value (near 100%)
   - Threshold (80%)
   - Timestamp

### 4.4 Stop the Stress & Watch Recovery

1. In the terminal, press `Ctrl+C` to stop stress-ng
2. Wait 2–3 minutes
3. The alarm should transition back to **OK** 🟢

### Expected Result

```
INSUFFICIENT_DATA → ALARM (CPU > 80%) → email received → OK (CPU drops)
```

---

## Step 5: Explore CloudTrail (10 min)

### 5.1 Open CloudTrail

1. Go to **AWS Console** → search for **CloudTrail**
2. Click **Event history** in the left sidebar

### 5.2 Find Your Own Actions

1. Click the **Lookup attributes** dropdown → select **Event name**
2. Type `RunInstances` → press Enter
3. You should see the EC2 launch you did in Step 1
4. Click the event row to expand it

### 5.3 Inspect the Event Details

1. Click **View event** to see the full JSON
2. Look for these fields:
   - `userIdentity.userName` — your IAM user
   - `eventTime` — when you launched the instance
   - `sourceIPAddress` — your IP
   - `requestParameters.instanceType` — `t3.micro`

### 5.4 Find Your Alarm Creation

1. Clear the filter → select **Event name** again
2. Type `PutMetricAlarm` → press Enter
3. You should see the alarm you created in Step 3

> 💡 **Key insight:** Every action in AWS is recorded. If something changes unexpectedly, CloudTrail tells you exactly who did it and when.

---

## Step 6: Explore Cost Explorer (10 min)

### 6.1 Open Cost Explorer

1. Go to **AWS Console** → search for **Cost Explorer**
2. If first time, click **Enable Cost Explorer** (may take up to 24 hours for full data)

### 6.2 View Cost Breakdown

1. On the main dashboard, observe the **monthly spend graph**
2. Change **Group by** to **Service**
3. Identify which AWS service costs the most in your account

### 6.3 Filter by Service

1. Click **Filters** on the right side
2. Under **Service**, select **Amazon Elastic Compute Cloud**
3. Click **Apply**
4. Now you see only EC2 costs — daily breakdown

### 6.4 Check Instance Right-Sizing Recommendations

1. In the left sidebar, click **Recommendations** → **Right Sizing**
2. If available, review any recommendations AWS provides
3. Notice it shows: current instance type, recommended type, and estimated savings

> 💡 **Key insight:** Cost Explorer + right-sizing recommendations use the same CloudWatch metrics we discussed. AWS does the computation for you — but now you know HOW it's calculated.

---

## Step 7: Explore Trusted Advisor (5 min)

### 7.1 Open Trusted Advisor

1. Go to **AWS Console** → search for **Trusted Advisor**
2. You'll see the dashboard with 5 categories

### 7.2 Review the Categories

1. Check each category for findings:
   - **Cost Optimization** — any idle or underutilized resources?
   - **Security** — any open security groups or missing MFA?
   - **Fault Tolerance** — any single-AZ resources without backups?
   - **Performance** — any over-provisioned instances?
   - **Service Limits** — approaching any AWS quotas?

### 7.3 Look at Security Checks

1. Click **Security** category
2. Look for:
   - "Security Groups — Unrestricted Access" (port 0.0.0.0/0)
   - "MFA on Root Account"
3. Note any ⚠️ warnings or 🔴 action required items

> 💡 **Key insight:** Trusted Advisor is like a free consultant that scans your account 24/7. Check it regularly — especially the Security and Cost Optimization tabs.

---

## Step 8: Clean Up (5 min)

1. **Terminate EC2:**
   - EC2 Console → select your instance → Instance state → **Terminate instance**

2. **Delete alarm:**
   - CloudWatch → Alarms → select → Actions → **Delete**

3. **Delete SNS topic & subscription:**
   - SNS → Subscriptions → select → **Delete**
   - SNS → Topics → select → **Delete**

---

## Key Concepts Reinforced

| Concept | What You Did |
|---------|-------------|
| **CloudWatch Agent** | Installed with sudo to collect memory/disk metrics |
| **CloudWatch Metrics** | Observed real-time CPU data from your EC2 |
| **Metric Period** | Set 1-minute granularity to see changes quickly |
| **SNS Topic** | Created a notification channel for alerts |
| **CloudWatch Alarm** | Set a threshold (CPU > 80%) with automatic email |
| **Alarm States** | Saw INSUFFICIENT_DATA → ALARM → OK lifecycle |
| **CloudTrail** | Found your own API calls (RunInstances, PutMetricAlarm) |
| **Cost Explorer** | Viewed spending breakdown by service and right-sizing recommendations |
| **Trusted Advisor** | Reviewed automated best-practice checks across 5 categories |
| **Right-Sizing Insight** | If CPU stays at 5% normally, this instance is over-provisioned |

---

## 🎉 Lab Complete!

You've successfully:
- ✅ Launched an EC2 instance
- ✅ Updated the system and installed CloudWatch Agent (with sudo)
- ✅ Generated CPU load and observed it in CloudWatch
- ✅ Checked CWAgent memory metrics
- ✅ Created an SNS topic with email subscription
- ✅ Created a CloudWatch Alarm that triggers on high CPU
- ✅ Received an alarm email notification
- ✅ Watched the alarm recover when load stopped
- ✅ Traced your own actions in CloudTrail
- ✅ Explored Cost Explorer and right-sizing recommendations
- ✅ Reviewed Trusted Advisor security and cost checks
