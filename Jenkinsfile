pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_PUBLIC_URI = 'public.ecr.aws/m9w1w7v4/app:latest'
        DEPLOYMENT_FILE = 'calculator-deployment.yaml'
        SERVICE_FILE = 'calculator-service.yaml'
        REPO_URL = 'https://github.com/bhargavpr99-sudo/calculator-app.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Lint Dockerfile') {
            steps {
                echo "Linting Dockerfile..."
                script {
                    // Assuming hadolint installed on Jenkins agent
                    sh "hadolint Dockerfile || true"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "sudo docker build -t app ."
            }
        }

        stage('Login to Public ECR') {
            steps {
                echo "Logging into Public ECR..."
                sh """
                    aws ecr-public get-login-password --region $AWS_REGION | \
                    sudo docker login --username AWS --password-stdin public.ecr.aws
                """
            }
        }

        stage('Tag & Push to Public ECR') {
            steps {
                echo "Tagging and pushing image..."
                sh """
                    sudo docker tag app:latest $ECR_PUBLIC_URI
                    sudo docker push $ECR_PUBLIC_URI
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                script {
                    // Update deployment YAML image
                    sh "sed -i 's|image: calculator:latest|image: $ECR_PUBLIC_URI|g' $DEPLOYMENT_FILE"
                    
                    // Scale replicas to 3
                    sh "kubectl apply -f $DEPLOYMENT_FILE"
                    sh "kubectl scale deployment calculator-deployment --replicas=3"

                    // Apply service
                    sh "kubectl apply -f $SERVICE_FILE"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
            // Optionally: send email or Slack notification
        }
        failure {
            echo "Pipeline failed!"
            // Optionally: send email or Slack notification
        }
    }
}
