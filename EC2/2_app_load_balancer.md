# Setting Up Web Servers on AWS EC2

This documentation provides an overview of setting up two web servers, WebServer1 and WebServer2, on Amazon EC2. The user data scripts automate the installation and configuration of Apache web servers.

## Introduction to User Data

User data scripts are used to automate the configuration of EC2 instances during launch. These scripts can install software, configure settings, and perform other tasks automatically.

## WebServer1 Setup

1. **Launch an EC2 Instance**
   - Go to the EC2 Dashboard and click on "Launch Instance".
   - Name your instance `WebServer1`.
![](/images/webserver1-name.png)

2. **Configure Instance Details**
   - Choose the instance type (e.g., `t2.micro` for free tier).
   - Select the appropriate VPC and subnet.
![](/images/webserver1-network.png)

3. **Add Storage**
   - Configure the storage as needed (default is usually sufficient).

4. **Add Tags**
   - Add any necessary tags for identification.

5. **Configure Security Group**
   - Create a new security group allowing SSH (port 22) and HTTP (port 80) access.
![](/images/webserver1-security.png)

6. **Add User Data**
   - Use the following script to set up Apache and deploy your website:
     ```bash
     #!/bin/bash
     sudo apt-get update
     sudo apt-get install apache2 -y
     sudo systemctl enable apache2
     sudo systemctl restart apache2
     cd /var/www/html
     echo "Server-1" > index.html
     ```
![](/images/webserver1-userdata.png)

7. **Review and Launch**
   - Review your settings and launch the instance.
   - Select an existing key pair or create a new one for SSH access.

## WebServer2 Setup

1. **Launch an EC2 Instance**
   - Go to the EC2 Dashboard and click on "Launch Instance".
   - Name your instance `WebServer2`.
![](/images/webserver2-name.png)

2. **Configure Instance Details**
   - Choose the instance type (e.g., `t2.micro` for free tier).
   - Select the appropriate VPC and subnet.
![](/images/webserver2-network.png)

3. **Add Storage**
   - Configure the storage as needed (default is usually sufficient).

4. **Add Tags**
   - Add any necessary tags for identification.

5. **Configure Security Group**
   - Create a new security group allowing SSH (port 22) and HTTP (port 80) access.
![](/images/webserver2-security.png)

6. **Add User Data**
   - Use the following script to set up Apache:
     ```bash
     #!/bin/bash
     sudo apt-get update
     sudo apt-get install apache2 -y
     sudo systemctl enable apache2
     sudo systemctl restart apache2
     cd /var/www/html
     echo "Server-2" > index.html
     ```
![](/images/webserver2-userdata.png)

7. **Review and Launch**
   - Review your settings and launch the instance.
   - Select an existing key pair or create a new one for SSH access.

## Conclusion

By following these steps, you will have two web servers running on AWS EC2, each configured to serve web content using Apache. Ensure that your security groups are properly configured to allow traffic to your instances.

# Creating an Application Load Balancer (ALB) on AWS

An Application Load Balancer (ALB) operates at the application layer (Layer 7) and provides advanced routing capabilities, including path-based and host-based routing. It is ideal for web applications and microservices.

## Steps to Create an Application Load Balancer

1. **Navigate to Load Balancers**
   - Go to the EC2 Dashboard and select "Load Balancers" from the left-hand menu.
   - Click on "Create Load Balancer".

2. **Select Load Balancer Type**
   - Choose "Application Load Balancer".
![](/images/alb-selection.png)

3. **Configure Basic Settings**
   - **Name**: Enter a unique name for your ALB (e.g., `alb`).
   - **Scheme**: Choose "Internet-facing" for public access or "Internal" for private access.
   - **IP Address Type**: Select "IPv4" or "Dualstack" for both IPv4 and IPv6.
![](/images/alb-basic-config.png)

4. **Network Mapping**
   - Select the VPC where your targets are located.
   - Choose at least two Availability Zones for high availability.
![](/images/alb-network-mapping.png)

5. **Configure Security Groups**
   - Assign a security group that allows inbound traffic on the necessary ports (e.g., HTTP:80).
![](/images/alb-security-groups.png)

6. **Configure Listeners and Routing**
   - Add a listener for HTTP on port 80.
   - Click "Create target group" to set up routing.
![](/images/alb-listeners-routing.png)

7. **Create Target Group**
   - **Target Group Name**: Enter a name (e.g., `tg80`).
   - **Protocol**: HTTP
   - **Port**: 80
   - **IP Address Type**: IPv4
   - **VPC**: Select the appropriate VPC.
![](/images/alb-target-group.png)

8. **Configure Health Checks**
   - Set up health checks to monitor the health of your targets.
   - **Protocol**: HTTP
   - **Path**: `/` (or another path to check)
   - **Port**: 80
![](/images/alb-health-checks.png)

9. **Register Targets**
   - Select the instances you want to include in the target group.
   - Ensure they are in the running state and associated with the correct security group.
![](/images/alb-register-targets.png)

10. **Review and Create**
    - Review all settings to ensure they are correct.
    - Click "Create Load Balancer" to finalize the setup.
![](/images/alb-create.png)

## Conclusion

By following these steps, you will have successfully created an Application Load Balancer on AWS, capable of distributing incoming traffic across multiple targets to ensure high availability and reliability.

To test the functionality of your Application Load Balancer (ALB) with the DNS name `alb-1285027674.us-east-1.elb.amazonaws.com`, you can perform a simple connectivity test. Open a web browser and enter the ALB's DNS name in the address bar. If the ALB is correctly configured and the target instances are healthy, you should see the web page served by your instances. This confirms that the ALB is successfully distributing traffic to the registered targets, ensuring high availability and reliability for your application.
This documentation provides an overview of setting up two web servers, WebServer1 and WebServer2, on Amazon EC2. The user data scripts automate the installation and configuration of Apache web servers.

