# Automated Deployment of Single-Tier Architecture: Two VPCs with Windows Bastion, Private Ubuntu Server, and VPC Peering

This guide details the automated deployment of a single-tier architecture using AWS. The setup includes two VPCs, each with a Windows bastion server and a private Ubuntu web server. Initially, we will build and test the scripts for the development and QA environments, troubleshoot any issues, and then add VPC peering to enable communication between the environments.

![VPC_Single_Tier_Bastion](/images/vpc-bastion-1.png)

The infrastructure consists of:

- **Development Environment**: 
  - A VPC with public and private subnets.
  - A Windows bastion server for secure access.
  - A private Ubuntu web server.
  - NAT and Internet gateways for network management.

- **QA Environment**:
  - Similar setup with distinct CIDR blocks and resource tags.
  - Initially isolated, with VPC peering to be added later for secure communication between environments.

The use of bastion servers ensures secure access to private resources, while VPC peering will allow seamless integration and testing across environments. This architecture supports efficient development and QA processes by providing isolated yet interconnected environments.

## Breakdown of the Development Environment Script
The key parts of the script are explained below:

### 1. **Configuring Variables**
Variables are defined for the AWS region, availability zones (AZ), CIDR blocks for VPC and subnets, and other AWS-specific details like instance types and AMI IDs.

```bash
REGION="us-east-1"
AZ1="us-east-1a"
AZ2="us-east-1b"
VPC_CIDR="192.168.0.0/16"
PUBLIC_SUBNET_CIDR_1="192.168.1.0/28"
PUBLIC_SUBNET_CIDR_2="192.168.2.0/28"
PRIVATE_SUBNET_CIDR_1="192.168.3.0/24"
PRIVATE_SUBNET_CIDR_2="192.168.4.0/24"
```

---

### 2. **Creating a VPC**
A VPC is created using the specified CIDR block.

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION --query 'Vpc.VpcId' --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=c3ops_dev_vpc}]" --output text)
```

---

### 3. **Creating Subnets**
Both public and private subnets are created within the VPC, each in different availability zones.

```bash
# Public Subnets
PUBLIC_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_1 --availability-zone $AZ1 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_public_subnet_1}]" --query 'Subnet.SubnetId' --output text)
PUBLIC_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_2 --availability-zone $AZ2 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_public_subnet_2}]" --query 'Subnet.SubnetId' --output text)

# Private Subnets
PRIVATE_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_1 --availability-zone $AZ1 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_private_subnet_1}]" --query 'Subnet.SubnetId' --output text)
PRIVATE_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_2 --availability-zone $AZ2 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_private_subnet_2}]" --query 'Subnet.SubnetId' --output text)
```

---

### 4. **Enabling Public IP Assignment**
Public subnets are configured to auto-assign public IPs to instances.

```bash
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_2 --map-public-ip-on-launch
```

---

### 5. **Creating and Attaching an Internet Gateway**
An internet gateway is created and attached to the VPC to enable internet access for public subnets.

```bash
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=c3ops_dev_igw}]" --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID
```

---

### 6. **Creating a NAT Gateway**
A NAT gateway is created in a public subnet, allowing private subnet instances to access the internet. An Elastic IP is also allocated for the NAT gateway.

```bash
# Allocate Elastic IP for NAT Gateway
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Elastic IP Allocation ID for NAT Gateway: $ALLOCATION_ID"

# Create NAT Gateway
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --tag-specifications "ResourceType=natgateway,Tags=[{Key=Name,Value=c3ops_dev_nat_gateway}]" --query 'NatGateway.NatGatewayId' --output text)
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID
echo "NAT Gateway ID: $NAT_GATEWAY_ID"
```

**Explanation**: The Elastic IP (EIP) is crucial for the NAT Gateway as it provides a stable public IP address for outbound internet traffic from instances in private subnets. This ensures that responses to requests from these instances can be routed back correctly.

---

### 7. **Creating Route Tables and Routes**
Separate route tables are created for public and private subnets. Routes to the internet gateway and NAT gateway are defined accordingly.

```bash
# Route Tables
PUBLIC_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=c3ops_dev_public_rtb}]" --query 'RouteTable.RouteTableId' --output text)
PRIVATE_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=c3ops_dev_private_rtb}]" --query 'RouteTable.RouteTableId' --output text)

# Routes
aws ec2 create-route --route-table-id $PUBLIC_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $INTERNET_GATEWAY_ID
aws ec2 create-route --route-table-id $PRIVATE_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID
```

---

### 8. **Associating Subnets with Route Tables**
The created subnets are associated with their respective route tables.

```bash
# Public Subnets with Public Route Table
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_1
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_2

# Private Subnets with Private Route Table
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_1
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_2
```

---

### 9. **Creating Security Groups**
Security groups are created for the bastion host and web server, allowing necessary ingress rules.

```bash
# Bastion Security Group
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "Bastion Security group" --vpc-id $VPC_ID --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=sg_c3ops_dev_bastion}]" --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr 0.0.0.0/0

# Web Server Security Group
WEB_SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MyWebSecurityGroup --description "Web Security group" --vpc-id $VPC_ID --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=sg_c3ops_dev_web}]" --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 22 --cidr $VPC_CIDR
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 80 --cidr $VPC_CIDR
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr $VPC_CIDR
```

---

### 10. **Launching EC2 Instances**
A Windows instance is launched in the public subnet as a bastion host, and a Linux web server is launched in the private subnet.

```bash
# Windows Bastion Host
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances --image-id $WINDOWS_AMI_ID --instance-type $INSTANCE_TYPE --key-name $KEY_NAME --subnet-id $PUBLIC_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=c3ops_dev_bastion}]" --query 'Instances[0].InstanceId' --output text)

# Linux Web Server
LINUX_INSTANCE_ID=$(aws ec2 run-instances --image-id $LINUX_AMI_ID --instance-type $INSTANCE_TYPE --key-name $KEY_NAME --subnet-id $PRIVATE_SUBNET_1 --security-group-ids $WEB_SECURITY_GROUP_ID --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=c3ops_dev_web}]" --query 'Instances[0].InstanceId' --output text)
```

---

### 11. **Allocating and Associating Elastic IP for Bastion Host**
An Elastic IP is allocated and associated with the bastion host for remote access.

```bash
Bastion_EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
aws ec2 associate-address --instance-id $WINDOWS_INSTANCE_ID --allocation-id $Bastion_EIP
```

---

## QA Environment Variations

The QA environment script is similar to the development script but uses different CIDR blocks and tags to distinguish resources. The key differences include:

- **VPC CIDR**: `172.18.0.0/16`
- **Public Subnets**: `172.18.1.0/28` and `172.18.2.0/28`
- **Private Subnets**: `172.18.3.0/24` and `172.18.4.0/24`
- **Resource Tags**: Prefixed with `c3ops_qa_` instead of `c3ops_dev_`

---

## Importance of Bastion Servers

Bastion servers are critical for securely accessing instances in private subnets. They act as a gateway, allowing administrators to connect to private instances without exposing them directly to the internet. This setup enhances security by limiting direct access and providing a controlled entry point for management tasks.

Here is the full script for the Dev and QA environments: 

DEV:
```bash
#!/bin/bash

# Configure Variables
REGION="us-east-1"
AZ1="us-east-1a"
AZ2="us-east-1b"
VPC_CIDR="192.168.0.0/16"
PUBLIC_SUBNET_CIDR_1="192.168.1.0/28"
PUBLIC_SUBNET_CIDR_2="192.168.2.0/28"
PRIVATE_SUBNET_CIDR_1="192.168.3.0/24"
PRIVATE_SUBNET_CIDR_2="192.168.4.0/24"

# Create VPC
echo "Creating VPC..."
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION --query 'Vpc.VpcId' --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=c3ops_dev_vpc}]" --output text)
echo "VPC ID: $VPC_ID"

# Create Public Subnets
echo "Creating Public Subnets..."
PUBLIC_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_1 --availability-zone $AZ1 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_public_subnet_1}]" --query 'Subnet.SubnetId' --output text)
PUBLIC_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_2 --availability-zone $AZ2 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_public_subnet_2}]" --query 'Subnet.SubnetId' --output text)
echo "Public Subnet 1 ID: $PUBLIC_SUBNET_1"
echo "Public Subnet 2 ID: $PUBLIC_SUBNET_2"

# Create Private Subnets
echo "Creating Private Subnets..."
PRIVATE_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_1 --availability-zone $AZ1 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_private_subnet_1}]" --query 'Subnet.SubnetId' --output text)
PRIVATE_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_2 --availability-zone $AZ2 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_dev_private_subnet_2}]" --query 'Subnet.SubnetId' --output text)
echo "Private Subnet 1 ID: $PRIVATE_SUBNET_1"
echo "Private Subnet 2 ID: $PRIVATE_SUBNET_2"

# Enable auto-assign public IPv4 addresses for Public Subnets
echo "Enabling auto-assign public IPv4 addresses for Public Subnets..."
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_2 --map-public-ip-on-launch

# Create Internet Gateway and attach it to VPC
echo "Creating Internet Gateway..."
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=c3ops_dev_igw}]" --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID
echo "Internet Gateway ID: $INTERNET_GATEWAY_ID"

# Create NAT Gateway
echo "Creating Elastic IP for NAT Gateway..."
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Allocation ID: $ALLOCATION_ID"

echo "Creating NAT Gateway in Public Subnet 1..."
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --tag-specifications "ResourceType=natgateway,Tags=[{Key=Name,Value=c3ops_dev_nat_gateway}]" --query 'NatGateway.NatGatewayId' --output text)
echo "NAT Gateway ID: $NAT_GATEWAY_ID"

# Wait for NAT Gateway to be available
echo "Waiting for NAT Gateway to become available..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID

# Create Route Tables
echo "Creating Route Tables..."
PUBLIC_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=c3ops_dev_public_rtb}]" --query 'RouteTable.RouteTableId' --output text)
PRIVATE_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=c3ops_dev_private_rtb}]" --query 'RouteTable.RouteTableId' --output text)
echo "Public Route Table ID: $PUBLIC_ROUTE_TABLE_ID"
echo "Private Route Table ID: $PRIVATE_ROUTE_TABLE_ID"

# Create Routes
echo "Creating route to Internet Gateway in Public Route Table..."
aws ec2 create-route --route-table-id $PUBLIC_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $INTERNET_GATEWAY_ID

echo "Creating route to NAT Gateway in Private Route Table..."
aws ec2 create-route --route-table-id $PRIVATE_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

# Associate Subnets with Route Tables
echo "Associating Public Subnets with Public Route Table..."
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_1
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_2

echo "Associating Private Subnets with Private Route Table..."
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_1
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_2

# Create Bastion Security Group
echo "Creating Security Group..."
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "Bastion Security group" --vpc-id $VPC_ID --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=sg_c3ops_dev_bastion}]" --query 'GroupId' --output text)
echo "Security Group ID: $SECURITY_GROUP_ID"

# Add rules to Bastion Security Group
echo "Adding rules to Security Group..."
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr 0.0.0.0/0

# Create Web Server Security Group
echo "Creating Security Group..."
WEB_SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MyWebSecurityGroup --description "Web Security group" --vpc-id $VPC_ID --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=sg_c3ops_dev_web}]" --query 'GroupId' --output text)
echo "Security Group ID: $WEB_SECURITY_GROUP_ID"

# Add rules to Web Server Security Group
echo "Adding rules to Security Group..."
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 22 --cidr $VPC_CIDR
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 80 --cidr $VPC_CIDR
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr $VPC_CIDR

Allocate Elastic IP for Bastion Host
echo "Allocating Elastic IP for Bastion Host..."
Bastion_EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Elastic IP Allocation ID: $Bastion_EIP"

WINDOWS_AMI_ID="ami-0888db1202897905c"  # Replace with a valid Windows AMI ID
LINUX_AMI_ID="ami-0866a3c8686eaeeba"      # Replace with a valid Linux AMI ID
INSTANCE_TYPE="t2.micro"                # Specify instance type
KEY_NAME="linux_sshkeys"              # Replace with your key pair name

# Launch Windows Bastion Host in Public Subnet
echo "Launching Windows Bastion Host in Public Subnet 1..."
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id "ami-0888db1202897905c" \
  --instance-type "t2.micro" \
  --key-name "linux_sshkeys" \
  --subnet-id $PUBLIC_SUBNET_1 \
  --security-group-ids $SECURITY_GROUP_ID \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=c3ops_dev_bastion}]" \
  --query 'Instances[0].InstanceId' --output text)
echo "Windows Bastion Host Instance ID: $WINDOWS_INSTANCE_ID"

# Wait for Bastion Host to be running
echo "Waiting for Bastion Host to be running..."
aws ec2 wait instance-running --instance-ids $WINDOWS_INSTANCE_ID
echo "Bastion Host is now running."

# Allocate Elastic IP for Bastion Host
echo "Allocating Elastic IP for Bastion Host..."
Bastion_EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Elastic IP Allocation ID: $Bastion_EIP"

# Associate Elastic IP with Bastion Host
echo "Associating Elastic IP with Bastion Host..."
aws ec2 associate-address --instance-id $WINDOWS_INSTANCE_ID --allocation-id $Bastion_EIP
echo "Elastic IP successfully associated with Bastion Host."

# Launch Linux Web Server in Private Subnet
echo "Launching Linux Web Server in Private Subnet 1..."
LINUX_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id "ami-0866a3c8686eaeeba" \
  --instance-type "t2.micro" \
  --key-name "linux_sshkeys" \
  --subnet-id $PRIVATE_SUBNET_1 \
  --security-group-ids $WEB_SECURITY_GROUP_ID \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=c3ops_dev_web}]" \
  --query 'Instances[0].InstanceId' --output text)
echo "Linux Web Server Instance ID: $LINUX_INSTANCE_ID"

echo "Infrastructure setup is complete!"
```
QA:
```bash
#!/bin/bash

# Configure Variables
REGION="us-east-1"
AZ1="us-east-1a"
AZ2="us-east-1b"
VPC_CIDR="172.18.0.0/16"
PUBLIC_SUBNET_CIDR_1="172.18.1.0/28"
PUBLIC_SUBNET_CIDR_2="172.18.2.0/28"
PRIVATE_SUBNET_CIDR_1="172.18.3.0/24"
PRIVATE_SUBNET_CIDR_2="172.18.4.0/24"

# Create VPC
echo "Creating VPC..."
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION --query 'Vpc.VpcId' --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=c3ops_qa_vpc}]" --output text)
echo "VPC ID: $VPC_ID"

# Create Public Subnets
echo "Creating Public Subnets..."
PUBLIC_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_1 --availability-zone $AZ1 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_qa_public_subnet_1}]" --query 'Subnet.SubnetId' --output text)
PUBLIC_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_2 --availability-zone $AZ2 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_qa_public_subnet_2}]" --query 'Subnet.SubnetId' --output text)
echo "Public Subnet 1 ID: $PUBLIC_SUBNET_1"
echo "Public Subnet 2 ID: $PUBLIC_SUBNET_2"

# Create Private Subnets
echo "Creating Private Subnets..."
PRIVATE_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_1 --availability-zone $AZ1 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_qa_private_subnet_1}]" --query 'Subnet.SubnetId' --output text)
PRIVATE_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_2 --availability-zone $AZ2 --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=c3ops_qa_private_subnet_2}]" --query 'Subnet.SubnetId' --output text)
echo "Private Subnet 1 ID: $PRIVATE_SUBNET_1"
echo "Private Subnet 2 ID: $PRIVATE_SUBNET_2"

# qEnable auto-assign public IPv4 addresses for Public Subnets
echo "Enabling auto-assign public IPv4 addresses for Public Subnets..."
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_1 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_2 --map-public-ip-on-launch

# Create Internet Gateway and attach it to VPC
echo "Creating Internet Gateway..."
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=c3ops_qa_igw}]" --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID
echo "Internet Gateway ID: $INTERNET_GATEWAY_ID"

# Create NAT Gateway
echo "Creating Elastic IP for NAT Gateway..."
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Allocation ID: $ALLOCATION_ID"

echo "Creating NAT Gateway in Public Subnet 1..."
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --tag-specifications "ResourceType=natgateway,Tags=[{Key=Name,Value=c3ops_qa__nat_gateway}]" --query 'NatGateway.NatGatewayId' --output text)
echo "NAT Gateway ID: $NAT_GATEWAY_ID"

# Wait for NAT Gateway to be available
echo "Waiting for NAT Gateway to become available..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID

# Create Route Tables
echo "Creating Route Tables..."
PUBLIC_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=c3ops_qa_public_rtb}]" --query 'RouteTable.RouteTableId' --output text)
PRIVATE_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=c3ops_qa_private_rtb}]" --query 'RouteTable.RouteTableId' --output text)
echo "Public Route Table ID: $PUBLIC_ROUTE_TABLE_ID"
echo "Private Route Table ID: $PRIVATE_ROUTE_TABLE_ID"

# Create Routes
echo "Creating route to Internet Gateway in Public Route Table..."
aws ec2 create-route --route-table-id $PUBLIC_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $INTERNET_GATEWAY_ID

echo "Creating route to NAT Gateway in Private Route Table..."
aws ec2 create-route --route-table-id $PRIVATE_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

# Associate Subnets with Route Tables
echo "Associating Public Subnets with Public Route Table..."
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_1
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_2

echo "Associating Private Subnets with Private Route Table..."
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_1
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_2

# Create Bastion Security Group
echo "Creating Security Group..."
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "Bastion Security group" --vpc-id $VPC_ID --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=sg_c3ops_qa_bastion}]" --query 'GroupId' --output text)
echo "Security Group ID: $SECURITY_GROUP_ID"

# Add rules to Bastion Security Group
echo "Adding rules to Security Group..."
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr 0.0.0.0/0

# Create Web Server Security Group
echo "Creating Security Group..."
WEB_SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MyWebSecurityGroup --description "Web Security group" --vpc-id $VPC_ID --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=sg_c3ops_qa_web}]" --query 'GroupId' --output text)
echo "Security Group ID: $WEB_SECURITY_GROUP_ID"

# Add rules to Web Server Security Group
echo "Adding rules to Security Group..."
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 22 --cidr $VPC_CIDR
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 80 --cidr $VPC_CIDR
aws ec2 authorize-security-group-ingress --group-id $WEB_SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr $VPC_CIDR

Allocate Elastic IP for Bastion Host
echo "Allocating Elastic IP for Bastion Host..."
Bastion_EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Elastic IP Allocation ID: $Bastion_EIP"

WINDOWS_AMI_ID="ami-0888db1202897905c"  # Replace with a valid Windows AMI ID
LINUX_AMI_ID="ami-0866a3c8686eaeeba"      # Replace with a valid Linux AMI ID
INSTANCE_TYPE="t2.micro"                # Specify instance type
KEY_NAME="linux_sshkeys"              # Replace with your key pair name

# Launch Windows Bastion Host in Public Subnet
echo "Launching Windows Bastion Host in Public Subnet 1..."
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id "ami-0888db1202897905c" \
  --instance-type "t2.micro" \
  --key-name "linux_sshkeys" \
  --subnet-id $PUBLIC_SUBNET_1 \
  --security-group-ids $SECURITY_GROUP_ID \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=c3ops_qa_bastion}]" \
  --query 'Instances[0].InstanceId' --output text)
echo "Windows Bastion Host Instance ID: $WINDOWS_INSTANCE_ID"

# Wait for Bastion Host to be running
echo "Waiting for Bastion Host to be running..."
aws ec2 wait instance-running --instance-ids $WINDOWS_INSTANCE_ID
echo "Bastion Host is now running."

# Allocate Elastic IP for Bastion Host
echo "Allocating Elastic IP for Bastion Host..."
Bastion_EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Elastic IP Allocation ID: $Bastion_EIP"

# Associate Elastic IP with Bastion Host
echo "Associating Elastic IP with Bastion Host..."
aws ec2 associate-address --instance-id $WINDOWS_INSTANCE_ID --allocation-id $Bastion_EIP
echo "Elastic IP successfully associated with Bastion Host."

# Launch Linux Web Server in Private Subnet
echo "Launching Linux Web Server in Private Subnet 1..."
LINUX_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id "ami-0866a3c8686eaeeba" \
  --instance-type "t2.micro" \
  --key-name "linux_sshkeys" \
  --subnet-id $PRIVATE_SUBNET_1 \
  --security-group-ids $WEB_SECURITY_GROUP_ID \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=c3ops_qa_web}]" \
  --query 'Instances[0].InstanceId' --output text)
echo "Linux Web Server Instance ID: $LINUX_INSTANCE_ID"

echo "Infrastructure setup is complete!"
```
## Starting the Development Environment with Debugging

To ensure a smooth setup of the development environment, it's beneficial to run the script with debugging enabled. By executing the script with `bash -x`, you can observe each command as it is executed, providing greater visibility into the script's operations and helping to identify any potential issues.

To start the development environment with debugging, use the following command:

```bash
bash -x c3ops_dev_vpc.sh
```

This command will execute the `c3ops_dev_vpc.sh` script, printing each command to the terminal as it runs. This detailed output is invaluable for troubleshooting and verifying that each step of the setup process is completed successfully.

Certainly! Here's a brief section you can include in your documentation to indicate successful deployment and reference a resource map:

---

## Successful Deployment

The development environment has been successfully deployed using the `c3ops_dev_vpc.sh` script. All components, including VPCs, subnets, NAT gateways, security groups, and EC2 instances, have been provisioned and configured as intended.

## Resource Map

To visualize the architecture and layout of the deployed resources, refer to the resource map below. This diagram provides a comprehensive overview of the VPC setup, illustrating the relationships and connections between the various components.

![Resource Map of VPC](/images/resource-map.png)

## Modifying IAM Role for Session Manager Access

To enhance security and simplify access management, you can modify the IAM role attached to your EC2 instance to include permissions for AWS Systems Manager Session Manager. This allows you to manage instances without needing a bastion host.

### Steps to Modify IAM Role

1. **Select the IAM Role**: Navigate to the EC2 console, select your instance, and choose "Modify IAM role."
2. **Attach the Role**: Choose an IAM role that includes the `AmazonSSMManagedInstanceCore` policy, such as `cb-ssm`.
3. **Update the Role**: Click "Update IAM role" to apply the changes.

![Modify_IAM](/images/modify-iam-role.png)

### Benefits of Using Session Manager

- **Secure Access**: Session Manager provides secure, auditable access to your instances without opening inbound ports or managing SSH keys.
- **No Bastion Host Needed**: By using Session Manager, you eliminate the need for a bastion host, reducing complexity and potential security risks.
- **Centralized Management**: Manage multiple instances from a single interface, streamlining operations and improving efficiency. 

This modern approach enhances security and simplifies the management of your AWS infrastructure.

## Connecting to the Bastion Host via RDP

Follow these steps to connect to your Windows bastion host using Remote Desktop Protocol (RDP).

### Step 1: Retrieve the RDP File
- **Download the RDP File**: Access the AWS Management Console, navigate to your EC2 instance, and download the RDP file.
- 
  - ![Download RDP File](/images/rdp-download.png)

### Step 2: Decrypt the Windows Password
- **Get the Password**: Use your private key to decrypt the Windows administrator password.
- 
  - ![Decrypt Password](/images/decrypt-password.png)

### Step 3: Open the RDP Client
- **Launch the RDP Client**: Open the downloaded RDP file with your RDP client.
- 
  - ![Open RDP Client](/images/open_rdp_client.png)

### Step 4: Enter Credentials
- **Enter Username and Password**: Use "Administrator" as the username and the decrypted password.
- 
  - ![Enter Credentials](/images/enter_credentials.png)

### Step 5: Accept Security Certificate
- **Accept the Certificate**: If prompted, accept the security certificate to proceed.

  - ![Accept Certificate](/images/accept_certificate.png)

### Step 6: Access the Bastion Host
- **Connect to the Desktop**: You should now see the Windows desktop of your bastion host.
  - ![Access Bastion Host](/images/access_bastion_host.png)
## Downloading and Installing Git Bash on the Bastion Host

To manage your repositories and perform version control tasks, you can download and install Git Bash on your Windows bastion host. Follow these steps:

### Step 1: Access the Git Website
- Open a web browser on your bastion host and navigate to the Git download page: [https://git-scm.com/downloads](https://git-scm.com/downloads).

### Step 2: Download Git for Windows
- Click on the "Download for Windows" button to start downloading the Git installer.
  - ![Download Git for Windows](/images/Git_Download_Page.png)

### Step 3: Run the Installer
- Once the download is complete, open the downloaded `.exe` file to start the installation process.
  - ![Run Git Installer](/images/Git_Installer_Setup.png)

### Step 4: Follow the Installation Wizard
- Proceed through the installation wizard by clicking "Next" and accepting the default settings. This will install Git Bash on your system.

### Step 5: Complete the Installation
- After the installation is complete, you can launch Git Bash from the Start menu or desktop shortcut.

By following these steps, you can successfully install Git Bash on your bastion host, enabling you to manage your code repositories directly from the server.

## Copying the Key Pair and SSHing into the Web Server

To securely connect from the bastion host to your private Linux web server, you need to copy your key pair and use it for SSH access. Follow these steps:

### Step 1: Copy the Key Pair

1. **Open Git Bash or Command Prompt**: On your bastion host, open Git Bash or a command prompt.
2. **Navigate to the Download Folder**: Ensure you are in the directory where you want to store the key pair, typically the Downloads folder.
   ```bash
   cd ~/Downloads
   ```
3. **Create and Edit the Key File**: Use `vi` to create a new file and paste your key pair.
   ```bash
   vi linux_sshkeys
   ```
   - Press `i` to enter insert mode.
   - Paste the contents of your `linux_sshkeys.pem` file.
   - Press `Esc` to exit insert mode.
   - Type `:wq` and press `Enter` to save and quit.

4. **Rename the Key File**: Ensure the file has the correct `.pem` extension.
   ```bash
   mv linux_sshkeys linux_sshkeys.pem
   ```

### Step 2: SSH into the Web Server

1. **Use SSH to Connect**: With the key pair in place, use SSH to connect to your private Ubuntu web server from the bastion host.
   ```bash
   ssh -i linux_sshkeys.pem ubuntu@192.168.3.76
   ```

By following these steps, you can securely access your web server from the bastion host, allowing you to manage and configure your server as needed.
