# 06 - Create a Launch Template

## Goal
Create an EC2 Launch Template using the custom AMI from the master instance. The Auto Scaling Group will use this template to launch identical, pre-configured web server instances — no bootstrapping needed at boot time.

## Prerequisites
- Completed [05 - Create the EC2 Master Instance & AMI](./05-create-ec2-master-and-ami.md)
- AMI `dost-ptri-day3-[yourname]-web-ami` is **available** (note the AMI ID)
- `dost-ptri-day3-[yourname]-web-sg` exists

---

## Step 1: Create the Launch Template

1. Go to **EC2 Console** → **Launch Templates** → **Create launch template**
2. Configure:

### Launch template name and description
- **Launch template name:** `dost-ptri-day3-[yourname]-web-lt`
- **Template version description:** `Web server from dost-ptri-day3-[yourname]-web-ami`
- Check ✅ **Provide guidance to help me set up a template that I can use with EC2 Auto Scaling**

### Application and OS Images (AMI)
- Click **My AMIs** tab
- Select **Owned by me**
- Select `dost-ptri-day3-[yourname]-web-ami`

> 💡 **Key difference from using a stock AMI:** Since the master instance already has Apache, PHP, and the web app installed, every instance launched from this AMI is ready to serve traffic immediately — no user data script needed.

### Instance Type
- Select **t2.micro** (Free tier eligible)

### Key Pair
- Select `dost-ptri-day3-[yourname]-keypair`

### Network Settings
- **Subnet:** Don't include in launch template (the ASG will handle subnet selection)
- **Security groups:** Select `dost-ptri-day3-[yourname]-web-sg`

### Advanced Details
- **No user data needed** — everything is pre-baked in the AMI

3. Click **Create launch template**

## Step 2: Verify the Launch Template

1. Go to **EC2 Console** → **Launch Templates**
2. Select `dost-ptri-day3-[yourname]-web-lt`
3. Click on the **Version** to see details
4. Verify:
   - **AMI:** `dost-ptri-day3-[yourname]-web-ami` (your custom AMI ID)
   - **Instance type:** t2.micro
   - **Security group:** `dost-ptri-day3-[yourname]-web-sg`
   - **User data:** Empty (not needed)

## Master Instance vs User Data: Why This Approach?

| Approach | Pros | Cons |
|----------|------|------|
| **Master/AMI (this tutorial)** | Fast boot, consistent, testable, no boot failures | Need to rebuild AMI for changes |
| **User Data script** | Flexible, always latest packages | Slower boot, can fail silently, harder to debug |

In production, the master/AMI approach is preferred because:
- Instances are **ready in seconds** instead of minutes
- No risk of a package install failing during boot
- You tested the exact image before deploying it

## How This Connects to the Architecture

```
dost-ptri-day3-[yourname]-web-ami (custom AMI)
    │
    └── dost-ptri-day3-[yourname]-web-lt (Launch Template)
              │
              └── Auto Scaling Group (next tutorials)
                        │
                        ├── EC2 Web Server 1 (exact copy of master)
                        └── EC2 Web Server 2 (exact copy of master)
```

## Verification Checklist

| Check | Expected |
|-------|----------|
| Template name | `dost-ptri-day3-[yourname]-web-lt` |
| AMI | `dost-ptri-day3-[yourname]-web-ami` (custom, owned by you) |
| Instance type | t2.micro |
| Security group | `dost-ptri-day3-[yourname]-web-sg` |
| User data | Empty |

---

## 📐 Update Your Architecture Diagram

No new icons needed for the Launch Template itself (it's a configuration, not a running resource). But update your diagram to show the relationship:

1. **Add a note/label** near the public subnet: `Launch Template: dost-ptri-day3-[yourname]-web-lt → Uses dost-ptri-day3-[yourname]-web-ami`
2. **You can remove the EC2 Master** from the diagram now (or grey it out) — it's stopped and the AMI replaces it

Your diagram should now show:

```
┌── Public Subnet ─────────────────────────────────────┐
│  [IGW]                                    [NAT GW]   │
│                                                      │
│  Launch Template (web-lt) → from web-ami             │
│  (ASG will launch instances here — next tutorials)   │
└──────────────────────────────────────────────────────┘
┌── Private Subnet ────────────────────────────────────┐
│         [EC2 DB Server]                              │
└──────────────────────────────────────────────────────┘
```

---

**← Previous:** [05 - Create the EC2 Master Instance & AMI](./05-create-ec2-master-and-ami.md) | **Next:** [07 - Set Up the Application Load Balancer →](./07-setup-load-balancer.md)
