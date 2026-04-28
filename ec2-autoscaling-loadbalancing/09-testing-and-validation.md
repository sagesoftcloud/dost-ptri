# 09 - Testing & Validation

## Goal
Verify the entire multi-tier architecture works end-to-end: load balancing distributes traffic, web servers connect to the database, and auto scaling responds to load.

## Prerequisites
- Completed all previous tutorials (01–08)
- ALB DNS name saved from [07 - Set Up the ALB](./07-setup-load-balancer.md)
- ASG instances are running and healthy in the target group

---

## Test 1: Load Balancer Distribution

1. Open a browser and go to your ALB DNS name:
   ```
   http://dost-ptri-day3-[yourname]-alb-XXXXXXXXXX.ap-southeast-1.elb.amazonaws.com
   ```
2. You should see a page showing:
   - **Instance ID** (e.g., `i-0abc123def456`)
   - **Availability Zone**
   - **Private IP**
   - **Database connection status**

3. **Refresh the page multiple times** (Ctrl+Shift+R or Cmd+Shift+R to bypass cache)
4. You should see the **Instance ID change** between refreshes — this proves the ALB is distributing traffic across both web servers

> ✅ **Pass:** Different instance IDs appear on refresh
> ❌ **Fail:** Same instance ID every time — check ALB listener and target group configuration

## Test 2: Database Connectivity

On the web page, check the database connection status:

> ✅ **Pass:** "Database: Connected successfully to 10.0.2.xx"
> ❌ **Fail:** "Database: Connection FAILED"

**If the database connection fails, check:**
1. Is `dost-ptri-day3-[yourname]-db-server` running? (EC2 Console → Instances)
2. Is the private IP in the AMI correct? (You may need to update the master, re-bake the AMI, and update the Launch Template)
3. Does `dost-ptri-day3-[yourname]-db-sg` allow inbound MySQL (3306) from `dost-ptri-day3-[yourname]-web-sg`?
4. Is MariaDB running on the DB server?

## Test 3: Auto Scaling (Scale Out)

Simulate high CPU to trigger the scaling policy.

1. Go to **EC2 Console** → **Instances**
2. Select one of the `dost-ptri-day3-[yourname]-web-server` instances
3. Click **Connect** → **EC2 Instance Connect** tab → **Connect**
4. Run this command to install and run a CPU stress tool:
   ```bash
   sudo yum install -y stress-ng
   stress-ng --cpu 4 --timeout 300
   ```
   This maxes out CPU for 5 minutes.

5. Do the same on the **second** web server instance.

6. **Monitor the scaling:**
   - Go to **EC2 Console** → **Auto Scaling Groups** → `dost-ptri-day3-[yourname]-asg`
   - Click the **Activity** tab
   - After ~2-3 minutes, you should see a new "Launching a new EC2 instance" event
   - The desired capacity should increase from 2 to 3 (or 4)

7. Go to **EC2 Console** → **Instances** — you should see new `dost-ptri-day3-[yourname]-web-server` instances launching

> ✅ **Pass:** New instances launch when CPU is high
> ❌ **Fail:** No scaling after 5 minutes — check the scaling policy in the ASG

## Test 4: Auto Scaling (Scale In)

1. Wait for the `stress-ng` command to finish (or press Ctrl+C to stop it)
2. After ~5-10 minutes of low CPU, the ASG should **terminate** the extra instances
3. Check the **Activity** tab for "Terminating an EC2 instance" events
4. The instance count should return to **2** (the desired/minimum capacity)

> ✅ **Pass:** Extra instances are terminated, count returns to 2

## Test 5: Self-Healing

Test that the ASG replaces failed instances.

1. Go to **EC2 Console** → **Instances**
2. Select one of the `dost-ptri-day3-[yourname]-web-server` instances
3. Click **Instance state** → **Terminate instance** → **Terminate**
4. Go to **Auto Scaling Groups** → `dost-ptri-day3-[yourname]-asg` → **Activity** tab
5. Within 1-2 minutes, you should see:
   - A termination event for the old instance
   - A launch event for a **new replacement** instance
6. The instance count should stay at **2**

> ✅ **Pass:** ASG automatically launches a replacement instance from the custom AMI

## Architecture Summary

After completing all tests, you have verified:

```
Users ──[HTTP]──→ Internet Gateway ──→ ALB (dost-ptri-day3-[yourname]-alb)
                                         │
                                    ┌────┴────┐
                                    │         │
                               Web Server  Web Server    ← Auto Scaling Group
                               (10.0.1.x)  (10.0.3.x)     (min:2, max:4)
                                    │         │              Launched from dost-ptri-day3-[yourname]-web-ami
                                    └────┬────┘
                                         │
                                    [Port 3306]
                                         │
                                    DB Server            ← Private Subnet
                                    (10.0.2.x)             (no internet inbound)
                                         │
                                    NAT Gateway          ← Outbound only
```

| Feature | Status |
|---------|--------|
| Load balancing across instances | ✅ Verified |
| Database connectivity from web tier | ✅ Verified |
| Auto scaling (scale out on high CPU) | ✅ Verified |
| Auto scaling (scale in on low CPU) | ✅ Verified |
| Self-healing (replace terminated instances) | ✅ Verified |
| Private subnet isolation (DB not internet-accessible) | ✅ Verified |
| Pre-baked AMI (fast instance launch) | ✅ Verified |

---

## 🧹 Cleanup (IMPORTANT)

To avoid ongoing charges, delete resources in this order:

1. **Auto Scaling Group:** EC2 Console → Auto Scaling Groups → Select `dost-ptri-day3-[yourname]-asg` → Delete → Set min/max/desired to 0 → Confirm
2. **Load Balancer:** EC2 Console → Load Balancers → Select `dost-ptri-day3-[yourname]-alb` → Actions → Delete
3. **Target Group:** EC2 Console → Target Groups → Select `dost-ptri-day3-[yourname]-tg` → Actions → Delete
4. **DB Instance:** EC2 Console → Instances → Select `dost-ptri-day3-[yourname]-db-server` → Instance state → Terminate
5. **Master Instance:** EC2 Console → Instances → Select `dost-ptri-day3-[yourname]-ec2-master` → Instance state → Terminate
6. **Custom AMI:** EC2 Console → AMIs → Select `dost-ptri-day3-[yourname]-web-ami` → Actions → Deregister AMI
7. **Snapshot:** EC2 Console → Snapshots → Delete the snapshot created with the AMI
8. **NAT Gateway:** VPC Console → NAT Gateways → Select `dost-ptri-day3-[yourname]-nat-gw` → Actions → Delete
9. **Elastic IP:** VPC Console → Elastic IPs → Select the EIP from the NAT Gateway → Actions → Release
10. **Launch Template:** EC2 Console → Launch Templates → Select `dost-ptri-day3-[yourname]-web-lt` → Actions → Delete
11. **Security Groups:** VPC Console → Security Groups → Delete `dost-ptri-day3-[yourname]-db-sg` first, then `dost-ptri-day3-[yourname]-web-sg`
12. **Subnets:** VPC Console → Subnets → Delete `dost-ptri-day3-[yourname]-public-subnet-2`, `dost-ptri-day3-[yourname]-public-subnet`, `dost-ptri-day3-[yourname]-private-subnet`
13. **Internet Gateway:** VPC Console → Internet Gateways → Detach from VPC → Delete
14. **Route Tables:** VPC Console → Route Tables → Delete `dost-ptri-day3-[yourname]-public-rt` and `dost-ptri-day3-[yourname]-private-rt`
15. **VPC:** VPC Console → Your VPCs → Select `dost-ptri-day3-[yourname]-vpc` → Actions → Delete VPC

> ⚠️ Delete the NAT Gateway and release the Elastic IP promptly — NAT Gateway charges ~$0.045/hour even when idle.

---

**← Previous:** [08 - Create the Auto Scaling Group](./08-create-auto-scaling-group.md) | **Back to:** [00 - Overview & Prerequisites](./00-overview-and-prerequisites.md)
