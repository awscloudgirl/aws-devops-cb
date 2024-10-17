# Automating AWS VPC Infrastructure Setup

This document outlines a shell script that automates the setup of AWS infrastructure, including VPC, subnets, NAT gateways, security groups, and EC2 instances.

---

## What is a Shell Script?

A **shell script** is a file containing a series of commands for a command-line interpreter (shell). It automates tasks by executing these commands sequentially. In this case, the script is written in **Bash** and utilizes the AWS CLI to provision AWS resources.

---

## Breakdown of the Script

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
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION --tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=MyVpc}] --query 'Vpc.VpcId' --output text)
```

---

### 3. **Creating Subnets**
Both public and private subnets are created within the VPC, each in different availability zones.

```bash
# Public Subnets
PUBLIC_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_1 --availability-zone $AZ1 --query 'Subnet.SubnetId' --output text)
PUBLIC_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_2 --availability-zone $AZ2 --query 'Subnet.SubnetId' --output text)

# Private Subnets
PRIVATE_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_1 --availability-zone $AZ1 --query 'Subnet.SubnetId' --output text)
PRIVATE_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_2 --availability-zone $AZ2 --query 'Subnet.SubnetId' --output text)
```

---

### 4. **Creating and Attaching an Internet Gateway**
An internet gateway is created and attached to the VPC to enable internet access for public subnets.

```bash
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID
```

---

### 5. **Creating a NAT Gateway**
A NAT gateway is created in a public subnet, allowing private subnet instances to access the internet. An Elastic IP is also allocated for the NAT gateway.

```bash
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --query 'NatGateway.NatGatewayId' --output text)
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID
```

---

### 6. **Creating Route Tables and Routes**
Separate route tables are created for public and private subnets. Routes to the internet gateway and NAT gateway are defined accordingly.

```bash
# Route Tables
PUBLIC_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
PRIVATE_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)

# Routes
aws ec2 create-route --route-table-id $PUBLIC_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $INTERNET_GATEWAY_ID
aws ec2 create-route --route-table-id $PRIVATE_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID
```

---

### 7. **Associating Subnets with Route Tables**
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

### 8. **Creating a Security Group**
A security group is created with rules to allow SSH, HTTP, and RDP traffic.

```bash
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id $VPC_ID --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr 0.0.0.0/0
```

---

### 9. **Launching EC2 Instances**
A Windows instance is launched in the public subnet, and a Linux web server is launched in the private subnet.

```bash
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances --image-id $WINDOWS_AMI_ID --instance-type $INSTANCE_TYPE --key-name $KEY_NAME --subnet-id $PUBLIC_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --query 'Instances[0].InstanceId' --output text)

LINUX_INSTANCE_ID=$(aws ec2 run-instances --image-id $LINUX_AMI_ID --instance-type $INSTANCE_TYPE --key-name $KEY_NAME --subnet-id $PRIVATE_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --query 'Instances[0].InstanceId' --output text)
```

---

### Full Infrastructure Setup Script

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
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyVpc}]' --query 'Vpc.VpcId' --output text)

# Create Subnets
PUBLIC_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_1 --availability-zone $AZ1 --query 'Subnet.SubnetId' --output text)
PUBLIC_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PUBLIC_SUBNET_CIDR_2 --availability-zone $AZ2 --query 'Subnet.SubnetId' --output text)
PRIVATE_SUBNET_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_1 --availability-zone $AZ1 --query 'Subnet.SubnetId' --output text)
PRIVATE_SUBNET_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $PRIVATE_SUBNET_CIDR_2 --availability-zone $AZ2 --query 'Subnet.SubnetId' --output text)

# Create Internet Gateway
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID

# Create NAT Gateway
ALLOCATION_ID=$(aws ec2 allocate-address

 --domain vpc --query 'AllocationId' --output text)
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --query 'NatGateway.NatGatewayId' --output text)
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID

# Create Route Tables and Routes
PUBLIC_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
PRIVATE_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)

aws ec2 create-route --route-table-id $PUBLIC_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $INTERNET_GATEWAY_ID
aws ec2 create-route --route-table-id $PRIVATE_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GATEWAY_ID

# Associate Subnets with Route Tables
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_1
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_2
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_1
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_2

# Create Security Group
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id $VPC_ID --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr 0.0.0.0/0

# Launch EC2 Instances
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances --image-id $WINDOWS_AMI_ID --instance-type $INSTANCE_TYPE --key-name $KEY_NAME --subnet-id $PUBLIC_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --query 'Instances[0].InstanceId' --output text)
LINUX_INSTANCE_ID=$(aws ec2 run-instances --image-id $LINUX_AMI_ID --instance-type $INSTANCE_TYPE --key-name $KEY_NAME --subnet-id $PRIVATE_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --query 'Instances[0].InstanceId' --output text)

echo "Infrastructure setup is complete."
```

--- 

### Conclusion

This script automates the creation of an AWS infrastructure setup that includes a VPC, public and private subnets, an internet gateway, a NAT gateway, security groups, and EC2 instances. It is flexible and can be modified to suit various configurations depending on your infrastructure requirements.
