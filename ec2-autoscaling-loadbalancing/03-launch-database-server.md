# 03 - Launch the Database Server

## Goal
Launch an EC2 instance in the private subnet to serve as the database server. This instance will only be accessible from the web tier through the database security group.

## Prerequisites
- Completed [02 - Configure Security Groups](./02-configure-security-groups.md)
- `dost-ptri-day3-[yourname]-vpc`, `dost-ptri-day3-[yourname]-private-subnet`, and `dost-ptri-day3-[yourname]-db-sg` exist
- Key pair `dost-ptri-day3-[yourname]-keypair` created

---

## Step 1: Launch the EC2 Instance

1. Go to **EC2 Console** → **Instances** → **Launch instances**
2. Configure:

### Name and Tags
- **Name:** `dost-ptri-day3-[yourname]-db-server`

### Application and OS Images (AMI)
- Select **Amazon Linux 2023 AMI** (Free tier eligible)
- Architecture: **64-bit (x86)**

### Instance Type
- Select **t2.micro** (Free tier eligible)

### Key Pair
- Select `dost-ptri-day3-[yourname]-keypair`

### Network Settings
Click **Edit**, then configure:
- **VPC:** Select `dost-ptri-day3-[yourname]-vpc`
- **Subnet:** Select `dost-ptri-day3-[yourname]-private-subnet`
- **Auto-assign public IP:** **Disable** (this is a private instance)
- **Firewall (security groups):** Select **Select existing security group**
- Choose `dost-ptri-day3-[yourname]-db-sg`

### Configure Storage
- **8 GiB** gp3 (default is fine for this lab)

### Advanced Details (User Data)
Expand **Advanced details**, scroll to **User data**, and paste:

```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y mariadb105-server
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Set root password and create a database
sudo mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED BY 'Day3LabPassword123!';"
sudo mysql -u root -p'Day3LabPassword123!' -e "CREATE DATABASE day3app;"
sudo mysql -u root -p'Day3LabPassword123!' -e "CREATE USER 'webuser'@'%' IDENTIFIED BY 'WebUser123!';"
sudo mysql -u root -p'Day3LabPassword123!' -e "GRANT ALL PRIVILEGES ON day3app.* TO 'webuser'@'%';"
sudo mysql -u root -p'Day3LabPassword123!' -e "FLUSH PRIVILEGES;"
```

> ⚠️ **Note:** These are lab-only credentials. Never use simple passwords in production. Use AWS Secrets Manager instead.

3. Click **Launch instance**

## Step 2: Verify the Instance

1. Go to **EC2 Console** → **Instances**
2. Select `dost-ptri-day3-[yourname]-db-server`
3. Verify:
   - **Instance state:** Running
   - **Private IPv4 address:** Should be in the `10.0.2.x` range
   - **Public IPv4 address:** Should be **empty** (no public IP)
   - **Security groups:** `dost-ptri-day3-[yourname]-db-sg`
   - **Subnet:** `dost-ptri-day3-[yourname]-private-subnet`

> 📝 **Note the Private IPv4 address** (e.g., `10.0.2.xx`) — you will need this in the Launch Template for the web servers.

## Step 3: Verify No Direct Internet Access (Inbound)

Since this instance is in the private subnet with `dost-ptri-day3-[yourname]-db-sg`:
- ❌ You **cannot** SSH to it directly from the internet
- ❌ You **cannot** reach it via HTTP from the internet
- ✅ It **can** reach the internet outbound (via NAT Gateway) for updates
- ✅ It **can** be reached on port 3306 from instances in `dost-ptri-day3-[yourname]-web-sg`

## How This Connects to the Architecture

```
                    [Private Subnet]
                          │
                    dost-ptri-day3-[yourname]-db-server
                    (10.0.2.x:3306)
                          │
              Only accepts from dost-ptri-day3-[yourname]-web-sg
```

The web servers (created in the next tutorials) will connect to this database using its **private IP address** on port 3306.

## Verification Checklist

| Check | Expected |
|-------|----------|
| Instance state | Running |
| Subnet | `dost-ptri-day3-[yourname]-private-subnet` |
| Public IP | None |
| Security group | `dost-ptri-day3-[yourname]-db-sg` |
| Private IP | `10.0.2.x` range |

---

## 📐 Update Your Architecture Diagram

Open your draw.io file and add the database server:

1. **EC2 Database Server** — Place an EC2 icon inside the **private subnet**
   - Label: `dost-ptri-day3-[yourname]-db-server`
   - Add the private IP below it (e.g., `10.0.2.xx`)
2. **Add a "Database Tier" label** next to the private subnet

> 💡 In draw.io, search for "ec2" in the AWS shape library to find the EC2 instance icon.

Your diagram should now show:

```
┌── Public Subnet ─────────────────────────┐
│  [IGW]              [NAT GW]             │
└──────────────────────────────────────────┘
┌── Private Subnet ────────────────────────┐
│         [EC2 DB Server]                  │  ← Database Tier
│          10.0.2.xx                       │
└──────────────────────────────────────────┘
```

---

**← Previous:** [02 - Configure Security Groups](./02-configure-security-groups.md) | **Next:** [04 - Test Connectivity with Telnet →](./04-test-connectivity.md)
