# JFrog Artifactory Setup Instructions

JFrog Artifactory is a highly versatile artifact repository manager that simplifies the management of binary resources across software development environments. As an integral part of DevOps toolchains, it enhances the efficiency of continuous integration and delivery processes by providing a robust platform for versioning and storing artifacts like Docker containers, Java archives, and more. The following documentation guides you through the process of setting up JFrog Artifactory on an AWS instance. This setup involves configuring the server, installing necessary software, and ensuring that the system is optimized for secure and efficient artifact management.

## AWS Launch Instance

- **Instance Name:** `jfrog.cloudbinary.io`
- **Operating System:** Ubuntu
- **Instance Type:** t2.medium

## Configuration Steps

### Set Time & Date

```bash
sudo timedatectl set-timezone Europe/London
```

### Setup Hostname

```bash
sudo hostnamectl set-hostname "jfrog.cloudbinary.io"
```

### Configure Hostname in Hosts File

```bash
echo "`hostname -I` `hostname`" >> /etc/hosts
echo "`hostname -I | awk '{ print $1}'` `hostname`" >> /etc/hosts
```

### Update the Repository

```bash
sudo apt-get update
```

### Install Required Utility Software for Ubuntu

```bash
sudo apt-get install vim curl elinks unzip wget tree git -y
```

### Download and Install Java 17 for Ubuntu

```bash
sudo apt-get install openjdk-17-jdk -y
```

### Backup the Environment File

```bash
sudo cp -pvr /etc/environment "/etc/environment_$(date +%F_%R)"
```

### Create JAVA_HOME Environment Variable for Ubuntu

```bash
echo "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/" >> /etc/environment
source /etc/environment
```

### Download JFROG Software

```bash
sudo wget https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/[RELEASE]/jfrog-artifactory-oss-[RELEASE]-linux.tar.gz
```

### Extract the Tar File

```bash
tar xvzf jfrog-artifactory-oss-[RELEASE]-linux.tar.gz
```

### Remove the Tar File

```bash
rm -f jfrog-artifactory-oss-[RELEASE]-linux.tar.gz
```

### Move the Tar File to /opt/jfrog

```bash
mv artifactory-oss-7.98.8/ /opt/jfrog
```

### Set JFROG_HOME Environment Variable

```bash
echo "JFROG_HOME=/opt/jfrog" >> /etc/environment
cat /etc/environment
```

### Start JFROG Artifactory

```bash
cd /opt/jfrog/app/bin/
./artifactory.sh status
./artifactory.sh start
```

### Configure INIT Scripts for JFrog Artifactory

```bash
sudo vi /etc/systemd/system/artifactory.service
```

Insert the following configuration into the file:

```ini
[Unit]
Description=JFrog artifactory service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/jfrog/app/bin/artifactory.sh start
ExecStop=/opt/jfrog/app/bin/artifactory.sh stop
User=root
Group=root 
Restart=always

[Install]
WantedBy=multi-user.target
```

### Enable and Check Status of JFrog Artifactory

```bash
systemctl enable artifactory.service
systemctl status artifactory.service
```

### Modify Instance Security Settings

Update the inbound rules as necessary to secure the service.

![jfrog](/images/jfrog-security.png)

### Manage JFrog Service

Commands to manage the Artifactory service:

```bash
systemctl daemon-reload
sudo systemctl status artifactory.service
sudo systemctl stop artifactory.service
sudo systemctl start artifactory.service
sudo systemctl restart artifactory.service
sudo systemctl enable artifactory.service
```

### Remove Directory

```bash
rm -rf jfrog/
```

### Re-download and Extract JFrog Software (Specific Version)

```bash
sudo wget https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/7.9.0/jfrog-artifactory-oss-7.9.0-linux.tar.gz
tar xvzf jfrog-artifactory-oss-7.9.0-linux.tar.gz
rm -rf jfrog-artifactory-oss-7.9.0-linux.tar.gz
mv artifactory-oss-7.9.0/ jfrog
```
