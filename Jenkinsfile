pipeline {
  agent any

  triggers {
    githubPush()
  }

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

    stage('Skip Jenkins Commit') {
      steps {
        script {
          def msg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()

          if (msg.contains('[skip ci]')) {
            echo "🚫 Jenkins generated commit detected — skipping pipeline"
            currentBuild.result = 'SUCCESS'
            return
          }
        }
      }
    }

    stage('Build Image') {
      steps {
        echo "🐳 Building Docker image..."
        sh 'docker build -t $DOCKERHUB/$IMAGE:${BUILD_NUMBER} .'
      }
    }

    stage('Push Image') {
      steps {
        echo "📤 Pushing Docker image..."
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
        }

        sh 'docker push $DOCKERHUB/$IMAGE:${BUILD_NUMBER}'
      }
    }

    stage('Update Deployment') {
      steps {
        echo "📝 Updating Kubernetes deployment..."

        sh '''
          sed -i "s|image:.*|image: shadow1234090/my-cicd-project:${BUILD_NUMBER}|g" k8s/deployment.yaml

          git config --global user.email "jenkins@local"
          git config --global user.name "jenkins"

          git add k8s/deployment.yaml
          git commit -m "Update image to ${BUILD_NUMBER} [skip ci]" || echo "No changes"

          git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/anees-rehman1/my-cicd-project.git HEAD:main
        '''
      }
    }
  }

  post {
    always {
      sh 'docker system prune -f || true'
    }

    failure {
      echo "❌ Build failed"
    }

    success {
      echo "✅ Pipeline completed successfully"
    }
  }
}
