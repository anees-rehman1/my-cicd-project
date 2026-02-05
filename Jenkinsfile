pipeline {
  agent any

  environment {
    DOCKERHUB = "shadow1234090"
    IMAGE = "my-cicd-project"
  }

  stages {

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
      }
    }

    stage('Update Deployment') {
      steps {
        sh '''
        sed -i "s|image:.*|image: shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml
        git config --global user.email "jenkins@local"
        git config --global user.name "jenkins"
        git add .
        git commit -m "image update"
        git push
        '''
      }
    }
  }
}
