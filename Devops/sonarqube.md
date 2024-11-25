# Setting Up SonarQube on AWS EC2

This guide walks through the steps to deploy SonarQube on an AWS EC2 instance using Docker. SonarQube is a powerful static code analysis tool used for continuous inspection of code quality. By following these steps, you will set up a dedicated SonarQube server in a Docker container, which allows for scalable and isolated analysis of your projects.

## Launch EC2 Instance

- **Instance Name:** `sonarqube.cloudbinary.io`
- **Operating System:** Ubuntu
- **Instance Type:** t2.medium

## Configuration Steps

### Set Time & Date

```bash
sudo timedatectl set-timezone Europe/London
```

### Setup Hostname

```bash
sudo hostnamectl set-hostname "sonarqube.cloudbinary.io"
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

### Install Required Utilities

For Ubuntu:

```bash
sudo apt-get install vim curl elinks unzip wget tree git -y
```

For CentOS or similar:

```bash
sudo yum install vim curl elinks unzip wget tree git net-tools -y
```

### Install Docker

For Ubuntu:

```bash
sudo apt-get install docker.io -y
```

For CentOS or similar:

```bash
sudo yum install docker -y
```

Verify Docker installation:

```bash
docker --version
docker volume ls
docker network ls
```

### Configure Docker Permissions

Add the ubuntu user to the Docker group and set permissions:

```bash
sudo usermod -aG docker ubuntu
sudo chmod 777 /var/run/docker.sock
```

## SonarQube Installation

### Pull Docker Image for SonarQube

```bash
docker pull sonarqube
```

### Setup Docker Volumes

Create and inspect volumes:

```bash
docker volume create sonarqube-conf
docker volume inspect sonarqube-conf
docker volume create sonarqube-data
docker volume create sonarqube-logs
docker volume create sonarqube-extensions
```

### Prepare SonarQube Directory

```bash
sudo mkdir /sonarqube
sudo ln -s /var/lib/docker/volumes/sonarqube-conf/_data /sonarqube/conf
sudo ln -s /var/lib/docker/volumes/sonarqube-data/_data /sonarqube/data
sudo ln -s /var/lib/docker/volumes/sonarqube-logs/_data /sonarqube/logs
sudo ln -s /var/lib/docker/volumes/sonarqube-extensions/_data /sonarqube/extensions
ls -lrta /sonarqube/
```

### Run SonarQube Container

```bash
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 -v sonarqube-conf:/sonarqube/conf -v sonarqube-data:/sonarqube/data -v sonarqube-logs:/sonarqube/logs -v sonarqube-extensions:/sonarqube/extensions sonarqube
docker ps
```

Check the container logs:

```bash
docker logs [containerID]
```

### Configure Security Settings

Add inbound rules to the EC2 instance:

- **HTTP:** Port 80
- **Custom:** Port 9000
![sonar](/images/sonarqube-security)

### Verification

Access SonarQube by entering the private IP of the EC2 instance in your browser, followed by `:9000` to reach the SonarQube interface.
