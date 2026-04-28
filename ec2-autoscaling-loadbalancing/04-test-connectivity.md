# 04 - Test Connectivity with Telnet

## Goal
Before building the AMI, verify that the network, security groups, and database are working correctly. You'll launch a temporary test instance in the public subnet and use `telnet` to confirm connectivity between tiers.

## Why Test Now?
If you skip this step and something is misconfigured (wrong security group, wrong subnet, DB not running), you'll only discover it after building the AMI and creating the full ASG — wasting time. Testing now catches issues early.

## Prerequisites
- Completed [03 - Launch the Database Server](./03-launch-database-server.md)
- `dost-ptri-day3-[yourname]-db-server` is **Running**
- `dost-ptri-day3-[yourname]-web-sg` and `dost-ptri-day3-[yourname]-db-sg` exist
- Note the **private IP** of `dost-ptri-day3-[yourname]-db-server` (e.g., `10.0.2.xx`)

---

## Step 1: Launch a Test Instance (EC2 Master)

We'll use the EC2 master instance for testing. If you haven't launched it yet, do so now:

1. Go to **EC2 Console** → **Instances** → **Launch instances**
2. Configure:
   - **Name:** `dost-ptri-day3-[yourname]-ec2-master`
   - **AMI:** Amazon Linux 2023 AMI
   - **Instance type:** t2.micro
   - **Key pair:** `dost-ptri-day3-[yourname]-keypair`
   - **Network:** `dost-ptri-day3-[yourname]-vpc`
   - **Subnet:** `dost-ptri-day3-[yourname]-public-subnet`
   - **Auto-assign public IP:** Enable
   - **Security group:** Select `dost-ptri-day3-[yourname]-web-sg`
3. Click **Launch instance**
4. Wait for **Running** + **2/2 status checks**

## Step 2: Connect to the EC2 Master

1. Select `dost-ptri-day3-[yourname]-ec2-master`
2. Click **Connect** → **EC2 Instance Connect** → **Connect**

## Step 3: Install Telnet

```bash
sudo yum install -y telnet
```

## Step 4: Test Connectivity to the Database (Port 3306)

Run telnet to the database server's private IP on port 3306:

```bash
telnet 10.0.2.xx 3306
```

> ⚠️ Replace `10.0.2.xx` with the actual private IP of `dost-ptri-day3-[yourname]-db-server`.

### ✅ Success — Connection Established
You should see something like:
```
Trying 10.0.2.xx...
Connected to 10.0.2.xx.
Escape character is '^]'.
```
You may also see some garbled characters — that's the MySQL protocol handshake. This means **the connection works**.

Press `Ctrl+]` then type `quit` to exit telnet.

### ❌ Failure — Connection Refused or Timeout

**If you see "Connection refused":**
- MariaDB is not running on the DB server
- SSH into the DB server (via a bastion or Session Manager) and run:
  ```bash
  sudo systemctl status mariadb
  sudo systemctl start mariadb
  ```

**If you see "Connection timed out" (hangs for 10+ seconds):**
- Security group issue — `dost-ptri-day3-[yourname]-db-sg` is not allowing port 3306 from `dost-ptri-day3-[yourname]-web-sg`
- Go to **VPC Console** → **Security Groups** → Select `dost-ptri-day3-[yourname]-db-sg`
- Verify inbound rule: **MySQL/Aurora (3306)** from source `dost-ptri-day3-[yourname]-web-sg`

**If you see "No route to host":**
- Route table issue — the public subnet can't reach the private subnet
- Both subnets should be in the same VPC (`dost-ptri-day3-[yourname]-vpc`)
- Check that the local route `10.0.0.0/16` exists in both route tables (it's added automatically)

## Step 5: Test Internet Connectivity (Outbound)

Verify the EC2 master can reach the internet (needed for installing packages):

```bash
telnet google.com 443
```

### ✅ Success
```
Trying xxx.xxx.xxx.xxx...
Connected to google.com.
```
Press `Ctrl+]` then type `quit` to exit.

### ❌ Failure
- Check that `dost-ptri-day3-[yourname]-public-rt` has route `0.0.0.0/0` → Internet Gateway
- Check that `dost-ptri-day3-[yourname]-web-sg` allows all outbound traffic

## Step 6: Test That the Private Subnet Has No Direct Inbound

From your **local machine** (not the EC2 instance), try to reach the DB server:

```bash
telnet 10.0.2.xx 3306
```

This should **fail** (timeout or no route) — confirming the database is not accessible from outside the VPC.

> ✅ **Pass:** Connection times out or fails — the private subnet is properly isolated.

## Step 7: Test DNS Resolution (Optional)

From the EC2 master, verify DNS works inside the VPC:

```bash
nslookup google.com
```

Should return an IP address. If it fails, check that **DNS resolution** and **DNS hostnames** are enabled on the VPC.

## Summary of Tests

| Test | Command | Expected Result |
|------|---------|----------------|
| EC2 Master → DB (port 3306) | `telnet 10.0.2.xx 3306` | Connected ✅ |
| EC2 Master → Internet (port 443) | `telnet google.com 443` | Connected ✅ |
| Local machine → DB (port 3306) | `telnet 10.0.2.xx 3306` | Timeout/Fail ✅ |
| DNS resolution | `nslookup google.com` | Returns IP ✅ |

> 📝 **Keep the EC2 master running** — you'll use it in the next tutorial to install the web server software and create the AMI.

---

## 📐 Update Your Architecture Diagram

Open your draw.io file and add the connectivity test results:

1. **EC2 Master** — Place an EC2 icon inside the **public subnet**
   - Label: `dost-ptri-day3-[yourname]-ec2-master`
   - This is temporary (will become the AMI source)
2. **Draw a dashed arrow** from EC2 Master → EC2 DB Server
   - Label: `telnet :3306 ✅`
3. **Draw a dashed arrow** from EC2 Master → Internet (outbound)
   - Label: `telnet :443 ✅`
4. **Add a crossed-out arrow** from Internet → EC2 DB Server
   - Label: `❌ Blocked`

This visually confirms the network isolation you just tested:

```
                    ┌── Public Subnet ──────────────────┐
Internet ──✅──→   │  [EC2 Master] ──telnet:3306──✅──→│──→ [EC2 DB Server]
Internet ──❌──→   │                                    │    (Private Subnet)
                    └───────────────────────────────────┘
```

> 📝 The EC2 Master icon will be updated in the next tutorial when it becomes the AMI source.

---

**← Previous:** [03 - Launch the Database Server](./03-launch-database-server.md) | **Next:** [05 - Create the EC2 Master Instance & AMI →](./05-create-ec2-master-and-ami.md)
