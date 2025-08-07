pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' 
        ECR_REPO = '917186637165.dkr.ecr.ap-south-1.amazonaws.com/capstone/petclinic'
        IMAGE_TAG = "spring-petclinic-${BUILD_NUMBER}"
        DOCKER_IMAGE = "${ECR_REPO}:${IMAGE_TAG}"
        LATEST_IMAGE = "${ECR_REPO}:latest"
        ECS_CLUSTER = 'spring-petclinic-cluster'
        ECS_SERVICE = 'spring-petclinic-service'
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Build with Maven') {
            steps {
                echo '🧱 Building Spring Petclinic with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image with tag: ${IMAGE_TAG}..."
                sh """
                    docker build -t ${DOCKER_IMAGE} -f Dockerfile .
                    docker tag ${DOCKER_IMAGE} ${LATEST_IMAGE}
                """
            }
        }

        stage('Login to AWS ECR') {
            steps {
                echo "🔐 Logging into AWS ECR..."
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo "📦 Pushing Docker images to ECR..."
                sh """
                    docker push ${DOCKER_IMAGE}
                    docker push ${LATEST_IMAGE}
                """
            }
        }

        stage('Verify Images in ECR') {
            steps {
                echo "🔍 Verifying both image tags in ECR..."
                sh """
                    aws ecr describe-images \
                        --repository-name capstone/petclinic \
                        --image-ids imageTag=${IMAGE_TAG} \
                        --region $AWS_REGION

                    aws ecr describe-images \
                        --repository-name capstone/petclinic \
                        --image-ids imageTag=latest \
                        --region $AWS_REGION
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                echo "🚀 Triggering ECS deployment of image ${LATEST_IMAGE}..."
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
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
