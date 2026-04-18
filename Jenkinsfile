pipeline {
    agent any

    environment {
        SONAR_SCANNER = "C:\\DevSecOps\\sonar-scanner\\bin\\sonar-scanner.bat"
        TRIVY_PATH = "C:\\DevSecOps\\trivy\\trivy.exe"
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Building the Docker image...'
                bat 'docker build -t pyimg .'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    bat """
                    \"${SONAR_SCANNER}\" ^
                    -Dsonar.projectKey=user-management-app ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=http://localhost:9000 ^
                    -Dsonar.login=%%SONAR_TOKEN%%
                    """
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                bat "\"${TRIVY_PATH}\" image --severity HIGH,CRITICAL pyimg"
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                bat 'docker-compose up -d'
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