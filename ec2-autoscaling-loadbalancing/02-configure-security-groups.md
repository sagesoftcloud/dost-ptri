# 02 - Configure Security Groups

## Goal
Create security groups that control inbound and outbound traffic for the web tier and database tier. The web tier accepts HTTP/HTTPS from the internet. The database tier only accepts traffic from the web tier.

## Prerequisites
- Completed [01 - Create a VPC with Subnets](./01-create-vpc-and-subnets.md)
- `dost-ptri-day3-[yourname]-vpc` exists and is available

---

## Step 1: Create the Web Tier Security Group

1. Go to **VPC Console** → **Security Groups** → **Create security group**
2. Configure:
   - **Security group name:** `dost-ptri-day3-[yourname]-web-sg`
   - **Description:** `Allow HTTP and HTTPS inbound for web tier`
   - **VPC:** Select `dost-ptri-day3-[yourname]-vpc`

### Inbound Rules
Click **Add rule** for each:

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| HTTP | TCP | 80 | `0.0.0.0/0` | Allow HTTP from anywhere |
| HTTPS | TCP | 443 | `0.0.0.0/0` | Allow HTTPS from anywhere |
| SSH | TCP | 22 | **My IP** | Allow SSH from your IP only |

> 🔒 For SSH, select "My IP" from the dropdown — this auto-fills your current public IP. Never use `0.0.0.0/0` for SSH in production.

### Outbound Rules
Keep the default:

| Type | Protocol | Port Range | Destination | Description |
|------|----------|------------|-------------|-------------|
| All traffic | All | All | `0.0.0.0/0` | Allow all outbound |

3. Click **Create security group**

## Step 2: Create the Database Tier Security Group

1. Click **Create security group**
2. Configure:
   - **Security group name:** `dost-ptri-day3-[yourname]-db-sg`
   - **Description:** `Allow database traffic from web tier only`
   - **VPC:** Select `dost-ptri-day3-[yourname]-vpc`

### Inbound Rules
Click **Add rule**:

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| MySQL/Aurora | TCP | 3306 | Select `dost-ptri-day3-[yourname]-web-sg` | Allow MySQL from web tier |

> 💡 **Key concept:** The source is the **web security group**, not an IP range. This means only instances attached to `dost-ptri-day3-[yourname]-web-sg` can reach the database. If you use PostgreSQL instead, use port **5432**.

### Outbound Rules
Keep the default:

| Type | Protocol | Port Range | Destination | Description |
|------|----------|------------|-------------|-------------|
| All traffic | All | All | `0.0.0.0/0` | Allow all outbound (for OS updates via NAT) |

3. Click **Create security group**

## How Traffic Flows with These Security Groups

```
Internet ──[HTTP/HTTPS]──→ dost-ptri-day3-[yourname]-web-sg (Port 80/443) ──[MySQL]──→ dost-ptri-day3-[yourname]-db-sg (Port 3306)
                                                                         │
                                                                    NAT Gateway
                                                                         │
                                                                  Internet (outbound only)
```

- **Inbound to web tier:** Anyone on the internet can reach ports 80 and 443
- **Inbound to DB tier:** Only instances in `dost-ptri-day3-[yourname]-web-sg` can reach port 3306
- **Outbound from DB tier:** Goes through NAT Gateway (no direct internet exposure)
- **No direct path** from the internet to the database server

## Verification

| Security Group | Inbound Rules | Outbound Rules |
|---------------|---------------|----------------|
| `dost-ptri-day3-[yourname]-web-sg` | HTTP (80) from `0.0.0.0/0`, HTTPS (443) from `0.0.0.0/0`, SSH (22) from your IP | All traffic to `0.0.0.0/0` |
| `dost-ptri-day3-[yourname]-db-sg` | MySQL (3306) from `dost-ptri-day3-[yourname]-web-sg` | All traffic to `0.0.0.0/0` |

> ✅ **Checkpoint:** Both security groups should appear in the VPC Console under Security Groups, associated with `dost-ptri-day3-[yourname]-vpc`

---

## 📐 Update Your Architecture Diagram

Open your draw.io file and add the security groups:

1. **Web Tier Security Group** — Add a label/note inside the public subnet:
   - Label: `dost-ptri-day3-[yourname]-web-sg`
   - Note: `Inbound: HTTP (80), HTTPS (443), SSH (22 - My IP)`
2. **DB Tier Security Group** — Add a label/note inside the private subnet:
   - Label: `dost-ptri-day3-[yourname]-db-sg`
   - Note: `Inbound: MySQL (3306) from Web SG only`
3. **Draw arrows** showing allowed traffic flow:
   - Solid blue arrow from Internet → Public Subnet (labeled `HTTP/HTTPS`)
   - Dashed red arrow from Public Subnet → Private Subnet (labeled `Port 3306`)
4. Add a **"⛔ No direct inbound from Internet"** label on the private subnet

Your diagram should now show:

```
Internet ──[HTTP/HTTPS]──→ Public Subnet (web-sg: 80, 443)
                                    │
                              [Port 3306]
                                    │
                           Private Subnet (db-sg: 3306 from web-sg only)
                           ⛔ No direct inbound from Internet
```

---

**← Previous:** [01 - Create a VPC with Subnets](./01-create-vpc-and-subnets.md) | **Next:** [03 - Launch the Database Server →](./03-launch-database-server.md)
