pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // Jenkins credential ID (username/password or token)
    DOCKERHUB_REPO = 'YOUR_DOCKERHUB_USER/spring-sample' // change this
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${env.DOCKERHUB_REPO}:${env.IMAGE_TAG}")
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', env.DOCKERHUB_CREDENTIALS) {
            docker.image("${env.DOCKERHUB_REPO}:${env.IMAGE_TAG}").push()
            // also tag 'latest' optionally
            docker.image("${env.DOCKERHUB_REPO}:${env.IMAGE_TAG}").push('latest')
          }
        }
      }
    }

    stage('Deploy (Restart Container)') {
      steps {
        // This example deploys on the Jenkins host by stopping previous container and running a new one.
        // Adjust for remote host or Kubernetes as needed.
        sh """
          docker rm -f spring-sample || true
          docker pull ${env.DOCKERHUB_REPO}:${env.IMAGE_TAG}
          docker run -d --name spring-sample -p 8080:8080 ${env.DOCKERHUB_REPO}:${env.IMAGE_TAG}
        """
      }
    }
  }

  post {
    always {
      junit '**/target/surefire-reports/*.xml'  // if you run tests producing reports
      archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
    }
    failure {
      echo "Build failed!"
    }
  }
}
