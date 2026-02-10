pipeline {
  agent any

  environment {
    DOCKERHUB = "shadow1234090"
    IMAGE = "my-cicd-project"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t $DOCKERHUB/$IMAGE:${BUILD_NUMBER} .'
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
        }
        sh 'docker push $DOCKERHUB/$IMAGE:${BUILD_NUMBER}'
        sh 'docker tag $DOCKERHUB/$IMAGE:${BUILD_NUMBER} $DOCKERHUB/$IMAGE:latest'
        sh 'docker push $DOCKERHUB/$IMAGE:latest'
      }
    }

    stage('Deploy') {
      steps {
        sh 'kubectl apply -f k8s/'
      }
    }
  }
}
