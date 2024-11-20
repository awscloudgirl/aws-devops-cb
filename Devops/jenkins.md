# Setting Up Jenkins on AWS

### Overview

This documentation guides you through setting up Jenkins on an AWS EC2 instance for use as a Continuous Integration (CI) server. Jenkins is an open-source automation server that enables developers to build, test, and deploy their software.

### Prerequisites

- An AWS account.
- Basic knowledge of Linux commands and AWS services.

### Step 1: Launch an EC2 Instance

1. **Log in to AWS Management Console** and navigate to the EC2 dashboard.
2. **Launch a new EC2 instance** using an Ubuntu Server image.
3. **Choose an instance type** that meets your resource needs (e.g., t2.medium).
4. **Configure the instance** with a public IP and make sure you can access it via SSH.
5. **Set up security groups** to allow traffic on port 8080 (for Jenkins UI) and port 22 (for SSH).

### Step 2: Set Timezone

Ensure the server's timezone is set correctly:

```bash
timedatectl
timedatectl list-timezones
timedatectl list-timezones | grep Europe/London
timedatectl set-timezone Europe/London
```

### Step 3: Configure Hostnames

Set the hostname for easier server identification:

```bash
hostnamectl set-hostname jenkins.cloudbinary.io
vi /etc/hosts
```
Append ```172.31.91.64  jenkins.cloudbinary.io``` at the end of the file:
```bash
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

172.31.91.64  jenkins.cloudbinary.io
```

### Step 4: Install Utility Software

Install necessary utilities for managing the system:

```bash
sudo apt update
sudo apt install wget curl unzip tree git vim net-tools -y
```

### Step 5: Install Java

Jenkins requires Java to run:

```bash
sudo apt update
sudo apt-get install openjdk-17-jdk -y
```

Backup the environment file and set Java environment variables:

```bash
cp -pvr /etc/environment "/etc/environment_$(date +%F_%R)"
echo "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/" >> /etc/environment
source /etc/environment
```

### Step 6: Install Jenkins

Add the Jenkins repository and install Jenkins:

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

### Step 7: Unlock Jenkins

Retrieve the initial admin password to unlock Jenkins for the first time:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Set login credentials and access the dashboard:

![](/images/jenkins_admin.png)

### Step 8: Access Jenkins

- **Access Jenkins** by navigating to `http://<Your_EC2_Public_IP>:8080`.
- **Unlock Jenkins** using the initial admin password.
- **Install suggested plugins** and set up an admin user.

### Step 9: Configure Jenkins

Set up your build jobs and configure Jenkins to meet your project requirements.

### Conclusion

By following these steps, you have successfully set up Jenkins on an AWS EC2 instance. You can now use this server as a CI tool to automate your build, test, and deployment processes.

For further customization and advanced configurations, refer to the [official Jenkins documentation](https://www.jenkins.io/doc/).


