Project Overview

Goal: Automate the deployment of a containerized web application using a CI/CD pipeline.

Tools and Technologies:

    Jenkins: For orchestrating the CI/CD pipeline.
    Git: For source code management.
    Docker: To containerize the web application.
    Azure: For hosting Jenkins and deploying the containerized application.

High-Level Steps:

    Set up infrastructure in Azure.
    Prepare a simple containerized web application.
    Install and configure Jenkins on an Azure VM.
    Create a Git repository to host the application code.
    Configure Jenkins for CI/CD with Git and Docker.
    Deploy the containerized application to an Azure container service (e.g., Azure Container Instances or Azure Kubernetes Service)

Step 1: Set up Infrastructure in Azure

You need infrastructure to host Jenkins and the web application.

    Create an Azure Virtual Machine (VM) for Jenkins:
        Log in to the Azure portal.
        Go to "Virtual Machines" and create a new VM:
            Image: Ubuntu Server 20.04 LTS.
            Size: Choose a size with at least 2 CPUs and 4 GB RAM.
            Networking: Allow inbound ports for SSH (22) and HTTP (80).
        Download the SSH private key to connect to the VM.

 Install Required Software on the VM: SSH into the VM and install the necessary software:

# Update the system
sudo apt update && sudo apt upgrade -y

# Install Java (required for Jenkins)
sudo apt install -y openjdk-11-jdk

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install -y jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

Access Jenkins Dashboard:

    Open a browser and go to http://<VM_Public_IP>:8080.
    Retrieve the Jenkins initial admin password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Complete the setup wizard and install recommended plugins.
