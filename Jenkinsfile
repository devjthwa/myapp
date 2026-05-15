pipeline {
    agent any

    environment {
        AWS_REGION   = 'us-east-2'
        ACCOUNT_ID   = '303238377772'
        ECR_REPO     = 'myapp'

        IMAGE_TAG    = "${BUILD_NUMBER}"

        ECS_CLUSTER  = 'myapp'
        ECS_SERVICE  = 'myapp-service'

        CONTAINER_NAME = 'myapp'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/devjthwa/myapp.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin \
                    $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                docker push \
                $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh '''
                    aws ecs update-service \
                      --cluster $ECS_CLUSTER \
                      --service $ECS_SERVICE \
                      --force-new-deployment \
                      --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Wait for ECS Stability') {
            steps {

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh '''
                    aws ecs wait services-stable \
                      --cluster $ECS_CLUSTER \
                      --services $ECS_SERVICE \
                      --region $AWS_REGION
                    '''
                }
            }
        }
    }

    post {

        success {
            echo 'ECS Deployment SUCCESS 🚀'
        }

        failure {
            echo 'ECS Deployment FAILED ❌'
        }

        always {
            sh 'docker system prune -af || true'
        }
    }
}