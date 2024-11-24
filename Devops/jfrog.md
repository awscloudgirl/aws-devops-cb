# JFrog Artifactory Setup on AWS

This guide details the steps to launch an AWS instance and configure JFrog Artifactory on Ubuntu.

## Launch AWS Instance

- **Instance Name:** `jfrog.cloudbinary.io`
- **OS:** Ubuntu
- **Instance Type:** t2.micro

## Initial Server Setup

### Set Timezone

```bash
sudo timedatectl set-timezone Europe/London
```

### Configure Hostname

```bash
sudo hostnamectl set-hostname "jfrog.cloudbinary.io"
```

### Update `/etc/hosts`

```bash
echo "`hostname -I` `hostname`" >> /etc/hosts
echo "`hostname -I | awk '{ print $1}'` `hostname`" >> /etc/hosts
```

### Update Repositories

```bash
sudo apt-get update
```

### Install Essential Utilities

For Ubuntu:

```bash
sudo apt-get install vim curl elinks unzip wget tree git -y
```

## Install Java

For Ubuntu:

```bash
sudo apt-get install openjdk-17-jdk -y
```

## Environment Setup

### Backup Environment File

```bash
sudo cp -pvr /etc/environment "/etc/environment_$(date +%F_%R)"
```

### Set JAVA_HOME

For Ubuntu:

```bash
echo "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/" >> /etc/environment
source /etc/environment
```

## Download and Install JFrog Artifactory

```bash
sudo wget https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/[RELEASE]/jfrog-artifactory-oss-[RELEASE]-linux.tar.gz
tar xvzf jfrog-artifactory-oss-[RELEASE]-linux.tar.gz 
mv artifactory-oss-7.98.8/ /opt/jfrog
```

### Configure JFROG_HOME

```bash
echo "JFROG_HOME=/opt/jfrog" >> /etc/environment
source /etc/environment
```

## Manage Artifactory Service

### Start JFrog Artifactory

```bash
cd /opt/jfrog/app/bin/
./artifactory.sh start
```

### Configure systemd Service

```bash
sudo vi /etc/systemd/system/artifactory.service
```

Insert the following configuration:

```plaintext
[Unit]
Description=JFrog Artifactory service
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

### Enable and Manage Service

```bash
systemctl daemon-reload
systemctl enable artifactory.service
systemctl start artifactory.service
systemctl status artifactory.service
```

## Cleanup

```bash
rm -rf /path/to/jfrog-artifactory-oss-[RELEASE]/
```

## Security Settings

Remember to adjust the security settings of your AWS instance as necessary to allow appropriate traffic.

```

