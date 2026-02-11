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
          def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
          echo "Current commit: ${currentCommit}"
          
          def lastBuiltCommitFile = 'last-built-commit.txt'
          def fileExists = fileExists(lastBuiltCommitFile)
          
          if (fileExists) {
            def lastBuiltCommit = readFile(lastBuiltCommitFile).trim()
            echo "Last built commit: ${lastBuiltCommit}"
            
            if (currentCommit == lastBuiltCommit) {
              echo "No changes detected. Skipping build."
              currentBuild.result = 'SUCCESS'
              error("No changes, build skipped")
            } else {
              echo "Changes detected! Building..."
            }
          } else {
            echo "First time build. Creating commit tracking file."
            writeFile file: lastBuiltCommitFile, text: currentCommit
          }
        }
      }
    }

    stage('Build Image') {
      steps {
        script {
          echo "Building Docker image..."
          // USING SUDO FOR DOCKER
          sh 'sudo docker build -t $DOCKERHUB/$IMAGE:${BUILD_NUMBER} .'
        }
      }
    }

    stage('Push Image') {
      steps {
        script {
          echo "Pushing Docker image..."
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh 'echo $PASS | sudo docker login -u $USER --password-stdin'
          }
          // USING SUDO FOR DOCKER
          sh 'sudo docker push $DOCKERHUB/$IMAGE:${BUILD_NUMBER}'
        }
      }
    }

    stage('Update Deployment') {
      steps {
        script {
          echo "Updating Kubernetes deployment file..."
          sh '''
            sed -i "s|image:.*|image: shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml
            git config --global user.email "jenkins@local"
            git config --global user.name "jenkins"
            git add k8s/deployment.yaml
            git commit -m "Update image to version ${BUILD_NUMBER} [skip ci]" || echo "No changes to commit"
            git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/anees-rehman1/my-cicd-project.git HEAD:main
          '''
        }
      }
    }
  }

  post {
    success {
      script {
        def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
        writeFile file: 'last-built-commit.txt', text: currentCommit
        echo "Saved commit hash: ${currentCommit}"
      }
    }
    failure {
      echo "Build failed! Not updating commit hash."
    }
    always {
      // USING SUDO FOR DOCKER
      sh 'sudo docker system prune -f || true'
    }
  }
}
