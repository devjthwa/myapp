pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-2'
        ACCOUNT_ID = '303238377772'
        ECR_REPO = 'myapp'

        // BEST PRACTICE: use build number OR git commit
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/devjthwa/myapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@172.31.15.122 "
                
                # stop old container safely
                docker stop myapp || true
                docker rm myapp || true

                # pull latest build
                docker pull $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

                # run new container
                docker run -d -p 80:80 --name myapp \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                "
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment SUCCESS 🚀'
        }
        failure {
            echo 'Deployment FAILED ❌'
        }
    }
}