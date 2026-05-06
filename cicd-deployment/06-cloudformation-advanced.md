# Lab 06 — CloudFormation Advanced: Serverless API Stack

## Difficulty: ⭐⭐⭐ Advanced

## Objective

Deploy a complete serverless application stack using a single CloudFormation template — S3 bucket, DynamoDB table, Lambda function, and API Gateway. This demonstrates how IaC can provision an entire application in one shot.

**What you'll create:**
- 1 S3 Bucket (for storing files)
- 1 DynamoDB Table (for data)
- 1 Lambda Function (application logic)
- 1 IAM Role (Lambda permissions)
- 1 API Gateway REST API (HTTP endpoint)
- 1 Lambda Permission (API Gateway → Lambda)

**Time:** ~40 min

---

## Step 1: Write the Template (20 min)

Create `cfn-advanced.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Lab 06 - Advanced: Serverless API Stack (S3 + DynamoDB + Lambda + API Gateway)'

Parameters:
  ProjectName:
    Type: String
    Default: dost-ptri-day6
    Description: Prefix for all resource names

  Environment:
    Type: String
    Default: training
    AllowedValues:
      - training
      - development
      - production

Resources:
  # --- S3 Bucket (file storage) ---
  StorageBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub ${ProjectName}-storage-${AWS::AccountId}
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # --- DynamoDB Table (data store) ---
  DataTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub ${ProjectName}-items
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # --- IAM Role for Lambda ---
  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub ${ProjectName}-lambda-role
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: AppPermissions
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - dynamodb:PutItem
                  - dynamodb:GetItem
                  - dynamodb:Scan
                Resource: !GetAtt DataTable.Arn
              - Effect: Allow
                Action:
                  - s3:GetObject
                  - s3:PutObject
                Resource: !Sub ${StorageBucket.Arn}/*

  # --- Lambda Function ---
  AppFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: !Sub ${ProjectName}-api-handler
      Runtime: python3.12
      Handler: index.handler
      Role: !GetAtt LambdaRole.Arn
      Timeout: 10
      Environment:
        Variables:
          TABLE_NAME: !Ref DataTable
          BUCKET_NAME: !Ref StorageBucket
      Code:
        ZipFile: |
          import json
          import os
          import boto3
          from datetime import datetime

          dynamodb = boto3.resource('dynamodb')
          table = dynamodb.Table(os.environ['TABLE_NAME'])

          def handler(event, context):
              method = event.get('httpMethod', 'GET')

              if method == 'GET':
                  result = table.scan()
                  return {
                      'statusCode': 200,
                      'headers': {'Content-Type': 'application/json'},
                      'body': json.dumps({
                          'items': result.get('Items', []),
                          'count': result.get('Count', 0)
                      })
                  }

              elif method == 'POST':
                  body = json.loads(event.get('body', '{}'))
                  item = {
                      'id': context.aws_request_id,
                      'data': body.get('data', ''),
                      'created_at': datetime.now().isoformat()
                  }
                  table.put_item(Item=item)
                  return {
                      'statusCode': 201,
                      'headers': {'Content-Type': 'application/json'},
                      'body': json.dumps(item)
                  }

              return {
                  'statusCode': 200,
                  'headers': {'Content-Type': 'application/json'},
                  'body': json.dumps({
                      'message': 'DOST PTRI Day 6 - Serverless API',
                      'environment': os.environ.get('TABLE_NAME')
                  })
              }
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # --- API Gateway ---
  ApiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: !Sub ${ProjectName}-api
      Description: Day 6 Serverless API

  ApiResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref ApiGateway
      ParentId: !GetAtt ApiGateway.RootResourceId
      PathPart: items

  ApiMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref ApiGateway
      ResourceId: !Ref ApiResource
      HttpMethod: ANY
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${AppFunction.Arn}/invocations

  ApiDeployment:
    Type: AWS::ApiGateway::Deployment
    DependsOn: ApiMethod
    Properties:
      RestApiId: !Ref ApiGateway
      StageName: !Ref Environment

  # --- Lambda Permission (allow API Gateway to invoke) ---
  LambdaApiPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref AppFunction
      Action: lambda:InvokeFunction
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${ApiGateway}/*

Outputs:
  ApiUrl:
    Description: API Gateway endpoint URL
    Value: !Sub https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/${Environment}/items

  FunctionName:
    Description: Lambda function name
    Value: !Ref AppFunction

  TableName:
    Description: DynamoDB table name
    Value: !Ref DataTable

  BucketName:
    Description: S3 bucket name
    Value: !Ref StorageBucket
```

### New concepts in this template:

| Concept | Example | Purpose |
|---------|---------|---------|
| `AWS::AccountId` | `!Sub ...-${AWS::AccountId}` | Pseudo parameter — your account ID |
| `AWS::Region` | `!Sub ...${AWS::Region}...` | Pseudo parameter — current region |
| Inline Lambda code | `ZipFile: \|` | Embed function code directly in template |
| IAM Policy | `Policies:` | Grant Lambda access to DynamoDB + S3 |
| API Gateway proxy | `AWS_PROXY` | Pass full request to Lambda |
| Cross-resource refs | `!GetAtt`, `!Ref`, `!Sub` | Wire everything together |

---

## Step 2: Deploy the Stack (10 min)

1. Open **CloudFormation Console** → **Create stack**
2. Upload `cfn-advanced.yaml`
3. Fill in:
   - Stack name: `day6-advanced-YOURNAME`
   - ProjectName: `dost-ptri-YOURNAME`
   - Environment: `training`
4. Click **Next** → **Next**
5. ⚠️ Check **"I acknowledge that AWS CloudFormation might create IAM resources with custom names"**
6. Click **Submit**
7. Wait for `CREATE_COMPLETE` (~2-3 minutes)

---

## Step 3: Test the API (5 min)

1. Go to CloudFormation **Outputs** tab → copy the **ApiUrl**

### Test with GET (list items):

```bash
curl YOUR_API_URL
```

Response:
```json
{"items": [], "count": 0}
```

### Test with POST (create an item):

```bash
curl -X POST YOUR_API_URL \
  -H "Content-Type: application/json" \
  -d '{"data": "Hello from CloudFormation!"}'
```

Response:
```json
{"id": "xxx-xxx", "data": "Hello from CloudFormation!", "created_at": "2026-05-07T..."}
```

### Test GET again:

```bash
curl YOUR_API_URL
```

Response now shows your item! ✅

---

## Step 4: Verify Resources (5 min)

Check each service in the console:

| Service | What to verify |
|---------|---------------|
| **S3** | Bucket exists with versioning enabled |
| **DynamoDB** | Table exists, items tab shows your POST data |
| **Lambda** | Function exists, environment variables set |
| **API Gateway** | REST API with `/items` resource, deployed to `training` stage |
| **IAM** | Role with DynamoDB + S3 permissions |

---

## Step 5: Clean Up

1. **Empty the S3 bucket first** (CloudFormation can't delete non-empty buckets):
   - Go to S3 → select bucket → **Empty** → confirm
2. Go to CloudFormation → select stack → **Delete**
3. All resources removed ✅

---

## ✅ Lab Complete!

You deployed a complete serverless application with a single template:

| Resource | Purpose |
|----------|---------|
| S3 | File storage |
| DynamoDB | Data persistence |
| Lambda | Application logic |
| API Gateway | HTTP endpoint |
| IAM Role | Permissions |

**6 resources, 1 template, 1 click to deploy, 1 click to destroy.**

**Key concepts practiced:** Pseudo parameters (`AWS::AccountId`, `AWS::Region`), inline Lambda code, IAM roles/policies, API Gateway proxy integration, cross-resource references
