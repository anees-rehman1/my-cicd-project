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

    stage('Update Deployment') {
      steps {
        sh '''
          sed -i "s|shadow1234090/my-cicd-project:.*|shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml
        '''
        script {
          withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
            sh """
              git config --global user.email "jenkins@local"
              git config --global user.name "jenkins"
              git remote set-url origin https://${GIT_TOKEN}@github.com/anees-rehman1/my-cicd-project.git
              git add k8s/deployment.yaml
              git commit -m "Update image to version ${BUILD_NUMBER}"
              git push origin HEAD:main
            """
          }
        }
      }
    }
  }
}
