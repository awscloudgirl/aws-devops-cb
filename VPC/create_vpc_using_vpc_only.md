# AWS VPC Setup Documentation

## Introduction
This documentation guides you through setting up a VPC in AWS, complete with public and private subnets, an internet gateway, and a NAT gateway. It ensures your resources within the VPC can securely access the internet while keeping private resources shielded from direct exposure.

We'll configure:
1. A VPC
2. Public and private subnets
3. Route tables for directing traffic
4. Internet Gateway (for public access)
5. NAT Gateway (for secure internet access from private subnets)

The final VPC architecture looks like this:

![VPC Diagram](/images/vpc_diagram.png)

---

## Table of Contents

1. [Create a VPC](#step-1-create-a-vpc)
2. [Create Subnets](#step-2-create-subnets)
3. [Create Route Tables](#step-3-create-route-tables)
4. [Associate Subnets with Route Tables](#step-4-associate-subnets-with-route-tables)
5. [Create an Internet Gateway](#step-5-create-an-internet-gateway)
6. [Attach Internet Gateway to the VPC](#step-6-attach-internet-gateway-to-the-vpc)
7. [Modify Public Route Table to Use Internet Gateway](#step-7-modify-public-route-table-to-use-internet-gateway)
8. [Verify Public Subnet Associations](#step-8-verify-public-subnet-associations)
9. [Create a NAT Gateway](#step-9-create-a-nat-gateway)
10. [Modify Private Route Table to Use NAT Gateway](#step-10-modify-private-route-table-to-use-nat-gateway)
11. [Verify Private Subnet Associations](#step-11-verify-private-subnet-associations)

---

## Step 1: Create a VPC
1. In the AWS Management Console, navigate to **VPC**.
2. Click on **Create VPC** and provide a name for your VPC, e.g., `c3ops_qa`.
3. Set the IP range for your VPC (CIDR block), for example, `192.168.0.0/16`.
4. Click **Create**.

## Step 2: Create Subnets
1. In the VPC dashboard, click on **Subnets** and then **Create subnet**.
2. Define your public and private subnets:
   - **Public Subnet 1 (us-east-1a)**: `192.168.1.0/28`
   - **Public Subnet 2 (us-east-1b)**: `192.168.2.0/28`
   - **Private Subnet 1 (us-east-1b)**: `192.168.3.0/24`
   - **Private Subnet 2 (us-east-1b)**: `192.168.4.0/24`
3. Click **Create subnet**.

![Create Subnets](/images/create_subnets.png)

## Step 3: Create Route Tables
1. Go to **Route Tables** in the VPC dashboard and click **Create route table**.
2. Create a public route table for your public subnets:
   - **Name**: `c3ops_qa_public_rtb`
   - **VPC**: `c3ops_qa`
3. Repeat the process to create a private route table for your private subnets:
   - **Name**: `c3ops_qa_private_rtb`
   - **VPC**: `c3ops_qa`
4. Click **Create route table**.

![Create Route Table](/images/create_route_table.png)

## Step 4: Associate Subnets with Route Tables
1. For the public route table (`c3ops_qa_public_rtb`), associate the public subnets:
   - **Public Subnet 1** and **Public Subnet 2**.
2. For the private route table (`c3ops_qa_private_rtb`), associate the private subnets:
   - **Private Subnet 1** and **Private Subnet 2**.
3. Click **Save**.

![Associate Subnets](/images/associate_subnets.png)

## Step 5: Create an Internet Gateway
1. In the **Internet Gateways** section, click **Create internet gateway**.
2. Provide a name for the internet gateway (e.g., `c3ops_qa_igw`).
3. Click **Create**.

![Create Internet Gateway](/images/create_internet_gateway.png)

## Step 6: Attach Internet Gateway to the VPC
1. Select the internet gateway (`c3ops_qa_igw`) and click **Actions**.
2. Choose **Attach to VPC**.
3. Select the VPC (`c3ops_qa`) and click **Attach**.

![Attach Internet Gateway](/images/attach_internet_gateway.png)

## Step 7: Modify Public Route Table to Use Internet Gateway
1. Select the public route table (`c3ops_qa_public_rtb`) and click **Edit routes**.
2. Add the following route:
   - **Destination**: `0.0.0.0/0`
   - **Target**: Select the internet gateway (`c3ops_qa_igw`).
3. Click **Save changes**.

![Edit Public Route Table](/images/edit_public_route_table.png)

## Step 8: Verify Public Subnet Associations
1. Verify that both public subnets (`c3ops_qa_public_subnet_1` and `c3ops_qa_public_subnet_2`) are associated with the public route table (`c3ops_qa_public_rtb`).

![Verify Public Subnet Associations](/images/verify_public_subnet_associations.png)

## Step 9: Create a NAT Gateway
1. Navigate to **NAT Gateways** and click **Create NAT Gateway**.
2. Provide a name for the NAT gateway (e.g., `c3ops_qa_nat_gw`).
3. Choose the public subnet to create the NAT gateway in (e.g., `c3ops_qa_public_subnet_1`).
4. Allocate an Elastic IP for the NAT gateway and click **Create**.

![Create NAT Gateway](/images/create_nat_gateway.png)

## Step 10: Modify Private Route Table to Use NAT Gateway
1. Select the private route table (`c3ops_qa_private_rtb`) and click **Edit routes**.
2. Add the following route:
   - **Destination**: `0.0.0.0/0`
   - **Target**: Select the NAT gateway (`c3ops_qa_nat_gw`).
3. Click **Save changes**.

![Edit Private Route Table](/images/edit_private_route_table.png)

## Step 11: Verify Private Subnet Associations
1. Verify that both private subnets (`c3ops_qa_private_subnet_1` and `c3ops_qa_private_subnet_2`) are associated with the private route table (`c3ops_qa_private_rtb`).

![Verify Private Subnet Associations](/images/verify_private_subnet_associations.png)

---

This completes the setup for your VPC, with public and private subnets, an internet gateway, and a NAT gateway. Public subnets are internet-facing, and private subnets access the internet securely via the NAT gateway.
