## Automating Deployment of an E-Commerce Website

#### _Project Scenario_
A technology consulting firm is adapting a cloud architecture for its software applications. As a DevOps Engineer, my task is to design and implement a robust CI/CD pipeline using Jenkins to automate the deployment of a web application. The goal is to achieve continous integration, continous deployment, and ensure the scalability and reliability of the applications.

#### _Project Deliverables_
##### Documentation:

  - Detailed documentation for each Jenkins component setup.
  - Explanation of security measures implemented at each step.
##### Demonstration:

  - Live demonstration of the CI/CD pipeline.

#### _Project Components_

1. **_Jenkins Server Setup_** <br>

   **_Objective:_** Configure Jenkins serveer for CI/CD pipeline automation. <br>
   
   **_Steps:_**
   
   - Logged in to the AWS console and created an EC2 instance (Ubuntu 22.04) and allowed port "8080" and "80801" in the inbound rule of the _Security Group_ settings
   - Installed docker engine on the EC2 instance. Created this script [install-docker.sh](install-docker.sh) in the home directoy and executed the script to install the docker engine on the EC2 instance:
     
     `Modified the permission of the script`
     ```
     chmod u+x install-docker.sh
     ```
     `Ran the script`
     ```
     ./install-docker.sh
     ```
     `Added the current user to docker group`
     ```
     sudo usermod -aG docker $USER
     newgrp docker
     ```
   - Installed Java 21 (used by Jenkins)
     ```
     sudo apt install openjdk-21-jdk
     ```
   - Created a 'jenkins-network'
     ```
     docker network create jenkins-network
     ```
   - Pulled the Jenkins image and ran the container as root user with the below settings:
     
     `Created a '_jenkins_home_' volumme for jenkins data retention, and bind the required docker volumes for Jenkins to be able to run docker commands using the docker engine installed on the host (EC2). Added the Jenkins container to the 'jenkins_network' created`
     ```
     docker run -d -u root \
     -v jenkins_home:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v /usr/bin/docker:/usr/bin/docker \
     -p 8080:8080 -p 50000:50000 \
     --network jenkins-network \
     --name jenkins-server \
     jenkins/jenkins:lts
     ```
   - Entered the Jenkins container's bash, created a 'docker' group and added the 'root' to docker group.
     ```
     docker exec -it jenkins-server bash
     groupadd docker
     usermod -aG docker root
     newgrp docker
     ```
   - Logged on to Jenkins' WebGUI configuration page at `http://hostIP:8080` to configure the Jenkins server:
   - Entered the Jenkins container and retrieved the initial password at: **_`/var/jenkins_home/secrets/initialAdminPassword`_** to create a "First Admin User"
     ```
     docker exec -it jenkins-server bash
     cat /var/jenkins_home/secrets/initialAdminPassword
     ```
   - Installed the 'Suggested plugins'
   - Created a new pipeline job and configured the jenkins to use the [_Jenkinsfile_](https://github.com/isaac-adebayo/jenkins-ecommerce-deploy/blob/main/Jenkinsfile) available in this repository to build the [_ecommerce-website_](https://github.com/isaac-adebayo/jenkins-ecommerce-deploy/tree/main/ecommerce-website) file into nginx container using the [_Dockerfile_](https://github.com/isaac-adebayo/jenkins-ecommerce-deploy/blob/main/Dockerfile) in this repo, added Github login credential to the configuration and configured Jenkins to deploy the ecommerce website by running the container and also uploading the image to [_dockerhub_](https://hub.docker.com/repository/docker/isaacreg/jenkins-ecomm-nginx/general)

     
3. **_Source Code Management Repository Integration_** <br>

   **_Objective:_** Connect Jenkins to the version control system for source code management. <br>
   
   **_Steps:_** <br>

   - Logged in to the Jenkins web GUI and activated the _"`Githbub hook trigger for GitSCM polling`"_ in the _"`Configure`"_ tab of the pipeline
   
   - This Github [repository](https://github.com/isaac-adebayo/jenkins-ecommerce-deploy.git) specifically meant for this project ontains the _`Dockerfile`_ for building the _`jenkins-ecommerce-nginx`_ image. This repository is also shared by the developers of the ecommerce website to maintain versions of the website.
     
   - Added a new webhook to the Github's repository:  _`Settings -> Webhooks -> Add webhook`_. The Webhook was congigured with the below settings:
       - Payload URL = "http://hostIP:8080/github-webhook/"
       - Content type = "application/json"
       - And left other settings on default.

4. **_Regigistry Push of the Docker Image_**

   **_Objective:_** Automate the process of pushing the created docker image for the web application to a container registry such as docker hub.

   **_Steps:_**

   - Installed 'Docker pipeline' plugin in the Jenkins server: _`Manage Jenkins -> Plugins -> Available plugins`_
   - Created a global credential (Credentials ID: "docker-hub-credentials") used for loggin in to Docker Hub: _`Manage Jenkins -> Plugins -> Credentials`_
   - Updated the [pipeline](https://github.com/isaac-adebayo/jenkins-ecommerce-deploy/blob/main/Jenkinsfile) to include the steps to push the image to [dockerhub](https://hub.docker.com/repositories/isaacreg).

5. _**Testing**_

   The goal of this project being continous integration, continous deployment, and to ensure scalability and reliability of the web applications was achieved at the end of the project. The Jenkins server being set up was able to build, deploy and push container to docker hub registry whenever it detects changes in the Github repository hosting the web application artifacts through Github's Webhook. The web application was accessible at: http://hostIP:8081
   
