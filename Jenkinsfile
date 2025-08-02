pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' 
        ECR_REPO = '917186637165.dkr.ecr.ap-south-1.amazonaws.com/capstone/petclinic'
        IMAGE_TAG = "spring-petclinic-${BUILD_NUMBER}"
        DOCKER_IMAGE = "${ECR_REPO}:${IMAGE_TAG}"
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
                echo "🐳 Building Docker image ${DOCKER_IMAGE}..."
                sh "docker build -t ${DOCKER_IMAGE} -f Dockerfile ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                echo "🔐 Logging into AWS ECR..."
                sh '''
                    aws --version
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo "📦 Pushing Docker image to ECR..."
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to ECS') {
            steps {
                echo "🚀 Triggering ECS deployment of image ${DOCKER_IMAGE}..."
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
