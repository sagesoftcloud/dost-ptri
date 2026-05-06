# Lab 02 — CloudFormation Console Walkthrough

## Objective

Deploy a CloudFormation stack using the provided template, observe how IaC creates resources, and clean up by deleting the stack.

---

## Step 1: Review the Template (10 min)

Open `lab-s3-versioning.yaml` and identify each section:

| Section | Purpose |
|---------|---------|
| `AWSTemplateFormatVersion` | Template format version (always `2010-09-09`) |
| `Description` | Human-readable description |
| `Parameters` | Inputs you customize per deployment |
| `Resources` | The AWS resources to create |
| `Outputs` | Values exported after creation |

### Questions to answer:

1. What resource does this template create?
2. What properties are configured on it?
3. What can you customize via Parameters?

---

## Step 2: Deploy the Stack (15 min)

1. Open the **AWS Console** → search for **CloudFormation**
2. Click **Create stack** → **With new resources (standard)**
3. Select **Upload a template file** → upload `lab-s3-versioning.yaml`
4. Click **Next**

### Fill in the parameters:

| Parameter | Value |
|-----------|-------|
| Stack name | `day6-lab-YOURNAME` |
| BucketName | `dost-ptri-day6-YOURNAME` (must be globally unique) |
| Environment | `training` |

5. Click **Next** → **Next** → **Submit**

### Watch the Events tab:

| Event | Meaning |
|-------|---------|
| `CREATE_IN_PROGRESS` | CloudFormation is creating the resource |
| `CREATE_COMPLETE` | Resource created successfully |

6. Wait until stack status shows **CREATE_COMPLETE**

### Verify in S3:

1. Open the **S3 Console**
2. Find your bucket (`dost-ptri-day6-YOURNAME`)
3. Click on it → **Properties** → confirm **Versioning** is Enabled ✅

---

## Step 3: Explore Stack Features (5 min)

### Outputs tab
- See the exported values: Bucket Name, ARN, Domain Name
- These can be referenced by other stacks

### Resources tab
- See the Physical Resource ID (actual S3 bucket name)
- Click the link to jump directly to the resource

### Detect Drift
1. Click **Stack actions** → **Detect drift**
2. Wait for detection to complete
3. Should show **IN_SYNC** (no manual changes were made)

---

## Step 4: Delete the Stack (5 min)

1. Select your stack → click **Delete**
2. Confirm deletion
3. Watch the Events tab: `DELETE_IN_PROGRESS` → `DELETE_COMPLETE`
4. Go to S3 Console — the bucket is **gone**

> 💡 One click → all resources cleaned up. No orphaned resources. That's the power of Infrastructure as Code.

---

## Key Takeaways

| Concept | What You Saw |
|---------|-------------|
| **Template** | YAML file describing desired infrastructure |
| **Stack** | Running instance of the template |
| **Parameters** | Customizable inputs (bucket name, environment) |
| **Outputs** | Exported values for reference |
| **Drift detection** | Verifies template matches actual state |
| **Stack deletion** | Automatic cleanup of all resources |

---

## ✅ Lab Complete!

You've successfully:
- Read and understood a CloudFormation template
- Deployed a stack via the console
- Verified the created resource (S3 bucket with versioning)
- Explored Outputs, Resources, and Drift detection
- Deleted the stack and confirmed cleanup
