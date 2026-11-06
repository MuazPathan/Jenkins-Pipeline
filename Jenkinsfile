pipeline {
    agent any

    environment {
        APP_NAME       = "jenkins-demo-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        CONTAINER_NAME = "demo-app-container"
        HOST_PORT      = "3000"
        CONTAINER_PORT = "3000"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git repository...'
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Executing automated test suite...'
                bat 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${APP_NAME}:${IMAGE_TAG}..."
                bat """
                    docker build -t %APP_NAME%:%IMAGE_TAG% .
                    docker tag %APP_NAME%:%IMAGE_TAG% %APP_NAME%:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying container: %CONTAINER_NAME%..."
                bat """
                    docker stop %CONTAINER_NAME% 2>nul || ver>nul
                    docker rm %CONTAINER_NAME% 2>nul || ver>nul
                    docker run -d --name %CONTAINER_NAME% -p %HOST_PORT%:%CONTAINER_PORT% %APP_NAME%:latest
                """
            }
        }

        stage('Health Check') {
            steps {
                echo 'Verifying container running status...'
                bat "docker ps -f name=%CONTAINER_NAME%"
            }
        }
    }

    post {
        always {
            echo 'Pruning dangling build cache...'
            bat 'docker image prune -f'
        }
        success {
            echo "Pipeline run #${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            echo "Pipeline run #${BUILD_NUMBER} failed. Review console logs."
        }
    }
}