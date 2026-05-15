pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-2'
        ACCOUNT_ID = '303238377772'
        ECR_REPO = 'myapp'

        IMAGE_TAG = "${BUILD_NUMBER}"

        ECS_CLUSTER = 'myapp-cluster'
        ECS_SERVICE = 'myapp-service'
    }

    stages {

        stage('Build Docker') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-creds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {

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

        stage('Push Docker Image') {
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
        docker stop myapp || true &&
        docker rm myapp || true &&
        docker pull 303238377772.dkr.ecr.us-east-2.amazonaws.com/myapp:${IMAGE_TAG} &&
        docker run -d -p 80:80 --name myapp 303238377772.dkr.ecr.us-east-2.amazonaws.com/myapp:${IMAGE_TAG}
        "
        '''
        }
    }
    }
}