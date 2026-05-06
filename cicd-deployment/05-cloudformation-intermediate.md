# Lab 05 — CloudFormation Intermediate: VPC with Subnets & Security Group

## Difficulty: ⭐⭐ Intermediate

## Objective

Build a networking foundation using CloudFormation — a VPC with public/private subnets, an Internet Gateway, and a security group. This introduces multi-resource templates and resource references.

**What you'll create:**
- 1 VPC
- 1 Public Subnet
- 1 Private Subnet
- 1 Internet Gateway (attached to VPC)
- 1 Route Table with internet route
- 1 Security Group (allow HTTP + SSH)

**Time:** ~30 min

---

## Step 1: Write the Template (15 min)

Create `cfn-intermediate.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Lab 05 - Intermediate: VPC with Subnets and Security Group'

Parameters:
  ProjectName:
    Type: String
    Default: dost-ptri-day6
    Description: Prefix for resource names

  VpcCidr:
    Type: String
    Default: 10.0.0.0/16
    Description: CIDR block for the VPC

  PublicSubnetCidr:
    Type: String
    Default: 10.0.1.0/24
    Description: CIDR block for the public subnet

  PrivateSubnetCidr:
    Type: String
    Default: 10.0.2.0/24
    Description: CIDR block for the private subnet

Resources:
  # --- VPC ---
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub ${ProjectName}-vpc

  # --- Internet Gateway ---
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub ${ProjectName}-igw

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # --- Public Subnet ---
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PublicSubnetCidr
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${ProjectName}-public-subnet

  # --- Private Subnet ---
  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PrivateSubnetCidr
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub ${ProjectName}-private-subnet

  # --- Route Table (Public) ---
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${ProjectName}-public-rt

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  # --- Security Group ---
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH access
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: Allow HTTP from anywhere
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
          Description: Allow SSH from anywhere
      Tags:
        - Key: Name
          Value: !Sub ${ProjectName}-web-sg

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC

  PublicSubnetId:
    Description: Public Subnet ID
    Value: !Ref PublicSubnet

  PrivateSubnetId:
    Description: Private Subnet ID
    Value: !Ref PrivateSubnet

  SecurityGroupId:
    Description: Security Group ID
    Value: !Ref WebSecurityGroup
```

### New concepts in this template:

| Concept | Example | Purpose |
|---------|---------|---------|
| `!Sub` | `!Sub ${ProjectName}-vpc` | String substitution with variables |
| `!Select` | `!Select [0, !GetAZs '']` | Pick an item from a list |
| `!GetAZs` | `!GetAZs ''` | Get available AZs in the region |
| `DependsOn` | `DependsOn: AttachGateway` | Explicit dependency ordering |
| Multiple resources | 8 resources | Resources reference each other |

---

## Step 2: Deploy the Stack (10 min)

1. Open **CloudFormation Console** → **Create stack**
2. Upload `cfn-intermediate.yaml`
3. Fill in:
   - Stack name: `day6-vpc-YOURNAME`
   - ProjectName: `dost-ptri-YOURNAME`
   - Leave CIDR defaults as-is
4. Click **Next** → **Next** → **Submit**
5. Watch the Events tab — notice the **order of creation**:
   - VPC first (other resources depend on it)
   - Internet Gateway + Subnets (depend on VPC)
   - Route Table + Security Group (depend on VPC)
   - Route + Association (depend on Route Table + IGW)

> 💡 CloudFormation figures out the dependency order automatically from `!Ref` usage.

---

## Step 3: Verify (5 min)

1. Go to **VPC Console**:
   - ✅ New VPC with CIDR `10.0.0.0/16`
   - ✅ 2 subnets (public + private)
   - ✅ Internet Gateway attached
2. Go to **Route Tables**:
   - ✅ Public route table has `0.0.0.0/0 → igw-xxx`
3. Go to **Security Groups**:
   - ✅ Inbound rules: HTTP (80) + SSH (22)
4. Check CloudFormation **Outputs** tab:
   - ✅ VPC ID, Subnet IDs, Security Group ID

---

## Step 4: Clean Up

1. Select your stack → **Delete**
2. All 8 resources are removed automatically in the correct reverse order

---

## ✅ Lab Complete!

You built a multi-resource CloudFormation template that:
- Creates a full VPC networking stack (8 resources)
- Uses resource references (`!Ref`) to wire them together
- CloudFormation resolves dependency order automatically
- One-click cleanup removes everything

**Key concepts practiced:** `!Sub`, `!Select`, `!GetAZs`, `DependsOn`, multi-resource dependencies
