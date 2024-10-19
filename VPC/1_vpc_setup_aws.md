# Creating a VPC using AWS VPC and more

This documentation explains how to create a Virtual Private Cloud (VPC) in AWS using the **"VPC and More"** option under the **Resources to Create** setting. This setup will involve creating public and private subnets, NAT gateways, and configuring the availability zones for high availability. The guide uses both a visual diagram and step-by-step screenshots from the AWS console to assist with the setup.

## Table of Contents

- [VPC Overview](#vpc-overview)
- [Step-by-Step Instructions](#step-by-step-instructions)
  - [Step 1: Create VPC](#step-1-create-vpc)
  - [Step 2: Select Availability Zones](#step-2-select-availability-zones)
  - [Step 3: Configure NAT Gateway and DNS Options](#step-3-configure-nat-gateway-and-dns-options)
  - [Step 4: Customize Subnets](#step-4-customize-subnets)
- [Key Components](#key-components)

## VPC Overview

A **Virtual Private Cloud (VPC)** allows you to create an isolated section of the AWS Cloud where you can launch AWS resources in a virtual network. This network configuration can mirror a traditional network but offers a much more scalable and cost-effective way to host applications.

Below is a diagram that shows the architecture of the VPC we are creating:

![VPC Architecture](/images/create_vpc_vpcandmore.png)

## Step-by-Step Instructions

### Step 1: Create VPC

1. Navigate to the **VPC** service in the AWS Management Console.
2. Click on **Create VPC**.
3. Select the **"VPC and More"** option in the **Resources to Create** section to generate additional resources (subnets, route tables, and gateways).
4. Name the VPC (e.g., `c3ops_dev`).
5. Enter the IPv4 CIDR block for the VPC (e.g., `10.0.0.0/16`).

![VPC Creation Settings](/images/vpc_creation_settings.png)

For more details on VPC and CIDR block configuration, refer to the [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html).

### Step 2: Select Availability Zones

1. In the **Number of Availability Zones** section, select **2** for high availability.
2. Customize the Availability Zones by selecting **us-east-1a** and **us-east-1b**.

![Select Availability Zones](/images/select_availability_zones.png)

### Step 3: Configure NAT Gateway and DNS Options

1. Under the **NAT Gateways** section, select **In 1 AZ** to reduce costs while maintaining redundancy.
2. Enable **DNS hostnames** and **DNS resolution** to ensure that resources within the VPC can resolve domain names.

![NAT Gateway Configuration](/images/nat_gateway_dns_options.png)

For more details about NAT Gateways, visit the [AWS NAT Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html).

### Step 4: Customize Subnets

1. For public subnets, enter CIDR blocks for **us-east-1a** and **us-east-1b**:
   - Public Subnet 1 in us-east-1a: `10.0.1.0/28`
   - Public Subnet 2 in us-east-1b: `10.0.2.0/28`
2. For private subnets, enter CIDR blocks for **us-east-1a** and **us-east-1b**:
   - Private Subnet 1 in us-east-1a: `10.0.3.0/24`
   - Private Subnet 2 in us-east-1b: `10.0.4.0/24`

![Subnet Configuration](/images/customize_subnets.png)

More information on customizing subnets can be found in the [AWS Subnets Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html).

## Key Components

### VPC
A **VPC (Virtual Private Cloud)** is an isolated virtual network in the AWS cloud that allows you to manage resources securely and customize IP address ranges, subnets, route tables, and network gateways.

- AWS Documentation: [VPC Overview](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

### NAT Gateway
A **NAT Gateway** enables instances in a private subnet to connect to the internet or other AWS services while preventing the internet from initiating connections with those instances.

- AWS Documentation: [NAT Gateway Overview](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)

### Subnets
A **Subnet** is a range of IP addresses within a VPC. Public subnets allow access to the internet, while private subnets are used for backend services without direct internet access.

- AWS Documentation: [Subnet Overview](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
