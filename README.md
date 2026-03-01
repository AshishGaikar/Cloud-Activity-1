# 🚀 AWS Resource Provisioning using Python (Boto3)

## 📘 Cloud Native Application – Activity 1

**Name:**Ashish Sanjay Gaikar
**PRN:** 202301040070  
**Semester:** VI  
**Course:** Cloud Native Application  
**Region Used:** ap-south-1 (Mumbai)

---

## 📌 Objective

To configure AWS SDK and programmatically launch AWS resources using Python (Boto3), including:

- VPC
- EC2
- IAM
- S3

This activity demonstrates Infrastructure as Code (IaC) principles using AWS SDK.

---

## 🛠 Environment Setup

- Python already installed  
- Boto3 already installed  
- AWS CLI already configured  
- Default region: `ap-south-1`

No additional installation was required.

![Terminal Output](screenshots/Execution.png)

---

## 🪜 Step-by-Step Implementation

### ✅ Step 1: Create VPC

- CIDR Block: `10.0.0.0/16`
- Enabled DNS support
- Tagged with name: `Activity-VPC`

![VPC Dashboard](screenshots/vpc.png)

---

### ✅ Step 2: Create Subnet

- CIDR Block: `10.0.1.0/24`
- Tagged with name: `Activity-Subnet`

![Subnet Details](screenshots/subnet.png)

---

### ✅ Step 3: Create Internet Gateway

- Created IGW
- Attached to VPC

![Internet Gateway](screenshots/igw.png)

---

### ✅ Step 4: Create Route Table

- Created Route Table
- Added default route `0.0.0.0/0 → IGW`
- Associated with subnet

![Route Table](screenshots/route-table.png)

---

### ✅ Step 5: Create Security Group

- Allowed inbound SSH (Port 22)
- Tagged as `Activity-SG`

![Security Group](screenshots/security-group.png)

---

### ✅ Step 6: Create Key Pair

- Key Name: `ActivityKeyPair`
- Used for EC2 SSH access

![Key Pair](screenshots/keypair.png)

---

### ✅ Step 7: Launch EC2 Instance

- Instance Type: `t3.micro`
- Hardcoded AMI
- Launched inside created subnet
- Attached security group
- Associated key pair

![EC2 Running](screenshots/ec2.png)

![EC2 Details](screenshots/ec2-details.png)

---

### ✅ Step 8: Create S3 Bucket

- Created bucket in `ap-south-1`

![S3 Bucket](screenshots/s3.png)

---

## 💻 Complete Python Implementation (`activity1.py`)

```python
import boto3

region = "ap-south-1"

ec2 = boto3.client("ec2", region_name=region)
s3 = boto3.client("s3", region_name=region)

# 1️⃣ Create VPC
vpc = ec2.create_vpc(CidrBlock="10.0.0.0/16")
vpc_id = vpc["Vpc"]["VpcId"]
ec2.create_tags(Resources=[vpc_id], Tags=[{"Key": "Name", "Value": "Activity-VPC"}])
ec2.modify_vpc_attribute(VpcId=vpc_id, EnableDnsSupport={"Value": True})
ec2.modify_vpc_attribute(VpcId=vpc_id, EnableDnsHostnames={"Value": True})

# 2️⃣ Create Subnet
subnet = ec2.create_subnet(VpcId=vpc_id, CidrBlock="10.0.1.0/24")
subnet_id = subnet["Subnet"]["SubnetId"]
ec2.create_tags(Resources=[subnet_id], Tags=[{"Key": "Name", "Value": "Activity-Subnet"}])

# 3️⃣ Create Internet Gateway
igw = ec2.create_internet_gateway()
igw_id = igw["InternetGateway"]["InternetGatewayId"]
ec2.attach_internet_gateway(InternetGatewayId=igw_id, VpcId=vpc_id)

# 4️⃣ Create Route Table
route_table = ec2.create_route_table(VpcId=vpc_id)
route_table_id = route_table["RouteTable"]["RouteTableId"]
ec2.create_route(RouteTableId=route_table_id,
                 DestinationCidrBlock="0.0.0.0/0",
                 GatewayId=igw_id)
ec2.associate_route_table(RouteTableId=route_table_id,
                          SubnetId=subnet_id)

# 5️⃣ Create Security Group
sg = ec2.create_security_group(
    GroupName="Activity-SG",
    Description="Allow SSH",
    VpcId=vpc_id
)
sg_id = sg["GroupId"]

ec2.authorize_security_group_ingress(
    GroupId=sg_id,
    IpPermissions=[{
        "IpProtocol": "tcp",
        "FromPort": 22,
        "ToPort": 22,
        "IpRanges": [{"CidrIp": "0.0.0.0/0"}]
    }]
)

# 6️⃣ Create Key Pair
key_name = "ActivityKeyPair"
ec2.create_key_pair(KeyName=key_name)

# 7️⃣ Launch EC2 Instance
ami_id = "ami-0f5ee92e2d63afc18"

ec2.run_instances(
    ImageId=ami_id,
    InstanceType="t3.micro",
    KeyName=key_name,
    MinCount=1,
    MaxCount=1,
    SubnetId=subnet_id,
    SecurityGroupIds=[sg_id]
)

# 8️⃣ Create S3 Bucket
s3.create_bucket(
    Bucket="activity-bucket-unique-123456",
    CreateBucketConfiguration={"LocationConstraint": region}
)

print("All resources created successfully")
```

## 🎯 Conclusion

This activity demonstrates automated AWS infrastructure provisioning using Python (Boto3), including networking, compute, and storage services within a custom VPC.

---