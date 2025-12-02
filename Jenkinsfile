pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build (Maven)') {
            steps {
                bat 'mvnw.cmd clean package'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t myapp:latest .'
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    // Stop old container if running
                    bat 'docker stop myapp-container || echo "No running container"'

                    // Remove old container
                    bat 'docker rm myapp-container || echo "No container to remove"'

                    // Start new container
                    bat 'docker run -d -p 9090:8080 --name myapp-container myapp:latest'
                }
            }
        }
    }

    post {
        success {
            echo "Build Successful!"
        }
        failure {
            echo "Build Failed."
        }
    }
}
