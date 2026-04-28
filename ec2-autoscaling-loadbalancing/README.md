# 🏗️ Day 3 Lab: Multi-Tier EC2 Architecture with Load Balancing & Auto Scaling

## DOST PTRI - AWS Fundamentals Training Program

### What You Will Build

A production-style multi-tier architecture on AWS using only the Console (ClickOps):

```
Users ──[HTTP]──→ Internet Gateway ──→ Application Load Balancer
                                              │
                                   ┌──────────┴──────────┐
                                   │   Auto Scaling Group │
                                   │  [EC2 Web] [EC2 Web] │  ← Public Subnet (Web Tier)
                                   └──────────┬──────────┘
                                         Port 3306
                                   ┌──────────┴──────────┐
                                   │   [EC2 Database]     │  ← Private Subnet (DB Tier)
                                   └─────────────────────┘
```

### What You Will Learn

- Creating a VPC with public and private subnets
- Configuring Security Groups for inbound/outbound traffic control
- Launching EC2 instances across different network tiers
- Testing network connectivity with telnet
- Building a custom AMI from a master/staging EC2 instance
- Setting up an Application Load Balancer (ALB)
- Creating an Auto Scaling Group with scaling policies
- Building an architecture diagram in draw.io as you go

### Tutorials

| # | Tutorial | Time Estimate |
|---|----------|:------------:|
| 00 | [Overview & Prerequisites](./00-overview-and-prerequisites.md) | 10 min |
| 01 | [Create a VPC with Subnets](./01-create-vpc-and-subnets.md) | 15 min |
| 02 | [Configure Security Groups](./02-configure-security-groups.md) | 10 min |
| 03 | [Launch the Database Server](./03-launch-database-server.md) | 10 min |
| 04 | [Test Connectivity with Telnet](./04-test-connectivity.md) | 10 min |
| 05 | [Create the EC2 Master Instance & AMI](./05-create-ec2-master-and-ami.md) | 20 min |
| 06 | [Create a Launch Template](./06-create-launch-template.md) | 5 min |
| 07 | [Set Up the Application Load Balancer](./07-setup-load-balancer.md) | 15 min |
| 08 | [Create the Auto Scaling Group](./08-create-auto-scaling-group.md) | 15 min |
| 09 | [Testing & Validation](./09-testing-and-validation.md) | 20 min |
| | **Total** | **~2 hours** |

### Prerequisites

- AWS account with AdministratorAccess
- Web browser (for AWS Console and draw.io)
- No CLI or coding experience required — everything is done via the Console

### Naming Convention

All resources follow the pattern: `dost-ptri-day3-[yourname]-<resource>`

Replace `[yourname]` with your name (e.g., `juan`, `maria`).

### Architecture Diagram

📐 You will build your architecture diagram in [draw.io](https://app.diagrams.net) step by step. Each tutorial ends with instructions on what to add to your diagram. A reference diagram is provided in `Day3-MultiTier-EC2-Architecture.drawio`.

### ⚠️ Cost Warning

This lab uses AWS resources that incur charges (~$2-5 USD for a few hours). **Follow the cleanup steps in [Tutorial 09](./09-testing-and-validation.md) immediately after completing the lab** to avoid ongoing charges.

### Estimated Costs (per hour while running)

| Resource | Cost/Hour |
|----------|-----------|
| NAT Gateway | ~$0.045 |
| ALB | ~$0.0225 |
| EC2 t2.micro (×3) | Free tier or ~$0.0116 each |

---

**Start here → [00 - Overview & Prerequisites](./00-overview-and-prerequisites.md)**
