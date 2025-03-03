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
   - Logged in to the AWS console and created an EC2 instance and allowed port "8080" in the inbound rule of the _Security Group_ settings
   - Installed docker engine on the EC2 instance. Created this script [install-docker.sh](install-docker.sh) in the home directoy and executed the script to install the docker engine on the EC2 instance:
     
     `Modified the permission of the script`
     ```
     chmod u+x install-docker.sh
     ```
     `Ran the script`
     ```
     ./install-docker.sh
     ```
   - Created a 'jenkins_network'
     ```
     docker network create jenkins_network
     ```
   - Pulled the jenkins image and ran the container with the below settings:
     
     `Created a '_jenkins_home_' volumme for jenkins data retention, and bind the required docker volumes for jenkins to be able to run docker commands using the docker engine installed on the host (EC2). Added the jenkins container to the 'jenkins_network' created`
     ```
     docker run -d \
     -v jenkins_home:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v /usr/bin/docker:/usr/bin/docker \
     -p 8080:8080 -p 50000:50000 \
     --network jenkins-network \
     --name jenkins-server \
     jenkins/jenkins:lts
     ```
   - Logged on to jenkins' WebGUI configuration page at `http://hostIP:8080` to configure the jenkins server
   - Entered the jenkins container and retrieved the initial password at: **_`/var/jenkins_home/secrets/initialAdminPassword`_** to create a "First Admin User"
     ```
     docker exec -it jenkins-server bash
     ```
     ```
     cat /var/jenkins_home/secrets/initialAdminPassword
     ```
   - Installed the 'Suggested plugins'
   - Created a new pipeline job with _'Github hook trigger for GitSCM'_ enabled and with below pipelines:
     ```
     pipeline {

     agent any

     stages {
   
        stage('Connect To Github') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/isaac-adebayo/jenkins-scm.git']])
            }
        }
        stage('Build Docker Image, Dockerfile in Github repo') {
            steps {
             sh 'docker build -t jenkins-ecomm-nginx:1.0 .'
            }
        }
        stage('Run Docker Container') {
            steps {
             sh '''
                container_name="ecommerce-container"
                container_id=$(docker ps -q -a -f name=$container_name)
                if [ "$container_id" ]; then
                    echo "Container $container_name is already running. Stopping container..."
                    docker stop $container_id
                else
                    echo "Container $container_name is not running. Starting container..."
                fi
                '''
             sh 'docker run --rm -itd --name ecommerce-container -p 8081:80 jenkins-ecomm-nginx:1.0'
            }
          }
        }
      }
     ```

