pipeline {
  agent any

  stages {

    stage('Build') {
      steps {
        sh 'docker build -t shadow1234090/my-cicd-project:${BUILD_NUMBER} .'
      }
    }

    stage('Push') {
      steps {
        sh 'docker push shadow1234090/my-cicd-project:${BUILD_NUMBER}'
      }
    }

    stage('Update Manifest') {
      steps {
        sh '''
        sed -i "s|image:.*|image: shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml
        git add .
        git commit -m "updated image"
        git push
        '''
      }
    }
  }
}
