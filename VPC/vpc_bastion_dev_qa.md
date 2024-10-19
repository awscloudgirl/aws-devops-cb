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
