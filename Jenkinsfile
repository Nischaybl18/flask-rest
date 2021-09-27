pipeline {
  environment {
    DOCKERHUB_CREDENTIALS = credentials('nischaybl18-dockerhub')
  }
  agent any
  stages {
    stage('Build') {
      steps {
        sh "docker build -t nischaybl18/flaskapp:latest ."
      }
    }
    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push nischaybl18/flaskapp:latest'
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
