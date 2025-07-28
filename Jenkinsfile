pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' 
        ECR_REPO = '917186637165.dkr.ecr.ap-south-1.amazonaws.com/capstone/petclinic'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "${ECR_REPO}"  
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
                echo "🐳 Building Docker image ${DOCKER_IMAGE}:${IMAGE_TAG}..."
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} -f Dockerfile ."
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
                sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
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
