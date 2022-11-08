pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    environment {
        registry = '740955001227.dkr.ecr.us-east-1.amazonaws.com/geo-devops-001'
        registryCredential = 'eks-cluster-user'
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
                    sh 'docker build -t geo-devops-001 .'
                    sh 'docker tag geo-devops-001:latest 740955001227.dkr.ecr.us-east-1.amazonaws.com/geo-devops-001:latest'
                    sh 'docker push 740955001227.dkr.ecr.us-east-1.amazonaws.com/geo-devops-001:latest'
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