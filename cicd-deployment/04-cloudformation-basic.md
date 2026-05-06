# Lab 04 — CloudFormation Basic: S3 Bucket with Versioning

## Difficulty: ⭐ Basic

## Objective

Write your first CloudFormation template from scratch, deploy it, and verify the resource was created correctly.

**What you'll create:**
- 1 S3 Bucket with versioning, encryption, and public access blocked

**Time:** ~20 min

---

## Step 1: Write the Template (10 min)

Create a new file called `cfn-basic.yaml` in your project folder:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Lab 04 - Basic: S3 Bucket with Versioning'

Parameters:
  BucketName:
    Type: String
    Description: Globally unique bucket name (lowercase, no spaces)

  Environment:
    Type: String
    Default: training
    AllowedValues:
      - training
      - development
      - production

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      Tags:
        - Key: Environment
          Value: !Ref Environment
        - Key: ManagedBy
          Value: CloudFormation

Outputs:
  BucketName:
    Description: Name of the bucket
    Value: !Ref MyBucket

  BucketARN:
    Description: ARN of the bucket
    Value: !GetAtt MyBucket.Arn
```

### Understand what each section does:

| Section | Purpose |
|---------|---------|
| `Parameters` | Inputs you provide at deploy time |
| `Resources` | The actual AWS resource to create |
| `!Ref BucketName` | References the parameter value |
| `!GetAtt MyBucket.Arn` | Gets an attribute of the created resource |
| `Outputs` | Values shown after stack creation |

---

## Step 2: Deploy the Stack (5 min)

1. Open **CloudFormation Console** → **Create stack** → **With new resources**
2. **Upload a template file** → select your `cfn-basic.yaml`
3. Click **Next**
4. Fill in:
   - Stack name: `day6-basic-YOURNAME`
   - BucketName: `dost-ptri-basic-YOURNAME` (must be globally unique)
   - Environment: `training`
5. Click **Next** → **Next** → **Submit**
6. Watch the **Events** tab until status is `CREATE_COMPLETE`

---

## Step 3: Verify (3 min)

1. Go to **S3 Console** → find your bucket
2. Click the bucket → **Properties**:
   - ✅ Versioning: Enabled
   - ✅ Encryption: SSE-S3 (AES-256)
3. Click **Permissions**:
   - ✅ Block all public access: On
4. Go back to CloudFormation → **Outputs** tab:
   - ✅ BucketName and BucketARN displayed

---

## Step 4: Clean Up (2 min)

1. Select your stack → **Delete**
2. Confirm → watch `DELETE_COMPLETE`
3. Verify the S3 bucket is gone

---

## ✅ Lab Complete!

You wrote a CloudFormation template from scratch that:
- Creates an S3 bucket with versioning and encryption
- Uses Parameters for customization
- Exports Outputs for reference
- Cleans up automatically on stack deletion

**Key concepts practiced:** Parameters, Resources, Outputs, `!Ref`, `!GetAtt`
