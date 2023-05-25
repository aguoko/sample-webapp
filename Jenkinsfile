def remote = [:]
    remote.name = 'Docker-server'
    remote.host = '18.119.130.224'
    remote.user = 'ubuntu'
    remote.password = 'December2023#'
    remote.allowAnyHosts = true

pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID="011138670495"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="docker-class"
        IMAGE_TAG= "${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    stages {
          
          stage('Logging into AWS ECR') {
                     environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                         
                   }
                     steps {
                       script{
             
                         sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                }
                 
            }
        }
         
          stage('Building docker image') {
            
            steps{
              script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
        
          stage('Pushing to ECR') {
             environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                         
                   }
              steps{  
                script {
                    // withAWS(role: "dec-role", roleAccount: '111393898725') {
                    sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                    sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                    sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
                             }
                         }
                     }
        stage('Deploy to Docker-Server Via SSH') {
          steps{
      sshCommand remote: remote, command: "ls -lrt"
      sshCommand remote: remote, command: """aws ecr --profile docker-user get-login-password --region us-east-2 | docker login --username AWS --password-stdin 011138670495.dkr.ecr.us-east-2.amazonaws.com"""
      sshCommand remote: remote, command: "docker pull 011138670495.dkr.ecr.us-east-2.amazonaws.com/docker-class:4"
      sshCommand remote: remote, command: "docker run -d -p 9090:80 --name webapp 011138670495.dkr.ecr.us-east-2.amazonaws.com/docker-class:4"
      }
      }  


    }
}