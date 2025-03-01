# CI/CD Pipeline for Static Website Deployment using Jenkins, GitHub Webhooks, and Apache2 on AWS EC2

## Overview
This project sets up a CI/CD pipeline to automate the deployment of a static website using Jenkins, GitHub Webhooks, and Apache2 on an AWS EC2 instance. The pipeline ensures that every code change pushed to GitHub is automatically built and deployed to the Apache2 web server, making the website live without manual intervention.

## Project Workflow
1. **Developer commits changes** to the GitHub repository.
2. **GitHub Webhook triggers Jenkins** upon a new push event.
3. **Jenkins pulls the latest code** from GitHub.
4. **Jenkins builds the website** in `/var/lib/jenkins/workspace/website`.
5. **Apache2 serves the website** from `/var/lib/jenkins/workspace/website`.

## Prerequisites
- AWS EC2 instance (Ubuntu preferred)
- Apache2 installed and configured
- Jenkins installed and configured
- GitHub repository for the static website
- GitHub Webhook configured to trigger Jenkins
- A user group with permissions for both Jenkins and Apache2

## Setup Instructions

### 1. Setup Apache2 on AWS EC2
```sh
sudo apt update -y
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

### 2. Install Jenkins on AWS EC2
```sh
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
echo "deb https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update -y
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### 3. Configure Apache2 to Serve Jenkins Workspace
Modify Apache’s configuration file to change the document root:
```sh
sudo vim /etc/apache2/sites-available/000-default.conf
```
Update the `DocumentRoot` to:
```apache
DocumentRoot /var/lib/jenkins/workspace/website
```
Restart Apache2:
```sh
sudo systemctl restart apache2
```

### 4. Configure User Group for Permissions
Create a new group and add both Jenkins and Apache2 users:
```sh
sudo groupadd webadmin
sudo usermod -aG webadmin jenkins
sudo usermod -aG webadmin www-data
sudo chown -R jenkins:webadmin /var/lib/jenkins/workspace/website
sudo chmod -R 775 /var/lib/jenkins/workspace/website
```
Restart services:
```sh
sudo systemctl restart jenkins
sudo systemctl restart apache2
```

### 5. Setup GitHub Webhook
- Navigate to your GitHub repository → *Settings* → *Webhooks*
- Click *Add webhook*
- Enter `http://your-ec2-instance-ip:8080/github-webhook/`
- Select *Just the push event* and click *Add webhook*

### 6. Create Jenkins Job (Freestyle)
- Configure *Source Code Management* → *Git* → Enter repository URL
- Configure *Build Triggers* → Select *GitHub hook trigger for GITScm polling*
- Configure *Build Steps*:
  ```sh
  # Pull the latest code
  git pull origin main
  ```
- Save and Build

## Testing the Setup
- Push a change to your GitHub repository
- Jenkins should automatically trigger a build
- Apache2 should serve the updated website from `/var/lib/jenkins/workspace/website`

## Conclusion
This setup ensures a fully automated CI/CD pipeline for deploying a static website, reducing manual deployment efforts and improving efficiency.

## Author
[Afaque Ahmed](https://www.linkedin.com/in/afaque490)  
GitHub: [afaque490](https://github.com/afaque490)

