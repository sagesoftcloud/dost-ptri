# 05 - Create the EC2 Master Instance & AMI

## Goal
Using the EC2 master instance (launched in the previous tutorial), manually install and configure all the web server software, verify everything works, then create a custom AMI (Amazon Machine Image) from it. This AMI becomes the golden image used by the Launch Template and Auto Scaling Group.

## Why Use a Master Instance?
- **Consistency:** Every instance launched from the AMI is identical — no reliance on user data scripts that might fail
- **Speed:** Instances launch faster because software is pre-installed (no bootstrapping at boot time)
- **Testability:** You can SSH in, verify everything works, and fix issues before baking the AMI
- **Repeatability:** The AMI can be versioned and reused across environments

## Prerequisites
- Completed [04 - Test Connectivity with Telnet](./04-test-connectivity.md)
- `dost-ptri-day3-[yourname]-ec2-master` is **Running** (launched in tutorial 04)
- Telnet test to DB on port 3306 passed ✅
- Note the **private IP** of `dost-ptri-day3-[yourname]-db-server` (e.g., `10.0.2.xx`)

---

## Step 1: Connect to the EC2 Master

1. Go to **EC2 Console** → **Instances**
2. Select `dost-ptri-day3-[yourname]-ec2-master`
3. Click **Connect** → **EC2 Instance Connect** → **Connect**

## Step 2: Install and Configure Web Server Software

Run the following commands one by one in the terminal:

### Update the system
```bash
sudo yum update -y
```

### Install Apache, PHP, and MySQL client
```bash
sudo yum install -y httpd php php-mysqlnd
```

### Start and enable Apache
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Verify Apache is running
```bash
sudo systemctl status httpd
```
You should see `active (running)`.

## Step 3: Create the Web Application

Create the PHP page that displays instance info and tests database connectivity.

> ⚠️ **Replace `10.0.2.xx`** with the actual private IP of your `dost-ptri-day3-[yourname]-db-server`.

```bash
sudo tee /var/www/html/index.php > /dev/null <<'ENDOFFILE'
<?php
// Get instance metadata using IMDSv2
$token = file_get_contents("http://169.254.169.254/latest/api/token", false,
    stream_context_create(["http" => ["method" => "PUT", "header" => "X-aws-ec2-metadata-token-ttl-seconds: 21600"]]));
$headers = ["http" => ["header" => "X-aws-ec2-metadata-token: $token"]];
$ctx = stream_context_create($headers);

$instance_id = file_get_contents("http://169.254.169.254/latest/meta-data/instance-id", false, $ctx);
$az = file_get_contents("http://169.254.169.254/latest/meta-data/placement/availability-zone", false, $ctx);
$private_ip = file_get_contents("http://169.254.169.254/latest/meta-data/local-ipv4", false, $ctx);

echo "<h1>Day 3 - Multi-Tier Architecture</h1>";
echo "<p><b>Instance ID:</b> $instance_id</p>";
echo "<p><b>Availability Zone:</b> $az</p>";
echo "<p><b>Private IP:</b> $private_ip</p>";
echo "<hr>";

// Test DB connection
$db_host = "10.0.2.xx";
$db_user = "webuser";
$db_pass = "WebUser123!";
$db_name = "day3app";

$conn = @new mysqli($db_host, $db_user, $db_pass, $db_name);
if ($conn->connect_error) {
    echo "<p style='color:red'><b>Database:</b> Connection FAILED - " . $conn->connect_error . "</p>";
} else {
    echo "<p style='color:green'><b>Database:</b> Connected successfully to $db_host</p>";
    $conn->close();
}
?>
ENDOFFILE
```

Now replace the placeholder IP with your actual DB IP:
```bash
sudo sed -i 's/10.0.2.xx/10.0.2.ACTUAL_IP/g' /var/www/html/index.php
```

> Replace `10.0.2.ACTUAL_IP` with the real private IP of your database server.

## Step 4: Test the Master Instance

### Test Apache locally
```bash
curl http://localhost/index.php
```
You should see HTML output with instance metadata.

### Test from your browser
1. Copy the **Public IPv4 address** of `dost-ptri-day3-[yourname]-ec2-master`
2. Open in browser: `http://<PUBLIC_IP>`
3. You should see:
   - Instance ID
   - Availability Zone
   - Private IP
   - Database connection status (green = connected, red = failed)

> ✅ **If the page loads and DB shows "Connected successfully"** — the master is ready for AMI creation.

> ❌ **If DB connection fails**, check:
> - Is `dost-ptri-day3-[yourname]-db-server` running?
> - Is the IP address correct in `index.php`?
> - Does `dost-ptri-day3-[yourname]-db-sg` allow port 3306 from `dost-ptri-day3-[yourname]-web-sg`?

### Test Apache starts on boot
```bash
sudo systemctl is-enabled httpd
```
Should output: `enabled`

## Step 5: Clean Up Before Creating the AMI

Before creating the AMI, clean up temporary files so the image is clean:

```bash
# Clear command history
history -c

# Clear temp files
sudo rm -rf /tmp/*

# Clear yum cache
sudo yum clean all
```

## Step 6: Create the Custom AMI

1. Go back to **EC2 Console** → **Instances**
2. Select `dost-ptri-day3-[yourname]-ec2-master`
3. Click **Actions** → **Image and templates** → **Create image**
4. Configure:
   - **Image name:** `dost-ptri-day3-[yourname]-web-ami`
   - **Image description:** `Day 3 web server - Apache, PHP, MySQL client pre-installed`
   - **No reboot:** Leave **unchecked** (the instance will reboot to ensure a consistent image)
   - **Storage:** Keep defaults
5. Click **Create image**

> ⏳ AMI creation takes **3-5 minutes**. The instance will reboot during this process.

### Verify the AMI
1. Go to **EC2 Console** → **AMIs** (left sidebar under Images)
2. You should see `dost-ptri-day3-[yourname]-web-ami` with status **available**
3. **Note the AMI ID** (e.g., `ami-0abc123def456`) — you'll need this for the Launch Template

## Step 7: Stop the Master Instance

The master instance is no longer needed for the lab. Stop it to save costs:

1. Select `dost-ptri-day3-[yourname]-ec2-master`
2. Click **Instance state** → **Stop instance**

> 💡 **Don't terminate it yet** — keep it stopped in case you need to make changes and create a new AMI. You can terminate it during cleanup.

## How This Connects to the Architecture

```
dost-ptri-day3-[yourname]-ec2-master (staging)
    │
    ├── [Tutorial 04] Telnet test — verified DB connectivity ✅
    ├── [Tutorial 05] Install Apache, PHP, MySQL client
    ├── Create web application
    ├── Test everything works
    │
    └── Create AMI (dost-ptri-day3-[yourname]-web-ami)
              │
              └── Used by: Launch Template (next tutorial)
                        │
                        └── Used by: Auto Scaling Group
                                  │
                                  ├── EC2 Web Server 1 (identical copy)
                                  └── EC2 Web Server 2 (identical copy)
```

## Verification Checklist

| Check | Expected |
|-------|----------|
| `dost-ptri-day3-[yourname]-ec2-master` | Running (or stopped after AMI creation) |
| Apache | Running, enabled on boot |
| Web page | Loads at `http://<PUBLIC_IP>`, shows instance info |
| DB connection | Green "Connected successfully" on the web page |
| AMI `dost-ptri-day3-[yourname]-web-ami` | Status: available |
| AMI ID | Noted for next tutorial |

---

## 📐 Update Your Architecture Diagram

Open your draw.io file and update the EC2 Master:

1. **Update the EC2 Master label** to: `dost-ptri-day3-[yourname]-ec2-master (AMI Source)`
2. **Add a dashed box** around the EC2 Master with label: `Custom AMI: dost-ptri-day3-[yourname]-web-ami`
3. **Remove the telnet test arrows** from the previous tutorial (those were for testing only)
4. **Add a note** near the EC2 Master: `Apache + PHP + MySQL Client pre-installed`

Your diagram should now show:

```
┌── Public Subnet ─────────────────────────────────────┐
│  [IGW]                                    [NAT GW]   │
│                                                      │
│  ┌─ AMI Source ────────────────────────┐             │
│  │  [EC2 Master]                       │             │
│  │  Apache + PHP + MySQL Client        │             │
│  │  → Creates: web-ami                 │             │
│  └─────────────────────────────────────┘             │
└──────────────────────────────────────────────────────┘
┌── Private Subnet ────────────────────────────────────┐
│         [EC2 DB Server]                              │
└──────────────────────────────────────────────────────┘
```

---

**← Previous:** [04 - Test Connectivity with Telnet](./04-test-connectivity.md) | **Next:** [06 - Create a Launch Template →](./06-create-launch-template.md)
