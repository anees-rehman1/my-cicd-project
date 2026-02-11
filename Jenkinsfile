pipeline {
  agent any

  environment {
    DOCKERHUB = "shadow1234090"
    IMAGE = "my-cicd-project"
    GIT_CREDENTIALS = credentials('github-credentials')
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Check for Changes') {
      steps {
        script {
          // Get the last built commit from Jenkins
          def lastCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
          def lastBuiltCommit = readFile('last-built-commit.txt').trim()
          
          // Skip build if no changes
          if (lastCommit == lastBuiltCommit) {
            echo "No changes detected. Skipping build."
            currentBuild.result = 'SUCCESS'
            error("No changes, build skipped")
          }
        }
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
      }
    }

    stage('Update Deployment') {
      steps {
        script {
          sh '''
            sed -i "s|image:.*|image: shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml
            git config --global user.email "jenkins@local"
            git config --global user.name "jenkins"
            git add k8s/deployment.yaml
            git commit -m "Update image to version ${BUILD_NUMBER} [skip ci]"
            git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/anees-rehman1/my-cicd-project.git HEAD:main
          '''
        }
      }
    }
  }

  post {
    success {
      script {
        // Save the current commit hash
        def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
        writeFile file: 'last-built-commit.txt', text: currentCommit
      }
    }
  }
}
