pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // Jenkins credential ID for Docker Hub (username/password)
    DOCKERHUB_REPO = 'YOUR_DOCKERHUB_USER/spring-sample' // change this
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    MAVEN_ARGS = '-B -DskipTests' // change to '' if you want tests run in CI
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup') {
      steps {
        script {
          // show which shell we will use
          if (isUnix()) {
            sh 'echo "Running on Unix agent"'
          } else {
            bat 'echo Running on Windows agent'
          }
        }
      }
    }

    stage('Build (Maven)') {
      steps {
        script {
          if (isUnix()) {
            sh "mvn ${env.MAVEN_ARGS} clean package"
          } else {
            // on Windows agent - assume mvn available on PATH
            bat "mvn %MAVEN_ARGS% clean package"
          }
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          if (isUnix()) {
            sh "docker build -t ${env.DOCKERHUB_REPO}:${env.IMAGE_TAG} ."
          } else {
            // Windows: use double quotes for variables in bat block
            bat "docker build -t %DOCKERHUB_REPO%:%IMAGE_TAG% ."
          }
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            if (isUnix()) {
              sh '''
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
                docker push ${DOCKERHUB_REPO}:latest
              '''
            } else {
              bat '''
                echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                docker push %DOCKERHUB_REPO%:%IMAGE_TAG%
                docker tag %DOCKERHUB_REPO%:%IMAGE_TAG% %DOCKERHUB_REPO%:latest
                docker push %DOCKERHUB_REPO%:latest
              '''
            }
          }
        }
      }
    }

    stage('Deploy (local Docker)') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              docker rm -f spring-sample || true
              docker run -d --name spring-sample -p 8080:8080 ${DOCKERHUB_REPO}:${IMAGE_TAG}
            '''
          } else {
            bat '''
              docker rm -f spring-sample || echo "no existing container"
              docker run -d --name spring-sample -p 8080:8080 %DOCKERHUB_REPO%:%IMAGE_TAG%
            '''
          }
        }
      }
    }

  }

  post {
    always {
      archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
      echo "Build finished (status: ${currentBuild.currentResult})"
    }
    failure {
      echo "Pipeline failed — check console output"
    }
  }
}
