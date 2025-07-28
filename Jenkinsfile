pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' 
        ECR_REPO = 'YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/spring-petclinic'
        IMAGE_TAG = "${BUILD_NUMBER}"
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
                echo "🐳 Building Docker image..."
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
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
                sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
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
