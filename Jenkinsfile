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
          // Get the current commit hash
          def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
          echo "Current commit: ${currentCommit}"
          
          // Check if last-built-commit.txt exists
          def lastBuiltCommitFile = 'last-built-commit.txt'
          def fileExists = fileExists(lastBuiltCommitFile)
          
          if (fileExists) {
            // Read the last built commit
            def lastBuiltCommit = readFile(lastBuiltCommitFile).trim()
            echo "Last built commit: ${lastBuiltCommit}"
            
            // Skip build if no changes
            if (currentCommit == lastBuiltCommit) {
              echo "No changes detected. Skipping build."
              currentBuild.result = 'SUCCESS'
              error("No changes, build skipped")
            } else {
              echo "Changes detected! Building..."
            }
          } else {
            echo "First time build. Creating commit tracking file."
            // Create the file with current commit
            writeFile file: lastBuiltCommitFile, text: currentCommit
          }
        }
      }
    }

    stage('Build Image') {
      steps {
        script {
          // Only run if we have changes
          echo "Building Docker image..."
          sh 'docker build -t $DOCKERHUB/$IMAGE:${BUILD_NUMBER} .'
        }
      }
    }

    stage('Push Image') {
      steps {
        script {
          echo "Pushing Docker image..."
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh 'echo $PASS | docker login -u $USER --password-stdin'
          }
          sh 'docker push $DOCKERHUB/$IMAGE:${BUILD_NUMBER}'
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
        // Save the current commit hash on successful build
        def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
        writeFile file: 'last-built-commit.txt', text: currentCommit
        echo "Saved commit hash: ${currentCommit}"
      }
    }
    failure {
      echo "Build failed! Not updating commit hash."
    }
    always {
      // Clean up Docker images to save space
      sh 'docker system prune -f || true'
    }
  }
}
