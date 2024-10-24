# Setting Up an RDS Database

## Introduction
This documentation provides a step-by-step guide for creating an Amazon RDS (Relational Database Service) database using the MySQL engine. It covers the various options available during the setup process, ensuring a comprehensive understanding for users.

## Steps to Create an RDS Database

### 1. Access the RDS Dashboard
- Navigate to the **RDS Dashboard** in your AWS Management Console.
- Click on **Databases** in the left sidebar.

### 2. Create a Database
- Click on the **Create Database** button to initiate the setup process.
  ![Create_Database](/images/create_database.png) 

### 3. Choose the Creation Method
- **Standard**: Select this option to customize your database settings.
- **Easy Create**: A simplified version that uses default settings.

### 4. Select the Database Engine
- In the **Engine options**, choose **MySQL** from the list of available database engines.
  ![](/images/mysql_engine.png) 

### 5. Engine Version
- Select the desired engine version, such as **MySQL 8.0.39**.
  ![](/images/mysql_version.png) 

### 6. Storage Configuration
- **Storage Type**: Choose **General Purpose SSD (gp3)**.
- **Allocated Storage**: Set the allocated storage to **20 GiB** (minimum).
  ![](/images/storage_configuration.png) 

### 7. Additional Configuration
- **Initial Database Name**: Enter a name for your database (e.g., **finops**).
- **DB Parameter Group**: Select the default parameter group (e.g., **default.mysql8.0**).
- **Option Group**: Use the default option group (e.g., **default:mysql-8-0**).
  ![](/images/additional_configuration.png) 

### 8. Public Access
- Choose whether to allow public access:
  - **Yes**: Assigns a public IP address to the database.
  - **No**: Only allows access from resources within the VPC.
  ![](/images/public_access.png) 

### 9. VPC Security Group
- Select an existing VPC security group or create a new one to control access to your database.
  ![](/images/vpc_security_group.png) 

### 10. Estimated Monthly Costs
- Review the estimated monthly costs. The **Amazon RDS Free Tier** is available for 12 months, allowing you to use certain resources for free.
  ![](/images/estimated_costs.png) 

### 11. Connectivity Settings
- Choose whether to connect to an EC2 compute resource.
- Select the network type (IPv4 or dual-stack).
- Choose the appropriate VPC and DB subnet group.
  ![](/images/connectivity_settings.png) 

## Conclusion
By following these steps, you can successfully create a MySQL database in Amazon RDS. Ensure to review each option carefully to tailor the database to your specific needs. For further assistance, refer to the AWS documentation or community forums. 
