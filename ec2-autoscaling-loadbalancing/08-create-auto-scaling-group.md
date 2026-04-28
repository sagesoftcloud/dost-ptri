# 08 - Create the Auto Scaling Group

## Goal
Create an Auto Scaling Group (ASG) that automatically launches, maintains, and scales EC2 web server instances using the launch template (with the custom AMI). The ASG registers instances with the ALB target group.

## Prerequisites
- Completed [07 - Set Up the Application Load Balancer](./07-setup-load-balancer.md)
- `dost-ptri-day3-[yourname]-web-lt` (Launch Template with custom AMI) exists
- `dost-ptri-day3-[yourname]-tg` (Target Group) exists
- `dost-ptri-day3-[yourname]-alb` (Load Balancer) is Active

---

## Step 1: Create the Auto Scaling Group

1. Go to **EC2 Console** → **Auto Scaling Groups** → **Create Auto Scaling group**

### Step 1 of 7: Choose launch template
- **Auto Scaling group name:** `dost-ptri-day3-[yourname]-asg`
- **Launch template:** Select `dost-ptri-day3-[yourname]-web-lt`
- Click **Next**

### Step 2 of 7: Choose instance launch options
- **VPC:** Select `dost-ptri-day3-[yourname]-vpc`
- **Availability Zones and subnets:** Select both:
  - `dost-ptri-day3-[yourname]-public-subnet`
  - `dost-ptri-day3-[yourname]-public-subnet-2`
- Click **Next**

### Step 3 of 7: Configure advanced options
- **Load balancing:** Select **Attach to an existing load balancer**
- **Attach to an existing load balancer:** Select **Choose from your load balancer target groups**
- **Existing load balancer target groups:** Select `dost-ptri-day3-[yourname]-tg`
- **Health checks:**
  - Check ✅ **Turn on Elastic Load Balancing health checks**
  - Health check grace period: `120` seconds
- Click **Next**

### Step 4 of 7: Configure group size and scaling
- **Group size:**
  - Desired capacity: `2`
  - Minimum capacity: `2`
  - Maximum capacity: `4`

- **Scaling policies:** Select **Target tracking scaling policy**
  - **Scaling policy name:** `dost-ptri-day3-[yourname]-cpu-scaling`
  - **Metric type:** Average CPU utilization
  - **Target value:** `50`
  - **Instance warmup:** `60` seconds

> 💡 **What this means:** The ASG will always run at least 2 instances. If average CPU goes above 50%, it scales up (max 4). If CPU drops, it scales back down (min 2).

- Click **Next**

### Step 5 of 7: Add notifications (optional)
- Skip — click **Next**

### Step 6 of 7: Add tags
Click **Add tag**:
- **Key:** `Name`
- **Value:** `dost-ptri-day3-[yourname]-web-server`
- Check ✅ **Tag new instances**

> This ensures every instance launched by the ASG is named `dost-ptri-day3-[yourname]-web-server` in the console.

- Click **Next**

### Step 7 of 7: Review
- Review all settings
- Click **Create Auto Scaling group**

## Step 2: Watch Instances Launch

1. Go to **EC2 Console** → **Instances**
2. You should see **2 new instances** launching with the name `dost-ptri-day3-[yourname]-web-server`
3. Wait for both to reach **Running** state and pass **2/2 status checks** (takes 1-2 minutes)

> 💡 Because the instances use a pre-baked AMI (from the master), they boot much faster than if they had to run a user data script.

## Step 3: Verify Target Group Health

1. Go to **EC2 Console** → **Target Groups** → Select `dost-ptri-day3-[yourname]-tg`
2. Click the **Targets** tab
3. Both instances should show status: **healthy**

> ⏳ It may take 1-2 minutes after launch for health checks to pass. Status will go from "initial" → "healthy".

## Step 4: Verify Auto Scaling Group

1. Go to **EC2 Console** → **Auto Scaling Groups** → Select `dost-ptri-day3-[yourname]-asg`
2. Check the **Activity** tab — you should see two "Launching a new EC2 instance" events
3. Check the **Instance management** tab — both instances should show "InService"

## How Auto Scaling Works

```
                    Auto Scaling Group (dost-ptri-day3-[yourname]-asg)
                    Min: 2 | Desired: 2 | Max: 4
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         Web Server 1    Web Server 2    (Web Server 3)
         [always on]     [always on]     [scales up if
                                          CPU > 50%]

         All launched from dost-ptri-day3-[yourname]-web-ami (pre-configured)
```

**Scale-out scenario:**
1. Traffic increases → CPU utilization rises above 50%
2. CloudWatch alarm triggers
3. ASG launches new instance(s) from `dost-ptri-day3-[yourname]-web-ami` — ready in seconds
4. New instance registers with ALB target group
5. ALB starts sending traffic to the new instance

**Scale-in scenario:**
1. Traffic decreases → CPU drops below 50%
2. ASG terminates excess instances (down to min of 2)
3. ALB stops sending traffic to terminated instances

## Verification Checklist

| Check | Expected |
|-------|----------|
| ASG `dost-ptri-day3-[yourname]-asg` | Created, desired=2, min=2, max=4 |
| Instances | 2 running, named `dost-ptri-day3-[yourname]-web-server` |
| Target group `dost-ptri-day3-[yourname]-tg` | 2 targets, both healthy |
| Scaling policy | Target tracking on CPU at 50% |
| Activity history | 2 successful launch events |

---

## 📐 Update Your Architecture Diagram

Open your draw.io file and add the Auto Scaling Group — this completes the architecture:

1. **Auto Scaling Group** — Draw a dashed orange box inside the public subnets
   - Label: `Auto Scaling Group: dost-ptri-day3-[yourname]-asg (min:2, max:4)`
   - In draw.io, search for "auto scaling" in the AWS shape library
2. **EC2 Web Servers** — Place 2 EC2 icons inside the ASG box
   - Label each: `dost-ptri-day3-[yourname]-web-server`
3. **Draw arrows:**
   - ALB → EC2 Web Server 1 (solid orange)
   - ALB → EC2 Web Server 2 (solid orange)
   - EC2 Web Server 1 → EC2 DB Server (dashed red, labeled `Port 3306`)
   - EC2 Web Server 2 → EC2 DB Server (dashed red, labeled `Port 3306`)
4. **Add tier labels:**
   - "Web Tier (Public)" next to the public subnets
   - "Database Tier (Private)" next to the private subnet
5. **Add a traffic flow legend** in the corner:
   - Blue solid = Inbound (HTTP/HTTPS)
   - Red dashed = Outbound Response
   - Orange solid = ALB to EC2
   - Red dashed = DB Traffic (Private)

### ✅ Your completed architecture diagram should look like this:

```
Users ──[HTTP]──→ [IGW] ──→ [ALB]
                               │
         ┌── Public Subnets ───┴──────────────────────┐
         │  ┌── Auto Scaling Group (min:2, max:4) ──┐ │
         │  │  [EC2 Web Server 1]  [EC2 Web Server 2]│ │  ← Web Tier
         │  └────────┬──────────────────┬────────────┘ │
         └───────────┼──────────────────┼──────────────┘
                     │   Port 3306      │
         ┌───────────┴──────────────────┴──────────────┐
         │              [EC2 DB Server]                 │  ← Database Tier
         │               10.0.2.xx                     │
         └──────────────────────────────────────────────┘
```

> 🎉 **Congratulations!** Your draw.io diagram now matches the architecture you built. Compare it with the reference diagram `Day3-MultiTier-EC2-Architecture.drawio` in the Day 3 folder.

---

**← Previous:** [07 - Set Up the Application Load Balancer](./07-setup-load-balancer.md) | **Next:** [09 - Testing & Validation →](./09-testing-and-validation.md)
