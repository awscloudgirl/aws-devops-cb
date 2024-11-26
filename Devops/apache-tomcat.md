# Setting Up Apache Tomcat on AWS EC2

## Introduction

Apache Tomcat is an open-source Java servlet container that acts as a web server and provides the environment to run Java code in web applications. It is widely used by developers due to its stability, reliability, and ease of use. This guide will walk you through the process of setting up Apache Tomcat on an AWS EC2 instance, configuring essential settings, and deploying a simple application.

## Prerequisites

- An AWS account
- Basic knowledge of Linux commands
- Familiarity with AWS EC2

## Steps

### Launch an EC2 Instance

- **Instance Name:** `dev.cloudbinary.io`
- **AMI:** Ubuntu
- **Instance Type:** t2.micro

### Configure the Server

#### Set Time & Date

```bash
sudo timedatectl set-timezone Europe/London
```

#### Setup Hostname

```bash
sudo hostnamectl set-hostname "jfrog.cloudbinary.io"
```

#### Update Hosts File

```bash
echo "`hostname -I` `hostname`" >> /etc/hosts
echo "`hostname -I | awk '{ print $1}'` `hostname`" >> /etc/hosts
cat /etc/hosts
```

#### Update the Repository

```bash
sudo apt-get update
```

#### Install Required Utilities

For Ubuntu:

```bash
sudo apt-get install vim curl elinks unzip wget tree git -y
```

#### Install Java

For Ubuntu:

```bash
sudo apt-get install openjdk-17-jdk -y
```

#### Set Environment Variables

```bash
echo "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/" >> /etc/environment
source /etc/environment
```

### Install and Configure Tomcat

#### Download and Extract Tomcat

```bash
cd /opt
sudo wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.97/bin/apache-tomcat-9.0.97.tar.gz
tar xvzf apache-tomcat-9.0.97.tar.gz
rm -f apache-tomcat-9.0.97.tar.gz
mv apache-tomcat-9.0.97/ tomcat
```

#### Start Tomcat

```bash
cd /tomcat/bin
./startup.sh
```

#### Open Port 8080 in Security Settings

### Configure Tomcat Users

#### Backup Configuration File

```bash
cp -pvr /opt/tomcat/conf/tomcat-users.xml "/opt/tomcat/conf/tomcat-users.xml_$(date +%F_%R)"
```

#### Edit Users and Roles

```bash
sed -i '$d' /opt/tomcat/conf/tomcat-users.xml
echo '<role rolename="manager-gui"/>' >> /opt/tomcat/conf/tomcat-users.xml
echo '<role rolename="manager-script"/>' >> /opt/tomcat/conf/tomcat-users.xml
# Add remaining roles and user
```

#### Adjust Manager and Host-Manager Contexts

```bash
vi /opt/tomcat/webapps/manager/META-INF/context.xml
# Add the line with allow="127....|.*"
vi /opt/tomcat/webapps/host-manager/META-INF/context.xml
# Add the line with allow="127....|.*"
```

### Deploy a Web Application

#### Manual Deployment via Manager App

- Navigate to the Manager App on your browser using the EC2 instance's public IP appended with `:8080`.
- Scroll to "WAR file to deploy" section.
- Choose the WAR file you want to deploy and click 'Deploy'.

### Verify Installation

- Access your Tomcat server by entering the public IP followed by `:8080` in your browser.
- Check the functionality by accessing the deployed applications.

## Conclusion

This setup guide provides a comprehensive approach to deploying Apache Tomcat on an AWS EC2 instance, ensuring your applications are served in a secure and efficient environment.
