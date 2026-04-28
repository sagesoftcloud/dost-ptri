# Day 3 - Multi-Tier EC2 Architecture: Overview & Prerequisites

## Architecture Overview

This series of tutorials walks you through building a **multi-tier architecture** on AWS using the Console (ClickOps). By the end, you will have:

- A **VPC** with public and private subnets
- An **Internet Gateway** for inbound/outbound internet traffic
- A **NAT Gateway** for private subnet outbound access
- **Security Groups** controlling inbound and outbound traffic per tier
- An **EC2 Master Instance** used as a staging area to build and test the web server image
- A **Custom AMI** created from the master instance
- An **Application Load Balancer (ALB)** distributing traffic to web servers
- An **Auto Scaling Group (ASG)** with EC2 web servers launched from the custom AMI
- An **EC2 Database Server** in a private subnet

```
Users → Internet Gateway → ALB → [Auto Scaling Group: EC2 Web Servers (from custom AMI)] → EC2 Database Server
              (Public Subnet)                                                                 (Private Subnet)
```

## Tutorial Order

Each tutorial builds on the previous one. Follow them in order. **At the end of each tutorial, you will update your architecture diagram in draw.io** to reflect the resources you just created — building the diagram step by step as you build the infrastructure.

> 📐 Open [draw.io](https://app.diagrams.net) and create a new blank diagram. Save it as `Day3-MultiTier-EC2-Architecture.drawio`. You will update this file after every tutorial.

| # | Tutorial | What It Creates |
|---|----------|----------------|
| 1 | [Create a VPC with Subnets](./01-create-vpc-and-subnets.md) | VPC, Public Subnet, Private Subnet, Internet Gateway, NAT Gateway, Route Tables |
| 2 | [Configure Security Groups](./02-configure-security-groups.md) | Web Tier SG (HTTP/HTTPS), Database Tier SG (MySQL from web tier only) |
| 3 | [Launch the Database Server](./03-launch-database-server.md) | EC2 instance in the private subnet running MySQL/PostgreSQL |
| 4 | [Test Connectivity with Telnet](./04-test-connectivity.md) | Verify DB reachability, security group rules, and network isolation |
| 5 | [Create the EC2 Master Instance & AMI](./05-create-ec2-master-and-ami.md) | EC2 master/staging instance, install & test software, create custom AMI |
| 6 | [Create a Launch Template](./06-create-launch-template.md) | Launch Template using the custom AMI |
| 7 | [Set Up the Application Load Balancer](./07-setup-load-balancer.md) | ALB, Target Group, Listener rules |
| 8 | [Create the Auto Scaling Group](./08-create-auto-scaling-group.md) | ASG attached to ALB with scaling policies |
| 9 | [Testing & Validation](./09-testing-and-validation.md) | End-to-end verification of the architecture |

## Prerequisites

### AWS Account
- An active AWS account with **AdministratorAccess** or equivalent IAM permissions
- Access to the **AWS Management Console**

### Region
- Use a single region throughout all tutorials (recommended: **ap-southeast-1** Singapore or **us-east-1** N. Virginia)
- Ensure the region supports all services used (EC2, VPC, ELB, Auto Scaling)

### Knowledge
- Basic understanding of networking concepts (IP addresses, CIDR, subnets, ports)
- Familiarity with the AWS Management Console navigation
- Understanding of what EC2 instances are

### Key Pair
- Create an **EC2 Key Pair** before starting:
  1. Go to **EC2 Console** → **Key Pairs** (left sidebar under Network & Security)
  2. Click **Create key pair**
  3. Name: `dost-ptri-day3-[yourname]-keypair`
  4. Key pair type: **RSA**
  5. Private key file format: **.pem** (macOS/Linux) or **.ppk** (Windows/PuTTY)
  6. Click **Create key pair** — the file downloads automatically
  7. **Save this file securely** — you cannot download it again

### Cost Awareness
- This lab uses resources that **incur charges** (EC2, NAT Gateway, ALB)
- Estimated cost: **~$2-5 USD** if completed and cleaned up within a few hours
- **Always clean up resources** after completing the tutorials to avoid ongoing charges
- NAT Gateway charges ~$0.045/hour + data processing fees
- ALB charges ~$0.0225/hour + LCU charges

### Naming Convention

Replace `[yourname]` with your actual name (e.g., `juan`, `maria`). Use consistent naming throughout all tutorials:

| Resource | Name |
|----------|------|
| VPC | `dost-ptri-day3-[yourname]-vpc` |
| Public Subnet | `dost-ptri-day3-[yourname]-public-subnet` |
| Private Subnet | `dost-ptri-day3-[yourname]-private-subnet` |
| Internet Gateway | `dost-ptri-day3-[yourname]-igw` |
| NAT Gateway | `dost-ptri-day3-[yourname]-nat-gw` |
| Web Security Group | `dost-ptri-day3-[yourname]-web-sg` |
| DB Security Group | `dost-ptri-day3-[yourname]-db-sg` |
| EC2 Master Instance | `dost-ptri-day3-[yourname]-ec2-master` |
| Custom AMI | `dost-ptri-day3-[yourname]-web-ami` |
| Launch Template | `dost-ptri-day3-[yourname]-web-lt` |
| Load Balancer | `dost-ptri-day3-[yourname]-alb` |
| Target Group | `dost-ptri-day3-[yourname]-tg` |
| Auto Scaling Group | `dost-ptri-day3-[yourname]-asg` |
| DB Instance | `dost-ptri-day3-[yourname]-db-server` |
| Key Pair | `dost-ptri-day3-[yourname]-keypair` |

> 💡 **Example:** If your name is Juan, your VPC would be `dost-ptri-day3-juan-vpc`, your ALB would be `dost-ptri-day3-juan-alb`, etc.

---

**Next:** [01 - Create a VPC with Subnets →](./01-create-vpc-and-subnets.md)
