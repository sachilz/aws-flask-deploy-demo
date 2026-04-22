pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_URL = "313772261399.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE = "flask-app"

        SONAR_SCANNER = "C:\\DevSecOps\\sonar-scanner\\bin\\sonar-scanner.bat"
        TRIVY_PATH = "C:\\DevSecOps\\trivy\\trivy.exe"
    }
    
    stages {

        stage('Build') {
            steps {
                echo 'Building the Docker image...'
                bat "docker-compose build"
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    bat """
                    \"${SONAR_SCANNER}\" ^
                    -Dsonar.projectKey=flask-app ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=http://localhost:9000 ^
                    -Dsonar.login=%SONAR_TOKEN%
                    """
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                bat "\"${TRIVY_PATH}\" image --severity HIGH,CRITICAL %IMAGE%:latest"
            }
        }

        stage('ECR Login') {
            steps {
                withCredentials([aws(
                    credentialsId: 'aws-creds',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    bat """
                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_URL%
                    """
                }
            }
        }

        stage('Tag Image') {
            steps {
                bat """
                docker tag %IMAGE%:latest %ECR_URL%/%IMAGE%:latest
                """
            }
        }

        stage('Push to ECR') {
            steps {
                bat """
                docker push %ECR_URL%/%IMAGE%:latest
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }
        success {
            echo 'Deployment successful.'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
