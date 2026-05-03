pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '827306792924.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO = 'my-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_SERVER = '10.0.2.111'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vishaldubey260396/my-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG} .'
            }
        }
        stage('Push to ECR') {
            steps {
                sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                sh 'docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}'
            }
        }
        stage('Deploy to App Server') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@${APP_SERVER} "
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 827306792924.dkr.ecr.us-east-1.amazonaws.com
                        docker pull 827306792924.dkr.ecr.us-east-1.amazonaws.com/my-app:${BUILD_NUMBER}
                        docker stop my-app || true
                        docker rm my-app || true
                        docker run -d --name my-app -p 80:5000 827306792924.dkr.ecr.us-east-1.amazonaws.com/my-app:${BUILD_NUMBER}
                    "
                '''
            }
        }
    }
}
