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
        git checkout main
        git pull origin main

        sed -i "s|image:.*|image: shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml

        git config user.email "jenkins@local"
        git config user.name "jenkins"

       git add k8s/deployment.yaml
      git commit -m "image update ${BUILD_NUMBER}" || true
        '''

         withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
         sh '''
         git remote set-url origin https://$GIT_USER:$GIT_PASS@github.com/anees-rehman1/my-cicd-project.git
         git push origin main
         '''
    }
  }
}

  }
}
