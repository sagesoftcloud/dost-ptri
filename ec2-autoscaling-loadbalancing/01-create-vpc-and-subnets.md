# 01 - Create a VPC with Subnets

## Goal
Create the network foundation: a VPC with a public subnet (for web servers and ALB) and a private subnet (for the database server), along with an Internet Gateway, NAT Gateway, and route tables.

## Prerequisites
- Completed [00 - Overview & Prerequisites](./00-overview-and-prerequisites.md)
- EC2 Key Pair created (`dost-ptri-day3-[yourname]-keypair`)

---

## Step 1: Create the VPC

1. Go to **VPC Console** → [https://console.aws.amazon.com/vpc](https://console.aws.amazon.com/vpc)
2. Click **Create VPC**
3. Configure:
   - **Resources to create:** VPC only
   - **Name tag:** `dost-ptri-day3-[yourname]-vpc`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **IPv6 CIDR block:** No IPv6 CIDR block
   - **Tenancy:** Default
4. Click **Create VPC**

> ✅ **Checkpoint:** You should see `dost-ptri-day3-[yourname]-vpc` in the VPC list with state "Available"

## Step 2: Create the Public Subnet

1. In the VPC Console, click **Subnets** → **Create subnet**
2. Configure:
   - **VPC ID:** Select `dost-ptri-day3-[yourname]-vpc`
   - **Subnet name:** `dost-ptri-day3-[yourname]-public-subnet`
   - **Availability Zone:** Choose the first AZ (e.g., `ap-southeast-1a`)
   - **IPv4 subnet CIDR block:** `10.0.1.0/24`
3. Click **Create subnet**

### Enable Auto-Assign Public IP
1. Select `dost-ptri-day3-[yourname]-public-subnet`
2. Click **Actions** → **Edit subnet settings**
3. Check ✅ **Enable auto-assign public IPv4 address**
4. Click **Save**

## Step 3: Create the Private Subnet

1. Click **Subnets** → **Create subnet**
2. Configure:
   - **VPC ID:** Select `dost-ptri-day3-[yourname]-vpc`
   - **Subnet name:** `dost-ptri-day3-[yourname]-private-subnet`
   - **Availability Zone:** Same AZ as the public subnet (e.g., `ap-southeast-1a`)
   - **IPv4 subnet CIDR block:** `10.0.2.0/24`
3. Click **Create subnet**

> ⚠️ Do NOT enable auto-assign public IP for the private subnet.

## Step 4: Create and Attach an Internet Gateway

1. Click **Internet Gateways** → **Create internet gateway**
2. **Name tag:** `dost-ptri-day3-[yourname]-igw`
3. Click **Create internet gateway**
4. On the next screen, click **Actions** → **Attach to VPC**
5. Select `dost-ptri-day3-[yourname]-vpc` → Click **Attach internet gateway**

> ✅ **Checkpoint:** The IGW state should show "Attached"

## Step 5: Create a NAT Gateway

The NAT Gateway allows the private subnet (database server) to access the internet for updates/patches without being directly reachable from the internet.

1. Click **NAT Gateways** → **Create NAT gateway**
2. Configure:
   - **Name:** `dost-ptri-day3-[yourname]-nat-gw`
   - **Subnet:** Select `dost-ptri-day3-[yourname]-public-subnet` (NAT Gateway must be in a public subnet)
   - **Connectivity type:** Public
   - **Elastic IP allocation ID:** Click **Allocate Elastic IP** → it auto-fills
3. Click **Create NAT gateway**

> ⏳ The NAT Gateway takes 1-2 minutes to become "Available"

## Step 6: Configure Route Tables

### Public Route Table

1. Click **Route Tables** → **Create route table**
2. Configure:
   - **Name:** `dost-ptri-day3-[yourname]-public-rt`
   - **VPC:** Select `dost-ptri-day3-[yourname]-vpc`
3. Click **Create route table**
4. Select `dost-ptri-day3-[yourname]-public-rt` → **Routes** tab → **Edit routes**
5. Click **Add route**:
   - **Destination:** `0.0.0.0/0`
   - **Target:** Select **Internet Gateway** → `dost-ptri-day3-[yourname]-igw`
6. Click **Save changes**
7. Go to **Subnet associations** tab → **Edit subnet associations**
8. Check ✅ `dost-ptri-day3-[yourname]-public-subnet`
9. Click **Save associations**

### Private Route Table

1. Click **Route Tables** → **Create route table**
2. Configure:
   - **Name:** `dost-ptri-day3-[yourname]-private-rt`
   - **VPC:** Select `dost-ptri-day3-[yourname]-vpc`
3. Click **Create route table**
4. Select `dost-ptri-day3-[yourname]-private-rt` → **Routes** tab → **Edit routes**
5. Click **Add route**:
   - **Destination:** `0.0.0.0/0`
   - **Target:** Select **NAT Gateway** → `dost-ptri-day3-[yourname]-nat-gw`
6. Click **Save changes**
7. Go to **Subnet associations** tab → **Edit subnet associations**
8. Check ✅ `dost-ptri-day3-[yourname]-private-subnet`
9. Click **Save associations**

## Verification

| Resource | Expected State |
|----------|---------------|
| `dost-ptri-day3-[yourname]-vpc` | Available, CIDR `10.0.0.0/16` |
| `dost-ptri-day3-[yourname]-public-subnet` | Available, CIDR `10.0.1.0/24`, auto-assign public IP enabled |
| `dost-ptri-day3-[yourname]-private-subnet` | Available, CIDR `10.0.2.0/24`, auto-assign public IP disabled |
| `dost-ptri-day3-[yourname]-igw` | Attached to `dost-ptri-day3-[yourname]-vpc` |
| `dost-ptri-day3-[yourname]-nat-gw` | Available, in `dost-ptri-day3-[yourname]-public-subnet` |
| `dost-ptri-day3-[yourname]-public-rt` | Route `0.0.0.0/0` → IGW, associated with public subnet |
| `dost-ptri-day3-[yourname]-private-rt` | Route `0.0.0.0/0` → NAT GW, associated with private subnet |

---

## 📐 Update Your Architecture Diagram

Open your `Day3-MultiTier-EC2-Architecture.drawio` file in [draw.io](https://app.diagrams.net) and add the following:

1. **AWS Cloud** — Draw the outer boundary box. Label: `AWS Cloud`
2. **VPC** — Draw a box inside the AWS Cloud. Label: `dost-ptri-day3-[yourname]-vpc (10.0.0.0/16)`
3. **Public Subnet** — Draw a green box inside the VPC. Label: `dost-ptri-day3-[yourname]-public-subnet (10.0.1.0/24)`
4. **Private Subnet** — Draw a red/orange box below the public subnet inside the VPC. Label: `dost-ptri-day3-[yourname]-private-subnet (10.0.2.0/24)`
5. **Internet Gateway** — Place the IGW icon on the left edge of the VPC. Label: `dost-ptri-day3-[yourname]-igw`
6. **NAT Gateway** — Place inside the public subnet. Label: `dost-ptri-day3-[yourname]-nat-gw`
7. **Route Tables** — Add small labels near each subnet:
   - Public: `0.0.0.0/0 → IGW`
   - Private: `0.0.0.0/0 → NAT GW`

> 💡 **Tip:** In draw.io, search for "aws" in the shape library to find official AWS architecture icons. Use **AWS 19 / Networking** for VPC, IGW, and NAT Gateway shapes.

Your diagram should look like this so far:

```
┌─────────────────── AWS Cloud ───────────────────┐
│  ┌──────────────── VPC (10.0.0.0/16) ─────────┐ │
│  │  ┌── Public Subnet (10.0.1.0/24) ────────┐ │ │
│  │  │  [IGW]              [NAT GW]           │ │ │
│  │  └────────────────────────────────────────┘ │ │
│  │  ┌── Private Subnet (10.0.2.0/24) ───────┐ │ │
│  │  │                                        │ │ │
│  │  └────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

---

**← Previous:** [00 - Overview & Prerequisites](./00-overview-and-prerequisites.md) | **Next:** [02 - Configure Security Groups →](./02-configure-security-groups.md)
