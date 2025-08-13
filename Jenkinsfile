pipeline {
    agent any
    environment {
        AWS_REGION   = 'ap-south-1'
        ECR_REPO     = '917186637165.dkr.ecr.ap-south-1.amazonaws.com/capstone/petclinic'
        IMAGE_TAG    = "spring-petclinic-${BUILD_NUMBER}"
        DOCKER_IMAGE = "${ECR_REPO}:${IMAGE_TAG}"
        LATEST_IMAGE = "${ECR_REPO}:latest"
        ECS_CLUSTER  = 'spring-petclinic-cluster'
        ECS_SERVICE  = 'spring-petclinic-service'
    }
    tools {
        maven 'Maven'
    }
    stages {
        stage('Build with Maven') {
            steps {
                echo ' Building Spring Petclinic with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                echo " Building Docker image and tagging with: ${IMAGE_TAG} and latest..."
                sh """
                    docker build -t ${DOCKER_IMAGE} -f Dockerfile .
                    docker tag ${DOCKER_IMAGE} ${LATEST_IMAGE}
                """
            }
        }
        stage('Login to AWS ECR') {
            steps {
                echo " Logging into AWS ECR..."
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }
        stage('Push to ECR (Both Tags)') {
            steps {
                echo " Pushing both tags (${IMAGE_TAG} and latest) to ECR..."
                sh """
                    docker push --all-tags ${ECR_REPO}
                """
            }
        }
        stage('Deploy to ECS') {
            steps {
                echo " Triggering ECS deployment of image: latest..."
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
    post {
        success {
            echo ' Pipeline completed successfully!'
        }
        failure {
            echo ' Pipeline failed.'
        }
    }
}
