pipeline {

 agent any

 environment {     
     DOCKERHUB_CREDENTIALS= credentials('docker-hub-credentials')     
 }

 stages {

    stage('Connect To Github') {
        steps {
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/isaac-adebayo/jenkins-ecommerce-deploy.git']])
        }
    }
    stage('Build Docker Image, Dockerfile in Github repo') {
        steps {
             sh 'docker build -t jenkins-ecomm-nginx:${BUILD_ID} .'
             sh 'docker tag jenkins-ecomm-nginx:latest isaacreg/jenkins-ecomm-nginx:${BUILD_ID}'
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
                sh 'docker run --rm -itd --name ecommerce-container -p 8081:80 jenkins-ecomm-nginx:${BUILD_ID}'
        }
    }
   stage ('Push image to docker hub') {
         steps {
             sh 'docker logout'
             sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
             sh 'docker push isaacreg/jenkins-ecomm-nginx:${BUILD_ID}'
         }
     }
  }
}
