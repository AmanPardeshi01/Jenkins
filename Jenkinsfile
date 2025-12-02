pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/AmanPardeshi01/Jenkins']]
                ])
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
                bat 'docker run -d -p 8080:8080 --name myapp-container myapp:latest'
            }
        }
    }

    post {
        always {
            cleanWs()
            echo "Cleaning workspace..."
        }
        success {
            echo "Build Successful."
        }
        failure {
            echo "Build Failed."
        }
    }
}
