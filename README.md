# Project Overview

Goal: Automate the deployment of a containerized web application using a CI/CD pipeline.

## Tools and Technologies:

1. Jenkins: For orchestrating the CI/CD pipeline.
2. Git: For source code management.
3. Docker: To containerize the web application.
4. Azure: For hosting Jenkins and deploying the containerized application.

## High-Level Steps:

1. Set up infrastructure in Azure.
2. Prepare a simple containerized web application.
3.  Install and configure Jenkins on an Azure VM.
4. Create a Git repository to host the application code.
5. Configure Jenkins for CI/CD with Git and Docker.
6. Deploy the containerized application to an Azure container service (e.g., Azure Container Instances or Azure Kubernetes Service).

### Step 1: Set up Infrastructure in Azure

You need infrastructure to host Jenkins and the web application.

1.Create an Azure Virtual Machine (VM) for Jenkins:
 - Log in to the Azure portal.
 - Go to "Virtual Machines" and create a new VM:
 -          Image: Ubuntu Server 20.04 LTS.
 -         Size: Choose a size with at least 2 CPUs and 4 GB RAM.
 - Networking: Allow inbound ports for SSH (22) and HTTP (80).
 - Download the SSH private key to connect to the VM.

 Install Required Software on the VM: SSH into the VM and install the necessary software:

Update the system
'''
	sudo apt update && sudo apt upgrade -y
'''
	Install Java (required for Jenkins)
'''
	sudo apt install -y openjdk-11-jdk
'''
Install Docker
'''
	sudo apt install -y docker.io
	sudo systemctl start docker
	sudo systemctl enable docker
	sudo usermod -aG docker $USER
'''

Install Jenkins
'''
	wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
	sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
	sudo apt update
	sudo apt install -y jenkins
'''

 Start Jenkins
'''
	sudo systemctl start jenkins
	sudo systemctl enable jenkins
'''
Access Jenkins Dashboard:

    Open a browser and go to http://<VM_Public_IP>:8080.
    Retrieve the Jenkins initial admin password:
    '''
        sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    '''
        Complete the setup wizard and install recommended plugins.

### Step 2: Prepare a Simple Containerized Web Application

You’ll use a simple web application to deploy through your pipeline.

- Create the Web Application:
- Clone or create a basic Node.js app (or any other web app).
- Example code (app.js):
'''
    const express = require("express");
    const app = express();
    app.get("/", (req, res) => res.send("Hello from the CI/CD Pipeline!"));
    app.listen(3000, () => console.log("Server running on port 3000"));
'''

- Create a Dockerfile to Containerize the Application:

- Example Dockerfile:
'''
        FROM node:16
        WORKDIR /app
        COPY package*.json ./
        RUN npm install
        COPY . .
        EXPOSE 3000
        CMD ["node", "app.js"]
'''
-  Push the Application to a Git Repository:
-  Create a Git repository (e.g., on GitHub or Azure Repos).
-  Push the code (including the Dockerfile) to the repository.

### Step 3: Configure Jenkins with Git and Docker

Now, you’ll configure Jenkins to automate the build and deployment process.

- Install Jenkins Plugins:
- Go to Jenkins Dashboard > Manage Jenkins > Plugins.
- Install the following plugins:
- Git: For source code integration.
- Docker Pipeline: To use Docker commands in the pipeline.
- Pipeline: For Jenkinsfile support.

   Create a New Pipeline Job in Jenkins:
   Go to Dashboard > New Item and select Pipeline.
   Name the job and configure it.

   Write a Jenkinsfile:
   Add the following Jenkinsfile to your Git repository:
'''
        pipeline {
            agent any
            stages {
                stage('Clone Repository') {
                    steps {
                        git 'https://github.com/your-repo-url.git'
                    }
                }
                stage('Build Docker Image') {
                    steps {
                        script {
                            docker.build('your-app:latest')
                        }
                    }
                }
                stage('Run Docker Container') {
                    steps {
                        sh 'docker stop your-app || true'
                        sh 'docker rm your-app || true'
                        sh 'docker run -d -p 80:3000 --name your-app your-app:latest'
                    }
                }
            }
        }

        Replace https://github.com/your-repo-url.git with your repository URL.
'''
 - Test the Pipeline:
 - Trigger the pipeline in Jenkins.
 - Verify that Jenkins clones the code, builds the Docker image, and runs the container.

### Step 4: Deploy to Azure

Instead of running the container locally, deploy it to Azure.

 Deploy to Azure Container Instances (ACI):
 Create a container instance in Azure:

    az container create \
        --resource-group <resource-group-name> \
        --name <container-name> \
        --image <image-name> \
        --cpu 1 --memory 1 \
        --dns-name-label <dns-label> \
        --ports 80

Push the Docker Image to Azure Container Registry (ACR):

Log in to Azure and set up ACR:

    az acr login --name <acr-name>
    docker tag your-app:latest <acr-name>.azurecr.io/your-app:latest
    docker push <acr-name>.azurecr.io/your-app:latest

Update the Jenkinsfile for ACI Deployment:

Add a deployment stage to the Jenkinsfile:

        stage('Deploy to Azure') {
            steps {
                sh 'az container create --resource-group <resource-group-name> --name <container-name> --image <acr-name>.azurecr.io/your-app:latest --dns-name-label <dns-label> --ports 80'
            }
        }

###  5: Validate and Monitor

 Access the Application:
 Access the application using the Azure Container Instance’s public URL: http://<dns-label>.azurecontainer.io.

 Set Up Monitoring:
 Use Azure Monitor or Application Insights to monitor the application and pipeline.
