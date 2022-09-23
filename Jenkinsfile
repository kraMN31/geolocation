pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    environment {
        registry = '740955001227.dkr.ecr.us-east-1.amazonaws.com/eks-ecr-repo'
        registryCredential = 'eks-ecr-repo'
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
                    sh 'docker build -t eks-ecr-repo .'
                    sh 'docker tag eks-ecr-repo:latest 740955001227.dkr.ecr.us-east-1.amazonaws.com/eks-ecr-repo:latest'
                    sh 'docker push 740955001227.dkr.ecr.us-east-1.amazonaws.com/eks-ecr-repo:latest'
                }
            }
        }
        //deploy the image that is in ECR to our EKS cluster
        stage ("Kube Deploy") {
            steps {
                withKubeConfig([credentialsId: 'eks_credential', serverUrl: '']) {
                 sh "kubectl apply -f eks_deploy_from_ecr.yaml"
                }
            }
        }
    }
}