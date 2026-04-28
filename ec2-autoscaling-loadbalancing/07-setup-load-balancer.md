# 07 - Set Up the Application Load Balancer

## Goal
Create an Application Load Balancer (ALB) that distributes incoming HTTP traffic across the web server instances. The ALB sits in the public subnet and forwards requests to a Target Group.

## Prerequisites
- Completed [06 - Create a Launch Template](./06-create-launch-template.md)
- `dost-ptri-day3-[yourname]-vpc`, `dost-ptri-day3-[yourname]-public-subnet`, and `dost-ptri-day3-[yourname]-web-sg` exist

### Important: You Need a Second Public Subnet
ALBs require **at least two subnets in different Availability Zones**. Create a second public subnet first:

1. Go to **VPC Console** → **Subnets** → **Create subnet**
2. Configure:
   - **VPC:** `dost-ptri-day3-[yourname]-vpc`
   - **Subnet name:** `dost-ptri-day3-[yourname]-public-subnet-2`
   - **Availability Zone:** Choose a **different AZ** from the first subnet (e.g., `ap-southeast-1b`)
   - **IPv4 CIDR block:** `10.0.3.0/24`
3. Click **Create subnet**
4. Select `dost-ptri-day3-[yourname]-public-subnet-2` → **Actions** → **Edit subnet settings**
5. Check ✅ **Enable auto-assign public IPv4 address** → **Save**
6. Go to **Route Tables** → Select `dost-ptri-day3-[yourname]-public-rt` → **Subnet associations** → **Edit subnet associations**
7. Check ✅ both `dost-ptri-day3-[yourname]-public-subnet` and `dost-ptri-day3-[yourname]-public-subnet-2` → **Save associations**

---

## Step 1: Create a Target Group

The Target Group defines where the ALB sends traffic and how it checks instance health.

1. Go to **EC2 Console** → **Target Groups** → **Create target group**
2. Configure:

### Basic configuration
- **Target type:** Instances
- **Target group name:** `dost-ptri-day3-[yourname]-tg`
- **Protocol / Port:** HTTP / 80
- **VPC:** Select `dost-ptri-day3-[yourname]-vpc`
- **Protocol version:** HTTP1

### Health checks
- **Health check protocol:** HTTP
- **Health check path:** `/index.php`
- **Advanced health check settings:**
  - Healthy threshold: `2`
  - Unhealthy threshold: `3`
  - Timeout: `5` seconds
  - Interval: `10` seconds
  - Success codes: `200`

3. Click **Next**
4. **Do NOT register any targets** — the Auto Scaling Group will do this automatically
5. Click **Create target group**

## Step 2: Create the Application Load Balancer

1. Go to **EC2 Console** → **Load Balancers** → **Create load balancer**
2. Select **Application Load Balancer** → **Create**
3. Configure:

### Basic configuration
- **Load balancer name:** `dost-ptri-day3-[yourname]-alb`
- **Scheme:** Internet-facing
- **IP address type:** IPv4

### Network mapping
- **VPC:** Select `dost-ptri-day3-[yourname]-vpc`
- **Mappings:** Check ✅ both Availability Zones
  - Select `dost-ptri-day3-[yourname]-public-subnet` for the first AZ
  - Select `dost-ptri-day3-[yourname]-public-subnet-2` for the second AZ

### Security groups
- Remove the default security group
- Select `dost-ptri-day3-[yourname]-web-sg`

### Listeners and routing
- **Protocol:** HTTP
- **Port:** 80
- **Default action:** Forward to → Select `dost-ptri-day3-[yourname]-tg`

4. Click **Create load balancer**

> ⏳ The ALB takes 2-3 minutes to become "Active"

## Step 3: Get the ALB DNS Name

1. Go to **EC2 Console** → **Load Balancers**
2. Select `dost-ptri-day3-[yourname]-alb`
3. Copy the **DNS name** (e.g., `dost-ptri-day3-[yourname]-alb-1234567890.ap-southeast-1.elb.amazonaws.com`)

> 📝 **Save this DNS name** — you'll use it to test the full architecture in the final tutorial.

> ⚠️ Don't test it yet — there are no instances registered. You'll get a 503 error until the Auto Scaling Group creates instances.

## How Traffic Flows Through the ALB

```
Users ──→ dost-ptri-day3-[yourname]-alb (DNS name)
              │
              ├── Health check: GET /index.php every 10s
              │
              ├──→ EC2 Web Server 1 (healthy ✅) ──→ serves response
              └──→ EC2 Web Server 2 (healthy ✅) ──→ serves response
```

- The ALB **round-robins** requests across healthy instances
- If an instance fails health checks, the ALB **stops sending traffic** to it
- The Auto Scaling Group replaces unhealthy instances automatically

## Verification Checklist

| Check | Expected |
|-------|----------|
| Target group `dost-ptri-day3-[yourname]-tg` | Created, protocol HTTP:80, health check `/index.php` |
| Load balancer `dost-ptri-day3-[yourname]-alb` | Active, internet-facing, in both public subnets |
| Listener | HTTP:80 forwarding to `dost-ptri-day3-[yourname]-tg` |
| Security group | `dost-ptri-day3-[yourname]-web-sg` attached |
| DNS name | Copied and saved |

---

## 📐 Update Your Architecture Diagram

Open your draw.io file and add the load balancer components:

1. **Second Public Subnet** — Add another green box next to the first public subnet
   - Label: `dost-ptri-day3-[yourname]-public-subnet-2 (10.0.3.0/24)`
2. **Application Load Balancer** — Place an ALB icon spanning both public subnets
   - Label: `dost-ptri-day3-[yourname]-alb`
   - In draw.io, search for "elastic load balancing" in the AWS shape library
3. **Draw arrows:**
   - Users → Internet Gateway (labeled `Inbound HTTP/HTTPS`)
   - Internet Gateway → ALB (labeled `Port 80/443`)
   - Internet Gateway ← ALB (dashed, labeled `Outbound Response`)

Your diagram should now show:

```
Users ──→ [IGW] ──→ [ALB]
                      │
          ┌── Public Subnet 1 ──┐  ┌── Public Subnet 2 ──┐
          │                     │  │                      │
          │    (ALB spans both subnets)                   │
          └─────────────────────┘  └──────────────────────┘
          ┌── Private Subnet ───────────────────────────────┐
          │         [EC2 DB Server]                         │
          └─────────────────────────────────────────────────┘
```

---

**← Previous:** [06 - Create a Launch Template](./06-create-launch-template.md) | **Next:** [08 - Create the Auto Scaling Group →](./08-create-auto-scaling-group.md)
