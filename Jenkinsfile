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

        // ✅ Replace your old Docker Run stage with this
        stage('Docker Run') {
            steps {
                // 🔹 Removes old container if it exists
                bat 'docker rm -f myapp-container || echo "No previous container"'

                // 🔹 Runs new container
                bat 'docker run -d -p 9090:8080 --name myapp-container myapp:latest'
            }
        }

    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Build Successful!"
        }
        failure {
            echo "Build Failed."
        }
    }
}
