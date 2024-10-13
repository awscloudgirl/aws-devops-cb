## Shell Script for AWS Infrastructure Setup and Cleanup

This documentation explains a shell script that automates the setup of AWS infrastructure, including VPC, subnets, NAT gateways, security groups, and EC2 instances. It also provides a cleanup script to delete the resources created by the original script.

---

## What is a Shell Script?

A **shell script** is a file containing a series of commands written for the command-line interpreter (shell). It allows you to automate repetitive tasks, like setting up infrastructure, by executing these commands sequentially. In this case, the script is written in **Bash** and uses the AWS CLI to create various AWS resources.

---

## Breakdown of the Script

This section breaks down the main parts of the shell script.

### 1. **Configuring Variables**
Variables are defined for AWS region, availability zones (AZ), CIDR blocks for VPC and subnets, and other AWS-specific details such as instance type and AMI IDs.

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
The script creates a VPC with the CIDR block specified in the variables.

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION --tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=MyVpc}] --query 'Vpc.VpcId' --output text)
```

---

### 3. **Creating Subnets**
Both public and private subnets are created within the specified VPC, each in different availability zones.

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
An internet gateway is created and attached to the VPC to enable internet access for resources in the public subnets.

```bash
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID
```

---

### 5. **Creating a NAT Gateway**
A NAT gateway is created in a public subnet to allow instances in private subnets to access the internet. An Elastic IP is also allocated for the NAT gateway.

```bash
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --query 'NatGateway.NatGatewayId' --output text)
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID
```

---

### 6. **Creating Route Tables and Routes**
Two route tables are created: one for public subnets (with a route to the internet gateway) and one for private subnets (with a route to the NAT gateway).

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
Subnets are associated with their respective route tables.

```bash
# Public Subnets with Public Route Table
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_1
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_2

# Private Subnets with Private Route Table
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_1
aws ec2 associate-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID --subnet-id $PRIVATE_SUBNET_2
```

---

### 8. **Creating Security Group**
A security group is created and configured with rules to allow traffic for SSH, HTTP, and RDP.

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

# Attach Internet Gateway to the VPC
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID

# Create NAT Gateway
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET_1 --allocation-id $ALLOCATION_ID --query 'NatGateway.NatGatewayId' --output text)

# Wait for NAT Gateway to become available
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

# Create Security Group and Add Rules
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id $VPC_ID --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 3389 --cidr 0.0.0.0/0

WINDOWS_AMI_ID="ami-0888db1202897905c"  # Replace with a valid Windows AMI ID
LINUX_AMI_ID="ami-0fff1b9a61dec8a5f"      # Replace with a valid Linux AMI ID
INSTANCE_TYPE="t2.micro"                # Specify instance type
KEY_NAME="linux_sshkeys"              # Replace with your key pair name

# Launch EC2 Instances
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances --image-id ami-0888db1202897905c --instance-type t2.micro --key-name linux_sshkeys --subnet-id $PUBLIC_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --query 'Instances[0].InstanceId' --output text)
LINUX_INSTANCE_ID=$(aws ec2 run-instances --image-id ami-0fff1b9a61dec8a5f --instance-type t2.micro --key-name linux_sshkeys --subnet-id $PRIVATE_SUBNET_1 --security-group-ids $SECURITY_GROUP_ID --query 'Instances[0].InstanceId' --output text)

echo "Infrastructure setup is complete!"
```

---

### Script to Delete Everything Created by the Original Script

The following script deletes all resources created by the initial script, including EC2 instances, security groups, route tables, subnets, NAT gateway, and VPC.

```bash
#!/bin/bash

# Terminate EC2 Instances
aws ec2 terminate-instances --instance-ids $WINDOWS_INSTANCE_ID $LINUX_INSTANCE_ID
aws ec2 wait instance-terminated --instance-ids $WINDOWS_INSTANCE_ID $LINUX_INSTANCE_ID

# Delete Security Group
aws ec2 delete-security-group --group-id $SECURITY_GROUP_ID

# Delete Route Tables
aws ec2 delete-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID
aws ec2 delete-route-table --route-table-id $PRIVATE_ROUTE_TABLE_ID

# Delete NAT Gateway and release Elastic IP
aws ec2 delete-nat-gateway --nat-gateway-id $NAT_GATEWAY_ID
aws ec2 release-address --allocation-id $ALLOCATION_ID

# Detach and Delete Internet Gateway
aws ec2 detach-internet-gateway --internet-gateway-id $INTERNET_GATEWAY_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $INTERNET_GATEWAY_ID

# Delete Subnets
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET_1
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET_2
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET_1
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET_2

# Delete VPC
aws ec2 delete-vpc --vpc-id $VPC_ID

echo "All resources created by the script have been deleted."
```

### Checking Resources Created and Deleted

To verify that the resources were created or deleted, use the following AWS CLI commands:

- **Check VPC**: `aws ec2 describe-vpcs --vpc-ids $VPC_ID`
- **Check Subnets**: `aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID"`
- **Check Instances**: `aws ec2 describe-instances --instance-ids $WINDOWS_INSTANCE_ID $LINUX_INSTANCE_ID`
- **Check Security Groups**: `aws ec2 describe-security-groups --group-ids $SECURITY_GROUP_ID`

If resources have been deleted, these commands should return no results or an error message indicating that the resources no longer exist.
