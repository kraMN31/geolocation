pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    environment {
        registry = '740955001227.dkr.ecr.us-east-1.amazonaws.com/emax_geo_ecr_repo'
        registryCredential = 'jenkins-ecr'
        dockerimage = ''
  }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/kraMN31/geolocation.git'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 740955001227.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker tag emax_geo_ecr_repo:latest 740955001227.dkr.ecr.us-east-1.amazonaws.com/emax_geo_ecr_repo:latest'
                    sh 'docker push 740955001227.dkr.ecr.us-east-1.amazonaws.com/emax_geo_ecr_repo:latest'
                }
            }
        }
    }
}